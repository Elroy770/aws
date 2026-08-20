---
lesson: 30
title: Application Decoupling
domain: Design High-Performing Architectures
services: [EC2, ELB, EC2 Auto Scaling, Route 53, ElastiCache, RDS, DynamoDB, EFS, EBS, SQS, SNS, Elastic Beanstalk, API Gateway, Lambda, ECS, CloudFront]
tags: [saa-c03, architecture, decoupling, stateless, three-tier, microservices]
---

# 30 — Application Decoupling

> [!abstract] בשורה אחת
> זה שיעור **ארכיטקטורה** ולא שיעור שירות: הוא לוקח אפליקציה משרת בודד ועד microservices,
> ובכל שלב שואל שלוש שאלות — **מה נשבר, מה תיקנו, ומה זה עלה**.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה ארכיטקטורה מתפתחת ולא נולדת מושלמת |
| 2 | איך זה עובד | **הסיפור האבולוציוני:** שרת בודד → vertical → horizontal → ELB → ASG → Multi-AZ |
| 3 | פירוק מפורט | Stateless מול Stateful, איפה מאחסנים session, EBS מול EFS, Beanstalk, Microservices |
| 4 | עלות | מה כל שלב באבולוציה עולה, ואיפה חוסכים |
| 5 | השוואות | stickiness מול cookies מול session store; monolith מול microservices |
| 6 | Well-Architected | ארכיטקטורה מנותקת לפי ששת ה-Pillars |
| 7 | מלכודות | "EBS משותף בין AZs" ו-"תור מאיץ עיבוד" |
| 8 | Scenario | הפיכת monolith עמוס לארכיטקטורה מנותקת |

**מונחי מפתח בשיעור:** `Stateless` · `Stickiness` · `Session Store` · `Golden AMI` · `Worker Tier` · `Backpressure` · `Loose Coupling`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- כל אפליקציה מתחילה כשרת אחד, וכל אפליקציה מצליחה נשברת בדיוק שם.
- **צימוד הדוק (tight coupling)** אומר שכשל ברכיב אחד מפיל את כולם.
- כשה-frontend קורא ישירות ל-backend ומחכה, פיק תעבורה מפיל את **שניהם**.
- **State על השרת** (עגלת קניות בזיכרון) הופך התרחבות אופקית לבלתי אפשרית.
- אי אפשר לשחרר גרסה של רכיב אחד בלי לתאם עם כל השאר.

### מה השירות פותר

- **Loose Coupling** — כל רכיב מכיר **ממשק**, לא מימוש: DNS name, ARN של תור, endpoint של ALB.
- **התרחבות עצמאית** — כל שכבה גדלה לפי העומס שלה בלבד.
- **בידוד כשלים** — רכיב שנפל לא מפיל את השרשרת כולה.
- **Backpressure** — התור בולם את הפיק במקום ש-DB או worker יתמוטטו.
- **מהירות פיתוח** — צוותים משחררים בנפרד.

> [!tip] האנלוגיה
> ארכיטקטורה מצומדת היא **שרשרת** — החוליה החלשה קובעת את חוזק הכול.
> ארכיטקטורה מנותקת היא **קו ייצור עם מלאי ביניים** — עצירה בתחנה אחת לא עוצרת את המפעל,
> כי יש מאגר שסופג את ההפרש.

---

## 2. ⚙️ איך זה עובד — הסיפור האבולוציוני

נעקוב אחרי אפליקציה אמיתית. בכל שלב: **מה נשבר, מה תיקנו, מה זה עלה.**

### 2.1 שלב 0 — שרת בודד

```text
User ──► Elastic IP ──► Public EC2 (t3.micro)
```

- אפליקציה **stateless** פשוטה. אין DB. נשמע מצוין.
- **מה נשבר:** התעבורה גדלה והשרת נחנק.

### 2.2 שלב 1 — Vertical Scaling (Scale Up)

```text
User ──► Elastic IP ──► EC2  t3.micro → t3.large → m5.xlarge
                             └─ downtime בכל שדרוג
```

| מה תיקנו | מה זה עלה |
|---|---|
| השרת חזק יותר וסופג יותר עומס | **Downtime בכל שדרוג** — עוצרים, משנים type, מפעילים |
| שינוי אחד ומהיר, בלי שינוי בקוד | **תקרה קשיחה** — אין instance גדול אינסופית |
| | **נקודת כשל יחידה** נשארה בדיוק כפי שהייתה |

> [!info] הכלל
> Vertical Scaling הוא הפתרון המהיר, לא הפתרון הנכון.
> **הוא לא פותר זמינות** — עדיין שרת אחד.

### 2.3 שלב 2 — Horizontal Scaling עם DNS

```text
                      ┌──► EC2 #1 (Public IP)
User ──DNS Query──►   ├──► EC2 #2 (Public IP)
   api.example.com    └──► EC2 #3 (Public IP)
   A Record, TTL 1h
```

| מה תיקנו | מה נשבר עכשיו |
|---|---|
| כמה שרתים חולקים את העומס | **instance נופל — והלקוחות עדיין מקבלים את ה-IP שלו** |
| אין תקרה קשיחה | **TTL של שעה** אומר שלקוח יתקע על IP מת עד שעה |
| | כל שינוי בצי דורש **עדכון רשומות DNS** ידני |
| | כל instance חשוף ישירות לאינטרנט עם IP ציבורי |

> [!warning] הלקח מהשלב הזה
> **DNS הוא לא Load Balancer.** אין לו health checks בעולם הזה, ו-**TTL הוא זיכרון של הלקוח**
> שאתם לא שולטים בו. זו בדיוק הסיבה שההמשך הוא ELB.

### 2.4 שלב 3 — Elastic Load Balancer

```text
User ──DNS Alias Record──► ELB ──┬──► EC2 #1 (Private subnet)
                          + Health Checks
                                 ├──► EC2 #2 (Private subnet)
                                 └──► EC2 #3 (Private subnet)
```

