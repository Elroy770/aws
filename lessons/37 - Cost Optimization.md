---
lesson: 37
title: Cost Optimization
domain: Design Cost-Optimized Architectures
services: [EC2, Savings Plans, Spot, S3, EBS, RDS, DynamoDB, NAT Gateway, VPC Endpoints, CloudFront, Cost Explorer, AWS Budgets, Cost Anomaly Detection, Trusted Advisor, Compute Optimizer]
tags: [saa-c03, cost, finops, pricing, data-transfer]
---

# 37 — Cost Optimization

> [!abstract] בשורה אחת
> השיעור הזה נותן לך את המפה של **על מה AWS מחייבת בפועל** — לפי חמש קטגוריות הוצאה — כדי שתזהה בשאלה "most cost-effective" איזו תשובה באמת מורידה חשבון ואיזו רק נשמעת זולה.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה "הכי זול" ≠ "cost-effective", ומה זה TCO |
| 2 | חמש קטגוריות ההוצאה | Compute · Storage · Database · Data Transfer · Ops |
| 3 | Compute | On-Demand / RI / Savings Plans / Spot / Dedicated / Capacity Reservation |
| 4 | Storage | S3 classes + Lifecycle, EBS gp3 מול gp2, snapshots |
| 5 | Database | RDS RI, Aurora Serverless, DynamoDB On-Demand מול Provisioned |
| 6 | Data Transfer | הקטגוריה שמפתיעה: same-AZ / cross-AZ / cross-Region / egress / NAT |
| 7 | כלי ניהול עלות | Cost Explorer · Budgets · Anomaly Detection · Tags · Trusted Advisor |
| 8 | מילת מפתח → פתרון זול | טבלת שליפה מהירה למבחן |

**מונחי מפתח בשיעור:** `Savings Plan` · `Spot` · `Lifecycle Policy` · `Gateway Endpoint` · `Egress` · `Cost Allocation Tags` · `Right-Sizing` · `Instance Scheduler`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- בדאטה-סנטר רכשת שרת פעם אחת; בענן אתה משלם עליו **כל שנייה שהוא קיים** — גם כשהוא בטלה.
- העלויות שמפילות ארגונים הן דווקא **הבלתי-נראות**: NAT Gateway, snapshots ישנים, egress לאינטרנט, cross-AZ traffic.
- אף אחד לא רואה את החשבון עד סוף החודש, ואז מאוחר מדי.
- מפתחים מקימים סביבות dev/test ושוכחים לכבות אותן בסופ"ש ובלילות.

### מה השיעור פותר

- נותן **מודל חשיבה לפי קטגוריות חיוב** במקום לזכור מחירון לפי שירות.
- מלמד לזהות בשאלה איזו קטגוריה היא זו שמייצרת את העלות, ולתקוף אותה.
- מבחין בין **הורדת מחיר ליחידה** (commitment, storage class) לבין **הורדת כמות היחידות** (right-sizing, כיבוי, caching).

> [!tip] האנלוגיה
> חשבון AWS הוא כמו חשבון סלולר: יש מנוי חודשי (compute), יש נפח אחסון,
> ויש **חיוב גלישה בחו"ל** — ה-data transfer — שהוא זה שמפוצץ את החשבון בלי שראית אותו מגיע.

> [!info] כלל הזהב של השיעור
> יש רק שלוש דרכים להוריד חשבון:
> 1. לשלם פחות ליחידה (commitment / storage class / Spot),
> 2. לצרוך פחות יחידות (right-size, כיבוי, lifecycle, caching),
> 3. לא לשלוח את הנתונים דרך מסלול יקר (Gateway Endpoint, CloudFront, same-AZ).

---

## 2. ⚙️ חמש קטגוריות ההוצאה

כל שאלת עלות במבחן נופלת לאחת מהחמש. הצעד הראשון הוא לזהות באיזו קטגוריה מדובר.

| קטגוריה | הנהג המרכזי של העלות | הכלי המרכזי לחיסכון |
|---|---|---|
| Compute | שעות instance / GB-seconds | Savings Plans, Spot, right-sizing, כיבוי |
| Storage | GB-חודש + requests + retrieval | Storage class + Lifecycle |
| Database | שעות instance + storage + I/O + capacity | RI, Aurora Serverless, On-Demand mode |
| **Data Transfer** | GB שעוברים גבול (AZ / Region / אינטרנט) | Gateway Endpoint, CloudFront, same-AZ |
| Operations & Tooling | לוגים, config items, monitoring, ניהול | סינון, retention, כלים מנוהלים |

```text
מבנה חשבון טיפוסי של אפליקציית web:

  Compute  ████████████████████  ~50%
  Storage  ████████              ~20%
  Database ██████                ~15%
  Transfer ████                  ~10%   ← זו שהכי מפתיעה
  Ops      ██                     ~5%
```

> [!warning] אזהרה על מספרים
> אין לשנן מחירים בדולרים — הם משתנים לפי Region ולפי הזמן.
> במבחן שואלים על **יחסים** ועל **מה מחויב**, לא על סכומים.

---

## 3. 🖥️ קטגוריה א' — Compute

### 3.1 שש אפשרויות הרכישה של EC2

| אפשרות | ההנחה | ההתחייבות | מתי בוחרים |
|---|---|---|---|
| On-Demand | 0 (המחיר הבסיסי) | אין | workload קצר, לא צפוי, ניסויים |
| Reserved Instance (RI) | עד ~72% | 1 או 3 שנים על attributes ספציפיים | baseline יציב, קלאסית DB |
| Convertible RI | עד ~66% | 1/3 שנים, אך ניתן להחליף family/OS/tenancy | baseline יציב עם אי-ודאות טכנולוגית |
| Savings Plan | עד ~72% (זהה ל-RI) | סכום $/שעה ל-1/3 שנים | baseline יציב + רצון בגמישות |
| Spot | עד ~90% | אין — אבל אפשר לאבד instance | batch, stateless, fault-tolerant |
| Dedicated Host | **היקר ביותר** | On-Demand או Reserved | BYOL per-socket/per-core, compliance |

