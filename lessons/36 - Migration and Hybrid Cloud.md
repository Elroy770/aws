---
lesson: 36
title: Migration and Hybrid Cloud
domain: Design Resilient Architectures
services: [Snow Family, Storage Gateway, DataSync, Transfer Family, DMS, SCT, MGN, Application Discovery Service, Outposts, VMware Cloud on AWS]
tags: [saa-c03, migration, hybrid, storage-gateway, dms, snowball]
---

# 36 — Migration and Hybrid Cloud

> [!abstract] בשורה אחת
> כל שאלת migration במבחן נפתרת בשתי שאלות: **מה מעבירים** (שרת / DB / קבצים) ו-**כמה זמן ורוחב פס יש** —
> ומהצלב הזה נגזר הכלי היחיד הנכון.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה לא פשוט "מעתיקים הכל ל-S3" |
| 2 | איך זה עובד | 6 Rs · Snow Family · Storage Gateway · DataSync · Transfer Family · DMS/SCT · MGN |
| 3 | פירוק מפורט | **טבלת 6 Rs** · **טבלת "כמה דאטה וכמה זמן"** · **3 סוגי Storage Gateway** · DMS מול SCT |
| 4 | עלות | על מה משלמים בכל כלי migration ואיפה מסתתרות ההפתעות |
| 5 | השוואות | DataSync מול Snowball מול Transfer Family · MGN מול DMS · Outposts מול Storage Gateway |
| 6 | Well-Architected | איך מריצים migration בלי לשבור את הפרודקשן |
| 7 | מלכודות | Snowball לא נכנס ל-Glacier · DMS הוא לא ממיר schema |
| 8 | Scenario | מעבר Data Center שלם: 200 שרתים + 300TB + Oracle → Aurora |

**מונחי מפתח בשיעור:** `Rehost` · `CDC` · `Full Load` · `Cutover` · `VTL` · `Homogeneous / Heterogeneous` · `Edge Computing`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- Data Center קיים עולה כסף גם כשהוא ריק — חוזה שכירות, חשמל, חומרה שמתיישנת.
- העברת 200TB דרך קו של 100 Mbps לוקחת **חודשים**, לא ימים.
- אפליקציית legacy שרצה על Oracle לא "עוברת" ל-Aurora בלחיצת כפתור — ה-schema שונה.
- אי אפשר לעצור מערכת פרודקשן לשבוע כדי להעתיק דאטה.
- לפעמים **אסור** להעביר הכל: רגולציה, data residency, או latency לציוד תעשייתי מקומי.

### מה השירות פותר

AWS לא נותנת כלי אחד. היא נותנת **ארגז כלים**, וכל כלי פותר מגבלה אחרת:

| המגבלה | הכלי |
|---|---|
| אין מספיק רוחב פס | **Snow Family** — מעבירים דיסקים פיזית |
| צריך להעביר קבצים באופן שוטף | **DataSync** — סנכרון מתוזמן דרך הרשת |
| האפליקציה המקומית חייבת להישאר, אבל הדאטה בענן | **Storage Gateway** — גשר NFS/SMB/iSCSI |
| שותפים חיצוניים מעלים קבצים ב-FTP | **Transfer Family** — SFTP/FTPS מנוהל מעל S3/EFS |
| צריך להעביר DB חי בלי downtime ארוך | **DMS** עם CDC |
| ה-DB עובר לסוג engine אחר | **SCT** להמרת schema, ואז DMS לדאטה |
| צריך להעביר שרתים שלמים כמו שהם | **MGN** — lift-and-shift ברפליקציה רציפה |
| אי אפשר לעזוב את ה-Data Center בכלל | **Outposts** — AWS פיזית אצלכם |

> [!tip] האנלוגיה
> Migration הוא מעבר דירה. **DataSync** הוא לשלוח קרטונים בדואר כל יום.
> **Snowball** הוא לשכור משאית כי יש יותר מדי דברים. **Storage Gateway** הוא מחסן חיצוני
> עם מפתח בבית. ו-**Outposts** הוא להביא את הרהיטים של הבית החדש לבית הישן.

---

## 2. ⚙️ איך זה עובד

### 2.1 ששת ה-Rs — אסטרטגיות המעבר

לפני שבוחרים כלי, בוחרים **אסטרטגיה**. זו שאלה עסקית, לא טכנית.

```text
כמה משנים את האפליקציה?
  לא נוגעים בכלל ─────────────► Retain   (משאירים on-prem)
  מכבים ──────────────────────► Retire   (מוחקים)
  מעתיקים כמו שזה ────────────► Rehost   (lift-and-shift)
  שינוי קטן בפלטפורמה ────────► Replatform
  קונים מוצר מדף במקום ───────► Repurchase
  כותבים מחדש cloud-native ───► Refactor / Re-architect
```

### 2.2 Snow Family — כשהרשת היא צוואר הבקבוק

- מכשירים פיזיים, מוקשחים ומוצפנים, ש-AWS שולחת אליכם בדואר.
- ממלאים אותם בדאטה, שולחים חזרה, ו-AWS מייבאת ל-S3.
- **כלל האצבע של הקורס:** אם ההעברה דרך הרשת לוקחת **יותר משבוע** — עוברים ל-Snow.
- Snowball Edge קיים בשתי גרסאות: **Storage Optimized** (הרבה אחסון) ו-**Compute Optimized** (הרבה כוח עיבוד).

**Edge Computing** — שימוש נוסף שלא קשור להעברה:

- מעבדים דאטה **במקום שבו הוא נוצר**: משאית בכביש, ספינה בים, מכרה תת-קרקעי.
- למקומות האלה אין אינטרנט טוב ואין כוח מחשוב.
- על Snowball Edge אפשר להריץ **EC2 instances ו-Lambda functions** מקומית.
- שימושים: עיבוד מקדים של דאטה, machine learning בשטח, transcoding של מדיה.

### 2.3 Storage Gateway — הגשר ההיברידי

הבעיה: **S3 הוא טכנולוגיה קניינית**. הוא לא NFS ולא SMB.
שרת on-prem לא יכול פשוט לעשות `mount` ל-bucket.

Storage Gateway הוא VM שמתקינים ב-Data Center (VMware / Hyper-V / KVM), והוא:

- מציג ל-Data Center פרוטוקול מוכר — **NFS, SMB או iSCSI**.
- מאחורי הקלעים כותב ל-**S3, EBS Snapshots או Glacier**.
- שומר **cache מקומי** של הדאטה שנקרא לאחרונה — כדי שהגישה תישאר מהירה.
- התעבורה מוצפנת ועוברת ב-Internet או ב-Direct Connect.

