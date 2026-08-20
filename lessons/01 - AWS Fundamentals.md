---
lesson: 01
title: AWS Fundamentals
domain: Design Resilient Architectures
services: [AWS Global Infrastructure, Regions, Availability Zones, Edge Locations, CloudFront, IAM]
tags: [saa-c03, fundamentals, global-infrastructure, shared-responsibility]
---

# 01 — AWS Fundamentals

> [!abstract] בשורה אחת
> כל שאלה במבחן מתחילה בשאלה "איפה זה רץ ומי אחראי על מה" — Region, AZ, Edge ו-Shared Responsibility הם התשתית שעליה בנויים כל 40 השיעורים הבאים.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה בכלל עברו מ-data center משלך ל-Cloud |
| 2 | איך זה עובד | ההיררכיה Region → AZ → Data Center, ומה זה Edge |
| 3 | פירוק מפורט | scope של שירותים, 4 קריטריונים לבחירת Region, Shared Responsibility |
| 4 | עלות | על מה משלמים באמת — ולמה data transfer זה המלכוד |
| 5 | השוואות | Single-AZ מול Multi-AZ מול Multi-Region מול Edge |
| 6 | Pillars | איך התשתית הגלובלית נראית דרך ששת העמודים |
| 7 | מלכודות | מילות מפתח שמסגירות את התשובה |
| 8 | Scenario | אפליקציה ישראלית שמתרחבת לאירופה |
| 9–11 | סיכום ובדיקה | Cheat Sheet ושאלות |

**מונחי מפתח בשיעור:** `Region` · `Availability Zone` · `Edge Location` · `Region-scoped` · `Global Service` · `Shared Responsibility Model` · `Data Residency`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- כדי להרים שרת היית צריך לקנות חומרה, להמתין שבועות לאספקה ולשלם מראש.
- היית חייב לנחש כמה capacity תצטרך בעוד שנתיים — ולטעות לשני הכיוונים.
- ניחוש גבוה מדי = כסף שנשרף על ברזלים שיושבים ריקים.
- ניחוש נמוך מדי = האתר נופל ביום המכירות הגדול.
- כדי לשרת לקוחות בשלוש יבשות היית צריך לשכור שטח ב-data center בכל יבשת.
- כדי לשרוד שריפה ב-data center היית צריך לבנות אתר שני — ולתחזק אותו כל השנה.

### מה השירות פותר

- **On-demand:** מרימים משאב בדקות, מכבים אותו כשלא צריך, משלמים על מה שהשתמשת.
- **Elasticity:** מגדילים ומקטינים לפי ביקוש אמיתי במקום לפי תחזית.
- **Global reach:** אותה אפליקציה נפרסת ביפן, בפרנקפורט ובווירג'יניה מאותה קונסולה.
- **Resilience מובנית:** AWS כבר בנתה לך אתרים מבודדים (AZs) — אתה רק צריך לפרוס עליהם.
- **Undifferentiated heavy lifting:** אבטחה פיזית, חשמל, קירור ותחזוקת חומרה כבר לא הבעיה שלך.

### קצת היסטוריה (מסגרת מנטלית, לא חומר למבחן)

| שנה | אבן דרך |
|---|---|
| 2002 | AWS מושקת פנימית בתוך Amazon |
| 2003 | Amazon מזהה שהתשתית שלה היא חוזק ליבה שאפשר למכור |
| 2004 | השקה פומבית ראשונה — SQS |
| 2006 | השקה מחדש עם השילוש: SQS, S3, EC2 |
| 2007 | הרחבה לאירופה |

> [!tip] האנלוגיה
> Data center משלך = לקנות מכונית: הון עצמי גדול מראש, ביטוח, טיפולים, וחניה שאתה משלם עליה גם כשאתה בחו"ל.
> Cloud = מונית/השכרה: משלם לפי נסיעה, מחליף לרכב גדול יותר כשצריך, ולא מטפל בשמן.

---

## 2. ⚙️ איך זה עובד

