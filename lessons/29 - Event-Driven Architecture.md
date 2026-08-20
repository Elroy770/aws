---
lesson: 29
title: Event-Driven Architecture
domain: Design Resilient Architectures
services: [EventBridge, Kinesis Data Streams, Amazon Data Firehose, Amazon MQ, SQS, SNS, Lambda, CloudTrail, S3]
tags: [saa-c03, integration, eventbridge, kinesis, streaming, event-driven]
---

# 29 — Event-Driven Architecture

> [!abstract] בשורה אחת
> ארכיטקטורה מונחית-אירועים היא בחירה בין חמישה כלים — **SQS, SNS, EventBridge, Kinesis, Amazon MQ** —
> וכל שאלה במבחן נפתרת בזיהוי שלוש תכונות: **האם צריך replay, האם צריך סדר, והאם יש כמה צרכנים עצמאיים.**

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה event הוא לא command, ומה זה קונה |
| 2 | איך זה עובד | EventBridge, Kinesis Data Streams, Data Firehose, Amazon MQ |
| 3 | פירוק מפורט | Capacity Modes, Buses, Rules, Archive/Replay, Schema Registry |
| 4 | עלות | shard-hour מול on-demand, event-based, delivery-based |
| 5 | השוואות | **טבלת ההחלטה הגדולה** של חמשת השירותים + KDS מול Firehose |
| 6 | Well-Architected | EDA לפי ששת ה-Pillars |
| 7 | מלכודות | "real-time" בשאלה לא תמיד אומר Kinesis |
| 8 | Scenario | פלטפורמת telemetry + אירועים עסקיים + ניטור אבטחה |

**מונחי מפתח בשיעור:** `Event Bus` · `Event Pattern` · `Shard` · `Partition Key` · `Replay` · `Archive` · `Enhanced Fan-Out` · `Near Real-Time`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- בקריאה סינכרונית, המפיק **חייב להכיר** את כל מי שצריך את המידע — ולעדכן קוד כשמצטרף עוד אחד.
- כשל של צרכן אחד **מפיל את הפעולה כולה**, גם אם הוא לא קריטי.
- אין דרך "לנגן מחדש" את מה שקרה. אם הצרכן היה מושבת שעה — השעה ההיא אבודה.
- מיליוני אירועי telemetry בשנייה לא נכנסים למודל של תור פשוט.
- כשמעבירים אפליקציה ותיקה לענן, היא כבר מדברת **MQTT/AMQP/STOMP** ולא SQS API.

### מה השירות פותר

- **Event = עובדה שקרתה**, לא פקודה למישהו. `OrderPlaced` ולא `ChargeCreditCard`.
- המפיק מפרסם ולא יודע מי מקשיב. **הוספת צרכן היא הגדרה, לא deploy.**
- **Routing לפי תוכן** — EventBridge מנתב לפי pattern במקום שהמפיק יחליט.
- **Replay** — Kinesis ו-EventBridge Archive מאפשרים לעבד מחדש היסטוריה.
- **Streaming בקנה מידה** — Kinesis בנוי למיליוני רשומות בשנייה, לא לתור.
- **תאימות לפרוטוקולים פתוחים** — Amazon MQ מאפשר lift-and-shift בלי לשכתב את האפליקציה.

> [!tip] האנלוגיה
> **SQS = רשימת מטלות.** מי שלוקח מטלה, המטלה נעלמת.
> **SNS = הודעה ברמקול.** נאמרת פעם אחת, מי שנכח שמע.
> **EventBridge = מרכזייה חכמה.** מקבלת הכול ומחליטה לפי תוכן ההודעה למי להעביר.
> **Kinesis = טייפ שמקליט הכול.** אפשר לחזור אחורה ולהאזין שוב, גם בעוד חודש.

---

## 2. ⚙️ איך זה עובד

### 2.1 Amazon EventBridge — ה-Event Bus של AWS

היה בעבר **CloudWatch Events**, וזה עדיין השם שמופיע בשאלות ישנות.

**שני מצבי הפעלה:**

| מצב | מה זה | דוגמה |
|---|---|---|
| **Schedule / Cron** | הפעלה מתוזמנת | כל שעה → הפעלת Lambda שמנקה נתונים |
| **Event Pattern** | תגובה לאירוע שקרה | Root User Sign-in → SNS Topic עם מייל למנהל |

**המקורות (Sources):**

- שירותי AWS — EC2 state change, CodeBuild failed build, S3 upload, Trusted Advisor finding.
- **CloudTrail — כל קריאת API בחשבון.** זה מה שהופך את EventBridge לכלי אבטחה.
- אפליקציות מותאמות אישית (PutEvents).
- שותפי SaaS — Zendesk, Datadog, Auth0 ודומיהם.
- Schedule / Cron.

**היעדים (Targets) — מעל 18 שירותים:**

```text
                          ┌──► Lambda           ┌──► Step Functions
AWS Services ──┐          ├──► SQS              ├──► CodePipeline
CloudTrail   ──┤          ├──► SNS              ├──► CodeBuild
SaaS Partners──┼──► EventBridge Rule ──►        ├──► SSM
Custom Apps  ──┤   (Event Pattern JSON)         ├──► EC2 Actions
Schedule     ──┘          ├──► Kinesis Data Streams
                          └──► ECS Task / AWS Batch
```

- **Rule** = תבנית JSON שמסננת אילו אירועים רלוונטיים + רשימת יעדים.
- הסינון הוא **על תוכן האירוע** — לא רק על סוג המקור.
- ניתן גם **לשנות את מבנה האירוע** (Input Transformer) לפני שהוא נשלח ליעד.

### 2.2 שלושת סוגי ה-Event Buses

