---
lesson: 20
title: EFS and File Storage
domain: Design High-Performing Architectures
services: [EFS, FSx, Storage Gateway, DataSync, Transfer Family, Snowball]
tags: [saa-c03, storage, file-storage, hybrid]
---

# 20 — EFS and File Storage

> [!abstract] בשורה אחת
> כשיותר ממכונה אחת צריכה את אותם קבצים — עוברים מ-block ל-file storage, וכל השאלה היא איזה פרוטוקול נדרש: NFS/Linux → EFS, SMB/Windows → FSx for Windows, HPC → FSx for Lustre, ו-on-premises → Storage Gateway.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה block storage לא מספיק ומתי צריך file share |
| 2 | איך EFS עובד | mount targets, NFS, אלסטיות, אבטחה |
| 3 | פירוק EFS | performance modes, throughput modes, storage classes, lifecycle |
| 4 | ארבע גרסאות FSx | Windows, Lustre, NetApp ONTAP, OpenZFS |
| 5 | Hybrid ומיגרציה | Storage Gateway, DataSync, Transfer Family, Snowball |
| 6 | עלות | על מה משלמים בכל שירות ואיפה מתחבאות ההוצאות |
| 7 | טבלת ההשוואה הגדולה | EBS מול EFS מול S3 מול FSx |
| 8 | מלכודות ו-Scenario | מילות מפתח שמכריעות את השאלה |

**מונחי מפתח בשיעור:** `NFS` · `SMB` · `Mount Target` · `Lustre` · `Storage Gateway` · `DataSync` · `VTL`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- EBS volume מתחבר ל-instance אחד בלבד, והוא נעול ל-AZ יחיד.
- אתר WordPress עם 30 שרתי web צריך שכולם יראו את אותה תיקיית uploads.
- כשמשתמש מעלה תמונה לשרת אחד, שאר השרתים חייבים לראות אותה מיד.
- לפעמים לאפליקציה מדור קודם צריך שיתוף SMB עם הרשאות Active Directory.
- ולפעמים חצי מהתשתית עדיין on-premises וצריכה גישה לדאטה בענן.

### מה השירות פותר

- **EFS** — file system מנוהל שמאות ואלפי instances מרכיבים במקביל, על פני כל ה-AZs ב-Region.
- **אלסטיות** — גדל ומצטמצם אוטומטית; אין capacity planning ואין provisioning.
- **FSx** — file systems של צד שלישי (Windows, Lustre, NetApp, ZFS) כשירות מנוהל.
- **Storage Gateway / DataSync / Transfer Family** — גשרים בין on-premises לענן.

> [!tip] האנלוגיה
> EBS הוא דיסק חיצוני של מחשב אחד. EFS הוא כונן הרשת המשותף של כל המשרד.
> Storage Gateway הוא ארון תיוק מקומי שמסתנכרן אוטומטית עם הארכיון המרכזי.

---

## 2. ⚙️ איך EFS עובד

### 2.1 העקרונות

- שירות **NFS מנוהל** — הפרוטוקול הוא NFSv4.1.
- מותקן על מכונות **Linux** בלבד; זהו file system מסוג POSIX עם API סטנדרטי.
- **Windows אינו נתמך** — זו מילת מפתח שמכריעה שאלות במבחן.
- הגישה נשלטת דרך **Security Group** (פורט TCP 2049), בדיוק כמו מכונה.
- הצפנה at rest באמצעות KMS, והצפנה in transit ב-TLS.
- קיבולת גדלה ומתכווצת אוטומטית — משלמים רק על מה שמאוחסן בפועל.
- קנה מידה: אלפי clients במקביל, throughput של 10+ GB/s, עד קנה מידה של פטה-בייטים.

### 2.2 Mount Targets — הארכיטקטורה