### 2.1 היררכיית התשתית הגלובלית

```text
AWS Global Infrastructure
│
├── Region  (eu-central-1, us-east-1, il-central-1 ...)
│    │  אזור גאוגרפי = אשכול של data centers
│    │
│    ├── Availability Zone  (eu-central-1a)
│    │     └── data center אחד או יותר, חשמל/רשת/קירור עצמאיים
│    ├── Availability Zone  (eu-central-1b)
│    └── Availability Zone  (eu-central-1c)
│              ↑ מחוברים ביניהם ברשת high-bandwidth / ultra-low-latency
│
└── Edge Locations / Points of Presence  (400+ אתרים)
      └── רק caching ומסירה — לא מריצים שם EC2
```

### 2.2 Region — הגבול הגאוגרפי

- Region הוא **אשכול של data centers** באזור גאוגרפי מוגדר.
- שמות בפורמט `us-east-1`, `eu-west-3`, `ap-southeast-2`.
- **רוב שירותי AWS הם region-scoped** — משאב שיצרת ב-Region אחד פשוט לא קיים באחר.
- דאטה **לא עוזבת Region** בלי שאתה מאשר זאת במפורש (זה הבסיס ל-data residency).
- לכל Region יש קטלוג שירותים ומחירון משלו.

### 2.3 Availability Zone — יחידת הבידוד

- כל Region מכיל כמה AZs: בדרך כלל 3, מינימום 3, מקסימום 6.
- שם ה-AZ הוא שם ה-Region + אות: `ap-southeast-2a`, `ap-southeast-2b`, `ap-southeast-2c`.
- כל AZ הוא **data center אחד או יותר** עם חשמל, רשת וקישוריות מיותרים (redundant).
- ה-AZs מופרדים פיזית זה מזה — כך ששריפה, הצפה או הפסקת חשמל לא מפילות את כולם יחד.
- ביניהם: רשת רחבת פס עם latency נמוך במיוחד — לכן sync replication בין AZs היא אופציה מעשית.

> [!info] נקודה עדינה
> אות ה-AZ (`a`, `b`, `c`) **ממופה אחרת בכל חשבון AWS**.
> ה-`us-east-1a` שלך אינו בהכרח ה-`us-east-1a` של חבר שלך. מזהה יציב חוצה-חשבונות נקרא AZ ID (למשל `use1-az1`).

### 2.4 Edge Locations / Points of Presence

- למעלה מ-400 Points of Presence: 400+ Edge Locations ועוד 10+ Regional Edge Caches.
- פרוסים ב-90+ ערים ב-40+ מדינות.
- התפקיד: להביא תוכן פיזית קרוב למשתמש הסופי ולהוריד latency.
- משתמשים בהם: CloudFront, Global Accelerator, Route 53, WAF/Shield.
- **לא** מריצים באדג' EC2 שלך, לא שמים שם RDS, ולא מאחסנים שם דאטה קבועה.

### 2.5 מה קורה בבקשה אחת

```text
משתמש בתל אביב
   │  DNS
   ▼
Route 53 (Global) ──► מחזיר את ה-Edge הקרוב
   │
   ▼
Edge Location (TLV)
   │  cache hit?  ──► כן → מוחזר מיד, ה-origin לא נגע
   │  לא
   ▼
Region eu-central-1
   ├── AZ-a: ALB + EC2
   ├── AZ-b: ALB + EC2
   └── RDS Multi-AZ (primary ב-a, standby ב-b)
```

---

## 3. 🔍 פירוק מפורט

### 3.1 Scope של שירותים — הטבלה שחוזרת במבחן

| Scope | דוגמאות | משמעות מעשית |
|---|---|---|
| **Global** | IAM, Route 53, CloudFront, WAF | ישות אחת לכל החשבון; אין לבחור Region בקונסולה |
| **Region-scoped** | EC2, Lambda, S3 (bucket), VPC, DynamoDB, RDS | קיים רק ב-Region שבו יצרת אותו |
| **AZ-scoped** | EBS Volume, Subnet, EC2 Instance | קשור ל-AZ ספציפי; לא זז ל-AZ אחר סתם כך |