### 3.2 RI מול Savings Plan — ההבחנה שנבדקת

| קריטריון | Reserved Instance | Savings Plan |
|---|---|---|
| מה מתחייבים עליו | instance type + Region + OS + tenancy | **סכום כסף לשעה** |
| גמישות גודל | Size flexibility בתוך family (Regional) | גמיש בין sizes, OS, tenancy |
| גמישות family | רק ב-Convertible RI | EC2 Instance SP נעול ל-family+Region; Compute SP גמיש לגמרי |
| שריון capacity | Zonal RI כן, Regional RI לא | **לא** — SP הוא הנחת חיוב בלבד |
| ניתן למכור | כן, ב-RI Marketplace | לא |
| חל גם על | EC2, RDS, ElastiCache, Redshift, OpenSearch | EC2, Fargate, Lambda (Compute SP) |

> [!info] שורה תחתונה
> רוצה **הנחה עם גמישות** → Savings Plan. רוצה **שריון capacity ב-AZ ספציפי** → Zonal RI או Capacity Reservation.

### 3.3 Spot — הכלי החזק והמסוכן

- הנחה של **עד ~90%** — הזול ביותר ב-AWS.
- מגדירים max price; כל עוד ה-spot price נמוך ממנו — ה-instance רץ.
- כשה-spot price עולה מעל ה-max → מקבלים **התראת 2 דקות** ואז stop/terminate.
- מתאים ל: batch jobs, data analysis, image processing, workloads מבוזרים, משימות עם זמן התחלה גמיש.
- **לא** מתאים ל: databases, jobs קריטיים, כל דבר stateful שאין לו checkpoint.

**Spot Fleet — אסטרטגיות הקצאה:**

| אסטרטגיה | מה עושה | מתי |
|---|---|---|
| `lowestPrice` | לוקח מה-pool הזול ביותר | workload קצר, אופטימיזציית עלות טהורה |
| `diversified` | פורס על כל ה-pools | workload ארוך, זמינות חשובה |
| `capacityOptimized` | pool עם ה-capacity האופטימלי | מזעור interruptions |
| `priceCapacityOptimized` | **המומלץ** — קודם capacity גבוה, אז המחיר הנמוך | ברירת המחדל הנכונה לרוב |

> [!warning] מלכודת Spot תפעולית
> ביטול Spot Request **לא** מסיים את ה-instances.
> קודם מבטלים את הבקשה, ורק אז עושים terminate ל-instances — אחרת הבקשה תמשיך להקים חדשים.

### 3.4 Capacity Reservation — לא הנחה

- משריין capacity ב-**AZ ספציפי** לכל משך זמן שתרצה.
- **אין שום הנחה** — משלמים On-Demand בין אם ה-instance רץ ובין אם לא.
- מטרתו: להבטיח שתמיד יהיה מקום פנוי (למשל event ידוע מראש).
- אפשר לשלב עם Regional RI / Savings Plan כדי לקבל גם את ההנחה.

### 3.5 Dedicated Host מול Dedicated Instance

| | Dedicated Host | Dedicated Instance |
|---|---|---|
| מה מקבלים | שרת פיזי שלם | חומרה שלא משותפת עם לקוחות אחרים |
| שליטה על placement | כן — רואים sockets/cores | לא |
| BYOL per-socket/per-core | כן | לא |
| יציבות אחרי Stop/Start | נשאר על אותו host | עשוי לעבור חומרה |
| עלות | היקרה ביותר | יקר, פחות מ-Host |

### 3.6 Lambda ו-Fargate — הצד השני של Compute

- Lambda מחייבת על **מספר invocations** ועל **GB-seconds** (duration × זיכרון), במדידה של 1ms.
- יש free tier נדיב: מיליון requests ו-400,000 GB-seconds בחודש.
- **הכלל הקריטי:** הגדלת זיכרון מגדילה גם CPU. לפעמים פונקציה עם יותר RAM מסיימת מהר יותר ועולה **פחות**.
- Lambda זולה מאוד ל-workload **ספורדי/burst**; ל-load רציף 24/7 — EC2/Fargate עם commitment יהיו זולים יותר.
- Fargate חוסך ניהול שרתים אך יקר יותר מ-EC2 **מנוצל היטב**; זול יותר מ-EC2 שרץ ב-15% ניצול.

### 3.7 Right-Sizing וכיבוי

- **AWS Compute Optimizer** מנתח מטריקות CloudWatch וממליץ על instance type מתאים יותר.
- **Instance Scheduler on AWS** — פתרון CloudFormation (לא שירות!) שמכבה ומדליק אוטומטית:
  - חיסכון של עד ~70% על סביבות non-production.
  - תומך ב-EC2, Auto Scaling Groups ו-RDS.
  - הלוחות זמנים נשמרים ב-**DynamoDB**, וההפעלה מתבצעת ב-**Lambda** לפי **tags** של המשאבים.
  - תומך cross-account ו-cross-region.

---

## 4. 🗄️ קטגוריה ב' — Storage

### 4.1 S3 Storage Classes — מה באמת מחייב

בכל class משלמים על **ארבעה** דברים: אחסון (GB-חודש), requests, retrieval, ו-data transfer.

