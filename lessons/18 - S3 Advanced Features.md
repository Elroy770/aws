---
lesson: 18
title: S3 Advanced Features
domain: Design Cost-Optimized Architectures
services: [S3, S3 Lifecycle, EventBridge, SQS, SNS, Lambda, S3 Storage Lens]
tags: [saa-c03, storage, s3, lifecycle, performance, automation, cost]
---

# 18 — S3 Advanced Features

> [!abstract] בשורה אחת
> Lifecycle Rules הן מנוע החיסכון של S3, Event Notifications הן מנוע האוטומציה שלו,
> וכללי הביצועים (prefix, multipart, byte-range) הם התשובה לכל שאלת "העלאה איטית".

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה ניהול ידני של אובייקטים לא מתאפשר בקנה מידה |
| 2 | איך זה עובד | **מדרג המעברים החוקיים**, Lifecycle Rules, Storage Class Analysis |
| 3 | פירוק מפורט | Event Notifications, EventBridge, Requester Pays, Batch Operations, Storage Lens |
| 4 | עלות | transitions, acceleration, replicas, multipart נטוש |
| 5 | השוואות | SNS/SQS/Lambda מול EventBridge · Multipart מול Transfer Acceleration |
| 6 | Well-Architected | אוטומציה וביצועים לפי ששת ה-Pillars |
| 7 | מלכודות | לא ניתן לעלות במדרג · אין exactly-once |
| 8 | Scenario | pipeline עיבוד מדיה מלא |

**מונחי מפתח בשיעור:** `Lifecycle Rule` · `Transition` · `Expiration` · `noncurrent version` · `prefix` · `Multi-Part Upload` · `Byte-Range Fetch` · `Transfer Acceleration` · `Batch Operations` · `Storage Lens`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- ב-bucket יש 400 מיליון אובייקטים. אף אחד לא ילך להחליף להם storage class ידנית.
- 80% מהנתונים לא נגעו בהם מזה שנה — ועדיין משלמים עליהם מחיר Standard.
- גרסאות ישנות ו-multipart uploads נטושים מצטברים בשקט ומנפחים את החשבון.
- כל העלאת קובץ אמורה להפעיל עיבוד — ולא רוצים לבנות poller שסורק את ה-bucket.
- העלאת קובץ 20 GB מאוסטרליה ל-Region בארה"ב לוקחת שעות, ואם היא נקטעת מתחילים מחדש.
- אין מושג איזה bucket גדל הכי מהר ואיפה מתחבא הבזבוז.

### מה השירות פותר

- **Lifecycle Rules** — מדיניות שמעבירה ומוחקת אובייקטים אוטומטית לפי גיל.
- **Storage Class Analysis** — מודדת דפוסי גישה בפועל ואומרת **מתי** כדאי להעביר.
- **Event Notifications** — S3 דוחפת אירוע ל-SNS/SQS/Lambda/EventBridge בכל שינוי.
- **Multi-Part Upload + Transfer Acceleration + Byte-Range Fetches** — פתרונות הביצועים.
- **Batch Operations** — פעולה אחת על מיליוני אובייקטים קיימים.
- **Storage Lens** — דשבורד ארגוני שמראה איפה הכסף מתבזבז.

> [!tip] האנלוגיה
> Lifecycle Rule הוא **חוזה עם המחסן**: "כל ארגז שלא נגעו בו 30 יום — יורד למרתף,
> וכל ארגז שלא נגעו בו שנה — נזרק."
> אתם כותבים אותו פעם אחת, והוא מנהל מיליארד ארגזים בלי שתעשו כלום.

---

## 2. ⚙️ איך זה עובד

### 2.1 מדרג המעברים — מה חוקי ומה לא

**הכלל המרכזי: אפשר רק לרדת במדרג. לא לעלות.**

```text
                         Standard
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
 Intelligent-Tiering ──▶ Standard-IA ──▶ One Zone-IA
        │                   │                   │
        └─────────┬─────────┴─────────┬─────────┘
                  ▼                   │
        Glacier Instant Retrieval     │
                  │                   │
                  └────────┬──────────┘
                           ▼
              Glacier Flexible Retrieval
                           │
                           ▼
                 Glacier Deep Archive
                    (סוף הדרך)
```

| מ- | אפשר לעבור ל- |
|---|---|
| **Standard** | כל השאר |
| **Intelligent-Tiering** | Standard-IA, One Zone-IA, Glacier IR, Glacier Flexible, Deep Archive |
| **Standard-IA** | Intelligent-Tiering, One Zone-IA, Glacier IR, Glacier Flexible, Deep Archive |
| **One Zone-IA** | Glacier Flexible, Deep Archive |
| **Glacier Instant Retrieval** | Glacier Flexible, Deep Archive |
| **Glacier Flexible Retrieval** | Deep Archive |
| **Glacier Deep Archive** | **לשום מקום** |

> [!warning] שלושת הכללים שמפילים אנשים במבחן
> 1. **אי אפשר לעלות במדרג ב-Lifecycle.** להחזיר אובייקט מ-Glacier ל-Standard
>    דורש **Restore** ואז **העתקה** ל-key חדש — לא כלל lifecycle.
> 2. **מינימום 30 יום ב-Standard** לפני מעבר ל-Standard-IA או One Zone-IA.
>    כלל שאומר "העבר ל-IA אחרי 10 ימים" פשוט לא יתבצע.
> 3. **אובייקטים קטנים מ-128 KB לא מועברים** ל-Standard-IA / One Zone-IA.
>    S3 מדלגת עליהם — כי המינימום המחויב הוא ממילא 128 KB והמעבר לא היה חוסך כלום.

### 2.2 Lifecycle Rules — שני סוגי פעולות

**Transition Actions — העברה בין storage classes:**

- "העבר ל-Standard-IA 60 יום אחרי היצירה."
- "העבר ל-Glacier לארכוב אחרי 6 חודשים."

**Expiration Actions — מחיקה:**

| מה מוחקים | דוגמה |
|---|---|
| אובייקטים ישנים | קבצי לוג נמחקים אחרי 365 יום |
| **גרסאות ישנות** (noncurrent versions) | רק אם versioning מופעל |
| **Multi-Part Uploads שלא הושלמו** | ניקוי חלקים נטושים שמחויבים בשקט |
| **Delete Markers מיותמים** | delete marker שאין מתחתיו אף גרסה |