```text
Region: us-east-1
┌────────────────┬────────────────┬────────────────┐
│  us-east-1a    │  us-east-1b    │  us-east-1c    │
│   EC2 × N      │   EC2 × N      │   EC2 × N      │
│      │         │      │         │      │         │
│  Mount Target  │  Mount Target  │  Mount Target  │
└──────┴─────────┴──────┴─────────┴──────┴─────────┘
                        │
             ┌──────────▼──────────┐
             │  EFS File System    │  ← אותה דאטה לכולם
             └─────────────────────┘
```

- Mount Target הוא ה-ENI שדרכו ה-instances באותו AZ ניגשים ל-file system.
- **יוצרים Mount Target בכל AZ** שבו יש clients — אחרת התעבורה חוצה AZ ומשלמים עליה.
- ה-Security Group שמוצמד ל-Mount Target חייב להתיר 2049 מה-SG של ה-instances.

### 2.3 Use cases טיפוסיים

- Content management ו-web serving (WordPress הוא הדוגמה הקנונית).
- שיתוף דאטה בין containers או בין שרתי אפליקציה.
- Home directories משותפים.
- Data analytics שבו כמה nodes קוראים את אותו dataset.

---

## 3. 🔍 פירוק EFS

### 3.1 Performance Mode — נקבע בזמן היצירה ולא ניתן לשינוי

| Mode | latency | מתי בוחרים |
|---|---|---|
| **General Purpose** (ברירת מחדל) | הנמוך ביותר | web servers, CMS, כל workload רגיש ל-latency |
| **Max I/O** | גבוה יותר | throughput אדיר במקביליות גבוהה: Big Data, עיבוד מדיה |

- **הכלל:** אם השאלה לא מדגישה במפורש מקביליות עצומה — התשובה היא General Purpose.

### 3.2 Throughput Mode

| Mode | איך זה מתנהג | מתי בוחרים |
|---|---|---|
| **Bursting** | 50 MiB/s לכל TB מאוחסן, עם התפרצות עד 100 MiB/s | workload רגיל שגדל יחד עם הדאטה |
| **Provisioned** | קובעים throughput קבוע ללא תלות בגודל (למשל 1 GiB/s על 1 TB) | הרבה throughput על מעט דאטה |
| **Elastic** | מתאים את עצמו אוטומטית לעומס | workload **בלתי צפוי**; עד 3 GiB/s קריאה ו-1 GiB/s כתיבה |

> [!warning] המלכודת של Bursting
> ב-Bursting ה-throughput צמוד לכמות הדאטה. file system של 100 GB מקבל ~5 MiB/s בלבד.
> אם השאלה מתארת "מעט דאטה אבל צורך גבוה ב-throughput" — התשובה היא Provisioned או Elastic.

### 3.3 Storage Classes ו-Lifecycle

| Class | למי מיועד | עלות |
|---|---|---|
| **Standard** | קבצים שניגשים אליהם תדיר | בסיס |
| **Infrequent Access (EFS-IA)** | קבצים שנוגעים בהם לעיתים רחוקות | אחסון זול משמעותית + **חיוב על retrieval** |
| **Archive** | קבצים שניגשים אליהם כמה פעמים בשנה | כ-50% זול מ-IA |

- **Lifecycle Policy** מעבירה קבצים אוטומטית לפי כמה זמן לא ניגשו אליהם (למשל 60 יום → IA).
- קובץ שניגשים אליו חוזר אוטומטית ל-Standard (אם מוגדרת מדיניות intelligent tiering).

### 3.4 Availability — Standard מול One Zone

| היבט | EFS Standard | EFS One Zone |
|---|---|---|
| פריסה | Multi-AZ בתוך ה-Region | AZ יחיד |
| עמידות בכשל AZ | ✅ | ❌ |
| מיועד ל- | production | dev, test, דאטה ניתנת לשחזור |
| Backup | אופציונלי | **מופעל כברירת מחדל** |
| שילוב עם IA | ✅ | ✅ (EFS One Zone-IA) |
| חיסכון | — | **מעל 90%** בשילוב עם IA |

---

## 4. 🗂️ ארבע גרסאות FSx