| Bus | מי כותב אליו | מתי משתמשים |
|---|---|---|
| **Default Event Bus** | שירותי AWS בחשבון | קיים אוטומטית, כל אירועי ה-AWS |
| **Partner Event Bus** | שותפי SaaS | לקבל אירועים מ-Zendesk / Datadog / Auth0 |
| **Custom Event Bus** | האפליקציות שלכם | אירועים עסקיים (`OrderPlaced`, `UserSignedUp`) |

**Resource-based Policy על ה-bus:**

- מנהלת הרשאות **ל-bus ספציפי**.
- מאפשרת/חוסמת אירועים **מחשבון AWS אחר או מ-Region אחר**.
- **Use case מרכזי:** לרכז את כל האירועים של כל ה-Organization ב**חשבון מרכזי אחד**.

```text
Account 111122223333 ──PutEvents──► central-event-bus (Account 123456789012) ──► Lambda
                                     ↑ Resource-based Policy מתירה לחשבון החיצוני לכתוב
```

### 2.3 Archive ו-Replay — היכולת שרק ל-EventBridge יש מבין ה-routers

- ניתן **לארכב** אירועים שנשלחו ל-bus — הכול או לפי סינון.
- תקופת הארכוב: **לצמיתות או לפרק זמן מוגדר**.
- ניתן **לנגן מחדש (Replay)** אירועים מהארכיון.
- **Use case:** באג בצרכן שגרם לעיבוד שגוי של יומיים — מתקנים, ואז replay של אותם יומיים.

> [!info] זו נקודת ההבדל מ-SNS
> ל-SNS **אין ארכיון ואין replay**. אירוע שלא נמסר — אבד.
> אם בשאלה מופיעות המילים "replay past events" — התשובה היא **EventBridge Archive** או **Kinesis**.

### 2.4 Schema Registry

- EventBridge **מנתח את האירועים** שעוברים ב-bus ומסיק מהם את ה-schema אוטומטית.
- מאפשר **לייצר קוד** (code bindings) שהאפליקציה משתמשת בו ויודעת מראש את מבנה הנתונים.
- **ה-schema הוא versioned** — אפשר לעקוב אחרי שינויים במבנה האירוע לאורך זמן.
- **Use case:** צוותים שונים שצורכים אירועים של צוות אחר, בלי תיעוד ידני שמתיישן.

### 2.5 EventBridge כשומר סף — Intercept API Calls

זהו אחד ה-use cases שנשאלים הכי הרבה, כי הוא מחבר בין integration לאבטחה:

```text
User ──DeleteTable API──► DynamoDB
              │
              ▼
        CloudTrail (מתעד כל קריאת API)
              │
              ▼
        EventBridge Rule (pattern: eventName = DeleteTable)
              │
              ▼
        SNS Topic ──► התראה למנהל
```

**דוגמאות נוספות מאותו דפוס:**

| קריאת API שמיירטים | למה זה מעניין |
|---|---|
| `DeleteTable` | מחיקת טבלת DynamoDB — פעולה הרסנית |
| `AuthorizeSecurityGroupIngress` | מישהו פתח פורט ב-Security Group |
| `AssumeRole` | שימוש ב-IAM Role — מעקב אחרי גישה מורשית |
| `ConsoleLogin` של Root | כניסת root — אירוע שאמור להיות נדיר מאוד |

- ההתראה יכולה להיות SNS, אבל גם **Lambda שמתקן את המצב אוטומטית** (auto-remediation).
- ראו גם [[31 - Monitoring and Logging]] ו-[[32 - Security Services]].

### 2.6 Kinesis Data Streams — הקלטה של זרם

```text
Producers                    Kinesis Data Streams              Consumers
─────────                    ────────────────────              ─────────
Applications ──┐             ┌─ Shard 1 ─┐                  ┌──► Lambda
Click Streams ─┤             ├─ Shard 2 ─┤                  ├──► Managed Service
IoT Devices ───┼──records──► ├─ Shard 3 ─┤ ── retention ──► ├──►   for Apache Flink
Metrics & Logs ┤             └─ Shard N ─┘   עד 365 יום     ├──► Amazon Data Firehose
Kinesis Agent ─┘                                            └──► Custom App (KCL)
```

**המאפיינים שקובעים מתי בוחרים ב-KDS:**

| מאפיין | ערך | למה זה חשוב |
|---|---|---|
| **Retention** | **עד 365 ימים** | הנתון נשאר גם אחרי שנצרך |
| **Replay** | **כן** | צרכן חדש יכול לקרוא היסטוריה מההתחלה |
| **מחיקה** | **בלתי אפשרית** — עד לפקיעה | בניגוד ל-SQS שבו הצרכן מוחק |
| **סדר** | מובטח לכל **Partition Key** | כל המפתחות הזהים נוחתים באותו shard |
| **גודל רשומה** | **עד 10 MiB** (ראו הערה) | ה-use case הטיפוסי הוא הרבה רשומות **קטנות** |
| **הצפנה** | KMS במנוחה, HTTPS בתעבורה | |
| **ספריות** | **KPL** לכתיבה יעילה, **KCL** לקריאה יעילה | |

> [!warning] הערת דיוק — גודל הרשומה
> המגבלה ההיסטורית והמוכרת של KDS הייתה **1 MB לרשומה**, ורוב שאלות המבחן הוותיקות מניחות אותה.
> חומר הקורס העדכני נוקב ב-**10 MiB** בעקבות הרחבת המגבלה על ידי AWS.
> **מה שבאמת נבדק:** ש-KDS מיועד ל**המון רשומות קטנות בזמן אמת**, ולא ל-payloads כבדים.