> [!warning] S3 מבלבל
> S3 **נראה** גלובלי כי שם ה-bucket חייב להיות ייחודי בעולם — אבל ה-bucket עצמו נוצר ב-Region ספציפי והדאטה יושבת שם.

### 3.2 ארבעת הקריטריונים לבחירת Region

| קריטריון | השאלה שאתה שואל | דגל אדום במבחן |
|---|---|---|
| **Compliance / Data governance** | האם החוק מחייב שהדאטה תישאר במדינה? | "GDPR", "data must remain in", "sovereignty" |
| **Proximity to customers** | איפה יושבים המשתמשים? | "reduce latency", "users in Asia" |
| **Available services** | האם השירות/הפיצ'ר בכלל קיים שם? | שירות חדש שעדיין לא הושק בכל Region |
| **Pricing** | כמה עולה אותו משאב שם? | "minimize cost", השוואת Regions |

- הסדר הזה הוא גם סדר העדיפויות: compliance גובר על latency, ו-latency גובר על מחיר.
- Compliance הוא הקריטריון היחיד שהוא **חסם קשיח** — אם הרגולציה אוסרת, אין דיון.

### 3.3 מודלי השירות — IaaS / PaaS / FaaS / SaaS

| מודל | דוגמה ב-AWS | אתה מנהל | AWS מנהלת |
|---|---|---|---|
| IaaS | EC2 | OS, patches, runtime, קוד, scaling | virtualization, חומרה, רשת |
| PaaS | Elastic Beanstalk | קוד + קונפיגורציה | פלטפורמה, deployment, OS |
| FaaS | Lambda | קוד בלבד | הכול מתחת לפונקציה |
| SaaS | Rekognition | קלט ופלט | המודל, התשתית, הכול |

- ככל שיורדים בטבלה — פחות שליטה, פחות עבודה, ופחות אחריות אבטחה עליך.

### 3.4 Shared Responsibility Model — הליבה של אבטחה ב-AWS

הכלל: **AWS אחראית ל-Security *of* the Cloud. אתה אחראי ל-Security *in* the Cloud.**

| שכבה | מי אחראי |
|---|---|
| אבטחה פיזית של data centers, שומרים, בקרת כניסה | AWS |
| חומרה, שרתים, דיסקים, השמדת דיסקים בסוף חיים | AWS |
| Hypervisor / virtualization layer | AWS |
| תשתית רשת גלובלית, backbone, DDoS ברמת התשתית | AWS |
| הפרדה בין Regions ובין AZs | AWS |
| Guest OS: patching, hardening, antivirus (ב-EC2) | הלקוח |
| Security Groups, NACLs, ארכיטקטורת VPC | הלקוח |
| IAM: users, roles, policies, MFA, rotation של keys | הלקוח |
| הצפנת דאטה (at rest / in transit) והחלטה מה להצפין | הלקוח |
| הדאטה עצמה, סיווגה, מי רואה אותה | הלקוח — **תמיד** |

### 3.5 איך הגבול זז בין שירותים

זו הטבלה שמפרידה תשובה נכונה מתשובה כמעט-נכונה:

| שירות | AWS אחראית ל... | אתה אחראי ל... |
|---|---|---|
| **EC2** | חומרה, hypervisor, רשת פיזית | OS patching, אנטי-וירוס, SG, ניהול מפתחות SSH, הצפנת EBS, backup |
| **RDS** | patching של מנוע ה-DB, OS, גיבויים אוטומטיים, Multi-AZ failover | הגדרות DB, סכימה, משתמשי DB, SG, בחירת encryption, בחירת retention |
| **S3** | עמידות של 11 תשיעיות, replication פנימי, תשתית | Bucket Policy, Block Public Access, versioning, encryption, מי ניגש |
| **Lambda** | OS, runtime, scaling, זמינות | קוד הפונקציה, IAM Role שלה, secrets, validation של קלט |
| **DynamoDB** | patching, replication, availability | סכמת מפתחות, IAM, encryption keys, capacity mode |

