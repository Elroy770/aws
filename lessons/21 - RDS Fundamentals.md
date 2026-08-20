---
lesson: 21
title: RDS Fundamentals
domain: Design Resilient Architectures
services: [RDS, Aurora, RDS Proxy, KMS, Secrets Manager]
tags: [saa-c03, databases, relational, managed-services]
---

# 21 — RDS Fundamentals

> [!abstract] בשורה אחת
> RDS הוא מסד נתונים רלציוני שבו AWS מנהלת את כל מה שמתחת ל-schema שלך — patching, גיבויים, failover ו-scaling — בתמורה לוויתור על גישת SSH לשרת.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה לא פשוט להתקין MySQL על EC2 |
| 2 | איך זה עובד | engines, ארכיטקטורה, storage, connectivity |
| 3 | Storage Auto Scaling | מה מפעיל אותו ומתי |
| 4 | גיבוי ושחזור | automated backups מול snapshots, PITR, Aurora Cloning |
| 5 | אבטחה | הצפנה, IAM auth, TLS, audit logs |
| 6 | RDS Proxy | למה Lambda ו-RDS לא מסתדרים בלעדיו |
| 7 | עלות | על מה בדיוק משלמים ואיפה מתחבאות ההוצאות |
| 8 | מלכודות | הניסוחים שמכריעים שאלות |

**מונחי מפתח בשיעור:** `PITR` · `Backup Window` · `Maintenance Window` · `DB Snapshot` · `RDS Proxy` · `RDS Custom`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- להתקין PostgreSQL על EC2 לוקח שעה. לתחזק אותו בייצור לוקח משרה.
- מי מריץ patch לגרסת ה-OS? מי מוודא שהגיבוי באמת עובד? מי מקים standby ב-AZ שני?
- שחזור לנקודת זמן ספציפית ("לפני שהסקריפט מחק את הטבלה ב-14:32") דורש ניהול transaction logs.
- כשהדיסק מתמלא ב-3 בלילה, מישהו צריך לקום ולהגדיל אותו.

### מה השירות פותר

- **provisioning אוטומטי** — DB עולה תוך דקות, מוגדר נכון מראש.
- **OS patching** — AWS מטפלת בעדכוני מערכת ההפעלה וה-engine במסגרת maintenance window.
- **continuous backups** עם יכולת **Point-in-Time Restore** לכל שנייה בטווח ה-retention.
- **Read Replicas** לשיפור ביצועי קריאה (מפורט ב-[[22 - RDS Scaling and Availability]]).
- **Multi-AZ** ל-DR ולזמינות גבוהה.
- **scaling** אנכי (instance גדול יותר) ואופקי (עוד replicas).
- **monitoring dashboards** מובנים.
- **מה מוותרים:** אין SSH ל-instance. אתה לא מנהל את השרת — AWS כן.

### ה-engines הנתמכים

- PostgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- IBM DB2
- **Aurora** — engine קנייני של AWS, תואם API ל-MySQL ול-PostgreSQL (מפורט ב-[[22 - RDS Scaling and Availability]])

> [!tip] האנלוגיה
> EC2 עם DB זה לשכור דירה ריקה: אתה אחראי לצנרת, לחשמל ולתיקונים.
> RDS זה מלון: החדר נקי, הכביסה מטופלת, אבל אסור לך לפרק את המזגן.

---

## 2. ⚙️ איך זה עובד

### 2.1 מה AWS מנהלת ומה אתה

| תחום | DB על EC2 (self-managed) | RDS (managed) |
|---|---|---|
| Provisioning של השרת | **אתה** | AWS |
| התקנת ה-DB engine | **אתה** | AWS |
| OS patching | **אתה** | AWS |
| DB engine patching | **אתה** | AWS (במסגרת maintenance window) |
| גיבויים אוטומטיים | **אתה** (cron + סקריפטים) | AWS |
| Point-in-Time Restore | **אתה** (ניהול transaction logs) | AWS, מובנה |
| הקמת standby ו-failover | **אתה** | AWS (Multi-AZ בקליק) |
| Read Replicas | **אתה** | AWS (עד 15) |
| הגדלת אחסון | **אתה** | AWS (Storage Auto Scaling) |
| Monitoring dashboards | **אתה** | AWS (CloudWatch, Performance Insights) |
| **Schema design** | אתה | **אתה** |
| **Indexes ו-query tuning** | אתה | **אתה** |
| **Connection pooling באפליקציה** | אתה | **אתה** (או RDS Proxy) |
| גישת SSH ל-OS | ✅ יש | ❌ **אין** |

