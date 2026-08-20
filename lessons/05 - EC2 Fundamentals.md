---
lesson: 05
title: EC2 Fundamentals
domain: Design High-Performing Architectures
services: [EC2, EBS, Security Groups, ENI, Elastic IP, Placement Groups]
tags: [saa-c03, compute, ec2, foundations]
---

# 05 — EC2 Fundamentals

> [!abstract] בשורה אחת
> EC2 הוא שרת וירטואלי שאתם שוכרים לפי שנייה — ובמבחן רוב השאלות נופלות על **בחירת instance type נכון**, **Security Group שמוגדר נכון**, ו**איפה ה-instance יושב פיזית**.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | מתי EC2 ומתי לא — מול Lambda ו-containers |
| 2 | איך זה עובד | AMI, sizing, User Data, מחזור חיים של instance |
| 3 | פירוק מפורט | **מוסכמת השמות** `m5.2xlarge`, משפחות, SG, פורטים, IP, Placement Groups, ENI, Hibernate |
| 4 | עלות | על מה בדיוק משלמים ב-EC2 (מודלי רכישה ב-[[06 - EC2 Pricing and Optimization]]) |
| 5 | השוואות | EBS מול Instance Store, Cluster/Spread/Partition, Public IP מול EIP |
| 6 | Well-Architected | החלטות EC2 לפי ששת ה-Pillars |
| 7 | מלכודות | timeout מול connection refused, IP שמתחלף אחרי stop |
| 8 | Scenario | אפליקציית legacy עם דרישת latency נמוך |

**מונחי מפתח בשיעור:** `AMI` · `User Data` · `Instance Type` · `Security Group` · `Elastic IP` · `Placement Group` · `ENI` · `Hibernate`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- קניית שרת פיזי היא CapEx: מזמינים, מחכים שבועות, משלמים מראש על שיא שלא הגיע.
- אם הזמנתם קטן מדי — האפליקציה נופלת בעומס. גדול מדי — משלמים על אוויר.
- תוכנה ישנה דורשת לפעמים גרסת OS מסוימת, agent, kernel module או רישיון צמוד לחומרה.
- Serverless לא תמיד מתאים: תהליך שרץ שעות, runtime לא נתמך, או צורך בגישה לרמת ה-OS.

### מה השירות פותר

- **Elastic Compute Cloud** — שוכרים מכונה וירטואלית ומשלמים לפי זמן ריצה בפועל.
- **שליטה מלאה** ב-OS, ב-CPU, ב-RAM, ב-storage ובחוקי ה-firewall.
- **גמישות גודל** — אפשר לעצור instance, לשנות type ולהפעיל מחדש.
- **בסיס לכל השאר** — ELB, Auto Scaling, ECS ו-EKS כולם רצים מעל EC2 או מתנהגים כמוהו.

> [!tip] האנלוגיה
> EC2 הוא **השכרת רכב**. אתם בוחרים דגם (instance type), מקבלים מפתח (key pair),
> משלמים לפי זמן ולא קונים את הרכב. אם צריך רכב גדול יותר — מחזירים ולוקחים אחר.

> [!info] מתי **לא** EC2
> - קוד קצר שרץ לפי אירוע → [[25 - Lambda]].
> - אפליקציה ארוזה ב-container שלא צריכה גישה ל-OS → [[26 - Containers]].
> - DB מנוהל → [[21 - RDS Fundamentals]]. אין סיבה לתחזק MySQL על EC2 בעצמכם.

---

## 2. ⚙️ איך זה עובד

### 2.1 מה בוחרים כשמשיקים instance

| החלטה | מה זה | איפה מרחיבים |
|---|---|---|
| **AMI** | תבנית ה-disk: OS + תוכנה מותקנת מראש | Linux / Windows / macOS / AMI משלכם |
| **Instance Type** | כמה vCPU ו-RAM, וכמה רשת | סעיף 3.1 |
| **Storage** | EBS (רשת, persistent) או Instance Store (מקומי, זמני) | [[19 - EBS and EC2 Storage]] |
| **Network** | VPC + subnet + האם IP ציבורי | [[09 - VPC Fundamentals]] |
| **Security Group** | חוקי firewall — מי מדבר עם ה-instance | סעיף 3.3 |
| **Key Pair** | מפתח SSH לגישה | סעיף 3.5 |
| **IAM Role** | הרשאות שה-instance מקבל לשירותי AWS | [[03 - IAM Fundamentals]] |
| **User Data** | סקריפט bootstrap שרץ בהפעלה הראשונה | סעיף 2.3 |

### 2.2 AMI — מה זה בעצם

- **AMI = Amazon Machine Image** — תבנית שממנה נולד ה-instance.
- מכילה: OS, חבילות מותקנות, הגדרות, ו-block device mapping.
- **AMI הוא לא Snapshot.** Snapshot הוא גיבוי של volume בודד; AMI עוטף snapshot(ים) + metadata של איך להשיק מכונה.
- **AMI הוא regional.** רוצים אותו ב-Region אחר → מעתיקים (copy AMI).
- אפשר לבנות AMI משלכם ("golden image") כדי לקצר את זמן ה-boot של האפליקציה.

> [!tip] Golden AMI מול User Data
> ככל שתדחפו יותר התקנות ל-**AMI** — ה-boot מהיר יותר וה-scaling מגיב מהר יותר.
> ככל שתדחפו יותר ל-**User Data** — גמיש יותר, אבל כל instance חדש מבזבז זמן על התקנה.
> בפרודקשן: AMI בסיס עם התוכנה, User Data רק לקונפיגורציה קלה.

### 2.3 EC2 User Data — bootstrap

- סקריפט שמריצים בהשקת ה-instance כדי להכין אותו לעבודה.
- **רץ פעם אחת בלבד** — בהפעלה **הראשונה** של ה-instance.
- **רץ כ-root**, כלומר בלי `sudo` ובלי סיסמה.
- שימושים אופייניים: עדכוני מערכת, התקנת web server, הורדת קוד או קונפיג, רישום ל-monitoring.