FSx מריץ file systems של צד שלישי כשירות מנוהל, כשה-use case לא מתאים ל-EFS.

| גרסה | פרוטוקולים | מערכות הפעלה | מה מייחד | Use case קלאסי |
|---|---|---|---|---|
| **FSx for Windows File Server** | SMB (NTFS) | Windows, וגם Linux | אינטגרציה מלאה ל-Active Directory, ACLs, user quotas, DFS Namespaces | העברת file shares של Windows לענן |
| **FSx for Lustre** | Lustre (POSIX) | Linux | file system מקבילי, latency תת-מילישנייה, אינטגרציה הדוקה עם S3 | HPC, Machine Learning, עיבוד וידאו, מודלים פיננסיים, EDA |
| **FSx for NetApp ONTAP** | NFS + SMB + **iSCSI** | Linux, Windows, macOS, VMware, WorkSpaces, EC2/ECS/EKS | תאימות ה-OS הרחבה ביותר; dedup, compression, cloning | הרמת workload קיים של NetApp/NAS לענן ללא שינוי |
| **FSx for OpenZFS** | NFS (v3, v4, v4.1, v4.2) | אותה רשימה רחבה | עד 1,000,000 IOPS ב-latency מתחת ל-0.5ms | הרמת workload קיים של ZFS; ביצועי NFS גבוהים |

### 4.1 פרטים על FSx for Windows

- שני סוגי אחסון: **SSD** ל-workloads רגישי latency, ו-**HDD** לשימוש רחב וזול (home directories, CMS).
- ניתן להגדיר **Multi-AZ** לזמינות גבוהה.
- גיבוי יומי אוטומטי ל-S3.
- נגיש מ-on-premises דרך VPN או Direct Connect.
- מגיע לעשרות GB/s, מיליוני IOPS ומאות פטה-בייט.

### 4.2 FSx for Lustre — שתי אפשרויות פריסה

| היבט | Scratch File System | Persistent File System |
|---|---|---|
| מטרה | אחסון זמני | אחסון לטווח ארוך |
| Replication | **אין** — הדאטה אבודה אם ה-file server נופל | משוכפל בתוך אותו AZ |
| התאוששות | אין | קבצים שנכשלו מוחלפים תוך דקות |
| ביצועים | burst גבוה (פי ~6 מהר יותר, 200 MB/s לכל TiB) | יציב לאורך זמן |
| מתי | עיבוד קצר טווח, אופטימיזציית עלות | עיבוד ממושך, דאטה רגישה |

**האינטגרציה עם S3** היא הנקודה הכי נשאלת:

```text
S3 bucket (הדאטה הגולמית)
    │  FSx "רואה" את ה-bucket כ-file system
    ▼
FSx for Lustre ──► compute cluster (HPC / ML training)
    │
    └──► כותב את התוצאות בחזרה ל-S3
```

- Lustre מציג את ה-objects ב-S3 כקבצים רגילים — בלי להעתיק ידנית.
- תוצאות החישוב נכתבות בחזרה ל-bucket.

---

## 5. 🌉 Hybrid ומיגרציית דאטה

### 5.1 AWS Storage Gateway — שלושת הסוגים

הבעיה: S3 הוא טכנולוגיה קניינית ואינו NFS. איך חושפים דאטה ב-S3 לשרתים on-premises?

| סוג | פרוטוקול ל-on-prem | מה עומד מאחור | Use case |
|---|---|---|---|
| **S3 File Gateway** | NFS / SMB | S3 (Standard, Standard-IA, One Zone-IA, Intelligent-Tiering) | חשיפת bucket כ-file share; lifecycle ל-Glacier |
| **Volume Gateway** | **iSCSI** (block) | S3, עם גיבוי כ-**EBS Snapshots** | גיבוי ושחזור של volumes on-premises |
| **Tape Gateway** | iSCSI **VTL** | S3 + Glacier | החלפת ספריית קלטות פיזית בלי לשנות את תהליכי הגיבוי |

