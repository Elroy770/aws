---
lesson: 39
title: Architecture Decision Making
domain: Design High-Performing Architectures
services: [Well-Architected Tool, Trusted Advisor, EC2, Lambda, S3, EFS, SQS, SNS, EventBridge, Kinesis]
tags: [saa-c03, methodology, decision-trees, well-architected, exam-strategy]
---

# 39 — Architecture Decision Making

> [!abstract] בשורה אחת
> במבחן יש כמעט תמיד **שתי תשובות שעובדות** — השיעור הזה הוא השיטה שמבדילה בין
> *technically possible* לבין **BEST**, ומזהה את מילת המפתח שפוסלת שלוש מתוך ארבע.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה "התשובה שעובדת" היא לא התשובה הנכונה |
| 2 | איך זה עובד | **השיטה בת 4 השלבים** לפענוח כל שאלת scenario |
| 3 | פירוק מפורט | **עצי החלטה**: Compute · Storage · Database · Integration · Networking · DR |
| 4 | עלות | איך משקללים עלות כשהיא **לא** הדרישה הקשיחה |
| 5 | השוואות | **טבלת מילות המפתח המגבילות** — הכלי החזק בשיעור |
| 6 | Well-Architected | ששת ה-Pillars ככלי הכרעה, לא כרשימת מכולת |
| 7 | מלכודות | "cheapest" ≠ "most cost-effective" |
| 8 | Scenario | פענוח שאלה אמיתית שלב אחר שלב |

**מונחי מפתח בשיעור:** `Hard Requirement` · `Elimination` · `Operational Overhead` · `BEST` · `Trade-off` · `Failure Domain`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- ב-SAA-C03 כמעט כל שאלה היא **scenario**, לא הגדרה. אף אחד לא ישאל "מה זה S3".
- לרוב **שתיים או שלוש** מהתשובות עובדות מבחינה טכנית. רק אחת היא ה-BEST.
- מי שלומד שירותים בלי שיטה — יודע מה כל שירות עושה, ועדיין נכשל בשאלות.
- הזמן לחוץ: ~2 דקות לשאלה. אין זמן לשקול כל אפשרות מאפס.

### מה השיעור פותר

- נותן **סדר קבוע** לפענוח שאלה, כך שהמוח לא מתחיל מאפס בכל פעם.
- מלמד לזהות את **הדרישה הקשיחה** — המילה או המשפט שפוסלים תשובות מיד.
- בונה **עצי החלטה** מוכנים לכל קטגוריה, כך ש"איזה compute" הופך לשאלה של שלוש שאלות כן/לא.
- מפריד בין **דרישה** (חובה) לבין **העדפה** (טיבר, מכריע רק בסוף).

> [!tip] האנלוגיה
> שאלת SAA היא לא חידה — היא **מכרז**. יש מפרט (הדרישות) ויש מציעים (התשובות).
> קודם פוסלים את מי שלא עומד במפרט, ורק בין הכשירים משווים מחיר ומורכבות.
> **מי שמתחיל מהמחיר — בוחר ספק שלא עומד במפרט.**

---

## 2. ⚙️ איך זה עובד — השיטה בת 4 השלבים

זו הליבה של השיעור. כל שאלת scenario עוברת דרך ארבעת השלבים האלה, באותו סדר.

```text
┌──────────────────────────────────────────────────────────────┐
│ שלב 1: זהה את הדרישה הקשיחה                                  │
│         מה חייב לקרות? מה אסור שיקרה?                        │
├──────────────────────────────────────────────────────────────┤
│ שלב 2: פסול מה שלא עומד בה                                   │
│         לרוב 2 מתוך 4 נופלות כאן                             │
├──────────────────────────────────────────────────────────────┤
│ שלב 3: השווה עלות ומורכבות בין הנותרים                       │
│         operational overhead · TCO · failure mode            │
├──────────────────────────────────────────────────────────────┤
│ שלב 4: בחר את ה-BEST                                         │
│         הפחות מורכב מבין אלה שעומדים בדרישה                  │
└──────────────────────────────────────────────────────────────┘
```

### שלב 1 — זהה את הדרישה הקשיחה

**קוראים את השאלה לפני התשובות.** תמיד.

מחפשים את המילים שהופכות "רצוי" ל-"חובה":

- `must` · `required` · `cannot` · `is not allowed` · `at all times` · `without`
- מספרים: `sub-millisecond` · `RTO of 15 minutes` · `99.99%` · `within 5 seconds`
- אילוצים ארגוניים: `no code changes` · `compliance` · `data must not leave the Region`

**הכלל:** אם השאלה מגדירה מספר או שוללת משהו — **זו הדרישה הקשיחה**, וממנה מתחילים.

### שלב 2 — פסול מה שלא עומד בה

לא מחפשים את התשובה הנכונה — **מחפשים למה כל תשובה שגויה**.

| סוג הפסילה | דוגמה |
|---|---|
| **טכנית בלתי אפשרי** | "Attach one EBS volume to instances in two AZs" |
| **לא עומד בדרישה** | דורשים RTO של דקות, והתשובה מציעה Backup & Restore |
| **פותר בעיה אחרת** | דורשים decoupling, והתשובה מציעה להגדיל instance |
| **נכון אבל חלקי** | "Attach an IGW" — נכון, אבל חסר ה-Route Table |
| **מוסיף overhead מיותר** | בונים תור בעצמם על EC2 במקום SQS |

> [!info] הכלל שמכפיל את הדיוק
> אם פסלתם שתי תשובות בביטחון, הסיכוי בניחוש קפץ מ-25% ל-50%.
> **גם כשלא יודעים — תמיד פוסלים לפני שמנחשים.**

### שלב 3 — השווה עלות ומורכבות

בין התשובות ששרדו, שואלים בסדר הזה:

1. **Operational overhead** — מי מתחזק את זה? כמה patching, scaling, monitoring?
2. **Failure mode** — מה קורה כשרכיב אחד נופל? האם יש single point of failure?
3. **עלות כוללת (TCO)** — לא מחיר ה-instance, אלא גם תעבורה, requests, וזמן אנשים.
4. **מהירות היישום** — האם הפתרון דורש כתיבת קוד או קונפיגורציה?

### שלב 4 — בחר את ה-BEST

