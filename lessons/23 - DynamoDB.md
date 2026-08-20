---
lesson: 23
title: DynamoDB
domain: Design High-Performing Architectures
services: [DynamoDB, DAX, DynamoDB Streams, Lambda, API Gateway, S3]
tags: [saa-c03, database, nosql, serverless, dynamodb]
---

# 23 — DynamoDB

> [!abstract] בשורה אחת
> DynamoDB הוא NoSQL serverless עם latency של מילישניות בודדות בכל קנה מידה —
> ובמבחן הוא כמעט תמיד התשובה ל-*key-value*, *serverless*, *millions of requests* ו-*session data*.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה RDS נשבר איפה ש-DynamoDB לא |
| 2 | איך זה עובד | Tables, Primary Key, Partition/Sort Key, סוגי נתונים |
| 3 | פירוק מפורט | **RCU/WCU**, **LSI מול GSI**, DAX, Streams, Global Tables, TTL, גיבויים |
| 4 | עלות | **Provisioned מול On-Demand — טבלת החלטה** |
| 5 | השוואות | DAX מול ElastiCache · Streams מול Kinesis · Query מול Scan |
| 6 | Well-Architected | DynamoDB לפי ששת ה-Pillars |
| 7 | מלכודות | hot partition · Scan · GSI לא נותן strong consistency |
| 8 | Scenario | Serverless API + משחק גלובלי |

**מונחי מפתח בשיעור:** `Partition Key` · `Sort Key` · `RCU` · `WCU` · `LSI` · `GSI` · `DAX` · `Streams` · `Global Tables` · `TTL` · `PITR`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- מסד relational מתחיל להיחנק סביב מיליוני בקשות בשנייה — צריך sharding ידני ומורכב.
- Scaling של RDS הוא **vertical** ברובו: instance גדול יותר, downtime, ותקרה.
- אפליקציית מובייל צריכה **מילישניות בודדות** בקצה השני של העולם.
- ה-schema משתנה כל ספרינט; `ALTER TABLE` על טבלה של מיליארד שורות הוא סיוט.
- קמפיין שיווקי מכפיל את העומס פי 50 בחצי שעה. אין זמן להקצות capacity.
- Session data של אתר לא צריך joins — צריך רק "תן לי את הרשומה הזאת, מהר".

### מה השירות פותר

- **Serverless לחלוטין** — אין instance, אין patching, אין maintenance window.
- **Multi-AZ כברירת מחדל** — שכפול בין AZs מובנה, בלי שתעשו כלום.
- **מיליוני בקשות בשנייה, טריליוני שורות, מאות TB** — בלי sharding ידני.
- **latency עקבי של מילישנייה חד-ספרתית**, ללא קשר לגודל הטבלה.
- **Schema גמיש** — כל item יכול להחזיק attributes שונים.
- **Read ו-Write מנותקים** — אפשר להקצות להם capacity נפרד.
- **תמיכה ב-Transactions** — למרות שזה NoSQL.
- **אינטגרציה מלאה עם IAM** — הרשאות ברמת item ואפילו attribute.

> [!tip] האנלוגיה
> RDS הוא **ספרייה עם קטלוג** — אפשר לשאול שאלות מורכבות ולצלוב מדפים (joins),
> אבל ככל שהספרייה גדלה הקטלוג נהיה איטי.
> DynamoDB הוא **מערך תאי דואר ענק**: אתם נותנים מספר תא — מקבלים את התוכן מיד,
> ולא משנה אם יש אלף תאים או מיליארד. אבל **אי אפשר לשאול "מי כל מי שגר ברחוב X"**
> אלא אם בניתם מראש אינדקס לזה.

---

## 2. ⚙️ איך זה עובד

### 2.1 המבנה הבסיסי

```text
Table
 └── Item   (= "שורה")           ← עד מיליארדים, ללא הגבלה
      └── Attribute  (= "עמודה")  ← יכול להתווסף בכל רגע, יכול להיות null
```

| עובדה | ערך |
|---|---|
| **Primary Key** | **חייב להיקבע ביצירת הטבלה. לא ניתן לשינוי** |
| מספר items בטבלה | **ללא הגבלה** |
| **גודל item מקסימלי** | **400 KB** |
| Attributes | מתווספים דינמית, יכולים להיות null |
| Table Classes | **Standard** ו-**Standard-IA** (לנתונים שנקראים לעיתים רחוקות) |

**סוגי הנתונים הנתמכים:**

| קטגוריה | טיפוסים |
|---|---|
| **Scalar** | String, Number, Binary, Boolean, Null |
| **Document** | List, Map |
| **Set** | String Set, Number Set, Binary Set |

> [!warning] 400 KB — המספר שנשאל
> item בודד **לא יכול לעבור 400 KB**.
> קובץ גדול? שומרים אותו ב-**S3** ומחזיקים ב-DynamoDB רק את ה-**pointer** (ה-key).
> זו תבנית סטנדרטית שנשאלת ישירות.

### 2.2 Primary Key — שתי הצורות

**צורה א': Partition Key בלבד (Simple Primary Key)**

- מפתח יחיד. חייב להיות **ייחודי** לכל item.
- DynamoDB מריצה עליו hash כדי להחליט באיזה partition פיזי ה-item יישב.

**צורה ב': Partition Key + Sort Key (Composite Primary Key)**

- הצירוף של שניהם חייב להיות ייחודי.
- כל ה-items עם אותו partition key יושבים **יחד**, ממוינים לפי ה-sort key.
- זה מה שמאפשר **Query על טווח** בתוך partition.

**דוגמה — טבלת תוצאות משחק:**

```text
┌──── Primary Key ────┐  ┌──── Attributes ────┐
Partition Key   Sort Key
User_ID         Game_ID   Score   Result
─────────────   ───────   ─────   ──────
7791a3d6-…      4421       92      Win
873e0634-…      1894       14      Lose
873e0634-…      4521       77      Win
```

