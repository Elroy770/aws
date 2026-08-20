---
lesson: 26
title: Containers
domain: Design High-Performing Architectures
services: [Amazon ECS, AWS Fargate, Amazon EKS, Amazon ECR, AWS Batch, EFS, ALB]
tags: [saa-c03, compute, containers, microservices]
---

# 26 — Containers

> [!abstract] בשורה אחת
> ECS הוא ה-orchestrator של AWS, Fargate הוא ה-compute שמריץ אותו בלי שרתים, EKS הוא Kubernetes מנוהל — ובמבחן ההבדל ביניהם הוא כמעט תמיד שאלה של "כמה תשתית אתה מוכן לנהל".

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה containers, ומה ההבדל מ-VM |
| 2 | איך זה עובד | Dockerfile → image → ECR → ECS Task; שני ה-launch types |
| 3 | פירוק מפורט | IAM roles, ALB, EFS, Auto Scaling, EKS, ECR, Batch |
| 4 | עלות | Fargate מול EC2 מול Spot, cluster fee של EKS |
| 5 | השוואות | הטבלה הגדולה: ECS EC2 / Fargate / EKS / Lambda / Batch |
| 6 | Well-Architected | מה כל pillar אומר על containers |
| 7 | מלכודות | Instance Profile מול Task Role, S3 לא ניתן ל-mount |
| 8 | Scenario | microservice מקצה לקצה |

**מונחי מפתח בשיעור:** `Task Definition` · `ECS Agent` · `EC2 Instance Profile` · `ECS Task Role` · `Capacity Provider` · `Managed Node Group` · `CSI Driver`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- "אצלי במחשב זה עובד" — גרסאות שונות של ספריות, OS ותצורה בין dev ל-production.
- כל deployment דורש התקנות ידניות על השרת.
- קשה לארוז שירות אחד ולהזיז אותו בין סביבות או בין עננים.
- VM נותן בידוד, אבל כל VM גורר **Guest OS שלם** — כבד, איטי לעלייה, בזבזני.

### מה Docker פותר

- האפליקציה נארזת ב-**container** יחד עם כל התלויות שלה.
- אותה התנהגות בכל מכונה, בכל OS, בכל שפה — בלי בעיות תאימות.
- עלייה בשניות, לא בדקות.
- Use cases: microservices, ו-lift-and-shift של אפליקציות מ-on-premises לענן.

### Docker מול Virtual Machines

```text
Virtual Machines                    Docker
----------------                    ------
 App   App   App                     App   App   App   App   App
 Guest Guest Guest                   ───────────────────────────
  OS    OS    OS                          Docker Daemon
 ─────────────────                   ───────────────────────────
    Hypervisor                            Host OS (EC2)
 ─────────────────                   ───────────────────────────
   Infrastructure                        Infrastructure
```

- ב-VM: כל אפליקציה גוררת **Guest OS משלה**.
- ב-Docker: כל ה-containers **חולקים** את ה-kernel של ה-host.
- התוצאה: הרבה יותר containers על אותו שרת, בעלייה מהירה יותר.
- Docker הוא "סוג של" וירטואליזציה — אבל **לא** וירטואליזציה מלאה.

> [!tip] האנלוגיה
> VM זה בית פרטי לכל דייר — כל אחד עם תשתיות משלו.
> Container זה דירה בבניין — לכל דייר יש חלל משלו, אבל התשתית (kernel) משותפת.

---

## 2. ⚙️ איך זה עובד

### 2.1 מחזור החיים של image

```text
Dockerfile ──build──> Image ──push──> Repository ──pull──> Container רץ
                                      (ECR / Docker Hub)
```

| מיקום | מה זה | הערה |
|---|---|---|
| Docker Hub | repository ציבורי | images בסיס: Ubuntu, MySQL, nginx |
| **Amazon ECR** (Private) | ה-registry הפרטי של AWS | הבחירה הדיפולטיבית בארכיטקטורת AWS |
| ECR Public Gallery | repository ציבורי של AWS | `gallery.ecr.aws` |

### 2.2 ארבעת השירותים — מי עושה מה

