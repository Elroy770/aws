---
lesson: 28
title: SQS and SNS
domain: Design Resilient Architectures
services: [SQS, SNS, Lambda, EC2 Auto Scaling, S3, CloudWatch, KMS]
tags: [saa-c03, integration, messaging, decoupling, queue, pubsub]
---

# 28 — SQS and SNS

> [!abstract] בשורה אחת
> SQS הוא **תור** שבו כל הודעה נצרכת פעם אחת על ידי צרכן אחד, SNS הוא **pub/sub** שבו כל מנוי מקבל עותק —
> ורוב שאלות ה-integration במבחן נפתרות ברגע שמזהים איזה משני המודלים השאלה מתארת.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה תקשורת סינכרונית נשברת ב-spike, ומה decoupling פותר |
| 2 | איך זה עובד | מחזור החיים של הודעה ב-SQS, Visibility Timeout, Polling, SNS Topic |
| 3 | פירוק מפורט | Standard מול FIFO, DLQ, Delay Queue, Long Polling, Filtering, אבטחה |
| 4 | עלות | חיוב ב-chunks של 64KB, מה מייקר, איך חוסכים |
| 5 | השוואות | SQS מול SNS, Standard מול FIFO, SNS+SQS מול SNS לבד |
| 6 | Well-Architected | messaging לפי ששת ה-Pillars |
| 7 | מלכודות | "SQS הוא broadcast" זו הטעות מספר 1 |
| 8 | Scenario | מערכת הזמנות עם fan-out, DLQ ו-ASG שמתרחב לפי עומק התור |

**מונחי מפתח בשיעור:** `Visibility Timeout` · `Long Polling` · `Dead Letter Queue` · `MessageGroupID` · `Deduplication ID` · `Fan Out` · `Filter Policy` · `Idempotency`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- שני שירותים מדברים **סינכרונית**: Buying Service קורא ישירות ל-Shipping Service ומחכה לתשובה.
- ברגע שיש spike — למשל צריך לקודד 1,000 וידאו במקום 10 הרגילים — הצד המקבל קורס.
- כשהצד המקבל נופל, **גם הצד השולח נופל איתו**: הבקשה נכשלת, המשתמש רואה שגיאה.
- אי אפשר לשחרר גרסה של שירות אחד בלי לתאם downtime עם השני.
- שני השירותים חייבים להתרחב **באותו קצב** — מה שכמעט אף פעם לא נכון.

### מה השירות פותר

- **Decoupling** — המפיק (producer) והצרכן (consumer) לא מכירים זה את זה ולא רצים באותו זמן.
- **ספיגת spikes** — התור סופג את הפיק, והצרכנים מעבדים בקצב שהם מסוגלים לו.
- **התרחבות עצמאית** — כל צד מתרחב לפי העומס שלו בלבד.
- **עמידות** — צרכן שקרס באמצע עיבוד לא מאבד את ההודעה; היא חוזרת לתור.
- **Fan-out** — אירוע אחד שנשלח פעם אחת מגיע לכמה מערכות שונות, בלי שהמפיק ידע עליהן.

> [!tip] האנלוגיה
> **SQS הוא תור בבנק:** יש דלפק אחד או עשרה, כל לקוח מטופל על ידי פקיד אחד בלבד, וכשעמוס — התור מתארך.
> **SNS הוא רמקול בתחנה:** ההודעה נאמרת פעם אחת וכל מי שנמצא בתחנה שומע אותה בו-זמנית.
> מי שלא היה בתחנה באותו רגע — פשוט פספס.

---

## 2. ⚙️ איך זה עובד

### 2.1 שני דגמי התקשורת

```text
סינכרוני (מסוכן ב-spike)
Buying Service  ──HTTP call──►  Shipping Service
                (מחכה לתשובה; אם השני נפל — גם הראשון נכשל)

אסינכרוני / מבוסס אירועים
Buying Service  ──►  Queue / Topic  ──►  Shipping Service
                (מחזיר תשובה מיד; הצד השני מעבד מתי שהוא יכול)
```

### 2.2 SQS — מחזור החיים של הודעה

```text
1. Producer   → SendMessage    → ההודעה נשמרת בתור (persisted)
2. Consumer   → ReceiveMessage → מקבל עד 10 הודעות; ההודעה הופכת ל-invisible
3. Consumer   → מעבד (למשל insert ל-RDS)
4. Consumer   → DeleteMessage  → רק עכשיו ההודעה נעלמת סופית
```

- **ההודעה נשארת בתור עד שצרכן מוחק אותה במפורש.** אין מחיקה אוטומטית אחרי קריאה.
- אם הצרכן קרס בין שלב 2 ל-4 — ההודעה **חוזרת להיות גלויה** ותעובד שוב על ידי צרכן אחר.
- זו בדיוק הסיבה שהמערכת עמידה, וגם הסיבה שהעיבוד חייב להיות **idempotent**.

### 2.3 Visibility Timeout — ציר זמן

זהו החלון שבו ההודעה "מוסתרת" משאר הצרכנים כדי שרק צרכן אחד יעבד אותה.
**ברירת המחדל: 30 שניות.** הטווח החוקי: 0 שניות עד 12 שעות.