| Class | עלות אחסון יחסית | Min Duration | Min Object Size | Retrieval Fee | זמן שליפה |
|---|---|---|---|---|---|
| Standard | הבסיס (היקר באחסון) | אין | אין | אין | מיידי |
| Intelligent-Tiering | נע בין Standard ל-Archive + דמי monitoring | אין | אין | אין | מיידי |
| Standard-IA | ~חצי מ-Standard | 30 יום | 128 KB | יש (per GB) | מיידי |
| One Zone-IA | זול מ-Standard-IA | 30 יום | 128 KB | יש | מיידי |
| Glacier Instant Retrieval | זול משמעותית | 90 יום | 128 KB | יש | מיידי |
| Glacier Flexible Retrieval | זול מאוד | 90 יום | 40 KB | יש | דקות עד שעות |
| Glacier Deep Archive | **הזול ביותר** (~פי 20 זול מ-Standard) | **180 יום** | 40 KB | יש | 12–48 שעות |

**עמידות וזמינות:**

- כל ה-classes: durability של 99.999999999% (11 תשיעיות).
- כולם פרוסים על **≥3 AZs** — **חוץ מ-One Zone-IA** שיושב ב-AZ אחד בלבד.
- Availability יורדת ככל שיורדים ב-tier (99.99% ב-Standard, 99.5% ב-One Zone-IA).

> [!warning] מלכודת ה-Minimum Duration
> אם מוחקים object מ-Glacier Deep Archive אחרי 10 ימים — **משלמים על 180 יום**.
> Lifecycle שמעביר קבצים קצרי-חיים לארכיון עלול **להעלות** את החשבון, לא להוריד אותו.

### 4.2 Lifecycle Policies — המנוע האמיתי לחיסכון

```text
דפוס lifecycle קלאסי ללוגים:

  יום 0-30    → S3 Standard              (גישה תכופה, אנליטיקה חמה)
  יום 30-90   → S3 Standard-IA           (גישה נדירה, עדיין מיידי)
  יום 90-365  → Glacier Flexible Retrieval (חקירות אירועים)
  יום 365+    → Glacier Deep Archive      (רגולציה בלבד)
  יום 2555    → Expiration (מחיקה)
```

- Transition rules מזיזות בין classes; Expiration rules מוחקות.
- אפשר לכוון גם על **noncurrent versions** ועל **incomplete multipart uploads** — שתי עלויות שקטות מאוד.
- **S3 Storage Lens** ו-**S3 Analytics** עוזרים להחליט מתי להעביר.
- אם דפוס הגישה **לא ידוע או משתנה** → Intelligent-Tiering, וחוסכים את עבודת ההחלטה.

### 4.3 EBS — gp3 מול gp2

| קריטריון | gp2 | gp3 |
|---|---|---|
| קשר בין גודל ל-IOPS | **צמוד** — 3 IOPS לכל GiB | **מנותק** לחלוטין |
| Baseline | תלוי בגודל, burst עד 3,000 IOPS בנפחים קטנים | 3,000 IOPS + 125 MiB/s תמיד |
| שדרוג ביצועים | חייבים להגדיל את הנפח (ולשלם על אחסון מיותר) | מגדילים IOPS/throughput בנפרד |
| מקסימום | 16,000 IOPS | 16,000 IOPS + 1,000 MiB/s |
| עלות | הבסיס | **זול יותר ל-GB מ-gp2** |

> [!tip] תשובה כמעט תמיד נכונה
> "צריך יותר IOPS בלי לשלם על אחסון מיותר" → **מעבר מ-gp2 ל-gp3**.
> זו הגדלת ביצועים שגם **מורידה** עלות — שילוב נדיר, ולכן אהוב על כותבי המבחן.

### 4.4 עלויות אחסון נסתרות

- **Snapshots** מצטברים: הראשון מלא, הבאים incremental — אבל אף אחד לא מוחק ישנים. פתרון: **DLM / AWS Backup lifecycle**.
- **EBS Snapshot Archive** מוזיל ארכיון של snapshots פי כמה, אבל השחזור לוקח 24–72 שעות.
- **נפחים unattached** ממשיכים לחייב במלוא המחיר גם אחרי terminate של ה-instance.
- **Elastic IP לא מקושר** — מחויב לפי שעה.
- **EFS**: יש Lifecycle Management ל-Infrequent Access ול-Archive; יש גם One Zone שזול משמעותית מ-Standard.
- **S3 Versioning ללא lifecycle** — כל גרסה ישנה היא אחסון שמשלמים עליו לנצח.

---

## 5. 🛢️ קטגוריה ג' — Database

| שירות | מודל החיוב | מנוף החיסכון המרכזי |
|---|---|---|
| RDS | שעות instance + storage + I/O + backup מעבר לגודל ה-DB | **Reserved Instances** ל-1/3 שנים |
| Aurora | שעות instance + storage לפי שימוש בפועל + I/O | Aurora I/O-Optimized ל-workloads עתירי I/O |
| Aurora Serverless v2 | **ACU-שעות** — מתרחב ומתכווץ אוטומטית | workload **לא צפוי / ספורדי** — אין תשלום על peak קבוע |
| DynamoDB Provisioned | RCU/WCU מוקצים לשעה | דפוס **צפוי ויציב** + Auto Scaling + Reserved Capacity |
| DynamoDB On-Demand | תשלום per-request | דפוס **בלתי צפוי / spiky** או מערכת חדשה שלא מכירים |
| ElastiCache | שעות node | RI ל-node; וגם — חוסך עלות DB במעלה הזרם |
| Redshift | שעות node | RI; ו-Redshift Serverless ל-workload לא רציף |

**כלל ההכרעה ב-DynamoDB:**