> [!info] מתי בכל זאת DB על EC2
> כשצריך גישה למערכת ההפעלה, קונפיגורציה שאינה נתמכת, או engine ש-RDS לא מציע.
> אם רק Oracle או SQL Server דורשים גישה ל-OS — יש פתרון ביניים: **RDS Custom**.

### 2.2 הארכיטקטורה הבסיסית

```text
VPC
├── Public Subnets      → ALB / NAT
└── Private Subnets (DB Subnet Group, ≥2 AZs)
        │
        ├── RDS Primary  (AZ-a)  ← Security Group: 3306/5432 מה-SG של האפליקציה בלבד
        └── RDS Standby  (AZ-b)  ← Multi-AZ, לא נגיש לקריאה

Application ──► endpoint DNS של ה-RDS ──► Primary
```

- **DB Subnet Group** — אוסף subnets בלפחות שני AZs; חובה כדי לאפשר Multi-AZ.
- ה-DB יושב ב-**private subnets** ואינו public accessible בייצור.
- ה-Security Group מתיר את פורט ה-DB **רק** מה-Security Group של שכבת האפליקציה.
- האפליקציה מתחברת דרך **endpoint DNS** — לעולם לא דרך כתובת IP.

### 2.3 האחסון

- האחסון מגובה ב-**EBS** — כלומר כל מה שנלמד ב-[[19 - EBS and EC2 Storage]] תקף גם כאן.
- בוחרים סוג volume (gp3 לרוב, io1/io2 ל-IOPS גבוהים) וגודל.
- Aurora הוא היוצא מן הכלל: יש לו שכבת אחסון ייעודית משלו שאינה EBS רגיל.

---

## 3. 📈 RDS Storage Auto Scaling

התכונה שמונעת את השיחה בשלוש לפנות בוקר.

- כש-RDS מזהה שהאחסון הפנוי אוזל, הוא מגדיל את הקיבולת **אוטומטית**.
- חובה להגדיר **Maximum Storage Threshold** — גבול עליון שמעליו לא יגדל.
- נתמך בכל ה-engines של RDS.

### התנאים המדויקים להפעלה

שלושת התנאים חייבים להתקיים **יחד**:

| תנאי | הערך |
|---|---|
| אחסון פנוי | פחות מ-**10%** מהקיבולת שהוקצתה |
| משך המצב | לפחות **5 דקות** ברציפות |
| מאז השינוי הקודם | עברו לפחות **6 שעות** |

- Use case מובהק: אפליקציות עם עומס **בלתי צפוי** שקצב הצמיחה שלהן לא ידוע מראש.
- שים לב: ה-scaling הוא **רק כלפי מעלה**. אחסון ב-RDS לא מצטמצם אוטומטית.

---

## 4. 💾 גיבוי ושחזור

### 4.1 Automated Backups מול Manual Snapshots

| היבט | Automated Backups | Manual DB Snapshots |
|---|---|---|
| מי מפעיל | RDS, אוטומטית | אתה, ידנית |
| תדירות | גיבוי מלא יומי ב-backup window | מתי שתרצה |
| Transaction logs | מגובים **כל 5 דקות** | לא רלוונטי |
| Point-in-Time Restore | ✅ | ❌ (נקודה בודדת) |
| Retention | 1–35 יום; **0 = כיבוי** (ב-RDS) | **כמה שתרצה**, עד שתמחק |
| מה קורה כשמוחקים את ה-DB | נמחקים איתו | **שורדים** |
| Use case | תאונות יומיומיות, RPO קצר | archive, compliance, שכפול סביבה |

**מה זה נותן בפועל:** אפשר לשחזר לכל נקודת זמן — מהגיבוי הישן ביותר ועד **5 דקות אחורה**.