> [!tip] כלל ההכרעה
> **ה-BEST הוא הפתרון הפשוט ביותר שעומד בכל הדרישות הקשיחות.**
> לא הזול ביותר. לא המתוחכם ביותר. **הפשוט ביותר שעובד במפרט.**
> אם שתי תשובות עומדות בדרישות — מנצחת זו עם פחות רכיבים לתחזק.

---

## 3. 🔍 פירוק מפורט — עצי ההחלטה

### 3.1 עץ החלטה: Compute

```text
צריך להריץ קוד. איזה compute?

├─ האם זה job שרץ ומסתיים (batch, ETL, רינדור)?
│    └─ כן → AWS Batch
│             (מנהל תור, מקצה EC2/Fargate, תומך multi-node parallel jobs)
│
├─ האם ה-workload הוא event-driven / קצר / משתנה מאוד?
│    ├─ כן, וזמן ריצה קצר, אין state, לא צריך שליטה על ה-OS
│    │    └─ Lambda
│    └─ כן, אבל צריך container image או ריצה ארוכה
│         └─ ECS/EKS on Fargate
│
├─ האם יש כבר containers, או צריך שליטה על התלויות?
│    ├─ רוצים אפס ניהול שרתים → Fargate
│    └─ צריך שליטה על ה-hosts, GPU, או instance types מיוחדים
│         └─ ECS/EKS on EC2
│
└─ צריך שליטה מלאה על ה-OS, רישוי מיוחד, או תוכנה שלא עוברת containerization?
     └─ EC2
```

**טבלת ההכרעה בין החמישה:**

| | **Lambda** | **Fargate** | **ECS/EKS on EC2** | **EC2** | **Batch** |
|---|---|---|---|---|---|
| מנהל שרתים | **לא** | **לא** | כן | כן | לא (מנהל בשבילכם) |
| Operational overhead | **הנמוך ביותר** | נמוך | בינוני | **הגבוה ביותר** | נמוך |
| זמן ריצה מקסימלי | **15 דקות** | ללא הגבלה | ללא הגבלה | ללא הגבלה | ללא הגבלה |
| מודל תמחור | לכל request + משך | vCPU/GB לשנייה | לפי ה-EC2 | לפי שעה/שנייה | לפי ה-compute מתחת |
| Cold start | **כן** | מינימלי | לא | לא | לא רלוונטי |
| מתי הוא ה-BEST | תעבורה spiky, אירועים, glue code | containers בלי ניהול צי | צריך שליטה על ה-hosts | legacy, רישוי, GPU ייחודי | עבודות תור ארוכות |

**מילות המפתח שמכריעות:**

- "serverless" / "no servers to manage" → **Lambda או Fargate**
- "runs for hours" → **לא Lambda** (תקרת 15 דקות)
- "existing Docker images" → **ECS/EKS/Fargate**
- "must control the operating system" / "specific kernel" → **EC2**
- "thousands of batch jobs" / "job queue" → **Batch**
- "spiky, unpredictable traffic, pay nothing when idle" → **Lambda**

### 3.2 עץ החלטה: Storage

```text
איפה שמים את הדאטה?

├─ האם זו גישת אובייקטים דרך HTTP API (תמונות, לוגים, גיבויים)?
│    ├─ כן → S3
│    │        └─ גישה נדירה? → S3 IA / Glacier לפי מהירות השליפה הנדרשת
│    └─ לא ↓
│
├─ האם צריך filesystem שמשותף לכמה שרתים בו-זמנית?
│    ├─ כן, Linux / POSIX ──────────────► EFS
│    ├─ כן, Windows / SMB / Active Directory ─► FSx for Windows File Server
│    ├─ כן, HPC / מיליוני IOPS ────────► FSx for Lustre
│    ├─ כן, תאימות מקסימלית ל-OS ──────► FSx for NetApp ONTAP
│    └─ כן, ZFS מנוהל ─────────────────► FSx for OpenZFS
│
├─ האם זה דיסק ל-instance אחד, שצריך לשרוד reboot והפסקה?
│    └─ כן → EBS
│             (io2 Block Express — עד 256,000 IOPS)
│
└─ האם זה cache/scratch זמני שדורש IOPS מקסימלי ומותר שייעלם?
     └─ כן → Instance Store
              (מיליוני IOPS, latency הנמוך ביותר, נמחק עם ה-instance)
```

**טבלת ההכרעה:**

| | **S3** | **EBS** | **EFS** | **FSx for Lustre** | **Instance Store** |
|---|---|---|---|---|---|
| סוג | אובייקטים | בלוקים | קבצים (NFS) | קבצים מבוזרים | בלוקים מקומיים |
| כמה instances במקביל | ללא הגבלה (API) | **אחד** בכל רגע | **הרבה** | הרבה | **אחד**, פיזית |
| Scope | Region | **AZ** | **Region** (Multi-AZ) | AZ | ה-instance עצמו |
| שורד סיום instance | כן | כן | כן | כן | **לא** |
| IOPS | לא רלוונטי | עד 256,000 (io2 BE) | לפי גודל או provisioned | **מיליונים** | **מיליונים** |
| מילת מפתח בשאלה | "objects", "static website", "archive" | "boot volume", "single database" | "shared across instances", "POSIX" | "HPC", "backed by S3" | "temporary", "highest IOPS", "cache" |

### 3.3 עץ החלטה: Database

זהו הסיכום המהיר. הפירוט המלא נמצא ב-[[24 - Database Selection]].

```text
איזה DB?

├─ יש joins, טרנזקציות, schema קשיח? (OLTP / SQL)
│    ├─ צריך תאימות לאופן שבו העסק עובד היום → RDS
│    └─ צריך ביצועים ו-scale גבוהים יותר, cloud-native → Aurora
│
├─ אין joins, צריך scale אופקי ו-latency קבוע?
│    ├─ document / key-value בקנה מידה עצום → DynamoDB
│    │       └─ צריך microseconds לקריאה? → DynamoDB + DAX
│    ├─ MongoDB API → DocumentDB
│    └─ Cassandra API → Keyspaces
│
├─ Cache בזיכרון (sub-millisecond)?
│    ├─ צריך persistence, replicas, pub/sub → ElastiCache for Redis
│    └─ cache פשוט, multi-threaded, בלי persistence → ElastiCache for Memcached
│
├─ Analytics / BI / OLAP על היסטוריה?
│    ├─ Data Warehouse עם SQL → Redshift
│    ├─ שאילתות ad-hoc ישירות על S3 → Athena
│    └─ Big Data frameworks (Spark/Hadoop) → EMR
│
├─ חיפוש חופשי בטקסט לא מובנה → OpenSearch
├─ קשרים בין ישויות (רשת חברתית, הונאות) → Neptune
├─ Ledger בלתי ניתן לשינוי עם היסטוריה מאומתת → QLDB
└─ סדרות עתיות (IoT, מטריקות) → Timestream
```