```text
Corporate Data Center                  AWS Cloud
─────────────────────                  ─────────
Application Server
      │ NFS / SMB / iSCSI
      ▼
Storage Gateway VM ──── HTTPS ────►  S3 / EBS Snapshots / Glacier
   (local cache)
```

### 2.4 Transfer Family — FTP מנוהל מעל S3

- שירות מנוהל לחלוטין להעברת קבצים לתוך **S3 או EFS** בפרוטוקולי FTP.
- שלושה פרוטוקולים: **FTP**, **FTPS** (מוצפן ב-SSL), **SFTP** (מוצפן ב-SSH).
- **FTP הרגיל נתמך רק בתוך VPC** — לא נחשף לאינטרנט, כי הוא לא מוצפן.
- מנוהל, מתרחב, זמין ב-Multi-AZ.
- **אימות משתמשים:** מנהל credentials בעצמו, או משתלב עם Active Directory, LDAP, Okta, Cognito או פתרון מותאם.
- אפשר להצמיד DNS ידידותי דרך Route 53.
- שימושים: שיתוף קבצים עם שותפים, datasets ציבוריים, feeds של CRM/ERP.

### 2.5 DataSync — סנכרון דאטה בקנה מידה

- מעביר כמויות גדולות של דאטה **דרך הרשת**, לא פיזית.
- **מ-on-premises או מענן אחר ל-AWS** — דורש התקנת **agent** (NFS, SMB, HDFS, S3 API).
- **בין שירותי AWS לבין עצמם** — S3↔EFS↔FSx — **לא צריך agent**.
- יעדים נתמכים: **S3 בכל storage class כולל Glacier**, EFS, ו-FSx על כל גרסאותיו.
- משימות רפליקציה מתוזמנות: **שעתי, יומי, שבועי**. זה לא סנכרון רציף.
- **משמר הרשאות קבצים ו-metadata** — POSIX ב-NFS, ACLs ב-SMB. זו התכונה שמפילה חלופות.
- agent אחד יכול לנצל עד **10 Gbps**, וניתן להגדיר תקרת רוחב פס.

### 2.6 DMS — Database Migration Service

- מעביר databases ל-AWS במהירות ובאבטחה, עם התאוששות עצמית.
- **ה-DB המקורי נשאר זמין לאורך כל ההעברה.** זו הנקודה המרכזית.
- שני מצבי מיגרציה:
  - **Homogeneous** — אותו engine. לדוגמה Oracle → Oracle.
  - **Heterogeneous** — engine שונה. לדוגמה SQL Server → Aurora.
- **Continuous Data Replication (CDC)** — אחרי ה-Full Load, DMS ממשיך לשדר שינויים
  בזמן אמת, כך שה-cutover הוא של דקות ולא של ימים.
- **חייבים להקים EC2 instance** שמריץ את משימות הרפליקציה (Replication Instance).

### 2.7 SCT — Schema Conversion Tool

- ממיר **schema** מ-engine אחד לאחר.
- OLTP לדוגמה: SQL Server או Oracle → MySQL, PostgreSQL, Aurora.
- OLAP לדוגמה: Teradata או Oracle → Redshift.
- מומלץ להריץ אותו על instance **compute-intensive** — ההמרה עתירת CPU.
- **הכלל שמפילים עליו במבחן:** אם ה-engine **לא משתנה** — **לא צריך SCT בכלל**.
  On-Premise PostgreSQL → RDS PostgreSQL זה עדיין PostgreSQL. RDS הוא הפלטפורמה, לא ה-engine.

### 2.8 MGN — Application Migration Service

- ה-"אבולוציה" של CloudEndure Migration; מחליף את Server Migration Service (SMS) הישן.
- פתרון **rehost / lift-and-shift** לשרתים.
- ממיר שרתים פיזיים, וירטואליים או מענן אחר כך שירוצו **natively** על AWS.
- מתקינים **AWS Replication Agent** על השרת המקורי, והוא משכפל ברמת הבלוקים ל-**staging area**
  שמורכב מ-EC2 ו-EBS זולים.
- כשמוכנים — מבצעים **cutover** ל-instances היעד.
- תומך במגוון רחב של פלטפורמות, מערכות הפעלה ו-databases. downtime מינימלי.

---

## 3. 🔍 פירוק מפורט

### 3.1 טבלת ששת ה-Rs — מלאה

| אסטרטגיה | מה עושים בפועל | דוגמה | מאמץ יחסי | מתי בוחרים |
|---|---|---|---|---|
| **Rehost** | מעתיקים את השרת כמו שהוא ל-EC2 | 200 VMs → EC2 עם MGN | **נמוך מאוד** | Deadline לחוץ, יציאה דחופה מ-DC |
| **Replatform** | שינוי קטן בפלטפורמה, בלי שינוי קוד | MySQL על EC2 → **RDS MySQL** | **נמוך-בינוני** | רוצים להיפטר מ-patching ו-backups ידניים |
| **Repurchase** | זורקים את המוצר וקונים SaaS | CRM עצמי → Salesforce | **בינוני** | המוצר הוא commodity, אין לו ערך ייחודי |
| **Refactor / Re-architect** | כותבים מחדש cloud-native | Monolith → Lambda + DynamoDB | **גבוה מאוד** | צריך scale, elasticity או מהירות פיתוח שה-legacy לא נותן |
| **Retire** | פשוט מכבים | 30% מהשרתים ב-DC לא בשימוש | **אפסי** | Discovery גילה שאף אחד לא משתמש |
| **Retain** | משאירים on-premises לעת עתה | מערכת mainframe עם רגולציה | **אפסי** | Dependency לא פתירה, או שהמעבר לא משתלם |

> [!info] שורה תחתונה
> **Retire ו-Retain הן ההחלטות הזולות ביותר** — הן חוסכות את כל הפרויקט.
> לכן שלב ה-Discovery בא **לפני** בחירת הכלים.
> יש גם ניסוח של **7 Rs** שמוסיף **Relocate** (העברת אשכול VMware שלם) — אותו רעיון.

### 3.2 טבלת "כמה דאטה, כמה זמן" — בחירת אמצעי ההעברה

זמן ההעברה נטו דרך הרשת, לפני overhead:

| נפח | 100 Mbps | 1 Gbps | 10 Gbps |
|---|---|---|---|
| **10 TB** | ~12 ימים | ~30 שעות | ~3 שעות |
| **100 TB** | ~124 ימים | ~12 ימים | ~30 שעות |
| **1 PB** | **~3 שנים** | ~124 ימים | ~12 ימים |

**החישוב הידני שכדאי לדעת:** 200TB על קו 100 Mbps ≈ 185 ימים. אותם 200TB על Direct Connect
של 1 Gbps ≈ 18.5 ימים — **אבל ה-provisioning של DX עצמו לוקח יותר מחודש**.
עם Snowball — **כשבוע מקצה לקצה**.

**טבלת ההחלטה:**

| הצורך | הכלי | למה |
|---|---|---|
| נפח קטן, edge, מקום מרוחק | **Snowcone** | המכשיר הקטן והנייד ביותר; גם ל-edge computing |
| עשרות עד מאות TB, חד-פעמי | **Snowball Edge** | Storage Optimized להעברה, Compute Optimized לעיבוד בשטח |
| Exabyte-scale, Data Center שלם | **Snowmobile** | מכולה על משאית; רק ל-scale שאין לו פתרון אחר |
| העברה שוטפת של קבצים דרך הרשת | **DataSync** | מתוזמן, משמר metadata, agent אחד עד 10 Gbps |
| חיבור קבוע, throughput צפוי | **Direct Connect** | private, יציב — אבל setup של חודש+ |
| האפליקציה המקומית נשארת, האחסון בענן | **Storage Gateway** | NFS/SMB/iSCSI עם cache מקומי |
| שותפים חיצוניים מעלים ב-FTP | **Transfer Family** | endpoint מנוהל, אימות מול AD/LDAP |
| רפליקציה מתמשכת של DB | **DMS** | Full Load + CDC |

> [!warning] הכלל שנשאל
> **"אם ההעברה דרך הרשת לוקחת יותר משבוע — Snowball."**
> זו שורת ההכרעה שהקורס מדגיש, וזו התשובה בשאלות עם "limited bandwidth" או "shared connection".

### 3.3 שלושת סוגי Storage Gateway

| | **S3 File Gateway** | **Volume Gateway** | **Tape Gateway** |
|---|---|---|---|
| **פרוטוקול ללקוח** | **NFS / SMB** | **iSCSI** (block) | **iSCSI VTL** |
| **סוג אחסון** | קבצים | בלוקים (volumes) | ספריית קלטות וירטואלית |
| **Backend ב-AWS** | S3 | S3 + **EBS Snapshots** | S3 + **Glacier / Deep Archive** |
| **Storage Classes** | Standard, Standard-IA, One Zone-IA, Intelligent-Tiering | S3 — **ללא Glacier** | S3 לקלטות פעילות, Glacier לארכיון |
| **מעבר ל-Glacier** | דרך **Lifecycle Policy** בלבד | לא | **מובנה** — קלטת שנשלפת עוברת לארכיון |
| **Cache מקומי** | הדאטה שנקרא לאחרונה | ראו שני המצבים למטה | כן |
| **אימות** | **IAM Role** לכל gateway; SMB משתלב עם **Active Directory** | IAM | IAM |
| **מתי בוחרים** | שרת אפליקציה צריך file share עם backend של S3 | צריך volumes לשרת + שחזור מ-snapshot | יש תהליך גיבוי לקלטות שלא רוצים לשנות |

**שני מצבי ה-Volume Gateway — הבדל שנשאל:**

| מצב | איפה יושב ה-dataset המלא | מה בענן | מתי |
|---|---|---|---|
| **Cached volumes** | **בענן** (S3) | הכל; מקומית רק cache של הדאטה החם | חוסכים אחסון מקומי, שומרים latency נמוך לדאטה החם |
| **Stored volumes** | **on-premises** (מלא) | גיבויים מתוזמנים כ-EBS Snapshots | צריכים גישה מקומית מלאה + DR בענן |

> [!tip] הזיכרון המהיר
> **File = NFS/SMB. Volume = iSCSI. Tape = VTL.**
> ואם השאלה מזכירה **"existing backup software"** או **"physical tapes"** — התשובה היא **Tape Gateway**, תמיד.

### 3.4 DMS מול SCT — מתי צריך המרת schema

| | **Homogeneous Migration** | **Heterogeneous Migration** |
|---|---|---|
| הגדרה | אותו DB engine במקור וביעד | engine שונה |
| דוגמאות | On-Prem PostgreSQL → RDS PostgreSQL · Oracle → Oracle on EC2 · RDS MySQL → Aurora MySQL | SQL Server → Aurora PostgreSQL · Oracle → MySQL · Teradata → Redshift |
| **צריך SCT?** | **לא** | **כן** |
| מה עושים | **DMS בלבד** — Full Load + CDC | **SCT** להמרת schema, stored procedures ו-views → ואז **DMS** לדאטה |
| הסיכון העיקרי | נמוך; בעיקר תזמון cutover | גבוה; קוד שלא ניתן להמרה אוטומטית דורש כתיבה ידנית |

**המקורות והיעדים של DMS — מה שכדאי לזכור:**

| Sources | Targets |
|---|---|
| DBs ב-on-prem וב-EC2: Oracle, SQL Server, MySQL, MariaDB, PostgreSQL, MongoDB, SAP, DB2 | אותם DBs ב-on-prem וב-EC2 |
| Azure SQL Database | **Amazon RDS** (כולל Aurora) |
| **Amazon RDS** — כולל Aurora | **Redshift, DynamoDB, S3** |
| **Amazon S3** | **OpenSearch, Kinesis Data Streams, Apache Kafka** |
| DocumentDB | DocumentDB, Neptune, Redis |

- **הנקודה החשובה:** ה-Targets רחבים בהרבה מ-Sources. DMS הוא לא רק "DB ל-DB" —
  הוא יכול להזרים לתוך **Kinesis או Kafka** ולשמש כ-pipeline לאירועים.
- הכיוונים הנתמכים: **on-prem → AWS**, **AWS → AWS**, וגם **AWS → on-prem**.

### 3.5 DMS Multi-AZ

- כשמפעילים Multi-AZ, DMS מקים ומתחזק **standby סינכרוני ב-AZ אחרת**.
- היתרונות: **יתירות דאטה**, **ביטול I/O freezes**, **צמצום קפיצות latency**.
- שימושי כשה-CDC רץ שבועות ואסור שייפול באמצע.

### 3.6 מסלולי מיגרציה ייעודיים ל-RDS ו-Aurora