> [!tip] הכלל שמחזיק תמיד
> ככל שהשירות "מנוהל" יותר — AWS לוקחת יותר שכבות.
> אבל **הדאטה שלך, ה-IAM שלך, וההחלטה מי ניגש למה — לעולם לא עוברים ל-AWS.**

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| Compute | שעות/שניות של instance או משך ריצה | משתנה בין Regions |
| Storage | GB-חודש | EBS, S3, EFS — מודלים שונים |
| Requests | מספר קריאות API / GET / PUT | בולט ב-S3, Lambda, DynamoDB |
| Data transfer OUT לאינטרנט | GB יוצא | **הסעיף שהכי מפתיע בחשבון** |
| Data transfer בין Regions | GB | יקר יחסית; מחויב תמיד |
| Data transfer בין AZs | GB בשני הכיוונים | מחויב ברוב המקרים |
| Data transfer IN מהאינטרנט | — | בדרך כלל ללא עלות |

### מה זול ומה יקר

| תנועה | עלות יחסית | הערה |
|---|---|---|
| כניסה מהאינטרנט (ingress) | 0 | AWS רוצה שהדאטה שלך תיכנס |
| בתוך אותו AZ, דרך private IP | 0 או זניח | הכי זול שיש |
| בין AZs באותו Region | נמוכה אך לא אפסית | המחיר של Multi-AZ |
| בין Regions | גבוהה | המחיר של Multi-Region |
| החוצה לאינטרנט | הגבוהה ביותר | כאן נשרף הכסף |

- מחירי שירותים **משתנים בין Regions** — אותו instance יכול לעלות אחרת בפרנקפורט ובווירג'יניה.
- לא לשנן מספרים בדולרים: המבחן בודק **יחסים** והבנת המנגנון.

### 🚩 עלויות נסתרות

- **NAT Gateway:** מחויב גם על שעות וגם על כל GB שעובר בו — מלכודת קלאסית.
- **Cross-AZ traffic** בין שכבות האפליקציה: ALB ב-AZ-a שמדבר עם EC2 ב-AZ-b עולה כסף.
- **Cross-Region replication** ב-S3 או ב-RDS: אתה משלם גם על ה-transfer וגם על העותק השני.
- **Elastic IP לא משויך:** IP שיושב ריק מחויב, בניגוד לאינטואיציה.
- **Multi-Region DR:** מכפיל compute, storage, ניטור — וגם את עבודת התפעול.

### 💡 טיפים לחיסכון

- לבחור Region לפי צירוף של latency, compliance **ומחיר** — לא רק לפי הרגל.
- לשים CloudFront מול origin: cache hit חוסך גם egress מה-origin וגם compute.
- להשתמש ב-VPC Endpoints כדי ש-traffic ל-S3/DynamoDB לא יעבור דרך NAT Gateway.
- לצמצם קפיצות cross-AZ מיותרות בין micro-services.
- לא לקפוץ ל-Multi-Region לפני שהוכח ש-Multi-AZ לא מספיק ל-RTO/RPO הנדרש.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 רמות הפריסה

| קריטריון | Single-AZ | Multi-AZ | Multi-Region |
|---|---|---|---|
| מפני מה מגן | כלום מעבר לכשל instance | כשל data center / AZ שלם | כשל Region שלם, אסון אזורי |
| Latency בין רכיבים | הנמוך ביותר | נמוך מאוד | גבוה (מאות ms) |
| מורכבות | נמוכה | בינונית | גבוהה |
| עלות יחסית | הזולה | בינונית | היקרה ביותר |
| מתי בוחרים | dev/test, batch לא קריטי | ברירת המחדל ל-production | DR אזורי, compliance, קהל גלובלי |

### 5.2 Region נוסף מול Edge