```text
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable --now httpd
echo "Hello from $(hostname -f)" > /var/www/html/index.html
```

> [!warning] אף פעם לא credentials ב-User Data
> User Data נקרא דרך ה-**Instance Metadata** וגלוי לכל תהליך שרץ על המכונה.
> access keys שם = דליפה. הדרך הנכונה היא **IAM Role** מוצמד ל-instance.

### 2.4 מחזור החיים של instance

```text
                 ┌─────────┐
   launch  ──▶   │ pending │
                 └────┬────┘
                      ▼
                 ┌─────────┐  stop     ┌─────────┐  start
                 │ running │ ────────▶ │ stopped │ ───────▶ running
                 └────┬────┘           └─────────┘
                      │ terminate
                      ▼
                 ┌────────────┐
                 │ terminated │   (סופי, אין חזרה)
                 └────────────┘
```

| פעולה | מה קורה ל-EBS root | מה קורה ל-RAM | מה קורה ל-IP הציבורי | חיוב compute |
|---|---|---|---|---|
| **Stop** | נשמר | **נמחק** | **משתחרר ומשתנה** בהפעלה הבאה | מפסיק (עדיין משלמים על EBS) |
| **Start** | נטען חזרה | OS עולה מחדש | מקבל IP ציבורי **חדש** | מתחיל |
| **Reboot** | נשמר | נמחק | **נשאר אותו IP** | ממשיך |
| **Terminate** | נמחק (אם `DeleteOnTermination`) | נמחק | משתחרר | מפסיק |
| **Hibernate** | נשמר | **נשמר לקובץ ב-root EBS** | משתחרר ומשתנה | מפסיק |

> [!warning] Stop/Start עלול להעביר אתכם ל-host פיזי אחר
> ולכן: **Instance Store** (דיסק מקומי) מתאפס, וכל state שנשמר עליו נעלם.
> זו הסיבה שמעצבים instances כ-**disposable**: כל ה-state חי ב-RDS, ב-S3 או ב-EFS.

---

## 3. 🔍 פירוק מפורט

### 3.1 מוסכמת השמות של Instance Types — לפענח `m5.2xlarge`

זו אחת השאלות הכי קלות במבחן אם מבינים את הפירוק, והכי מבלבלת אם לא.

```text
        m      5      .     2xlarge
        │      │            │
        │      │            └── Size — כמה vCPU ו-RAM בתוך המשפחה
        │      └─────────────── Generation — דור. מספר גבוה = חדש, מהיר, בד"כ זול יותר
        └────────────────────── Instance Class / Family — לאיזה סוג עומס הוא מותאם
```

| חלק | בשם `m5.2xlarge` | משמעות |
|---|---|---|
| **Class** | `m` | General Purpose — איזון בין CPU, RAM ורשת |
| **Generation** | `5` | הדור החמישי. `m6` יהיה חדש יותר וחזק יותר לאותו כסף |
| **Size** | `2xlarge` | 8 vCPU. `xlarge` = 4, `2xlarge` = 8, `4xlarge` = 16 — כל קפיצה מכפילה |

**סולם הגדלים (העיקרון, לא לשינון):**

```text
nano → micro → small → medium → large → xlarge → 2xlarge → 4xlarge → 8xlarge → 16xlarge ...
                                              ×2       ×2        ×2        ×2
```

- כל דרגה בערך **מכפילה** vCPU ו-RAM — וגם את המחיר.
- ולכן `2 × m5.large` ו-`1 × m5.xlarge` עולים בערך אותו דבר. ההבדל הוא **עמידות**:
  שני instances קטנים בשתי AZs עדיפים על אחד גדול.

### 3.2 משפחות ה-Instance — מה לבחור למה

| משפחה | אות טיפוסית | מה מודגש | Use case קלאסי במבחן |
|---|---|---|---|
| **General Purpose** | `t`, `m` | איזון CPU / RAM / רשת | web servers, code repositories, סביבות dev, אפליקציה קטנה-בינונית |
| **Compute Optimized** | `c` | **מעבדים חזקים** ליחידת מחיר | batch processing, media transcoding, HPC, מודלים מדעיים, gaming servers, web בעל ביצועים גבוהים |
| **Memory Optimized** | `r`, `x`, `z` | **RAM גדול** | DB רלציוני/NoSQL בביצועים גבוהים, in-memory cache מבוזר, in-memory DB ל-BI, עיבוד real-time של data לא מובנה |
| **Storage Optimized** | `i`, `d`, `h` | **דיסק מקומי מהיר**, קריאה/כתיבה סדרתית | OLTP בתדירות גבוהה, DB רלציוני ו-NoSQL, cache ל-in-memory DB (Redis), data warehousing, file systems מבוזרים |
| **Accelerated Computing** | `p`, `g`, `inf` | GPU / מאיצים | אימון ML, inference, רינדור |

> [!tip] כלל זיהוי מהיר במבחן
> **"CPU-intensive" / "batch" / "transcoding"** → `c`.
> **"large data sets in memory" / "in-memory database" / "cache"** → `r`.
> **"high sequential read/write to local storage" / "high IOPS locally"** → `i`.
> **"web server" / "general workload"** → `t` או `m`.

**דוגמה להשוואה בין types (להמחשת היחסים):**

| Instance | vCPU | RAM (GiB) | Storage | הערה |
|---|---|---|---|---|
| `t2.micro` | 1 | 1 | EBS-Only | ה-instance של ה-Free Tier |
| `t2.xlarge` | 4 | 16 | EBS-Only | general purpose, burstable |
| `c5d.4xlarge` | 16 | 32 | NVMe SSD מקומי | יחס CPU:RAM גבוה — compute |
| `r5.16xlarge` | 64 | **512** | EBS-Only | יחס RAM:CPU גבוה — memory |
| `m5.8xlarge` | 32 | 128 | EBS-Only | יחס מאוזן 1:4 — general |