```text
האם דפוס התעבורה ידוע ויציב?
  כן  → Provisioned + Auto Scaling   (זול משמעותית ליחידת בקשה)
  לא  → On-Demand                    (יקר ליחידה, אבל אין תשלום על capacity לא מנוצל)

האם יש spike חד ובלתי צפוי (viral / flash sale)?
  → On-Demand, או Provisioned עם Auto Scaling אגרסיבי
```

**חיסכונות DB נוספים שמופיעים בשאלות:**

- **Read Replicas** מוזילים? לא ישירות — אבל הם מונעים שדרוג ה-primary ל-instance ענק ויקר.
- **ElastiCache לפני ה-DB** — מוריד את ה-load ומאפשר instance קטן יותר. חיסכון עקיף משמעותי.
- **RDS Storage Auto Scaling** — מונע over-provisioning של storage מראש.
- כיבוי RDS non-production (עד 7 ימים ברצף) דרך Instance Scheduler.

---

## 6. 🌐 קטגוריה ד' — Data Transfer (הקטגוריה המפתיעה)

זו הקטגוריה שהכי הרבה שאלות "hidden cost" נשענות עליה. הכלל: **תנועה פנימה כמעט תמיד חינם, תנועה החוצה עולה.**

### 6.1 טבלת המחירים היחסיים — לשנן

| מסלול | עלות יחסית | הערה |
|---|---|---|
| **Ingress** לאינטרנט → AWS | **חינם** | תמיד. זו הסיבה שהעלאות זולות. |
| בתוך אותו **AZ**, דרך **Private IP** | **חינם** | החיסכון המקסימלי |
| בתוך אותו AZ, דרך **Public IP / Elastic IP** | ~פי 2 מ-cross-AZ private | טעות תצורה קלאסית! |
| **Cross-AZ** באותו Region, private IP | יחידת בסיס (נניח ×1) | המחיר של HA |
| **Cross-Region** | ~פי 2 מ-cross-AZ | replication, DR |
| **Egress לאינטרנט** מ-EC2/S3 | **~פי 9 מ-cross-AZ** — היקר ביותר | זה הרוצח |
| **Egress דרך CloudFront** | זול מעט מ-S3 ישיר | + חוסך requests לחלוטין |
| **S3 → CloudFront** | **חינם** | לכן CloudFront לפני S3 הוא כמעט תמיד נכון |
| **S3 Cross-Region Replication** | ~פי 2 מ-cross-AZ | חלק מעלות ה-DR |
| **S3 Transfer Acceleration** | תוספת על גבי ה-transfer הרגיל | קונים מהירות, לא חיסכון |

### 6.2 NAT Gateway מול Gateway Endpoint — השאלה החוזרת ביותר

```text
❌ המסלול היקר:
Private Subnet EC2 → NAT Gateway → Internet Gateway → S3
   משלמים: NAT לפי שעה  +  NAT לפי GB מעובד  +  data transfer

✅ המסלול הזול:
Private Subnet EC2 → S3 Gateway Endpoint → S3 (באותו Region)
   משלמים: 0 על ה-Endpoint  +  0 על transfer באותו Region
```

| | NAT Gateway | **Gateway Endpoint** | Interface Endpoint (PrivateLink) |
|---|---|---|---|
| שירותים נתמכים | כל האינטרנט | **S3 ו-DynamoDB בלבד** | רוב שירותי AWS |
| חיוב לפי שעה | כן | **לא — חינם** | כן, per-AZ |
| חיוב לפי GB | כן, data processing | **לא** | כן |
| איך מוגדר | route ל-NAT ב-route table | **route table entry** (prefix list) | **ENI** עם private IP + DNS |
| יוצא לאינטרנט | כן | לא (רק לשירות) | לא |

> [!tip] הכלל שיחסוך לך שאלה
> "instances ב-private subnet ניגשים ל-S3, איך מורידים עלות?" → **S3 Gateway Endpoint**.
> הוא גם חוסך כסף וגם משפר אבטחה (התעבורה לא יוצאת מ-AWS backbone).

### 6.3 Egress — כיצד מצמצמים

- **CloudFront לפני S3/ALB** — התעבורה מ-S3 ל-CloudFront חינם, וה-egress מ-CloudFront זול יותר. גם ה-requests זולים משמעותית.
- **לעבד את הנתונים איפה שהם יושבים.** אם ה-DB בענן וה-application ב-on-prem — כל query שולח 100MB החוצה ומחזיר 50KB. הפוך: תעביר את ה-application לענן ותשלח רק את התוצאה.
- **Private IP במקום Public IP** בתוך ה-VPC — חיסכון מיידי וגם latency טוב יותר.
- **Direct Connect** בעלות egress נמוכה יותר מאינטרנט, במיוחד ב-DX location שנמצא באותו Region.
- **אותו AZ** לכל ה-chatty traffic — אבל שים לב: זה בא **על חשבון ה-HA**. במבחן, HA כמעט תמיד מנצח חיסכון של cross-AZ.

> [!warning] מלכודת ה-cross-AZ
> תשובה שכתוב בה "העבר הכול ל-AZ אחת כדי לחסוך cross-AZ traffic" היא **מלכודת** בכל שאלה
> שמזכירה high availability או fault tolerance. חיסכון שהורס עמידות — נכשל.

---

## 7. 🧰 קטגוריה ה' — כלי ניהול העלות