| שירות | תפקיד | לא לבלבל |
|---|---|---|
| **ECS** | ה-orchestrator של AWS — מחליט איפה ומתי לרוץ | זה ה-"מנצח" |
| **EKS** | Kubernetes מנוהל — orchestrator חלופי | API אחר, מטרה דומה |
| **Fargate** | ה-**compute** שמריץ containers ללא שרתים | **לא** orchestrator! עובד גם עם ECS וגם עם EKS |
| **ECR** | אחסון images | registry, לא runtime |

### 2.3 ECS — EC2 Launch Type

```text
ECS Cluster
 ├── EC2 Instance  [ECS Agent] ── Task ── Task
 ├── EC2 Instance  [ECS Agent] ── Task
 └── EC2 Instance  [ECS Agent] ── Task ── Task
        ▲
   אתה מקצה ומתחזק את ה-instances האלה
```

- אתה מקצה, מתחזק ומתקן (patching) את ה-EC2 instances.
- כל instance **חייב להריץ ECS Agent** כדי להירשם ל-cluster.
- AWS מטפלת בהפעלה/עצירה של ה-containers עצמם.
- להריץ container ב-AWS = להריץ **ECS Task** על **ECS Cluster**.

### 2.4 ECS — Fargate Launch Type

```text
ECS Cluster (Fargate)
 └── AWS מריצה Tasks לפי ה-CPU/RAM שביקשת. אין instances לראות.
```

- **אין תשתית להקצות** — serverless לחלוטין.
- אתה רק מגדיר **Task Definition**: איזה image, כמה CPU, כמה RAM.
- כדי להתרחב — פשוט מעלים את מספר ה-tasks. זהו. אין ASG של EC2 לנהל.

### 2.5 מה זה Task Definition

- קובץ JSON שמתאר task: image, CPU/memory, פורטים, environment variables, volumes, logging, ו-**Task Role**.
- זו "התוכנית" — ה-Task הוא מופע רץ שלה, וה-**Service** מבטיח שירוץ תמיד מספר מסוים מהם.

---

## 3. 🔍 פירוק מפורט

### 3.1 IAM ב-ECS — ההבחנה שנופלים עליה

זו ההבחנה הכי נשאלת בנושא. **שני roles שונים לגמרי**:

| | EC2 Instance Profile | ECS Task Role |
|---|---|---|
| למי שייך | ל-**EC2 instance** ולסוכן (ECS Agent) שרץ עליו | ל-**task/container** הבודד |
| קיים ב-Fargate? | **לא** — רק ב-EC2 Launch Type | כן, בשני ה-launch types |
| איפה מוגדר | על ה-instance | בתוך ה-**Task Definition** |
| למה משמש | קריאות API ל-ECS, שליחת לוגים ל-CloudWatch, **משיכת image מ-ECR**, שליפת סודות מ-Secrets Manager / SSM | ההרשאות של **האפליקציה** — גישה ל-S3, DynamoDB, SQS... |
| כמה יש | אחד לכל instance | role שונה לכל Service/Task |

```text
EC2 Instance ── Instance Profile ──> ECS API, CloudWatch Logs, ECR, Secrets Manager
   ├── Task A ── Task Role A ──> S3 bucket X
   └── Task B ── Task Role B ──> DynamoDB table Y
```

> [!warning] המלכודת
> "ה-container לא מצליח לגשת ל-S3" → הבעיה ב-**Task Role**.
> "ה-instance לא מצליח למשוך image מ-ECR" → הבעיה ב-**Instance Profile**.
> ב-Fargate אין Instance Profile בכלל — שם יש Task Execution Role (למשיכת image ולוגים) ו-Task Role (לאפליקציה).

### 3.2 אינטגרציה עם Load Balancers

| LB | מצב | מתי |
|---|---|---|
| **ALB** | נתמך, ומתאים לרוב המקרים | ברירת המחדל ל-HTTP/HTTPS |
| **NLB** | מומלץ רק ל-throughput/ביצועים גבוהים במיוחד, או בשילוב **AWS PrivateLink** | TCP/UDP, latency נמוך קיצוני |
| **CLB** | נתמך אך **לא מומלץ** | אין תכונות מתקדמות, **לא עובד עם Fargate** |

- ALB מבצע **dynamic port mapping** ב-EC2 Launch Type — כך אפשר להריץ כמה tasks של אותו שירות על אותו instance.