פרטים חשובים:

- **S3 File Gateway** — הדאטה שנעשה בה שימוש לאחרונה נשמרת ב-cache מקומי ל-latency נמוך.
  הגישה ל-bucket דרך IAM Role לכל gateway; פרוטוקול SMB משתלב עם Active Directory לאימות משתמשים.
- **Volume Gateway** מגיע בשני מצבים:
  - **Cached volumes** — כל ה-dataset ב-S3, רק החם נשמר מקומית. גישה נמוכת latency לדאטה עדכנית.
  - **Stored volumes** — כל ה-dataset נמצא on-premises, וגיבויים מתוזמנים נשלחים ל-S3.
- **Tape Gateway** — Virtual Tape Library. עובד עם תוכנות הגיבוי המובילות בשוק.
- כל ה-gateways נפרסים כ-VM (VMware / Hyper-V / KVM) או כ-appliance פיזי, עם cache מקומי.

### 5.2 AWS DataSync

- העברה וסנכרון של כמויות דאטה גדולות.
- **on-premises → AWS**: דורש התקנת **agent** (NFS, SMB, HDFS, S3 API).
- **AWS → AWS**: בין שירותי אחסון שונים, **ללא agent**.
- יעדים: S3 (כל storage class כולל Glacier), EFS, וכל גרסאות FSx.
- משימות הרפליקציה מתוזמנות — שעתי, יומי או שבועי.
- **הרשאות הקבצים והמטא-דאטה נשמרות** (NFS POSIX, SMB) — זו מילת מפתח במבחן.
- agent יחיד מגיע ל-10 Gbps; ניתן להגדיר bandwidth limit.

### 5.3 AWS Transfer Family

- ממשק **FTP / FTPS / SFTP** מנוהל מעל S3 או EFS.
- התשתית מנוהלת, סקיילבילית ו-Multi-AZ.
- ניהול credentials בתוך השירות, או אינטגרציה ל-Active Directory, LDAP, Okta או Cognito.
- **FTP רגיל** נתמך רק בתוך VPC (אין הצפנה — לא נחשף לאינטרנט).
- Use case: שותפים עסקיים שמסרבים לעבור מ-SFTP ל-API.

### 5.4 Snowball — כשהרשת פשוט לא מספיקה

| מכשיר | vCPU | זיכרון | אחסון |
|---|---|---|---|
| Snowball Edge Storage Optimized | 104 | 416 GB | 210 TB |
| Snowball Edge Compute Optimized | 104 | 416 GB | 28 TB |

**כלל האצבע לזכור:** אם ההעברה ברשת אורכת **יותר משבוע** — משתמשים ב-Snowball.

| נפח | 100 Mbps | 1 Gbps | 10 Gbps |
|---|---|---|---|
| 10 TB | 12 יום | 30 שעות | 3 שעות |
| 100 TB | 124 יום | 12 יום | 30 שעות |
| 1 PB | 3 שנים | 124 יום | 12 יום |

**Edge Computing** — Snowball Edge לא רק מעביר דאטה, אלא גם מריץ EC2 ו-Lambda במקום שבו אין אינטרנט:
משאית בכביש, ספינה בים, אתר כרייה. שימושים: עיבוד מקדים, ML, transcoding.

> [!warning] Snowball לא מייבא ישירות ל-Glacier
> חייבים לייבא ל-**S3** ואז להשתמש ב-Lifecycle Policy כדי להעביר ל-Glacier.

---

## 6. 💰 עלות ותמחור

### על מה מחייבים