```text
t=0s    Consumer A: ReceiveMessage  →  קיבל את ההודעה
        ├─────────── Visibility Timeout = 30s ───────────┤
t=5s    Consumer B: ReceiveMessage  →  ✗ לא מקבל כלום (ההודעה מוסתרת)
t=20s   Consumer C: ReceiveMessage  →  ✗ לא מקבל כלום
t=25s   Consumer A: DeleteMessage   →  ✓ ההודעה נעלמה. סוף הסיפור.

תרחיש הכישלון:
t=0s    Consumer A: ReceiveMessage
t=30s   ⏰ ה-timeout פג ו-A עדיין מעבד
t=31s   Consumer B: ReceiveMessage  →  ✓ מקבל את אותה הודעה!
        ← עכשיו שני צרכנים מעבדים את אותה הזמנה = חיוב כפול
```

**איך מונעים את זה:**

| הבעיה | הסימפטום | הפתרון |
|---|---|---|
| Visibility Timeout **קצר מדי** | הודעות מעובדות פעמיים, duplicates | להאריך את ה-timeout, או לקרוא ל-`ChangeMessageVisibility` תוך כדי עיבוד |
| Visibility Timeout **ארוך מדי** (שעות) | צרכן שקרס — ההודעה תקועה שעות עד שתחזור | לקצר; לכוון ל-~פי 2–6 מזמן העיבוד הממוצע |

> [!tip] הכלל המעשי
> קובעים את ה-Visibility Timeout מעט מעל **זמן העיבוד המקסימלי** של הודעה,
> ולעיבוד שאורכו משתנה — קוראים ל-**`ChangeMessageVisibility`** כדי לבקש עוד זמן תוך כדי ריצה.

### 2.4 Short Polling מול Long Polling

- **Short Polling** (ברירת מחדל): הצרכן שואל, ואם התור ריק — מקבל תשובה ריקה **מיד** וחוזר לשאול.
  התוצאה: אלפי API calls ריקים בשנייה, שכולם מחויבים.
- **Long Polling**: הצרכן מבקש **לחכות** עד שתגיע הודעה, עד `WaitTimeSeconds` שניות.

| מאפיין | Short Polling | Long Polling |
|---|---|---|
| זמן המתנה | 0 | **1–20 שניות** (20 מומלץ) |
| API calls ריקים | המון | כמעט אפס |
| Latency בקבלת הודעה | מיידית אבל בזבזנית | **נמוך יותר בפועל** — ההודעה נמסרת ברגע שהיא מגיעה |
| עלות | גבוהה מיותר | **נמוכה** |
| איפה מגדירים | ברירת מחדל | ברמת התור (`ReceiveMessageWaitTimeSeconds`) או ברמת ה-API |

> [!info] שורה תחתונה
> **Long Polling תמיד עדיף.** אם בשאלה כתוב "reduce the number of empty API responses" או
> "reduce SQS costs for a mostly-empty queue" — התשובה היא Long Polling עם 20 שניות.

### 2.5 SNS — Topic ו-Subscribers

```text
                      ┌──► SQS Queue
                      ├──► Lambda Function
Publisher ──publish──►│──► Amazon Data Firehose
          (SNS Topic) ├──► HTTP(S) Endpoint
                      ├──► Email / Email-JSON
                      └──► SMS / Mobile Push
```

- המפיק שולח **פעם אחת ל-topic אחד** ולא יודע כלל מי המנויים.
- כל מנוי (subscription) מקבל **עותק של כל הודעה** — אלא אם הוגדר לו Filter Policy.
- **עד 12,500,000 מנויים ל-topic.** **עד 100,000 topics לחשבון.**
- **SNS לא שומר נתונים.** מנוי שלא היה זמין ולא הצליח לקבל — ההודעה אבודה לגביו.
  זו הסיבה המרכזית לדפוס SNS + SQS.

### 2.6 מי שולח ל-SNS מתוך AWS

הרבה שירותים יודעים לפרסם ל-SNS ישירות, וזה מה שהופך אותו לצומת ההתראות של החשבון:

- **CloudWatch Alarms** — ההתראה הקלאסית.
- **AWS Budgets** — חריגה מתקציב.
- **Auto Scaling Group Notifications** — launch/terminate של instance.
- **S3 Bucket Events** — יצירה/מחיקה של אובייקט.
- **RDS Events** · **DynamoDB** · **CloudFormation State Changes** · **AWS DMS**.
- **Lambda** ו-SDK של כל אפליקציה.

### 2.7 SNS + SQS — דפוס ה-Fan Out

```text
                        ┌──► SQS Queue  ──► Fraud Service
Buying Service ──PUT──► SNS Topic
                        ├──► SQS Queue  ──► Shipping Service
                        └──► SQS Queue  ──► Billing Service
```

**למה זה הדפוס הנכון ולא לשלוח לשלושה תורים ידנית:**

| השוואה | Option 1: המפיק שולח ל-3 תורים | Option 2: Fan Out עם SNS |
|---|---|---|
| מספר קריאות מהמפיק | 3 (או N) | **1** |
| כשל אחרי הקריאה השנייה | חוסר עקביות — שניים קיבלו, אחד לא | **אטומי מבחינת המפיק** |
| הוספת צרכן רביעי | שינוי קוד + deploy של המפיק | **הוספת subscription בלבד** |
| צימוד | המפיק מכיר את כל הצרכנים | **מנותק לחלוטין** |

**נקודות שנשאלות על Fan Out:**

- **חובה ש-Access Policy של ה-SQS Queue יאפשר ל-SNS לכתוב אליו.** זו הטעות התפעולית הנפוצה.
- **Cross-Region Delivery** נתמך — ה-SNS Topic יכול לפרסם ל-SQS ב-Region אחר.
- כל תור נותן **persistence, delayed processing ו-retries** משלו — כשל בשירות אחד לא נוגע באחרים.

### 2.8 יישום: S3 Events לכמה תורים