### 3.3 Data Volumes — EFS

- אפשר לעשות **mount** ל-EFS file system לתוך ECS tasks.
- עובד ב-**שני** ה-launch types: EC2 וגם Fargate.
- tasks בכל AZ חולקים את **אותם נתונים** — אחסון משותף רב-AZ.
- **Fargate + EFS = אחסון persistent לחלוטין serverless.**
- Use case: state משותף בין containers, קבצי משתמשים, content management.

> [!warning] עובדה שנשאלת ישירות
> **את S3 אי אפשר למאונט כ-file system.** אם השאלה מדברת על "shared file system for containers" — התשובה היא **EFS**, לא S3.

### 3.4 ECS Service Auto Scaling (רמת ה-task)

- מגדיל/מקטין את מספר ה-**tasks** הרצויים אוטומטית.
- מבוסס על **AWS Application Auto Scaling**.
- המטריקות:
  - ECS Service Average **CPU** Utilization.
  - ECS Service Average **Memory** Utilization.
  - **ALB Request Count Per Target** — מטריקה שמגיעה מה-ALB.
- סוגי מדיניות:

| מדיניות | איך עובדת |
|---|---|
| Target Tracking | שומרת על ערך יעד למטריקת CloudWatch |
| Step Scaling | מגיבה ל-CloudWatch Alarm בצעדים |
| Scheduled Scaling | לפי תאריך/שעה — לשינויים צפויים |

> [!info] הבחנה קריטית
> **ECS Service Auto Scaling ≠ EC2 Auto Scaling.**
> הראשון מוסיף **tasks**. השני מוסיף **instances**. ב-EC2 Launch Type צריך את שניהם.
> ב-Fargate צריך רק את הראשון — ולכן ההגדרה שם הרבה יותר פשוטה.

### 3.5 EC2 Launch Type — הרחבת התשתית

יש שתי דרכים להוסיף EC2 instances כשה-tasks לא נכנסים:

| שיטה | איך |
|---|---|
| **ASG Scaling** | מרחיבים את ה-Auto Scaling Group לפי CPU Utilization — מוסיף instances לאורך זמן |
| **ECS Cluster Capacity Provider** | הדרך המומלצת: הוא מקצה ומרחיב את התשתית **אוטומטית לפי הצורך של ה-tasks** (CPU/RAM חסרים), בשילוב עם ASG |

```text
CloudWatch Metric (ECS Service CPU) → Alarm → ECS Service Auto Scaling → Task 3 (new)
                                                        │
                                          אין מקום? → Capacity Provider → ASG מוסיף EC2
```

### 3.6 ECS + אירועים (דפוסים שנשאלים)

| דפוס | תיאור |
|---|---|
| **EventBridge → ECS Task** | קובץ עולה ל-S3 → EventBridge Rule מריצה Fargate Task שקוראת את האובייקט ושומרת תוצאה ב-DynamoDB (דרך ה-Task Role) |
| **EventBridge Schedule → ECS Task** | כל שעה מריצים task ל-batch processing על S3 — CRON בלי שרת |
| **SQS → ECS Service** | ה-tasks מושכים הודעות מהתור, ו-ECS Service Auto Scaling מגדיל את מספרם לפי עומק התור |
| **Intercept Stopped Tasks** | container נופל → EventBridge Event Pattern תופס את האירוע → SNS שולח מייל לאדמין |

### 3.7 Amazon ECR

- אחסון וניהול של Docker images ב-AWS.
- repositories **פרטיים** וגם **ציבוריים** (ECR Public Gallery).
- משולב לחלוטין עם ECS, ומגובה ב-**S3** מאחורי הקלעים.
- הגישה נשלטת דרך **IAM** — ולכן **שגיאת הרשאה במשיכת image = בעיית policy**, לא בעיית רשת.
- תכונות: **image vulnerability scanning**, versioning, tags, ו-**lifecycle policies** למחיקת images ישנים.

### 3.8 Amazon EKS