| מה תיקנו | מה זה עלה |
|---|---|
| **Health Checks** — instance לא תקין מוסר מהרוטציה תוך שניות | עלות שעתית + LCU של ה-ELB |
| **Alias Record** ב-Route 53 → ה-ELB, בלי לנהל IPs | |
| ה-EC2 עברו ל-**subnet פרטי בלי IP ציבורי** — שיפור אבטחה עצום | |
| **Security Group של ה-EC2 מתיר תעבורה רק מה-SG של ה-ELB** | |
| נקודת כניסה יחידה ויציבה | |

**מה עדיין נשבר:** מישהו צריך להוסיף ולהסיר instances ידנית.

### 2.5 שלב 4 — Auto Scaling Group

```text
User ──► ELB ──► ┌─ Auto Scaling Group ─────────┐
                 │  EC2  EC2  EC2  ... (min–max)│
                 └──────────────────────────────┘
```

| מה תיקנו | מה זה עלה |
|---|---|
| הצי גדל ומתכווץ **אוטומטית** לפי מטריקה | צריך AMI/User Data שמאפשרים launch אוטומטי |
| instance שנכשל ב-health check **מוחלף אוטומטית** | האפליקציה **חייבת להיות stateless** — אחרת ה-terminate מוחק state |
| משלמים רק על מה שרץ | |

פירוט מלא ב-[[07 - Auto Scaling]].

### 2.6 שלב 5 — Multi-AZ

```text
                    ┌── AZ-1: EC2, EC2
User ──► ELB ──►────┼── AZ-2: EC2, EC2
       (Multi-AZ)   └── AZ-3: EC2, EC2
```

| מה תיקנו | מה זה עלה |
|---|---|
| **כשל AZ שלמה לא מפיל את המערכת** | תעבורה **cross-AZ** מחויבת |
| ה-ELB עצמו פרוס בכמה AZs | צריך subnets מקבילים בכל AZ |
| ASG מאזן instances בין AZs אוטומטית | |

> [!tip] הצעד הכלכלי בשלב הזה
> אם ה-ASG **חייב מינימום** של N instances שרצים תמיד (למשל 2 לכל AZ) —
> אותה קיבולת מינימלית היא מועמדת מושלמת ל-**Reserved Instances או Savings Plans**
> (עד ~72% הנחה). את הפיק משאירים On-Demand או Spot. ראו [[06 - EC2 Pricing and Optimization]].

### 2.7 שלב 6 — האפליקציה הפכה Stateful

עכשיו יש עגלת קניות. הבעיה: המשתמש מגיע ל-instance אחר בכל בקשה, והעגלה נעלמת.

**ארבע דרכים לפתור, לפי סדר האבולוציה:**

**א. ELB Stickiness (Session Affinity)**

```text
User ──► ELB ──[cookie מכוון תמיד לאותו instance]──► EC2 #2
```

- הכי פשוט. אפס שינוי בקוד.
- **הבעיה:** ה-instance שהוקצה יכול למות → **המשתמש איבד את הסל**.
- **בעיה נוספת:** עומס לא מאוזן — instance עם משתמשים "כבדים" נחנק.

**ב. Cookies בצד הלקוח**

```text
User ──[cookie מכיל את תוכן העגלה]──► ELB ──► כל instance
```

| יתרון | חיסרון |
|---|---|
| האפליקציה **stateless לחלוטין** | **כל בקשת HTTP כבדה יותר** — ה-cookie נשלח בכל פעם |
| אין תשתית נוספת | **סיכון אבטחה — הלקוח יכול לשנות את ה-cookie** |
| | חובה **לאמת (validate)** את ה-cookie בכל בקשה |
| | **הגבלת גודל: פחות מ-4KB** |

**ג. Session Store חיצוני — הפתרון הנכון**

```text
User ──[cookie מכיל session_id בלבד]──► ELB ──► EC2 ──► ElastiCache
                                                        (store/retrieve session)
```

- ה-cookie מכיל **רק מזהה** — קטן, ולא ניתן לזייף ממנו נתונים.
- ה-state נשמר ב-**ElastiCache** (Redis/Memcached) — **latency של תת-מילישנייה**.
- **חלופה: DynamoDB** — serverless, ללא ניהול, עם TTL אוטומטי לפקיעת sessions.
- כל instance יכול לשרת כל משתמש → **stateless לחלוטין** → ASG חופשי להרוג ולהחליף.

**ד. נתוני משתמש קבועים → מסד נתונים**

- כתובת, שם, היסטוריית הזמנות — לא session, אלא **נתון קבוע** → **RDS**.
- הפרדה חשובה: **session = זמני, ב-cache. user data = קבוע, ב-DB.**

### 2.8 שלב 7 — הרחבת קריאות

ה-DB נחנק. שתי גישות משלימות:

```text
גישה 1 — Read Replicas
App ──writes──► RDS Master ──replication──► Read Replica #1
    ──reads───────────────────────────────► Read Replica #2

גישה 2 — Cache (Lazy Loading)
App ──► ElastiCache ──hit──► מחזיר מיד
              │ miss
              ▼
            RDS ──► App כותב לcache ומחזיר
```

| קריטריון | Read Replicas | ElastiCache |
|---|---|---|
| מה פותר | throughput של קריאות | **latency** + throughput |
| השהיה | milliseconds | **sub-millisecond** |
| עקביות | **eventual** — יש lag ברפליקציה | תלוי במדיניות ה-cache |
| שינוי בקוד | הפניית קריאות ל-endpoint אחר | **לוגיקת cache-aside בקוד** |
| עלות | instance נוסף מלא | node של cache, לרוב זול יותר |

הפירוט המלא ב-[[22 - RDS Scaling and Availability]].

### 2.9 שלב 8 — Multi-AZ בשכבת הנתונים ו-Security Groups משורשרים

```text
Internet ──0.0.0.0/0 על 80/443──► ELB SG
                                    │  ↓ מקור = ELB SG בלבד
                                  EC2 SG
                                    │  ↓ מקור = EC2 SG בלבד
                          ┌─────────┴─────────┐
                    ElastiCache SG         RDS SG
```