### 2.7 Amazon Data Firehose — טעינה ליעד, לא הקלטה

```text
Producers ──► Amazon Data Firehose ──[buffer by size/time]──► Destination
   │                    │
   │                    ├─ Lambda transformation (למשל CSV → JSON)
   │                    ├─ המרה ל-Parquet / ORC
   │                    └─ דחיסה ב-gzip / snappy
   │
   └─ SDK · Kinesis Agent · Kinesis Data Streams · CloudWatch Logs & Events · AWS IoT
```

| מאפיין | Firehose |
|---|---|
| ניהול | **מנוהל לחלוטין, serverless, auto-scaling** — אין shards |
| זמן | **Near real-time** — buffering לפי **גודל או זמן** |
| **אחסון** | **אין.** לא שומר את הנתון |
| **Replay** | **לא נתמך** |
| יעדי AWS | **S3 · Redshift · OpenSearch** |
| יעדי צד שלישי | Splunk · MongoDB · Datadog · New Relic |
| יעד מותאם | **Custom HTTP Endpoint** |
| פורמטים | CSV, JSON, Parquet, Avro, Raw Text, Binary |
| טרנספורמציה | **Lambda** לשינוי כל רשומה |
| גיבוי | **S3 backup bucket** — לכל הנתונים או רק לאלה שנכשלו |

- **Firehose לא מחליף את KDS** — הוא לרוב הצרכן **שאחריו**: KDS קולט בזמן אמת, Firehose טוען ל-S3.

### 2.8 Amazon MQ — broker לפרוטוקולים פתוחים

- SQS ו-SNS הם שירותים **cloud-native** עם פרוטוקולים **קנייניים של AWS**.
- אפליקציות ותיקות מ-on-premises מדברות **MQTT, AMQP, STOMP, Openwire, WSS**.
- במקום לשכתב את האפליקציה כדי שתשתמש ב-SQS API — מרימים **Amazon MQ**.

| מאפיין | Amazon MQ |
|---|---|
| מה זה | **Managed Message Broker** (ActiveMQ / RabbitMQ) |
| יכולות | **גם queue (כמו SQS) וגם topic (כמו SNS)** |
| ארכיטקטורה | **רץ על שרתים** — לא serverless |
| התרחבות | **לא מתרחב כמו SQS/SNS.** מוגבל בגודל ה-broker |
| **זמינות גבוהה** | **Multi-AZ עם failover:** Active ב-AZ אחת, Standby ב-AZ שנייה |
| אחסון משותף | **Amazon EFS** — מה שמאפשר ל-standby להשתלט על אותם נתונים |

```text
Region us-east-1
├── AZ us-east-1a:  Amazon MQ Broker  [ACTIVE]  ◄── Client
│                          │                          │ failover
└── AZ us-east-1b:  Amazon MQ Broker  [STANDBY] ◄─────┘
                    └──────┬──────┘
                      Amazon EFS (אחסון משותף)
```

> [!info] מתי Amazon MQ הוא התשובה
> **רק** כשהשאלה מזכירה במפורש **פרוטוקול פתוח** (MQTT/AMQP/STOMP/JMS/Openwire)
> או **מיגרציה של אפליקציה קיימת בלי שינוי קוד**.
> בכל מקרה אחר — SQS/SNS זולים יותר, מתרחבים יותר, ולא דורשים ניהול.

---

## 3. 🔍 פירוק מפורט

### 3.1 Kinesis Data Streams — שני מצבי הקיבולת

| קריטריון | **Provisioned Mode** | **On-Demand Mode** |
|---|---|---|
| ניהול shards | **אתם בוחרים** את המספר | **אין מה לנהל** |
| קיבולת כניסה | **1 MB/s או 1,000 רשומות/שנייה לכל shard** | ברירת מחדל **4 MB/s או 4,000 רשומות/שנייה** |
| קיבולת יציאה | **2 MB/s לכל shard** | מתרחב אוטומטית |
| התרחבות | **ידנית** — הוספה/הסרה של shards | **אוטומטית**, לפי **הפיק של 30 הימים האחרונים** |
| חיוב | **לכל shard לשעה** | **לכל stream לשעה + GB נכנס/יוצא** |
| מתי מתאים | עומס **צפוי ויציב** — הכי זול | עומס **בלתי צפוי** או חדש |

**Enhanced Fan-Out:**

- במצב הרגיל (Standard) הצרכנים **מושכים** ומתחלקים ב-**2 MB/s לכל shard**.
- ב-**Enhanced Fan-Out**, Kinesis **דוחף** לכל צרכן **2 MB/s משלו לכל shard**.
- **מתי:** כמה צרכנים עצמאיים על אותו stream שמתחילים להתחרות על רוחב הפס.

### 3.2 Partition Key — ההחלטה שקובעת את הביצועים

- ה-Partition Key ממופה ל-**shard** באמצעות hash.
- **כל הרשומות עם אותו Partition Key נוחתות באותו shard** — ולכן הסדר ביניהן מובטח.
- **הסדר מובטח ברמת ה-shard בלבד**, לא ברמת ה-stream כולו.

| בחירת Partition Key | תוצאה |
|---|---|
| `device-id` (מיליוני מכשירים) | פיזור טוב, מקביליות מלאה ✅ |
| `country` (5 ערכים אפשריים) | **hot shard** — רוב התעבורה על shard אחד ❌ |
| ערך קבוע אחד | כל ה-stream על shard יחיד — throttling מיידי ❌ |

> [!warning] הכלל למבחן
> **סדר ב-Kinesis = ברמת ה-shard, לפי Partition Key.**
> אם בשאלה כתוב "records for the same device must be processed in order" — זהו Partition Key = device id.

