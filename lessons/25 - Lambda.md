---
lesson: 25
title: Lambda
domain: Design High-Performing Architectures
services: [AWS Lambda, CloudWatch, API Gateway, S3, EventBridge, RDS Proxy]
tags: [saa-c03, serverless, compute, event-driven]
---

# 25 — Lambda

> [!abstract] בשורה אחת
> Lambda מריצה פונקציה קצרה בתגובה לאירוע, משלמים רק על מילישניות ריצה, ובמבחן היא התשובה כשכתוב "no servers to manage" יחד עם עבודה שנמשכת פחות מ-15 דקות.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה שרת שרץ 24/7 בשביל 3 שניות עבודה הוא בזבוז |
| 2 | איך זה עובד | מודל ההפעלה, sync מול async, execution role, לוגים |
| 3 | פירוק מפורט | טבלת המגבלות המדויקות + concurrency + cold start + VPC |
| 4 | עלות | requests + GB-seconds, ולמה יותר RAM לפעמים זול יותר |
| 5 | השוואות | Lambda מול EC2 / Fargate / Batch |
| 6 | Well-Architected | מה כל pillar אומר על פונקציות |
| 7 | מלכודות | 15 דקות, VPC ואינטרנט, throttling, idempotency |
| 8 | Scenario | thumbnail pipeline מקצה לקצה |

**מונחי מפתח בשיעור:** `Execution Role` · `Concurrency` · `Reserved Concurrency` · `Provisioned Concurrency` · `Cold Start` · `SnapStart` · `ENI` · `RDS Proxy`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- יש לך קוד שרץ 200 מילישניות בכל פעם שמישהו מעלה קובץ ל-S3.
- כדי להריץ אותו צריך EC2 instance.
- ה-instance רץ 24 שעות ביממה, גם כשאף אחד לא מעלה כלום.
- אתה משלם על 24 שעות בשביל אולי 4 דקות עבודה אמיתית ביום.
- וכשפתאום מגיעות 5,000 העלאות בדקה — צריך Auto Scaling, Launch Template, health checks, patching.

### מה השירות פותר

- אתה מעלה **קוד בלבד** — לא AMI, לא OS, לא patching.
- AWS מריצה אותו כשקורה אירוע, ומכבה מיד אחרי.
- Scaling אוטומטי לחלוטין: 1 בקשה או 1000 בקשות במקביל — אותו קוד, בלי הגדרה.
- כשאין תעבורה — **0 עלות compute**. זה ה-scale-to-zero האמיתי.

### מה זה serverless בכלל

- זה **לא** אומר שאין שרתים. זה אומר שאתה לא מקצה, לא רואה ולא מתחזק אותם.
- בהתחלה serverless == FaaS (Function as a Service), כלומר Lambda בלבד.
- היום המונח מכסה כל שירות מנוהל שמתרחב לבד: DynamoDB, S3, SQS, SNS, Firehose, Fargate, Aurora Serverless, Step Functions, Cognito, API Gateway.

> [!tip] האנלוגיה
> EC2 זה לשכור דירה — משלם על החודש גם אם ישנת שם לילה אחד.
> Lambda זה מונית — משלם על הנסיעה, ובין נסיעות המונה לא רץ.

---

## 2. ⚙️ איך זה עובד

### 2.1 מודל ההפעלה

```text
Event Source                 Lambda Service                 היעד
-------------                --------------                 -----
S3 / API GW / SQS   ──────>  1. שולף את הקוד
EventBridge / SNS            2. מקצה execution environment
Kinesis / DynamoDB           3. מריץ init (קוד מחוץ ל-handler)
                             4. מריץ handler(event, context)  ──> DynamoDB / S3 / SNS
                             5. מחזיר תשובה + כותב ל-CloudWatch Logs
```