### 4.2 ההבדל של Aurora

| היבט | RDS | Aurora |
|---|---|---|
| טווח retention | 1–35 יום | 1–35 יום |
| ניתן לכבות | ✅ (retention = 0) | ❌ **לא ניתן לכבות** |
| PITR | ✅ | ✅ |
| Snapshots ידניים | ✅ | ✅ |

### 4.3 אפשרויות שחזור

- **שחזור תמיד יוצר מסד נתונים חדש.** אין "restore in place" מעל DB קיים.
  זו נקודה שנשאלת ישירות — אחרי restore צריך להפנות את האפליקציה ל-endpoint החדש.
- **שחזור MySQL RDS מ-S3:** מגבים DB מקומי, מעלים את הקובץ ל-S3, ומשחזרים ממנו ל-RDS MySQL חדש.
- **שחזור Aurora MySQL מ-S3:** אותו מסלול, אבל הגיבוי חייב להיווצר עם **Percona XtraBackup**.

### 4.4 Aurora Database Cloning

```text
Production Aurora Cluster
        │  clone (copy-on-write)
        ▼
Staging Aurora Cluster  ← בהתחלה משתמש באותו data volume בדיוק
                          כשמשנים דאטה, רק הבלוקים ששונו מועתקים
```

- יוצר cluster חדש מ-cluster קיים, **מהר יותר מ-snapshot & restore**.
- מבוסס **copy-on-write**: בהתחלה לא מועתק דבר, ולכן זה מיידי וזול.
- רק כשמבצעים עדכונים ב-cluster החדש מוקצה אחסון נוסף ומועתקים הבלוקים ששונו.
- Use case קלאסי: יצירת סביבת staging מתוך production **בלי להשפיע על ה-production**.

> [!warning] מלכודת החיוב על DB עצור
> RDS שנמצא במצב **stopped ממשיך לחייב על האחסון**.
> אם מתכננים לעצור לתקופה ארוכה — עושים snapshot, מוחקים את ה-instance, ומשחזרים כשצריך.

---

## 5. 🔐 אבטחה ב-RDS ו-Aurora

### הצפנה at rest

- מבוססת **KMS**, ו**חייבת להיות מוגדרת בזמן היצירה (launch time)**.
- אם ה-master אינו מוצפן — **ה-Read Replicas לא יכולים להיות מוצפנים**.
- להצפנת DB קיים שאינו מוצפן: snapshot → restore כ-encrypted → החלפת ה-endpoint.
  (בדיוק אותו דפוס כמו הצפנת EBS volume קיים.)

### הצפנה in flight

- **TLS זמין כברירת מחדל** בכל ה-engines.
- הלקוח משתמש ב-AWS TLS root certificates בצד האפליקציה.

### הרשאות וגישה

| מנגנון | מה הוא נותן |
|---|---|
| **IAM Authentication** | התחברות ל-DB עם IAM role במקום שם משתמש וסיסמה |
| **Secrets Manager** | אחסון וסבב אוטומטי של סיסמאות ה-DB |
| **Security Groups** | שליטה ברמת הרשת — מי בכלל יכול להגיע לפורט |
| **Audit Logs** | ניתן להפעיל ולשלוח ל-CloudWatch Logs ל-retention ארוך |

> [!warning] אין SSH
> ב-RDS **אין** גישת SSH ל-instance. החריג היחיד הוא **RDS Custom** (Oracle ו-SQL Server בלבד),
> שנועד בדיוק למקרים שבהם צריך לגעת ב-OS או להתקין agent.

---

## 6. 🔌 Amazon RDS Proxy

### הבעיה שהוא פותר

- כל חיבור פתוח ל-DB צורך CPU ו-RAM על ה-instance.
- Lambda פותחת חיבור חדש בכל invocation — 1,000 invocations מקבילות = 1,000 חיבורים.
- ה-DB קורס או מתחיל להחזיר timeouts, בלי שום קשר לעומס השאילתות עצמן.

### מה RDS Proxy עושה