**על מה חל הכלל — הגדרת ה-scope:**

| מסנן | דוגמה |
|---|---|
| **Prefix** | `s3://mybucket/mp3/*` — רק תיקיית ה-mp3 |
| **Object Tags** | `Department: Finance` — רק אובייקטים מתויגים |
| **גודל אובייקט** | מינימום/מקסימום בבייטים |
| ללא מסנן | כל ה-bucket |

> [!tip] ה-lifecycle שכדאי בכל bucket, תמיד
> כלל שמבטל **Multi-Part Uploads שלא הושלמו** אחרי 7 ימים.
> החלקים האלה **מחויבים** ולא מופיעים ברשימת האובייקטים הרגילה — בזבוז שקט לחלוטין.

### 2.3 Lifecycle — תרחיש 1 (תמונות ו-thumbnails)

**הדרישה:**
אפליקציה ב-EC2 מייצרת thumbnails מתמונות פרופיל שהועלו ל-S3.
ה-thumbnails **ניתנים ליצירה מחדש** בקלות ונחוצים רק ל-60 יום.
תמונות המקור חייבות להיות זמינות **מיידית** ל-60 יום, ואחרי זה מותר לחכות **עד 6 שעות**.

**הפתרון:**

| נכס | Storage Class | כלל Lifecycle | ההנמקה |
|---|---|---|---|
| **תמונות מקור** | Standard | **Transition** ל-Glacier Flexible Retrieval אחרי 60 יום | 60 יום גישה מיידית; אחר כך Standard retrieval של 3–5 שעות עומד בדרישת 6 השעות |
| **Thumbnails** | **One Zone-IA** | **Expiration** (מחיקה) אחרי 60 יום | ניתנים ליצירה מחדש → אין צורך בעמידות רב-AZ. אין טעם לארכב משהו שאפשר לחשב מחדש |

> [!warning] למה Glacier Flexible ולא Deep Archive?
> Deep Archive מתחיל ב-**12 שעות**. הדרישה היא **6 שעות**. Flexible עם Standard retrieval
> (3–5 שעות) עומד בה, ו-Expedited (1–5 דקות) נותן מרווח ביטחון.

### 2.4 Lifecycle — תרחיש 2 (שחזור מחיקות)

**הדרישה:**
צריך לשחזר אובייקטים שנמחקו **באופן מיידי במשך 30 יום**.
לאחר מכן, ועד 365 יום, השחזור צריך להיות אפשרי **תוך 48 שעות**.

**הפתרון:**

| שלב | פעולה | ההנמקה |
|---|---|---|
| 1 | **הפעלת Versioning** | "מחיקה" הופכת ל-**Delete Marker** שמסתיר את הגרסה. הגרסה עצמה שרדה |
| 2 | Transition של **noncurrent versions** ל-Standard-IA | 30 הימים הראשונים — שחזור מיידי, במחיר נמוך מ-Standard |
| 3 | Transition של noncurrent versions ל-**Glacier Deep Archive** | עד 365 יום. Bulk retrieval של 48 שעות עומד בדיוק בדרישה |
| 4 | (מומלץ) **Expiration** של noncurrent versions אחרי 365 יום | אחרת הן מצטברות לנצח |

> [!tip] המילה שמסגירה את התשובה
> **"noncurrent versions"** בכלל lifecycle היא סימן מובהק ששאלת השחזור נפתרת ב-versioning.
> בלי versioning אין בכלל "גרסה קודמת" לשמור.

### 2.5 Storage Class Analysis

- מנתחת דפוסי גישה בפועל ומייצרת **המלצה מתי** להעביר אובייקטים.
- מכסה **Standard ו-Standard-IA בלבד**.
- **לא עובדת** על One Zone-IA ולא על אף Glacier class.
- הדוח מתעדכן **יומית**, אבל צריך **24–48 שעות** עד שיש נתונים ראשונים.
- הפלט הוא קובץ **.csv** עם עמודות כמו Date, StorageClass, ObjectAge.

> [!tip] סדר הפעולות הנכון
> **קודם Storage Class Analysis, אחר כך Lifecycle Rules.**
> זה מונע מכם להעביר ל-IA נתונים שבפועל נקראים כל יום — ולשלם על retrieval יותר ממה שחסכתם.

---

## 3. 🔍 פירוק מפורט

### 3.1 Requester Pays

| מודל | מי משלם על אחסון | מי משלם על הורדה (networking) |
|---|---|---|
| **Bucket רגיל** | בעל ה-bucket | **בעל ה-bucket** |
| **Requester Pays** | בעל ה-bucket | **המבקש** |

- שימושי לשיתוף **datasets גדולים** עם חשבונות אחרים בלי לספוג את עלות התעבורה.
- **המבקש חייב להיות מאומת ב-AWS.** גישה אנונימית **לא אפשרית** ב-bucket כזה.
- בעל ה-bucket עדיין משלם על האחסון — רק ה-transfer עובר.

### 3.2 S3 Event Notifications

**אילו אירועים:**

| Event | מתי נורה |
|---|---|
| `s3:ObjectCreated:*` | PUT, POST, COPY, השלמת multipart |
| `s3:ObjectRemoved:*` | מחיקה או יצירת delete marker |
| `s3:ObjectRestore:*` | שחזור מ-Glacier — התחלה או סיום |
| `s3:Replication:*` | אירועי שכפול, כולל כשלים |

**עובדות:**

- אפשר **סינון לפי שם אובייקט** — למשל רק `*.jpg`, או רק prefix מסוים.
- אפשר להגדיר **כמה שרוצים** של אירועים על אותו bucket.
- **זמן מסירה:** בדרך כלל **שניות**, אבל לפעמים **דקה או יותר**.
- **אין ערובה ל-exactly-once ואין ערובה לסדר** — הצרכן חייב להיות **idempotent**.

**שלושת היעדים הישירים:**

```text
              ┌──▶ SNS Topic      →  fan-out לכמה מנויים
Amazon S3 ────┼──▶ SQS Queue      →  buffering, עיבוד לפי קצב
              └──▶ Lambda Function →  עיבוד מיידי ללא שרת
```