- Kubernetes מנוהל על AWS. חלופה ל-ECS — מטרה דומה, **API שונה לגמרי**.
- ה-use case המובהק: החברה **כבר משתמשת ב-Kubernetes** on-premises או בענן אחר, ורוצה להעביר ל-AWS.
- Kubernetes הוא **cloud-agnostic** — רץ גם ב-Azure וב-GCP. זה בדיוק היתרון שמחפשים.
- **cluster אחד לכל region.** אין EKS גלובלי.
- לוגים ומטריקות: **CloudWatch Container Insights**.

**סוגי nodes ב-EKS:**

| סוג | מי מנהל | Spot? |
|---|---|---|
| **Managed Node Groups** | EKS יוצר ומנהל את ה-EC2 nodes בתוך ASG | On-Demand או Spot |
| **Self-Managed Nodes** | אתה יוצר ורושם ל-cluster; אפשר להשתמש ב-**EKS Optimized AMI** | On-Demand או Spot |
| **AWS Fargate** | **אין nodes בכלל** — אין תחזוקה | לא רלוונטי |

**Data Volumes ב-EKS:**

- צריך להגדיר **StorageClass** ב-cluster ולהשתמש ב-driver תואם **CSI** (Container Storage Interface).
- נתמכים: **EBS**, **EFS** (היחיד שעובד עם Fargate), **FSx for Lustre**, **FSx for NetApp ONTAP**.

### 3.9 AWS Batch — הקרוב המשפחתי

- עיבוד batch מנוהל לחלוטין בכל סקייל — עד מאות אלפי jobs.
- "Batch job" = עבודה עם **התחלה וסוף** מוגדרים, בניגוד לתהליך רציף.
- Batch מפעיל דינמית **EC2 או Spot Instances**, ומקצה בדיוק את כמות ה-compute/memory הנדרשת.
- ה-jobs מוגדרים כ-**Docker images** ורצים על ECS / EKS / Fargate.
- אתה שולח או מתזמן job — Batch עושה את השאר.

**Batch מול Lambda:**