- הפונקציה מקבלת אובייקט `event` — התוכן שלו תלוי במי הפעיל אותה.
- ההרשאות מגיעות מ-**Execution Role** (IAM Role). אף פעם לא access keys בקוד.
- כל `print`/`console.log` הולך אוטומטית ל-CloudWatch Logs (בתנאי שה-role מרשה).

### 2.2 שלוש דרכי הפעלה

| סוג | מי מפעיל כך | מה קורה בכישלון | מה מקבל הקורא |
|---|---|---|---|
| Synchronous | API Gateway, ALB, Cognito, קריאה מה-CLI/SDK | הכישלון חוזר לקורא — **הוא** אחראי ל-retry | תשובת הפונקציה |
| Asynchronous | S3, SNS, EventBridge | Lambda מנסה שוב אוטומטית, ואז DLQ / Destination | רק אישור קבלה |
| Event Source Mapping (poll) | SQS, Kinesis, DynamoDB Streams | Lambda מושכת בעצמה בקבוצות (batch) | לא רלוונטי |

### 2.3 שירותים שמפעילים Lambda (הנפוצים במבחן)

- **API Gateway** — REST/HTTP API ללא שרתים.
- **S3 Event Notifications** — קובץ נוצר/נמחק.
- **EventBridge / CloudWatch Events** — CRON וכללי אירועים.
- **SQS / SNS** — עיבוד הודעות.
- **Kinesis Data Streams / DynamoDB Streams** — עיבוד זרם.
- **CloudFront (Lambda@Edge)** — התאמה בקצה (מפורט ב-[[15 - CloudFront and Global Delivery]]).
- **CloudWatch Logs** — subscription filter.

### 2.4 שתי הדוגמאות הקלאסיות בשקפים

```text
דוגמה א' — Thumbnail
העלאת תמונה ל-S3 ──trigger──> Lambda ──> כותבת thumbnail ל-bucket/prefix אחר
                                    └──> כותבת metadata (שם, גודל, תאריך) ל-DynamoDB

דוגמה ב' — CRON ללא שרת
EventBridge (כל שעה) ──trigger──> Lambda ──> מבצעת משימה מתוזמנת
```

- בעבר היה צריך EC2 שרץ תמיד רק כדי שיהיה בו crontab. עכשיו לא.

### 2.5 שפות נתמכות

- Node.js, Python, Java, C# (.NET) / PowerShell, Ruby.
- **Custom Runtime API** — כל שפה אחרת (Rust, Go) דרך runtime שהקהילה מתחזקת.
- **Container Image** — אפשר לארוז את הפונקציה כ-image, אבל היא **חייבת לממש את Lambda Runtime API**.

> [!warning] הבחנה שנשאלת
> "יש לי Docker image שרירותי שאני רוצה להריץ" → זה **ECS/Fargate**, לא Lambda.
> Lambda Container Image הוא רק אריזה חלופית לפונקציית Lambda, לא דרך להריץ כל image.

---

## 3. 🔍 פירוק מפורט

### 3.1 המגבלות שחייבים לשנן (per region)

| מגבלה | ערך | למה זה קריטי במבחן |
|---|---|---|
| Memory | 128 MB – 10 GB, בקפיצות של 1 MB | זה הידית היחידה — היא קובעת גם CPU וגם רשת |
| זמן ריצה מקסימלי | **900 שניות = 15 דקות** | המספר הכי נשאל. עבודה ארוכה יותר → Batch/Fargate/EC2 |
| דיסק זמני `/tmp` | 512 MB (ברירת מחדל) ועד 10 GB | ephemeral — נמחק, לא מקום לאחסון קבוע |
| Environment variables | 4 KB סה"כ | סודות גדולים → Secrets Manager / Parameter Store |
| Concurrent executions | **1000 לאזור** (ניתן להעלות בבקשת תמיכה) | ברירת מחדל שנופלים עליה |
| Deployment package (zip) | 50 MB | חורגים → להעלות ל-S3 או container image |
| Deployment לא דחוס (קוד + תלויות) | 250 MB | כולל layers |