| שירות | רכיב חיוב | הערה |
|---|---|---|
| EFS | GB-month בפועל, לפי storage class | **אין provisioning** — משלמים על מה שמאוחסן |
| EFS | Provisioned Throughput (אם נבחר) | חיוב נפרד לכל MiB/s |
| EFS | retrieval ב-IA / Archive | קריאה מ-tier קר עולה כסף |
| FSx | קיבולת provisioned + throughput capacity | כאן **כן** מקצים מראש |
| FSx Lustre | לפי סוג הפריסה; Scratch זול מ-Persistent | |
| Storage Gateway | לפי GB שנשמר ב-S3 + חיוב שעתי לכל gateway + data transfer out | |
| Transfer Family | **לכל endpoint provisioned לשעה** + GB שהועברו | ה-endpoint מחויב גם כשאיש לא מתחבר |
| DataSync | לכל GB שהועבר | פשוט וצפוי |
| Snowball | לכל job + ימי שימוש נוספים + משלוח | |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| S3 | הזול ביותר לכל GB | כשהאפליקציה יודעת לדבר object API |
| EBS gp3 | בינוני | דיסק של מכונה אחת |
| EFS One Zone-IA | חיסכון של מעל 90% מ-Standard | dev/test, דאטה שאפשר לשחזר |
| EFS Standard | **~פי 3 מ-gp2** | production shared file system |
| FSx for Windows HDD | בינוני | file shares כלליים של Windows |
| FSx for Lustre Persistent | יקר | HPC ממושך |

### 🚩 עלויות נסתרות

- **cross-AZ data transfer** ב-EFS כשאין Mount Target ב-AZ של ה-client.
- **retrieval fee** ב-EFS-IA — lifecycle אגרסיבי מדי על דאטה שכן ניגשים אליה יוצא יקר יותר.
- **Provisioned Throughput** ב-EFS שנשאר מוגדר גבוה אחרי שהעומס ירד.
- **endpoint שעתי** ב-Transfer Family — מחויב 24/7 גם בלי תעבורה.
- **FSx over-provisioning** — בניגוד ל-EFS, כאן מקצים קיבולת ומשלמים עליה.
- **data transfer out** מ-Storage Gateway חזרה ל-on-premises.

### 💡 טיפים לחיסכון

- Lifecycle Policy ב-EFS ל-IA אחרי 30–60 יום, ול-Archive אחרי מספר חודשים.
- EFS One Zone לכל סביבות dev/test.
- Mount Target בכל AZ — ביטול מלא של תשלומי cross-AZ.
- Throughput Mode = Elastic כשהעומס בלתי צפוי, במקום Provisioned גבוה "ליתר ביטחון".
- FSx Lustre **Scratch** כשהעיבוד קצר וניתן להרצה מחדש.
- אם האפליקציה יכולה לעבוד עם object API — S3 יהיה תמיד זול יותר מ-EFS.

---

## 7. ⚖️ טבלת ההשוואה הגדולה

| קריטריון | EBS | EFS | S3 | FSx |
|---|---|---|---|---|
| סוג אחסון | **Block** | **File** | **Object** | **File** |
| חיבור מרובה | instance אחד (או 16 ב-Multi-Attach) | אלפי clients | בלתי מוגבל (HTTP) | מאות/אלפי clients |
| Scope | AZ יחיד | **Region** (multi-AZ) | Region, גישה גלובלית | AZ או Multi-AZ לפי הגרסה |
| פרוטוקול | חיבור block ל-instance | **NFSv4.1** | HTTPS / REST API | SMB, Lustre, NFS, iSCSI |
| מערכות הפעלה | הכל | **Linux בלבד** | לא רלוונטי | לפי הגרסה |
| קיבולת | provisioned, שינוי ידני | **אלסטי אוטומטי** | אינסופי | provisioned |
| תמחור | GB provisioned | GB בשימוש | GB + בקשות | קיבולת + throughput |
| Use case | boot volume, DB על EC2 | קבצים משותפים בין שרתי Linux | backup, static assets, data lake | Windows shares, HPC |

> [!info] שורה תחתונה
> "מכונה אחת" → EBS. "הרבה שרתי Linux, אותם קבצים" → EFS. "Windows/SMB/Active Directory" → FSx for Windows.
> "HPC / ML עם דאטה ב-S3" → FSx for Lustre. "האפליקציה יכולה לדבר API" → S3, וזה תמיד הזול ביותר.