- **pooling ושיתוף** של חיבורים בין כל הלקוחות — האפליקציה חושבת שיש לה חיבור פרטי.
- מוריד עומס CPU/RAM מה-DB וממזער חיבורים פתוחים ו-timeouts.
- **serverless, auto-scaling, זמין ב-Multi-AZ.**
- **מקצר את זמן ה-failover ב-עד 66%** — מספר שנשאל במבחן.
- אכיפת **IAM Authentication** ואחסון מאובטח של credentials ב-Secrets Manager.
- **אין צורך בשינויי קוד** ברוב האפליקציות — רק החלפת ה-endpoint.

### מגבלות ותמיכה

- נתמך ב-RDS: MySQL, PostgreSQL, MariaDB, SQL Server.
- נתמך ב-Aurora: MySQL ו-PostgreSQL.
- **RDS Proxy לעולם אינו נגיש פומבית** — הגישה אליו רק מתוך ה-VPC.

```text
VPC (private subnet)
Lambda × 1000 ──IAM auth──► RDS Proxy ──pool קטן──► RDS Instance
                                 │
                          Secrets Manager
```

---

## 7. 💰 עלות ותמחור

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **Instance hours** | לפי DB instance class ולפי שעה | הרכיב הגדול ביותר בדרך כלל |
| **Storage** | GB-month של אחסון **provisioned** | לא לפי מה שמלא בפועל |
| **IOPS / Throughput** | ב-io1/io2 לפי IOPS שהוקצו; ב-gp3 מעל הבסיס | ב-gp2 כלול בגודל |
| **Backups** | חינם עד לגודל ה-DB המוקצה; **מעבר לזה בתשלום** | retention ארוך = חיוב |
| **Snapshots ידניים** | GB-month | שורדים גם אחרי מחיקת ה-DB, וממשיכים לחייב |
| **Data transfer** | יציאה החוצה, ו-cross-Region | ראה סעיף העלויות הנסתרות |
| **Multi-AZ** | **מכפיל את עלות ה-instance ואת עלות האחסון** | משלמים על ה-standby במלואו |
| **Read Replicas** | כל replica מחויבת כ-instance נוסף מלא | |
| **RDS Proxy** | לפי vCPU של ה-DB instance לשעה | |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Single-AZ On-Demand | בסיס | dev/test בלבד |
| Reserved DB Instance (1–3 שנים) | הנחה משמעותית מול On-Demand | baseline יציב שרץ 24/7 |
| Multi-AZ | **פי ~2** מ-Single-AZ | ייצור — זו לא הוצאה אופציונלית |
| gp3 storage | הזול ב-SSD | ברירת המחדל |
| io1/io2 | יקר משמעותית | DB עם דרישת IOPS מוכחת |
| Aurora Serverless | לפי ACU בפועל | עומס לסירוגין או בלתי צפוי |

### 🚩 עלויות נסתרות

- **DB עצור עדיין מחויב על אחסון** — עצירה חוסכת רק את שעות ה-instance.
- **Backup storage מעבר לגודל ה-DB** — retention של 35 יום על DB עמוס מצטבר.
- **snapshots יתומים** — נשארים אחרי שה-DB נמחק ואף אחד לא זוכר אותם.
- **cross-Region data transfer** — Read Replica ב-Region אחר משלמת על כל בייט של רפליקציה.
- **Multi-AZ מכפיל גם את האחסון**, לא רק את ה-instance.
- **over-provisioning של IOPS** — קל להקצות io1 "ליתר ביטחון" ולשלם על ביצועים שלא נוצלים.
- **Performance Insights** מעבר לתקופת ה-retention החינמית.

### 💡 טיפים לחיסכון

- Reserved DB Instances לכל DB שרץ 24/7 בייצור.
- Single-AZ ב-dev/test, Multi-AZ רק בייצור.
- gp3 כברירת מחדל; לעבור ל-io1/io2 רק אחרי מדידה ב-CloudWatch.
- להוריד retention של automated backups לרמה שבאמת נדרשת ב-RPO.
- לסביבות שרצות רק בשעות עבודה: לעצור את ה-DB, ולזכור שהאחסון עדיין מחויב.
- ל-DB שיושב חודשים בלי שימוש: snapshot + מחיקה, במקום stop.
- Aurora Cloning ליצירת staging במקום לשלם על עותק מלא של ה-production.