- **ElastiCache Multi-AZ · RDS Multi-AZ** — כל שכבה שורדת כשל AZ.
- **SG שמפנה ל-SG אחר** ולא ל-CIDR — הכלל נשאר תקף גם כשה-IPs משתנים כל הזמן ב-ASG.
- זו הצורה הנכונה לבנות שכבות; פירוט ב-[[11 - VPC Security]].

### 2.10 שלב 9 — Decoupling עם SQS

עד עכשיו כל השכבות עדיין **סינכרוניות**. שני דפוסים משנים את זה:

**א. ניתוק בין שכבות (decouple tiers)**

```text
requests ──► Front-end Web App (ASG) ──SendMessage──► SQS Queue
                                                         │
                                        ReceiveMessages  ▼
                                            Back-end Processing (ASG)
```

- ה-frontend מחזיר תשובה **מיד** ולא מחכה לעיבוד.
- ה-backend מתרחב לפי **עומק התור**, לא לפי תעבורת HTTP.
- קריסת ה-backend לא נראית ללקוח — רק גורמת לתור להתארך.

**ב. תור כ-buffer לכתיבות ל-DB**

```text
לפני:  requests ──► App (ASG) ──insert──► RDS
       בעומס גבוה: חלק מהעסקאות נכשלות ואובדות

אחרי:  requests ──► App ──enqueue──► SQS ──dequeue──► Workers ──insert──► RDS
       התור סופג את הפיק; קצב הכתיבה נקבע על ידי מספר ה-workers
```

- זו התשובה לכל שאלה שבה כתוב **"transactions may be lost under heavy load"**.
- פירוט מלא של SQS ב-[[28 - SQS and SNS]].

### 2.11 שלב 10 — Microservices

```text
                          ┌─ service1.example.com ──► ELB ──► ECS ──► DynamoDB
Users ──► Route 53 ──►────┼─ service2.example.com ──► API Gateway ──► Lambda ──► ElastiCache
                          └─ service3.example.com ──► ELB ──► EC2 ASG ──► RDS
```

- כל שירות מתקשר עם האחרים דרך **REST API**, ולכל אחד **ארכיטקטורה משלו**.
- **דפוסים סינכרוניים:** API Gateway, Load Balancers.
- **דפוסים אסינכרוניים:** SQS, SNS, Kinesis, Lambda triggers (למשל מ-S3).
- כל שירות בוחר את מסד הנתונים שמתאים לו — DynamoDB, RDS, ElastiCache.

**האתגרים האמיתיים של microservices:**

| אתגר | פירוט |
|---|---|
| **Overhead חוזר** | כל שירות חדש דורש pipeline, ניטור, IAM, רשת — מחדש |
| **ניצול שרתים** | server density נמוך: הרבה שירותים קטנים על הרבה instances חצי-ריקים |
| **ריבוי גרסאות** | כמה גרסאות של כמה שירותים רצות במקביל — סיוט תאימות |
| **מורכבות בצד הלקוח** | הלקוח צריך לדעת לדבר עם עשרות שירותים נפרדים |

**מה Serverless פותר מזה:**

- **API Gateway ו-Lambda מתרחבים אוטומטית** ומשלמים **לפי שימוש** — נעלמת בעיית ה-density.
- **שכפול API וסביבות** הופך לטריוויאלי.
- **יצירת SDK ללקוח** אוטומטית מ-API Gateway (Swagger/OpenAPI).

ראו [[26 - Containers]], [[27 - API Gateway]], [[38 - Serverless and Modern Architectures]].

---

## 3. 🔍 פירוק מפורט

### 3.1 Stateless מול Stateful — הטבלה המרכזית

| קריטריון | **Stateless** | **Stateful** |
|---|---|---|
| איפה ה-state | **מחוץ לשרת** — cache/DB/cookie | **בזיכרון או בדיסק של השרת** |
| החלפת instance | שקופה לחלוטין | **המשתמש מאבד את ה-session** |
| Auto Scaling | עובד מצוין | בעייתי — כל terminate כואב |
| דורש stickiness | לא | **כן** |
| התאוששות מכשל | מיידית | דורשת שחזור state |
| מתאים ל | web tier, API tier, workers | DB, cache, file server |

> [!warning] הכלל הזהב
> **שכבת ה-web חייבת להיות stateless.** ה-state עובר לשכבה שנועדה לו:
> ElastiCache ל-sessions, RDS/DynamoDB לנתונים, EFS/S3 לקבצים.
> ASG שמנהל instances stateful הוא באג ממתין.

### 3.2 איפה מאחסנים Session State — ארבע האפשרויות

| שיטה | איפה נשמר | יתרון | חיסרון | מתי |
|---|---|---|---|---|
| **ELB Stickiness** | על ה-instance | אפס שינוי בקוד | **מות instance = אובדן session**; עומס לא מאוזן | פתרון זמני בלבד |
| **Cookie מלא** | **אצל הלקוח** | stateless אמיתי, אין תשתית | **מוגבל ל-4KB**; בקשות כבדות; **ניתן לזיוף — חובה validation** | נתונים קטנים ולא רגישים |
| **ElastiCache** | Redis / Memcached | **sub-millisecond**; מתאים ל-sessions בקצב גבוה | node לתחזק, עלות שעתית | **ברירת המחדל לאפליקציית web** |
| **DynamoDB** | טבלה מנוהלת | serverless, **TTL אוטומטי** לניקוי, ללא ניהול | single-digit ms — איטי מ-cache | ארכיטקטורה serverless |

> [!tip] מה שנשאל במבחן
> "Users lose their shopping cart when an instance is replaced" →
> **להוציא את ה-session ל-ElastiCache או ל-DynamoDB.**
> Stickiness היא תשובה מפתה אבל היא **לא פותרת** את מות ה-instance.

### 3.3 EBS מול EFS — הקייס של MyWordPress.com

זהו אחד ה-scenarios שחוזרים הכי הרבה: אתר WordPress שמאפשר העלאת תמונות.

**ניסיון 1 — EBS:**

```text
AZ-1: EC2 ──► EBS Volume  [תמונה A]
AZ-2: EC2 ──► EBS Volume  [תמונה B]

המשתמש מעלה תמונה דרך instance ב-AZ-1.
הבקשה הבאה מגיעה ל-instance ב-AZ-2 → התמונה לא קיימת שם. ❌
```