זה הנושא שבו נופלים כי בוחרים DMS כשיש דרך פשוטה יותר.

**MySQL:**

| מ- | ל- | האפשרויות |
|---|---|---|
| **RDS MySQL** | **Aurora MySQL** | (1) **DB Snapshot** של RDS ושחזור כ-Aurora. (2) יצירת **Aurora Read Replica** מה-RDS, וכשה-lag מגיע ל-0 — **promote** לאשכול עצמאי (לוקח זמן ועולה כסף) |
| **MySQL חיצוני** | **Aurora MySQL** | (1) **Percona XtraBackup** → קובץ ב-S3 → יצירת Aurora מה-S3 (**המהיר**). (2) יצירת Aurora ואז **mysqldump** (**איטי יותר**) |
| שני ה-DBs חיים ופעילים | | **DMS** |

**PostgreSQL:**

| מ- | ל- | האפשרויות |
|---|---|---|
| **RDS PostgreSQL** | **Aurora PostgreSQL** | (1) **DB Snapshot** ושחזור כ-Aurora. (2) **Aurora Read Replica** ואז promote כשה-lag הוא 0 |
| **PostgreSQL חיצוני** | **Aurora PostgreSQL** | גיבוי ל-**S3** וייבוא באמצעות ה-extension **`aws_s3`** |
| שני ה-DBs חיים ופעילים | | **DMS** |

> [!tip] הכלל שחוזר בשני המקרים
> **DB כבוי או snapshot → מסלול Snapshot/S3 (זול ופשוט).**
> **שני ה-DBs חיים וצריך downtime מינימלי → DMS.**
> **Read Replica + promote → כשרוצים lag אפס אבל מוכנים לשלם על replica במקביל.**

### 3.7 אסטרטגיית on-premises — כלים משלימים

| כלי | מה עושה | מתי |
|---|---|---|
| **Amazon Linux 2 AMI כ-VM** | הורדה בפורמט `.iso` ל-VMware, KVM, VirtualBox, Hyper-V | לפתח מקומית על אותה מערכת הפעלה כמו בענן |
| **VM Import / Export** | מייבא VMs קיימים ל-EC2, **וגם מייצא חזרה** | DR repository ל-VMs מקומיים; יציאה מ-lock-in |
| **Application Discovery Service** | אוסף מידע על ה-DC לפני ההעברה | **תמיד בשלב הראשון** |
| **Migration Hub** | לוח מחוונים שמרכז את התוצאות ומעקב אחר ההתקדמות | ניהול הפרויקט |
| **DMS** | רפליקציית DB לשני הכיוונים | DBs |
| **MGN** | רפליקציה אינקרמנטלית של שרתים חיים | שרתים |

**שני מצבי ה-Application Discovery Service:**

| מצב | מה נאסף | מתי בוחרים |
|---|---|---|
| **Agentless Discovery** (Agentless Discovery Connector) | מלאי VMs, קונפיגורציה, והיסטוריית ביצועים — CPU, זיכרון, דיסק | סביבת VMware; אסור/לא מעשי להתקין agent על כל שרת |
| **Agent-based Discovery** (Application Discovery Agent) | קונפיגורציית מערכת, ביצועים, **תהליכים רצים**, ו-**מפת חיבורי הרשת בין שרתים** | צריך **dependency mapping** מדויק לפני חלוקה ל-waves |

> [!warning] מה שקובע את בחירת המצב
> אם השאלה מדגישה **"dependency mapping"** או **"which servers talk to which"** —
> התשובה היא **Agent-based**. Agentless נותן מלאי וביצועים, לא מפת תלויות.

### 3.8 VMware Cloud on AWS

- לקוחות שמנהלים את ה-DC שלהם ב-VMware רוצים להרחיב קיבולת ל-AWS **בלי להחליף כלים**.
- VMware Cloud on AWS מריץ סביבת **vSphere** על תשתית AWS.
- שימושים: העברת workloads מבוססי vSphere, הרצת פרודקשן בסביבה היברידית, ואסטרטגיית **DR**.
- מתחבר לשירותי AWS — EC2, S3, FSx, RDS, Redshift — דרך Direct Connect.
- **מילת המפתח:** אם בשאלה כתוב **"keep using VMware / vSphere"** — זו התשובה.

### 3.9 AWS Outposts

- **מדפי שרתים פיזיים** של AWS שמותקנים ב-Data Center שלכם.
- מריצים את **אותה תשתית, אותם שירותים, אותם APIs ואותם כלים** כמו בענן.
- **AWS מקימה ומתחזקת** את ה-Racks; **אתם אחראים לאבטחה הפיזית** של המדף.
- פותר את בעיית ה"שתי מערכות" — במקום console אחד לענן ותהליך אחר ל-on-prem, יש אחד.

**היתרונות:**

- **Latency נמוך** לגישה למערכות מקומיות.
- **עיבוד דאטה מקומי** בלי לצאת החוצה.
- **Data residency** — הדאטה לא עוזב את המתקן. זו התשובה לרגולציה.
- מעבר קל יותר לענן בהמשך.
- שירות מנוהל לחלוטין.

**שירותים שרצים על Outposts:** EC2, EBS, S3, EKS, ECS, RDS, EMR.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| שירות | רכיב חיוב | הערה |
|---|---|---|
| **Snow Family** | לפי **job** + ימי החזקה מעבר למכסה + משלוח | **Data transfer IN חינם** — זו כל הפואנטה |
| **DataSync** | **לפי GB שהועבר** | פשוט וצפוי; אין חיוב שעתי |
| **Storage Gateway** | לפי **GB שנכתב ל-AWS** + עלות ה-S3/EBS מאחור + requests | ה-VM המקומי רץ על החומרה שלכם |
| **Transfer Family** | **לפי שעת endpoint מוקצה** + GB שהועבר | **Endpoint שרץ ולא בשימוש עדיין מחויב** |
| **DMS** | לפי **שעות ה-Replication Instance** + אחסון הלוגים | ה-instance רץ עד שמכבים אותו ידנית |
| **SCT** | הכלי עצמו **חינם** | משלמים רק על ה-instance שמריץ אותו |
| **MGN** | **חינם ל-90 יום** לכל שרת; משלמים על ה-**staging** (EC2 + EBS) | ה-staging הוא ההוצאה האמיתית |
| **Outposts** | מחויב כמו **חוזה חומרה** — Upfront + תשלום חודשי לתקופה | ההפך מ-on-demand; התחייבות ארוכה |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **Retire** — מכבים שרת מיותר | **שלילית** — חוסכת | תמיד. הרווח הכי מהיר בכל migration |
| **Snowball** ל-100TB+ | נמוכה יחסית ל-transfer | כשהחלופה היא חודשי העברה או שדרוג קו |
| **DataSync** לנפח בינוני | לפי GB — בינונית | סנכרון חוזר, לא חד-פעמי |
| **Direct Connect** | גבוהה + setup ארוך | חיבור **קבוע** ל-hybrid, לא העברה חד-פעמית |
| **Rehost (MGN)** | זול בפרויקט, **יקר ב-run-rate** | Deadline לחוץ; אחר כך עושים אופטימיזציה |
| **Refactor** | יקר בפרויקט, **הזול ביותר לאורך זמן** | מערכת ליבה שתישאר שנים |
| **Outposts** | **היקרה ביותר** בקטגוריה | רק כשיש דרישת residency או latency אמיתית |