**השאלות שהקורס מציע לשאול לפני שבוחרים DB:**

| השאלה | למה היא משנה |
|---|---|
| Read-heavy, write-heavy או מאוזן? מה ה-throughput? | קובע אם צריך replicas, cache או DB שמתרחב אופקית |
| כמה דאטה ולכמה זמן? מה גודל האובייקט הממוצע? | קובע בין DB לבין object store |
| מה דרישת ה-durability? האם זה **source of truth**? | cache לעולם לא יהיה source of truth |
| מה דרישת ה-latency? כמה משתמשים במקביל? | sub-millisecond פוסל כל DB דיסקי |
| מה מודל הדאטה? יש joins? מובנה או חצי-מובנה? | joins → SQL. חצי-מובנה → NoSQL |
| Schema קשיח או גמישות? צריך reporting? search? | reporting → Redshift/Athena. search → OpenSearch |
| כמה עולה הרישוי? האם לעבור ל-cloud native? | Oracle/SQL Server יקרים; Aurora חוסך רישוי |

### 3.4 עץ החלטה: Integration ו-Messaging

```text
איך רכיבים מדברים ביניהם?

├─ צריך תור עמיד שמבטיח שהודעה תעובד פעם אחת על ידי consumer אחד?
│    └─ SQS
│         ├─ צריך סדר מדויק וללא כפילויות → SQS FIFO
│         └─ throughput מקסימלי, סדר לא קריטי → SQS Standard
│
├─ צריך שהודעה אחת תגיע לכמה נמענים במקביל (fan-out)?
│    └─ SNS
│         └─ ורוצים גם עמידות לכל נמען → SNS + SQS Fan-Out
│                (כל subscriber מקבל תור משלו)
│
├─ צריך ניתוב לפי תוכן האירוע, או אירועים משירותי AWS / SaaS?
│    └─ EventBridge
│         (rules לפי pattern, schema registry, scheduling)
│
├─ צריך זרם רציף בנפח גבוה, עם שמירת סדר לפי partition
│  ואפשרות שכמה consumers יקראו את אותו דאטה שוב?
│    └─ Kinesis Data Streams
│
└─ צריך לתזמר תהליך רב-שלבי עם retries, branching ו-state?
     └─ Step Functions
```

**טבלת ההכרעה:**

| | **SQS** | **SNS** | **EventBridge** | **Kinesis** |
|---|---|---|---|---|
| דגם | תור, pull | pub/sub, push | event bus, ניתוב לפי כלל | stream |
| כמה consumers להודעה | **אחד** | **הרבה** | הרבה, לפי rules | הרבה, **קוראים במקביל** |
| ההודעה נשמרת אחרי קריאה | לא (נמחקת) | לא | לא | **כן** — לפי retention |
| קריאה חוזרת של אותו דאטה | לא | לא | לא | **כן** (replay) |
| שמירת סדר | FIFO בלבד | לא | לא | **לפי partition key** |
| מילת מפתח | "decouple", "buffer", "process once" | "notify multiple", "fan out" | "route based on event content", "SaaS integration", "scheduled" | "real-time streaming", "replay", "ordered by device/user" |

### 3.5 עץ החלטה: Networking וגישה פרטית

```text
איך משאב מגיע ליעד?

├─ EC2 בסאבנט פרטי צריך להגיע ל-S3 או DynamoDB?
│    └─ Gateway Endpoint  (חינם — לא NAT!)
│
├─ EC2 בסאבנט פרטי צריך להגיע לשירות AWS אחר?
│    └─ Interface Endpoint (PrivateLink) — חיוב לשעה לכל AZ + GB
│
├─ EC2 בסאבנט פרטי צריך לצאת לאינטרנט הכללי (עדכונים, API חיצוני)?
│    └─ NAT Gateway  (שעות + GB מעובד)
│
├─ צריך לחבר שני VPCs?
│    ├─ שניים בלבד → VPC Peering (לא טרנזיטיבי!)
│    └─ הרבה, בטופולוגיית hub → Transit Gateway
│
└─ צריך לחבר on-premises?
     ├─ מהר, זול, מעל האינטרנט → Site-to-Site VPN
     └─ פרטי, יציב, throughput צפוי → Direct Connect (+VPN כגיבוי)
```

### 3.6 עץ החלטה: חסימת כתובת IP

זה נושא שמופיע כשאלה עצמאית, כי התשובה **משתנה לפי הארכיטקטורה**.

| הארכיטקטורה | מה רואה את ה-IP של הלקוח | איפה חוסמים |
|---|---|---|
| **EC2 עם public IP, בלי LB** | ה-EC2 עצמו | **NACL** (יש בו deny) · או firewall על ה-instance. **SG לא יכול** |
| **ALB לפני EC2 פרטי** | ה-ALB בלבד — הוא **מסיים את החיבור** | **NACL** של הסאבנט הציבורי, או **WAF** על ה-ALB |
| **NLB לפני EC2 פרטי** | ה-NLB מעביר את ה-IP המקורי | **NACL** של הסאבנט הציבורי. **WAF לא נתמך על NLB** |
| **ALB + WAF** | WAF רואה את ה-IP | **WAF IP filtering** — הפתרון הנקי |
| **CloudFront + WAF + ALB** | CloudFront הוא מי שמגיע ל-ALB | **WAF ברמת CloudFront**. ה-**NACL של ה-VPC כבר לא עוזר** — הוא יראה רק IPs של CloudFront |

> [!warning] הנקודה שנשאלת
> ברגע ש-**CloudFront** נכנס לתמונה, ה-NACL של ה-VPC **מפסיק להיות רלוונטי** לחסימת לקוחות —
> ה-VPC רואה רק את כתובות CloudFront. החסימה חייבת לקרות **בקצה**: WAF ב-CloudFront,
> או **Geo Restriction** של CloudFront לחסימה לפי מדינה.
>
> ועוד כלל: **Security Group לא יכול לחסום** — יש בו רק כללי **allow**. לחסימה מפורשת צריך **NACL** או **WAF**.