| כלי | מה עושה | מסתכל אחורה או קדימה? |
|---|---|---|
| **Cost Explorer** | ויזואליזציה וניתוח של עלות ושימוש; דוחות מותאמים | אחורה **וגם** קדימה (Forecast עד 18 חודשים) |
| **AWS Budgets** | קובע סף ומתריע (או מפעיל action) כשמתקרבים/חורגים | קדימה — מניעתי |
| **Cost Anomaly Detection** | ML שלומד את דפוס ההוצאה שלך ומזהה חריגות | קדימה — **בלי להגדיר threshold** |
| **Cost & Usage Report (CUR)** | הדאטה הגולמי ברמת השורה, ל-S3/Athena | אחורה, מקסימום פירוט |
| **Cost Allocation Tags** | מייחס עלות ל-team/project/env | אחורה — ייחוס |
| **Compute Optimizer** | המלצות right-sizing לפי מטריקות | קדימה — אופטימיזציה |
| **Trusted Advisor** | בדיקות אוטומטיות על 6 קטגוריות, ביניהן Cost | אחורה — ניקיון |

### 7.1 Cost Explorer — היכולות שנשאלות

- ניתוח **ברמת חשבון, שירות, tag או resource**.
- גרנולריות: חודשית, יומית ואפילו **שעתית ורמת resource**.
- **Forecast** של שימוש ועלות עד **18 חודשים** קדימה על בסיס ההיסטוריה.
- **המלצות Savings Plan** — כמה $/שעה כדאי להתחייב עליהם.

### 7.2 Cost Anomaly Detection — ההבדל מ-Budgets

| | AWS Budgets | Cost Anomaly Detection |
|---|---|---|
| דורש להגדיר סף? | **כן** — אתה קובע מספר | **לא** — ML לומד לבד |
| מזהה spike חד-פעמי? | רק אם חצה את הסף | כן, גם מתחת לסף |
| מזהה מגמת עלייה איטית? | לא | **כן** |
| ניתוח שורש הבעיה | לא | **כן** — root-cause analysis |
| התראות | SNS / email | SNS, בודדות או סיכום יומי/שבועי |

> [!info] שורה תחתונה
> "רוצים להתריע כשעוברים X דולר" → **Budgets**.
> "רוצים לזהות הוצאה חריגה בלי לדעת מראש מה הסף" → **Cost Anomaly Detection**.

### 7.3 Trusted Advisor

- הערכה ברמת החשבון, **ללא התקנה**.
- שש קטגוריות: **Cost Optimization**, Performance, Security, Fault Tolerance, **Service Limits**, Operational Excellence.
- הבדיקות המלאות זמינות רק בתוכניות **Business ו-Enterprise Support**.
- גישה תכנותית דרך **AWS Support API**.
- בדיקות עלות טיפוסיות: EC2 בניצול נמוך, Elastic IP לא מקושר, load balancers בטלים, RI לא מנוצלים.

### 7.4 AWS Organizations — חיסכון ברמת הארגון

- **Consolidated Billing**: חשבון תשלום אחד לכל החשבונות.
- **Volume discounts מצטברים** — כל השימוש בארגון נספר יחד לצורך מדרגות ההנחה (EC2, S3).
- **RI ו-Savings Plans משותפים** בין החשבונות — RI לא מנוצל בחשבון אחד מכסה חשבון אחר.
- **Tag Policies** מבטיחות tagging עקבי, וזה תנאי הכרחי ל-Cost Allocation Tags אמינים.

---

## 8. 💰 עלות ותמחור — מבט מרוכז

### על מה מחייבים — לפי סוג משאב

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| EC2 | לפי שנייה (Linux/Windows, אחרי הדקה הראשונה); לפי שעה בשאר ה-OS | Stopped instance לא מחויב על compute — אבל **כן** על ה-EBS |
| EBS | GB-חודש מוקצה (לא בשימוש!) + IOPS/throughput מוקצים | gp3 מחייב IOPS/throughput בנפרד |
| S3 | GB-חודש + requests + retrieval + transfer | ב-IA/Glacier: גם min duration ו-min object size |
| Lambda | invocations + GB-seconds | מדידה ב-1ms; free tier נדיב |
| NAT Gateway | **שעות + GB מעובדים** | שני חיובים במקביל — זו המלכודת |
| ALB/NLB | שעות + LCU/NLCU | LCU מודד connections, bytes, rules |
| Data Transfer | GB לפי מסלול | ראה סעיף 6 |
| CloudWatch Logs | ingestion + storage + queries | פילטור בצד ה-agent חוסך המון |

### 🚩 עלויות נסתרות — רשימת בדיקה

- **NAT Gateway** — חיוב כפול (שעות + GB), וגם NAT לכל AZ ל-HA. תדיר הפריט השלישי בגודלו בחשבון.
- **Elastic IP לא מקושר** — משלמים דווקא כשלא משתמשים.
- **EBS volumes ו-snapshots יתומים** — נשארים אחרי terminate.
- **S3 noncurrent versions** ו-**incomplete multipart uploads** — אחסון שלא רואים בקונסולה.
- **Cross-AZ traffic** בין application ל-DB, בין nodes של Kafka/Cassandra.
- **CloudWatch Logs ingestion** — לוגים ב-DEBUG בפרודקשן.
- **Interface Endpoints** — חיוב per-AZ per-hour, מצטבר כשיש הרבה שירותים.
- **Data Transfer Out ב-Glacier retrieval** — Expedited retrieval יקר מאוד יחסית ל-Bulk (שלעיתים חינם).

### 💡 עשרת החיסכונות המהירים

1. העבר את כל נפחי ה-gp2 ל-**gp3** — יותר ביצועים בפחות כסף.
2. הפעל **S3 Lifecycle** ללוגים ולגיבויים, כולל מחיקת גרסאות ישנות ו-multipart תקועים.
3. החלף NAT Gateway ב-**S3/DynamoDB Gateway Endpoint** לכל תעבורה לשירותים אלה.
4. שים **CloudFront** לפני S3 ולפני ALB — egress זול יותר + פחות requests.
5. הפעל **Instance Scheduler** לכיבוי dev/test מחוץ לשעות עבודה.
6. קנה **Compute Savings Plan** לגובה ה-baseline הקבוע (לא ל-peak).
7. העבר batch/CI ל-**Spot** עם `priceCapacityOptimized`.
8. הרץ **Compute Optimizer** ובצע right-sizing ל-instances בניצול נמוך.
9. נקה **Elastic IPs, EBS volumes ו-snapshots יתומים** (Trusted Advisor מוצא אותם).
10. הפעל **Cost Anomaly Detection** + **Budgets** + **Cost Allocation Tags** כדי לא לגלות בסוף החודש.