| קריטריון | הוספת Region | CloudFront / Edge |
|---|---|---|
| מה משפר | latency + ריבונות דאטה + DR | latency של תוכן cacheable |
| מה רץ שם | האפליקציה המלאה | רק cache ו-logic קטן |
| דאטה דינמית | כן | חלקית (יש caching מוגבל) |
| מורכבות תפעול | גבוהה — replication, routing | נמוכה — origin אחד נשאר |
| עלות | הכפלת סטאק | תוספת requests/egress, לרוב זולה יותר |

> [!info] שורה תחתונה
> "לשרוד כשל AZ" ← Multi-AZ. "משתמשים גלובליים סובלים מ-latency על תוכן סטטי" ← CloudFront.
> "לשרוד אסון שמוחק Region שלם" או "החוק מחייב דאטה במדינה" ← Region נוסף.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בתשתית הגלובלית** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | פריסה ל-Regions/AZs חייבת להיות ניתנת לשחזור, לא ידנית | להגדיר את הפריסה כ-IaC כך שאותו stack יעלה בכל Region |
| Security | Region הוא גבול הדאטה; Shared Responsibility מגדיר מה עליך | לבחור Region לפי דרישות ריבונות, ולסגור SG/IAM כי זה בצד שלך |
| Reliability | AZ הוא יחידת הכשל שאתה מתכנן סביבה | לפרוס כל שכבה ב-2+ AZs, ולוודא שאין רכיב יחיד ב-AZ אחד |
| Performance Efficiency | המרחק הפיזי הוא רצפת ה-latency שלא תעקוף בקוד | לבחור Region קרוב לקהל, ולהוסיף Edge לתוכן cacheable |
| Cost Optimization | תנועת דאטה בין גבולות היא מרכיב עלות אמיתי | למדוד egress ו-cross-AZ, ולהחזיק Multi-Region רק כשמוצדק |
| Sustainability | cache hit = בקשה שלא הפעילה שרת בכלל | להעלות TTL, לעשות right-sizing, ולכבות סביבות dev בלילה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "survive the failure of a data center" | Multi-AZ |
| "survive the failure of an entire Region" | Multi-Region / DR |
| "data must not leave the country" | בחירת Region (compliance) |
| "global users experience high latency on static content" | CloudFront / Edge |
| "the service is not available in that Region" | קריטריון Available Services |
| "who is responsible for patching the guest OS" | הלקוח — ב-EC2 |
| "who is responsible for patching the database engine" | AWS — ב-RDS |
| "lowest cost while keeping data in Europe" | Region אירופאי הזול ביותר שעומד ב-compliance |

### טעויות נפוצות

> [!warning] מלכודת 1 — Edge כמקום הרצה
> **הניסוח:** "Deploy the application to Edge Locations to reduce latency."
> **הטעות:** לחשוב שאפשר להריץ EC2 או RDS ב-Edge Location.
> **הנכון:** Edge עושה caching ומסירה. הרצת compute מתבצעת ב-Region (או פונקציות קטנות ב-Edge דרך CloudFront).

> [!warning] מלכודת 2 — Multi-AZ כפתרון DR
> **הניסוח:** "The architecture is Multi-AZ, so it is disaster-recovery ready."
> **הטעות:** לזהות זמינות עם התאוששות מאסון.
> **הנכון:** Multi-AZ מגן מכשל AZ. אסון אזורי, מחיקה בטעות או תקלת אפליקציה דורשים גיבוי ותוכנית DR נפרדת.

> [!warning] מלכודת 3 — אותיות AZ בין חשבונות
> **הניסוח:** "Place both accounts' resources in us-east-1a for lowest latency."
> **הטעות:** להניח ש-`us-east-1a` הוא אותו מקום פיזי בשני חשבונות.
> **הנכון:** המיפוי של האות שונה בין חשבונות. להשוואה חוצת-חשבונות משתמשים ב-AZ ID.