**הקיצור המנטלי:** `c` ≈ 2 GiB לכל vCPU · `m` ≈ 4 GiB · `r` ≈ 8 GiB.

> [!note] `t` — Burstable Performance
> משפחת `t` צוברת **CPU Credits** כשהיא בעומס נמוך ומוציאה אותם בשיאים.
> כשהקרדיטים נגמרים — הביצועים נחתכים לרמת ה-baseline.
> ולכן `t` מצוין לעומס לא אחיד, ומסוכן לעומס גבוה **מתמשך**.
> ב-`unlimited mode` אפשר לחרוג — בתשלום נוסף.

### 3.3 Security Groups — ה-firewall של EC2

- ה-**יסוד** של אבטחת רשת ב-AWS ברמת המשאב.
- שולטים על **פורטים**, **טווחי IP** (IPv4 ו-IPv6), בכיוון **inbound** ו-**outbound**.
- **מכילים רק כללי allow.** אין כלל deny ב-Security Group.
- **Stateful** — אם התרתם בקשה נכנסת, התשובה יוצאת אוטומטית בלי כלל נוסף.

| מאפיין | ההתנהגות | למה זה חשוב |
|---|---|---|
| **ברירת מחדל inbound** | **הכול חסום** | instance חדש לא נגיש עד שתפתחו פורט |
| **ברירת מחדל outbound** | **הכול מותר** | ה-instance יכול לפנות החוצה מיד |
| **allow בלבד** | אין deny | חסימת IP ספציפי דורשת **NACL** — ראו [[11 - VPC Security]] |
| **Stateful** | תשובה חוזרת אוטומטית | אין צורך בכלל ל-ephemeral ports |
| **מוצמד ל-ENI** | לא ל-instance עצמו | instance עם 2 ENIs יכול לשאת SGs שונים |
| **מרובה** | SG אחד → הרבה instances, instance אחד → הרבה SGs | הכללים **מצטברים** (union) |
| **Scope** | **Region + VPC** | SG לא עובר ל-VPC אחר או ל-Region אחר |
| **חי מחוץ ל-EC2** | הסינון קורה לפני שהחבילה מגיעה | תעבורה חסומה — ה-instance כלל לא רואה אותה |

**היכולת החשובה ביותר: הפניה ל-Security Group אחר**

במקום לכתוב טווחי IP, כלל ה-inbound יכול לציין **SG-מקור**:

```text
sg-web   (ALB / web tier)
   │
   │  כלל ב-sg-app:
   │  Inbound TCP 8080  ← Source: sg-web
   ▼
sg-app   (application tier)
   │
   │  כלל ב-sg-db:
   │  Inbound TCP 3306  ← Source: sg-app
   ▼
sg-db    (RDS)
```

- **למה זה מנצח:** ה-IP של instances משתנה כל הזמן ב-Auto Scaling. ה-SG לא.
- כל instance חדש שנולד עם `sg-app` מקבל גישה ל-DB **אוטומטית**, בלי לעדכן שום כלל.
- זו התשובה הנכונה כמעט תמיד בשאלות "tier מדבר עם tier".

### 3.4 פורטים קלאסיים שחייבים לזכור

| פורט | פרוטוקול | למה משמש |
|---|---|---|
| **22** | **SSH** | התחברות ל-instance של Linux |
| **22** | **SFTP** | העברת קבצים מעל SSH (אותו פורט) |
| 21 | FTP | העלאת קבצים ל-file share (לא מוצפן) |
| **80** | **HTTP** | אתר לא מאובטח |
| **443** | **HTTPS** | אתר מאובטח |
| **3389** | **RDP** | התחברות ל-instance של **Windows** |
| 3306 | MySQL / Aurora MySQL | DB |
| 5432 | PostgreSQL | DB |
| 1433 | MS SQL Server | DB |
| 6379 | Redis | cache |
| 11211 | Memcached | cache |

> [!tip] הזוג שהמבחן אוהב
> **Linux → 22 (SSH)**. **Windows → 3389 (RDP)**.
> שאלה שמזכירה "Windows instance" ופורט 22 היא כמעט תמיד הסחה.

### 3.5 דרכי התחברות ל-instance

| שיטה | דורש key file | דורש פורט 22 פתוח | הערה |
|---|---|---|---|
| **SSH** (Mac/Linux/Windows 10+) | כן | כן | הדרך הסטנדרטית |
| **PuTTY** (Windows ישן) | כן (`.ppk`) | כן | ממיר את ה-`.pem` |
| **EC2 Instance Connect** | **לא** | **כן** | AWS מעלה מפתח **זמני** ל-instance; עובד מהדפדפן |
| **Session Manager** (SSM) | לא | **לא!** | הדרך המומלצת היום — ראו [[31 - Monitoring and Logging]] |

- **EC2 Instance Connect** עובד out-of-the-box עם Amazon Linux 2, **ועדיין דורש פורט 22 פתוח** (מטווחי ה-IP של השירות).
- **Session Manager** הוא היחיד שלא דורש פורט 22 בכלל ולא דורש IP ציבורי — ולכן זו התשובה
  בשאלות "גישה מאובטחת ל-instance פרטי בלי bastion ובלי לפתוח פורטים".

### 3.6 Private IP מול Public IP מול Elastic IP

| סוג | ייחודי בתוך | משתנה? | עולה כסף? | שימוש |
|---|---|---|---|---|
| **Private IPv4** | הרשת הפרטית בלבד | קבוע לאורך חיי ה-instance | לא | תקשורת פנימית ב-VPC |
| **Public IPv4** | **כל האינטרנט** | **משתנה בכל stop/start** | **כן**, לשעה | גישה מבחוץ |
| **Elastic IP** | כל האינטרנט | **קבוע** עד שתשחררו | כן (וגם כשלא בשימוש) | IP ציבורי יציב |

- שתי רשתות פרטיות שונות **יכולות** להשתמש באותן כתובות פרטיות. כתובת ציבורית — לעולם לא.
- כתובת ציבורית ניתנת ל-geo-location; פרטית לא.
- מכונה בעלת IP פרטי בלבד יוצאת לאינטרנט דרך **NAT + Internet Gateway** — ראו [[10 - VPC Internet Connectivity]].