- ב-S3, לאותו שילוב של **event type** (למשל `ObjectCreated`) ו-**prefix** (למשל `images/`)
  אפשר להגדיר **כלל אחד בלבד**.
- אם צריך שאותו אירוע יגיע לשלושה תורים — לא מגדירים שלושה כללים, אלא:

```text
S3 Bucket ──event──► SNS Topic ──┬──► SQS Queue (thumbnails)
                                 ├──► SQS Queue (virus scan)
                                 └──► Lambda   (metadata index)
```

---

## 3. 🔍 פירוק מפורט

### 3.1 SQS Standard — המספרים שחייבים לזכור

| מאפיין | ערך | הערה למבחן |
|---|---|---|
| Throughput | **בלתי מוגבל** | אין מה לספק (provision) מראש |
| מספר הודעות בתור | **בלתי מוגבל** | |
| **Retention — ברירת מחדל** | **4 ימים** | |
| **Retention — מקסימום** | **14 ימים** | מינימום: 60 שניות |
| Latency | פחות מ-10ms ב-publish וב-receive | |
| **גודל הודעה מקסימלי** | **1,024 KB** לפי חומר הקורס | ראו הערת הדיוק למטה |
| מסירה | **At-least-once** | ייתכנו כפילויות מדי פעם |
| סדר | **Best-effort ordering** | אין הבטחה |
| קבלה בבת אחת | **עד 10 הודעות** לכל `ReceiveMessage` | בסיס ל-batching |

> [!warning] הערת דיוק — גודל ההודעה
> חומר הקורס נוקב ב-**1,024 KB**. במשך שנים המגבלה הרשמית של SQS הייתה **256 KB**,
> ושאלות מבחן ותיקות עדיין בנויות סביב 256 KB.
> **מה שבאמת נבדק:** אם ההודעה גדולה מהמגבלה — משתמשים ב-**SQS Extended Client Library**,
> ששומר את ה-payload ב-**S3** ומעביר בתור רק מצביע אליו. זו התשובה לכל שאלה על "large payloads".

### 3.2 SQS Standard מול FIFO

| קריטריון | Standard Queue | FIFO Queue |
|---|---|---|
| **Throughput** | בלתי מוגבל | **300 msg/s** ללא batching · **3,000 msg/s** עם batching |
| **סדר** | Best-effort — לא מובטח | **מובטח** בתוך אותו Message Group |
| **כפילויות** | at-least-once, ייתכנו | **exactly-once send** באמצעות Deduplication |
| **שם התור** | חופשי | **חייב להסתיים ב-`.fifo`** |
| **Deduplication ID** | לא קיים | מזהה ייחודי; הודעה זהה בחלון של **5 דקות** נזרקת |
| Content-Based Dedup | לא | אופציה — hash של גוף ההודעה משמש כ-Dedup ID |
| **MessageGroupID** | לא קיים | **פרמטר חובה.** קובע את יחידת הסדר והמקביליות |
| מקביליות בצריכה | ללא הגבלה | צרכן אחד פעיל לכל Message Group |
| מתי בוחרים | ברירת המחדל — כמעט תמיד | רק כשהסדר או מניעת כפילות הם **דרישה עסקית** |

> [!tip] MessageGroupID — הרעיון שהכי כדאי להבין
> הסדר מובטח **בתוך קבוצה**, לא בכל התור.
> אם `MessageGroupID = customer-id`, אז ההזמנות של כל לקוח מעובדות בסדר,
> אבל לקוחות שונים מעובדים **במקביל**. ככה מקבלים גם סדר וגם throughput.
> קבוצה אחת לכל התור = סדר מושלם אבל צרכן אחד בלבד = צוואר בקבוק.

### 3.3 Dead Letter Queue (DLQ)

**הבעיה:** הודעה פגומה (poison pill) שגורמת לצרכן לקרוס תמיד. היא חוזרת לתור, קורסת שוב,
וחוזרת חלילה — וחוסמת את התור למשך ימים.

**הפתרון:** מגדירים על התור **Redrive Policy** עם שני שדות:

| שדה | מה זה |
|---|---|
| `deadLetterTargetArn` | ה-ARN של התור שיקבל את ההודעות הכושלות |
| `maxReceiveCount` | כמה פעמים הודעה יכולה להתקבל לפני שהיא מועברת ל-DLQ |

```text
הודעה נכשלת:
receive #1 → כשל → חוזרת לתור
receive #2 → כשל → חוזרת לתור
receive #3 → כשל → maxReceiveCount=3 הושג
             └──► ההודעה עוברת ל-DLQ ולא חוסמת יותר את התור הראשי
```

**כללים שחייבים לדעת:**

- ה-DLQ של תור **FIFO חייב להיות FIFO**; של תור **Standard חייב להיות Standard**.
- **מגדירים ל-DLQ retention ארוך — עדיף 14 ימים** — כדי שיהיה זמן לחקור.
- **DLQ Redrive** — פיצ'ר שמאפשר להחזיר את ההודעות מה-DLQ לתור המקורי אחרי שתיקנתם את הבאג,
  בלי לכתוב קוד.
- **CloudWatch Alarm על `ApproximateNumberOfMessagesVisible` של ה-DLQ** — זו ההתראה התפעולית
  שכל ארכיטקטורה חייבת.

### 3.4 Delay Queue

- מעכבת את ההופעה של הודעות חדשות בתור — הצרכנים פשוט לא רואים אותן.
- **ברירת מחדל: 0 שניות. מקסימום: 15 דקות.**
- מוגדר ברמת התור (`DelaySeconds`), וניתן לדריסה **לכל הודעה בנפרד** בעת השליחה.
- **Use case:** לתת ל-DB זמן להשלים commit לפני שהעיבוד האסינכרוני קורא את הרשומה.