### 3.7 עץ החלטה: Caching — איפה שמים את ה-cache

הקורס מציג את שרשרת ה-caching מהקצה ועד ל-DB. כל שכבה חוסכת משהו אחר.

```text
Client
  │
  ├─► CloudFront (edge)      → חוסך: רשת + latency גיאוגרפי. TTL על תוכן סטטי ודינמי
  │
  ├─► API Gateway caching    → חוסך: קריאות ל-backend על תשובות זהות
  │
  ├─► App logic (EC2/Lambda) → כאן מחליטים מה בכלל צריך לחשב
  │      │
  │      ├─► ElastiCache (Redis/Memcached) → חוסך: קריאות ל-DB, חישוב חוזר
  │      └─► DAX                            → חוסך: latency של DynamoDB (microseconds)
  │
  └─► Database / S3          → המקור. הכי איטי והכי יקר לפנייה
```

**ארבעת המימדים שכל שכבת cache משפיעה עליהם:** **Network** · **Computation** · **Cost** · **Latency**.
ה-**TTL** הוא הידית שמאזנת בין טריות הדאטה לבין חיסכון.

### 3.8 עץ החלטה: DR ו-High Availability

הפירוט המלא ב-[[34 - Disaster Recovery]]; זו רק ההכרעה המהירה.

```text
מה ה-RTO וה-RPO?

├─ RTO/RPO של שעות עד ימים, עלות מינימלית → Backup & Restore
├─ RTO של עשרות דקות, גרסה מוקטנת רצה     → Pilot Light
├─ RTO של דקות, סביבה מלאה בקנה מידה קטן  → Warm Standby
└─ RTO/RPO כמעט אפס, שתי סביבות פעילות    → Multi-Site / Active-Active
```

> [!warning] הבחנה שנופלים עליה
> **Multi-AZ אינו DR.** הוא מגן מפני כשל **AZ**, לא מפני אובדן **Region** או מחיקת דאטה בטעות.
> **Read Replica אינה Multi-AZ** — היא נועדה ל-scaling של קריאות, וה-failover אליה **ידני**.

---

## 4. 💰 עלות ותמחור — איך משקללים עלות בהחלטה

### על מה מחייבים — הרכיבים ששוכחים

| רכיב חיוב | איך נמדד | למה שוכחים אותו |
|---|---|---|
| **Data transfer OUT** | GB לאינטרנט | לא מופיע בטבלת המחירים של השירות |
| **Cross-AZ traffic** | GB בשני הכיוונים | Multi-AZ "בחינם" עד שרואים את החשבון |
| **Requests** | לכל מיליון (S3, Lambda, API GW, DynamoDB) | ב-scale גבוה זה עולה על ה-compute |
| **NAT Gateway** | שעות + GB **מעובד** | ה-GB המעובד לרוב גדול מהשעות |
| **Provisioned capacity** | מוקצה, לא מנוצל | Provisioned IOPS, DynamoDB provisioned, endpoints |
| **זמן אנשים** | לא בחשבונית בכלל | הרכיב היקר ביותר, ולא נמדד |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **Serverless** (Lambda, Fargate, DynamoDB On-Demand) | **0 ב-idle**, יקר יחסית ליחידה בעומס גבוה | תעבורה spiky, בלתי צפויה, או נמוכה |
| **EC2 On-Demand** | בינונית, גמישה | עומס משתנה שרץ שעות |
| **Savings Plans / RI** | **עד ~72% הנחה** | עומס קבוע וידוע לשנה+ |
| **Spot** | **עד ~90% הנחה** | Batch, HPC, כל מה שסובל הפסקה |
| **Gateway Endpoint** | **0** | תמיד עדיף על NAT ל-S3/DynamoDB |
| **NAT Gateway** | גבוהה | רק כשצריך יציאה כללית לאינטרנט |
| **Multi-Site DR** | **היקרה ביותר** | רק כש-RTO קרוב לאפס הוא דרישה עסקית |

### 🚩 עלויות נסתרות

- **NAT Gateway לגישה ל-S3** — הטעות היקרה והנפוצה ביותר. **Gateway Endpoint חינם.**
- **Cross-AZ בין שכבות** — ALB ב-AZ אחת ל-EC2 באחרת, בכל בקשה.
- **CloudWatch Logs מפורטים מדי** — DEBUG בפרודקשן הוא דליפה שקטה.
- **Provisioned throughput לא מנוצל** — DynamoDB ב-Provisioned כשהעומס לא צפוי.
- **Snapshots שנצברים** — גיבויים בלי lifecycle policy מצטברים לנצח.
- **Idle resources** — dev environment שרץ בסופ"ש, EIPs יתומות, load balancers ללא targets.

### 💡 טיפים לחיסכון

- **תמיד Gateway Endpoint** ל-S3 ו-DynamoDB במקום NAT.
- **התאם את מודל התמחור לפרופיל העומס** — קבוע→SP/RI, מקוטע→Spot, spiky→Serverless.
- **שמור תעבורה בתוך AZ** כשאין דרישת HA שמחייבת אחרת.
- **CloudFront לפני origin** — חוסך גם egress וגם compute.
- **Lifecycle policies** על S3 ועל snapshots — אוטומטי, פעם אחת.
- **Trusted Advisor** נותן המלצות cost optimization ישירות; ראו סעיף 6.

---

## 5. ⚖️ טבלת מילות המפתח המגבילות

זו הטבלה החזקה בשיעור. כל שורה כאן פוסלת תשובות **לפני** שהשווית ביניהן.