**Elastic IP — מה שחייבים לדעת:**

- כתובת IPv4 ציבורית שאתם "מחזיקים בבעלות" כל עוד לא שחררתם אותה.
- מוצמדת ל-**instance אחד בכל רגע**, וניתן להעביר אותה במהירות ל-instance אחר — כדי **להסתיר כשל**.
- מגבלה: **5 Elastic IPs לחשבון** (ניתן לבקש הגדלה).

> [!warning] AWS ממליצה **להימנע** מ-Elastic IP
> ברוב המקרים EIP מסמנת החלטה ארכיטקטונית לא טובה. החלופות:
> - IP ציבורי רגיל + רשומת **DNS** ב-[[14 - Route 53 and DNS]].
> - **Load Balancer** מלפנים, וה-instances בכלל בלי IP ציבורי — הפתרון המועדף.
>
> במבחן: אם התשובה "assign an Elastic IP to each instance" מופיעה לצד "put them behind an ALB" —
> ה-ALB כמעט תמיד נכון.

### 3.7 Placement Groups — איפה ה-instances יושבים פיזית

לפעמים חשוב לשלוט **איפה** AWS מציבה את המכונות ביחס זו לזו.

| | **Cluster** | **Spread** | **Partition** |
|---|---|---|---|
| **המטרה** | latency מינימלי, throughput מקסימלי | בידוד כשלים מקסימלי | בידוד ברמת קבוצות, בקנה מידה |
| **איך** | הכול על אותו rack, **AZ אחת** | כל instance על **חומרה נפרדת** | קבוצות (partitions) על **מערכי racks נפרדים** |
| **חוצה AZ?** | **לא** — AZ יחידה | **כן** | **כן** (באותו Region) |
| **מגבלה** | — | **7 instances לכל AZ** לכל group | **7 partitions לכל AZ**, מאות instances |
| **היתרון** | רשת 10 Gbps בין ה-instances (עם Enhanced Networking) | סיכון כמעט אפסי לכשל בו-זמני | כשל partition לא נוגע ב-partitions אחרים |
| **הסיכון** | **כשל AZ מפיל את הכול בבת אחת** | לא מתאים לצי גדול — התקרה נמוכה | כשל partition יכול להפיל הרבה instances יחד |
| **Use case** | Big Data שחייב להסתיים מהר, HPC, latency קיצוני | אפליקציות קריטיות שכל instance חייב להיות מבודד | **HDFS, HBase, Cassandra, Kafka** |

```text
Cluster                 Spread                      Partition
────────────            ─────────────────           ────────────────────────
  AZ-a                    AZ-a  AZ-b  AZ-c            AZ-a              AZ-b
┌──────────┐             ┌──┐  ┌──┐  ┌──┐          ┌─────┐ ┌─────┐   ┌─────┐
│ E E E E  │  ← rack     │HW│  │HW│  │HW│          │ P1  │ │ P2  │   │ P3  │
│ E E E E  │             │1 │  │3 │  │5 │          │E E E│ │E E E│   │E E E│
└──────────┘             └──┘  └──┘  └──┘          └─────┘ └─────┘   └─────┘
 מהיר, שביר             מבודד, מוגבל ל-7          racks נפרדים, מאות instances
```

- ב-**Partition**, כל instance מקבל את **מספר ה-partition שלו כ-metadata** — מה שמאפשר ל-Cassandra/HDFS
  לפזר replicas בין partitions בצורה חכמה.
- **הזיהוי במבחן:** מילים כמו `Cassandra`, `Kafka`, `HDFS`, `HBase` → **Partition**.
  `lowest possible latency between nodes` / `HPC` → **Cluster**.
  `each instance isolated from failure of others` → **Spread**.

### 3.8 ENI — Elastic Network Interface

- **כרטיס רשת וירטואלי** ב-VPC. ה-instance עצמו הוא לא בעל ה-IP — ה-**ENI** הוא.
- מה ה-ENI מחזיק:

| תכונה | פירוט |
|---|---|
| Private IPv4 ראשי | חובה, אחד |
| Private IPv4 משניות | אופציונלי, כמה |
| Elastic IP | **אחת לכל private IPv4** |
| Public IPv4 | אחת |
| Security Groups | אחד או יותר |
| MAC address | קבוע ל-ENI |

- **ENI קשור ל-AZ אחת בלבד** — אי אפשר להזיז אותו ל-AZ אחרת.
- אפשר ליצור ENI **בנפרד** מ-instance ולחבר/לנתק אותו **תוך כדי ריצה**.
- **הכוח שבזה — failover:** אם ה-instance נופל, מנתקים את ה-ENI ומחברים ל-instance אחר —
  והכתובת, ה-MAC וה-SGs עוברים איתו. הלקוחות ממשיכים לפנות לאותה כתובת.

```text
Availability Zone
  EC2-A                              EC2-B
  ├─ eth0 (primary ENI)  10.0.0.31   ├─ eth0 (primary ENI)  10.0.0.55
  └─ eth1 (secondary)    10.0.0.42 ──────▶ ניתן להעביר בזמן כשל
```

### 3.9 EC2 Hibernate

**הבעיה:** אחרי `stop` → `start`, ה-OS עולה מחדש, ה-cache מתאפס, והאפליקציה צריכה דקות להתחמם.

**מה Hibernate עושה:**

- שומר את **מצב ה-RAM** לקובץ ב-**root EBS volume**.
- ב-start הבא — ה-RAM נטען חזרה, ה-OS **לא** עובר boot מלא, והאפליקציה ממשיכה מאיפה שהפסיקה.
- ה-boot מהיר משמעותית, וה-cache כבר חם.

**התנאים (המבחן אוהב את אלה):**