> [!warning] מלכודת 4 — "S3 הוא גלובלי"
> **הניסוח:** "Store the data in S3; it is a global service so residency is not an issue."
> **הטעות:** לבלבל בין ייחודיות שם ה-bucket לבין מיקום הדאטה.
> **הנכון:** ה-bucket יושב ב-Region ספציפי. הדאטה שם, וכללי ה-residency חלים עליו.

> [!warning] מלכודת 5 — "AWS מאבטחת הכול"
> **הניסוח:** "AWS handles security, so the S3 bucket is safe by default."
> **הטעות:** להעביר ל-AWS את האחריות על הרשאות.
> **הנכון:** מי ניגש לדאטה שלך זו תמיד ההחלטה שלך — Bucket Policy, IAM ו-Block Public Access הם בצד הלקוח.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

- אתר מסחר ישראלי, רוב המשתמשים בישראל, ומתחילים למכור גם לגרמניה.
- הרגולציה מחייבת שנתוני לקוחות אירופאים יישמרו באיחוד האירופי.
- דרישה: לשרוד נפילה של data center שלם בלי downtime מורגש.
- תמונות המוצרים כבדות ומעמיסות על ה-origin.
- התקציב מוגבל — Multi-Region מלא לא מאושר בשלב זה.

```text
                    Route 53 (Global)
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
      CloudFront (Edge)        API traffic
       cache: images/CSS/JS           │
              │ cache miss            │
              ▼                       ▼
        ┌─────────── Region: eu-central-1 ───────────┐
        │                    ALB                     │
        │        ┌───────────┴───────────┐           │
        │   AZ-a: EC2 (ASG)         AZ-b: EC2 (ASG)  │
        │        └───────────┬───────────┘           │
        │        RDS Multi-AZ (primary a / standby b)│
        │        S3 (תמונות) ← origin ל-CloudFront   │
        └────────────────────────────────────────────┘
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Region יחיד באירופה (eu-central-1) | עונה על ה-compliance האירופאי, וה-latency לישראל סביר |
| פריסה על 2+ AZs בכל שכבה | עומד בדרישת "לשרוד נפילת data center" בלי Multi-Region |
| RDS Multi-AZ | failover אוטומטי ל-standby ב-AZ אחר, בלי לבנות replication ידנית |
| CloudFront מול S3 | מוריד latency לתמונות ומקטין egress ו-load מה-origin |
| Route 53 בקדמה | שירות גלובלי, לא תלוי ב-Region שנפל |
| S3 לתמונות ולא EBS | עמידות גבוהה ללא ניהול, ו-origin טבעי ל-CDN |

**למה לא Multi-Region?**

- הדרישה שהוגדרה היא שרידות של data center — לא של Region שלם.
- Multi-Region מכפיל עלות, מוסיף replication ו-routing, ומייצר עבודת תפעול מתמשכת.
- אם בעתיד ידרשו RTO/RPO אזוריים, זו שדרוגה נפרדת ומכוונת. ראה [[34 - Disaster Recovery]].

**למה לא רק להוסיף AZ שלישי במקום CloudFront?**

- AZ נוסף לא מקצר את המרחק הפיזי למשתמש — הוא מטפל בשרידות, לא ב-latency.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- לשנן את המספר המדויק של Regions או Edge Locations — הוא משתנה כל הזמן.
- תאריכים מדויקים בהיסטוריה של AWS או נתוני נתח שוק והכנסות.
- שמות של כל ה-Regions בעל פה. מספיק לזהות את הפורמט ואת המשמעות.
- מבנה פנימי של data center, ספקי חשמל, או פרטי ה-backbone הפיזי.
- AWS Outposts / Local Zones / Wavelength ברמת עומק — נלמדים בהקשר היברידי ב-[[36 - Migration and Hybrid Cloud]].

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- Region = אשכול data centers; רוב השירותים region-scoped; דאטה לא עוזבת Region בלי אישור.
- AZ = data center אחד או יותר, מבודד פיזית; בדרך כלל 3 לכל Region, מינימום 3, מקסימום 6.
- AZs מחוברים ברשת מהירה עם latency נמוך — מאפשר sync replication.
- אות ה-AZ ממופה שונה בכל חשבון; מזהה יציב = AZ ID.
- Edge / PoP: 400+ אתרים ב-90+ ערים; caching בלבד, לא compute כללי.
- שירותים גלובליים לזכור: IAM, Route 53, CloudFront, WAF.
- 4 קריטריונים ל-Region: Compliance → Latency → Available Services → Pricing.
- Multi-AZ = שרידות בתוך Region. Multi-Region = DR אזורי + קהל גלובלי.
- Shared Responsibility: AWS = security **of** the cloud; הלקוח = security **in** the cloud.
- EC2 → אתה מטפל ב-OS patching. RDS → AWS מטפלת בו. תמיד: הדאטה וה-IAM עליך.
- Data transfer IN בדרך כלל חינם; OUT לאינטרנט הכי יקר; cross-Region ו-cross-AZ מחויבים.
- S3 נראה גלובלי (שם ייחודי עולמית) אבל ה-bucket יושב ב-Region.

---

## 11. ✅ בדיקת הבנה

1. לקוח דורש לשרוד נפילה מלאה של data center, בלי להכפיל את התקציב. מה הפתרון המינימלי?
2. חברה אירופאית שואלת אם אפשר לאחסן ב-S3 בלי לדאוג ל-GDPR "כי S3 גלובלי". מה התשובה?
3. ב-EC2 התגלתה חולשה ב-kernel של ה-OS. מי אחראי להתקין את ה-patch? ומה אם זה RDS?
4. שני חשבונות רוצים למזער latency ביניהם. האם מספיק ששניהם יבחרו `us-east-1a`?
5. משתמשים באוסטרליה מתלוננים על טעינה איטית של תמונות מאתר שמתארח באירלנד. מה הפתרון הזול?
6. מה סדר העדיפויות בין compliance, latency ומחיר בבחירת Region?

<details>
<summary>תשובות</summary>

1. **Multi-AZ בתוך Region אחד** — ASG על 2+ AZs, ALB חוצה-AZ, ו-RDS Multi-AZ. Multi-Region מיותר כאן ויקר בהרבה.

2. **לא.** רק שם ה-bucket ייחודי עולמית; ה-bucket עצמו נוצר ב-Region. יש לבחור Region אירופאי, ודרישות ה-residency חלות במלואן.

3. **EC2 — הלקוח.** ה-guest OS הוא באחריותך (security in the cloud). **RDS — AWS**, כי היא מנהלת את ה-OS ואת מנוע ה-DB; אתה עדיין אחראי ל-SG, למשתמשי ה-DB ולהצפנה.

4. **לא.** מיפוי האות ל-AZ פיזי שונה בין חשבונות. יש להשוות לפי **AZ ID** (למשל `use1-az1`).

5. **CloudFront** מול ה-origin. תמונות הן תוכן cacheable מובהק, וה-Edge מקרב אותן פיזית למשתמש בלי לפרוס Region שני.

6. **Compliance → Proximity/Latency → Available Services → Pricing.** Compliance הוא חסם קשיח; מחיר הוא שיקול אחרון, כי אין טעם ב-Region זול שאסור להשתמש בו.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[02 - AWS Well-Architected Framework]] · [[03 - IAM Fundamentals]] · [[15 - CloudFront and Global Delivery]] · [[33 - High Availability and Scalability]] · [[34 - Disaster Recovery]] · [[37 - Cost Optimization]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 121–330, 16140–16181

> [!note] הערת דיוק
> מודל ה-Shared Responsibility אינו מפורט בשקפי הקורס, אך הוא חומר מבחן מרכזי ב-SAA-C03.
> הטבלאות בסעיף 3.4–3.5 מבוססות על המודל הרשמי של AWS. מספרי ה-Edge Locations הם נכון לזמן הכנת השקפים ומשתנים תדיר.