- `User_ID` לבדו **לא ייחודי** — למשתמש `873e0634` יש שתי שורות.
- הצירוף `User_ID + Game_ID` **כן ייחודי**.
- אפשר לשאול "כל המשחקים של המשתמש הזה" ביעילות — זה Query על partition אחד.

> [!warning] בחירת ה-Partition Key היא ההחלטה החשובה ביותר
> Partition key עם מעט ערכים שונים (למשל `country` עם 3 מדינות) יוצר **hot partition**:
> כל התעבורה נופלת על אותו partition פיזי, שמקבל רק חלק מה-capacity → **throttling**.
> **הכלל:** partition key צריך **high cardinality** ו**התפלגות אחידה**.

### 2.3 Query מול Scan — ההבדל שקובע עלות וביצועים

| קריטריון | **Query** | **Scan** |
|---|---|---|
| מה עושה | ניגש **ישירות** ל-partition לפי partition key | קורא **את כל הטבלה** ואז מסנן |
| חובה לספק | partition key | כלום |
| עלות | קורא רק את מה שצריך | **צורך RCU על כל הטבלה** |
| ביצועים | מילישניות בודדות, יציב | מידרדר ככל שהטבלה גדלה |
| מתי בכל זאת | הדפוס הרגיל | ETL חד-פעמי, טבלה זעירה |

> [!tip] הכלל שנשאל בכל ניסוח אפשרי
> **צריך לשאול לפי attribute שאינו ה-partition key? התשובה היא GSI, לא Scan.**
> "חיפוש משתמש לפי email כשה-PK הוא userId" → **Global Secondary Index**.

### 2.4 Read/Write Capacity Modes

| מצב | איך עובד |
|---|---|
| **Provisioned** (ברירת מחדל) | אתם מצהירים על מספר קריאות/כתיבות בשנייה. משלמים על **RCU** ו-**WCU** שהוקצו. אפשר להוסיף **Auto Scaling** |
| **On-Demand** | ה-capacity עולה ויורד **אוטומטית**. אין תכנון. משלמים **לפי בקשה בפועל** |

- **Provisioned** דורש תכנון מראש — אבל **זול משמעותית** לעומס חזוי.
- **On-Demand יקר יותר לכל בקשה** — אבל מצוין ל**עומס בלתי צפוי** ולזינוקים חדים.
- אפשר להחליף בין המצבים (עם מגבלת תדירות).

פירוט מלא של השיקול הכלכלי — סעיף 4.

---

## 3. 🔍 פירוק מפורט

### 3.1 RCU ו-WCU — הנוסחאות

זה החישוב שנשאל ישירות במבחן.

**RCU — Read Capacity Unit:**

```text
1 RCU  =  קריאה אחת strongly consistent  של item עד 4 KB בשנייה
       =  שתי קריאות eventually consistent של item עד 4 KB בשנייה
```

**WCU — Write Capacity Unit:**

```text
1 WCU  =  כתיבה אחת של item עד 1 KB בשנייה
```

**כללי העיגול:** תמיד **מעגלים כלפי מעלה** לגודל היחידה הבא.

| תרחיש | חישוב | תוצאה |
|---|---|---|
| 10 קריאות/שנייה, item 4 KB, strongly consistent | 10 × (4÷4) | **10 RCU** |
| 10 קריאות/שנייה, item 4 KB, eventually consistent | 10 × (4÷4) ÷ 2 | **5 RCU** |
| 10 קריאות/שנייה, item 6 KB, strongly consistent | 6 KB → מעגלים ל-8 KB → 2 יחידות · 10 × 2 | **20 RCU** |
| 20 כתיבות/שנייה, item 2 KB | 2 KB → 2 יחידות · 20 × 2 | **40 WCU** |
| 20 כתיבות/שנייה, item 0.5 KB | מעגלים ל-1 KB · 20 × 1 | **20 WCU** |

**מכפילים מיוחדים:**

| סוג פעולה | עלות |
|---|---|
| Eventually consistent read | **חצי** RCU |
| Strongly consistent read | RCU מלא |
| **Transactional read** | **פי 2** RCU |
| **Transactional write** | **פי 2** WCU |

> [!tip] קיצור זיכרון
> **קריאה = 4 KB · כתיבה = 1 KB.** קריאות **זולות פי 4** מכתיבות באותו נפח.
> **Eventually consistent = חצי מחיר. Transactional = כפול.**

### 3.2 LSI מול GSI — הטבלה המכרעת

שני האינדקסים מאפשרים לשאול לפי attribute שאינו ה-primary key. הם **שונים מאוד**.

| קריטריון | **LSI** (Local Secondary Index) | **GSI** (Global Secondary Index) |
|---|---|---|
| **Partition Key** | **חייב להיות זהה** לזה של הטבלה | **יכול להיות אחר לגמרי** |
| **Sort Key** | **חובה**, ושונה מזה של הטבלה | אופציונלי |
| **מתי מגדירים** | ⚠️ **רק בזמן יצירת הטבלה. אי אפשר להוסיף אחר כך** | **בכל רגע** — ליצור, לשנות ולמחוק |
| **Capacity** | **משתמש ב-RCU/WCU של הטבלה** | ⚠️ **capacity משלו, נפרד לגמרי** |
| **Strongly consistent reads** | ✅ **כן** | ❌ **eventually consistent בלבד** |
| **מגבלת גודל** | ⚠️ **10 GB לכל ערך של partition key** | אין מגבלה |
| כמה מותרים לטבלה | **5** | **20** (soft limit, ניתן להגדלה) |
| מה קורה כשמותקן throttle | מושך מה-capacity של הטבלה | ⚠️ **throttle ב-GSI יכול לחסום כתיבות בטבלה הבסיסית** |
| Projection | ניתן לבחור אילו attributes | ניתן לבחור אילו attributes |