- **EBS מוצמד ל-instance אחד ול-AZ אחת.** לא ניתן לשיתוף בין AZs.
- מתאים **רק לאפליקציה שרצה על instance בודד**.

**ניסיון 2 — EFS:**

```text
AZ-1: EC2 ──ENI──┐
                 ├──► EFS (מערכת קבצים משותפת, Multi-AZ)
AZ-2: EC2 ──ENI──┘

כל instance רואה את אותם קבצים בדיוק. ✅
```

- **EFS הוא NFS משותף** שנגיש מכל AZ דרך **Mount Target (ENI)** בכל subnet.
- זו התשובה ל**אפליקציה מבוזרת** שצריכה מערכת קבצים משותפת.

| קריטריון | EBS | EFS |
|---|---|---|
| **Scope** | **AZ אחת, instance אחד** | **Region — כל ה-AZs** |
| שיתוף | לא (למעט Multi-Attach ב-io1/io2 באותה AZ) | **כן, אלפי instances במקביל** |
| פרוטוקול | block device | **NFS** |
| מחיר | זול יותר ל-GB | **יקר משמעותית ל-GB** |
| מתי | root volume, DB, אפליקציה יחידה | **content משותף, uploads, WordPress** |

> [!info] החלופה הטובה ביותר לשניהם
> ל-static assets ותמונות משתמשים, **S3 עדיף על שניהם**: זול בהרבה, עמיד ב-11 תשיעיות,
> ומתחבר ישירות ל-CloudFront. EFS נבחר כשהאפליקציה **חייבת** ממשק של מערכת קבצים ואי אפשר לשנות קוד.

### 3.4 השכבה הנתונית — התקדמות ל-Aurora

- **RDS Multi-AZ** — standby סינכרוני ב-AZ אחרת, failover אוטומטי.
- **Aurora MySQL** — Multi-AZ ו-Read Replicas בקלות רבה יותר, עד 15 רפליקות עם lag נמוך.
- ראו [[21 - RDS Fundamentals]] ו-[[22 - RDS Scaling and Availability]].

### 3.5 הקמה מהירה — Golden AMI ו-User Data

כשמשיקים stack שלם (EC2 + EBS + RDS), הזמן הולך על התקנה, הזרמת נתונים והגדרות.
שלוש דרכים לקצר:

| שיטה | מה עושים | מתי מתאים |
|---|---|---|
| **Golden AMI** | מתקינים אפליקציה ותלויות **מראש**, יוצרים AMI, ומשיקים ממנו | הרוב המכריע של ההתקנה — **הכי מהיר** |
| **User Data (bootstrap)** | סקריפט שרץ בהפעלה הראשונה | **הגדרות דינמיות** שמשתנות בין סביבות |
| **היברידי** | Golden AMI + User Data | **בדיוק מה ש-Elastic Beanstalk עושה** |
| **RDS from Snapshot** | שחזור מ-snapshot — סכמה ונתונים כבר בפנים | הקמת סביבות dev/test |
| **EBS from Snapshot** | הדיסק כבר מפורמט ועם נתונים | שחזור מהיר |

> [!tip] למה זה מופיע דווקא כאן
> ASG לא יכול להתרחב מהר אם כל instance חדש דורש 10 דקות של `apt install`.
> **Golden AMI הוא מה שהופך elasticity אמיתית לאפשרית.**

### 3.6 Elastic Beanstalk — הארכיטקטורה כשירות

**הבעיה שהוא פותר:** רוב אפליקציות ה-web הן בדיוק **ALB + ASG**, והמפתחים רק רוצים שהקוד ירוץ.

| מאפיין | פירוט |
|---|---|
| מה זה | מבט **developer-centric** על deployment ב-AWS |
| מה מתחת | משתמש ברכיבים המוכרים: **EC2, ASG, ELB, RDS** |
| מה מנוהל | provisioning, load balancing, scaling, health monitoring, הגדרת instances |
| מה באחריותכם | **רק הקוד** |
| שליטה | **שליטה מלאה בהגדרות** נשמרת |
| **עלות** | **Beanstalk עצמו חינם.** משלמים רק על המשאבים שמתחת |

**שלושת המושגים:**

| מושג | מה זה |
|---|---|
| **Application** | אוסף של environments, versions והגדרות |
| **Application Version** | איטרציה של הקוד |
| **Environment** | אוסף משאבי AWS שמריצים **גרסה אחת** בכל רגע. אפשר dev/test/prod |

**שני סוגי Environment Tiers — וזו הנקודה הרלוונטית לשיעור הזה:**

```text
Web Server Tier                        Worker Tier
myapp.us-east-1.elasticbeanstalk.com
      │                                       ▲
    ALB                                       │ pull messages
      │                                  SQS Queue
   ASG(EC2 web servers) ──push messages──────┘
                                         ASG(EC2 workers)
                                         ↑ מתרחב לפי מספר ההודעות ב-SQS
```

- **Web Server Tier** — מטפל בבקשות HTTP מול משתמשים.
- **Worker Tier** — **מושך הודעות מ-SQS** ומעבד ברקע.
- ה-**Worker Tier מתרחב לפי מספר ההודעות בתור** — Beanstalk בנה את דפוס ה-decoupling בשבילכם.
- ה-Web Tier יכול **לדחוף הודעות לתור** של ה-Worker Tier.

**שני מצבי Deployment:**

| מצב | מבנה | מתי |
|---|---|---|
| **Single Instance** | instance אחד + Elastic IP (+ RDS יחיד) | **dev** |
| **High Availability with Load Balancer** | ALB + ASG בכמה AZs (+ RDS Multi-AZ) | **prod** |

**פלטפורמות נתמכות:** Go, Java SE, Java with Tomcat, .NET (Linux/Windows), Node.js, PHP, Python, Ruby,
Packer Builder, Docker (single container, multi-container, preconfigured).

### 3.7 Offloading — לנתק עומס בלי לגעת בקוד