| מילת המפתח בשאלה | הדרישה הקשיחה | מה נפסל מיד | לאן זה מוביל |
|---|---|---|---|
| **"must be serverless"** | אפס שרתים לניהול | כל תשובה עם EC2, ECS on EC2, ElastiCache | Lambda · Fargate · DynamoDB · S3 · API Gateway |
| **"minimal / least operational overhead"** | הכי פחות תחזוקה | פתרונות self-managed על EC2 | השירות המנוהל ביותר שעומד בדרישה |
| **"lowest cost" / "most cost-effective"** | עלות היא המכריע — **אחרי** שהדרישות מולאו | פתרון שלא עומד ב-RTO/אבטחה גם אם זול | Spot · S3 Glacier · Gateway Endpoint · serverless ב-idle |
| **"no code changes" / "without modifying the application"** | האפליקציה נשארת כמו שהיא | Refactor, Lambda, שכתוב ל-DynamoDB | Rehost · Replatform · ElastiCache · EFS |
| **"sub-millisecond latency"** | זיכרון, לא דיסק | RDS · DynamoDB לבדו · S3 | **ElastiCache** · **DAX** (microseconds) |
| **"single-digit millisecond"** | ביצועים גבוהים אבל לא זיכרון | RDS בעומס גבוה | **DynamoDB** |
| **"must survive an AZ failure"** | יתירות בין AZs | single instance · Instance Store · EBS יחיד | **Multi-AZ** · ASG על כמה AZs · EFS |
| **"must survive Region loss"** | יתירות בין Regions | Multi-AZ | **Cross-Region Replication** · Global Tables · DR strategy |
| **"durable" / "must not lose messages"** | עמידות ההודעה | SNS לבדו · Instance Store · in-memory queue | **SQS** · S3 · EBS |
| **"exactly once" / "in order"** | סדר וייחודיות | SQS Standard · SNS | **SQS FIFO** · Kinesis (לפי partition) |
| **"compliance" / "data must not leave"** | גבול פיזי או משפטי | כל פתרון שמעביר דאטה החוצה | **Outposts** · encryption + KMS · Region ספציפי |
| **"private, no internet"** | אין מסלול ציבורי | IGW · public subnet · NAT ל-S3 | **VPC Endpoints** · PrivateLink |
| **"global users, low latency"** | הפצה גיאוגרפית | origin יחיד | **CloudFront** · **Global Accelerator** · Route 53 latency routing |
| **"decouple"** | הרכיבים לא יודעים זה על זה | קריאה סינכרונית ישירה | **SQS** · SNS · EventBridge |
| **"spiky / unpredictable traffic"** | scale אלסטי | קיבולת מוקצית קבועה | **Lambda** · DynamoDB **On-Demand** · ASG |
| **"read-heavy, database is overloaded"** | הורדת עומס קריאה | להגדיל את ה-instance | **Read Replicas** · **ElastiCache** |
| **"encrypt existing unencrypted volume"** | הצפנה בדיעבד | "enable encryption" ישירות — לא קיים | Snapshot → **copy מוצפן** → volume חדש |
| **"cannot tolerate downtime during migration"** | rehearsal + cutover קצר | העתקה חד-פעמית | **DMS + CDC** · Blue/Green |
| **"highest possible IOPS"** | ביצועי דיסק קיצוניים | EBS gp3 · EFS | **Instance Store** · **FSx for Lustre** · io2 Block Express |
| **"tightly coupled HPC nodes"** | latency בין nodes | instances מפוזרים | **Cluster Placement Group** + **EFA** |

> [!info] שורה תחתונה
> **"Most cost-effective" אינו "cheapest".** הוא אומר: *הזול ביותר מבין אלה שעומדים בכל הדרישות.*
> תשובה זולה שמפרה דרישה קשיחה היא **תמיד שגויה**, גם אם היא הזולה בטבלה.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars ככלי הכרעה

### עקרונות ההנחיה הכלליים

אלה העקרונות שה-Framework בנוי עליהם, והם גם עצות טובות לפענוח שאלות:

- **Stop guessing your capacity needs** — אלסטיות במקום ניחוש. זו הסיבה ש-ASG ו-serverless מנצחים.
- **Test systems at production scale** — בענן זה זול, כי מכבים אחרי הבדיקה.
- **Automate to make architectural experimentation easier** — IaC מאפשר לנסות ולחזור.
- **Allow for evolutionary architectures** — הארכיטקטורה משתנה עם הדרישות, לא נקבעת פעם אחת.
- **Drive architectures using data** — מחליטים לפי מטריקות, לא לפי תחושה.
- **Improve through game days** — מדמים כשלים ועומסי שיא **לפני** שהם קורים באמת.

> [!info] הנקודה שהקורס מדגיש
> ששת ה-Pillars **אינם trade-offs שצריך לאזן ביניהם — הם סינרגיה.**
> ארכיטקטורה מאובטחת היא בדרך כלל גם אמינה יותר, ואוטומציה משפרת גם תפעול וגם עלות.

### הפילרים כשאלות הכרעה

| Pillar | השאלה שהוא שואל **על ההחלטה שלכם** | פעולה קונקרטית בשאלת scenario |
|---|---|---|
| **Operational Excellence** | מי מתחזק את זה ואיך יודעים שהוא שבור? | העדיפו managed; דרשו CloudWatch alarms; IaC במקום קליקים; runbook לכל כשל |
| **Security** | האם הרכיב חשוף יותר מהנדרש? | private subnet כברירת מחדל; IAM Roles ולא keys; הצפנה ב-rest וב-transit; least privilege |
| **Reliability** | מה קורה כשרכיב אחד נופל? | לפחות 2 AZs; health checks; retries עם backoff; אין single point of failure |
| **Performance Efficiency** | האם בחרתם את הכלי הנכון לעומס הנכון? | cache בשכבה הנכונה; DB שמתאים למודל הדאטה; instance family שמתאים ל-bottleneck |
| **Cost Optimization** | האם משלמים על מה שלא מנוצל? | התאמת מודל תמחור לעומס; Gateway Endpoints; lifecycle; כיבוי סביבות dev |
| **Sustainability** | האם יש כאן עבודה מיותרת? | scale-to-demand; מזעור data transfer; Region יעיל; לא לאחסן מה שלא צריך |

### שני הכלים שבודקים ארכיטקטורה בפועל

| | **Well-Architected Tool** | **Trusted Advisor** |
|---|---|---|
| מה זה | כלי **חינמי** לסקירת workload מול ששת ה-Pillars | הערכה אוטומטית **ברמת החשבון** |
| איך עובד | בוחרים workload, עונים על שאלון, מקבלים ניתוח | **אין מה להתקין** — סורק את החשבון ומייצר המלצות |
| מה מקבלים | ייעוץ, סרטונים, תיעוד, **דוח** ו-dashboard | המלצות ב-**6 קטגוריות** |
| הקטגוריות | ששת ה-Pillars | **Cost Optimization · Performance · Security · Fault Tolerance · Service Limits · Operational Excellence** |
| מגבלת תוכנית | חינמי לכולם | **סט הבדיקות המלא רק ב-Business ו-Enterprise Support** |
| גישה תוכניתית | דרך ה-console/API | **AWS Support API** |