> [!warning] Delay Queue מול Visibility Timeout
> **Delay** = ההודעה מוסתרת **לפני** שמישהו קיבל אותה אי פעם (עד 15 דקות).
> **Visibility Timeout** = ההודעה מוסתרת **אחרי** שצרכן קיבל אותה (עד 12 שעות).
> שני מנגנונים שונים לגמרי שקל לבלבל ביניהם.

### 3.5 SQS + Auto Scaling Group

זהו הדפוס שחוזר כמעט בכל scenario של עיבוד אסינכרוני:

```text
Front-end (ASG)  ──SendMessage──►  SQS Queue  ◄──poll──  Back-end (ASG)
                                       │
                                       ▼
                       CloudWatch Metric: ApproximateNumberOfMessages
                                       │
                                       ▼
                              CloudWatch Alarm  ──►  Scaling Policy
```

- המטריקה שעליה מתרחבים היא **`ApproximateNumberOfMessages`** — עומק התור.
- מטריקה טובה יותר לפרודקשן: **backlog per instance** = עומק התור חלקי מספר ה-instances.
- **`ApproximateAgeOfOldestMessage`** — המטריקה שמזהה שהצרכנים לא מדביקים את הקצב.

### 3.6 SQS כ-buffer לכתיבות ל-DB

**הבעיה:** אפליקציה כותבת ישירות ל-RDS. בפיק, ה-DB מגיע לרוויה ו**עסקאות הולכות לאיבוד**.

**הפתרון:** מכניסים תור בין האפליקציה ל-DB.

```text
לפני:   requests → App (ASG) ──insert──► RDS     ← בפיק: כתיבות נכשלות
אחרי:   requests → App (ASG) ──enqueue──► SQS ──dequeue──► Worker (ASG) ──insert──► RDS
```

- התור סופג את הפיק במקום ה-DB.
- קצב הכתיבה ל-DB נקבע על ידי מספר ה-workers — כלומר **הפך לנשלט**.
- אף בקשה לא הולכת לאיבוד; היא רק מחכה.

### 3.7 SNS FIFO ו-Message Filtering

**SNS FIFO Topic:**

- אותם מאפיינים כמו SQS FIFO: **MessageGroupID** לסדר, **Deduplication ID** לכפילויות.
- **ה-throughput זהה למגבלת SQS FIFO.**
- המנויים יכולים להיות תורי **SQS FIFO וגם SQS Standard**.
- הדפוס `SNS FIFO + SQS FIFO` נותן **fan-out + סדר + דדופליקציה** יחד.

**SNS Message Filtering:**

- מדיניות JSON שמוגדרת **על ה-subscription** ולא על ה-topic.
- **מנוי בלי Filter Policy מקבל את כל ההודעות.**
- מסננים לפי message attributes (או לפי גוף ההודעה ב-payload-based filtering).

```text
SNS Topic (Order Events)
├── Filter {State: "Placed"}    ──► SQS Queue: הזמנות חדשות
├── Filter {State: "Cancelled"} ──► SQS Queue: ביטולים
├── Filter {State: "Declined"}  ──► SQS Queue: סירובים
├── Filter {State: "Cancelled"} ──► Email Subscription
└── (ללא filter)                ──► SQS Queue: הכול, לאנליטיקה
```

- **Use case קלאסי:** לצמצם עלות ורעש — במקום שכל צרכן יקבל הכול ויזרוק 90%, הוא מקבל רק את שלו.

### 3.8 אבטחה — SQS ו-SNS זהים כמעט לחלוטין

| שכבה | SQS | SNS |
|---|---|---|
| **הצפנה בתעבורה** | HTTPS API | HTTPS API |
| **הצפנה במנוחה** | **KMS keys** (SSE-SQS או SSE-KMS) | **KMS keys** |
| הצפנת client-side | הלקוח מצפין ומפענח בעצמו | הלקוח מצפין ומפענח בעצמו |
| **בקרת גישה** | IAM Policies על ה-API | IAM Policies על ה-API |
| **Resource Policy** | **SQS Access Policy** | **SNS Access Policy** |

**למה ה-Resource Policy קריטי (בדומה ל-S3 Bucket Policy):**

- **גישה cross-account** לתור או ל-topic.
- **לאפשר לשירותי AWS אחרים לכתוב** — S3 שכותב ל-topic, SNS שכותב לתור, EventBridge, CloudWatch.
- שאלה קלאסית: "S3 events לא מגיעים לתור" → **חסר SQS Access Policy שמתיר ל-S3/SNS לכתוב.**

### 3.9 גישה פרטית מ-VPC

- גם SQS וגם SNS הם שירותים **ציבוריים** (public endpoints).
- כדי לגשת אליהם מ-subnet פרטי **בלי NAT Gateway** — משתמשים ב-**Interface VPC Endpoint (PrivateLink)**.
- פירוט ב-[[12 - VPC Private Connectivity]].

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **SQS Requests** | לכל API request (Send/Receive/Delete/Change) | **כל 64KB של payload = request נפרד לחיוב** |
| **SQS FIFO Requests** | אותה שיטה, **תעריף גבוה יותר** | FIFO יקר יותר ל-request מ-Standard |
| **SQS Data Transfer** | GB יוצא מה-Region | תעבורה בתוך אותו Region בין שירותי AWS — זולה/חינם |
| **SQS Storage** | **0** | אחסון ההודעות עצמו לא מחויב בנפרד |
| **SNS Publish** | לכל בקשת publish | |
| **SNS Deliveries** | **לפי סוג היעד** | וזה ההבדל הגדול — ראו למטה |
| **DLQ** | כמו כל תור רגיל | הודעה שעברה ל-DLQ כבר נצרכה כמה פעמים = כמה requests |