---

## 8. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר בנושא הזה | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | file system משותף הוא נקודת כשל תפעולית משותפת | ניטור `BurstCreditBalance` ו-`PercentIOLimit` ב-CloudWatch; אוטומציה של mount ב-User Data |
| Security | קבצים משותפים = הרשאות משותפות | SG על פורט 2049 בלבד; EFS Access Points לבידוד תיקיות בין אפליקציות; הצפנה at rest ו-in transit |
| Reliability | AZ יחיד הוא הימור, גם ב-file storage | EFS Standard (Multi-AZ) ל-production; FSx Multi-AZ; AWS Backup על ה-file system |
| Performance Efficiency | הפרמטרים של EFS נבחרים לפי דפוס העומס | Elastic Throughput לעומס בלתי צפוי; FSx for Lustre כשצריך latency תת-מילישנייה |
| Cost Optimization | EFS יקר פי ~3 מ-EBS — כל GB מיותר עולה | Lifecycle ל-IA/Archive; One Zone ל-dev; העברת archive מוחלט ל-S3 |
| Sustainability | קיבולת אלסטית משמעה פחות חומרה מסתובבת סרק | מחיקת קבצים זמניים, tiering אגרסיבי, Scratch במקום Persistent ב-Lustre |

---

## 9. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "shared file system", "multiple EC2 instances", "POSIX" | EFS |
| "hundreds of instances across AZs share content" | EFS |
| "Windows", "SMB", "Active Directory", "NTFS ACLs" | FSx for Windows File Server |
| "HPC", "machine learning training", "sub-millisecond", "parallel" | FSx for Lustre |
| "process data that lives in S3 as a file system" | FSx for Lustre |
| "existing NetApp / NAS workload", "iSCSI + NFS + SMB" | FSx for NetApp ONTAP |
| "existing ZFS workload", "NFS with 1M IOPS" | FSx for OpenZFS |
| "on-premises needs NFS/SMB access to S3" | S3 File Gateway |
| "back up on-premises volumes to the cloud, iSCSI" | Volume Gateway |
| "replace physical tape backups" | Tape Gateway |
| "SFTP for business partners into S3" | AWS Transfer Family |
| "sync on-premises NFS to AWS on a schedule, keep permissions" | DataSync |
| "transfer would take months over the network" | Snowball |
| "compute at a remote site with no connectivity" | Snowball Edge (Compute Optimized) |
| "unpredictable throughput" | EFS Elastic Throughput |
| "dev environment, minimize file storage cost" | EFS One Zone-IA |

### טעויות נפוצות

> [!warning] מלכודת 1 — EFS ל-Windows
> **הניסוח:** "Windows servers need a shared drive with AD-based permissions."
> **הטעות:** לבחור EFS כי "צריך שיתוף קבצים".
> **הנכון:** EFS הוא POSIX/Linux בלבד. התשובה היא FSx for Windows File Server.

> [!warning] מלכודת 2 — EFS כ-boot volume
> **הניסוח:** "Reduce cost of the operating system disk."
> **הטעות:** להציע EFS.
> **הנכון:** לא ניתן לאתחל מ-EFS. boot volume הוא תמיד EBS מסוג SSD.

> [!warning] מלכודת 3 — DataSync מול Storage Gateway
> **הניסוח:** "On-premises application must read cloud data with local caching, continuously."
> **הטעות:** לבחור DataSync.
> **הנכון:** DataSync הוא **העברה מתוזמנת חד-פעמית או תקופתית**. גישה שוטפת עם cache = Storage Gateway.

> [!warning] מלכודת 4 — Snowball ישירות ל-Glacier
> **הניסוח:** "Import 200 TB of archives directly into S3 Glacier Deep Archive."
> **הטעות:** להניח שאפשר לבחור Glacier כיעד ה-import.
> **הנכון:** מייבאים ל-S3 ואז מעבירים ב-Lifecycle Policy.