### 3.3 דפוסי Lambda + SNS + SQS

| דפוס | התנהגות | טיפול בכישלון |
|---|---|---|
| **SQS + Lambda** | Lambda **מושך (poll)** מהתור | ניסיונות חוזרים; אחרי `maxReceiveCount` → **DLQ של התור** |
| **SNS + Lambda** | SNS **דוחף** ל-Lambda, **אסינכרוני** | Lambda מנסה שוב אוטומטית ואז → **DLQ של ה-Lambda** |
| **SQS FIFO + Lambda** | עיבוד לפי סדר | הודעה שנכשלת **חוסמת** את ה-Message Group שלה עד DLQ |
| **EventBridge + Lambda** | EventBridge דוחף לפי rule | retry לפי מדיניות + **DLQ ליעד** |

> [!warning] הנקודה הקריטית ב-FIFO + Lambda
> ב-FIFO, הודעה תקועה **חוסמת** את כל שאר ההודעות באותו Message Group.
> ב-Standard, היא רק נכשלת לבדה. זו הסיבה ש-DLQ ב-FIFO הוא חובה ולא המלצה.

### 3.4 Fan Out — שתי הדרכים, ולמה אחת נכונה

```text
Option 1 — המפיק כותב לכל תור בעצמו
App ──SDK PUT #1──► SQS Queue A
    ──SDK PUT #2──► SQS Queue B
    ──SDK PUT #3──► SQS Queue C
    (3 קריאות; כשל באמצע = חוסר עקביות; הוספת תור = שינוי קוד)

Option 2 — Fan Out
App ──PUT──► SNS Topic ──subscribe──┬──► SQS Queue A
                                    ├──► SQS Queue B
                                    └──► SQS Queue C
    (קריאה אחת; הוספת תור = הגדרה בלבד)
```

פירוט מלא של הדפוס ב-[[28 - SQS and SNS]].

### 3.5 S3 Event Notifications — ושדרוג ל-EventBridge

**הדרך הקלאסית:**

| מאפיין | S3 Event Notifications |
|---|---|
| סוגי אירועים | `S3:ObjectCreated` · `S3:ObjectRemoved` · `S3:ObjectRestore` · `S3:Replication` |
| סינון | **לפי שם אובייקט בלבד** (למשל `*.jpg`) |
| יעדים | **SNS · SQS · Lambda** |
| כמות | ניתן ליצור כמה אירועים שרוצים |
| **Latency** | בדרך כלל **שניות**, אבל **לפעמים דקה ויותר** |
| Use case | יצירת thumbnails לתמונות שהועלו |

**השדרוג — S3 → EventBridge:**

```text
S3 Bucket ──all events──► EventBridge ──rules──► מעל 18 שירותי AWS כיעד
```

| יתרון | פירוט |
|---|---|
| **סינון מתקדם** | כללי JSON על **metadata, גודל אובייקט, שם** — לא רק על prefix/suffix |
| **ריבוי יעדים** | Step Functions, Kinesis Data Streams, Firehose ועוד |
| **Archive ו-Replay** | אפשר לנגן מחדש אירועי S3 |
| **מסירה אמינה** | מנגנון retry ו-DLQ ברמת ה-rule |

> [!tip] מילות מפתח
> "filter S3 events by object size" או "replay S3 events" → **EventBridge**, לא S3 Event Notifications.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| שירות | רכיב חיוב | הערה |
|---|---|---|
| **EventBridge** | לכל מיליון **אירועים שנשלחו ל-bus** | **אירועי AWS ב-Default Bus הם ללא עלות** |
| EventBridge Archive/Replay | GB מאוחסן + אירועים ששוחזרו | הארכיון עולה רק אם מפעילים אותו |
| **KDS Provisioned** | **shard-hour** + PUT payload units | חיוב גם כשה-stream ריק |
| **KDS On-Demand** | **stream-hour** + GB נכנס + GB יוצא | נוח, אך יקר יותר בעומס יציב |
| KDS Extended Retention | תוספת לכל shard-hour מעל 24 שעות | 7 ימים ו-365 ימים בתעריפים עולים |
| KDS Enhanced Fan-Out | לכל consumer-shard-hour + GB | מכפיל את החשבון במספר הצרכנים |
| **Firehose** | **GB שנטענו** (ותוספת להמרות פורמט) | **אין חיוב על קיבולת מסופקת** |
| **Amazon MQ** | **broker-instance-hour + אחסון** | משלמים גם ב-0 הודעות; Multi-AZ = פי 2 |
| **היעדים** | Lambda / SQS / Step Functions מחויבים **בנפרד** | טעות חישוב נפוצה |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **EventBridge על Default Bus** | **0 לאירועי AWS** | ניטור, cron, תגובה לשינויי מצב |
| **SQS + Lambda** | נמוכה | תור פשוט בלי replay ובלי סדר גלובלי |
| **Firehose ל-S3** | נמוכה — משלמים על GB בלבד | טעינת לוגים ו-analytics ל-data lake |
| **KDS Provisioned** | בינונית, אך **קבועה 24/7** | עומס יציב וידוע |
| **KDS On-Demand** | גבוהה יותר בעומס יציב | עומס חדש או בלתי צפוי |
| **KDS + Enhanced Fan-Out** | גבוהה | כמה צרכנים שצריכים כל אחד את מלוא רוחב הפס |
| **Amazon MQ Multi-AZ** | **הגבוהה ביותר יחסית לתפוקה** | רק כשחייבים פרוטוקול פתוח |

### 🚩 עלויות נסתרות