### 🚩 עלויות נסתרות

- **Replication Instance של DMS שלא כיבו** — ה-migration הסתיים לפני חודש, ה-instance עדיין רץ.
- **Staging area של MGN** — EC2 + EBS לכל שרת שמשוכפל, במקביל לשרת המקורי.
- **Endpoint של Transfer Family** — מחויב לשעה גם בלילה, גם בסופ"ש, גם כשאף אחד לא מעלה קובץ.
- **תקופת החזקה של Snowball** — ימים מעבר למכסה מחויבים בנפרד, ומשלוח חזרה עולה.
- **Data transfer OUT** — הכנסת דאטה ל-AWS זולה או חינם; **הוצאה החוצה עולה**.
- **Requests ב-Storage Gateway** — Gateway עמוס מייצר מיליוני PUT/GET ל-S3.
- **חפיפה כפולה** — במהלך ההעברה משלמים גם על ה-DC הישן וגם על AWS. ככל שה-migration ארוך יותר, כך זה כואב יותר.

### 💡 טיפים לחיסכון

- **תריצו Discovery לפני הכל** — בממוצע חלק לא מבוטל מהשרתים ב-DC פשוט לא בשימוש. Retire הוא חינם.
- **כבו את ה-Replication Instance** ברגע שה-cutover אומת.
- **Transfer Family** — אם ההעלאות הן batch יומי, שקלו אם endpoint שרץ 24/7 באמת נחוץ.
- **Snowball במקום שדרוג קו** — אל תשדרגו את הקו לצמיתות בשביל העברה חד-פעמית.
- **Lifecycle Policy מאחורי File Gateway** — הדאטה הקר יורד ל-Glacier אוטומטית.
- **קצרו את חלון החפיפה** — כל שבוע של תשלום כפול הוא בזבוז נטו.
- אחרי Rehost — עשו **Replatform** למה שאפשר. EC2 שרץ 24/7 עם MySQL עליו הוא בזבוז מול RDS.

---

## 5. ⚖️ השוואות מכריעות

### שלוש הדרכים להעביר קבצים

| קריטריון | **Snowball** | **DataSync** | **Transfer Family** |
|---|---|---|---|
| אמצעי | מכשיר פיזי בדואר | רשת, agent | רשת, endpoint FTP |
| חד-פעמי או שוטף | **חד-פעמי** | **חוזר, מתוזמן** | **שוטף, יוזם המשתמש** |
| מי יוזם | אתם | לוח זמנים | **הלקוח/השותף החיצוני** |
| נפח אופייני | TB עד PB | GB עד TB במחזור | קבצים בודדים |
| משמר metadata/ACL | חלקית | **כן, במלואם** | לא רלוונטי |
| יעדים | S3 | S3, EFS, FSx | S3, EFS |
| דורש רוחב פס | **לא** | כן | כן |

### MGN מול DMS — הבלבול הנפוץ ביותר

| קריטריון | **MGN** | **DMS** |
|---|---|---|
| מה מעביר | **שרת שלם** — OS, אפליקציה, דיסקים | **דאטה של DB בלבד** |
| רמת הרפליקציה | **בלוקים** | **רשומות / טרנזקציות** |
| מתאים ל- | Rehost של VMs ושרתים פיזיים | מיגרציית DB עם downtime מינימלי |
| ממיר schema? | **לא** | **לא** — לזה יש SCT |
| ה-source ממשיך לרוץ | כן, ברפליקציה רציפה | כן, ה-DB זמין לאורך כל התהליך |
| הסיום | **Cutover** ל-EC2 יעד | הפסקת CDC והפניית האפליקציה ליעד |

### Storage Gateway מול Outposts מול VMware Cloud

| קריטריון | **Storage Gateway** | **Outposts** | **VMware Cloud on AWS** |
|---|---|---|---|
| מה מביאים ל-DC | VM קטן (גשר אחסון) | **מדפי שרתים של AWS** | שום דבר — vSphere רץ **בענן** |
| מה נשאר מקומי | הקומפיוט המקומי | **compute + storage מלאים** | רק ה-vCenter המנהל |
| הבעיה שנפתרת | S3 לא נגיש כ-file share | latency + **data residency** | רוצים להישאר ב-VMware |
| עלות יחסית | נמוכה | **הגבוהה ביותר** | בינונית-גבוהה |
| מילת מפתח בשאלה | "NFS/SMB/iSCSI on-premises" | "data must stay on-site" / "single-digit ms to local systems" | "keep using vSphere" |