---

## 9. ⚖️ השוואות מכריעות

### מודל רכישה — איזה מתי

| הצורך בשאלה | התשובה |
|---|---|
| workload יציב 24/7, בטוחים ב-instance type | Reserved Instance (Standard) |
| workload יציב אך ייתכן שינוי טכנולוגי | Savings Plan (או Convertible RI) |
| baseline יציב + burst משתנה | RI/SP ל-baseline + On-Demand/Spot ל-burst |
| batch שאפשר להפסיק ולהתחיל | **Spot** |
| חייבים capacity מובטח ב-AZ מסוים | **Capacity Reservation** (או Zonal RI) |
| licensing per-socket / compliance פיזי | **Dedicated Host** |
| workload קצר ולא צפוי לחלוטין | On-Demand |

### Serverless מול Server-based מבחינת עלות

| קריטריון | Lambda / Fargate / Aurora Serverless | EC2 / ECS on EC2 / RDS |
|---|---|---|
| דפוס ספורדי או spiky | **זול משמעותית** — לא משלמים על idle | משלמים על idle |
| דפוס רציף 24/7 בעומס גבוה | יקר יותר ליחידת עבודה | **זול יותר** עם RI/SP |
| עלות תפעול (ops) | נמוכה מאוד | גבוהה — patching, scaling, ניטור |
| commitment | אין (Compute SP מכסה Lambda/Fargate) | RI/SP זמינים |

> [!info] שורה תחתונה
> Serverless מנצח כשהניצול נמוך או משתנה. Server-based עם commitment מנצח כשהניצול גבוה וקבוע.
> "Most cost-effective" תמיד כולל גם את **עלות התפעול**, לא רק את חשבון ה-AWS.

---

## 10. 🏛️ Well-Architected — ששת ה-Pillars בהקשר עלות

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | אי אפשר לנהל מה שלא מודדים ולא מייחסים לבעלים | Cost Allocation Tags + Tag Policies + review חודשי ב-Cost Explorer |
| Security | חיסכון שמסיר בקרות אבטחה הוא לא חיסכון | Gateway Endpoint חוסך כסף **וגם** מוריד חשיפה — זו הבחירה הנכונה משני הצדדים |
| Reliability | Spot ו-single-AZ חוסכים אבל מסכנים | Spot רק ל-fault-tolerant; שמור Multi-AZ בפרודקשן גם אם cross-AZ עולה |
| Performance Efficiency | over-provisioning הוא ביטוח יקר על תכנון גרוע | right-size לפי Compute Optimizer; caching ב-ElastiCache/CloudFront במקום instance גדול |
| Cost Optimization | התאם את מודל התשלום לצורת ה-workload | SP ל-baseline, Spot ל-burst, Lifecycle ל-cold data, Gateway Endpoint ל-transfer |
| Sustainability | פחות משאבים בטלים = פחות אנרגיה | כיבוי non-prod, Graviton, Auto Scaling צמוד לביקוש, Deep Archive במקום דיסק חם |

---

## 11. 🪤 מלכודות במבחן

### מילות מפתח → הכלי/הפתרון הזול

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "most cost-effective" + batch / fault-tolerant | **Spot Instances** |
| "steady state" / "predictable" / "24/7 database" | **Reserved Instances / Savings Plans** |
| "flexibility across instance families" | **Compute Savings Plan** (או Convertible RI) |
| "reserve capacity in a specific AZ" | **Capacity Reservation / Zonal RI** |
| "per-socket license" / "BYOL" / "regulatory hardware" | **Dedicated Host** |
| "private subnet accessing S3" + reduce cost | **S3 Gateway Endpoint** (במקום NAT) |
| "high egress to internet" / "reduce data transfer out" | **CloudFront** לפני המקור |
| "data accessed rarely but must be immediate" | **S3 Standard-IA** או Glacier Instant Retrieval |
| "archive for 7 years, retrieval within 12 hours OK" | **Glacier Deep Archive** |
| "access pattern unknown / changing" | **S3 Intelligent-Tiering** |
| "need more IOPS without paying for size" | **gp3** |
| "unpredictable/intermittent database load" | **Aurora Serverless v2** |
| "unpredictable DynamoDB traffic" | **On-Demand capacity mode** |
| "dev/test running after hours" | **Instance Scheduler** / stop schedules |
| "alert when spending exceeds a threshold" | **AWS Budgets** |
| "detect unusual spend without setting thresholds" | **Cost Anomaly Detection** |
| "forecast future costs" / "hourly resource-level cost" | **Cost Explorer** |
| "recommendations to reduce cost, no setup" | **Trusted Advisor** |
| "right-size EC2 based on utilization" | **Compute Optimizer** |
| "volume discounts across many accounts" | **Organizations + Consolidated Billing** |

### טעויות נפוצות

> [!warning] מלכודת 1 — Glacier "תמיד זול"
> **הניסוח:** "העבר את כל האובייקטים ל-Glacier Deep Archive כדי לחסוך."
> **הטעות:** מתעלמים מ-minimum storage duration של 180 יום ומ-retrieval fee.
> **הנכון:** Deep Archive זול רק לדאטה שנשמר **שנים** ונשלף **לעיתים נדירות**.
> אם שולפים תדיר — משלמים יותר מ-Standard.