- **shards שנשארו אחרי הפיק** — Provisioned Mode לא מקטין את עצמו. שדרגתם ל-Black Friday ושכחתם.
- **Extended Retention** — מעבר מ-24 שעות ל-7 ימים או 365 ימים מייקר כל shard-hour.
- **Enhanced Fan-Out מוכפל בצרכנים** — כל צרכן נוסף מוסיף עלות לכל shard.
- **Amazon MQ פועל 24/7** — גם broker בלי תעבורה מחויב במלואו, ו-Multi-AZ מכפיל.
- **היעדים של EventBridge** — כלל שמפעיל Lambda על כל אירוע S3 בדלי עמוס הוא חשבון Lambda, לא EventBridge.
- **Cross-Region delivery** — data transfer בין Regions מתווסף.
- **Replay** — ניגון מחדש של ארכיון גדול מפעיל את כל היעדים שוב, על כל העלות שלהם.

### 💡 טיפים לחיסכון

- להשתמש ב-**Default Event Bus** לאירועי AWS — אין חיוב על האירועים עצמם.
- **סינון ב-Rule** ולא בקוד היעד — לא לשלם על Lambda invocation שמסתיים ב-`return`.
- **Firehose במקום KDS** כשהיעד הוא רק S3/Redshift/OpenSearch ואין צורך ב-replay.
- **Provisioned Mode** לעומס יציב; **On-Demand** רק בהתחלה או לעומס בלתי צפוי.
- **retention של 24 שעות** אלא אם באמת צריך יותר.
- **Enhanced Fan-Out רק לצרכנים שנחנקים** — ולא כברירת מחדל.
- **Amazon MQ רק כשאין ברירה** — SQS/SNS זולים ומתרחבים בהרבה.
- **המרה ל-Parquet ודחיסה ב-Firehose** — מקטינה משמעותית את חשבון ה-S3 וה-Athena במורד הזרם.

---

## 5. ⚖️ השוואות מכריעות

### טבלת ההחלטה הגדולה — חמשת שירותי ה-Integration

| קריטריון | **SQS** | **SNS** | **EventBridge** | **Kinesis Data Streams** | **Amazon MQ** |
|---|---|---|---|---|---|
| **דגם** | Queue | Pub/Sub | **Event Bus + Router** | **Stream** | Broker (queue + topic) |
| **שמירת נתונים** | עד **14 ימים** | **לא נשמר** | לא (אלא אם Archive) | **עד 365 ימים** | עד ACK של הצרכן |
| **ריבוי צרכנים** | מתחלקים בעבודה | כל מנוי עותק (עד 12.5M) | כל rule מקבל עותק | **כולם קוראים הכול** | לפי queue/topic |
| **Replay** | ❌ | ❌ | ✅ דרך **Archive** | ✅ **מובנה** | ❌ |
| **סדר** | רק ב-FIFO | רק ב-FIFO Topic | ❌ אין ערובה | ✅ **ברמת shard** | ✅ ב-queue |
| **סינון לפי תוכן** | ❌ | Filter Policy בסיסי | ✅ **JSON patterns עשירים** | ❌ (בקוד הצרכן) | Selectors |
| **קנה מידה** | בלתי מוגבל | בלתי מוגבל | גבוה מאוד | לפי shards | **מוגבל לגודל ה-broker** |
| **תמחור** | לכל request (64KB) | publish + delivery | לכל אירוע ל-bus | **shard-hour** / stream-hour | **instance-hour 24/7** |
| **Use case** | decoupling, buffering | fan-out, התראות | routing עסקי, אוטומציה, SaaS | telemetry, IoT, analytics, ETL | מיגרציה מ-on-prem |
| **מילת מפתח במבחן** | "decouple", "buffer" | "notify many", "fan out" | **"rules", "route", "schedule", "any API call"** | **"real-time streaming", "replay", "shards"** | **"MQTT / AMQP / STOMP / JMS"** |

### Kinesis Data Streams מול Amazon Data Firehose

| קריטריון | **Kinesis Data Streams** | **Amazon Data Firehose** |
|---|---|---|
| מטרה | **איסוף ואחסון** של זרם | **טעינת** זרם ליעד |
| זמן | **Real-time** | **Near real-time** (buffer לפי גודל/זמן) |
| ניהול | shards — **provisioned או on-demand** | **מנוהל לחלוטין, auto-scaling** |
| **קוד** | צריך לכתוב **producer וגם consumer** | **אין קוד צרכן** |
| **אחסון** | **עד 365 ימים** | **אין אחסון** |
| **Replay** | ✅ | ❌ |
| יעדים | כל אפליקציה (KCL, Lambda, Flink, Firehose) | **S3, Redshift, OpenSearch, 3rd party, HTTP** |
| טרנספורמציה | בקוד הצרכן | **Lambda מובנה** + המרה ל-Parquet/ORC |
| תמחור | shard-hour / stream-hour | **GB שנטענו בלבד** |
| **מתי בוחרים** | צריך replay, עיבוד מותאם, כמה צרכנים | רק להעביר נתונים ליעד — **הכי פשוט וזול** |