> [!warning] ההבחנה שנשאלת
> **Well-Architected Tool = סקירה של ארכיטקטורה** לפי שאלון שאתם ממלאים.
> **Trusted Advisor = סריקה אוטומטית של החשבון** בלי שאלון.
> אם השאלה מזכירה **"service limits"** או **"check my account for unused resources"** —
> זה **Trusted Advisor**. אם היא מזכירה **"review the workload against best practices"** — זה **WA Tool**.

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "which is the MOST cost-effective solution that meets the requirements" | הזול **מבין הכשירים** — קודם לפסול, אז להשוות |
| "least operational overhead" | השירות המנוהל ביותר, לא בהכרח הזול |
| "check the account for underutilized resources" | **Trusted Advisor** |
| "review the workload against the six pillars" | **Well-Architected Tool** |
| "approaching a service limit" | **Trusted Advisor** — Service Limits היא אחת מ-6 הקטגוריות |
| "block a specific IP" — ויש CloudFront | **WAF ברמת CloudFront**, לא NACL |
| "block a country" | CloudFront **Geo Restriction** |
| "tightly coupled nodes, MPI" | **EFA** + **Cluster Placement Group** |
| "millions of IOPS from a shared file system" | **FSx for Lustre** |
| "run thousands of jobs across many instances" | **AWS Batch** (multi-node parallel jobs) |
| "deploy an HPC cluster from config files" | **AWS ParallelCluster** |

### טעויות נפוצות

> [!warning] מלכודת 1 — לקרוא את התשובות לפני השאלה
> **הניסוח:** שאלה ארוכה עם ארבע תשובות שכולן נשמעות סבירות.
> **הטעות:** לסרוק את התשובות ולחפש "מה מוכר לי".
> **הנכון:** לקרוא את השאלה עד הסוף, **לנסח לעצמכם את הדרישה הקשיחה**,
> ורק אז להסתכל בתשובות. אחרת התשובות מכתיבות לכם את הפרשנות.

> [!warning] מלכודת 2 — "Cheapest" במקום "most cost-effective"
> **הניסוח:** "Which is the MOST cost-effective solution that meets the RTO of 10 minutes?"
> **הטעות:** לבחור Backup & Restore כי הוא הזול.
> **הנכון:** Backup & Restore **לא עומד ב-RTO של 10 דקות** — הוא נפסל בשלב 2.
> "Cost-effective" מכריע **רק בין הכשירים**. כנראה **Warm Standby**.

> [!warning] מלכודת 3 — לבחור את השירות המתוחכם ביותר
> **הניסוח:** "Static website for a marketing campaign, minimal cost and overhead."
> **הטעות:** לבנות ALB + ASG + EC2, כי זו ארכיטקטורה "רצינית".
> **הנכון:** **S3 Static Website + CloudFront.** אפס שרתים, אפס תחזוקה, עלות מינימלית.
> **הפשוט ביותר שעומד בדרישה מנצח.**

> [!warning] מלכודת 4 — Security Group כמנגנון חסימה
> **הניסוח:** "Block a malicious IP address using a security group deny rule."
> **הטעות:** להניח ש-SG הוא firewall דו-כיווני מלא.
> **הנכון:** **ב-SG יש רק כללי allow.** אין בו deny. לחסימה מפורשת של IP —
> **NACL** (יש בו deny) או **WAF** (IP filtering).

> [!warning] מלכודת 5 — לבלבל Multi-AZ עם DR
> **הניסוח:** "We have RDS Multi-AZ, so we're protected against a regional outage."
> **הטעות:** להתייחס ל-Multi-AZ כאסטרטגיית DR.
> **הנכון:** Multi-AZ מגן מפני כשל **AZ** בלבד, ובאותו Region.
> להגנה מפני אובדן Region צריך **Cross-Region** — replica, snapshot copy או Global Tables.

> [!warning] מלכודת 6 — להתעלם ממילת "without"
> **הניסוח:** "...**without requiring changes to the application code**."
> **הטעות:** לבחור פתרון serverless אלגנטי שדורש שכתוב.
> **הנכון:** המילה `without` היא **דרישה קשיחה**. היא פוסלת מיד כל refactor.
> הכיוון הוא ElastiCache, EFS, Read Replica — דברים שמתחברים מתחת לאפליקציה.

> [!warning] מלכודת 7 — NAT Gateway כדי להגיע ל-S3
> **הניסוח:** "Private EC2 instances need to read objects from S3 at the lowest cost."
> **הטעות:** NAT Gateway — הוא אכן יעבוד.
> **הנכון:** **S3 Gateway Endpoint** — **עלות אפס**, לא יוצא לאינטרנט כלל, ומסלול פרטי לגמרי.
> NAT נשאר נכון רק ליעדים **שאינם** S3/DynamoDB.

---

## 8. 🏗️ Scenario מהעולם האמיתי — פענוח שלב אחר שלב

**השאלה:**

> חברת ביוטכנולוגיה מריצה סימולציות גנומיות. כל job רץ על **מאות nodes שמתקשרים ביניהם בצפיפות**
> ודורש **מערכת קבצים משותפת במיליוני IOPS**. ה-jobs רצים בגלים — לפעמים אין כלום ליומיים,
> ולפעמים 500 nodes במקביל. הדאטה הגולמי (עשרות TB) כבר יושב ב-S3.
> ה-jobs **סובלים הפסקה ואפשר להריץ אותם מחדש**.
> מהו הפתרון היעיל ביותר מבחינת ביצועים **ובעלות הנמוכה ביותר**?

**שלב 1 — הדרישות הקשיחות:**

| הדרישה בשאלה | מה היא מחייבת |
|---|---|
| "nodes שמתקשרים ביניהם בצפיפות" | **latency נמוך בין nodes** → placement + networking מיוחדים |
| "מיליוני IOPS ממערכת קבצים משותפת" | **לא EBS ולא EFS** — הם לא מגיעים לשם |
| "הדאטה כבר ב-S3" | היתרון הולך למי שמשתלב עם S3 |
| "רץ בגלים, לפעמים אפס" | **אסור קיבולת קבועה** |
| "סובל הפסקה" | **מותר Spot** |
| "העלות הנמוכה ביותר" | הטיבר הסופי |