> [!info] שורה תחתונה
> **Snowball = בעיית רוחב פס. DataSync = סנכרון חוזר. Storage Gateway = ה-DC נשאר.
> DMS = דאטה של DB. MGN = שרת שלם. Outposts = הדאטה אסור שיעזוב.**

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-Migration והיברידיות** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | ההעברה מנוהלת כפרויקט עם שלבים, לא כאירוע חד-פעמי | Application Discovery Service לפני הכל; חלוקה ל-**waves** לפי תלויות; **test launch** ב-MGN לפני cutover; מעקב ב-Migration Hub; runbook לכל wave כולל **rollback** |
| **Security** | הדאטה מוצפן במעבר ובמנוחה, וה-migration לא פותח דלתות | Snowball מוצפן ב-KMS; Storage Gateway מצפין ב-transit; **IAM Role לכל File Gateway** ולא credentials משותפים; DMS ב-**private subnet** ללא endpoint ציבורי; SFTP ולא FTP מחוץ ל-VPC |
| **Reliability** | ה-cutover לא הופך ל-נקודת כשל | **CDC** במקום העתקה חד-פעמית; **DMS Multi-AZ** ל-CDC ארוך; ולידציית דאטה לפני הפניית התעבורה; TTL נמוך ב-Route 53 לפני cutover כדי לאפשר חזרה מהירה |
| **Performance Efficiency** | הכלי מותאם לנפח ולרוחב הפס בפועל | חישוב זמן ההעברה **לפני** בחירת הכלי; Replication Instance בגודל שמתאים לקצב ה-writes; **cache מקומי** ב-Storage Gateway לדאטה החם; SCT על instance compute-intensive |
| **Cost Optimization** | לא משלמים על מה שכבר לא צריך, ולא כפול | **Retire** לפני שמעבירים; כיבוי Replication Instances וה-staging של MGN אחרי cutover; קיצור חלון החפיפה; Snowball במקום שדרוג קו קבוע |
| **Sustainability** | פחות חומרה פיזית פועלת בעולם | פירוק ה-DC בסיום; העברת **הדאטה הנחוץ בלבד** ולא ארכיונים מתים; מעבר ל-managed services במקום שרתים idle |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "limited bandwidth" / "would take months" | **Snowball Edge** |
| "process data at the edge, no internet" | **Snowball Edge Compute Optimized** |
| "existing tape backup software" / "physical tapes" | **Tape Gateway** |
| "on-premises servers need NFS or SMB access to S3" | **S3 File Gateway** |
| "iSCSI block volumes with cloud backup" | **Volume Gateway** |
| "entire dataset must stay on-premises, backups to AWS" | Volume Gateway — **Stored volumes** |
| "partners upload files over SFTP" | **Transfer Family** |
| "scheduled sync, preserve file permissions and metadata" | **DataSync** |
| "copy between EFS and FSx" | **DataSync** (ללא agent — זה AWS→AWS) |
| "migrate database with minimal downtime" | **DMS עם CDC** |
| "Oracle to Aurora PostgreSQL" | **SCT + DMS** (heterogeneous) |
| "on-prem PostgreSQL to RDS PostgreSQL" | **DMS בלבד** — אין צורך ב-SCT |
| "lift-and-shift 500 VMs" | **MGN** |
| "which servers depend on which" | Application Discovery Service — **Agent-based** |
| "data must not leave our facility" | **AWS Outposts** |
| "keep using vSphere / VMware tooling" | **VMware Cloud on AWS** |
| "RDS MySQL to Aurora, simplest way" | **DB Snapshot** ושחזור כ-Aurora |
| "zero replication lag before switching" | Aurora **Read Replica** ואז **promote** |

### טעויות נפוצות

> [!warning] מלכודת 1 — Snowball ישירות ל-Glacier
> **הניסוח:** "Use Snowball to import 500 TB of archives directly into S3 Glacier Deep Archive."
> **הטעות:** להניח ש-Snowball יכול לכתוב ל-Glacier.
> **הנכון:** **Snowball לא מייבא ל-Glacier ישירות.** הוא מייבא ל-**S3**, ומשם
> **S3 Lifecycle Policy** מעבירה ל-Glacier. זו שאלה שחוזרת כמעט מילה במילה.

> [!warning] מלכודת 2 — DMS כממיר schema
> **הניסוח:** "Migrate Oracle to Amazon Aurora PostgreSQL using DMS."
> **הטעות:** לבחור DMS לבד כתשובה שלמה.
> **הנכון:** זו **heterogeneous migration**. צריך **SCT** להמרת ה-schema, ה-stored procedures
> וה-views — ורק אחר כך DMS מעביר את הדאטה. DMS לבדו מעביר רשומות, לא לוגיקה.

> [!warning] מלכודת 3 — SCT מיותר
> **הניסוח:** "Migrate on-premises MySQL to Amazon RDS for MySQL. Which tools?"
> **הטעות:** להוסיף SCT כי "עוברים ל-RDS".
> **הנכון:** ה-engine לא השתנה — MySQL נשאר MySQL. **RDS הוא פלטפורמה, לא engine.**
> זו **homogeneous migration** ו-**DMS לבדו מספיק**.

> [!warning] מלכודת 4 — MGN ל-database
> **הניסוח:** "Use Application Migration Service to migrate the production Oracle database."
> **הטעות:** לחשוב ש-MGN מטפל בהכל כי הוא "מעביר שרתים".
> **הנכון:** MGN עושה rehost ברמת בלוקים — הוא **יעתיק את השרת עם ה-DB עליו כמו שהוא**,
> אבל הוא **לא ייתן downtime מינימלי ל-DB פעיל ולא ימיר כלום**. ל-DB חי — **DMS**.

> [!warning] מלכודת 5 — "Minimal downtime" זה לא "zero downtime"
> **הניסוח:** "DMS with CDC provides zero-downtime migration."
> **הטעות:** לקרוא CDC כאילו הוא מבטל את ה-cutover.
> **הנכון:** CDC **מצמצם** את החלון לדקות. עדיין צריך לעצור writes, לחכות ש-lag יגיע ל-0,
> לאמת דאטה ולהפנות את האפליקציה. **תמיד יש חלון.**

> [!warning] מלכודת 6 — Volume Gateway ו-Glacier
> **הניסוח:** "Volume Gateway archives volumes to S3 Glacier Deep Archive."
> **הטעות:** להניח שכל Storage Gateway מגיע ל-Glacier.
> **הנכון:** **Volume Gateway אינו כולל Glacier ו-Deep Archive.** Tape Gateway כן — זה כל
> הרעיון של ארכיון קלטות. ו-File Gateway מגיע ל-Glacier רק דרך **Lifecycle Policy**.

> [!warning] מלכודת 7 — FTP רגיל חשוף לאינטרנט
> **הניסוח:** "Expose AWS Transfer for FTP to external partners over the internet."
> **הטעות:** להתייחס לשלושת הפרוטוקולים כשווים.
> **הנכון:** **FTP רגיל נתמך רק בתוך VPC**, כי הוא לא מוצפן.
> לשותפים חיצוניים — **SFTP** או **FTPS**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברה סוגרת Data Center תוך 9 חודשים. יש בו:

- **200 VMs** של אפליקציות, רובן Windows ו-Linux סטנדרטיים.
- **300 TB** של קבצי ארכיון ותמונות רפואיות על NAS מקומי.
- **Oracle** בפרודקשן, 4 TB, שצריך לעבור ל-**Aurora PostgreSQL**.
- **מערכת ניתוח תמונות** שחייבת latency חד-ספרתי למכשור הרפואי בבניין — **אסור שהדאטה יעזוב את המתקן**.
- הקו לאינטרנט: **1 Gbps משותף**.
- Downtime מותר ל-DB: **פחות מ-30 דקות**.