> [!warning] כלל ה-64KB — נקודה שמפילה חישובי עלות
> SQS מחייב לפי **chunks של 64KB**, לא לפי הודעות.
>
> | גודל הודעה | כמה requests לחיוב | הסבר |
> |---|---|---|
> | 10 KB | **1** | פחות מ-chunk אחד |
> | 64 KB | **1** | בדיוק chunk אחד |
> | **256 KB** | **4** | 256 ÷ 64 = 4 |
> | 1,024 KB | **16** | 1024 ÷ 64 = 16 |
>
> **המשמעות:** הודעה של 256KB עולה **פי 4** מהודעה של 60KB — על אותה פעולה לוגית.
> וזה מוכפל: Send + Receive + Delete הן שלוש פעולות נפרדות על אותה הודעה.

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| SQS Standard + Long Polling + batching | **הזולה ביותר** | ברירת המחדל |
| SQS Standard + Short Polling | גבוהה — API calls ריקים | אף פעם |
| **SQS FIFO** | יקר יותר ל-request ומוגבל ב-throughput | רק כשסדר/דדופליקציה הם דרישה |
| **SNS → SQS (Fan Out)** | מוסיף publish + delivery + כל תור | כמעט תמיד שווה את זה — עמידות ו-decoupling |
| **SNS → SMS** | **היקר ביותר בפער עצום** | התראות קריטיות בלבד |
| SNS → Email | יקר יחסית ל-SQS/Lambda | התראות תפעוליות |
| SNS → SQS / Lambda / HTTP | זול | הדפוס הרגיל בין שירותים |

### 🚩 עלויות נסתרות

- **Empty receives** — Short Polling על תור ריק מייצר מיליוני API calls מחויבים שלא החזירו כלום.
- **הודעות גדולות** — כל 64KB נוספים מוכפלים ב-3 (send, receive, delete) וב-N צרכנים ב-fan-out.
- **Fan-out מכפיל הכול** — הודעה אחת ל-topic עם 5 תורים = publish אחד + 5 deliveries + 5×3 פעולות SQS.
- **Retries שקטים** — הודעה שנכשלת 5 פעמים לפני DLQ נצרכה ומוחזרה 5 פעמים, וכל סיבוב מחויב.
- **SMS ו-Mobile Push** — עלות פר-הודעה גבוהה מאוד; לולאת התראות תקועה היא חשבונית מפתיעה.
- **Cross-Region fan-out** — SNS ל-SQS ב-Region אחר מוסיף data transfer בין Regions.

### 💡 טיפים לחיסכון

- **Long Polling עם `WaitTimeSeconds=20`** על כל תור. זה החיסכון הכי גדול ביחס למאמץ.
- **Batching** — `SendMessageBatch` ו-`ReceiveMessage` עם עד **10 הודעות** בבקשה אחת.
- **payload קטן בתור** — לשלוח מצביע ל-S3 במקום את הנתון עצמו (Claim Check pattern).
- **Filter Policy ב-SNS** — לא לשלוח לצרכן הודעות שהוא ממילא יזרוק.
- **Standard במקום FIFO** אלא אם יש דרישה עסקית מפורשת לסדר.
- **לקצר retention** לתורים שבהם הודעה ישנה ממילא חסרת ערך.
- **Alarm על DLQ** כדי לתפוס לולאות retry לפני שהן מצטברות.

---

## 5. ⚖️ השוואות מכריעות

### SQS מול SNS — הטבלה המרכזית

| קריטריון | **SQS** | **SNS** |
|---|---|---|
| **מודל** | Queue — point-to-point | **Pub/Sub** — broadcast |
| **מי צורך הודעה** | **צרכן אחד בלבד** | **כל המנויים**, כל אחד עותק |
| כיוון | **Pull** — הצרכן מושך | **Push** — SNS דוחף |
| **שמירת נתונים** | כן — עד **14 ימים** | **לא נשמר.** לא נמסר = אבד |
| מספר צרכנים | כמה שרוצים, מתחלקים בעבודה | עד **12,500,000 מנויים** |
| Provisioning | לא נדרש | לא נדרש |
| סדר | רק ב-**FIFO Queue** | רק ב-**FIFO Topic** |
| עיכוב הודעה | **כן** — Delay Queue עד 15 דקות | לא |
| Retry על כישלון | מובנה — ההודעה חוזרת לתור | מוגבל; ל-HTTP יש retry policy |
| **מילת מפתח בשאלה** | "decouple", "buffer", "worker", "process later" | "notify", "fan out", "multiple subscribers", "alert" |

### Standard מול FIFO — מתי משלמים את המחיר

| קריטריון | Standard | FIFO |
|---|---|---|
| Throughput | בלתי מוגבל | 300/s · 3,000/s עם batching |
| סדר | best-effort | מובטח בתוך Message Group |
| כפילויות | ייתכנו | נמנעות (חלון 5 דקות) |
| עלות | נמוכה יותר | גבוהה יותר |
| **מתי** | ברירת מחדל | תשלומים, ledger, state machine שרגיש לסדר |

### SNS לבד מול SNS + SQS