> [!warning] ההרשאה היא Resource Policy, לא IAM Role
> זו נקודה שנשאלת. S3 **דוחפת** את האירוע, ולכן היעד חייב **להתיר ל-S3 לכתוב אליו**:
>
> | יעד | מה מגדירים |
> |---|---|
> | SNS | **SNS Resource (Access) Policy** |
> | SQS | **SQS Resource (Access) Policy** |
> | Lambda | **Lambda Resource Policy** |
>
> לא IAM Role על ה-bucket. תשובה שמדברת על "IAM role for S3" היא בדרך כלל המסיחה.

### 3.3 S3 + Amazon EventBridge

אפשר לשלוח **את כל** אירועי ה-bucket ל-**EventBridge** במקום (או בנוסף) ליעדים הישירים.

| יכולת | מה זה נותן |
|---|---|
| **סינון מתקדם** | כללי JSON על metadata, גודל אובייקט, שם, ועוד |
| **יעדים מרובים** | **מעל 18 שירותי AWS** — Step Functions, Kinesis Data Streams/Firehose, ועוד |
| **Archive & Replay** | לשמור אירועים ולהריץ אותם מחדש — לדיבוג או לשחזור |
| **מסירה אמינה** | retry ו-DLQ מובנים |

> [!tip] מתי EventBridge ומתי notification ישיר
> **notification ישיר** מספיק ל-1→1 פשוט (העלאה → Lambda).
> **EventBridge** הוא התשובה כשמופיע *"multiple destinations"*, *"advanced filtering"*,
> *"replay events"*, או *"route to Step Functions"*.

### 3.4 Baseline Performance — הכללים שנשאלים

- S3 מתרחבת אוטומטית לקצבי בקשות גבוהים. Latency טיפוסי: **100–200 ms**.
- **לכל prefix ב-bucket** אפשר להשיג לפחות:

| סוג בקשה | קצב לכל prefix |
|---|---|
| PUT / COPY / POST / DELETE | **3,500 בשנייה** |
| GET / HEAD | **5,500 בשנייה** |

- **אין שום הגבלה על מספר ה-prefixes** ב-bucket.
- ה-prefix הוא כל מה שבין שם ה-bucket לשם הקובץ:

```text
bucket/folder1/sub1/file  →  prefix = /folder1/sub1/
bucket/folder1/sub2/file  →  prefix = /folder1/sub2/
bucket/1/file             →  prefix = /1/
bucket/2/file             →  prefix = /2/
```

> [!tip] החישוב שנשאל ישירות
> פיזור אחיד של הקריאות על **4 prefixes** נותן `4 × 5,500` = **22,000 בקשות GET/HEAD בשנייה**.
> **המסקנה המעשית:** נתקעתם בתקרת ביצועים? **הוסיפו prefixes ופזרו את המפתחות** —
> אין צורך בעוד bucket.

### 3.5 Multi-Part Upload

- מפצל קובץ גדול לחלקים שמועלים **במקביל**.
- **מומלץ מעל 100 MB. חובה מעל 5 GB.**
- יתרונות: מהירות (מקביליות), ו-**retry רק על החלק שנכשל** במקום על כל הקובץ.
- **חובה להשלים או לבטל** — חלקים שנשארו באוויר **ממשיכים להיות מחויבים**.
  לכן ה-lifecycle rule לניקוי multipart נטוש.

### 3.6 S3 Transfer Acceleration

- מעלים את הקובץ ל-**Edge Location** קרובה, ומשם AWS מעבירה אותו ל-bucket
  דרך **הרשת הפרטית שלה**.

```text
  קובץ בארה"ב  ──── מהיר (אינטרנט ציבורי, מרחק קצר) ────▶  Edge Location USA
                                                                 │
                                        מהיר (רשת AWS פרטית)     │
                                                                 ▼
                                                    S3 Bucket (Australia)
```

- **תואם ל-Multi-Part Upload** — אפשר ומומלץ לשלב את השניים.
- משתלם כשה-latency בין המעלה ל-Region **משמעותי** (יבשת אחרת).
- **לא עוזר** כשהמעלה כבר קרוב ל-Region — שם רק מוסיפים עלות.

### 3.7 Byte-Range Fetches

- קוראים **טווח בייטים ספציפי** מהאובייקט במקום את כולו.
- שני שימושים עיקריים:

| שימוש | איך |
|---|---|
| **האצת הורדה** | מבקשים טווחים שונים **במקביל** ומרכיבים בצד הלקוח |
| **קריאה חלקית** | מושכים רק את ה-**header** של קובץ ענק (X הבייטים הראשונים) בלי להוריד GB |

- בונוס: **עמידות בפני כשלים** — נפל טווח אחד, מנסים רק אותו מחדש.

### 3.8 S3 Batch Operations

פעולה אחת על **מיליוני אובייקטים קיימים**.

| מה אפשר לעשות |
|---|
| שינוי metadata ומאפיינים של אובייקטים |
| **העתקה** בין buckets |
| **הצפנת אובייקטים שלא היו מוצפנים** |
| שינוי ACLs ו-tags |
| **שחזור אובייקטים מ-Glacier** בהיקף גדול |
| **הפעלת Lambda** לפעולה מותאמת על כל אובייקט |

**מבנה Job:** רשימת אובייקטים + פעולה + פרמטרים אופציונליים.

**מה השירות מנהל בשבילכם:** retries, מעקב התקדמות, התראות סיום, ודוחות.

> [!tip] הצמד הקלאסי במבחן
> **S3 Inventory** מייצר את רשימת האובייקטים →
> **Athena** מסנן אותה (למשל "כל האובייקטים הלא מוצפנים") →
> **S3 Batch Operations** מריץ את הפעולה על הרשימה המסוננת.
> אם השאלה מדברת על *"encrypt all existing unencrypted objects"* — זו התשובה.

### 3.9 S3 Storage Lens