```text
 שלב 0 — Discovery
 Application Discovery Service (Agent-based)  →  Migration Hub
        │ dependency map + utilization
        ▼
 ┌──────────────────────────────────────────────────────────┐
 │ 200 VMs                                                  │
 │   40 לא בשימוש  ──────────────► Retire                    │
 │   150 סטנדרטיות ──── MGN ─────► EC2  (waves של 20)        │
 │   10 מיוחדות ────── Replatform ► RDS / ECS               │
 ├──────────────────────────────────────────────────────────┤
 │ 300 TB ארכיון ─── Snowball Edge × N ──► S3 ──Lifecycle──► │
 │                                          Glacier Deep     │
 ├──────────────────────────────────────────────────────────┤
 │ Oracle 4TB ── SCT (schema) ──► Aurora PostgreSQL          │
 │            └─ DMS Full Load + CDC (Multi-AZ) ─────────►   │
 ├──────────────────────────────────────────────────────────┤
 │ ניתוח תמונות ──► AWS Outposts בבניין (EC2 + EBS + S3)     │
 └──────────────────────────────────────────────────────────┘
        │
        ▼  דאטה חדש שוטף אחרי ה-cutover
   DataSync (יומי)  +  Direct Connect
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **Discovery לפני הכל, Agent-based** | צריך **dependency mapping** כדי לדעת איזה שרתים חייבים לעבור באותו wave. Agentless לא נותן את זה |
| **Retire ל-40 שרתים** | החיסכון המיידי הגדול ביותר בפרויקט, בעלות אפס |
| **MGN ל-150 השרתים** | Rehost הוא היחיד שעומד ב-9 חודשים. רפליקציה רציפה + test launch לפני כל cutover |
| **Snowball ל-300 TB** | 300TB על 1 Gbps **משותף** = הרבה מעבר לשבוע → הכלל אומר Snow. גם לא נחנוק את הקו לשאר הפרויקט |
| **S3 ואז Lifecycle ל-Glacier** | **Snowball לא כותב ל-Glacier.** נוחתים ב-S3 Standard, ומדיניות מעבירה לארכיון |
| **SCT + DMS ל-Oracle → Aurora** | זו **heterogeneous migration** — engine משתנה. SCT ל-schema, DMS לדאטה |
| **DMS עם CDC** | Full Load של 4TB לוקח שעות; CDC סוגר את הפער כך שה-cutover הוא דקות — עומד בדרישת ה-30 דקות |
| **DMS Multi-AZ** | ה-CDC ירוץ שבועות עד ה-cutover. נפילת ה-Replication Instance באמצע = להתחיל מהתחלה |
| **Outposts למערכת הרפואית** | **הדאטה אסור שיעזוב את המתקן** + latency חד-ספרתי. אף פתרון רשת לא עונה על שניהם |
| **DataSync + DX אחרי ה-cutover** | Snowball פתר את הדאטה ההיסטורי. הדאטה החדש היומי עובר ברשת, מתוזמן, עם שימור הרשאות |

**למה לא DataSync לכל 300 ה-TB?**
כי הקו הוא 1 Gbps **משותף** — לא ייעודי. גם בתנאים אידיאליים זה שבועות, ובפועל נחנוק את הקו
שדרוש ל-MGN, ל-DMS ולעבודה השוטפת. DataSync הוא הכלי **אחרי** ה-bulk, לא במקומו.

**למה לא MGN ל-Oracle?**
MGN היה מעתיק את השרת עם Oracle עליו — כלומר נשארים על Oracle, על EC2, עם רישוי ותחזוקה.
הדרישה היא **Aurora PostgreSQL**. זה שינוי engine, ולזה יש רק מסלול אחד: SCT + DMS.

**למה לא Storage Gateway למערכת הרפואית?**
Storage Gateway פותר **גישה לאחסון**, לא **הרצת compute מקומי**. המערכת צריכה לעבד תמונות
בזמן אמת ליד המכשור. Outposts נותן EC2 ו-EBS אמיתיים בתוך הבניין.

**מה עוד נדרש:**
תכנון DR ליעד החדש ([[34 - Disaster Recovery]]), מדיניות גיבוי מרכזית ([[35 - Backup and Data Protection]]),
ואופטימיזציית עלות אחרי הייצוב — Rehost משאיר instances מנופחים ([[37 - Cost Optimization]]).

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **הקיבולת המדויקת** של כל מכשיר Snow. מספיק לזכור **Snowcone < Snowball < Snowmobile**.
- **פקודות ה-CLI** של DataSync, DMS או ה-Replication Agent.
- **רשימת ה-Sources וה-Targets המלאה** של DMS מילה במילה. מספיק לזהות את הכיוונים הלא-מובנים מאליהם
  (S3 כמקור, Kinesis ו-Kafka כיעד).
- **תהליך ההתקנה** של Storage Gateway VM על VMware או Hyper-V.
- **מפרט החומרה** של Outposts Racks או מודלי הרישוי שלהם.
- **מודל התמחור של VMware Cloud on AWS.** מספיק לזהות מתי הוא התשובה.
- **פרטי ה-flags** של Percona XtraBackup או `mysqldump`.
- **המבנה הפנימי** של ה-staging area ב-MGN.

> [!info] הערת עדכניות
> Snowmobile ו-Snowcone מופיעים בחומר הלימוד ובשאלות, אבל AWS צמצמה את זמינותם בפועל בשנים האחרונות.
> **למבחן — לומדים אותם כרגיל.** בעולם האמיתי בודקים מה עדיין מוצע ב-Region שלכם.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **6 Rs:** Rehost · Replatform · Repurchase · Refactor · Retire · Retain. (ויש ניסוח 7 Rs עם **Relocate**.)
- **הזול ביותר הוא Retire.** לכן Discovery בא ראשון.
- **יותר משבוע להעביר דרך הרשת → Snowball.** זה הכלל.
- **Snowball לא מייבא ל-Glacier.** רק ל-**S3**, ומשם **Lifecycle Policy**.
- **Snowball Edge Compute Optimized** — מריץ **EC2 ו-Lambda** ב-edge בלי אינטרנט.
- **Storage Gateway: File=NFS/SMB · Volume=iSCSI · Tape=VTL.**
- **Volume Gateway:** Cached = הדאטה בענן · Stored = הדאטה המלא מקומית, גיבוי לענן.
- **Volume Gateway לא תומך ב-Glacier.** Tape Gateway כן.
- **File Gateway** — IAM Role לכל gateway; **SMB משתלב עם Active Directory**.
- **Transfer Family:** SFTP/FTPS לאינטרנט; **FTP רגיל רק בתוך VPC**. תמחור לפי **שעת endpoint** + GB.
- **DataSync** — מתוזמן (שעתי/יומי/שבועי), **משמר הרשאות ו-metadata**, עד **10 Gbps** ל-agent.
  **AWS→AWS לא דורש agent.**
- **DMS:** ה-source נשאר **זמין**; **Full Load + CDC** ל-downtime מינימלי; דורש **Replication Instance**.
- **Homogeneous → DMS בלבד. Heterogeneous → SCT + DMS.**
- **On-prem PostgreSQL → RDS PostgreSQL = homogeneous.** RDS הוא פלטפורמה, לא engine.
- **DMS Multi-AZ** — standby סינכרוני; מבטל I/O freezes ומצמצם latency spikes.
- **RDS→Aurora:** Snapshot restore (פשוט) או Read Replica + **promote כשה-lag הוא 0**.
- **MySQL חיצוני → Aurora:** Percona XtraBackup דרך S3 (מהיר) או mysqldump (איטי).
- **MGN = rehost של שרתים** ברמת בלוקים, עם staging ואז cutover. **לא ממיר כלום.**
- **Application Discovery:** Agentless = מלאי וביצועים · **Agent-based = dependency mapping**.
- **Outposts** — AWS פיזית אצלכם; **data residency** ו-latency מקומי. אתם אחראים לאבטחה **הפיזית**.
- **VMware Cloud on AWS** — כשרוצים להישאר עם **vSphere**.
- **Hybrid connectivity:** VPN מהיר וזול להקמה; **Direct Connect** יציב ופרטי אבל setup של חודש+.

---

## 11. ✅ בדיקת הבנה

1. צריך להעביר 400 TB לענן. הקו הוא 1 Gbps. מה בוחרים ולמה?
2. יש 100 TB ארכיון שצריכים לשבת ב-Glacier Deep Archive. איך מגיעים לשם עם Snowball?
3. חברה מעבירה SQL Server ל-Aurora MySQL. אילו כלים נדרשים ובאיזה סדר?
4. חברה מעבירה MariaDB מ-on-prem ל-RDS MariaDB. האם צריך SCT?
5. שרת אפליקציה מקומי צריך לכתוב לקבצים ב-SMB, אבל האחסון צריך לשבת ב-S3. מה הפתרון?
6. מערכת גיבוי ותיקה עובדת מול ספריית קלטות פיזית ואי אפשר להחליף אותה. מה עושים?
7. מה ההבדל בין Cached ל-Stored ב-Volume Gateway, ומתי בוחרים כל אחד?
8. צריך למפות אילו שרתים ב-DC מדברים עם אילו לפני חלוקה ל-waves. איזה מצב Discovery?
9. בית חולים חייב לעבד תמונות מקומית ואסור שהדאטה יעזוב את הבניין. מה הפתרון?
10. למה DMS עם CDC לא נחשב "zero downtime"?

<details>
<summary>תשובות</summary>

1. **Snowball Edge.** 400TB על 1 Gbps ≈ 37 ימים בתנאים אידיאליים, ובפועל יותר —
   הרבה מעל השבוע. בנוסף, ההעברה תחנוק את הקו לכל שאר הפרויקט.
2. **לא מייבאים ל-Glacier ישירות.** Snowball מייבא ל-**S3**, ואז מגדירים
   **S3 Lifecycle Policy** שמעבירה את האובייקטים ל-Glacier Deep Archive.
3. זו **heterogeneous migration** — ה-engine משתנה. קודם **SCT** ממיר את ה-schema,
   ה-stored procedures וה-views; מה שלא ניתן להמרה אוטומטית נכתב ידנית.
   אחר כך **DMS** עושה Full Load ואז CDC עד ה-cutover.
4. **לא.** MariaDB → MariaDB זו **homogeneous migration**. RDS הוא פלטפורמה מנוהלת,
   לא engine אחר. **DMS לבדו מספיק.**
5. **S3 File Gateway.** הוא חושף את ה-bucket כ-share ב-**NFS או SMB**, שומר cache מקומי
   של הדאטה החם, ו-SMB יכול להשתלב עם **Active Directory** לאימות משתמשים.
6. **Tape Gateway.** הוא מציג **Virtual Tape Library** דרך iSCSI, כך שתוכנת הגיבוי הקיימת
   ממשיכה לעבוד בדיוק כמו קודם — אבל הקלטות יושבות ב-S3, והמאורכבות ב-Glacier.
7. **Cached** — ה-dataset המלא יושב **בענן**, ומקומית יש רק cache של הדאטה שנקרא לאחרונה.
   בוחרים כשרוצים לחסוך אחסון מקומי.
   **Stored** — ה-dataset המלא יושב **on-premises**, ולענן נשלחים גיבויים מתוזמנים כ-EBS Snapshots.
   בוחרים כשצריך גישה מקומית מלאה ומהירה, וה-ענן משמש ל-DR.
8. **Agent-based Discovery** (Application Discovery Agent). הוא היחיד שאוסף
   **תהליכים רצים וחיבורי רשת בין שרתים**. Agentless נותן רק מלאי, קונפיגורציה וביצועים.
9. **AWS Outposts.** זו הדרישה הקלאסית שלו — **data residency** יחד עם **latency נמוך
   לציוד מקומי**. Storage Gateway לא מספיק כי צריך **compute** מקומי, לא רק אחסון.
10. כי CDC **מצמצם** את החלון, לא מבטל אותו. עדיין צריך לעצור writes ב-source,
    לחכות שה-replication lag יגיע ל-0, לאמת שהדאטה תואם, ולהפנות את האפליקציה ליעד.
    זה חלון של דקות — "minimal", לא "zero".

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[16 - S3 Fundamentals]] · [[18 - S3 Advanced Features]] · [[20 - EFS and File Storage]] · [[12 - VPC Private Connectivity]] · [[21 - RDS Fundamentals]] · [[22 - RDS Scaling and Availability]] · [[34 - Disaster Recovery]] · [[35 - Backup and Data Protection]] · [[37 - Cost Optimization]] · [[39 - Architecture Decision Making]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 6235–6335, 6497–6772, 14924–15096, 15166–15257, 15957–16006