---

## 8. ⚖️ השוואות מכריעות

### RDS מול DB על EC2

| קריטריון | RDS | DB על EC2 |
|---|---|---|
| מאמץ תפעולי | נמוך | גבוה |
| גישת OS / SSH | ❌ (למעט RDS Custom) | ✅ |
| PITR מובנה | ✅ | לבנות לבד |
| Multi-AZ failover | קליק | לבנות לבד |
| גמישות בקונפיגורציה | מוגבלת | מלאה |
| עלות ישירה | גבוהה יותר לשעה | נמוכה יותר לשעה |
| עלות כוללת (כולל אנשים) | נמוכה יותר בדרך כלל | גבוהה |

### Automated Backup מול Snapshot

| קריטריון | Automated Backup | Manual Snapshot |
|---|---|---|
| דיוק בזמן | כל שנייה בטווח (PITR) | הרגע שבו צולם |
| Retention מקסימלי | 35 יום | ללא הגבלה |
| שורד מחיקת DB | ❌ | ✅ |

> [!info] שורה תחתונה
> Automated Backups הם ה-RPO היומיומי; Manual Snapshots הם הארכיון ארוך הטווח.
> צריך את שניהם — הם לא מחליפים זה את זה.

---

## 9. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר בנושא הזה | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | ה-DB הוא הרכיב שהכי כואב כשמשהו משתבש בו | Performance Insights + Enhanced Monitoring; maintenance window בשעה שקטה; תרגול restore אמיתי |
| Security | מסד הנתונים מחזיק את הנכס היקר ביותר | הצפנת KMS מהיצירה, Secrets Manager עם rotation, private subnets, audit logs ל-CloudWatch |
| Reliability | גיבוי שלא ניסית לשחזר אינו גיבוי | Multi-AZ בייצור, retention שתואם ל-RPO, snapshots cross-Region ל-DR |
| Performance Efficiency | RDS מנהל את התשתית — ה-queries עדיין שלך | right-sizing של instance class, indexes, RDS Proxy במקום עוד CPU |
| Cost Optimization | Multi-AZ ו-replicas מכפילים את החשבון | Reserved Instances ל-baseline, gp3, מחיקת snapshots יתומים, Aurora Cloning ל-staging |
| Sustainability | DB שרץ סרק צורך משאבים אמיתיים | כיבוי סביבות dev בלילה, right-sizing, מחיקת replicas שאיש לא קורא מהן |

---

## 10. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "managed relational database", "SQL queries", "OLTP" | RDS |
| "restore to a specific point in time" | Automated Backups + PITR |
| "retain backups beyond 35 days", "compliance archive" | Manual DB Snapshot |
| "storage keeps filling up unexpectedly" | RDS Storage Auto Scaling |
| "too many database connections", "Lambda exhausts connections" | RDS Proxy |
| "reduce failover time" | RDS Proxy (עד 66% שיפור) |
| "create a staging copy of production quickly and cheaply" | Aurora Database Cloning |
| "encrypt an existing unencrypted database" | Snapshot → restore as encrypted |
| "connect without a database password" | IAM Database Authentication |
| "need OS-level access to Oracle/SQL Server" | RDS Custom |
| "database will be idle for six months, minimize cost" | Snapshot ואז מחיקת ה-instance |
| "no code changes allowed" | RDS Proxy |

### טעויות נפוצות

> [!warning] מלכודת 1 — "אעצור את ה-DB ולא אשלם"
> **הניסוח:** "The dev database is unused for months. Minimize cost."
> **הטעות:** לבחור "stop the RDS instance".
> **הנכון:** DB עצור עדיין מחויב על **אחסון**. הפתרון הזול הוא snapshot ואז מחיקת ה-instance.

> [!warning] מלכודת 2 — הצפנה בדיעבד
> **הניסוח:** "Enable encryption at rest on the existing production database."
> **הטעות:** לבחור "modify the DB instance and enable encryption".
> **הנכון:** ההצפנה נקבעת ב-launch time בלבד. חייבים snapshot → restore as encrypted.