> [!info] שורה תחתונה
> שאלו את עצמכם שלוש שאלות, לפי הסדר:
> **1. צריך replay או היסטוריה?** → Kinesis (או EventBridge Archive). אחרת לא Kinesis.
> **2. צריך לנתב לפי תוכן האירוע או לתזמן?** → EventBridge.
> **3. יש כמה צרכנים עצמאיים שכל אחד צריך הכול?** → SNS + SQS.
> ואם השאלה מזכירה **פרוטוקול פתוח** — התשובה היא **Amazon MQ**, בלי קשר לכל השאר.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-EDA** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | אפשר להבין מה קרה ולתקן בדיעבד | **Schema Registry** עם versioning; correlation ID בכל אירוע; **EventBridge Archive** ו-runbook ל-Replay |
| **Security** | לכל bus/stream הרשאה מינימלית | Resource-based Policy ל-bus חוצה חשבונות; KMS על streams; **EventBridge + CloudTrail** להתראה על API רגישות |
| **Reliability** | צרכן שנפל לא מאבד אירועים | **DLQ לכל rule ולכל תור**; צרכנים **idempotent**; Kinesis retention כרשת ביטחון; Amazon MQ ב-**Multi-AZ** |
| **Performance Efficiency** | לא נוצרים hot shards ולא throttling | **Partition Key בעל קרדינליות גבוהה**; Enhanced Fan-Out לצרכנים רעבים; batching |
| **Cost Optimization** | לא משלמים על קיבולת שלא בשימוש | **Firehose במקום KDS** כשאין replay; סינון ב-rule ולא ב-Lambda; retention קצר; להימנע מ-MQ מיותר |
| **Sustainability** | פחות עיבוד מיותר | סינון בשכבת ה-routing; **המרה ל-Parquet ודחיסה** ב-Firehose; batching במקום רשומה-רשומה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "run a script every hour" / "cron" | **EventBridge Schedule Rule** |
| "react to any API call in the account" | **EventBridge + CloudTrail** |
| "alert when the root user signs in" | **EventBridge Rule → SNS** |
| "aggregate events from all accounts in the org" | **Custom Event Bus + Resource-based Policy** |
| "replay events from last week" | **EventBridge Archive** או **Kinesis retention** |
| "know the structure of events in advance" | **EventBridge Schema Registry** |
| "millions of records per second, real-time" | **Kinesis Data Streams** |
| "load streaming data into S3 / Redshift / OpenSearch" | **Amazon Data Firehose** |
| "near real-time, fully managed, no code" | **Firehose** |
| "records from the same device in order" | **Kinesis Partition Key** |
| "convert to Parquet before storing in S3" | **Firehose transformation** |
| "MQTT / AMQP / STOMP / JMS / Openwire" | **Amazon MQ** |
| "migrate the message broker without rewriting the app" | **Amazon MQ** |
| "broker must survive an AZ failure" | **Amazon MQ Multi-AZ (Active/Standby + EFS)** |
| "filter S3 events by object size" | **S3 → EventBridge** |
| "one message, many independent systems" | **SNS + SQS Fan Out** |

### טעויות נפוצות

> [!warning] מלכודת 1 — "real-time" ⇐ Kinesis
> **הניסוח:** "Route order events in real time to the billing and shipping services."
> **הטעות:** לבחור Kinesis כי כתוב real-time.
> **הנכון:** זה **routing עסקי**, לא stream analytics. **EventBridge או SNS+SQS.**
> Kinesis הוא לזרמים בנפח עצום עם replay — לא לניתוב אירועים בודדים.

> [!warning] מלכודת 2 — EventBridge כתור
> **הניסוח:** "The Lambda target was throttled and events were lost."
> **הטעות:** לחפש retention ב-EventBridge.
> **הנכון:** **EventBridge הוא router, לא תור עמיד.** שמים **SQS כיעד של ה-rule**,
> וה-Lambda צורך מהתור — כך מקבלים buffering, retries ו-DLQ.

> [!warning] מלכודת 3 — סדר גלובלי ב-Kinesis
> **הניסוח:** "All records in the stream will be processed in the exact order they were sent."
> **הטעות:** להניח שסדר ב-Kinesis הוא גלובלי.
> **הנכון:** **הסדר מובטח בתוך shard בלבד**, כלומר לכל Partition Key. בין shards — אין ערובה.

> [!warning] מלכודת 4 — Firehose עם replay
> **הניסוח:** "Use Firehose so we can reprocess the last 3 days of data if the pipeline fails."
> **הטעות:** להניח ש-Firehose מאחסן.
> **הנכון:** **Firehose לא שומר נתונים ולא תומך ב-replay.**
> ל-replay צריך **KDS** (עד 365 ימים) — או לקרוא מחדש את מה ש-Firehose כתב ל-S3.

> [!warning] מלכודת 5 — Amazon MQ כברירת מחדל
> **הניסוח:** "We need a managed message broker for a new cloud-native application."
> **הטעות:** לבחור Amazon MQ כי נשמע "מקצועי".
> **הנכון:** לאפליקציה **חדשה** בוחרים **SQS/SNS** — serverless, מתרחבים בלי גבול, זולים.
> Amazon MQ הוא **רק** למיגרציה של אפליקציה שכבר מדברת פרוטוקול פתוח.

> [!warning] מלכודת 6 — Partition Key עם קרדינליות נמוכה
> **הניסוח:** "Use the country code as the partition key for 50M IoT events per hour."
> **הטעות:** לחשוב שזה סביר כי יש הרבה מדינות.
> **הנכון:** מספר ערכים קטן = **hot shard**. רוב התעבורה תיפול על shard אחד ותיתקל ב-throttling
> בזמן ששאר ה-shards ריקים. בוחרים מפתח בעל קרדינליות גבוהה כמו `device-id`.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

פלטפורמת IoT לניהול צי רכבים.
מיליון מכשירים שולחים telemetry כל 10 שניות — צריך לאחסן הכול ל-analytics ולזהות חריגות בזמן אמת.
במקביל, המערכת העסקית מייצרת אירועים (`TripStarted`, `MaintenanceDue`) שכמה צוותים צריכים.
צוות האבטחה דורש התראה על כל שינוי ב-Security Group בחשבון.
דרישה נוספת: מערכת ה-billing הישנה שהועברה מ-on-premises מדברת **AMQP**.
בנוסף — אם יימצא באג בעיבוד, צריך יכולת לעבד מחדש את הנתונים של 3 הימים האחרונים.