> [!warning] שתי הנקודות שנשאלות הכי הרבה
> 1. **LSI חייב להיות מוגדר ביצירת הטבלה.** שאלה שאומרת *"add an index to an existing table"*
>    → התשובה היא **GSI**, כי LSI כבר לא אפשרי.
> 2. **GSI לא נותן strongly consistent reads.** שאלה שדורשת *"index with strong consistency"*
>    → **LSI** (ורק אם הטבלה נבנתה איתו מראש).

> [!warning] המלכודת של throttling ב-GSI
> אם ה-GSI מקבל פחות WCU ממה שהוא צריך, הכתיבות אליו נכשלות —
> ו-DynamoDB **תחסום את הכתיבה לטבלה הבסיסית** כדי לשמור על עקביות.
> **המסקנה:** תמיד להקצות ל-GSI capacity **לפחות** כמו לטבלה, ולצמצם את ה-**projection**
> כדי לא לכתוב אליו attributes מיותרים.

### 3.3 DAX — DynamoDB Accelerator

- **cache in-memory מנוהל במלואו**, ייעודי ל-DynamoDB, זמין מאוד.
- פותר **read congestion** — עומס קריאות על אותם items.
- **latency של מיקרו-שניות** לנתונים ב-cache (מול מילישניות ב-DynamoDB עצמה).
- ⚠️ **לא דורש שינוי בקוד האפליקציה** — תואם ל-API הקיים של DynamoDB.
- **TTL ברירת מחדל של 5 דקות** ל-cache.

```text
Application ──▶ DAX Cluster (nodes) ──▶ DynamoDB Tables
                     │
              cache hit → מיקרו-שניות
              cache miss → ממשיך ל-DynamoDB
```

### 3.4 DAX מול ElastiCache

הם **לא מתחרים** — הם משלימים.

| קריטריון | **DAX** | **ElastiCache** |
|---|---|---|
| מה מאחסן | **items בודדים** + תוצאות **Query ו-Scan** | **תוצאות אגרגציה** ומחושבות |
| שינוי בקוד | ❌ **לא נדרש** — אותו API | ✅ **נדרש** — קוד cache מפורש |
| ייעוד | **DynamoDB בלבד** | כל מקור נתונים |
| Latency | מיקרו-שניות | מיקרו/מילישניות |
| דוגמה | `GetItem` על אותו user_id מיליון פעם | "10 השחקנים המובילים" שחושב מ-scan כבד |

> [!tip] המשפט שפותר את השאלה
> **DAX = לשמור את מה שקראתי. ElastiCache = לשמור את מה שחישבתי.**
> אם השאלה אומרת *"without changing application code"* → **DAX**.
> אם היא אומרת *"store the aggregated result of an expensive computation"* → **ElastiCache**.

### 3.5 DynamoDB Streams

**stream מסודר של שינויים ברמת ה-item** (create / update / delete) בטבלה.

**Use cases:**

- להגיב לשינוי בזמן אמת — למשל שליחת welcome email כשנוצר משתמש.
- analytics של שימוש בזמן אמת.
- הזנת טבלאות נגזרות.
- מימוש שכפול בין Regions.
- הפעלת **Lambda** על כל שינוי.

**שתי אפשרויות ה-stream:**

| קריטריון | **DynamoDB Streams** | **Kinesis Data Streams** (החדש יותר) |
|---|---|---|
| **Retention** | **24 שעות** | **שנה** |
| מספר consumers | **מוגבל** | **גבוה** |
| מי צורך | Lambda Triggers, DynamoDB Streams Kinesis Adapter (KCL) | Lambda, Kinesis Data Analytics, Firehose, Glue Streaming ETL |
| מתי בוחרים | תגובה פשוטה לשינוי | כמה צרכנים, שמירה ארוכה, pipeline אנליטי |

```text
                     ┌──▶ Lambda ──▶ SNS (התראות)
Table ──▶ Streams ───┤
                     └──▶ KCL Adapter ──▶ OpenSearch (אינדוקס)

Table ──▶ Kinesis Data Streams ──▶ Firehose ──▶ S3 / Redshift (ארכוב, analytics)
```

### 3.6 Global Tables

- הופך טבלה ל-**נגישה ב-latency נמוך מכמה Regions**.
- ⚠️ **Active-Active** — האפליקציה יכולה **לקרוא ולכתוב בכל Region**.
- השכפול הוא **דו-כיווני** בין כל ה-Regions.
- ⚠️ **תנאי מוקדם חובה: DynamoDB Streams חייב להיות מופעל.**

```text
   US-EAST-1  ◀────── two-way replication ──────▶  AP-SOUTHEAST-2
   read + write                                    read + write
```

> [!warning] Active-Active הוא לא רק פיצ'ר — הוא גם סיכון
> שתי כתיבות לאותו item בשני Regions באותו רגע נפתרות ב-**last writer wins**.
> אם האפליקציה לא בנויה לזה — יש אובדן עדכונים שקט.

### 3.7 TTL — Time To Live

- **מוחק items אוטומטית** אחרי חותמת זמן תפוגה (epoch timestamp) שאתם שומרים ב-attribute.
- שני תהליכי רקע: אחד **סורק ומסמן** items שפגו, ואחד **מוחק** אותם.
- ⚠️ המחיקה היא **eventual** — יכולה לקחת עד 48 שעות. אין ערובה למחיקה ברגע הפקיעה.
- ⚠️ **המחיקה לא צורכת WCU** — היא חינם.

**Use cases:**

| מקרה | למה TTL |
|---|---|
| **Session data** של אתר | הסשן פג — הרשומה נעלמת מעצמה |
| הקטנת נפח אחסון | שומרים רק את מה שרלוונטי עכשיו |
| **חובות רגולטוריות** | "לא לשמור נתוני לקוח מעל X ימים" |

> [!tip] הצמד שנשאל
> **DynamoDB + TTL הוא תחליף ל-ElastiCache כ-session store.**
> זה כתוב במפורש בסיכום הרשמי: *"can replace ElastiCache as a key/value store"*.

### 3.8 גיבויים ו-DR