- דשבורד ל-**ניתוח ואופטימיזציה של אחסון בכל ה-Organization**.
- מזהה חריגות, הזדמנויות חיסכון, וסטיות מ-best practices של הגנת נתונים.
- **30 יום** של מדדי שימוש ופעילות בתצוגה.
- אגרגציה לפי: **Organization · Accounts · Regions · Buckets · Prefixes**.
- **Default Dashboard** — מוכן מראש, multi-Region ו-multi-Account.
  **לא ניתן למחוק אותו, רק להשבית.**
- ניתן לייצא מדדים יומית ל-S3 בפורמט **CSV או Parquet**.

**קטגוריות המדדים:**

| קטגוריה | דוגמאות למדדים | למה משמש |
|---|---|---|
| **Summary** | `StorageBytes`, `ObjectCount` | לזהות את ה-buckets שגדלים הכי מהר או לא בשימוש |
| **Cost-Optimization** | `NonCurrentVersionStorageBytes`, `IncompleteMultipartUploadStorageBytes` | לאתר multipart נטוש מעל 7 ימים, ומועמדים למעבר ל-class זול |
| **Data-Protection** | `VersioningEnabledBucketCount`, `MFADeleteEnabledBucketCount`, `SSEKMSEnabledBucketCount`, `CrossRegionReplicationRuleCount` | לזהות buckets שלא עומדים ב-best practices |
| **Access-Management** | `ObjectOwnershipBucketOwnerEnforcedBucketCount` | לראות אילו הגדרות Object Ownership בשימוש |
| **Event** | `EventNotificationEnabledBucketCount` | לדעת אילו buckets מחוברים לאוטומציה |
| **Performance** | `TransferAccelerationEnabledBucketCount` | לזהות היכן Acceleration דלוק |
| **Activity** | `AllRequests`, `GetRequests`, `PutRequests`, `BytesDownloaded` | להבין איך ניגשים לאחסון |
| **Detailed Status Code** | `200OKStatusCount`, `403ForbiddenErrorCount`, `404NotFoundErrorCount` | לאתר בעיות הרשאה וקישורים שבורים |

**Free מול Paid — טבלה שנשאלת:**

| קריטריון | **Free Metrics** | **Advanced Metrics & Recommendations** |
|---|---|---|
| זמינות | אוטומטית לכל הלקוחות | **בתשלום** |
| כמות מדדים | **כ-28 מדדי usage** | + Activity, Advanced Cost Optimization, Advanced Data Protection, Status Code |
| שמירת נתונים לשאילתות | **14 יום** | **15 חודשים** |
| פרסום ל-CloudWatch | ❌ | ✅ **ללא חיוב נוסף** על ה-CloudWatch |
| אגרגציה ברמת Prefix | ❌ | ✅ |

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **Lifecycle Transitions** | **לכל 1,000 אובייקטים שהועברו** | מיליוני אובייקטים קטנים = מעבר יקר מהחיסכון |
| **Transfer Acceleration** | תוספת לכל GB שהואץ | נוסף על ה-transfer הרגיל |
| **Multi-Part Upload נטוש** | אחסון של החלקים | **מחויב ולא נראה** ב-ListObjects |
| **Batch Operations** | לכל job + לכל מיליון אובייקטים | + עלות ה-Lambda אם מפעילים אותה |
| **Storage Lens Free** | **0** | 28 מדדים, 14 יום |
| **Storage Lens Advanced** | לכל מיליון אובייקטים מנוטרים לחודש | פרסום ל-CloudWatch כלול ללא תוספת |
| **Storage Class Analysis** | לכל מיליון אובייקטים מנוטרים לחודש | זול, ובדרך כלל משתלם |
| **S3 Inventory** | לכל מיליון אובייקטים ברשימה | + אחסון קובץ הפלט |
| **Event Notifications** | **0** מצד S3 | משלמים על היעד: SNS/SQS/Lambda/EventBridge |
| **Replication** | אחסון ביעד + transfer (ב-CRR) | ראו [[16 - S3 Fundamentals]] |
| **Requester Pays** | ה-**מבקש** משלם על requests ו-download | בעל ה-bucket עדיין משלם על אחסון |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **Lifecycle לניקוי multipart נטוש** | **0 עלות, חיסכון מיידי** | **תמיד** |
| **Lifecycle לאובייקטים גדולים** | זול — מעט transitions, הרבה GB | תמיד |
| **Lifecycle למיליוני אובייקטים זעירים** | יכול **לעלות יותר** מהחיסכון | לבדוק לפני |
| **Intelligent-Tiering** במקום lifecycle | עמלת monitoring, בלי transitions | כשהדפוס לא ידוע |
| **Transfer Acceleration** | תוספת ל-GB | רק כשהמרחק גדול והשיפור מדיד |
| **Byte-Range Fetches** | חוסך GB שלא הורדתם | קבצים גדולים שצריך מהם חלק |
| **Storage Lens Free** | **0** | תמיד להדליק |
| **Storage Lens Advanced** | לפי אובייקטים | ארגון גדול עם בזבוז לא ממופה |

### 🚩 עלויות נסתרות

- **Transitions על אובייקטים זעירים** — 100 מיליון אובייקטים של 5 KB שעוברים ל-Glacier
  יעלו יותר בעמלות המעבר מכל החיסכון באחסון.
- **מעבר מוקדם מדי** — אובייקט שעבר ל-Glacier ונמחק אחרי 20 יום מחויב על **90 יום**.
  ראו את טבלת ה-minimum duration ב-[[16 - S3 Fundamentals]].
- **Multi-Part נטוש** — הבזבוז השקט הנפוץ ביותר ב-S3.
- **Transfer Acceleration שנשאר דלוק** — כשהמעלים כבר קרובים ל-Region, זו תוספת נטו.
- **Replicas בלי lifecycle משלהם** — ה-bucket ביעד גדל באותו קצב כמו המקור.
  צריך lifecycle **גם שם** (למשל ישר ל-Deep Archive).
- **Event storm** — bucket עם מיליוני PUT ביום מייצר מיליוני הפעלות Lambda.
- **EventBridge Archive** — שמירת אירועים לצורך replay צוברת עלות אחסון.
- **S3 Inventory יומי** על bucket ענק — הדוח עצמו הוא אובייקט גדול.

### 💡 טיפים לחיסכון