**הארכיטקטורה:**

```text
                                          ┌──► Lambda (זיהוי חריגות בזמן אמת) ──► SNS
1M IoT Devices ──► Kinesis Data Streams ──┤
                   Partition Key=device-id├──► Amazon Data Firehose ──► S3 (Parquet)
                   retention = 7 days     │                              │
                   Provisioned shards     └──► (Replay זמין 7 ימים)      └──► Athena / QuickSight

                                          ┌──► SQS ──► Billing Workers
Business App ──PutEvents──► EventBridge ──┤
                (Custom Bus)              ├──► SQS ──► Notification Lambda
                Archive = 30 days         └──► Step Functions (תהליך תחזוקה)

CloudTrail ──► EventBridge Rule ──► SNS ──► צוות אבטחה
              (AuthorizeSecurityGroupIngress)

Legacy Billing App (AMQP) ◄──► Amazon MQ (Multi-AZ Active/Standby)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **Kinesis Data Streams** ל-telemetry | מיליוני רשומות קטנות בשנייה; **כמה צרכנים קוראים את אותו זרם** |
| **Partition Key = `device-id`** | קרדינליות עצומה = פיזור מושלם; והסדר לכל מכשיר מובטח |
| **Retention = 7 ימים** | עונה בדיוק על דרישת ה-replay של 3 ימים, עם מרווח |
| **Provisioned Mode** | הקצב ידוע ויציב (מיליון מכשירים כל 10 שניות) — הזול ביותר |
| **Firehose כצרכן** של ה-stream | טעינה ל-S3 בלי קוד צרכן, עם המרה ל-**Parquet** ודחיסה |
| **Lambda כצרכן שני** במקביל | זיהוי חריגות בזמן אמת בלי להפריע לצינור האחסון |
| **EventBridge Custom Bus** לאירועים עסקיים | ניתוב לפי תוכן; הוספת צוות רביעי = rule חדש, בלי deploy |
| **SQS בין ה-rule ל-Lambda** | EventBridge אינו תור — התור נותן buffering, retry ו-DLQ |
| **EventBridge Archive = 30 ימים** | replay של אירועים עסקיים אחרי תיקון באג |
| **EventBridge + CloudTrail** לאבטחה | כל `AuthorizeSecurityGroupIngress` מייצר אירוע → SNS |
| **Amazon MQ Multi-AZ** ל-billing הישן | האפליקציה מדברת AMQP; שכתוב ל-SQS יקר ומסוכן |
| **Athena על S3** ל-analytics | שאילתות ad-hoc על Parquet, בלי אשכול שרץ 24/7 |

**למה לא SQS ל-telemetry?**
כי ב-SQS כל הודעה נצרכת פעם אחת ונמחקת — **אין replay ואין כמה צרכנים שקוראים הכול**.
בנוסף, החיוב לפי request על מיליוני רשומות זעירות אינו כלכלי.

**למה לא Firehose ישירות מהמכשירים, בלי KDS?**
כי אז אין **replay** ואין אפשרות לצרכן שני (זיהוי החריגות) לקרוא את אותו זרם.
Firehose הוא צינור חד-כיווני ליעד, בלי אחסון.

**למה לא EventBridge ל-telemetry?**
EventBridge מתומחר **לכל אירוע**. מיליוני אירועים בשנייה הופכים אותו ליקר מאוד,
והוא גם לא נותן replay מובנה ולא ordering.

**למה לא Amazon MQ לכל המערכת?**
כי הוא רץ על שרתים, **לא מתרחב כמו Kinesis/SQS**, ומחויב 24/7 גם בלי תעבורה.
משתמשים בו רק בנקודה שבה הפרוטוקול הפתוח הוא אילוץ אמיתי.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **התחביר המדויק של Event Pattern JSON** — מספיק להבין שאפשר לסנן על כל שדה באירוע.
- **KPL ו-KCL לעומק** — מספיק לדעת שהן ספריות אופטימיזציה ל-producer/consumer.
- **הפרוטוקולים עצמם** (MQTT מול AMQP מול STOMP) — מספיק לזהות שהם **פתוחים** ⇒ Amazon MQ.
- **חישוב מדויק של מספר shards** — מספיק לדעת **1 MB/s נכנס · 2 MB/s יוצא** לכל shard.
- **תצורות ActiveMQ מול RabbitMQ** לעומק.
- **פרטי Managed Service for Apache Flink** — רק לזהות שהוא מעבד זרמים בזמן אמת.
- **תמחור מדויק בדולרים** — משתנה לפי Region. חשוב **מודל** החיוב.
- **רשימת כל 18+ היעדים** של EventBridge בעל פה — מספיק להכיר את המרכזיים.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **EventBridge = CloudWatch Events לשעבר.** שני מצבים: **Schedule/Cron** ו-**Event Pattern**.
- **שלושה סוגי Bus:** Default (AWS) · Partner (SaaS) · Custom (האפליקציות שלכם).
- **Resource-based Policy על ה-bus** = ריכוז אירועים מכל ה-Organization בחשבון אחד.
- **EventBridge Archive + Replay** — היכולת היחידה בין ה-routers לנגן היסטוריה מחדש.
- **Schema Registry** — מסיק schema, מייצר code bindings, **versioned**.
- **EventBridge + CloudTrail = יירוט כל קריאת API** (`DeleteTable`, `AuthorizeSecurityGroupIngress`).
- **EventBridge הוא router, לא תור.** לעמידות — שמים **SQS כיעד**.
- **KDS: retention עד 365 ימים · replay ✅ · הנתון לא נמחק עד לפקיעה.**
- **סדר ב-KDS = ברמת shard, לפי Partition Key.** קרדינליות נמוכה = **hot shard**.
- **Provisioned: 1 MB/s או 1,000 רשומות/שנייה נכנס · 2 MB/s יוצא לכל shard.** חיוב **shard-hour**.
- **On-Demand: ברירת מחדל 4 MB/s או 4,000 רשומות/שנייה**, מתרחב לפי פיק **30 הימים האחרונים**.
- **Enhanced Fan-Out: 2 MB/s לכל צרכן לכל shard**, בדחיפה במקום משיכה.
- **Firehose: near real-time · מנוהל לחלוטין · אין אחסון · אין replay.**
- **יעדי Firehose: S3, Redshift, OpenSearch, צד שלישי, HTTP endpoint.** המרה ל-**Parquet/ORC** + Lambda.
- **Amazon MQ = ActiveMQ/RabbitMQ מנוהל** לפרוטוקולים פתוחים: **MQTT, AMQP, STOMP, Openwire, WSS**.
- **Amazon MQ רץ על שרתים, לא מתרחב כמו SQS/SNS**, ותומך ב-**Multi-AZ Active/Standby עם EFS**.
- **S3 Event Notifications:** ObjectCreated/Removed/Restore/Replication, סינון בשם, יעדים SNS/SQS/Lambda.
- **S3 → EventBridge** נותן סינון לפי **metadata וגודל**, ריבוי יעדים, ו-**Archive/Replay**.
- **הכול at-least-once. הצרכנים חייבים להיות idempotent.**

---

## 11. ✅ בדיקת הבנה

1. צריך להריץ סקריפט ניקוי כל 4 שעות. באיזה שירות בוחרים?
2. מה ההבדל המרכזי בין Kinesis Data Streams ל-Amazon Data Firehose?
3. איפה מובטח הסדר ב-Kinesis, ואיך שולטים בו?
4. חייבים להתריע על כל שינוי ב-Security Group בחשבון. איך בונים את זה?
5. יעד ה-Lambda של rule ב-EventBridge היה throttled ואירועים אבדו. מה מתקנים בארכיטקטורה?
6. אפליקציה ותיקה מדברת STOMP ועוברת לענן. מה בוחרים ולמה לא SQS?
7. `country` נבחר כ-Partition Key ל-50 מיליון אירועים בשעה. מה יקרה?
8. צריך לנגן מחדש אירועים עסקיים מלפני שבועיים. אילו שתי אפשרויות יש?
9. מתי Provisioned Mode של KDS עדיף על On-Demand?
10. למה SNS לא מתאים כשחייבים ערובה שאף אירוע לא ילך לאיבוד?

<details>
<summary>תשובות</summary>

1. **EventBridge Schedule Rule** (cron) עם Lambda כיעד. זו ההחלפה המודרנית ל-cron על EC2.
2. **KDS מאחסן** (עד 365 ימים), תומך ב-**replay**, ודורש קוד producer וגם consumer — real-time.
   **Firehose לא מאחסן, לא תומך ב-replay**, מנוהל לחלוטין ובלי קוד צרכן — **near real-time**,
   ותפקידו לטעון ל-S3/Redshift/OpenSearch/HTTP.
3. הסדר מובטח **בתוך shard בלבד**. שולטים בו דרך **Partition Key** — כל הרשומות עם אותו מפתח
   נוחתות באותו shard ולכן נשמרות בסדר. אין ערובת סדר בין shards.
4. **CloudTrail** מתעד את `AuthorizeSecurityGroupIngress` → **EventBridge Rule** עם pattern על שם ה-API
   → **SNS Topic** להתראה (ואפשר גם Lambda ל-auto-remediation).
5. **EventBridge אינו תור עמיד.** משנים את היעד של ה-rule ל-**SQS**, וה-Lambda צורך מהתור.
   כך מתקבל buffering, ניסיונות חוזרים ו-**DLQ**.
6. **Amazon MQ** — הוא תומך ב-STOMP ובפרוטוקולים פתוחים נוספים, כך שאין צורך לשכתב את האפליקציה.
   SQS משתמש ב-API קנייני של AWS, ומעבר אליו דורש שינוי קוד.
7. **Hot shard.** מספר קטן של ערכים = רוב הרשומות נופלות על אותו shard →
   **throttling** בזמן ששאר ה-shards כמעט ריקים. צריך מפתח בעל קרדינליות גבוהה, כמו `device-id`.
8. (א) **EventBridge Archive + Replay** אם האירועים עברו ב-bus והארכוב הופעל.
   (ב) **Kinesis retention** אם הזרם מוגדר עם retention של 14 יום או יותר.
   SQS ו-SNS לא נותנים replay.
9. כשהעומס **צפוי ויציב** — אז מספר ה-shards ידוע וחיוב shard-hour זול יותר מ-On-Demand.
   On-Demand מתאים לעומס חדש או בלתי צפוי, ומתרחב לפי הפיק של 30 הימים האחרונים.
10. כי **SNS לא שומר נתונים**. אם המנוי לא זמין והמסירה נכשלה — ההודעה אבודה.
    מוסיפים **SQS בין ה-topic לצרכן** (Fan Out), וכך מקבלים persistence, retries ו-DLQ.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[28 - SQS and SNS]] · [[30 - Application Decoupling]] · [[25 - Lambda]] · [[31 - Monitoring and Logging]] · [[32 - Security Services]] · [[18 - S3 Advanced Features]] · [[38 - Serverless and Modern Architectures]] · [[39 - Architecture Decision Making]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 7300–7495, 10713–10822, 11000–11041, 15257–15371