| קריטריון | SNS → Lambda ישירות | SNS → SQS → Lambda |
|---|---|---|
| עמידות לכשל בצרכן | ההודעה עלולה ללכת לאיבוד | **התור שומר אותה** |
| Backpressure | Lambda מוצף | **התור בולם** |
| Retry | מוגבל | מלא + **DLQ** |
| עיבוד מושהה | לא | **כן** |
| עלות | נמוכה יותר | תור נוסף לכל צרכן |
| **מתי** | התראה שאובדנה נסבל | **כל תהליך עסקי** |

> [!info] שורה תחתונה
> **הודעה שאסור לאבד → SQS.** **אירוע שכמה מערכות צריכות → SNS.**
> **שניהם ביחד → SNS + SQS Fan Out**, וזו התשובה הנכונה ברוב שאלות הארכיטקטורה.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-SQS/SNS** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | רואים את מצב התור לפני שהלקוחות מרגישים | Alarm על `ApproximateAgeOfOldestMessage` ועל עומק ה-DLQ; runbook ל-**DLQ Redrive** |
| **Security** | רק מי שצריך כותב וקורא, והנתון מוצפן | SSE-KMS על תורים ו-topics; SQS/SNS Access Policy מצומצם; Interface Endpoint במקום NAT |
| **Reliability** | קריסת צרכן לא מאבדת עבודה | Visibility Timeout מותאם לזמן העיבוד; DLQ עם `maxReceiveCount`; צרכנים **idempotent** |
| **Performance Efficiency** | לא נחנקים ולא מבזבזים סיבובים | Long Polling 20s; batch של 10; **Standard** אלא אם FIFO נדרש; ASG לפי backlog per instance |
| **Cost Optimization** | לא משלמים על אוויר | Long Polling נגד empty receives; payload קטן (64KB chunks); Filter Policy; להימנע מ-SMS מיותר |
| **Sustainability** | פחות מחזורי CPU ופחות רשת | batching מקטין invocations; buffering מאפשר עיבוד בקצב יעיל במקום צי מנופח לפיקים |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "decouple the tiers" / "buffer spikes" | **SQS** |
| "notify multiple systems" / "fan out" | **SNS** (בדרך כלל SNS + SQS) |
| "each message processed exactly once, in order" | **SQS FIFO** |
| "reduce the number of empty responses / API calls" | **Long Polling** (עד 20 שניות) |
| "messages that repeatedly fail" / "poison pill" | **Dead Letter Queue** + `maxReceiveCount` |
| "message processed twice" | **Visibility Timeout קצר מדי** או צרכן לא idempotent |
| "give the consumer more processing time" | **`ChangeMessageVisibility` API** |
| "delay processing by a few minutes" | **Delay Queue** (עד 15 דקות) |
| "scale workers based on backlog" | ASG על **`ApproximateNumberOfMessages`** |
| "message larger than the size limit" | **SQS Extended Client** — payload ב-**S3** |
| "same S3 event to multiple queues" | **SNS Fan Out** (יש רק כלל אחד לכל type+prefix) |
| "only some subscribers should get this event" | **SNS Filter Policy** |
| "on-premises app using MQTT / AMQP / STOMP" | **Amazon MQ** — ראו [[29 - Event-Driven Architecture]] |
| "replay the events later" | לא SQS ולא SNS → **Kinesis / EventBridge Archive** |

### טעויות נפוצות

> [!warning] מלכודת 1 — "SQS משדר לכולם"
> **הניסוח:** "Three microservices must all receive every order event. Use a single SQS queue with three consumers."
> **הטעות:** להניח שכל צרכן יקבל עותק.
> **הנכון:** ב-SQS **כל הודעה נצרכת פעם אחת בלבד** — שלושת הצרכנים יתחלקו בהודעות, כל אחד יקבל שליש.
> הפתרון: **SNS Topic + תור SQS נפרד לכל שירות**.

> [!warning] מלכודת 2 — לסמוך על SNS לבדו לתהליך עסקי
> **הניסוח:** "SNS delivers to a Lambda; when Lambda was down, some notifications were lost."
> **הטעות:** לחפש הגדרת retry ב-SNS.
> **הנכון:** **SNS לא שומר הודעות.** מוסיפים **SQS בין ה-topic ל-Lambda** — התור שומר עד שהצרכן חוזר.

> [!warning] מלכודת 3 — Delay Queue מול Visibility Timeout
> **הניסוח:** "Hide the message for 10 minutes before any consumer can see it for the first time."
> **הטעות:** לבחור Visibility Timeout.
> **הנכון:** **Delay Queue** — הוא פועל לפני הצריכה הראשונה, עד **15 דקות**.
> Visibility Timeout פועל רק **אחרי** ש-`ReceiveMessage` הוחזרה, ועד **12 שעות**.

> [!warning] מלכודת 4 — FIFO "כי זה יותר טוב"
> **הניסוח:** "We need to process 20,000 messages per second in order."
> **הטעות:** לבחור FIFO אוטומטית.
> **הנכון:** FIFO תקוע על **3,000 msg/s עם batching**. 20,000/s **לא אפשרי בתור FIFO יחיד**.
> או שמפצלים ל-Message Groups רבים, או שהדרישה לסדר לא באמת נחוצה ובוחרים Standard.

> [!warning] מלכודת 5 — לשכוח את ה-Access Policy
> **הניסוח:** "S3 event notifications configured, but nothing arrives in the SQS queue."
> **הטעות:** לחפש בהגדרות ה-S3.
> **הנכון:** צריך **SQS Access Policy** (resource policy) שמתיר ל-service principal של S3/SNS לכתוב.
> ה-IAM Role של המשתמש לא רלוונטי כאן.