- **תמיד** כלל lifecycle שמבטל multipart נטוש אחרי 7 ימים.
- **תמיד** כלל expiration ל-noncurrent versions ב-bucket עם versioning.
- **הדליקו Storage Lens (החינמי)** והסתכלו על `IncompleteMultipartUploadStorageBytes`
  ועל `NonCurrentVersionStorageBytes` — שם מתחבא הכסף.
- **Storage Class Analysis לפני Lifecycle** — כדי לא להעביר נתונים חמים ל-IA.
- **בדקו את גודל האובייקט הממוצע** לפני שמפעילים transitions המוניים.
- **Lifecycle גם על bucket היעד של ה-replication** — לא רק על המקור.
- **סינון באירועים** (`*.jpg`, prefix) — כדי לא להפעיל Lambda על כל אובייקט ב-bucket.
- **Transfer Acceleration** — הריצו את ה-speed comparison tool לפני שמדליקים.

---

## 5. ⚖️ השוואות מכריעות

### יעדי Event Notifications

| קריטריון | **SNS** | **SQS** | **Lambda** | **EventBridge** |
|---|---|---|---|---|
| דפוס | fan-out לכמה מנויים | תור עם buffering | עיבוד מיידי | routing מתקדם |
| שומר הודעות | לא (push) | ✅ עד שנצרכות | לא | Archive אופציונלי |
| סינון | Message Filtering בסיסי | לא (בצד הצרכן) | לא | ✅ **כללי JSON מתקדמים** |
| Replay | ❌ | ❌ | ❌ | ✅ **Archive & Replay** |
| מספר יעדים | מנויים מרובים | תור אחד | פונקציה אחת | **מעל 18 שירותים** |
| מה מגדירים ב-S3 | SNS Access Policy | SQS Access Policy | Lambda Resource Policy | הפעלת EventBridge על ה-bucket |
| מתי בוחרים | להודיע לכמה גורמים | לספוג פיקים ולעבד לפי קצב | טרנספורמציה פשוטה | *"multiple destinations"*, *"advanced filtering"*, Step Functions |

### Multi-Part Upload מול Transfer Acceleration מול Byte-Range

| קריטריון | Multi-Part Upload | Transfer Acceleration | Byte-Range Fetches |
|---|---|---|---|
| כיוון | **העלאה** | **העלאה** (בעיקר) | **הורדה** |
| מה פותר | קובץ גדול, retry יקר | **מרחק גיאוגרפי** מה-Region | הורדה איטית / צורך בחלק מהקובץ |
| מנגנון | פיצול לחלקים במקביל | Edge Location + רשת AWS פרטית | בקשות מקבילות לטווחי בייטים |
| עלות נוספת | אין | ✅ **יש** — לכל GB | אין |
| ניתן לשלב | ✅ עם Acceleration | ✅ עם Multi-Part | — |
| סף | מומלץ >100 MB, **חובה >5 GB** | כשה-latency משמעותי | קבצים גדולים |

### Lifecycle Rules מול Intelligent-Tiering

| קריטריון | Lifecycle Rules | Intelligent-Tiering |
|---|---|---|
| מי מחליט | **אתם**, לפי גיל האובייקט | **AWS**, לפי גישה בפועל |
| דורש ידיעת דפוס | ✅ כן | ❌ לא |
| עלות ניהול | עמלת transition לכל 1,000 אובייקטים | עמלת monitoring לכל 1,000 אובייקטים |
| Retrieval fees | לפי ה-class שאליו עברתם | **אין** |
| מתי בוחרים | דפוס **ידוע וצפוי** (חם 30 יום ואז קר) | דפוס **לא ידוע או משתנה** |

### Storage Class Analysis מול Storage Lens

| קריטריון | Storage Class Analysis | Storage Lens |
|---|---|---|
| Scope | **bucket אחד** | **Organization / accounts / regions / buckets / prefixes** |
| מטרה | מתי להעביר ל-Standard-IA | תמונת מצב כוללת: עלות, הגנה, פעילות |
| מכסה | Standard ו-Standard-IA בלבד | כל ה-classes |
| פלט | דוח **.csv** יומי | **דשבורד** + ייצוא CSV/Parquet |
| זמן עד נתונים | **24–48 שעות** | מיידי (Free) |

> [!info] שורה תחתונה
> **Lifecycle = חיסכון. Events = אוטומציה. Multipart/Acceleration/Byte-Range = ביצועים.**
> **Batch Operations = תיקון רטרואקטיבי. Storage Lens = איפה לחפש את הבזבוז.**

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בפיצ'רים המתקדמים של S3** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | האחסון מנהל את עצמו והבזבוז נראה | Lifecycle במקום ניקוי ידני; **Storage Lens** כדשבורד קבוע; מדדי replication lag; Batch Operations לתיקונים המוניים במקום סקריפטים |
| **Security** | האוטומציה לא פותחת דלתות | Resource Policies מינימליות ליעדי האירועים; Batch Operations להצפנת אובייקטים ישנים שהוחמצו; מדדי Data-Protection ב-Storage Lens לאיתור buckets בלי versioning/SSE |
| **Reliability** | עיבוד עמיד לכשלים ולכפילויות | צרכנים **idempotent** — S3 לא מבטיחה exactly-once ולא סדר; **SQS + DLQ** לספיגת פיקים; Multi-Part לחידוש חלקי בלבד |
| **Performance Efficiency** | ההעברה והקריאה מנוצלות במלואן | פיזור על **prefixes** מרובים (3,500 PUT / 5,500 GET לכל prefix); Multi-Part מעל 100 MB; Byte-Range להורדה מקבילית; Acceleration רק לנתיבים רחוקים |
| **Cost Optimization** | כל אובייקט יושב ב-class הנכון, וכלום לא מחויב לחינם | Lifecycle לניקוי multipart ו-noncurrent versions; Storage Class Analysis לפני החלטה; **Requester Pays** לשיתוף datasets גדולים; מדדי Cost-Optimization ב-Storage Lens |
| **Sustainability** | פחות בייטים מועברים ופחות עיבוד מיותר | עיבוד **event-driven** במקום polling; סינון אירועים כדי לא להפעיל Lambda לחינם; Byte-Range במקום הורדת קובץ שלם; ארכוב נתונים לא פעילים |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "automatically move objects after N days" | **Lifecycle Rule — Transition** |
| "delete old versions to save cost" | **Lifecycle Rule — Expiration על noncurrent versions** |
| "unexplained storage charges / hidden costs" | **Multi-Part Uploads שלא הושלמו** |
| "recover deleted objects for 30 days" | **Versioning** + lifecycle על noncurrent versions |
| "decide when to transition to Standard-IA" | **Storage Class Analysis** |
| "trigger processing when a file is uploaded" | **S3 Event Notifications** |
| "route events to Step Functions / multiple targets" | **EventBridge** |
| "advanced filtering on object size or metadata" | **EventBridge** |
| "replay past events" | **EventBridge Archive & Replay** |
| "uploads from another continent are slow" | **Transfer Acceleration** (+ Multi-Part) |
| "file larger than 5 GB" | **Multi-Part Upload** (חובה) |
| "download only the first bytes / header of a huge file" | **Byte-Range Fetch** |
| "hitting request rate limits on S3" | לפזר על **prefixes** נוספים |
| "encrypt all existing objects in the bucket" | **S3 Inventory → Athena → S3 Batch Operations** |
| "restore millions of objects from Glacier" | **S3 Batch Operations** |
| "share a large dataset without paying for egress" | **Requester Pays** |
| "analyze storage across the entire organization" | **S3 Storage Lens** |
| "identify buckets without versioning or encryption" | **Storage Lens — Data-Protection metrics** |
| "metrics available for 15 months" | **Storage Lens Advanced** (ה-Free הוא 14 יום) |