| סוג | פירוט |
|---|---|
| **PITR — Point-In-Time Recovery** | גיבוי **רציף**, אופציונלי, ל-**35 הימים האחרונים**. שחזור לכל נקודת זמן בחלון |
| **On-Demand Backups** | גיבוי **מלא** לשמירה ארוכת טווח, עד שמוחקים אותו במפורש. **לא משפיע על ביצועים או latency** |

- ⚠️ **בשני המקרים תהליך השחזור יוצר טבלה חדשה.** הוא **לא** דורס את הקיימת.
- On-Demand Backups ניתנים לניהול דרך **AWS Backup**, שמאפשר גם **העתקה בין Regions**.

### 3.9 אינטגרציה עם S3

**Export ל-S3:**

- ⚠️ **דורש PITR מופעל.**
- עובד לכל נקודת זמן ב-**35 הימים האחרונים**.
- ⚠️ **לא צורך RCU** — לא משפיע על ה-capacity של הטבלה.
- פורמטים: **DynamoDB JSON** או **ION**.
- שימושים: ניתוח נתונים ב-**Athena**, שמירת snapshots לביקורת, ETL לפני החזרה.

**Import מ-S3:**

- פורמטים: **CSV**, DynamoDB JSON, ION.
- ⚠️ **לא צורך WCU.**
- ⚠️ **יוצר טבלה חדשה** — לא מוסיף לטבלה קיימת.
- שגיאות import נרשמות ב-**CloudWatch Logs**.

```text
DynamoDB ──export (PITR, ללא RCU)──▶ S3 ──▶ Athena (שאילתות SQL)
DynamoDB ◀──import (ללא WCU, טבלה חדשה)── S3 (.csv / .json / .ion)
```

### 3.10 Serverless API — התבנית הקלאסית

```text
Client ──REST──▶ API Gateway ──PROXY──▶ Lambda ──CRUD──▶ DynamoDB
```

זו הארכיטקטורה ה-serverless הנפוצה ביותר במבחן. **אין תשתית לנהל.**

מה **API Gateway** מוסיף מעל Lambda:

| יכולת | פירוט |
|---|---|
| **WebSocket** | תמיכה בפרוטוקול לתקשורת דו-כיוונית |
| **גרסאות** | v1, v2 במקביל |
| **סביבות** | dev / test / prod (stages) |
| **אבטחה** | Authentication ו-Authorization |
| **API Keys + Throttling** | הגבלת קצב לכל לקוח |
| **Swagger / OpenAPI** | ייבוא הגדרת API מוכנה |
| **Transform & Validate** | עיבוד ובדיקת requests ו-responses |
| **SDK Generation** | יצירת SDK ומפרטים ללקוחות |
| **Caching** | cache לתשובות ה-API |

פירוט מלא ב-[[27 - API Gateway]] וב-[[25 - Lambda]].

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **Capacity** | RCU/WCU שהוקצו (Provisioned) **או** בקשות בפועל (On-Demand) | הפריט הראשי בחשבון |
| **Storage** | GB-month | Standard או **Standard-IA** (זול יותר לאחסון, יקר יותר לגישה) |
| **Indexes (GSI)** | ⚠️ **capacity ואחסון נפרדים משלו** | הגורם הכי מופתע בחשבון |
| **Streams** | לכל read request unit מה-stream | Kinesis Data Streams מתומחר בנפרד |
| **PITR** | GB-month של הגיבוי הרציף | לפי גודל הטבלה |
| **On-Demand Backups** | GB-month | נשמרים עד מחיקה מפורשת |
| **Global Tables** | ⚠️ **replicated WCU בכל Region** + אחסון בכל Region | מכפילים את העלות במספר ה-Regions |
| **DAX** | לפי **שעת node** | כמו EC2 — משלמים גם כשה-cache ריק |
| **Data Transfer** | GB יוצא, ובין Regions | Global Tables מייצר transfer בין Regions |
| **TTL deletions** | **0** | המחיקה לא צורכת WCU |
| **Export/Import ל-S3** | לפי GB | **לא צורך RCU/WCU** |

### Provisioned מול On-Demand — טבלת ההחלטה

| קריטריון | **Provisioned** | **On-Demand** |
|---|---|---|
| תכנון capacity | ✅ **נדרש** | ❌ אין |
| מודל תשלום | על מה ש**הוקצה**, גם אם לא נוצל | על **בקשות בפועל** בלבד |
| מחיר לבקשה | **נמוך משמעותית** | **גבוה** |
| Auto Scaling | ✅ אופציונלי (RCU/WCU) | מובנה |
| התמודדות עם spike פתאומי | ⚠️ Auto Scaling מגיב באיחור → **throttling** | ✅ **מיידית** |
| מתאים ל | עומס **חזוי ויציב** | עומס **בלתי צפוי**, זינוקים חדים, טבלה חדשה בלי היסטוריה |
| Reserved Capacity | ✅ הנחה נוספת בהתחייבות | ❌ לא קיים |

**מתי כל אחד זול יותר:**

| מצב | הזול יותר | למה |
|---|---|---|
| עומס יציב 24/7 | **Provisioned** | ניצול גבוה של ה-capacity שהוקצה |
| עומס יציב + התחייבות שנתית | **Provisioned + Reserved Capacity** | ההוזלה הגדולה ביותר |
| שיא של שעתיים ביום, שקט בשאר | **On-Demand** (או Provisioned + Auto Scaling) | ב-Provisioned משלמים על השיא כל היממה |
| Launch של מוצר חדש, אין נתוני עבר | **On-Demand** | אין על מה לבסס תחזית, ו-throttling הוא הרסני |
| Dev/Test עם תעבורה אקראית | **On-Demand** | הרוב המוחלט של הזמן אין תעבורה בכלל |
| מיליוני בקשות/שנייה, קצב ידוע | **Provisioned** | הפער במחיר לבקשה מצטבר לסכומים גדולים |