**התרחיש:** אפליקציה על EC2 מפיצה עדכוני תוכנה. בכל שחרור גרסה יש הצפת בקשות
ותכנים כבדים שנשלחים ברשת. יקר ב-CPU וב-bandwidth.

```text
לפני:  Users ──► ELB ──► ASG(EC2) ──► EFS   ← הצי גדל אדיר בכל שחרור

אחרי:  Users ──► CloudFront ──► ELB ──► ASG(EC2) ──► EFS
                 (cache at edge)          ← הצי כמעט לא גדל
```

**למה זה עובד:**

- קובצי עדכון הם **סטטיים** — לא משתנים לעולם. מושלמים ל-cache.
- ה-EC2 אינם serverless, **אבל CloudFront כן** — הוא מתרחב במקומם.
- **אין שינוי בארכיטקטורה ואין שינוי בקוד.**
- חוסכים ב-EC2, ב-bandwidth של המקור, ומשפרים זמינות ו-latency.

זו הצורה הזולה ביותר של decoupling: **להוציא עומס מהמערכת במקום להתרחב איתו.**
ראו [[15 - CloudFront and Global Delivery]].

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים בכל שלב באבולוציה

| שלב | רכיב חיוב | הערה |
|---|---|---|
| שרת בודד | EC2 שעות + EBS | הזול, אבל בלי זמינות |
| Vertical Scaling | instance גדול יותר | **קפיצה לא לינארית** בעלות |
| Horizontal + ELB | EC2 × N + **ELB שעות + LCU** | |
| ASG | רק ה-instances שרצים בפועל | **החיסכון האמיתי** — לא משלמים על פיק כל היום |
| Multi-AZ | **cross-AZ data transfer** | לכל GB, בשני הכיוונים |
| ElastiCache | node-hours | |
| RDS Multi-AZ | **פי ~2** — standby מחויב במלואו | |
| Read Replicas | instance מלא לכל רפליקה | |
| EFS | GB מאוחסן, **יקר משמעותית מ-EBS** | |
| SQS | requests ב-chunks של 64KB | ראו [[28 - SQS and SNS]] |
| **Elastic Beanstalk** | **0 על השירות** | משלמים רק על EC2/ELB/RDS מתחת |
| CloudFront | GB יוצא + requests | לרוב **זול יותר** מ-data transfer ישיר מ-EC2 |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **Offload ל-CloudFront/S3** | **הזול ביותר** — מסיר עומס במקום להתרחב | תוכן סטטי, תמיד |
| ASG עם min מבוסס RI/SP | חיסכון **עד ~72%** על הבסיס | הקיבולת המינימלית שרצה 24/7 |
| Spot לצי ה-workers של SQS | **עד ~90% הנחה** | עיבוד אסינכרוני שסובל הפרעות |
| ElastiCache במקום Read Replicas | לרוב זול יותר | קריאות חוזרות על אותם נתונים |
| **EFS במקום S3** לתמונות | יקר משמעותית | רק כשחייבים ממשק filesystem |
| **RDS Multi-AZ** | פי ~2 | דרישת HA — לא ניתן לוותר בפרודקשן |
| Vertical Scaling מתמשך | יקר ולא פותר זמינות | כמעט אף פעם |

### 🚩 עלויות נסתרות

- **Cross-AZ data transfer** — ה-Multi-AZ שהצלתם איתו את הזמינות מחויב לכל GB שעובר בין AZs.
- **ELB LCU** — חיוב לפי connections, בקשות ורוחב פס, לא רק לפי שעות.
- **Read Replicas ששכחו מהן** — instance מלא שמחויב 24/7 גם בלי תעבורה.
- **EFS שגדל בשקט** — אין תקרה, וחשבון האחסון מטפס.
- **Beanstalk "חינם"** — ה-environment מריץ ALB + ASG + לפעמים RDS, וזה מה שמחויב.
- **צי workers שלא מתכווץ** — ASG בלי scale-in policy נכון נשאר גדול אחרי הפיק.
- **NAT Gateway** — כל השכבה הפרטית יוצאת דרכו, וזה מחויב לשעה **ולכל GB**.

### 💡 טיפים לחיסכון

- **להוציא static ל-S3 + CloudFront** לפני שמתרחבים ב-EC2. זה החיסכון הגדול ביותר.
- **RI/Savings Plans על ה-min של ה-ASG**, On-Demand/Spot על הפיק.
- **Spot Instances לצי ה-workers** שקורא מ-SQS — הפסקה היא רק הודעה שחוזרת לתור.
- **לשמור תעבורה בתוך AZ** כשאפשר.
- **DynamoDB with TTL ל-sessions** — ניקוי אוטומטי בלי node שרץ 24/7.
- **Gateway Endpoint ל-S3** במקום מסלול דרך NAT.
- **לכבות סביבות Beanstalk של dev/test** מחוץ לשעות עבודה.

---

## 5. ⚖️ השוואות מכריעות

### שלוש הדרכים לטפל ב-session

| קריטריון | ELB Stickiness | Cookie בצד הלקוח | Session Store (ElastiCache/DynamoDB) |
|---|---|---|---|
| האפליקציה stateless? | ❌ | ✅ | ✅ |
| שורד מות instance | ❌ | ✅ | ✅ |
| גודל ה-state | ללא הגבלה | **מתחת ל-4KB** | ללא הגבלה מעשית |
| אבטחה | תקין | **ניתן לזיוף — חובה validation** | תקין |
| משקל הבקשה | רגיל | **כבד יותר** בכל בקשה | רגיל |
| תשתית נוספת | אין | אין | **כן** |
| **המלצה** | פתרון ביניים | נתונים קטנים בלבד | **הפתרון הנכון** |

### Monolith מול Microservices

| קריטריון | Monolith | Microservices |
|---|---|---|
| Deployment | הכול ביחד | כל שירות בנפרד |
| התרחבות | מתרחב הכול כדי לפתור צוואר אחד | **רק השירות העמוס** |
| בידוד כשלים | חלש | **חזק** |
| מורכבות תפעולית | נמוכה | **גבוהה** — ניטור, גרסאות, רשת |
| ניצול שרתים | טוב | **נמוך** — הרבה instances חצי-ריקים |
| מורכבות בלקוח | נמוכה | **גבוהה** — עשרות endpoints |
| **מתי** | התחלה, צוות קטן | **צוותים רבים, קצבי שינוי שונים** |