### טעויות נפוצות

> [!warning] מלכודת 1 — Lifecycle שמעלה במדרג
> **הניסוח:** "Create a lifecycle rule to move objects from Glacier back to S3 Standard when accessed."
> **הטעות:** להניח שהמדרג דו-כיווני.
> **הנכון:** **Lifecycle יורד בלבד.** להחזיר מ-Glacier דורש **Restore** ואז **העתקה** ל-key חדש.
> אם רוצים תנועה דו-כיוונית אוטומטית — זה **Intelligent-Tiering**.

> [!warning] מלכודת 2 — מעבר ל-IA אחרי פחות מ-30 יום
> **הניסוח:** "Transition objects to Standard-IA after 10 days to save money."
> **הטעות:** לבחור בזה כי המספר נשמע הגיוני.
> **הנכון:** יש **מינימום של 30 יום ב-Standard** לפני מעבר ל-Standard-IA או One Zone-IA.
> הכלל פשוט **לא יתבצע**.

> [!warning] מלכודת 3 — לצפות שאובייקטים קטנים יעברו
> **הניסוח:** "Millions of 10 KB log entries did not transition to Standard-IA. Why?"
> **הטעות:** לחפש בעיה בהרשאות או בכלל.
> **הנכון:** **S3 לא מעבירה אובייקטים קטנים מ-128 KB** ל-IA classes.
> ממילא הם היו מחויבים כ-128 KB, אז אין חיסכון. הפתרון הוא **לאחד אובייקטים** לפני האחסון.

> [!warning] מלכודת 4 — הנחת exactly-once באירועים
> **הניסוח:** "Each upload triggers the Lambda exactly once, so we can increment a counter."
> **הטעות:** להניח מסירה יחידה ומסודרת.
> **הנכון:** S3 Event Notifications יכולות להימסר **יותר מפעם אחת** ו**לא בסדר מובטח**.
> הצרכן חייב להיות **idempotent**, ורצוי SQS + DLQ באמצע.

> [!warning] מלכודת 5 — IAM Role במקום Resource Policy
> **הניסוח:** "Create an IAM role for S3 so it can publish to the SQS queue."
> **הטעות:** לחשוב במונחי IAM Role.
> **הנכון:** צריך **SQS Resource (Access) Policy** שמתירה ל-S3 לשלוח.
> אותו דבר ל-SNS Access Policy ול-Lambda Resource Policy.

> [!warning] מלכודת 6 — Transfer Acceleration תמיד מהיר יותר
> **הניסוח:** "Enable Transfer Acceleration to speed up uploads from EC2 in the same region."
> **הטעות:** להניח שזה תמיד שיפור.
> **הנכון:** Acceleration עוזר כשיש **מרחק גיאוגרפי**. כשהמקור כבר ב-Region —
> הוא **מוסיף עלות ולא מוסיף מהירות**.

> [!warning] מלכודת 7 — Replication כתחליף לגיבוי
> **הניסוח:** "We replicate to another region, so we're protected against accidental deletion."
> **הטעות:** להשוות replication ל-backup.
> **הנכון:** מחיקה **עם version ID לא משוכפלת**, ו-delete markers משוכפלים רק אם הגדרתם.
> ההגנה מפני מחיקה היא **Versioning + Object Lock** ([[17 - S3 Security and Data Management]]),
> לא replication.

> [!warning] מלכודת 8 — עוד bucket כדי לפרוץ תקרת בקשות
> **הניסוח:** "We hit 5,500 GET/s — create a second bucket and split the data."
> **הטעות:** להניח שהמגבלה היא ברמת ה-bucket.
> **הנכון:** המגבלה היא **לכל prefix**, ואין הגבלה על מספר ה-prefixes.
> מפזרים את המפתחות על prefixes נוספים באותו bucket — 4 prefixes = **22,000 GET/s**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

פלטפורמת וידאו עולמית.

- יוצרי תוכן מעלים קבצים של **2–30 GB** מכל העולם, כולל מאוסטרליה ל-Region בארה"ב.
- כל העלאה חייבת להפעיל pipeline של transcoding + יצירת thumbnails.
- ה-pipeline יכול להיכשל ולהיות מופעל שוב — אסור שיווצרו כפילויות.
- הגלם נחוץ מיידית 30 יום, ואחרי שנה מספיק שחזור תוך יממה.
- הנהלה רוצה דוח חודשי: מי ה-buckets שגדלים הכי מהר ואיפה הבזבוז.
- צריך להצפין רטרואקטיבית 200 מיליון אובייקטים ישנים שהועלו לפני שההצפנה נאכפה.

**הארכיטקטורה:**