> [!tip] כלל האצבע
> **ניצול גבוה ויציב → Provisioned. חוסר ודאות או spikes → On-Demand.**
> אפשר גם להתחיל ב-On-Demand, למדוד חודש, ולעבור ל-Provisioned עם Auto Scaling.

### 🚩 עלויות נסתרות

- **GSI הוא טבלה נוספת לכל דבר** — capacity משלו, אחסון משלו, וכל כתיבה לטבלה
  מייצרת כתיבה גם אליו. **Projection של `ALL` מכפיל את עלות האחסון.**
- **Global Tables מכפילות הכל** — כתיבה ב-Region אחד = **replicated WCU** בכל Region אחר,
  ועוד אחסון ועוד transfer.
- **DAX nodes רצים 24/7** — משלמים לשעה, גם כשה-hit rate נמוך.
  DAX משתלם רק ב-workload **read-heavy עם hit rate גבוה**.
- **Scan** — צורך RCU על **כל הטבלה** בכל הרצה. Scan מתוזמן על טבלה גדולה הוא חור בתקציב.
- **Provisioned שהוקצה לשיא** — משלמים 24 שעות על capacity שנחוץ שעתיים.
- **PITR על טבלה ענקית** — גיבוי רציף של TB עולה כסף אמיתי.
- **Streams עם הרבה consumers** — כל צרכן קורא ומחויב.
- **Retries בגלל throttling** — כל retry הוא בקשה נוספת שמחויבת.

### 💡 טיפים לחיסכון

- **Query במקום Scan.** תמיד. אם צריך גישה לפי attribute אחר — **GSI**.
- **צמצמו את ה-Projection של GSI** ל-`KEYS_ONLY` או `INCLUDE` במקום `ALL`.
- **Eventually consistent reads** היכן שאפשר — **חצי מחיר** מול strongly consistent.
- **TTL** למחיקת נתונים זמניים — המחיקה **חינם** ומקטינה אחסון.
- **Standard-IA Table Class** לטבלאות שנקראות לעיתים רחוקות.
- **Provisioned + Auto Scaling + Reserved Capacity** לעומס יציב.
- **DAX רק ב-read-heavy** — לפני שמדליקים, למדוד את יחס הקריאות לכתיבות.
- **Global Tables רק ל-Regions שבאמת צריך** — כל Region נוסף הוא עלות מלאה.
- **Export ל-S3 + Athena** לניתוחים כבדים במקום Scan על הטבלה החיה.

---

## 5. ⚖️ השוואות מכריעות

### DynamoDB מול RDS

| קריטריון | DynamoDB | RDS |
|---|---|---|
| מודל | NoSQL key-value / document | Relational (SQL) |
| Schema | **גמיש**, משתנה מיידית | קשיח, `ALTER TABLE` |
| Joins ושאילתות מורכבות | ❌ **לא** | ✅ כן |
| Scaling | **אופקי, אוטומטי, ללא הגבלה** | ורטיקלי בעיקר + read replicas |
| Latency | **מילישנייה חד-ספרתית, עקבי** | תלוי בשאילתה ובעומס |
| ניהול | **Serverless מלא** | instance, patching, maintenance window |
| Multi-Region active-active | ✅ **Global Tables** | Aurora Global (read בלבד ברוב המקרים) |
| מתי בוחרים | access pattern ידוע, scale עצום, serverless | שאילתות מורכבות, transactions רלציוניות, joins |

### Query מול Scan מול GSI

| מצב | הכלי הנכון |
|---|---|
| יודעים את ה-partition key | **Query** |
| יודעים partition key + טווח sort key | **Query** עם condition |
| צריך לחפש לפי attribute אחר, קבוע | **GSI** + Query עליו |
| צריך sort key חלופי עם strong consistency | **LSI** (רק אם הוגדר ביצירה) |
| ETL חד-פעמי על כל הטבלה | **Scan** (או Export ל-S3 + Athena) |

### DynamoDB Streams מול Kinesis Data Streams

| קריטריון | DynamoDB Streams | Kinesis Data Streams |
|---|---|---|
| Retention | **24 שעות** | **שנה** |
| Consumers | מספר מוגבל | **גבוה** |
| צרכנים נתמכים | Lambda Triggers, KCL Adapter | Lambda, Kinesis Analytics, Firehose, Glue Streaming ETL |
| מתי | תגובה מיידית פשוטה, Global Tables | pipeline אנליטי, כמה צרכנים, replay ארוך |

### DAX מול ElastiCache מול Global Tables

| קריטריון | DAX | ElastiCache | Global Tables |
|---|---|---|---|
| מה פותר | **read congestion** על DynamoDB | cache כללי לתוצאות מחושבות | **latency גיאוגרפי** + DR |
| שינוי קוד | ❌ לא נדרש | ✅ נדרש | ❌ לא נדרש |
| Latency | מיקרו-שניות | מיקרו/מילישניות | מילישניות **ב-Region המקומי** |
| מה משכפל | cache בלבד | cache בלבד | **את הנתונים עצמם** |