- אין מגבלת CPU שמגדירים ישירות — **מגדילים RAM ומקבלים גם CPU ורשת**.
- אפשר להוריד קבצים נוספים ל-`/tmp` בזמן ה-init כדי לעקוף את מגבלת ה-250 MB.

### 3.2 Concurrency ו-Throttling

- **Concurrency** = כמה עותקים של הפונקציה רצים באותו רגע.
- הקצאה כברירת מחדל: **1000 לכל האזור**, משותפים בין **כל** הפונקציות בחשבון.
- חרגת מהמכסה → **Throttle**.

| סוג ההפעלה | מה קורה ב-Throttle |
|---|---|
| Synchronous | מוחזר `ThrottleError` עם קוד **429** לקורא |
| Asynchronous | Lambda מנסה שוב אוטומטית, ואם נכשל — ההודעה עוברת ל-**DLQ** |

- ב-async: על שגיאות 429 ו-5xx, Lambda מחזירה את האירוע לתור ומנסה שוב **עד 6 שעות**.
- מרווח ה-retry גדל אקספוננציאלית — משנייה אחת ועד מקסימום 5 דקות.

### 3.3 בעיית ה-Concurrency (השקף הקלאסי)

```text
חשבון אחד, מכסה של 1000 concurrent executions

Function A (מאחורי ALB)  ─ המון משתמשים ─> תופסת 1000  ──> THROTTLE
Function B (מאחורי API GW) ─ מעט משתמשים ─> לא נשאר לה כלום ──> THROTTLE!
```

- בלי הגבלה, פונקציה אחת "רעבה" מרעיבה את כל השאר בחשבון.
- הפתרון: **Reserved Concurrency** לכל פונקציה — גם תקרה וגם רצפה מובטחת.

### 3.4 Reserved מול Provisioned Concurrency