> [!warning] מלכודת 3 — restore "מעל" ה-DB הקיים
> **הניסוח:** "Restore the database to yesterday at 09:00."
> **הטעות:** להניח שה-endpoint נשאר זהה.
> **הנכון:** כל restore יוצר **instance חדש** עם endpoint חדש. צריך להפנות את האפליקציה אליו.

> [!warning] מלכודת 4 — Read Replica מוצפנת מ-master לא מוצפן
> **הניסוח:** "Add an encrypted read replica to the existing database."
> **הטעות:** להניח שאפשר להצפין רק את ה-replica.
> **הנכון:** אם ה-master אינו מוצפן, ה-replicas לא יכולים להיות מוצפנים. קודם מצפינים את ה-master.

> [!warning] מלכודת 5 — עוד CPU במקום Proxy
> **הניסוח:** "Serverless application exhausts database connections during traffic spikes."
> **הטעות:** להציע instance class גדול יותר.
> **הנכון:** הבעיה היא **מספר החיבורים**, לא כוח העיבוד. התשובה היא RDS Proxy.

---

## 11. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** מערכת הזמנות פנימית של חברת ביטוח. PostgreSQL, נתונים רגישים תחת רגולציה.
RPO של 5 דקות, RTO של דקות בודדות. הצוות רוצה סביבת staging עם דאטה אמיתית מדי שבוע.
ה-backend עובר ל-Lambda ומייצר ספייקים של מאות invocations מקבילות.