> [!info] שורה תחתונה
> **קריאות חוזרות איטיות → DAX. אגרגציות יקרות → ElastiCache.
> משתמשים ביבשת אחרת → Global Tables.** אלה שלוש בעיות שונות.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-DynamoDB** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | הטבלה מנוטרת ויש runbook לכשל הצפוי | CloudWatch Alarms על `ThrottledRequests` ועל `ConsumedCapacity`; מעקב אחרי hot keys דרך CloudWatch Contributor Insights; runbook לשינוי partition key design |
| **Security** | הרשאות מדויקות והצפנה בכל שכבה | IAM fine-grained (הרשאה ברמת item/attribute עם `dynamodb:LeadingKeys`); הצפנה ב-**KMS**; **VPC Gateway Endpoint** ל-DynamoDB; CloudTrail לכל פעולת control plane |
| **Reliability** | הנתונים שורדים אסון והלקוח שורד throttling | **PITR** דלוק (35 יום) + on-demand backups ב-AWS Backup עם עותק cross-region; **Global Tables** לפי ה-RTO; **retries עם exponential backoff** בצד הלקוח |
| **Performance Efficiency** | ה-latency נשאר קבוע גם בשיא | partition key בעל **cardinality גבוהה**; **Query במקום Scan**; pagination במקום משיכת הכל; **DAX** ל-read-heavy; GSI לדפוסי גישה נוספים |
| **Cost Optimization** | משלמים על מה שבאמת נצרך | **Provisioned + Auto Scaling** לעומס יציב, **On-Demand** לספייקים; **projection מצומצם** ב-GSI; **TTL** למחיקה חינמית; Standard-IA לטבלאות קרות |
| **Sustainability** | פחות עבודה מיותרת של המסד | serverless מבטל instances בטלים; **מניעת Scan** חוסכת קריאה של טבלה שלמה; TTL מונע אחסון נתונים מתים; right-sizing של capacity |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "serverless NoSQL, single-digit millisecond latency" | **DynamoDB** |
| "millions of requests per second, no capacity planning" | **DynamoDB On-Demand** |
| "predictable, steady traffic — reduce cost" | **Provisioned + Auto Scaling** (+ Reserved Capacity) |
| "query by an attribute that is not the primary key" | **GSI** — **לא** Scan |
| "add an index to an existing table" | **GSI** (LSI כבר לא אפשרי) |
| "index that supports strongly consistent reads" | **LSI** |
| "microsecond latency without changing application code" | **DAX** |
| "cache an expensive aggregation result" | **ElastiCache** |
| "read and write in multiple regions" / "active-active" | **Global Tables** (+ Streams כתנאי מוקדם) |
| "automatically delete expired session data" | **TTL** |
| "replace ElastiCache for session storage" | **DynamoDB + TTL** |
| "trigger a Lambda when an item changes" | **DynamoDB Streams** |
| "stream retention longer than 24 hours" | **Kinesis Data Streams** (במקום DynamoDB Streams) |
| "restore the table to any point in the last 35 days" | **PITR** |
| "run SQL analytics on DynamoDB data" | **Export to S3** (דורש PITR) **+ Athena** |
| "store files larger than 400 KB" | **S3** + pointer ב-DynamoDB |
| "requests are being throttled" | **hot partition** — partition key לא מפוזר |

### טעויות נפוצות

> [!warning] מלכודת 1 — Scan במקום GSI
> **הניסוח:** "Users must be searchable by email, but the partition key is userId."
> **הטעות:** לבחור Scan עם filter.
> **הנכון:** **Global Secondary Index על email.**
> Scan קורא את **כל** הטבלה וצורך RCU על הכל — הוא לא פתרון, הוא תסמין.

> [!warning] מלכודת 2 — להוסיף LSI לטבלה קיימת
> **הניסוח:** "Add a Local Secondary Index to the existing orders table."
> **הטעות:** להניח שאפשר להוסיף אינדקסים חופשי.
> **הנכון:** **LSI חייב להיות מוגדר בזמן יצירת הטבלה** ואי אפשר להוסיף אותו אחר כך.
> לטבלה קיימת — **GSI**.

> [!warning] מלכודת 3 — לצפות ל-strong consistency מ-GSI
> **הניסוח:** "Use a GSI to guarantee the latest value is always returned."
> **הטעות:** להניח שאינדקס = אותה עקביות.
> **הנכון:** **GSI תומך רק ב-eventually consistent reads.**
> ל-strongly consistent על מפתח חלופי — **LSI** בלבד.

> [!warning] מלכודת 4 — partition key בעל cardinality נמוכה
> **הניסוח:** "Use `status` (ACTIVE/INACTIVE) as the partition key."
> **הטעות:** לבחור מפתח נוח לוגית.
> **הנכון:** **hot partition.** שני ערכים בלבד → כל התעבורה על שני partitions →
> throttling מיידי. צריך **cardinality גבוהה** והתפלגות אחידה.

> [!warning] מלכודת 5 — DAX כמקור אמת
> **הניסוח:** "Write directly to DAX to improve write performance."
> **הטעות:** להתייחס ל-DAX כאל מסד נתונים.
> **הנכון:** DAX הוא **read cache**. הוא פותר **read congestion**, לא כתיבות.
> ה-source of truth תמיד DynamoDB, ויש **TTL של 5 דקות** ל-cache — כלומר נתון שעודכן
> יכול להיקרא ישן.

> [!warning] מלכודת 6 — item גדול מ-400 KB
> **הניסוח:** "Store the 5 MB PDF as an attribute in the DynamoDB item."
> **הטעות:** להניח שאין תקרה.
> **הנכון:** **item מקסימלי 400 KB.** שומרים את הקובץ ב-**S3**
> ואת ה-**S3 key** כ-attribute ב-DynamoDB.

> [!warning] מלכודת 7 — Global Tables בלי Streams
> **הניסוח:** "Enable Global Tables to replicate the table to eu-west-1."
> **הטעות:** להניח שזו הגדרה עצמאית.
> **הנכון:** ⚠️ **DynamoDB Streams חייב להיות מופעל** כתנאי מוקדם.

> [!warning] מלכודת 8 — שחזור דורס את הטבלה
> **הניסוח:** "Restore from PITR to roll back the production table in place."
> **הטעות:** להניח שהשחזור מחזיר את הטבלה למצב קודם.
> **הנכון:** **כל שחזור — PITR או on-demand — יוצר טבלה חדשה.**
> צריך לתכנן את המעבר (שינוי ה-endpoint באפליקציה או rename).

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

משחק מובייל גלובלי לקראת launch.