```text
  יוצר תוכן (Australia)
        │  Multi-Part Upload + Transfer Acceleration
        ▼
   Edge Location  ──── רשת AWS פרטית ────▶  raw-bucket (us-east-1)
                                                │
                                    S3 Event: s3:ObjectCreated:*
                                       filter: *.mp4
                                                ▼
                                          EventBridge
                                                │
                                    ┌───────────┴───────────┐
                                    ▼                       ▼
                              SQS Queue  ─▶ Lambda    Step Functions
                              (+ DLQ)      transcode   (pipeline ארוך)
                                                │
                                                ▼
                                        derived-bucket (One Zone-IA)

  raw-bucket Lifecycle:
    יום 0–30    Standard
    יום 30      → Standard-IA
    יום 365     → Glacier Flexible Retrieval
    כל הזמן     abort incomplete multipart אחרי 7 ימים

  Storage Lens (Organization) ─▶ דוח יומי CSV ל-reporting-bucket

  תיקון רטרואקטיבי:
    S3 Inventory ─▶ Athena (סינון unencrypted) ─▶ S3 Batch Operations (Copy + SSE-KMS)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **Multi-Part Upload** | קבצים מעל 5 GB — **חובה**. גם נותן retry רק על החלק שנכשל |
| **Transfer Acceleration** | המעלים באוסטרליה וה-Region בארה"ב. **מרחק גיאוגרפי אמיתי** = שיפור מדיד |
| **Lifecycle: abort multipart אחרי 7 ימים** | קבצי 30 GB שנקטעים משאירים חלקים מחויבים ובלתי נראים |
| **Event Notification עם סינון `*.mp4`** | לא להפעיל את ה-pipeline על thumbnails, לוגים או קבצים זמניים |
| **EventBridge ולא notification ישיר** | צריך **כמה יעדים** (SQS ו-Step Functions) ו-**Archive & Replay** לדיבוג |
| **SQS באמצע + DLQ** | סופג פיקים של העלאות, מאפשר עיבוד לפי קצב, ותופס הודעות שנכשלו |
| **צרכן idempotent** | S3 **לא מבטיחה exactly-once**. עובדים לפי מפתח (bucket+key+versionId) |
| **derived-bucket ב-One Zone-IA** | thumbnails ו-transcodes **ניתנים ליצירה מחדש** מהגלם |
| **Lifecycle: יום 30 → Standard-IA** | 30 יום הוא בדיוק המינימום החוקי. מוקדם יותר לא היה עובד |
| **Lifecycle: שנה → Glacier Flexible** | הדרישה היא "יממה" — Bulk (5–12 שעות) עומד בה בנוחות |
| **לא Deep Archive** | Bulk שם הוא **48 שעות** — חורג מדרישת היממה |
| **Storage Lens ברמת Organization** | הדוח החודשי שההנהלה ביקשה, כולל `IncompleteMultipartUploadStorageBytes` |
| **Inventory → Athena → Batch Operations** | הדרך הסטנדרטית להצפין 200 מיליון אובייקטים קיימים בלי סקריפט משלכם |

**למה לא Intelligent-Tiering על ה-raw?**
דפוס הגישה כאן **ידוע**: חם 30 יום, קר אחר כך.
Intelligent-Tiering נועד לחוסר ודאות, וגובה עמלת monitoring על כל אובייקט לחינם כשהדפוס ברור.

**למה לא Lambda ישירות מ-S3, בלי SQS?**
Lambda ישירה עובדת, אבל אין buffering. פיק של אלפי העלאות בו-זמנית יגרום ל-throttling
ולאובדן retries. **SQS מנתקת את קצב ההעלאה מקצב העיבוד** ומוסיפה DLQ.

**למה לא סתם עוד bucket כשנגיע לתקרת בקשות?**
המגבלה היא **לכל prefix** — 3,500 PUT ו-5,500 GET.
מפזרים את המפתחות (למשל לפי hash של ה-creator ID) על עשרות prefixes ומקבלים
עשרות אלפי בקשות בשנייה **באותו bucket**.

**מה משלים את הפתרון?**
בחירת ה-storage classes ומשמעויות העלות ב-[[16 - S3 Fundamentals]],
ההצפנה וה-Access Points ב-[[17 - S3 Security and Data Management]],
ותבניות ה-decoupling ב-[[28 - SQS and SNS]] וב-[[29 - Event-Driven Architecture]].

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **גודל החלק (part size) המדויק** ב-Multi-Part Upload ומספר החלקים המקסימלי.
- **תחביר ה-JSON המלא** של Lifecycle Configuration.
- **כל האופציות** של replication rules ו-S3 Replication Time Control לעומק.
- **המבנה המדויק** של דוח S3 Inventory ושל קובץ Storage Class Analysis.
- **רשימת כל 18+ היעדים** של EventBridge — מספיק להכיר Step Functions, Kinesis, Lambda, SQS, SNS.
- **שמות כל מדדי Storage Lens** — מספיק להכיר את הקטגוריות ואת ההבדל Free/Advanced.
- **האלגוריתם הפנימי** של Transfer Acceleration ובחירת ה-Edge Location.
- **תמחור מדויק** של transitions ושל Batch Operations — צריך להבין את **המבנה**, לא את המספר.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **Lifecycle יורד במדרג בלבד.** להחזיר מ-Glacier = **Restore + Copy**, לא lifecycle.
- **מינימום 30 יום ב-Standard** לפני מעבר ל-Standard-IA / One Zone-IA.
- **אובייקטים מתחת ל-128 KB לא מועברים** ל-IA classes.
- **Lifecycle עושה שני דברים:** Transition (העברה) ו-Expiration (מחיקה).
- **Expiration חל גם על:** noncurrent versions · **incomplete multipart uploads** · delete markers מיותמים.
- **סינון כללי lifecycle** לפי **prefix** או **object tags**.
- **Storage Class Analysis:** רק Standard ו-Standard-IA · **לא** One Zone-IA ולא Glacier ·
  דוח יומי · **24–48 שעות** עד נתונים.
- **Requester Pays:** המבקש משלם על request ו-download · **חייב להיות מאומת** (לא אנונימי).
- **Event Notifications:** ObjectCreated / ObjectRemoved / ObjectRestore / Replication ·
  סינון לפי שם · שניות עד דקה · **אין exactly-once ואין סדר** → צרכן **idempotent**.
- **הרשאות אירועים = Resource Policy** על SNS/SQS/Lambda. **לא** IAM Role.
- **EventBridge:** סינון JSON מתקדם · **18+ יעדים** · **Archive & Replay** · מסירה אמינה.
- **Baseline Performance: 3,500 PUT/COPY/POST/DELETE ו-5,500 GET/HEAD — לכל prefix.**
  **אין הגבלה על מספר prefixes.** 4 prefixes = **22,000 GET/s**.
- **Latency טיפוסי: 100–200 ms.**
- **Multi-Part Upload:** מומלץ **>100 MB**, **חובה >5 GB**. מקביליות + retry חלקי.
- **Transfer Acceleration:** Edge Location + רשת AWS פרטית. **תואם ל-Multi-Part**. עולה כסף.
- **Byte-Range Fetches:** הורדה מקבילית או משיכת ה-header בלבד. גם עמידות לכשלים.
- **Batch Operations:** metadata · copy · **הצפנת אובייקטים קיימים** · ACL/tags ·
  restore מ-Glacier · Lambda מותאמת. מנהל retries, התקדמות ודוחות.
- **הצמד הקלאסי: S3 Inventory → Athena → S3 Batch Operations.**
- **Storage Lens:** רמת Organization · 30 יום פעילות · Default Dashboard **לא נמחק, רק מושבת** ·
  ייצוא CSV/Parquet.
- **Storage Lens Free = ~28 מדדים, 14 יום. Advanced = בתשלום, 15 חודשים + prefix aggregation + CloudWatch.**

---

## 11. ✅ בדיקת הבנה

1. כלל lifecycle אומר "העבר ל-Standard-IA אחרי 10 ימים". למה שום דבר לא קורה?
2. מיליוני קבצי לוג של 8 KB לא עוברים ל-IA למרות הכלל. למה?
3. רוצים שאובייקטים יחזרו אוטומטית מ-Glacier ל-Standard כשניגשים אליהם. מה הפתרון הנכון?
4. יש חיוב אחסון שלא מסתדר עם רשימת האובייקטים ב-bucket. מה כנראה קורה?
5. הגעתם ל-5,500 GET בשנייה. מה עושים, ומה **לא** עושים?
6. איזה שילוב שירותים מצפין 200 מיליון אובייקטים קיימים שלא הוצפנו?
7. אירוע העלאה נשלח ל-Lambda שמעלה מונה. למה זה באג?
8. מה בדיוק מגדירים כדי ש-S3 תוכל לשלוח אירוע ל-SQS?
9. צריך לשמור אובייקטים 30 יום עם שחזור מיידי, ואז עד שנה עם שחזור תוך 48 שעות. מה בונים?
10. מה ההבדל בין Storage Lens Free ל-Advanced בשורה אחת?

<details>
<summary>תשובות</summary>

1. יש **מינימום של 30 יום ב-Standard** לפני מעבר ל-Standard-IA או One Zone-IA.
   כלל שמבקש 10 ימים לא יתבצע. צריך לשנות ל-30 ומעלה.
2. **S3 לא מעבירה אובייקטים קטנים מ-128 KB** ל-IA classes.
   ממילא הם היו מחויבים כ-128 KB, ולכן אין חיסכון. הפתרון: לאחד קבצים קטנים לפני האחסון.
3. **Lifecycle לא יכול לעלות במדרג.** התשובה היא **S3 Intelligent-Tiering** —
   הוא מזיז אובייקטים בין tiers אוטומטית לפי גישה בפועל, **בשני הכיוונים**, ובלי retrieval fees.
4. **Multi-Part Uploads שלא הושלמו.** החלקים מאוחסנים ומחויבים, אבל **לא מופיעים** ב-ListObjects.
   הפתרון: כלל lifecycle שמבטל אותם אחרי 7 ימים. אפשר לאתר עם
   `IncompleteMultipartUploadStorageBytes` ב-Storage Lens.
5. **עושים:** מפזרים את המפתחות על **prefixes נוספים** — המגבלה היא לכל prefix,
   ואין הגבלה על מספר ה-prefixes. 4 prefixes = 22,000 GET/s.
   **לא עושים:** לא יוצרים bucket שני. זה לא פותר כלום כי המגבלה אינה ברמת ה-bucket.
6. **S3 Inventory** מייצר רשימה → **Athena** מסנן את הלא-מוצפנים →
   **S3 Batch Operations** מריץ Copy עם הצפנה על הרשימה.
7. כי S3 Event Notifications **לא מבטיחות exactly-once ולא מבטיחות סדר** —
   אירוע יכול להימסר **יותר מפעם אחת**. מונה שעולה בכל הפעלה יספור יותר מדי.
   הצרכן חייב להיות **idempotent** (למשל לפי `bucket + key + versionId`).
8. **SQS Resource (Access) Policy** שמתירה ל-service principal של S3 לשלוח לתור.
   זו resource policy על ה-**תור**, לא IAM Role על ה-bucket.
   אותו עיקרון ל-SNS Access Policy ול-Lambda Resource Policy.
9. **Versioning** (כדי ש"מחיקה" תהיה delete marker מעל גרסה ששרדה) +
   **Lifecycle על noncurrent versions**: ל-Standard-IA מיד, ואז ל-**Glacier Deep Archive**
   (Bulk = 48 שעות, בדיוק הדרישה), ובסוף Expiration אחרי 365 יום.
10. **Free:** ~28 מדדי usage, נתונים ל-**14 יום**, ללא עלות.
    **Advanced:** בתשלום, מוסיף Activity / Cost Optimization / Data Protection / Status Code,
    **15 חודשים** של נתונים, **prefix aggregation**, ופרסום ל-CloudWatch ללא חיוב נוסף.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[16 - S3 Fundamentals]] · [[17 - S3 Security and Data Management]] · [[28 - SQS and SNS]] · [[29 - Event-Driven Architecture]] · [[25 - Lambda]] · [[31 - Monitoring and Logging]] · [[37 - Cost Optimization]] · [[35 - Backup and Data Protection]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 4894–4929, 5157–5534