```text
VPC
├── Private Subnet A ── Aurora PostgreSQL Writer
├── Private Subnet B ── Aurora PostgreSQL Reader (Multi-AZ)
│
├── RDS Proxy (private) ◄── Lambda × N  [IAM auth]
│         │
│   Secrets Manager (rotation אוטומטי)
│
└── Backups: automated 35 יום + snapshots ידניים ל-archive
    Staging: Aurora Clone שבועי (copy-on-write)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Aurora PostgreSQL ולא RDS PostgreSQL | ביצועים ו-HA טובים יותר; ה-API זהה כך שהקוד לא משתנה |
| Multi-AZ | RTO של דקות בודדות דורש standby מוכן; זו לא הוצאה אופציונלית בייצור |
| Automated backups עם PITR | ה-transaction logs מגובים כל 5 דקות — עומד בדיוק ב-RPO הנדרש |
| Snapshots ידניים תקופתיים | automated backups מוגבלים ל-35 יום; הרגולציה דורשת יותר |
| הצפנת KMS **מהיצירה** | לא ניתן להצפין בדיעבד; חייבים להחליט בהקמה |
| RDS Proxy | Lambda מייצרת מאות חיבורים; ה-Proxy מאחד אותם ומקצר failover ב-עד 66% |
| Secrets Manager עם rotation | אין סיסמאות בקוד; ה-Proxy אוכף IAM auth |
| Aurora Cloning ל-staging | copy-on-write — מיידי, זול, ולא נוגע ב-production |
| Private subnets + SG ייעודי | ה-DB וה-Proxy אינם נגישים מהאינטרנט כלל |

**למה לא DB על EC2?** נדרש PITR, failover אוטומטי וניהול patching — כל אלה יהיו פרויקט תחזוקה שלם.

**למה לא snapshot & restore ל-staging?** איטי יקר ומייצר עותק מלא של האחסון. Cloning נותן את אותה תוצאה מיידית.

**למה לא להגדיל את ה-instance במקום Proxy?** מיצוי חיבורים אינו בעיית CPU; instance גדול יותר היה מייקר בלי לפתור.

---

## 12. 🚫 מה לא צריך לדעת למבחן

- גרסאות engine ספציפיות ומתי כל אחת מגיעה ל-end of support.
- פרמטרי `parameter group` ו-`option group` ברמת הפרט.
- תחביר SQL, tuning של queries או תכנון indexes.
- הבדלי רישוי בין Oracle SE ל-EE ומודלי BYOL.
- הפרטים הפנימיים של Percona XtraBackup — מספיק לדעת שזה הכלי לייבוא Aurora MySQL מ-S3.
- מדדי Performance Insights ברמת המטריקה הבודדת.

---

## 13. ⚡ Cheat Sheet

- RDS מנהל: provisioning, OS patching, backups, PITR, Multi-AZ, replicas, monitoring. **אין SSH.**
- Engines: PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, Aurora.
- **RDS Custom** = היחיד שנותן גישת OS, ורק ל-Oracle ו-SQL Server.
- האחסון מגובה **EBS**; Aurora הוא היוצא מן הכלל עם שכבת אחסון משלו.
- Storage Auto Scaling: פנוי מתחת ל-**10%**, במשך **5 דקות**, ו-**6 שעות** מאז השינוי האחרון.
- Automated Backups: full יומי + transaction logs כל **5 דקות** → PITR עד 5 דקות אחורה.
- Retention: **1–35 יום**. ב-RDS אפשר לכבות (0); **ב-Aurora לא ניתן לכבות**.
- Manual Snapshots: retention ללא הגבלה, ושורדים את מחיקת ה-DB.
- **כל restore יוצר DB חדש** עם endpoint חדש.
- Aurora Cloning: copy-on-write, מהיר וזול מ-snapshot & restore, אידיאלי ל-staging.
- הצפנה at rest נקבעת **בזמן היצירה בלבד**; master לא מוצפן ⇒ replicas לא מוצפנות.
- הצפנת DB קיים = snapshot → restore as encrypted.
- TLS in flight זמין כברירת מחדל בכל ה-engines.
- IAM Database Authentication מחליף סיסמאות; Secrets Manager מסובב אותן.
- **RDS Proxy**: connection pooling, serverless, Multi-AZ, מקצר failover ב-עד **66%**, אף פעם לא public.
- **DB עצור ממשיך לחייב על אחסון** — לתקופה ארוכה עדיף snapshot + מחיקה.
- Multi-AZ מכפיל את עלות ה-instance ואת עלות האחסון.

---

## 14. ✅ בדיקת הבנה

1. ה-DB של סביבת ה-QA לא בשימוש ל-8 חודשים. ה-CFO רוצה עלות אפסית. מה עושים ולמה לא פשוט stop?
2. מה בדיוק צריך לקרות כדי ש-RDS Storage Auto Scaling יפעל?
3. הרגולטור דורש לשמור גיבויים 7 שנים. Automated Backups עונים על זה?
4. אפליקציית Lambda מקבלת שגיאות "too many connections" בשעות שיא. מה הפתרון, ומה **לא** הפתרון?
5. ה-DB בייצור לא מוצפן והביקורת דורשת הצפנה. מה המסלול?

<details>
<summary>תשובות</summary>

1. **snapshot ידני ואז מחיקת ה-DB instance.** עצירה (stop) מפסיקה רק את חיוב שעות ה-instance — **האחסון ממשיך להיות מחויב במלואו**. ה-snapshot שורד את מחיקת ה-instance וממנו משחזרים כשצריך.
2. שלושת התנאים יחד: אחסון פנוי מתחת ל-**10%** מהמוקצה, המצב נמשך לפחות **5 דקות**, ועברו לפחות **6 שעות** מאז שינוי האחסון הקודם. בנוסף חייב להיות מוגדר Maximum Storage Threshold.
3. לא. Automated Backups מוגבלים ל-**35 יום** לכל היותר. ל-7 שנים צריך **Manual DB Snapshots** (retention ללא הגבלה), רצוי עם copy ל-Region נוסף.
4. הפתרון הוא **RDS Proxy** — הוא מאגד ומשתף חיבורים כך ש-1,000 invocations לא מייצרות 1,000 חיבורים. מה שלא יעזור: הגדלת ה-instance class. הבעיה היא מספר החיבורים, לא כוח העיבוד.
5. אין הצפנה במקום. יוצרים **snapshot**, מבצעים restore ממנו כ-**encrypted** עם מפתח KMS, ואז מפנים את האפליקציה ל-endpoint החדש. שים לב שההצפנה חייבת להיקבע ב-launch time, ולכן זו הדרך היחידה.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[22 - RDS Scaling and Availability]] · [[24 - Database Selection]] · [[19 - EBS and EC2 Storage]] · [[35 - Backup and Data Protection]] · [[32 - Security Services]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 2570–2626, 3020–3125, 9303–9335