| דרישה | הפירוט |
|---|---|
| **Root volume** | חייב להיות **EBS**, **מוצפן**, ומספיק גדול להכיל את ה-RAM |
| **גודל RAM** | **פחות מ-150 GB** |
| **Instance size** | **לא נתמך ב-bare metal** |
| משפחות נתמכות | C3/C4/C5, I3, M3/M4/M5, R3/R4/R5, T2/T3 ועוד |
| AMI | Amazon Linux 2, Ubuntu, RHEL, CentOS, Windows ועוד |
| מודלי רכישה | **On-Demand, Reserved ו-Spot** |
| **משך מקסימלי** | **עד 60 יום** ב-hibernation רצוף |

**Use cases:** תהליכים ארוכים שאסור לאבד את ההתקדמות שלהם, שירותים שלוקח להם זמן להתאתחל, ו-workstations לפיתוח.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

> [!info] היקף הסעיף הזה
> כאן מדובר על **מה** מחייבים ב-EC2. **מודלי הרכישה** (On-Demand, RI, Savings Plans, Spot,
> Dedicated Hosts, Dedicated Instances, Capacity Reservations) הם שיעור שלם —
> [[06 - EC2 Pricing and Optimization]].

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **זמן compute** | לפי **שנייה** ברוב ה-AMIs (מינימום 60 שניות) | תלוי ב-instance type וב-Region |
| **EBS volumes** | GB-חודש שהוקצו | **ממשיכים לחייב גם כשה-instance stopped** |
| **EBS snapshots** | GB-חודש, אינקרמנטלי | ראו [[19 - EBS and EC2 Storage]] |
| **Public IPv4** | **לשעה, לכל כתובת** | מאז 2024 גם IPv4 בשימוש מחויב |
| **Elastic IP לא מחוברת** | לשעה | EIP יתומה = כסף על כלום |
| **Data Transfer החוצה** | GB egress | ראו [[10 - VPC Internet Connectivity]] |
| **Cross-AZ traffic** | GB, **בשני הכיוונים** | טעות תכנון נפוצה ויקרה |
| **Instance Store** | **0 בנפרד** | כלול במחיר ה-instance |
| **Placement Group** | **0** | הרכיב עצמו חינם |
| **Security Groups / ENIs** | **0** | ENI שמחזיקה EIP — משלמים על ה-EIP |
| **Detailed CloudWatch monitoring** | בתשלום | ברירת מחדל (5 דקות) חינם |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **דור חדש** (`m6` מול `m5`) | **זול יותר** לאותם ביצועים | כמעט תמיד — שדרוג דור הוא חיסכון "בחינם" |
| Right-sizing (להוריד דרגה) | חוסך ~50% לכל דרגה | כשה-CPU/RAM בשימוש נמוך באופן קבוע |
| `t` burstable | הזול ביותר לעומס לא אחיד | dev, אתרים קטנים, עומס מקוטע |
| `t` בעומס מתמשך | **יקר** — burst credits נגמרים | אף פעם. עברו ל-`m`/`c` |
| Instance Store במקום EBS מהיר | חוסך את חיוב ה-IOPS | cache בלבד — הנתונים לא שורדים |
| **Stop** ל-dev בלילה ובסופ"ש | חוסך ~70% מזמן ה-compute | תמיד בסביבות שאינן פרודקשן |
| Instance גדול יחיד מול שניים קטנים | **מחיר compute דומה** | אבל שניים קטנים ב-2 AZs = זמינות טובה יותר |

### 🚩 עלויות נסתרות

- **EBS של instance ש-stopped** — ה-compute מפסיק, ה-storage לא. volume של 500 GB "כבוי" עדיין משלם.
- **EBS יתומים** — volumes שנשארו אחרי termination כי `DeleteOnTermination` היה כבוי.
- **Elastic IPs יתומות** — הכי קל לשכוח, והכי מעצבן לגלות בחשבונית.
- **Snapshots שנצברים** — מדיניות גיבוי בלי retention מייצרת זנב ארוך.
- **Cross-AZ data transfer** — instance ב-AZ-a שמדבר עם DB ב-AZ-b משלם GB לשני הכיוונים.
- **Public IPv4 על צי גדול** — 200 instances עם IP ציבורי מיותר זו דליפה שקטה וקבועה.
- **Detailed monitoring** שהודלק "ליתר ביטחון" על כל הצי.

### 💡 טיפים לחיסכון

- **Right-size לפי CloudWatch**, לא לפי תחושה. `CPUUtilization` נמוך לאורך שבועות = דרגה מיותרת.
- **שדרגו דור** לפני שמנסים כל דבר אחר. הרווח מיידי וללא סיכון.
- **Stop סביבות dev** אוטומטית — Instance Scheduler, ראו [[06 - EC2 Pricing and Optimization]].
- **בלי IP ציבורי** ל-instances שלא צריכים אותו — private subnet + NAT/Endpoint.
- **שחררו EIPs, volumes ו-snapshots יתומים** בסבב ניקיון קבוע.
- **שמרו את התעבורה בתוך AZ** כשאפשר — instance ו-DB באותה AZ, replica בשנייה.
- **Instance Store ל-cache**, EBS ל-persistent. אל תשלמו IOPS גבוה על מה שאפשר לאבד.

---

## 5. ⚖️ השוואות מכריעות

### EBS מול Instance Store מול EFS

| קריטריון | **EBS** | **Instance Store** | **EFS** |
|---|---|---|---|
| היכן פיזית | **דיסק ברשת** | **דיסק על ה-host עצמו** | שירות NFS מנוהל |
| שורד `stop`/`start` | **כן** | **לא** | לא רלוונטי — חיצוני |
| שורד כשל host | כן | **לא** | כן |
| Latency | נמוך | **הנמוך ביותר** | גבוה יותר |
| שיתוף בין instances | רק Multi-Attach מוגבל | לא | **כן, במקביל** |
| חוצה AZ | לא (volume ל-AZ אחת) | לא | **כן** |
| מתאים ל | boot volume, DB, נתונים קבועים | **cache**, scratch, buffer | קבצים משותפים, home dirs, CMS |
| **לא** מתאים ל | filesystem משותף POSIX | **כל דבר שחייב לשרוד** | boot volume |