| | Reserved Concurrency | Provisioned Concurrency |
|---|---|---|
| מה זה עושה | מקצה נתח קבוע מתוך ה-1000 לפונקציה מסוימת | מחמם מראש N סביבות ריצה |
| המטרה | בידוד — למנוע מפונקציה אחת לחנוק את השאר | חיסול cold start — latency קבוע ונמוך |
| השפעה על אחרים | מוריד את המכסה הזמינה לשאר הפונקציות | לא נוגע במכסה הכללית |
| האם עולה כסף | **לא** — זו רק חלוקה של מכסה | **כן** — משלמים על ההקצאה גם בלי הפעלות |
| Auto Scaling | לא רלוונטי | Application Auto Scaling (לפי לו"ז או target utilization) |
| מילת מפתח במבחן | "prevent one function from consuming all" / "guarantee capacity" | "consistent low latency" / "eliminate cold starts" |

### 3.5 Cold Start

- **Cold start** = יצירת סביבת ריצה חדשה: טעינת הקוד + הרצת כל מה שמחוץ ל-handler (init).
- init כבד (תלויות, SDK, חיבורי DB) → הבקשה **הראשונה** לכל סביבה חדשה איטית משמעותית.
- הבקשות הבאות באותה סביבה מהירות (warm).
- דרכים להקטין:
  - להעביר קוד יקר ל-init ולהשתמש בו מחדש בין הפעלות.
  - להקטין את חבילת הפריסה.
  - Provisioned Concurrency (עולה כסף).
  - **SnapStart** (חינם, ראה למטה).

### 3.6 SnapStart

- משפר ביצועים **עד פי 10, ללא עלות נוספת**.
- נתמך ב-**Java, Python ו-.NET**.
- איך: כשמפרסמים version חדש, Lambda מריצה init פעם אחת, לוקחת **snapshot של הזיכרון והדיסק**, ושומרת אותו במטמון.
- כל הפעלה מתחילה מהמצב המאותחל במקום מאפס.
- ההבחנה במבחן: "reduce cold start **at no additional cost**" → SnapStart. "guaranteed low latency, willing to pay" → Provisioned Concurrency.

### 3.7 Lambda ו-VPC

**ברירת המחדל:**

```text
Lambda (ב-VPC של AWS, לא שלך)
   ├── ✅ יכולה לפנות לשירותים ציבוריים: DynamoDB, S3, SNS, האינטרנט
   └── ❌ לא יכולה להגיע ל-RDS פרטי, ElastiCache, internal ELB שלך
```

**כשמחברים ל-VPC שלך:**

- מגדירים VPC ID + Subnets + Security Groups.
- Lambda יוצרת **ENI** (Elastic Network Interface) ב-subnets שציינת.
- ה-Security Group של ה-ENI הוא זה שצריך להיות מורשה ב-SG של ה-RDS.

```text
Lambda ב-Private Subnet
   ├── ✅ מגיעה ל-RDS / ElastiCache / internal ALB
   └── ❌ אין לה יותר גישה לאינטרנט — צריך NAT Gateway,
        ולשירותי AWS עדיף VPC Endpoint (זול יותר מ-NAT)
```

> [!warning] המלכודת הכי נשאלת בנושא
> חיבור ל-VPC **לא** נותן גישה לאינטרנט. הוא דווקא **מבטל** אותה.
> Public IP לא עוזר לפונקציית Lambda. הפתרון הוא NAT Gateway (לאינטרנט) או VPC Endpoint (לשירותי AWS).

### 3.8 Lambda + RDS Proxy

- הבעיה: Lambda מתרחבת ל-1000 עותקים, כל אחד פותח חיבור ל-DB — ומיצית את ה-connection pool.
- **RDS Proxy** פותר:
  - **Scalability** — pooling ושיתוף חיבורים.
  - **Availability** — מקצר את זמן ה-failover ב-**כ-66%** ושומר על החיבורים הפתוחים.
  - **Security** — כופה IAM authentication ומחזיק את הסודות ב-Secrets Manager.
- מגבלה קריטית: RDS Proxy **לעולם לא נגיש מהאינטרנט**, ולכן ה-Lambda **חייבת** לשבת ב-VPC.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| Requests | לפי מספר ההפעלות | Free tier: מיליון בקשות בחודש |
| Duration | **GB-seconds** = זיכרון × זמן ריצה, במדידה של 1ms | Free tier: 400,000 GB-seconds בחודש |
| Provisioned Concurrency | לפי כמות מוקצית × זמן | משלמים גם כשהפונקציה בטלה |
| Ephemeral storage | על `/tmp` מעל 512 MB | לרוב זניח |
| Data transfer | יציאה מהאזור | כרגיל |

- 400,000 GB-seconds חינם = 400,000 שניות בפונקציית 1 GB, או 3,200,000 שניות בפונקציית 128 MB.

### הטריק שחייבים להבין: יותר RAM לפעמים זול יותר

- החיוב הוא זיכרון **כפול** זמן. אינטואיטיבית — יותר זיכרון = יותר כסף.
- אבל הגדלת הזיכרון מגדילה גם **CPU ורשת**, ולכן הפונקציה מסיימת מהר יותר.
- דוגמה מספרית פשוטה:

| הגדרה | זמן ריצה | GB-seconds | תוצאה |
|---|---|---|---|
| 512 MB | 4 שניות | 0.5 × 4 = 2.0 | בסיס |
| 1024 MB | 1.8 שניות | 1.0 × 1.8 = 1.8 | **זול יותר וגם מהיר יותר** |
| 2048 MB | 1.7 שניות | 2.0 × 1.7 = 3.4 | בזבוז — ה-CPU כבר לא צוואר הבקבוק |

- המסקנה: יש **נקודת אופטימום**, ומוצאים אותה במדידה (Lambda Power Tuning), לא בניחוש.
- זה נכון רק לעבודה שתלויה ב-CPU. פונקציה שרק מחכה ל-API חיצוני לא תרוץ מהר יותר עם עוד RAM.

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Lambda on-demand | הזול ביותר לעומסים ספוראדיים | תעבורה לא רציפה, burst, idle ארוך |
| Lambda + Provisioned Concurrency | יקר יותר — משלמים על זמן בטלה | latency קריטי ויציב |
| Fargate | משתלם יותר בעומס רציף | תהליך ארוך / מכולה קיימת |
| EC2 עם RI/Savings Plans | הכי זול בעומס גבוה וקבוע 24/7 | שימוש צפוי ומתמשך |

### 🚩 עלויות נסתרות

- **NAT Gateway** — פונקציה ב-VPC שצריכה אינטרנט משלמת על NAT לפי שעה + לפי GB. ה-NAT יכול לעלות יותר מהפונקציה עצמה.
- **CloudWatch Logs** — ingestion + אחסון. פונקציה עם לוג מפורט שרצה מיליוני פעמים מייצרת חשבון לוגים משמעותי.
- **Provisioned Concurrency** — רץ על השעון גם בלילה. שכחת לכבות = תשלום קבוע.
- **שירותים במעלה הזרם** — API Gateway, SQS, Kinesis מחויבים בנפרד.
- **retry של async** — כל ניסיון חוזר הוא הפעלה בתשלום.

### 💡 טיפים לחיסכון

- למדוד ולכוונן זיכרון במקום להשאיר 128 MB או 3 GB "ליתר ביטחון".
- להגדיר **retention** ללוגים (למשל 14 יום) במקום ברירת המחדל האינסופית.
- להעדיף **VPC Endpoint** על NAT Gateway כשהיעד הוא S3/DynamoDB.
- לעבד ב-**batch** מ-SQS/Kinesis — פחות הפעלות, פחות requests.
- SnapStart במקום Provisioned Concurrency כשהשפה תומכת — אותה מטרה, בחינם.

---

## 5. ⚖️ השוואות מכריעות

### Lambda מול EC2

| קריטריון | EC2 | Lambda |
|---|---|---|
| מה זה | שרת וירטואלי | פונקציה וירטואלית |
| מוגבל ע"י | RAM ו-CPU שהקצית | **זמן** — ריצות קצרות בלבד |
| מתי רץ | ברציפות | on-demand בלבד |
| Scaling | דורש התערבות / ASG | אוטומטי לחלוטין |
| חיוב בזמן בטלה | כן | לא |

### Lambda מול Fargate מול Batch

| קריטריון | Lambda | Fargate | AWS Batch |
|---|---|---|---|
| מגבלת זמן | 15 דקות | אין | **אין** |
| Runtime | רשימה מוגדרת + custom runtime | כל Docker image | כל Docker image |
| דיסק | `/tmp` עד 10 GB, ephemeral | volume של המכולה / EFS | EBS / instance store |
| Serverless | כן | כן | לא — רץ על EC2 (מנוהל ע"י AWS) |
| Spot | לא | כן (Fargate Spot) | כן, זה עיקר החיסכון |
| Use case | אירוע קצר, API, glue code | שירות/מכולה ללא ניהול EC2 | עשרות אלפי job-ים עם התחלה וסוף |

> [!info] שורה תחתונה
> קצר ואירועי → Lambda. מכולה ששרה ברציפות → Fargate. עיבוד batch כבד וארוך → AWS Batch. צריך שליטה ב-OS → EC2.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר בנושא הזה | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | פונקציה בלי תצפיתיות היא קופסה שחורה | לוגים מובנים ל-CloudWatch, X-Ray לטרייסינג, versions + aliases ל-rollback מיידי |
| Security | ההרשאות הן ה-execution role, לא הקוד | role בעל least-privilege לכל פונקציה בנפרד; סודות ב-Secrets Manager ולא ב-env vars; VPC רק כשבאמת צריך משאב פרטי |
| Reliability | async מנסה שוב — הקוד חייב לעמוד בזה | לכתוב פונקציות **idempotent**; להגדיר DLQ / on-failure Destination; reserved concurrency כדי לא להיחנק |
| Performance Efficiency | הזיכרון הוא ידית הביצועים היחידה | לכוונן זיכרון במדידה; להעביר אתחול ל-init; SnapStart/Provisioned Concurrency ל-p99 יציב |
| Cost Optimization | משלמים GB-seconds — כל מילישנייה נספרת | right-sizing של זיכרון, retention ללוגים, VPC Endpoint במקום NAT, batching |
| Sustainability | pay-per-use = אין חומרה שרצה סרק | scale-to-zero במקום EC2 בטל; ארכיטקטורה אירועית במקום polling רציף |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "no servers to manage" + עבודה קצרה | Lambda |
| "job runs for 2 hours" / "no time limit" | לא Lambda — Fargate / AWS Batch / EC2 |
| "run any arbitrary Docker image" | ECS / Fargate |
| "eliminate cold starts", מוכנים לשלם | Provisioned Concurrency |
| "reduce cold starts **at no extra cost**" (Java/Python/.NET) | SnapStart |
| "one function is throttling all the others" | Reserved Concurrency |
| "too many DB connections from Lambda" | RDS Proxy (+ Lambda ב-VPC) |
| "Lambda must access private RDS" | לחבר את ה-Lambda ל-VPC (ENI ב-private subnet) |
| "Lambda in VPC needs internet" | NAT Gateway; לשירותי AWS — VPC Endpoint |
| "run a task every hour without a server" | EventBridge Schedule → Lambda |
| "process file right after upload" | S3 Event Notification → Lambda |

### טעויות נפוצות

> [!warning] מלכודת 1 — 15 הדקות
> **הניסוח:** "עיבוד וידאו שנמשך 40 דקות לכל קובץ."
> **הטעות:** לבחור Lambda כי כתוב serverless.
> **הנכון:** תקרת ה-900 שניות היא קשיחה. התשובה היא Fargate / AWS Batch / MediaConvert.

> [!warning] מלכודת 2 — VPC ואינטרנט
> **הניסוח:** "חיברתי את הפונקציה ל-VPC והיא הפסיקה להגיע ל-API חיצוני."
> **הטעות:** להוסיף Public IP או Internet Gateway.
> **הנכון:** Lambda ב-VPC יוצאת החוצה רק דרך **NAT Gateway** ב-public subnet. לשירותי AWS — VPC Endpoint, וזול יותר.

> [!warning] מלכודת 3 — Reserved מול Provisioned
> **הניסוח:** "רוצים להבטיח שפונקציה קריטית תמיד תקבל capacity."
> **הטעות:** לבלבל בין השניים כי שניהם "concurrency".
> **הנכון:** Reserved = הקצאת מכסה, בלי עלות. Provisioned = סביבות מחוממות מראש, בתשלום.

> [!warning] מלכודת 4 — idempotency
> **הניסוח:** "אחרי תקלה נוצרו רשומות כפולות ב-DynamoDB."
> **הטעות:** להאשים את הקוד ולהוסיף try/catch.
> **הנכון:** async invocation מנסה שוב עד 6 שעות. הפונקציה **חייבת** להיות idempotent — למשל מפתח דטרמיניסטי או conditional write.

> [!warning] מלכודת 5 — S3 loop
> **הניסוח:** "פונקציה שמופעלת מהעלאה ל-bucket וכותבת את התוצאה לאותו bucket."
> **הטעות:** לא לשים לב שהכתיבה מפעילה את הטריגר שוב.
> **הנכון:** לכתוב ל-bucket אחר, או לפחות ל-prefix אחר עם סינון בטריגר. אחרת נוצר לולאה אינסופית בתשלום.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** אתר תמונות. משתמשים מעלים תמונה, וצריך ליצור thumbnail תוך שניות, לשמור metadata, ולעדכן טבלת מנויים ב-RDS פרטי. העומס לא צפוי — לפעמים 0 העלאות בשעה, לפעמים 3,000 בדקה. אסור לנהל שרתים.

```text
                    ┌────────────────────────────────────────┐
User ──upload──> S3 (originals)                              │
                    │ Event Notification (ObjectCreated)     │
                    ▼                                        │
              Lambda "make-thumb"                            │
              memory: מכוון במדידה                            │
              Reserved Concurrency = 200                     │
                    ├──> S3 (thumbnails)  ← bucket נפרד!     │
                    ├──> DynamoDB (metadata)                 │
                    └──> on-failure Destination ──> SQS DLQ  │
                                                             │
              Lambda "update-billing" (בתוך VPC, private subnet)
                    └──> RDS Proxy ──> RDS (private)         │
                              ▲                              │
                        VPC Endpoint ל-S3/DynamoDB (במקום NAT)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Lambda ולא EC2 | עומס ספוראדי — 0 עלות כשאין העלאות, scaling אוטומטי ב-burst |
| S3 Event Notification | טריגר מובנה, ללא polling |
| bucket נפרד ל-thumbnails | מונע לולאת טריגר אינסופית |
| Reserved Concurrency = 200 | burst של 3,000 לא יחנוק פונקציות אחרות בחשבון |
| DLQ / on-failure Destination | async מנסה 6 שעות; מה שנכשל לא נעלם אלא נשמר לבדיקה |
| פונקציה שנייה בתוך VPC | רק היא צריכה RDS פרטי — לא מכניסים את פונקציית ה-thumbnail ל-VPC לחינם |
| RDS Proxy | מונע מיצוי connection pool כשמאות עותקים רצים במקביל |
| VPC Endpoint | הפונקציה ב-VPC לא צריכה NAT יקר בשביל S3/DynamoDB |
| Execution Role ייעודי לכל פונקציה | least privilege — thumbnail לא צריכה גישה ל-RDS |

**למה לא EC2 + ASG?** אפשר, אבל תשלם על instances גם בשעות ללא העלאות, ותנהל AMI, patching ו-health checks בשביל 200ms עבודה.

**למה לא Fargate?** אם העיבוד היה חורג מ-15 דקות או דורש image מורכב — כן. לתמונה בודדת זה overkill.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- מיפוי מדויק של MB זיכרון ל-vCPU — לא נשאל, מספיק לדעת ש"יותר RAM = יותר CPU".
- מחירים בדולרים לפי אזור — צריך את **המודל** (requests + GB-seconds), לא מספרים.
- תחביר קוד של handler בכל שפה — זה מבחן ארכיטקטורה, לא פיתוח.
- הגדרת Lambda Layers ברמת CLI — מספיק לדעת שהם דרך לשתף תלויות.
- פרטי Lambda Runtime API עצמו.
- Lambda@Edge מול CloudFront Functions נלמד לעומק ב-[[15 - CloudFront and Global Delivery]].

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **900 שניות = 15 דקות** — התקרה הקשיחה. ארוך יותר → Fargate / Batch.
- Memory **128 MB – 10 GB**; היא קובעת גם CPU וגם רשת.
- `/tmp`: 512 MB עד 10 GB, **ephemeral**.
- Deployment: 50 MB zipped, 250 MB unzipped. Env vars: 4 KB.
- Concurrency ברירת מחדל: **1000 לאזור**, משותף לכל הפונקציות, ניתן להעלאה בבקשת תמיכה.
- Sync throttle → **429**. Async throttle → retry עד **6 שעות** ואז DLQ.
- **Reserved** = חלוקת מכסה, בחינם. **Provisioned** = חימום מראש, בתשלום.
- **SnapStart** = עד פי 10 מהר יותר, **ללא עלות**, ב-Java/Python/.NET.
- ברירת מחדל: הפונקציה **מחוץ** ל-VPC שלך — מגיעה לאינטרנט ולשירותים ציבוריים, לא ל-RDS פרטי.
- חיבור ל-VPC יוצר **ENI** ו**מבטל** גישה ישירה לאינטרנט → NAT Gateway או VPC Endpoint.
- **RDS Proxy** = pooling + failover מהיר ב-~66% + IAM auth; מחייב Lambda בתוך VPC.
- הרשאות = **Execution Role**. אף פעם לא credentials בקוד.
- Lambda היא **stateless** — state הולך ל-DynamoDB / S3 / RDS / ElastiCache.
- כתוב **idempotent** — retry אוטומטי הוא ברירת המחדל, לא חריג.
- תמחור: requests + **GB-seconds**. יותר RAM לפעמים מוזיל את הסך הכל.

---

## 11. ✅ בדיקת הבנה

1. פונקציה צריכה לעבד קובץ CSV שלוקח 22 דקות. מה עושים?
2. מה ההבדל המעשי בין Reserved ל-Provisioned Concurrency, ומי מהם עולה כסף?
3. חיברת Lambda ל-VPC והיא הפסיקה להגיע ל-API ציבורי. למה, ומה הפתרון הזול לשירותי AWS?
4. פונקציה מוגדרת ל-128 MB ורצה 6 שניות. למה כדאי לנסות 1024 MB?
5. פונקציה async נכשלת שוב ושוב — כמה זמן Lambda תנסה, ולאן ההודעה תגיע בסוף?
6. איך מונעים מפונקציה אחת לחנוק את כל שאר הפונקציות בחשבון?
7. יש לך Docker image שרירותי להרצה. Lambda Container Image או ECS?

<details>
<summary>תשובות</summary>

1. לא Lambda — התקרה היא 900 שניות. עוברים ל-AWS Batch או ECS/Fargate. אפשרות נוספת: לפצל את הקובץ ולעבד chunks במקביל דרך Step Functions + Lambda.
2. **Reserved** מקצה נתח מתוך מכסת ה-1000 של האזור — מטרתו בידוד, והוא **לא עולה כסף**. **Provisioned** מחזיק סביבות ריצה מאותחלות מראש כדי לבטל cold start — והוא **כן עולה כסף**, גם כשאין הפעלות.
3. חיבור ל-VPC מציב את הפונקציה בתוך ה-subnets שלך, ואין לה נתיב יציאה. לאינטרנט כללי צריך NAT Gateway ב-public subnet; לשירותי AWS כמו S3/DynamoDB עדיף **VPC Endpoint** — זול יותר ולא עובר דרך NAT.
4. הגדלת הזיכרון מגדילה גם CPU ורשת. אם העבודה תלוית CPU, הריצה עשויה לרדת ל-1.5 שניות — והחיוב הוא memory × time, כך שהסך הכל ב-GB-seconds יכול לרדת. מהיר יותר **וגם** זול יותר.
5. Lambda מחזירה את האירוע לתור ומנסה שוב **עד 6 שעות**, במרווחים שגדלים אקספוננציאלית מ-1 שנייה עד 5 דקות. מה שממשיך להיכשל מגיע ל-**DLQ** (SQS/SNS) או ל-on-failure Destination.
6. **Reserved Concurrency** ברמת הפונקציה — זו תקרה שמונעת ממנה לצרוך את כל ה-1000, ובמקביל מבטיחה לפונקציות אחרות capacity.
7. **ECS/Fargate**. Lambda Container Image מחייב שה-image יממש את Lambda Runtime API — הוא לא דרך להריץ כל image שרירותי.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[26 - Containers]] · [[27 - API Gateway]] · [[28 - SQS and SNS]] · [[29 - Event-Driven Architecture]] · [[38 - Serverless and Modern Architectures]] · [[22 - RDS Scaling and Availability]] · [[12 - VPC Private Connectivity]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 7938–8211, 8360–8473, 16033–16048