### תקשורת סינכרונית מול אסינכרונית

| קריטריון | סינכרוני (API Gateway / ELB) | אסינכרוני (SQS / SNS / Kinesis) |
|---|---|---|
| תשובה מיידית | ✅ | ❌ |
| התנהגות ב-spike | הצד המקבל קורס | **התור סופג** |
| כשל בצרכן | הבקשה נכשלת | **ההודעה ממתינה** |
| מורכבות | נמוכה | eventual consistency, כפילויות |
| **מתי** | המשתמש מחכה לתשובה | **עיבוד ברקע** |

> [!info] שורה תחתונה
> **מוסיפים רכיב רק אחרי שמזהים מה בדיוק נשבר.** אין ערך ב-microservices לאפליקציה עם צוות אחד,
> ואין ערך ב-ElastiCache לאפליקציה שכבר stateless.
> האבולוציה הזאת היא **סדר פעולות**, לא רשימת קניות.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בארכיטקטורה מנותקת** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | instance הוא מוצר מתכלה, לא נכס | **Golden AMI + User Data**; Beanstalk/IaC לסביבות זהות; dashboards על queue depth ועל health checks |
| **Security** | כל שכבה נגישה רק לשכבה שמעליה | **SG שמפנה ל-SG** ולא ל-CIDR; EC2 ב-subnet פרטי בלי IP ציבורי; ולידציה של כל cookie מהלקוח |
| **Reliability** | שום רכיב יחיד אינו נקודת כשל | **Multi-AZ בכל שכבה**; ELB Health Checks; ASG שמחליף אוטומטית; **SQS שסופג** נפילת backend |
| **Performance Efficiency** | כל שכבה מתרחבת לפי מה שמעמיס עליה | ASG על מטריקה נכונה; **ElastiCache ל-sessions ולקריאות חוזרות**; **CloudFront לסטטי** |
| **Cost Optimization** | לא משלמים על קיבולת שרויה בטל | **RI/SP על ה-min של ה-ASG**, Spot ל-workers; offload ל-S3+CloudFront; scale-in אגרסיבי |
| **Sustainability** | פחות שרתים לאותה עבודה | cache במקום עוד instances; batching בעיבוד אסינכרוני; right-sizing אחרי כל שלב באבולוציה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "users lose their shopping cart when scaling in" | **session ב-ElastiCache או DynamoDB** |
| "shared file system across multiple instances / AZs" | **EFS** (לא EBS) |
| "single instance application needs a disk" | **EBS** |
| "downtime while resizing the server" | Vertical Scaling → **לעבור ל-ASG + ELB** |
| "clients still hitting a terminated instance" | **DNS TTL** → **ELB עם health checks** |
| "transactions are lost during traffic spikes" | **SQS כ-buffer לפני ה-DB** |
| "decouple the front-end from the back-end" | **SQS** |
| "developer just wants to deploy code, no infra" | **Elastic Beanstalk** |
| "background processing that scales with the queue" | **Beanstalk Worker Tier** |
| "reduce load and cost for static software updates" | **CloudFront** |
| "launch new instances faster" | **Golden AMI** (+ User Data להגדרות) |
| "reserve the minimum capacity" | **Reserved Instances / Savings Plans** על ה-min של ה-ASG |
| "restrict database access to the app servers only" | **SG של ה-DB שמקורו ב-SG של ה-EC2** |

### טעויות נפוצות

> [!warning] מלכודת 1 — EBS משותף בין instances
> **הניסוח:** "Attach the same EBS volume to instances in two Availability Zones so they share uploads."
> **הטעות:** להניח ש-EBS הוא storage משותף.
> **הנכון:** **EBS שייך ל-AZ אחת ולרוב ל-instance אחד.** לשיתוף בין AZs — **EFS** (או S3).

> [!warning] מלכודת 2 — Stickiness פותר את בעיית ה-session
> **הניסוח:** "Enable ELB sticky sessions so users never lose their cart."
> **הטעות:** לחשוב ש-affinity = עמידות.
> **הנכון:** ה-instance שאליו הודבקתם עדיין יכול למות, **ואז ה-session אבד**.
> הפתרון האמיתי: **להוציא את ה-session מהשרת** ל-ElastiCache/DynamoDB.

> [!warning] מלכודת 3 — תור מאיץ עיבוד
> **הניסוח:** "Add SQS to make order processing faster."
> **הטעות:** לחשוב שתור משפר throughput.
> **הנכון:** תור **לא מאיץ כלום.** הוא **מפריד וסופג**.
> הוא הופך את המערכת לעמידה ומאפשר ל-workers להתרחב — אבל ה-latency מקצה לקצה בדרך כלל **עולה**.

> [!warning] מלכודת 4 — לסמוך על DNS כ-Load Balancer
> **הניסוח:** "Add all instance IPs as A records; DNS will distribute the load."
> **הטעות:** להתייחס ל-round-robin DNS כאיזון עומסים.
> **הנכון:** אין **health checks** אמיתיים, ו-**TTL אצל הלקוח** גורם לתעבורה לזרום ל-instance מת.
> הפתרון: **ELB + Alias Record**.

> [!warning] מלכודת 5 — Cookie ככספת
> **הניסוח:** "Store the user's account balance in a browser cookie to keep the app stateless."
> **הטעות:** להניח שנתון ב-cookie אמין.
> **הנכון:** **הלקוח יכול לשנות כל cookie.** נתונים רגישים לעולם לא נשמרים שם.
> ב-cookie שומרים **session_id בלבד**, וכל מה שמגיע מהלקוח **חייב validation**.
> וגם: המגבלה היא **מתחת ל-4KB**.