| | Lambda | AWS Batch |
|---|---|---|
| מגבלת זמן | 15 דקות | **אין** |
| Runtime | רשימה מוגדרת + custom runtime | כל runtime, כל עוד ארוז כ-Docker image |
| דיסק | `/tmp` מוגבל ו-ephemeral | EBS / instance store |
| מודל | Serverless | נשען על EC2 (מנוהל ע"י AWS) |

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **ECS עצמו** | **0** | ה-orchestration חינם — משלמים רק על ה-compute |
| ECS on EC2 | instance-hours + EBS | משלמים גם על capacity לא מנוצלת |
| ECS/EKS on Fargate | **vCPU-שניות + GB-שניות** לפי משך ה-task | אין תשלום בין tasks |
| **EKS Control Plane** | **תשלום קבוע לשעה לכל cluster** | קיים גם אם ה-cluster ריק |
| ECR | אחסון GB/חודש + data transfer החוצה | הודעות pull בתוך אותו region זולות |
| ALB/NLB | שעות + LCU / NLCU | ראה [[08 - Elastic Load Balancing]] |
| EFS | GB מאוחסן (+ throughput mode) | ראה [[20 - EFS and File Storage]] |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| ECS on EC2 **Spot** | הזול ביותר — עד ~90% הנחה | עבודות שסובלות הפסקה, batch, CI |
| ECS on EC2 + Savings Plans | זול מאוד — עד ~72% הנחה | עומס יציב וצפוי 24/7, bin packing טוב |
| ECS on EC2 On-Demand | בינוני | עומס יציב בלי התחייבות |
| **Fargate** | יקר יותר ל-vCPU-שעה, אבל **0 בזמן idle** | עומס משתנה/ספוראדי, צוות קטן, פחות תפעול |
| Fargate **Spot** | זול משמעותית מ-Fargate רגיל | tasks שסובלים הפסקה |
| **EKS** | Fargate/EC2 **בתוספת** דמי cluster קבועים | רק כשיש דרישת Kubernetes אמיתית |

- הנקודה החשובה: Fargate יקר יותר **לשעת compute**, אבל אתה משלם רק על מה שרץ. ב-EC2 אתה משלם על ה-instance גם כשהוא 30% מנוצל.

### 🚩 עלויות נסתרות

- **capacity לא מנוצלת ב-EC2 Launch Type** — bin packing גרוע = instances חצי ריקים בתשלום מלא.
- **דמי ה-cluster של EKS** — קיימים גם ב-cluster dev שאף אחד לא נוגע בו.
- **NAT Gateway** — tasks ב-private subnet שמושכים images מהאינטרנט משלמים לפי GB. VPC Endpoint ל-ECR חוסך את זה.
- **ECR storage** — images ישנים שמצטברים לנצח בלי lifecycle policy.
- **CloudWatch Logs** — כל stdout של כל container.
- **data transfer בין AZ** — tasks שמדברים עם DB ב-AZ אחר.

### 💡 טיפים לחיסכון

- **lifecycle policy ב-ECR** — למחוק אוטומטית images ישנים ולא מתויגים.
- **Fargate Spot** או **EC2 Spot** ל-workloads שסובלים הפסקה.
- **VPC Endpoints ל-ECR ו-S3** במקום NAT Gateway.
- **right-sizing** של CPU/memory ב-Task Definition — ב-Fargate זה ישירות כסף.
- **Capacity Provider** ב-EC2 Launch Type — כדי לא לשלם על instances מיותרים.
- לא לפתוח EKS "כי זה סטנדרט" — אם אין דרישת Kubernetes, ECS זול יותר ופשוט יותר.

---

## 5. ⚖️ השוואות מכריעות

### הטבלה הגדולה — חמש אפשרויות ה-compute

| קריטריון | ECS on EC2 | ECS on Fargate | EKS | Lambda | AWS Batch |
|---|---|---|---|---|---|
| מי מנהל את השרתים | **אתה** (patching, AMI, ASG) | AWS | אתה או AWS (לפי סוג ה-node) | AWS | AWS (מקצה EC2/Spot בשבילך) |
| יחידת חיוב | instance-hours | vCPU-שניות + GB-שניות | דמי cluster + nodes/Fargate | requests + GB-seconds | ה-EC2/Spot שרץ |
| חיוב בזמן idle | **כן** | לא | כן (control plane) | לא | לא |
| זמן הפעלה | מהיר (instance כבר חם) | עשרות שניות | תלוי node type | מילישניות (חם) | דקות |
| מגבלת זמן ריצה | אין | אין | אין | **15 דקות** | אין |
| Spot | כן | Fargate Spot | כן | לא | כן — עיקר החיסכון |
| Use case | עומס יציב, שליטה ב-host, GPU | מיקרו-שירות בלי ניהול תשתית | ארגון שכבר על Kubernetes | אירוע קצר, glue code | עשרות אלפי jobs עם התחלה וסוף |
| מילת מפתח במבחן | "control over instances", "Spot for containers", "GPU" | "**least operational overhead**" + containers | "**Kubernetes**", "cloud-agnostic", "migrate existing K8s" | "no servers" + קצר | "**batch jobs**", "no time limit", "any runtime" |

### ECS מול EKS

| קריטריון | ECS | EKS |
|---|---|---|
| מקור | AWS-native | Kubernetes (open source) |
| עקומת למידה | נמוכה | גבוהה — צריך לדעת Kubernetes |
| ניידות בין עננים | לא | **כן** — cloud-agnostic |
| עלות בסיס | ה-orchestration חינם | דמי cluster קבועים לשעה |
| מתי בוחרים | ברירת המחדל ב-AWS | כשיש K8s קיים, כלים, או דרישת multi-cloud |

> [!info] שורה תחתונה
> containers + "least operational overhead" → **ECS on Fargate**.
> containers + המילה **Kubernetes** בשאלה → **EKS**.
> containers + שליטה ב-host / Spot / GPU → **ECS on EC2**.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר בנושא הזה | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | ה-image הוא יחידת ה-deployment — היא צריכה להיות ניתנת לגלגול אחורה | rolling/blue-green deployments ב-ECS Service, tags ל-image במקום `latest`, EventBridge שתופס stopped tasks ומתריע ב-SNS |
| Security | ההרשאות הן ברמת ה-task, לא ברמת ה-host | **Task Role** נפרד לכל שירות; **ECR image scanning** לחולשות; tasks ב-private subnets; סודות מ-Secrets Manager ולא ב-env vars |
| Reliability | task שנופל צריך לחזור לבד, ו-AZ שנופל לא מפיל את השירות | ECS Service עם desired count > 1 פרוס על כמה AZ, ALB health checks, EFS ל-state משותף |
| Performance Efficiency | CPU/memory ב-Task Definition הם חוזה — לא ניחוש | right-sizing לפי מטריקות, ECS Service Auto Scaling על ALB Request Count Per Target, NLB רק ל-throughput קיצוני |
| Cost Optimization | ההבדל בין Fargate ל-EC2 הוא idle | Spot ל-workloads סובלי הפסקה, Capacity Provider ל-bin packing, lifecycle policy ב-ECR, לא לפתוח EKS ללא צורך |
| Sustainability | container צפוף = פחות חומרה שרצה | bin packing טוב, scale-in אגרסיבי, Fargate כדי לא להשאיר nodes ריקים |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "containers" + "least operational overhead" | ECS on **Fargate** |
| "Kubernetes" / "already using K8s on-prem" / "cloud-agnostic" | **EKS** |
| "need control over the underlying instances" / GPU | ECS on **EC2** |
| "shared file system across containers in multiple AZ" | **EFS** (לא S3!) |
| "store container images privately" | **ECR** |
| "permission error pulling image from ECR" | IAM policy — של ה-Instance Profile / Task Execution Role |
| "each microservice needs different AWS permissions" | **ECS Task Role** לכל שירות |
| "scan container images for vulnerabilities" | ECR **Image Scanning** |
| "hundreds of thousands of batch jobs" | **AWS Batch** |
| "run a container on a schedule" | **EventBridge Schedule** → ECS Task |
| "scale tasks based on queue depth" | SQS + **ECS Service Auto Scaling** |
| "notify when a container stops unexpectedly" | **EventBridge** Event Pattern → SNS |

### טעויות נפוצות

> [!warning] מלכודת 1 — Fargate הוא לא orchestrator
> **הניסוח:** "בחר orchestrator ל-containers: ECS / EKS / Fargate / ECR."
> **הטעות:** לסמן Fargate.
> **הנכון:** Fargate הוא **launch type / compute engine**. ה-orchestrators הם ECS ו-EKS. Fargate עובד מתחת לשניהם.

> [!warning] מלכודת 2 — Instance Profile מול Task Role
> **הניסוח:** "ה-container לא מצליח לכתוב ל-DynamoDB."
> **הטעות:** להוסיף הרשאות ל-EC2 Instance Profile.
> **הנכון:** הרשאות של **האפליקציה** מגיעות מה-**ECS Task Role** שמוגדר ב-Task Definition. ה-Instance Profile משרת את ה-ECS Agent (ECS API, לוגים, משיכת image מ-ECR).

> [!warning] מלכודת 3 — S3 כ-file system
> **הניסוח:** "כמה containers צריכים לקרוא ולכתוב לאותם קבצים."
> **הטעות:** לבחור S3 כי הוא "אחסון".
> **הנכון:** **S3 לא ניתן ל-mount כ-file system.** התשובה היא **EFS** — עובד גם עם Fargate ומשותף בין AZ.

> [!warning] מלכודת 4 — EKS כברירת מחדל
> **הניסוח:** "צוות קטן, שירות containerized חדש, מינימום תפעול."
> **הטעות:** לבחור EKS כי "Kubernetes זה הסטנדרט".
> **הנכון:** EKS **מוסיף** תפעול — צריך לנהל K8s, וגם משלמים דמי cluster. אם המילה Kubernetes לא מופיעה כדרישה — התשובה היא ECS on Fargate.

> [!warning] מלכודת 5 — שני סוגי ה-Auto Scaling
> **הניסוח:** "ECS Service מתרחב ל-20 tasks אבל הם תקועים ב-PENDING."
> **הטעות:** להגדיל עוד את ה-Service Auto Scaling.
> **הנכון:** ב-EC2 Launch Type אין מספיק **instances**. צריך **Capacity Provider** או ASG scaling ברמת ה-EC2. ב-Fargate הבעיה הזו לא קיימת.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** חברת SaaS מפרקת מונוליט ל-4 מיקרו-שירותים. הצוות קטן ולא מכיר Kubernetes. כל שירות צריך הרשאות AWS שונות. שירות אחד מעבד קבצים גדולים וצריך אחסון משותף. יש גם עיבוד לילי כבד שרץ שעתיים.

```text
Users ──HTTPS──> Application Load Balancer
                        │ path-based routing
        ┌───────────────┼───────────────┬───────────────┐
        ▼               ▼               ▼               ▼
   ECS Service A   ECS Service B   ECS Service C   ECS Service D
   (Fargate)       (Fargate)       (Fargate)       (Fargate)
   TaskRole A      TaskRole B      TaskRole C      TaskRole D
   → DynamoDB      → SQS           → S3            → EFS mount
        │
        └─ Service Auto Scaling על ALB Request Count Per Target

   images ──> Amazon ECR (private, scanning + lifecycle policy)
   tasks   ──> private subnets, VPC Endpoints ל-ECR/S3 (בלי NAT)

   עיבוד לילי (שעתיים) ──> AWS Batch על Spot, מופעל ע"י EventBridge Schedule
   כשל task ──> EventBridge Event Pattern ──> SNS ──> אדמין
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| ECS ולא EKS | הצוות לא מכיר Kubernetes ואין דרישת multi-cloud — EKS רק יוסיף תפעול ועלות cluster |
| Fargate ולא EC2 | אין instances לתחזק, אין patching, וה-Service Auto Scaling פשוט בהרבה |
| ALB עם path-based routing | LB אחד לכל 4 השירותים; NLB מיותר כאן — אין דרישת throughput קיצונית |
| Task Role נפרד לכל שירות | least privilege — שירות ה-S3 לא צריך גישה ל-DynamoDB |
| ECR פרטי + image scanning | images פנימיים, בדיקת חולשות אוטומטית, lifecycle policy כדי לא לצבור אחסון |
| EFS לשירות הקבצים | S3 לא ניתן ל-mount; EFS עובד עם Fargate ומשותף בין AZ |
| VPC Endpoints ל-ECR/S3 | tasks ב-private subnet מושכים images בלי NAT Gateway יקר |
| AWS Batch על Spot לעיבוד הלילי | שעתיים חורגות מ-Lambda; Batch מקצה Spot ומכבה בסוף — עד ~90% חיסכון |
| EventBridge → SNS על stopped tasks | התראה מיידית על container שנפל, בלי polling |

**למה לא Lambda לכל השירותים?** העיבוד הלילי נמשך שעתיים — מעל תקרת 15 הדקות. ובנוסף, השירותים כבר ארוזים כ-images ורצים ברציפות.

**למה לא ECS on EC2?** אפשרי, ואף זול יותר בעומס יציב — אבל דורש ניהול AMI, patching ו-Capacity Providers. עם צוות קטן, ה-overhead לא שווה את החיסכון.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- כתיבת Kubernetes manifests, YAML של Deployments/Services, או פקודות `kubectl`.
- תחביר Dockerfile ואופטימיזציית layers.
- פרטי ה-ECS Agent ברמת קונפיגורציה.
- אלגוריתמי task placement strategies (binpack/spread/random) לעומק — מספיק המושג.
- שמות ה-CSI drivers המדויקים — מספיק לדעת אילו סוגי אחסון נתמכים ב-EKS.
- מבנה ה-Job Queue / Compute Environment ב-Batch ברמת קונפיגורציה.
- מחירים מדויקים לפי אזור.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **ECS** = orchestrator של AWS. **EKS** = Kubernetes מנוהל. **Fargate** = compute serverless (לא orchestrator). **ECR** = registry.
- Docker חולק את ה-**kernel** של ה-host; VM גורר Guest OS מלא.
- **EC2 Launch Type**: אתה מתחזק instances, כל אחד מריץ **ECS Agent**.
- **Fargate**: אין תשתית. רק Task Definition עם CPU/RAM. מתרחבים במספר tasks.
- **EC2 Instance Profile** = לסוכן (ECS API, לוגים, ECR, סודות). **ECS Task Role** = לאפליקציה. שני דברים שונים.
- **ALB** לרוב המקרים. **NLB** ל-throughput קיצוני או PrivateLink. **CLB** לא מומלץ ולא עובד עם Fargate.
- **EFS** ניתן ל-mount לתוך tasks בשני ה-launch types. **S3 לא ניתן ל-mount.**
- Fargate + EFS = אחסון persistent רב-AZ, serverless מלא.
- **ECS Service Auto Scaling ≠ EC2 Auto Scaling** — tasks מול instances.
- מטריקות scaling: CPU, Memory, ו-**ALB Request Count Per Target**.
- **Capacity Provider** מקצה EC2 אוטומטית כשחסר CPU/RAM ל-tasks.
- **ECR**: פרטי+ציבורי, מגובה ב-S3, גישה דרך IAM, image scanning, lifecycle policies.
- **EKS**: cluster אחד לכל region, node types = Managed / Self-Managed / Fargate, מטריקות ב-Container Insights.
- EKS volumes דרך **CSI**: EBS, EFS (היחיד עם Fargate), FSx for Lustre, FSx for NetApp ONTAP.
- **AWS Batch**: אין מגבלת זמן, כל runtime כ-Docker image, רץ על EC2/Spot דרך ECS/EKS/Fargate.
- ECS עצמו **חינם** — משלמים רק על ה-compute. ל-EKS יש דמי cluster קבועים.

---

## 11. ✅ בדיקת הבנה

1. השאלה אומרת "containers with the least operational overhead". מה עונים, ולמה לא EKS?
2. מה ההבדל בין EC2 Instance Profile ל-ECS Task Role, ומי צריך הרשאה למשוך image מ-ECR?
3. שלושה containers צריכים לקרוא ולכתוב לאותם קבצים משלוש AZ. מה בוחרים?
4. ECS Service התרחב ל-30 tasks אבל 10 תקועים ב-PENDING. מה קרה ומה הפתרון?
5. חברה מריצה Kubernetes on-premises ורוצה לעבור ל-AWS בלי לשכתב. מה בוחרים?
6. יש עיבוד שרץ 3 שעות על 50,000 קבצים. Lambda, Fargate או Batch?
7. איזו מטריקה מיוחדת מאפשרת ל-ECS להתרחב לפי עומס אמיתי מה-load balancer?

<details>
<summary>תשובות</summary>

1. **ECS on Fargate**. אין instances לנהל, אין patching, וה-Auto Scaling ברמת tasks בלבד. EKS דווקא **מוסיף** overhead — צריך לתפעל Kubernetes וגם משלמים דמי cluster קבועים.
2. **Instance Profile** שייך ל-EC2 instance ומשמש את ה-ECS Agent: קריאות ל-ECS API, שליחת לוגים ל-CloudWatch, **משיכת image מ-ECR**, וקריאת סודות. **Task Role** מוגדר ב-Task Definition ונותן לאפליקציה עצמה גישה ל-S3/DynamoDB וכו'. משיכת ה-image היא באחריות ה-Instance Profile (או Task Execution Role ב-Fargate).
3. **EFS** — mount לתוך ה-tasks, עובד ב-EC2 וב-Fargate, ומשותף בין כל ה-AZ. S3 אינו ניתן ל-mount כ-file system.
4. אין מספיק **EC2 capacity** ב-cluster (EC2 Launch Type). ECS Service Auto Scaling מוסיף tasks אבל לא instances. הפתרון: **ECS Cluster Capacity Provider** משולב ב-ASG, או scaling של ה-ASG לפי CPU. ב-Fargate הבעיה לא קיימת.
5. **EKS**. Kubernetes הוא cloud-agnostic, וזה בדיוק ה-use case שמופיע בשקפים — מיגרציה מ-K8s קיים לענן.
6. **AWS Batch**. Lambda נופלת על תקרת 15 הדקות. Batch מיועד בדיוק ל"job עם התחלה וסוף", מקצה EC2/Spot דינמית ומכבה בסוף — הזול ביותר לעבודה מהסוג הזה.
7. **ALB Request Count Per Target** — מטריקה שמגיעה מה-ALB ומשקפת עומס בקשות אמיתי לכל target, לא רק CPU.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[25 - Lambda]] · [[27 - API Gateway]] · [[20 - EFS and File Storage]] · [[08 - Elastic Load Balancing]] · [[07 - Auto Scaling]] · [[30 - Application Decoupling]] · [[38 - Serverless and Modern Architectures]] · [[06 - EC2 Pricing and Optimization]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 7495–7938, 16006–16048