> [!warning] מלכודת 6 — "Standard נותן exactly-once"
> **הניסוח:** "Charge the customer's credit card from an SQS Standard queue."
> **הטעות:** להניח שכל הודעה תעובד בדיוק פעם אחת.
> **הנכון:** Standard הוא **at-least-once**. הצרכן **חייב להיות idempotent** —
> למשל לשמור `order-id` שכבר טופל ב-DynamoDB ולבדוק לפני החיוב. אחרת: חיוב כפול.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חנות אונליין. כשמתבצעת הזמנה צריך: לעדכן מלאי, לחייב, לשלוח מייל אישור,
ולהריץ בדיקת הונאה. אסור שכשל בשירות המייל יעכב את החיוב.
בשיא מבצע ה-Black Friday יש פי 50 הזמנות מהרגיל.
בדיקת ההונאה חייבת לעבד את פעולות אותו לקוח **לפי הסדר**.

**הארכיטקטורה:**

```text
                                     ┌──► SQS Standard ──► Inventory Workers (ASG)  ──► DLQ
                                     │
Client ─► ALB ─► Order API ──publish─┤    SNS Topic
              (ASG)                  ├──► SQS Standard ──► Billing Workers (ASG)    ──► DLQ
                                     │
                                     ├──► SQS Standard ──► Email Lambda             ──► DLQ
                                     │    (Filter: State = "Placed")
                                     │
                                     └──► SQS FIFO ────► Fraud Service              ──► DLQ FIFO
                                          (MessageGroupID = customer-id)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **SNS Topic אחד** לאירוע ההזמנה | ה-Order API מפרסם פעם אחת ולא מכיר את ארבעת הצרכנים |
| **תור SQS נפרד לכל שירות** | בידוד מלא — כשל במייל לא נוגע בחיוב; לכל תור retry ו-DLQ משלו |
| **Standard** לשלושה מהתורים | throughput בלתי מוגבל; פי 50 עומס נספג בלי provisioning |
| **FIFO** רק לבדיקת ההונאה | היחיד עם דרישת סדר אמיתית — משלמים את מגבלת ה-throughput רק שם |
| `MessageGroupID = customer-id` | סדר לכל לקוח, אבל **לקוחות שונים מעובדים במקביל** — לא צוואר בקבוק |
| **Filter Policy** על תור המיילים | לא נשלח מייל על הזמנות שבוטלו; חוסך invocations ורעש |
| **DLQ עם `maxReceiveCount=3`** לכל תור | הודעה פגומה לא חוסמת את התור; Alarm מתריע |
| **retention 14 ימים ב-DLQ** | זמן לחקור ואז **Redrive** בחזרה אחרי תיקון |
| **ASG על `ApproximateNumberOfMessages`** | מספר ה-workers עוקב אחרי עומק התור בפועל |
| **Long Polling 20s** בכל הצרכנים | ברוב שעות היממה התורים כמעט ריקים — חוסך את רוב חשבון ה-API |
| **צרכנים idempotent** על `order-id` | Standard הוא at-least-once; בלי זה יהיו חיובים כפולים |

**למה לא לקרוא ישירות מה-Order API לארבעת השירותים?**
כי אז ה-API מחכה לארבעתם, ותקלה באחד מפילה את ההזמנה כולה.
בנוסף, הוספת שירות חמישי דורשת deploy של ה-API.

**למה לא תור SQS אחד עם ארבעה צרכנים?**
כי ב-SQS **כל הודעה נצרכת פעם אחת**. השירותים היו מתחלקים בהזמנות במקום שכל אחד יקבל את כולן.

**למה לא SNS ישירות ל-Lambda בלי תור?**
כי אם ה-Lambda נכשל או throttled, **SNS לא שומר** את ההודעה. התור נותן persistence, retry ו-DLQ.

**למה לא הכול FIFO?**
כי FIFO תקוע על 3,000 msg/s ויקר יותר. משלמים על סדר רק במקום היחיד שבו הוא נדרש.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **תחביר ה-SDK והפרמטרים המדויקים** של `SendMessage` / `ReceiveMessage`.
- **המבנה המדויק של Filter Policy JSON** — מספיק לדעת שהיא קיימת ומה היא עושה.
- **KPL ו-KCL** לעומק — מספיק לדעת שהן ספריות לאופטימיזציה של producer/consumer ב-Kinesis.
- **תמחור מדויק בדולרים** — משתנה לפי Region. חשוב להבין את **מנגנון** ה-64KB.
- **פרוטוקולי mobile push** (GCM, APNS, ADM) לעומק — מספיק לדעת ש-SNS תומך ב-Direct Publish.
- **ההגדרות הפנימיות** של short polling ואיזה שרתים נדגמים.
- **מגבלות quota מדויקות** שהן soft limits.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **SQS = queue.** הודעה נצרכת על ידי **צרכן אחד**. **SNS = pub/sub.** כל מנוי מקבל **עותק**.
- **SQS Standard:** throughput בלתי מוגבל · **at-least-once** · **best-effort ordering**.
- **Retention:** ברירת מחדל **4 ימים**, מקסימום **14 ימים**.
- **`ReceiveMessage` מחזיר עד 10 הודעות.** **ההודעה נמחקת רק ב-`DeleteMessage`.**
- **Visibility Timeout: ברירת מחדל 30 שניות, עד 12 שעות.** קצר מדי = כפילויות; ארוך מדי = תקיעות.
- **`ChangeMessageVisibility`** = לבקש עוד זמן עיבוד.
- **Long Polling: 1–20 שניות.** מפחית API calls ריקים ומוריד latency. **תמיד עדיף.**
- **Delay Queue: עד 15 דקות**, לפני הצריכה הראשונה. לא לבלבל עם Visibility Timeout.
- **FIFO: 300 msg/s · 3,000 עם batching.** שם התור **חייב להסתיים ב-`.fifo`**.
- **MessageGroupID = יחידת הסדר.** **Deduplication ID = חלון של 5 דקות**.
- **DLQ = `maxReceiveCount` + `deadLetterTargetArn`.** DLQ של FIFO חייב להיות FIFO. **retention 14 יום**.
- **הודעה גדולה מהמגבלה → SQS Extended Client עם payload ב-S3.**
- **SNS: עד 12,500,000 מנויים ל-topic, עד 100,000 topics.** **SNS לא שומר נתונים.**
- **Fan Out = SNS + SQS.** חובה **SQS Access Policy** שמתיר ל-SNS לכתוב. עובד **cross-Region**.
- **אותו S3 event לכמה תורים → fan-out** (יש רק כלל אחד לכל event type + prefix).
- **Filter Policy על ה-subscription.** בלי policy — המנוי מקבל הכול.
- **חיוב SQS ב-chunks של 64KB:** הודעה של **256KB = 4 requests**.
- **צרכני Standard חייבים להיות idempotent.** at-least-once אינו exactly-once.

---

## 11. ✅ בדיקת הבנה

1. שלושה שירותים צריכים כל אחד לקבל כל אירוע הזמנה. תור SQS אחד עם שלושה צרכנים — יעבוד?
2. עיבוד הודעה לוקח בממוצע 45 שניות. מה הבעיה עם ההגדרות ברירת המחדל, ומה מתקנים?
3. תור כמעט תמיד ריק, וחשבון ה-SQS גבוה בצורה מוזרה. מה קורה ומה הפתרון?
4. הודעה אחת פגומה חוזרת לתור כבר יומיים וחוסמת workers. מה מגדירים?
5. צריך לעבד 20,000 הודעות בשנייה עם סדר מלא בתור FIFO יחיד. אפשרי?
6. הגדרתם S3 Event Notification לתור SQS ושום דבר לא מגיע. מה בודקים ראשון?
7. מה ההבדל בין Delay Queue ל-Visibility Timeout?
8. כמה SQS requests יחויבו על שליחה של הודעה בגודל 256KB?
9. למה SNS ישירות ל-Lambda מסוכן יותר מ-SNS → SQS → Lambda?
10. `MessageGroupID` נקבע לערך קבוע אחד לכל התור. מה יקרה?

<details>
<summary>תשובות</summary>

1. **לא.** ב-SQS כל הודעה נצרכת **פעם אחת בלבד** — שלושת הצרכנים יתחלקו בהודעות.
   הפתרון: **SNS Topic + תור SQS נפרד לכל שירות** (Fan Out).
2. ברירת המחדל של **Visibility Timeout היא 30 שניות** — כלומר ההודעה תיהפך גלויה שוב
   באמצע העיבוד ותעובד **פעמיים**. מעלים את ה-timeout (למשל 120 שניות),
   או קוראים ל-**`ChangeMessageVisibility`** תוך כדי עיבוד.
3. **Short Polling** — הצרכנים שואלים ללא הרף ומקבלים תשובות ריקות, וכל בקשה כזו מחויבת.
   הפתרון: **Long Polling** עם `WaitTimeSeconds=20`.
4. **Dead Letter Queue** עם **Redrive Policy**: `maxReceiveCount` (למשל 3) ו-`deadLetterTargetArn`.
   כדאי גם retention של 14 ימים ב-DLQ ו-CloudWatch Alarm על עומק ה-DLQ.
5. **לא.** FIFO מוגבל ל-**3,000 msg/s עם batching**. או שמפצלים ל-**Message Groups** רבים
   (סדר בתוך כל קבוצה, מקביליות ביניהן), או שבוחרים **Standard** אם הסדר לא באמת דרישה.
6. **ה-SQS Access Policy** — צריך resource policy שמתירה ל-service principal של S3 לכתוב לתור.
   זו הסיבה הנפוצה ביותר.
7. **Delay Queue** מסתיר הודעה **לפני** שנצרכה אי פעם, עד **15 דקות**, ומוגדר ברמת התור או ההודעה.
   **Visibility Timeout** מסתיר הודעה **אחרי** שצרכן קיבל אותה, עד **12 שעות**.
8. **4 requests** — SQS מחייב ב-chunks של **64KB**, ו-256 ÷ 64 = 4.
   (וזה חוזר על עצמו ב-Receive וב-Delete.)
9. כי **SNS לא שומר הודעות**. אם ה-Lambda נכשל, throttled או מושבת — ההודעה אבודה.
   SQS באמצע נותן **persistence, retries ו-DLQ**.
10. הסדר יהיה מושלם, אבל **צרכן אחד בלבד** יוכל לעבד בכל רגע — כלומר **צוואר בקבוק** חמור.
    בוחרים MessageGroupID שמפצל לוגית (למשל `customer-id`) כדי לקבל סדר **ומקביליות**.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[29 - Event-Driven Architecture]] · [[30 - Application Decoupling]] · [[25 - Lambda]] · [[27 - API Gateway]] · [[07 - Auto Scaling]] · [[18 - S3 Advanced Features]] · [[33 - High Availability and Scalability]] · [[38 - Serverless and Modern Architectures]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 6772–7300, 15257–15300