> [!warning] מלכודת 6 — "Beanstalk עולה כסף"
> **הניסוח:** "Elastic Beanstalk adds a management fee on top of the resources."
> **הטעות:** להניח תוספת תשלום.
> **הנכון:** **Beanstalk עצמו חינם.** משלמים אך ורק על ה-EC2, ה-ELB, ה-RDS וכו' שמתחת.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חנות אונליין רצה כ-monolith על שני שרתי EC2 גדולים מאחורי ELB עם stickiness.
הבעיות: משתמשים מאבדים את העגלה כשמחליפים שרת; תמונות מוצרים שהועלו דרך שרת אחד לא נראות מהשני;
בפיקים חלק מההזמנות "נעלמות" כי ה-DB נחנק; ובכל שחרור גרסה יש downtime.
היעד: זמינות גבוהה, אפס אובדן הזמנות, ועלות נשלטת.

**הארכיטקטורה החדשה:**

```text
                        Route 53 (Alias)
                              │
                    CloudFront (static + images)
                              │
                             ALB              ◄─ Public subnets, 3 AZs
                              │
            ┌─────────── Auto Scaling Group ───────────┐   ◄─ Private subnets
            │   Web Tier (stateless) — Golden AMI      │
            └───────┬──────────────┬───────────────┬───┘
                    │              │               │
            ElastiCache        SQS Queue      S3 (uploads)
            (sessions)             │
                                   ▼
                        ┌─ Worker ASG (Spot) ─┐
                        │  Order Processing    │──► DLQ
                        └──────────┬───────────┘
                                   ▼
                          RDS Multi-AZ + Read Replica
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **session ל-ElastiCache**, ביטול stickiness | ה-web tier הופך **stateless** — החלפת instance שקופה למשתמש |
| **תמונות ל-S3** במקום דיסק מקומי | נגיש מכל instance ומכל AZ; זול בהרבה מ-EFS; מתחבר ל-CloudFront |
| **CloudFront לפני ה-ALB** | תוכן סטטי נשלח מה-edge — הצי לא גדל בכל פיק צפייה |
| **ASG על פני 3 AZs** | כשל AZ שלמה לא מפיל את האתר; scale-out אוטומטי במקום שדרוג ידני |
| **Golden AMI** | instance חדש עולה בשניות — בלי זה ה-ASG מגיב מאוחר מדי |
| **SQS בין ה-web ל-order processing** | ה-web מחזיר תשובה מיד; **אף הזמנה לא אובדת** גם כשה-DB עמוס |
| **Worker ASG לפי עומק התור** | מספר ה-workers עוקב אחרי ה-backlog האמיתי, לא אחרי תעבורת HTTP |
| **Workers על Spot** | הפרעה היא רק הודעה שחוזרת לתור — חיסכון של **עד ~90%** |
| **DLQ ל-worker queue** | הזמנה פגומה לא חוסמת את שאר התור |
| **RDS Multi-AZ + Read Replica** | failover אוטומטי לזמינות; רפליקה מורידה עומס קריאות |
| **RI/Savings Plans על ה-min** של ה-web ASG | הקיבולת שרצה 24/7 בהנחה של **עד ~72%** |
| **SG משורשרים** ALB→EC2→Cache/DB | אף שכבה לא נגישה חוץ משכבת האב שלה |

**למה לא פשוט להגדיל את שני השרתים?**
Vertical Scaling **לא פותר זמינות** — עדיין שני שרתים בלי החלפה אוטומטית,
עדיין downtime בכל שדרוג, ועדיין תקרה קשיחה.

**למה לא EFS לתמונות?**
היה עובד, אבל **יקר משמעותית מ-S3**, ולא מתחבר ישירות ל-CloudFront.
EFS נכון רק כשהאפליקציה **חייבת** ממשק filesystem ואי אפשר לגעת בקוד.

**למה לא לפרק מיד ל-microservices?**
כי זה מוסיף **overhead תפעולי, ניצול שרתים ירוד וריבוי גרסאות** — בלי לפתור אף אחת מהבעיות שנמנו.
הניתוק הראוי כאן הוא **הוצאת ה-state החוצה + תור לעיבוד ההזמנות**. Microservices הוא שלב מאוחר יותר,
כשיש כמה צוותים עם קצבי שינוי שונים.

**למה לא Elastic Beanstalk?**
היה בהחלט אפשרי — הוא היה בונה ALB + ASG וגם **Worker Tier שמתרחב לפי SQS**.
בוחרים בו כשהצוות רוצה להתמקד בקוד; בוחרים בבנייה ידנית כשצריך שליטה עדינה בכל רכיב.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **רשימת כל הפלטפורמות** של Elastic Beanstalk בעל פה — מספיק לדעת שיש Docker ושפות נפוצות.
- **מדיניות ה-deployment** של Beanstalk לעומק (rolling, immutable, blue/green) — זה יותר לתחום DevOps.
- **קונפיגורציה של Redis מול Memcached** ברמת הפרמטרים — ההבדל המושגי נלמד בשיעור הרלוונטי.
- **מבנה ה-cookie** ואלגוריתמי חתימה.
- **תמחור מדויק בדולרים** של ELB LCU או EFS.
- **פרטי הפנימיות של round-robin DNS**.
- **תבניות עיצוב של microservices** (saga, CQRS) — מעבר להיקף SAA-C03.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **סדר האבולוציה:** שרת בודד → vertical → horizontal → **ELB** → **ASG** → **Multi-AZ** → stateless → **SQS** → microservices.
- **Vertical Scaling = downtime + תקרה קשיחה**, ולא פותר זמינות.
- **DNS אינו Load Balancer.** TTL אצל הלקוח שולח תעבורה ל-instance מת → **ELB + Alias Record**.
- **ELB Health Checks** מוציאים instance לא תקין תוך שניות.
- **EC2 בשכבת האפליקציה שייכים ל-subnet פרטי**, וה-SG שלהם מקבל תעבורה **רק מה-SG של ה-ELB**.
- **ASG דורש stateless.** כל terminate מוחק כל state מקומי.
- **ה-min של ה-ASG הוא מועמד ל-RI/Savings Plans** (עד ~72%); הפיק ל-On-Demand/Spot.
- **Session state:** Stickiness (חלש) · Cookie **מתחת ל-4KB** וניתן לזיוף · **ElastiCache** · **DynamoDB עם TTL**.
- **ה-cookie מחזיק session_id בלבד**, לא נתונים. כל קלט מהלקוח דורש **validation**.
- **session = זמני ב-cache. user data = קבוע ב-RDS/DynamoDB.**
- **הרחבת קריאות:** Read Replicas (throughput, eventual consistency) או **ElastiCache** (latency).
- **EBS = AZ אחת, instance אחד. EFS = משותף בין AZs.** לתמונות — **S3 עדיף על שניהם**.
- **SQS מנתק שכבות** ומשמש **buffer לכתיבות ל-DB** — "transactions lost under load" ⇒ SQS.
- **תור לא מאיץ עיבוד.** הוא מפריד, סופג ומגן.
- **Golden AMI + User Data** = מה שהופך התרחבות אוטומטית למהירה באמת.
- **Elastic Beanstalk חינם**; משלמים רק על המשאבים. **Web Tier** מול **Worker Tier שמתרחב לפי SQS**.
- **Single Instance ל-dev · HA with LB ל-prod.**
- **CloudFront לפני האפליקציה** = offload של תוכן סטטי בלי שינוי בקוד ובארכיטקטורה.
- **Microservices:** סינכרוני = API Gateway/ELB · אסינכרוני = SQS/SNS/Kinesis/Lambda triggers.
- **אתגרי microservices:** overhead לכל שירות, ניצול שרתים ירוד, ריבוי גרסאות, מורכבות בצד הלקוח.

---

## 11. ✅ בדיקת הבנה

1. אתר על שני EC2 עם ELB stickiness. משתמשים מאבדים את העגלה מדי פעם. מה קורה ומה מתקנים?
2. WordPress בשתי AZs. תמונה שהועלתה נראית לפעמים ולפעמים לא. מה הסיבה ומה הפתרון?
3. למה round-robin DNS עם A records אינו תחליף ל-ELB?
4. "בעומס גבוה חלק מהעסקאות אובדות." מה מוסיפים לארכיטקטורה?
5. איפה עדיף לשמור session state ולמה — cookie, ElastiCache או DynamoDB?
6. מה ההבדל בין Web Server Tier ל-Worker Tier ב-Elastic Beanstalk?
7. אפליקציה מפיצה קובצי עדכון כבדים ומשלמת הון על EC2 ו-bandwidth. פתרון בלי לגעת בקוד?
8. למה ASG דורש שהאפליקציה תהיה stateless?
9. איזה חלק מצי ה-ASG כדאי לכסות ב-Reserved Instances ולמה?
10. הוספתם SQS והלקוחות מתלוננים שההזמנה "לא מאושרת מיד". האם התור נכשל?

<details>
<summary>תשובות</summary>

1. Stickiness מקשרת את המשתמש ל-instance ספציפי, ו**כשה-instance מת ה-session מת איתו**.
   הפתרון: להוציא את ה-session ל-**ElastiCache** (או DynamoDB), לשמור ב-cookie רק `session_id`,
   ולבטל את התלות ב-affinity.
2. התמונות נשמרות על **EBS** שמוצמד ל-instance אחד ב-AZ אחת, ולכן ה-instance בשנייה לא רואה אותן.
   הפתרון: **EFS** (מערכת קבצים משותפת בין AZs) — או, עדיף וזול יותר, **S3**.
3. אין **health checks** אמיתיים, ו-**TTL** גורם ללקוחות להחזיק כתובת של instance שכבר מת עד שעה.
   בנוסף, כל שינוי בצי דורש עדכון DNS ידני. **ELB + Alias Record** פותר את שלושתם.
4. **SQS כ-buffer** בין האפליקציה ל-DB: האפליקציה כותבת לתור, workers מושכים ומכניסים ל-DB
   בקצב שה-DB מסוגל לו. שום עסקה לא אובדת — היא רק ממתינה.
5. **ElastiCache** לאפליקציית web קלאסית — latency של תת-מילישנייה.
   **DynamoDB** בארכיטקטורה serverless — ללא ניהול ועם **TTL** לניקוי אוטומטי.
   **Cookie** רק לנתונים קטנים ולא רגישים: **מתחת ל-4KB**, מכביד על כל בקשה, ו**ניתן לזיוף**.
6. **Web Server Tier** מקבל בקשות HTTP מאחורי ALB.
   **Worker Tier** **מושך הודעות מ-SQS** ומעבד ברקע, ו**מתרחב לפי מספר ההודעות בתור**.
7. **CloudFront לפני האפליקציה.** קובצי העדכון סטטיים ולכן ניתנים ל-cache ב-edge;
   ה-ASG כמעט לא גדל, ה-bandwidth מהמקור צונח, והזמינות משתפרת — **בלי שינוי בקוד**.
8. כי ASG **מוסיף ומוחק instances בלי התראה**. כל state מקומי — session, קובץ שהועלה, cache —
   נמחק עם ה-instance. לכן ה-state חייב לשבת בשכבה חיצונית.
9. את ה-**minimum capacity** — ה-instances שרצים 24/7 בכל מקרה. הם ידועים מראש ולכן מתאימים
   ל-**RI/Savings Plans** בהנחה של עד ~72%. את הפיק המשתנה משאירים On-Demand או Spot.
10. **לא.** התור עשה בדיוק את מה שהוא אמור: הפך את הזרימה ל**אסינכרונית**.
    ה-latency מקצה לקצה עולה, וזה ה-trade-off המכוון. אם הלקוח חייב אישור מיידי —
    מחזירים "התקבל" מיד ומודיעים על ההשלמה בנפרד, למשל דרך SNS.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[28 - SQS and SNS]] · [[29 - Event-Driven Architecture]] · [[07 - Auto Scaling]] · [[08 - Elastic Load Balancing]] · [[33 - High Availability and Scalability]] · [[20 - EFS and File Storage]] · [[19 - EBS and EC2 Storage]] · [[22 - RDS Scaling and Availability]] · [[15 - CloudFront and Global Delivery]] · [[26 - Containers]] · [[27 - API Gateway]] · [[38 - Serverless and Modern Architectures]] · [[39 - Architecture Decision Making]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 4081–4698, 6902–6917, 7048–7078, 7422–7462, 9169–9267