הרחבה: [[19 - EBS and EC2 Storage]] ו-[[20 - EFS and File Storage]].

### Public IP מול Elastic IP מול DNS/ALB

| קריטריון | Public IP רגיל | Elastic IP | ALB + DNS |
|---|---|---|---|
| יציבות הכתובת | משתנה בכל stop/start | **קבועה** | הכתובת של ה-ALB יציבה |
| מגבלה | — | **5 לחשבון** | — |
| מתאים ל-Auto Scaling | לא | **לא** | **כן** |
| המלצת AWS | סביר לזמני | **להימנע** | **מועדף** |

### Cluster מול Spread מול Partition — בשורה אחת כל אחד

| אם הדרישה היא... | בוחרים |
|---|---|
| **latency מינימלי** ו-throughput מקסימלי בין nodes | **Cluster** |
| **כל instance מבודד** מכשל של האחרים, צי קטן | **Spread** |
| צי גדול של **HDFS/Cassandra/Kafka** עם בידוד לפי קבוצות | **Partition** |

> [!info] שורה תחתונה
> **Cluster = מהירות תמורת סיכון. Spread = בטיחות תמורת קנה מידה. Partition = פשרה שמתרחבת.**

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-EC2** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | כל instance ניתן לשחזור מאפס בלי ידיים | Golden AMI + Launch Template; User Data ל-bootstrap; SSM Session Manager במקום SSH ידני; tagging עקבי |
| **Security** | אין credentials על המכונה, ואין פורט פתוח מיותר | **IAM Role** במקום access keys; **IMDSv2** בלבד; SG שמפנה ל-SG אחר במקום ל-`0.0.0.0/0`; הצפנת EBS; ללא IP ציבורי ב-app tier |
| **Reliability** | כשל instance או AZ שלמה אינו אירוע | ASG על **2+ AZs**; **Spread/Partition** במקום Cluster לעומסים קריטיים; state מחוץ ל-instance; ENI ל-failover מהיר |
| **Performance Efficiency** | ה-type תואם לפרופיל העומס, לא לניחוש | `c` ל-CPU, `r` ל-RAM, `i` ל-IOPS מקומי; Enhanced Networking; **Cluster PG** ל-HPC; Instance Store ל-cache חם |
| **Cost Optimization** | לא משלמים על מה שלא רץ ולא על מה שגדול מדי | דור עדכני; right-sizing לפי CloudWatch; stop ל-dev; ניקוי EIP/EBS/snapshots יתומים; מודל רכישה מתאים ([[06 - EC2 Pricing and Optimization]]) |
| **Sustainability** | פחות חומרה פיזית לאותה עבודה | דורות חדשים (יעילות אנרגטית גבוהה יותר); **Graviton** כשה-workload תומך; כיבוי idle; scaling לפי ביקוש אמיתי |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "connection **times out**" | בעיית **Security Group** (או NACL / route) |
| "connection **refused**" | האפליקציה **לא רצה** או לא מאזינה — לא בעיית רשת |
| "public IP changed after restart" | להשתמש ב-**Elastic IP**, ועדיף ב-**ALB + DNS** |
| "web tier must reach app tier, IPs keep changing" | **SG שמפנה ל-SG** אחר |
| "lowest latency between nodes" / "HPC" | **Cluster Placement Group** |
| "Cassandra / Kafka / HDFS / HBase" | **Partition Placement Group** |
| "each instance isolated from hardware failure" | **Spread Placement Group** (מקס' 7 ל-AZ) |
| "in-memory database" / "large datasets in memory" | **Memory Optimized** (`r`) |
| "batch processing" / "media transcoding" | **Compute Optimized** (`c`) |
| "high sequential read/write to local storage" | **Storage Optimized** (`i`) |
| "preserve RAM state, fast restart" | **EC2 Hibernate** (root EBS **מוצפן**, RAM < 150 GB) |
| "no SSH keys, no open ports, no bastion" | **SSM Session Manager** |
| "move the IP to a standby instance on failure" | **ENI** מנותק ומחובר מחדש (או EIP) |
| "run a script at first boot" | **EC2 User Data** |
| "grant the instance access to S3" | **IAM Role**, לא access keys ב-User Data |

### טעויות נפוצות

> [!warning] מלכודת 1 — timeout מול connection refused
> **הניסוח:** "Users report the site times out." לעומת "Users get connection refused."
> **הטעות:** לטפל בשתיהן אותו דבר.
> **הנכון:** **timeout = SG/NACL/route חוסם.** **refused = השרת מגיב אבל אין תהליך שמאזין** בפורט
> — בעיית אפליקציה, לא רשת. זו הבחנה שנשאלת ישירות.

> [!warning] מלכודת 2 — "פתחנו הכול ב-SG ועדיין אין גישה"
> **הניסוח:** "Security group allows all traffic but the instance is unreachable from the internet."
> **הטעות:** להוסיף עוד כללי SG.
> **הנכון:** SG הוא רק שכבה אחת. בדקו **route ל-IGW**, **כתובת ציבורית**, ו-**NACL**.
> ראו [[09 - VPC Fundamentals]].

> [!warning] מלכודת 3 — הנחה שה-IP הציבורי קבוע
> **הניסוח:** "We hardcoded the instance's public IP in the client config."
> **הטעות:** להניח שהוא לא ישתנה.
> **הנכון:** `stop`/`start` **משנה** את ה-IP הציבורי. `reboot` לא.
> הפתרון הנכון: **ALB + DNS**, לא EIP לכל instance.

> [!warning] מלכודת 4 — Instance Store כאחסון "רגיל"
> **הניסוח:** "Store the database files on the instance store for maximum IOPS."
> **הטעות:** לבחור בזה כי הביצועים מפתים.
> **הנכון:** Instance Store **נמחק** ב-stop, ב-terminate ובכשל host. מתאים ל-**cache** בלבד.

> [!warning] מלכודת 5 — Cluster Placement Group לזמינות
> **הניסוח:** "Use a cluster placement group for a highly available application."
> **הטעות:** להניח ש-placement group = חוסן.
> **הנכון:** **Cluster הוא AZ יחידה** — כשל AZ מפיל את הכול. ל-HA בוחרים **Spread** או **Partition**.

> [!warning] מלכודת 6 — credentials ב-User Data
> **הניסוח:** "Embed the access key in the user data script so the app can write to S3."
> **הטעות:** זה עובד, ולכן מפתה.
> **הנכון:** User Data גלוי דרך ה-metadata. **IAM Role** מוצמד ל-instance הוא התשובה היחידה.

> [!warning] מלכודת 7 — לשכוח שה-EBS ממשיך לחייב
> **הניסוח:** "We stopped all dev instances, why is the bill still high?"
> **הטעות:** להניח ש-stop = 0 עלות.
> **הנכון:** stop מפסיק את ה-**compute** בלבד. **EBS, snapshots ו-EIP ממשיכים לחייב.**

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברה מריצה אפליקציית ניתוח פיננסי legacy. מאפיינים:

- מנוע החישוב דורש **agent מותקן** ו-kernel module ייעודי — לא ניתן להעביר ל-Lambda.
- שלב החישוב **CPU-intensive** ורץ במקבץ של 40 nodes שמדברים ביניהם בצפיפות.
- שכבת ה-web מקבלת משתמשים מהאינטרנט וצריכה זמינות גבוהה.
- ה-DB חייב להיות בלתי נגיש מהאינטרנט.
- הצוות מתלונן שה-instances לוקחים 6 דקות להתחמם אחרי כל restart.

```text
                       Internet
                          ↓
                   ┌──────────────┐
                   │     ALB      │   public subnets, 2 AZs
                   └──────┬───────┘
                          ↓  sg-web → sg-app
        ┌─────────────────────────────────────┐
        │  Web/App tier — ASG (m6i.large)     │  private subnets, 2 AZs
        │  Spread / ללא Placement Group        │
        └────────┬──────────────────┬──────────┘
                 │ sg-app → sg-calc │ sg-app → sg-db
                 ↓                  ↓
   ┌───────────────────────┐   ┌──────────────┐
   │ Compute cluster       │   │  RDS Multi-AZ│  data subnets
   │ 40 × c6i.8xlarge      │   └──────────────┘
   │ Cluster Placement Grp │
   │ AZ יחידה               │
   └───────────────────────┘
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **EC2 ולא Lambda** | agent ו-kernel module דורשים שליטה ברמת ה-OS |
| שכבת web על **`m` (General Purpose)** | עומס מאוזן — לא CPU-bound ולא RAM-bound |
| מנוע החישוב על **`c` (Compute Optimized)** | "CPU-intensive batch" הוא ההגדרה של משפחת `c` |
| **Cluster Placement Group** לצי החישוב | ה-nodes מדברים ביניהם בצפיפות — latency ו-throughput קובעים את זמן הריצה |
| **מקבלים** שה-cluster ב-AZ יחידה | ה-job הוא batch שניתן להריץ מחדש. הסיכון מקובל תמורת המהירות |
| שכבת web **ללא Cluster PG**, על **2 AZs** | כאן זמינות חשובה יותר מ-latency — ההפך המדויק מהחישוב |
| **SG מפנה ל-SG** בכל מעבר שכבה | ב-ASG ה-IPs משתנים כל הזמן; כללים לפי IP יישברו |
| **RDS ב-data subnet ללא route לאינטרנט** | הגנה מבנית, לא הגדרתית — ראו [[09 - VPC Fundamentals]] |
| **Golden AMI** עם ה-agent מותקן מראש | חותך את זמן ההתחממות; User Data נשאר לקונפיג בלבד |
| **Hibernate** ל-instances של אנליסטים | ה-RAM נשמר, ה-cache חם, ההתחלה מיידית — root EBS **מוצפן** ו-RAM < 150 GB |
| **ללא IP ציבורי** לשום instance | הכניסה רק דרך ALB; ניהול דרך SSM Session Manager |

**למה לא Spread ל-cluster החישוב?**
כי Spread מוגבל ל-**7 instances לכל AZ** — 40 nodes לא נכנסים, ובעיקר: Spread מפזר בכוונה
על חומרה מרוחקת, וזה בדיוק ההפך ממה שה-job הזה צריך.

**למה לא Elastic IP לכל instance?**
5 EIPs לחשבון, והן לא נעות עם Auto Scaling. ALB פותר את הכתובת היציבה בלי EIP בכלל.

**למה לא instance אחד ענק במקום 40?**
Single point of failure, ותקרת גודל. 40 nodes מקבילים מסיימים מהר יותר ומאפשרים retry נקודתי.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **לשנן את כל משפחות ה-instance** ואת המפרט המדויק שלהן. מספיק להכיר `t`/`m`/`c`/`r`/`i` ומה כל אחת פותרת.
- **מהירויות CPU מדויקות** או דגמי מעבד ספציפיים.
- **טבלת מחירים בדולרים** — משתנה לפי Region ולפי זמן. הבחינה שואלת יחסים, לא מספרים.
- **הפרוצדורה המדויקת של PuTTY** או המרת `.pem` ל-`.ppk`. מספיק לדעת שזו דרך התחברות מ-Windows ישן.
- **רשימת ה-AMIs המדויקת** שתומכת ב-Hibernate. מספיק לזכור את התנאים העיקריים.
- **Enhanced Networking / ENA לעומק** — מספיק לדעת שהוא נדרש כדי להגיע ל-10 Gbps ב-Cluster PG.
- **CLI/API syntax** להשקת instances.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **`m5.2xlarge`** = class `m` · generation `5` · size `2xlarge`. כל דרגת size ≈ **מכפילה** vCPU ו-RAM.
- **`t`/`m` = General · `c` = Compute · `r` = Memory · `i` = Storage · `p`/`g` = GPU.**
- **AMI ≠ Snapshot.** AMI הוא template להשקה, והוא **regional**.
- **User Data רץ פעם אחת בלבד, כ-root, בהפעלה הראשונה.** אף פעם לא credentials שם.
- **Security Group: allow בלבד, stateful, מוצמד ל-ENI.** inbound חסום כברירת מחדל, outbound פתוח.
- **SG יכול להפנות ל-SG אחר** — התשובה הנכונה כמעט תמיד ב-tier-to-tier.
- **timeout = בעיית Security Group. connection refused = בעיית אפליקציה.**
- **פורטים: 22 SSH · 3389 RDP · 80 HTTP · 443 HTTPS · 3306 MySQL · 5432 Postgres.**
- **stop/start משנה את ה-IP הציבורי. reboot לא.**
- **Elastic IP: 5 לחשבון, קבועה, עולה גם כשלא בשימוש.** AWS ממליצה להעדיף **ALB + DNS**.
- **Cluster** = AZ אחת, latency מינימלי, כשל AZ מפיל הכול.
- **Spread** = חומרה נפרדת, **מקס' 7 instances ל-AZ**, חוצה AZ.
- **Partition** = **7 partitions ל-AZ**, מאות instances, ל-Cassandra/Kafka/HDFS/HBase.
- **ENI** = כרטיס רשת וירטואלי, **קשור ל-AZ אחת**, ניתן להעברה בין instances ל-failover.
- **Hibernate:** RAM נשמר ל-root EBS, שחייב להיות **מוצפן**; RAM **< 150 GB**; לא bare metal; **עד 60 יום**.
- **EC2 Instance Connect** לא דורש key file אבל **כן** דורש פורט 22. **SSM Session Manager** לא דורש כלום.
- **stop מפסיק חיוב compute — לא חיוב EBS, snapshots ו-EIP.**

---

## 11. ✅ בדיקת הבנה

1. פענחו את `c6i.4xlarge`: מה כל חלק בשם אומר, ולאיזה עומס ה-instance הזה מתאים?
2. משתמש מדווח על "connection timed out" ומשתמש אחר על "connection refused". איפה מחפשים כל אחת?
3. שכבת ה-app ב-Auto Scaling צריכה לגשת ל-RDS. למה כלל SG לפי טווח IP הוא פתרון גרוע?
4. צריך להריץ אשכול Cassandra של 60 nodes עם בידוד כשלים. איזה Placement Group, ולמה לא השניים האחרים?
5. עצרתם את כל instances ה-dev בסוף היום. למה החשבון עדיין לא ירד לאפס?
6. אפליקציה לוקחת 8 דקות לחמם cache אחרי כל הפעלה. איזה מנגנון EC2 פותר את זה, ומה שני התנאים הקריטיים?
7. instance נפל ואתם רוצים שהתעבורה תמשיך להגיע לאותה כתובת פרטית תוך שניות. מה מעבירים?
8. למה AWS ממליצה להימנע מ-Elastic IP, ומה שתי החלופות שהיא מציעה?

<details>
<summary>תשובות</summary>

1. `c` = **Compute Optimized**, `6` = דור שישי, `i` = וריאנט מעבדי Intel, `4xlarge` = 16 vCPU.
   מתאים ל-**batch processing, transcoding, HPC, web בעל ביצועים גבוהים** — עומס שחסום ב-CPU ולא ב-RAM.
2. **timeout** → התעבורה נחסמת לפני שהגיעה: **Security Group**, NACL, route table, או ה-instance לא באמת רץ.
   **connection refused** → החבילה **הגיעה** וה-OS ענה "אין כאן אף אחד": **האפליקציה לא רצה** או מאזינה בפורט אחר.
3. כי ב-ASG ה-instances נולדים ומתים כל הזמן וה-**IP משתנה בכל פעם**. כלל לפי IP יישבר או יאלץ לפתוח טווח רחב מדי.
   הנכון: כלל ב-`sg-db` שמתיר פורט 3306 **מ-`sg-app`** — כל instance חדש עם ה-SG הזה מקבל גישה אוטומטית.
4. **Partition.** Cassandra מקבלת את מספר ה-partition כ-metadata ומפזרת replicas בין partitions.
   **לא Spread** — מוגבל ל-7 instances לכל AZ, ו-60 nodes לא נכנסים.
   **לא Cluster** — הוא דוחס הכול ל-AZ אחת ולחומרה סמוכה, כלומר **מגדיל** את הסיכון לכשל משותף.
5. `stop` מפסיק רק את חיוב ה-**compute**. ממשיכים לחייב על **EBS volumes** שמוקצים,
   על **snapshots**, ועל **Elastic IPs** שאינן מחוברת ל-instance רץ.
6. **EC2 Hibernate** — שומר את מצב ה-RAM ומחזיר אותו, כך שה-cache כבר חם.
   התנאים הקריטיים: **root volume מסוג EBS ומוצפן**, ו-**RAM קטן מ-150 GB**
   (ובנוסף: לא bare metal, ולא יותר מ-60 יום ב-hibernation).
7. **את ה-ENI.** מנתקים אותו מה-instance שנפל ומחברים ל-instance תקין —
   ה-private IP, ה-MAC וה-Security Groups נעים איתו. חשוב: **ENI לא עובר בין AZs**.
8. כי EIP בדרך כלל מסמנת תלות בכתובת קבועה שהיא החלטה ארכיטקטונית חלשה,
   ויש רק **5 לחשבון**. החלופות: **IP ציבורי רגיל + רשומת DNS**, או — העדיף —
   **Load Balancer מלפנים** וה-instances בלי IP ציבורי בכלל.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[06 - EC2 Pricing and Optimization]] · [[07 - Auto Scaling]] · [[08 - Elastic Load Balancing]] · [[09 - VPC Fundamentals]] · [[11 - VPC Security]] · [[19 - EBS and EC2 Storage]] · [[20 - EFS and File Storage]] · [[03 - IAM Fundamentals]] · [[26 - Containers]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 592–953, 1140–1420