> [!warning] מלכודת 5 — Bursting על file system קטן
> **הניסוח:** "50 GB dataset requires 300 MiB/s of throughput."
> **הטעות:** להניח ש-EFS ייתן את זה בברירת מחדל.
> **הנכון:** ב-Bursting, 50 GB נותנים כ-2.5 MiB/s בלבד. צריך Provisioned או Elastic Throughput.

---

## 10. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** אתר WordPress עם 30 שרתי web ב-3 AZs. כל השרתים חייבים לראות את אותה ספריית uploads.
נפח התמונות 4 TB, אבל 90% מהן לא נצפו יותר מחודשיים. העומס הוא ספייקים בלתי צפויים סביב קמפיינים.
בנוסף, צוות ה-BI on-premises צריך לקרוא ארכיון דוחות מה-bucket הראשי.

```text
                    Route 53 → CloudFront → ALB
                                              │
        ┌─────────────────┬───────────────────┴───┐
     AZ-1a             AZ-1b                   AZ-1c
   EC2 × 10          EC2 × 10                EC2 × 10
       │                 │                       │
   MountTarget       MountTarget            MountTarget
       └─────────────────┴───────────────────────┘
                         │
              EFS Standard (Elastic Throughput)
              Lifecycle: 60 יום → EFS-IA

    On-Premises BI ──NFS──► S3 File Gateway ──► S3 (reports bucket)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| EFS Standard (לא One Zone) | production על 3 AZs; One Zone היה יוצר תלות ב-AZ יחיד |
| Mount Target בכל אחד מ-3 ה-AZs | מונע לחלוטין חיובי cross-AZ data transfer |
| Elastic Throughput | העומס בספייקים בלתי צפויים; Provisioned היה יקר ומיותר בין הקמפיינים |
| Performance Mode = General Purpose | אתר web רגיש ל-latency, לא ל-throughput מקבילי מסיבי |
| Lifecycle ל-EFS-IA אחרי 60 יום | 90% מהדאטה קרה — חיסכון משמעותי בלי לשנות קוד |
| CloudFront לפני ה-ALB | רוב בקשות התמונות בכלל לא יגיעו ל-EFS |
| S3 File Gateway ל-BI | חושף את ה-bucket כ-NFS מקומי עם cache, בלי לשכתב את כלי ה-BI |
| SG ייעודי ל-Mount Target, פורט 2049 בלבד | מזעור משטח התקיפה |

**למה לא EBS Multi-Attach?** מוגבל ל-16 instances, ל-AZ יחיד, ודורש cluster-aware file system.

**למה לא DataSync ל-BI?** DataSync מסנכרן בלוחות זמנים ומייצר עותק; הצוות צריך גישת קריאה **שוטפת** לדאטה המקורית.

**למה לא FSx for Lustre?** אין כאן HPC. Lustre היה יקר משמעותית ולא מוסיף דבר ל-web serving.

---

## 11. 🚫 מה לא צריך לדעת למבחן

- פקודות ה-mount המדויקות ואפשרויות ה-`amazon-efs-utils`.
- הנוסחאות המדויקות של burst credits ב-EFS.
- הגדרות פנימיות של Lustre (stripe count, OST) או של ONTAP.
- דגמי Snowcone ו-Snowmobile לפרטי פרטים — מספיק לדעת שהם למיגרציה פיזית בסקאלה קטנה/ענקית.
- אופן ההתקנה של ה-Storage Gateway VM וגודלי ה-cache המומלצים.
- גרסאות NFS מדויקות שכל שירות תומך בהן (חוץ מהעובדה ש-EFS הוא NFSv4.1).

---

## 12. ⚡ Cheat Sheet

- EFS = NFS מנוהל, **Linux/POSIX בלבד**, Multi-AZ ברמת Region, אלסטי לחלוטין.
- Windows/SMB → **FSx for Windows**, לעולם לא EFS.
- Mount Target בכל AZ — אחרת משלמים cross-AZ.
- EFS נשלט ב-Security Group על פורט **2049**.
- Performance Mode: General Purpose (ברירת מחדל) מול Max I/O — נקבע פעם אחת בלבד.
- Throughput Mode: Bursting (50 MiB/s לכל TB) / Provisioned / **Elastic** (לבלתי צפוי).
- Storage classes: Standard → IA → Archive, עם Lifecycle לפי זמן מאז הגישה האחרונה.
- EFS One Zone + IA = חיסכון של מעל 90%, אבל AZ יחיד.
- EFS יקר בערך **פי 3 מ-gp2**.
- FSx for Lustre = HPC/ML, קורא וכותב ישירות מול S3; Scratch (זמני) מול Persistent (משוכפל ב-AZ).
- FSx for NetApp ONTAP = תאימות ה-OS הרחבה ביותר, כולל **iSCSI**.
- FSx for OpenZFS = עד 1M IOPS בפחות מ-0.5ms, NFS בלבד.
- Storage Gateway: File (NFS/SMB→S3) · Volume (iSCSI→S3 עם EBS snapshots) · Tape (VTL→S3/Glacier).
- DataSync = העברה מתוזמנת ששומרת הרשאות; agent נדרש רק מ-on-premises.
- Transfer Family = FTP/FTPS/SFTP מעל S3 או EFS; חיוב לפי endpoint לשעה.
- Snowball: אם ההעברה ברשת לוקחת מעל שבוע. לא מייבא ישירות ל-Glacier.

---

## 13. ✅ בדיקת הבנה

1. יש לך EFS בגודל 200 GB ב-Bursting Mode, והאפליקציה דורשת 400 MiB/s. מה הבעיה ומה הפתרון?
2. חברה מעבירה 500 TB מ-data center לענן. הקו הוא 1 Gbps. מה תמליץ ולמה?
3. אפליקציית .NET ישנה על Windows צריכה שיתוף קבצים עם הרשאות מ-Active Directory. EFS?
4. צוות DevOps רוצה לאפשר לשותפים עסקיים להעלות קבצים ב-SFTP ישירות ל-bucket. מה השירות?
5. מתי בוחרים FSx for Lustre Scratch ולא Persistent?

<details>
<summary>תשובות</summary>

1. ב-Bursting ה-throughput צמוד לגודל: 200 GB × 50 MiB/s לכל TB = כ-10 MiB/s בלבד. הפתרון הוא **Provisioned Throughput** (קובעים 400 MiB/s ללא תלות בגודל) או **Elastic Throughput** אם העומס בלתי צפוי.
2. **Snowball Edge Storage Optimized**. לפי טבלת הזמנים, 1 PB ב-1 Gbps לוקח 124 יום, ולכן 500 TB יקחו כחודשיים — הרבה מעל כלל השבוע. יש לזכור שהיעד הוא S3, ואם צריך ארכיון — Lifecycle ל-Glacier אחרי הייבוא.
3. לא. EFS תומך רק ב-Linux ובפרוטוקול NFS. התשובה היא **FSx for Windows File Server**, שנותן SMB, NTFS ואינטגרציה מלאה ל-Active Directory.
4. **AWS Transfer Family** (AWS Transfer for SFTP). הוא מספק endpoint מנוהל ו-Multi-AZ מעל S3, עם אימות מול AD/LDAP/Okta/Cognito. שים לב לחיוב השעתי הקבוע לכל endpoint.
5. כשהעיבוד קצר-טווח וניתן להרצה חוזרת מ-S3. Scratch אינו משוכפל — כשל של file server מאבד את הדאטה — אבל הוא זול יותר ונותן burst גבוה יותר (200 MB/s לכל TiB). ל-workload ממושך או לדאטה רגישה בוחרים Persistent.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[19 - EBS and EC2 Storage]] · [[16 - S3 Fundamentals]] · [[18 - S3 Advanced Features]] · [[36 - Migration and Hybrid Cloud]] · [[35 - Backup and Data Protection]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 1679–1798, 6235–6772