**שלב 2 — מה נפסל:**

| נפסל | למה |
|---|---|
| EFS למערכת הקבצים | לא מגיע למיליוני IOPS ל-HPC; לא מגובה S3 |
| EBS משותף | EBS משרת **instance אחד** בכל רגע |
| On-Demand קבוע ל-500 nodes | יושב בטל בין הגלים — פוסל את "העלות הנמוכה" |
| Lambda | תקרת 15 דקות, אין MPI, אין filesystem משותף |
| Instances מפוזרים על כמה AZs | הורס את ה-latency בין ה-nodes |

**הארכיטקטורה:**

```text
                 S3 (הדאטה הגולמי, עשרות TB)
                      │  lazy load / export
                      ▼
        ┌──── FSx for Lustre ────┐   ← מיליוני IOPS, מגובה S3
        │                        │
   ┌────┴────────────────────────┴────┐
   │  Cluster Placement Group (AZ אחת) │
   │  ┌────┐ ┌────┐ ┌────┐  ... ┌────┐│
   │  │EC2 │─│EC2 │─│EC2 │      │EC2 ││  ← Spot Instances, CPU/GPU optimized
   │  └────┘ └────┘ └────┘      └────┘│
   │     └── EFA (MPI, bypass OS) ────┘
   └───────────────┬───────────────────┘
                   │
             AWS Batch  ← מנהל את התור, מקצה ומכבה nodes
                   │
        (או AWS ParallelCluster להקמה מקבצי קונפיגורציה)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **FSx for Lustre** | מערכת הקבצים המבוזרת היחידה שנותנת **מיליוני IOPS** ל-HPC, והיא **מגובה S3** — הדאטה כבר שם |
| **Cluster Placement Group** | ממקם את כל ה-instances על **אותו rack באותה AZ** — רשת 10Gbps ו-latency מינימלי בין nodes |
| **EFA (Elastic Fabric Adapter)** | הדרישה היא **tightly coupled** ו-MPI. EFA עוקף את ה-Linux OS ונותן transport אמין ב-latency נמוך. **Linux בלבד** |
| **EC2 CPU/GPU optimized** | סימולציות גנומיות הן compute-bound; בוחרים משפחה שמתאימה ל-bottleneck |
| **Spot Instances + Spot Fleet** | ה-jobs **סובלים הפסקה** — זו בדיוק ההגדרה של Spot. **עד ~90% הנחה** |
| **AWS Batch** | מנהל תור, מקצה instances לפי הצורך ומכבה כשנגמר. תומך ב-**multi-node parallel jobs** |
| **ParallelCluster כחלופה** | אם הצוות רוצה סביבת HPC קלאסית מוגדרת בקבצי טקסט, כולל הפעלת EFA |

**למה לא Enhanced Networking רגיל במקום EFA?**
Enhanced Networking (SR-IOV) עם **ENA** נותן עד 100 Gbps — מצוין ל-throughput.
אבל **EFA** הוא ENA משופר שנועד ל-**inter-node communication** ב-MPI:
הוא עוקף את מערכת ההפעלה ומוריד latency בצורה שקריטית ל-workload **tightly coupled**.

**למה לא לפזר על כמה AZs לזמינות?**
כי הדרישה הקשיחה כאן היא **latency בין nodes**, ו-cross-AZ הורס אותה.
ה-jobs ניתנים להרצה מחדש, אז אובדן AZ הוא לא אסון — פשוט מריצים שוב.
**כאן Performance Efficiency גובר על Reliability, כי הדרישה העסקית אמרה כך.**

**למה לא S3 ישירות במקום Lustre?**
S3 הוא object store, **לא filesystem**. קוד HPC מצפה ל-POSIX ולגישה מקבילה ברמת בלוקים.
Lustre נותן בדיוק את זה, וטוען את הדאטה מ-S3 מאחורי הקלעים.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **קטלוג שירותים מלא** של AWS. יש מאות שירותים; המבחן חוזר על אותם ~40.
- **פרמטרי קונפיגורציה מדויקים** של כל שירות.
- **תחביר של ParallelCluster** או קבצי הקונפיגורציה שלו — מספיק לדעת מה הוא ומתי.
- **רשימת הבדיקות המלאה** של Trusted Advisor. מספיק **6 הקטגוריות**.
- **המבנה הפנימי של MPI** — מספיק לזהות "MPI / tightly coupled" → **EFA**.
- **תוכן ה-Whitepapers מילה במילה.** מספיק העקרונות ו-6 הפילרים.
- **Intel 82599 VF** — legacy. מספיק לדעת ש-**ENA** הוא הסטנדרט (עד 100 Gbps).

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **השיטה:** דרישה קשיחה → פסילה → השוואת עלות/מורכבות → **הפשוט ביותר שעומד בדרישה**.
- **קוראים את השאלה לפני התשובות.** תמיד.
- **מחפשים למה כל תשובה שגויה**, לא למה אחת נכונה.
- **`must` · `cannot` · `without` · מספרים** = דרישה קשיחה. הכל השאר טיבר.
- **"Most cost-effective" = הזול מבין הכשירים.** לא הזול בטבלה.
- **"Least operational overhead" = השירות המנוהל ביותר** שעומד בדרישה.
- **"Serverless"** פוסל כל תשובה עם EC2 או ElastiCache.
- **"No code changes"** פוסל כל refactor — הכיוון הוא ElastiCache/EFS/Replica.
- **Compute:** Lambda(<15 דק') · Fargate(containers בלי ניהול) · EC2(שליטה ב-OS) · Batch(תור).
- **Storage:** S3(אובייקטים) · EBS(instance אחד) · EFS(משותף POSIX) · FSx Lustre(HPC) · Instance Store(זמני, IOPS מקסימלי).
- **Integration:** SQS(תור, consumer אחד) · SNS(fan-out) · EventBridge(ניתוב לפי תוכן) · Kinesis(stream עם replay).
- **Kinesis הוא היחיד שמאפשר לקרוא את אותו דאטה שוב.**
- **sub-millisecond → ElastiCache. microseconds ל-DynamoDB → DAX. single-digit ms → DynamoDB.**
- **SG לא חוסם** — יש בו רק allow. חסימה = **NACL** או **WAF**.
- **יש CloudFront? ה-NACL לא רלוונטי לחסימת לקוחות** — חוסמים ב-WAF/Geo Restriction בקצה.
- **HPC:** Cluster Placement Group + **EFA** + Spot + **Batch/ParallelCluster** + **FSx for Lustre**.
- **EFA = MPI ו-tightly coupled, Linux בלבד. ENA = throughput כללי, עד 100 Gbps.**
- **Multi-AZ ≠ DR. Read Replica ≠ Multi-AZ** (failover ידני, נועדה ל-scaling).
- **ל-S3/DynamoDB מסאבנט פרטי — Gateway Endpoint (חינם), לא NAT.**
- **6 Pillars:** Operational Excellence · Security · Reliability · Performance Efficiency ·
  Cost Optimization · Sustainability. **הם סינרגיה, לא trade-off.**
- **WA Tool** = שאלון על workload. **Trusted Advisor** = סריקת חשבון ב-**6 קטגוריות**
  (כולל **Service Limits**), סט מלא ב-Business/Enterprise Support.

---

## 11. ✅ בדיקת הבנה

1. שאלה דורשת RTO של 5 דקות ומבקשת את הפתרון **המשתלם ביותר**. האם Backup & Restore יכול לנצח?
2. אפליקציה איטית, ה-DB עמוס בקריאות, ו**אסור לשנות קוד**. מה שתי האפשרויות ומה ההבדל?
3. "Block a specific IP" — למה התשובה משתנה כשיש CloudFront בארכיטקטורה?
4. Job רץ 40 דקות ומעבד קבצים. למה Lambda נפסלת מיד?
5. צריך שכמה שירותים יקבלו את אותו אירוע, וכל אחד יעבד בקצב שלו בלי לאבד הודעות. מה הדפוס?
6. מתי בוחרים Kinesis ולא SQS?
7. Workload HPC עם MPI. מה שלושת הרכיבים שחייבים להופיע בתשובה?
8. מה ההבדל בין Well-Architected Tool ל-Trusted Advisor, ומה הרמז שמכריע?
9. EC2 פרטי צריך לקרוא מ-S3 בעלות מינימלית. למה NAT Gateway היא תשובה שגויה?
10. שתי תשובות עומדות בכל הדרישות הקשיחות. איך מכריעים?

<details>
<summary>תשובות</summary>

1. **לא.** Backup & Restore נפסל בשלב 2 — הוא לא עומד ב-RTO של 5 דקות.
   "משתלם ביותר" מכריע **רק בין הכשירים**. כאן זה כנראה **Warm Standby**.
2. **Read Replicas** מפזרות קריאות ל-instances נוספים — שקוף לאפליקציה אם היא כותבת ל-writer
   וקוראת מ-endpoint הקריאה. **ElastiCache** שומר תוצאות בזיכרון ונותן **sub-millisecond**,
   אבל דורש לוגיקת cache — ולכן פחות "ללא שינוי קוד". אם הדגש הוא **אפס שינוי** → Read Replica.
3. כי **CloudFront מסיים את החיבור בקצה**. ה-VPC כבר לא רואה את ה-IP של הלקוח אלא את כתובות
   CloudFront, ולכן **NACL חסר תועלת**. החסימה חייבת לקרות בקצה: **WAF** על ה-CloudFront
   distribution, או **Geo Restriction** לחסימה לפי מדינה.
4. **תקרת זמן הריצה של Lambda היא 15 דקות.** 40 דקות פשוט לא אפשריות.
   הכיוונים: **Fargate**, **ECS/EC2** או **AWS Batch**.
5. **SNS + SQS Fan-Out.** SNS מפיץ לכל ה-subscribers, ולכל שירות יש **תור SQS משלו** —
   כך כל אחד צורך בקצב שלו, ואם consumer נפל ההודעות **ממתינות בתור** ולא אובדות.
   SNS לבדו היה מאבד הודעה שנמענה לא הצליח לקבל.
6. כשצריך **סדר לפי partition key**, **קריאה חוזרת של אותו דאטה (replay)**,
   **כמה consumers שקוראים את אותו stream במקביל**, או **נפח real-time גבוה מאוד**.
   ב-SQS ההודעה נמחקת אחרי עיבוד ו-consumer אחד מקבל אותה.
7. **Cluster Placement Group** (אותו rack, latency מינימלי) + **EFA** (MPI, עוקף את ה-OS, Linux בלבד)
   + **FSx for Lustre** (מערכת קבצים במיליוני IOPS מגובת S3).
   ולניהול: **AWS Batch** או **ParallelCluster**, ולעלות: **Spot**.
8. **WA Tool** — ממלאים **שאלון על workload** ומקבלים ניתוח מול **ששת ה-Pillars**.
   **Trusted Advisor** — **סורק את החשבון אוטומטית**, בלי שאלון, ומחזיר המלצות ב-**6 קטגוריות**.
   **הרמז המכריע:** "service limits" או "unused/underutilized resources" → **Trusted Advisor**.
9. כי **S3 Gateway Endpoint עולה אפס**, בעוד NAT Gateway מחייב **גם שעות וגם GB מעובד**.
   בנוסף ה-Endpoint שומר את התעבורה **בתוך רשת AWS** ולא מוציא אותה לאינטרנט — גם זול יותר וגם מאובטח יותר.
10. **בוחרים את זו עם פחות רכיבים לתחזק.** אם עדיין תיקו — זו עם ה-operational overhead הנמוך,
    כלומר הפתרון המנוהל יותר. ואם גם זה שווה — משווים TCO כולל תעבורה ו-requests, לא רק מחיר בסיס.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[02 - AWS Well-Architected Framework]] · [[24 - Database Selection]] · [[33 - High Availability and Scalability]] · [[34 - Disaster Recovery]] · [[29 - Event-Driven Architecture]] · [[30 - Application Decoupling]] · [[37 - Cost Optimization]] · [[38 - Serverless and Modern Architectures]] · [[40 - Integrated SAA Scenarios]] · [[41 - Final Review and Exam Strategy]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 6753–6772, 9267–9303, 15371–15558, 16124–16210