> [!warning] מלכודת 2 — Spot ל-database
> **הניסוח:** "כדי לחסוך, הרץ את שרתי ה-database על Spot."
> **הטעות:** Spot יכול להיעלם בהתראה של 2 דקות.
> **הנכון:** Spot רק ל-stateless/batch/fault-tolerant. ל-DB → RI או Aurora Serverless.

> [!warning] מלכודת 3 — Capacity Reservation = הנחה
> **הניסוח:** "השתמש ב-Capacity Reservation כדי להוזיל את החשבון."
> **הטעות:** Capacity Reservation **לא נותן שום הנחה**, ומחייב גם כשלא רצים.
> **הנכון:** הוא מבטיח **זמינות capacity**. להנחה — RI או Savings Plan (אפשר לשלב).

> [!warning] מלכודת 4 — NAT Gateway כברירת מחדל
> **הניסוח:** "instances ב-private subnet כותבים ללוג ב-S3 דרך NAT Gateway."
> **הטעות:** משלמים גם שעות NAT וגם GB מעובדים, על תעבורה שכלל לא צריכה לצאת מ-AWS.
> **הנכון:** **S3 Gateway Endpoint** — אפס עלות, ובאותו Region גם ה-transfer אפס.

> [!warning] מלכודת 5 — חיסכון על חשבון HA
> **הניסוח:** "רכז את כל ה-instances ב-AZ אחת כדי לבטל cross-AZ data transfer."
> **הטעות:** מפרקים את ה-high availability עבור חיסכון קטן יחסית.
> **הנכון:** cross-AZ הוא המחיר של עמידות. במבחן, דרישת HA גוברת על חיסכון ב-transfer.

> [!warning] מלכודת 6 — "Reserved" ל-workload חדש
> **הניסוח:** "מערכת חדשה עולה לאוויר, קנה RI ל-3 שנים מראש."
> **הטעות:** מתחייבים לפני שיודעים מה דפוס השימוש.
> **הנכון:** קודם On-Demand, מודדים ב-Cost Explorer, ורק אז מתחייבים על ה-baseline שהתגלה.

---

## 12. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** פלטפורמת e-commerce עם חשבון AWS שגדל 40% ברבעון. צריך להוריד עלות **בלי לפגוע ב-availability** או ב-latency ללקוחות.

**מצב קיים:**