- שחקנים בארה"ב, אירופה ואסיה — כולם צריכים **latency נמוך** ויכולת **כתיבה מקומית**.
- ה-launch צפוי לזינוק עומס בלתי צפוי לחלוטין.
- לוח תוצאות שנקרא **הרבה יותר משנכתב** — אותם items שוב ושוב.
- צריך לחפש שחקן גם לפי `username`, לא רק לפי `player_id`.
- Session של שחקן צריך להימחק אוטומטית אחרי 24 שעות.
- כל שינוי בתוצאה צריך להזין pipeline analytics שנשמר לשנה.
- דרישה: שחזור לכל נקודת זמן ב-35 הימים האחרונים.
- קבצי אווטאר בגודל 2 MB לכל שחקן.

**הארכיטקטורה:**

```text
   Client (Mobile)
        │  REST
        ▼
   API Gateway ──PROXY──▶ Lambda ──┐
                                   │
                              DAX Cluster        ← cache ללוח התוצאות
                                   │
                                   ▼
        ┌──────────── DynamoDB Global Table ────────────┐
        │  us-east-1  ◀── two-way ──▶  eu-west-1        │
        │      ▲                          ▲             │
        │      └────── two-way ───────────┘             │
        │           ap-southeast-1                      │
        │                                               │
        │  PK: player_id   SK: game_id                  │
        │  GSI: username                                │
        │  TTL: session_expiry                          │
        │  PITR ✅   On-Demand capacity                  │
        └────────────────┬──────────────────────────────┘
                         │ Kinesis Data Streams
                         ▼
                   Firehose ──▶ S3 ──▶ Athena (analytics)

   אווטארים (2 MB) ──▶ S3   |   DynamoDB מחזיקה רק את ה-S3 key
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **DynamoDB ולא RDS** | scale עצום, access pattern פשוט לפי מפתח, serverless, latency עקבי |
| **On-Demand capacity ל-launch** | אין נתוני עבר לבסס עליהם תחזית. Auto Scaling ב-Provisioned מגיב באיחור → throttling בדיוק ברגע הקריטי |
| מעבר ל-**Provisioned + Auto Scaling** אחרי חודש | אחרי שהעומס מתייצב, Provisioned זול משמעותית לכל בקשה |
| **PK = `player_id`** | **cardinality גבוהה** והתפלגות אחידה. `country` או `status` היו יוצרים hot partition |
| **SK = `game_id`** | מאפשר Query יעיל של "כל המשחקים של השחקן" בתוך partition אחד |
| **GSI על `username`** | חיפוש לפי attribute שאינו ה-PK. **Scan היה קורא את כל הטבלה** בכל חיפוש |
| **Projection מצומצם ב-GSI** | לא לשכפל את כל ה-attributes → פחות אחסון ופחות WCU |
| **Global Tables** בשלושה Regions | active-active: שחקן באסיה **כותב** מקומית, לא חוצה אוקיינוס |
| **Streams מופעל** | ⚠️ תנאי מוקדם חובה ל-Global Tables — ובנוסף מזין את ה-analytics |
| **Kinesis Data Streams ולא DynamoDB Streams** | הדרישה היא **שנה** של retention. DynamoDB Streams נותן **24 שעות** בלבד |
| **DAX** מול לוח התוצאות | read-heavy על אותם items. מיקרו-שניות, **בלי שינוי קוד** |
| **TTL על `session_expiry`** | מחיקה אוטומטית אחרי 24 שעות, **בלי לצרוך WCU** |
| **PITR דלוק** | הדרישה המפורשת של 35 יום. גם מאפשר **Export ל-S3** ללא RCU |
| **אווטארים ב-S3** | 2 MB עוברים בהרבה את **תקרת ה-400 KB** לכל item |
| **API Gateway + Lambda** | אין תשתית לנהל, throttling ו-API keys מובנים, scaling אוטומטי |

**למה לא ElastiCache במקום DAX?**
כי כאן מדובר ב-cache של **items ותוצאות Query** מ-DynamoDB עצמה — בדיוק מה ש-DAX עושה,
ו**בלי שינוי בקוד**. ElastiCache היה מחייב לכתוב לוגיקת cache ידנית ולהתאים את הקוד.

**למה לא Read Replicas?**
DynamoDB לא עובד עם replicas. הפתרון המקביל הוא **Global Tables** (רב-Region)
ו-**DAX** (הקלת עומס קריאה) — שני מנגנונים שונים לשתי בעיות שונות.

**למה לא Scan עם filter על `username`?**
Scan קורא את **כל** הטבלה וצורך RCU על הכל, ומידרדר ככל שהטבלה גדלה.
GSI נותן גישה ישירה — עלות קבועה ללא קשר לגודל.

**מה משלים את הפתרון?**
בחירת סוג המסד מול חלופות ב-[[24 - Database Selection]],
שכבת ה-API ב-[[27 - API Gateway]], ה-compute ב-[[25 - Lambda]],
ואסטרטגיית ה-DR ב-[[34 - Disaster Recovery]].

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **תחביר ה-API המדויק** של `PutItem` / `Query` / `UpdateExpression`.
- **מנגנון ה-partitioning הפנימי** של DynamoDB ואיך היא מפצלת partitions.
- **חישובי RCU/WCU קיצוניים** עם transactions מקוננות — מספיק הכללים הבסיסיים והמכפילים.
- **מגבלות quota מדויקות** של מספר טבלאות ו-GSIs — soft limits.
- **פרטי DynamoDB Streams Kinesis Adapter (KCL)** ברמת הקוד.
- **פורמט ION** לעומק — מספיק לדעת שהוא אחת מאפשרויות ה-export/import.
- **אלגוריתם פתרון הקונפליקטים** של Global Tables — מספיק **last writer wins**.
- **תמחור מדויק** של RCU/WCU בדולרים — משתנה לפי Region.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **DynamoDB = NoSQL serverless, Multi-AZ כברירת מחדל, מילישנייה חד-ספרתית, IAM לאבטחה.**
- **Primary Key נקבע ביצירה ולא ניתן לשינוי.** Partition Key בלבד, או Partition + Sort.
- **גודל item מקסימלי 400 KB.** קובץ גדול → **S3** + pointer.
- **1 RCU = קריאה strongly consistent אחת של 4 KB/שנייה, או שתיים eventually consistent.**
- **1 WCU = כתיבה אחת של 1 KB/שנייה.** Transactional = **פי 2** בשני הכיוונים.
- **תמיד מעגלים כלפי מעלה** לגודל היחידה הבא.
- **LSI:** אותו PK, SK שונה · **רק ביצירת הטבלה** · capacity **של הטבלה** · **strong consistency** ·
  **10 GB לכל ערך PK** · עד 5.
- **GSI:** PK שונה · **בכל רגע** · **capacity משלו** · **eventually consistent בלבד** ·
  ללא מגבלת גודל · עד 20. **throttle ב-GSI חוסם כתיבות בטבלה.**
- **Query > Scan.** Scan קורא את כל הטבלה. חיפוש לפי attribute אחר → **GSI**.
- **hot partition** = partition key בעל cardinality נמוכה → throttling.
- **DAX:** cache מנוהל, **מיקרו-שניות**, **בלי שינוי קוד**, **TTL של 5 דקות**, **read בלבד**.
- **DAX = items ו-Query/Scan. ElastiCache = תוצאות אגרגציה.**
- **DynamoDB Streams: 24 שעות, מעט consumers.** **Kinesis Data Streams: שנה, הרבה consumers.**
- **Global Tables = active-active, read+write בכל Region. דורש Streams מופעל.**
- **TTL:** מחיקה אוטומטית לפי epoch timestamp · **eventual** (עד ~48 שעות) · **לא צורך WCU**.
- **DynamoDB + TTL יכול להחליף ElastiCache כ-session store.**
- **PITR: 35 יום, שחזור לכל נקודה. On-Demand Backups: עד מחיקה מפורשת.**
  **בשניהם השחזור יוצר טבלה חדשה.**
- **Export ל-S3 דורש PITR ולא צורך RCU. Import מ-S3 לא צורך WCU ויוצר טבלה חדשה.**
- **Serverless API הקלאסי: API Gateway → Lambda → DynamoDB.**
- **Provisioned = יציב וזול. On-Demand = לא צפוי ויקר יותר לבקשה.**

---

## 11. ✅ בדיקת הבנה

1. כמה RCU צריך ל-100 קריאות בשנייה של item בגודל 6 KB ב-eventually consistent?
2. הטבלה קיימת שנתיים וצריך אינדקס חדש לחיפוש לפי `email`. LSI או GSI, ולמה?
3. מה גורם ל-hot partition, ואיך מונעים?
4. צריך latency של מיקרו-שניות בלי לגעת בקוד האפליקציה. מה הפתרון?
5. מתי DynamoDB Streams לא מספיק וצריך Kinesis Data Streams?
6. מה תנאי הסף להפעלת Global Tables?
7. איך מאחסנים תמונת פרופיל של 3 MB לכל משתמש?
8. עומס בלתי צפוי לגמרי ב-launch. Provisioned עם Auto Scaling או On-Demand?
9. שיחזרתם מ-PITR. למה האפליקציה עדיין רואה את הנתונים הישנים?
10. GSI מוגדר עם WCU נמוך. מה קורה לכתיבות בטבלה הבסיסית?

<details>
<summary>תשובות</summary>

1. 6 KB מעוגל כלפי מעלה ל-**8 KB** = **2 יחידות של 4 KB**.
   Strongly consistent היה `100 × 2 = 200 RCU`. **Eventually consistent = חצי → 100 RCU.**
2. **GSI.** **LSI ניתן להגדרה רק בזמן יצירת הטבלה** ואי אפשר להוסיף אותו לטבלה קיימת.
   מחיר: ה-GSI יהיה **eventually consistent** ויצרוך capacity ואחסון נפרדים.
3. **Partition key בעל cardinality נמוכה או התפלגות לא אחידה** — למשל `status` עם שני ערכים.
   כל התעבורה נופלת על מעט partitions פיזיים → throttling.
   **מונעים** בבחירת מפתח בעל cardinality גבוהה (למשל `user_id`),
   ובמקרי קצה מוסיפים suffix אקראי (write sharding).
4. **DAX.** הוא cache מנוהל **תואם ל-API הקיים של DynamoDB**, ולכן לא דורש שינוי קוד.
   ElastiCache היה מחייב כתיבת לוגיקת cache מפורשת.
5. כשצריך **retention מעל 24 שעות** (Kinesis נותן **שנה**),
   כשיש **הרבה consumers** במקביל, או כשרוצים לצרוך דרך
   Kinesis Data Analytics / Firehose / Glue Streaming ETL.
6. ⚠️ **DynamoDB Streams חייב להיות מופעל** על הטבלה. זה תנאי מוקדם מפורש.
7. **ב-S3** — item ב-DynamoDB מוגבל ל-**400 KB**.
   ב-DynamoDB שומרים רק את ה-**S3 object key** כ-attribute.
8. **On-Demand.** אין נתוני עבר לבסס עליהם תחזית, ו-Auto Scaling ב-Provisioned **מגיב באיחור** —
   כלומר throttling בדיוק בשיא. אחרי שהעומס מתייצב אפשר לעבור ל-Provisioned ולחסוך.
9. כי **השחזור יוצר טבלה חדשה** ולא דורס את הקיימת.
   צריך להפנות את האפליקציה לטבלה החדשה (או לבצע rename מתוכנן).
10. הכתיבות ל-GSI נכשלות, ו-DynamoDB **תחסום (throttle) את הכתיבות לטבלה הבסיסית**
    כדי לשמור על עקביות בין השתיים. הפתרון: להקצות ל-GSI capacity מספיק
    ולצמצם את ה-projection שלו.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[24 - Database Selection]] · [[21 - RDS Fundamentals]] · [[22 - RDS Scaling and Availability]] · [[25 - Lambda]] · [[27 - API Gateway]] · [[29 - Event-Driven Architecture]] · [[38 - Serverless and Modern Architectures]] · [[34 - Disaster Recovery]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 8473–8729, 9350–9368