```text
Internet → ALB → ASG (20 × m5.large On-Demand, ניצול ממוצע 25%)
                    ↓ private IP, cross-AZ
                 RDS MySQL Multi-AZ (db.r5.2xlarge On-Demand)
                    ↓ NAT Gateway (×2, HA)
                 S3 (500TB לוגים ב-Standard, כולם)
תמונות מוצרים מוגשות ישירות מ-S3 לאינטרנט
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Right-size ל-m5.small/medium לפי Compute Optimizer | ניצול 25% = משלמים פי 4 על מה שצריך. זו הפעולה הכי משתלמת. |
| Compute Savings Plan על ה-baseline (למשל 8 instances) | ~72% הנחה על החלק שרץ תמיד; לא מתחייבים על ה-peak |
| ה-instances מעל ה-baseline מ-ASG כ-On-Demand | גמישות ל-spikes של קמפיינים |
| RDS Reserved Instance ל-3 שנים | ה-DB רץ 24/7 בכל מקרה — commitment ללא סיכון |
| ElastiCache לפני RDS | מוריד load ומאפשר instance קטן יותר בהמשך |
| S3 Lifecycle: 30d→IA, 90d→Glacier FR, 1y→Deep Archive | 500TB לוגים ב-Standard זה בזבוז ענק; רובם לא נקראים |
| **CloudFront** לפני S3 לתמונות | S3→CloudFront חינם, egress זול יותר, requests זולים משמעותית, latency טוב יותר |
| **S3 Gateway Endpoint** לכתיבת לוגים מה-private subnets | מבטל את מרבית ה-data processing של ה-NAT |
| Instance Scheduler לסביבות dev/test | עד ~70% חיסכון על non-prod |
| Cost Allocation Tags + Anomaly Detection + Budgets | מונע חזרה של אותה בעיה |

**למה לא Spot ל-web tier?**
אפשר — כחלק מ-Mixed Instances Policy ב-ASG, למשל baseline On-Demand/SP + חלק Spot.
אבל **לא** כל ה-tier על Spot, כי אירוע capacity יכול להוריד את האתר בזמן peak.

**למה לא Single-AZ ל-RDS?**
זה יחסוך כמחצית מעלות ה-DB — אבל מפר את דרישת ה-availability. פסול.

**מה זה עולה?**
העלות החדשה נשלטת בעיקר על ידי ה-baseline המחויב ב-SP, storage בטירים קרים, ו-CloudFront egress —
כל שלושתם זולים משמעותית מהמצב הקודם, בלי שום ויתור ארכיטקטוני.

---

## 13. 🚫 מה לא צריך לדעת למבחן

- אין צורך לשנן מחירים בדולרים — הם משתנים לפי Region ולפי הזמן.
- אין צורך לדעת את אחוזי ההנחה המדויקים של RI/SP (מספיק "עד ~72%", "Spot עד ~90%").
- אין צורך לדעת את מבנה קובץ ה-CUR או את סכמת ה-Athena שמעליו.
- אין צורך לדעת את ה-API של Cost Explorer או של Budgets.
- אין צורך להכיר את כל בדיקות Trusted Advisor — רק את שש הקטגוריות ואת מגבלת ה-Support plan.

---

## 14. ⚡ Cheat Sheet — סיכום מהיר

- **On-Demand** = הכי גמיש והכי יקר. **Spot** = עד ~90% הנחה, 2 דקות התראה, לא ל-DB.
- **RI** = התחייבות על instance ספציפי, ניתן למכור, יש Zonal ששומר capacity. **SP** = התחייבות על $/שעה, גמיש יותר.
- **Capacity Reservation** = capacity מובטח, **אפס הנחה**, משלמים גם כשלא רצים.
- **Dedicated Host** = היקר ביותר; רק ל-BYOL per-socket ול-compliance.
- **gp3 > gp2** תמיד: יותר ביצועים, פחות כסף, IOPS מנותק מגודל.
- **Glacier Deep Archive** = הזול ביותר, אבל min 180 יום ושליפה של 12–48 שעות.
- **Intelligent-Tiering** = הבחירה כשדפוס הגישה **לא ידוע** (יש דמי monitoring, אין retrieval fee).
- **Ingress חינם. Egress יקר.** Same-AZ private IP = חינם. Public IP בתוך AZ = משלמים!
- **S3 → CloudFront = חינם.** CloudFront → אינטרנט זול מ-S3 → אינטרנט.
- **Gateway Endpoint (S3/DynamoDB) = חינם לחלוטין.** NAT Gateway = שעות + GB.
- **Budgets** = סף שאתה קובע. **Anomaly Detection** = ML בלי סף. **Cost Explorer** = ניתוח + forecast 18 חודשים.
- **Instance Scheduler** = פתרון CloudFormation (לא שירות), DynamoDB + Lambda + tags, עד ~70% על non-prod.
- **Organizations** = consolidated billing, volume discounts מצטברים, RI/SP משותפים.
- **Trusted Advisor** = 6 קטגוריות, בדיקות מלאות רק ב-Business/Enterprise Support.

---

## 15. ✅ בדיקת הבנה

1. אפליקציה ב-private subnet כותבת 5TB לוגים ליום ל-S3 באותו Region, דרך NAT Gateway. מה הפעולה בעלת ההשפעה הגדולה ביותר על העלות?
2. מה ההבדל המהותי בין Capacity Reservation לבין Zonal Reserved Instance?
3. חברה רוצה להעביר 200TB של קבצים שנוגעים בהם פעם בחודשיים ל-storage class זול יותר, אך דורשת גישה **מיידית**. מה תבחר?
4. מתי Savings Plan עדיף על Reserved Instance, ומתי ההיפך?
5. אפליקציה חדשה עם דפוס תעבורה לא ידוע לחלוטין עולה על DynamoDB. איזה capacity mode תבחר ולמה?
6. למה CloudFront לפני S3 מוזיל את החשבון בשני אופנים שונים?

<details>
<summary>תשובות</summary>

1. **S3 Gateway Endpoint.** הוא מבטל גם את ה-data processing של ה-NAT (חיוב לכל GB) וגם את ה-data transfer, כי התעבורה נשארת ב-AWS ובאותו Region. Lifecycle ל-IA/Glacier יחסוך על **אחסון** — אבל 5TB ליום דרך NAT זו ההוצאה הדומיננטית כאן.

2. **Capacity Reservation** משריין capacity ב-AZ ללא כל הנחה, ללא התחייבות זמן, וניתן ליצור/לבטל מתי שרוצים. **Zonal RI** נותן גם שריון capacity **וגם** הנחה, אבל דורש התחייבות של שנה או שלוש. אפשר לשלב Capacity Reservation עם Regional RI/SP כדי לקבל את שני היתרונות.

3. **S3 Standard-IA.** גישה נדירה (פעם בחודשיים) אך שליפה מיידית ו-availability של 99.9% על ≥3 AZs. One Zone-IA יהיה זול יותר אך פחות עמיד מול אובדן AZ — מתאים רק לדאטה שניתן לשחזר. Glacier Instant Retrieval שווה שקילה אם הגישה נדירה עוד יותר, אך יש לו min duration של 90 יום.

4. **Savings Plan** עדיף כשרוצים גמישות — לשנות instance size, OS, tenancy, ואפילו לעבור ל-Fargate/Lambda (ב-Compute SP) בלי לאבד את ההנחה. **Reserved Instance** עדיף כשצריך **שריון capacity** ב-AZ ספציפי (Zonal RI), כשרוצים אפשרות **למכור** את ההתחייבות ב-Marketplace, או כשמדובר ב-RDS/ElastiCache/Redshift.

5. **On-Demand mode.** אין מידע היסטורי להקצות לפיו RCU/WCU, ו-over-provisioning יעלה כסף על capacity לא מנוצל בעוד under-provisioning יגרום ל-throttling. אחרי מספר שבועות, כשהדפוס יתבהר ב-CloudWatch/Cost Explorer, אפשר לעבור ל-Provisioned עם Auto Scaling ולחסוך משמעותית ליחידת בקשה.

6. ראשית, **התעבורה מ-S3 ל-CloudFront חינם** וה-egress מ-CloudFront לאינטרנט זול יותר מ-egress ישיר מ-S3. שנית, ה-cache ב-edge מקטין דרסטית את מספר ה-**GET requests** שמגיעים ל-S3, וחיוב ה-requests ב-CloudFront זול משמעותית. בונוס: latency נמוך יותר למשתמשים.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[06 - EC2 Pricing and Optimization]] · [[16 - S3 Fundamentals]] · [[18 - S3 Advanced Features]] · [[19 - EBS and EC2 Storage]] · [[12 - VPC Private Connectivity]] · [[15 - CloudFront and Global Delivery]] · [[02 - AWS Well-Architected Framework]] · [[39 - Architecture Decision Making]] · [[41 - Final Review and Exam Strategy]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 939–1140, 1569–1600, 5010–5157, 8077–8093, 11170–11185, 14524–14660, 15902–15957, 16105–16124
