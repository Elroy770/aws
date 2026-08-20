---
lesson: 34
title: Disaster Recovery
domain: Design Resilient Architectures
services: [Route 53, RDS, Aurora Global Database, S3, EBS, CloudFormation, AWS Elastic Disaster Recovery, Storage Gateway, Snowball]
tags: [saa-c03, resilience, disaster-recovery, rto, rpo, multi-region]
---

# 34 — Disaster Recovery

> [!abstract] בשורה אחת
> DR הוא **מסחר בין כסף לזמן**: כל אחת מארבע האסטרטגיות קונה RTO ו-RPO נמוכים יותר במחיר גבוה יותר,
> והמבחן כמעט תמיד נותן לכם את שני המספרים ומצפה שתבחרו את **הזולה ביותר שעדיין עומדת בהם**.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | מה נחשב "אסון" ולמה Multi-AZ לא מספיק |
| 2 | איך זה עובד | **RPO מול RTO** עם ציר זמן, וארבע האסטרטגיות |
| 3 | פירוק מפורט | הטיפים המעשיים: Backup, HA, Replication, Automation, Chaos, ו-DRS |
| 4 | עלות | מה כל אסטרטגיה עולה בפועל, ואיפה מסתתרת ההוצאה |
| 5 | השוואות | **טבלת ארבע האסטרטגיות** — הטבלה שנשאלת כמעט תמיד |
| 6 | Well-Architected | DR לפי ששת ה-Pillars |
| 7 | מלכודות | "Multi-AZ הוא DR" ו-"snapshot cross-Region זה פתרון" |
| 8 | Scenario | בחירת אסטרטגיה לשלושה workloads עם דרישות שונות |

**מונחי מפתח בשיעור:** `RPO` · `RTO` · `Pilot Light` · `Warm Standby` · `Hot Site` · `Active-Active` · `Failback` · `Continuous Replication`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- **אסון** = כל אירוע שפוגע בהמשכיות העסקית או בכספי החברה. לא רק רעידת אדמה.
- כשל של **Region שלם** — נדיר, אבל קורה, ו-Multi-AZ לא מגן מפניו.
- **טעות אנוש רחבת היקף** — מישהו מחק טבלה, bucket או stack שלם.
- **Ransomware** — הנתונים קיימים, אבל מוצפנים ולא נגישים.
- DR מסורתי **on-premise ל-on-premise** דורש אתר שני מלא — **יקר להחריד**.

### מה השירות פותר

- **DR = היערכות והתאוששות** מאסון, לפי שני מספרים מוסכמים מראש.
- הענן מאפשר **לשלם רק על המוכנות שבאמת צריך** — לא על אתר שני שיושב ריק.
- שלושה תרחישים אפשריים:

| תרחיש | מה זה | הערה |
|---|---|---|
| **On-premise → On-premise** | DR מסורתי | **יקר מאוד** — אתר פיזי שני |
| **On-premise → AWS Cloud** | **Hybrid recovery** | הענן כאתר ה-DR; משלמים רק במימוש |
| **Region A → Region B** | DR בתוך AWS | הנפוץ בשאלות המבחן |

> [!tip] האנלוגיה
> **RPO הוא כמה זמן אחורה תצטרכו לחזור.** אם אתם שומרים מסמך כל 10 דקות,
> קריסה תעלה לכם עד 10 דקות של עבודה.
> **RTO הוא כמה זמן ייקח לכם לחזור לעבוד.** להדליק מחשב חלופי, לפתוח את הקובץ, ולהמשיך.
> שני מספרים שונים לחלוטין, ומשלמים על כל אחד מהם בנפרד.

---

## 2. ⚙️ איך זה עובד

### 2.1 RPO ו-RTO — ציר הזמן שחייבים לצייר בראש

```text
                          ה-💥 אסון
                              │
◄──── RPO ─────┤              │              ├──── RTO ────►
               │              │              │
      ┌────────┴──────────────┼──────────────┴────────┐
      │                       │                       │
   הגיבוי               רגע הכשל              המערכת חזרה
   האחרון                                       לפעולה
   שהצליח
      │◄───── Data Loss ─────►│◄──── Downtime ───────►│
      │   כמה נתונים איבדנו   │   כמה זמן היינו למטה  │
```

| מדד | לאיזה כיוון | מודד | מה מקטין אותו |
|---|---|---|---|
| **RPO** — Recovery **Point** Objective | **אחורה** מהאסון | **כמה נתונים איבדנו** | **תדירות** הגיבוי / הרפליקציה |
| **RTO** — Recovery **Time** Objective | **קדימה** מהאסון | **כמה זמן היינו מושבתים** | **מוכנות** התשתית באתר המשני |

**דוגמה מספרית:**

- גיבוי כל 6 שעות → **RPO של עד 6 שעות**. אסון בשעה 14:00 והגיבוי האחרון ב-12:00 = איבדנו שעתיים.
- שחזור לוקח 4 שעות → **RTO של 4 שעות**. המערכת חזרה ב-18:00.

> [!info] הזיכרון הפשוט
> **RPO = Point in time** → נקודה **בעבר**. **RTO = Time to recover** → זמן **קדימה**.
> RPO נמוך יותר = לגבות **יותר תכופות** (או להעביר לרפליקציה).
> RTO נמוך יותר = להחזיק **יותר תשתית דולקת** מראש.

### 2.2 ארבע האסטרטגיות — סקירה

הן מסודרות בסולם אחד: **מהזול והאיטי לימיני והמהיר.**

```text
עלות ↑
  │                                              ● Multi Site / Hot Site
  │                                              (Active-Active, RTO שניות)
  │                          ● Warm Standby
  │                          (מערכת מלאה בגודל מינימלי)
  │          ● Pilot Light
  │          (רק הליבה הקריטית דולקת)
  │  ● Backup & Restore
  │  (רק נתונים, שום דבר לא רץ)
  └────────────────────────────────────────────────────► RTO נמוך יותר
```

### 2.3 Backup and Restore

**הרעיון:** לא רץ כלום באתר המשני. רק הנתונים נשמרים, ומשחזרים הכול בעת אירוע.

```text
Corporate Data Center                    AWS Cloud
        │                                    │
   AWS Snowball ──(העברה חד-פעמית גדולה)──►  S3 ──lifecycle──► Glacier
   Storage Gateway ──(העברות שוטפות)──────►  S3
        │
   EC2 ──► AMI                    EBS ──► Snapshots מתוזמנים
   RDS ──► Automated Backups      Redshift ──► Snapshots
```

| מאפיין | ערך |
|---|---|
| **RPO** | **הגבוה ביותר** — כתדירות הגיבוי (שעות) |
| **RTO** | **הגבוה ביותר** — שעות עד ימים |
| מה רץ ב-Region המשני | **כלום.** רק אחסון |
| **עלות** | **הנמוכה ביותר** — רק אחסון והעתקה |
| מתאים ל | workloads לא קריטיים, ארכיונים, סביבות פנימיות |

- **AWS Snowball** — להעברה ראשונית של נפח עצום מ-on-premises.
- **AWS Storage Gateway** — גיבוי שוטף מ-on-premises ל-S3.
- **S3 Lifecycle Policy** — הזרמה אוטומטית ל-**S3 IA** ואז ל-**Glacier** לחיסכון.
- **Cross-Region Replication** ב-S3 — כדי שהגיבוי לא יישב באותו Region שנפל.

### 2.4 Pilot Light

**הרעיון:** **גרסה קטנה של האפליקציה רצה תמיד בענן** — רק ה**ליבה הקריטית**.
השם מגיע מלהבת הפיילוט של דוד גז: קטנה, דולקת תמיד, ומאפשרת הצתה מהירה.

```text
Corporate Data Center          Route 53          AWS Cloud (DR Region)
        │                         │
        └── Data Replication ─────┼──────────►  RDS  ✅ רץ
                                  │              │
                                  └──────────►  EC2  ❌ לא רץ (AMI מוכן)
```

| מאפיין | ערך |
|---|---|
| **RPO** | נמוך — הרפליקציה של הנתונים רציפה |
| **RTO** | **בינוני** — עשרות דקות. צריך להשיק את שכבת ה-compute |
| מה רץ ב-Region המשני | **רק ה-DB / הליבה.** ה-compute כבוי |
| **עלות** | **נמוכה-בינונית** — משלמים על אחסון ורפליקציה, כמעט לא על compute |
| מתאים ל | מערכות חשובות שיכולות לספוג עשרות דקות של downtime |

- **דומה מאוד ל-Backup and Restore**, אבל **מהיר יותר** כי המערכות הקריטיות **כבר למעלה**.
- הפער בין השניים הוא בדיוק זה: הנתונים כבר שם ומעודכנים, ולא צריך לשחזר אותם.

### 2.5 Warm Standby

**הרעיון:** **המערכת המלאה רצה — אבל בגודל מינימלי.** באסון מרחיבים אותה לעומס הפרודקשן.

```text
Corporate Data Center ──Reverse Proxy──► Route 53
   App Server                               │
   Primary DB                               ▼   AWS Cloud (DR Region)
        │                                  ELB
        └── Data Replication ──────────►    │
                                     EC2 Auto Scaling (minimum) ✅ רץ
                                     RDS Secondary ✅ רץ
                                          ↑ failover
```

| מאפיין | ערך |
|---|---|
| **RPO** | נמוך מאוד |
| **RTO** | **נמוך — דקות.** רק צריך scale-out ולהפנות תעבורה |
| מה רץ ב-Region המשני | **הכול — ELB, ASG, DB — בקיבולת מינימלית** |
| **עלות** | **גבוהה** — סביבה מלאה שמחויבת 24/7 |
| מתאים ל | שירותים קריטיים לעסק |

- ההבדל מ-Pilot Light: כאן **גם שכבת האפליקציה רצה**, ולא רק ה-DB.
- ההבדל מ-Hot Site: כאן היא **לא מקבלת תעבורה** ולא בקיבולת מלאה — היא **ממתינה**.

### 2.6 Multi Site / Hot Site (Active-Active)

**הרעיון:** **קיבולת פרודקשן מלאה רצה בשני האתרים בו-זמנית**, ושניהם מקבלים תעבורה.

```text
Corporate Data Center ◄──active──► Route 53 ◄──active──► AWS Cloud
   App Server (production)              │              ELB
   Primary DB                           │              EC2 Auto Scaling (production)
        │                               │              RDS Secondary ✅
        └────── Data Replication ───────┴────────────► ↑ failover
```

| מאפיין | ערך |
|---|---|
| **RPO** | **הנמוך ביותר** — כמעט אפס |
| **RTO** | **הנמוך ביותר — דקות או שניות** |
| מה רץ ב-Region המשני | **הכול, בקיבולת פרודקשן מלאה, ומקבל תעבורה** |
| **עלות** | **הגבוהה ביותר בפער** — כמעט כפול |
| מתאים ל | mission-critical, מערכות תשלומים, מסחר |

### 2.7 All AWS Multi Region — הגרסה הענן-טהורה

```text
Region A (active)                Route 53                Region B (active)
   ELB                              │                       ELB
   EC2 Auto Scaling (production)    │       EC2 Auto Scaling (production)
   Aurora Global (primary) ──── Data Replication ──── Aurora Global (secondary)
                                                          ↑ failover
```

- **Aurora Global Database** הוא הרכיב שהופך active-active בין Regions למעשי:
  רפליקציה חוצת-Regions בהשהיה של שנייה או פחות, ו-failover מהיר.
- **Route 53** מנתב בין ה-Regions ומבצע failover לפי health checks.
- ראו [[22 - RDS Scaling and Availability]] ו-[[14 - Route 53 and DNS]].

---

## 3. 🔍 פירוק מפורט — חמשת תחומי ה-DR המעשיים

### 3.1 Backup — מה מגבים ואיך

| מקור | מנגנון |
|---|---|
| **EBS** | Snapshots מתוזמנים |
| **RDS** | Automated Backups + Manual Snapshots |
| **Redshift** | Snapshots |
| **EC2** | **AMI** — כדי שאפשר יהיה לשחזר instance מוכן |
| **קבצים** | דחיפה שוטפת ל-**S3** → **S3 IA** → **Glacier** דרך **Lifecycle Policy** |
| **מ-on-premises** | **Snowball** להעברה חד-פעמית גדולה · **Storage Gateway** לשוטף |
| **חוצה Regions** | **S3 Cross-Region Replication**, העתקת snapshots ל-Region אחר |

פירוט מלא ב-[[35 - Backup and Data Protection]].

### 3.2 High Availability — מה שמצמצם את הצורך ב-DR מלכתחילה

- **Route 53** להעברת ה-DNS מ-Region ל-Region בזמן אירוע.
- **RDS Multi-AZ · ElastiCache Multi-AZ · EFS · S3** — עמידות בתוך Region.
- **Site-to-Site VPN כגיבוי ל-Direct Connect** — אם הקו הייעודי נופל, ה-VPN תופס את מקומו.

> [!warning] ההבחנה שנשאלת שוב ושוב
> **Multi-AZ אינו DR.** הוא HA **בתוך Region אחד**.
> אם ה-Region כולו נפל — Multi-AZ לא עוזר. DR הוא **חוצה Regions** (או ענן-על-פרמיסס).

### 3.3 Replication — לצמצם את ה-RPO

- **RDS Cross-Region Read Replica** — רפליקה קריאה ב-Region אחר, שניתן לקדם ל-primary.
- **Aurora Global Databases** — הפתרון החזק ביותר לרפליקציה חוצת-Regions.
- **רפליקציה מ-on-premises ל-RDS** — כדי שהענן יהיה מוכן לקלוט.
- **Storage Gateway** — סנכרון קבצים ונפחים בין ה-Data Center ל-AWS.

### 3.4 Automation — מה שמצמצם את ה-RTO

- **CloudFormation / Elastic Beanstalk** — לבנות מחדש **סביבה שלמה** בפקודה אחת.
  זה ההבדל בין RTO של ימים ל-RTO של שעה.
- **CloudWatch Alarms** שמפעילים **Recover / Reboot** על EC2 שנכשל.
- **Lambda functions** לאוטומציות מותאמות בתהליך ההתאוששות.

> [!tip] הנקודה שהכי מפספסים
> **snapshot לבדו אינו תוכנית DR.** צריך גם **התבניות (IaC), ה-AMIs, ה-IAM roles,
> ה-secrets, מפתחות ה-KMS וה-DNS** ב-Region המשני.
> בלעדיהם יש לכם נתונים — ואין לכם מערכת.

### 3.5 Chaos Engineering

- **Netflix** מפעילה "simian army" שמסיימת instances באקראי בפרודקשן.
- הרעיון: **תוכנית DR שלא תורגלה היא השערה, לא תוכנית.**
- תרגול failover מגלה את מה ששכחתם: מכסות, הרשאות, תלויות ב-Region המקורי.

### 3.6 AWS Elastic Disaster Recovery (DRS)

נקרא בעבר **CloudEndure Disaster Recovery**.

```text
Corporate Data Center / כל ענן אחר                AWS Cloud
  OS · Apps · DB · Disks                    Staging Area
        │                                    EC2 + EBS בעלות נמוכה
   AWS Replication Agent ──רפליקציה רציפה──►      │
                          (השהיה של שניות)        │ failover (דקות)
                                                  ▼
        ◄────────── failback ──────────  Production Target EC2 + EBS
```

| מאפיין | פירוט |
|---|---|
| מה זה עושה | משחזר שרתים **פיזיים, וירטואליים או ענניים** לתוך AWS |
| **מנגנון** | **רפליקציה רציפה ברמת ה-block** דרך Replication Agent |
| **RPO** | **שניות** |
| **RTO** | **דקות** |
| Staging | ה-EC2 וה-EBS באזור ההיערכות הם **בעלות נמוכה** עד לאירוע |
| **Failback** | נתמך — חזרה לאתר המקורי אחרי שהאסון נפתר |
| Use cases | מסדי נתונים קריטיים (Oracle, MySQL, SQL Server), אפליקציות ארגוניות (SAP), **הגנה מ-ransomware** |

> [!info] מתי DRS הוא התשובה
> כשהשאלה מתארת **שרתים קיימים ב-on-premises או בענן אחר** שצריך להגן עליהם
> עם **RPO של שניות** ובלי לשנות את האפליקציה.
> זה למעשה Pilot Light אוטומטי ומנוהל: משלמים על staging זול, ומשלמים על הקיבולת המלאה רק ב-failover.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים לפי אסטרטגיה

| אסטרטגיה | רכיבי החיוב | הערה |
|---|---|---|
| **Backup & Restore** | **אחסון בלבד** (S3/Glacier/snapshots) + העתקות | הזול. משלמים ב**זמן** בעת השחזור |
| **Pilot Light** | אחסון + **רפליקציית נתונים** + מעט compute | ה-DB רץ; ה-compute כבוי |
| **Warm Standby** | **סביבה מלאה בקיבולת מינימלית** 24/7 + ELB + DB | קפיצת עלות משמעותית |
| **Hot Site / Active-Active** | **כמעט כפול compute** + רפליקציה + ניטור + cross-Region transfer | הגבוה ביותר |
| **DRS** | staging זול (EC2/EBS מוקטנים) + רפליקציה | קיבולת מלאה מחויבת **רק ב-failover** |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **Backup & Restore + Glacier** | **הזול ביותר** | RTO של שעות מקובל |
| **Pilot Light** | נמוכה-בינונית | RTO של עשרות דקות מקובל |
| **AWS DRS** | נמוכה יחסית לביצועים שהוא נותן | שרתים קיימים, RPO של שניות |
| **Warm Standby** | גבוהה | RTO של דקות נדרש |
| **Active-Active** | **הגבוהה ביותר** | RTO של שניות, mission-critical |
| **Savings Plans על ה-compute הקבוע** | מוריד את החלק היציב **עד ~72%** | Warm Standby ו-Active-Active |

### 🚩 עלויות נסתרות

- **Cross-Region Data Transfer** — הרפליקציה רצה 24/7 ומחויבת לכל GB. ב-active-active זה פריט מרכזי.
- **Snapshots שמצטברים** — בלי lifecycle policy הם גדלים לנצח.
- **קיבולת שלא נבדקה** — Warm Standby שלא תורגל עלול פשוט לא לעמוד בעומס בעת האמת.
  זו לא עלות בחשבונית, אבל זו העלות היקרה מכולן.
- **Savings Plans לא מכסים data transfer** — רק compute.
- **מפתחות KMS ו-secrets ב-Region המשני** — לרוב נשכחים, ואז השחזור נכשל.
- **DR account נפרד** — נכון מבחינת אבטחה, אבל מוסיף ניהול והעברות בין חשבונות.
- **Glacier retrieval** — משיכה מ-Glacier לוקחת זמן **ועולה כסף**, וזה משפיע ישירות על ה-RTO.

### 💡 טיפים לחיסכון

- **לבחור אסטרטגיה לפי ה-business impact האמיתי** — לא לפי מה שנשמע בטוח.
- **להשהות compute ב-Pilot Light** — לשלם על אחסון ורפליקציה, לא על שרתים.
- **Lifecycle Policy** ל-S3 IA ו-Glacier לגיבויים ישנים.
- **Savings Plans** על הקיבולת הקבועה של Warm Standby.
- **CloudFormation במקום קיבולת דולקת** — אוטומציה קונה RTO בזול יותר מברזל.
- **לדרג לפי workload** — לא כל המערכות זכאיות לאותה רמת DR.
- **למחוק snapshots ישנים** לפי מדיניות שמירה מוגדרת.

---

## 5. ⚖️ השוואות מכריעות

### 🔥 טבלת ארבע האסטרטגיות — הטבלה שנשאלת כמעט תמיד

| קריטריון | **Backup & Restore** | **Pilot Light** | **Warm Standby** | **Multi Site / Hot Site** |
|---|---|---|---|---|
| **RTO** | **שעות–ימים** | **עשרות דקות** | **דקות** | **דקות עד שניות** |
| **RPO** | **שעות** (כתדירות הגיבוי) | דקות (רפליקציה) | דקות–שניות | **כמעט אפס** |
| **עלות יחסית** | **הנמוכה ביותר** | נמוכה-בינונית | גבוהה | **הגבוהה ביותר** |
| **מה רץ ב-Region המשני** | **כלום** — רק אחסון | **רק ה-DB / הליבה**; compute כבוי | **הכול, בקיבולת מינימלית** | **הכול, בקיבולת פרודקשן מלאה** |
| מקבל תעבורה בשגרה | ❌ | ❌ | ❌ | ✅ **active-active** |
| מה עושים באסון | לשחזר הכול מאפס | **להשיק את ה-compute** ולהפנות DNS | **scale-out** ולהפנות DNS | **כלום** — כבר עובד |
| **Use case** | ארכיונים, dev/test, לא קריטי | מערכות חשובות עם סבילות לדקות | שירותים קריטיים לעסק | תשלומים, מסחר, mission-critical |

> [!tip] איך פותרים את השאלה בפועל
> 1. מוצאים את ה-**RTO** בשאלה.
> 2. פוסלים כל אסטרטגיה שלא עומדת בו.
> 3. מבין הנותרות — בוחרים את **הזולה ביותר**.
>
> "RTO of 4 hours" ⇒ **Backup & Restore**. "RTO of 10 minutes" ⇒ **Warm Standby**.
> "near-zero downtime" ⇒ **Multi Site**. "cheapest possible DR" ⇒ תמיד **Backup & Restore**.

### HA מול DR — ההבחנה המושגית

| קריטריון | **High Availability** | **Disaster Recovery** |
|---|---|---|
| מפני מה מגן | כשל **instance או AZ** | כשל **Region**, טעות אנוש רחבה, ransomware |
| היקף | **בתוך Region אחד** | **בין Regions** (או on-prem ↔ ענן) |
| מנגנונים | Multi-AZ, ELB, ASG | רפליקציה חוצת-Regions, backups, failover DNS |
| התערבות אנושית | לרוב אוטומטי לחלוטין | לרוב מעורב החלטה ותהליך |
| ראו | [[33 - High Availability and Scalability]] | השיעור הזה |

### Warm Standby מול Hot Site — ההבדל שקל לפספס

| קריטריון | Warm Standby | Hot Site / Active-Active |
|---|---|---|
| הסביבה רצה | ✅ **בקיבולת מינימלית** | ✅ **בקיבולת פרודקשן מלאה** |
| מקבלת תעבורה בשגרה | ❌ ממתינה | ✅ **משרתת משתמשים** |
| פעולה נדרשת באסון | **scale-out + DNS** | אין — או רק הסטת משקל ב-DNS |
| עלות | גבוהה | **הגבוהה ביותר** |

> [!info] שורה תחתונה
> **DR נמדד בשני מספרים, לא בתחושה.** קודם קובעים RTO ו-RPO עם העסק,
> ורק אז בוחרים את **האסטרטגיה הזולה ביותר** שעומדת בהם.
> אסטרטגיה יקרה מהנדרש היא בזבוז; זולה מהנדרש היא סיכון עסקי.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-DR** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | התוכנית כתובה, אוטומטית ומתורגלת | RTO/RPO **מתועדים לכל workload**; **CloudFormation** לשחזור סביבה; **תרגילי failover** קבועים; post-incident review |
| **Security** | הגיבוי עצמו לא ייפגע מאותו אירוע | **חשבון DR מבודד**; גיבויים **immutable** (Vault Lock / Object Lock) נגד ransomware; **מפתחות KMS משוכפלים** ל-Region המשני |
| **Reliability** | ה-Region המשני באמת מסוגל לקלוט | **Cross-Region replication** (Aurora Global, CRR); **Route 53 health checks** ל-failover; מיפוי כל התלויות |
| **Performance Efficiency** | ההתאוששות לא נכשלת מחוסר קיבולת | **AMIs מוכנים מראש**; warm capacity ל-RTO קצר; **load test על סביבת ה-DR** ולא רק על הפרודקשן |
| **Cost Optimization** | לא משלמים על מוכנות מיותרת | לבחור אסטרטגיה לפי business impact; **להשהות compute ב-Pilot Light**; lifecycle ל-Glacier; **Savings Plans** על קיבולת קבועה |
| **Sustainability** | לא מחזיקים חוות שרתים שנייה בטל | להעדיף **Backup/Pilot Light** כשה-RTO מאפשר; **DRS עם staging זול** במקום סביבה כפולה דולקת |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "cheapest DR solution" | **Backup and Restore** |
| "RTO of several hours is acceptable" | **Backup and Restore** |
| "core database always running, app servers off" | **Pilot Light** |
| "scaled-down but fully functional environment" | **Warm Standby** |
| "near-zero downtime" / "seconds" | **Multi Site / Hot Site (Active-Active)** |
| "how much data can we afford to lose" | **RPO** |
| "how long can we be down" | **RTO** |
| "replicate on-premises servers continuously into AWS" | **AWS Elastic Disaster Recovery (DRS)** |
| "protect servers from ransomware, RPO in seconds" | **DRS** + גיבויים immutable |
| "failover DNS between Regions" | **Route 53 health checks** |
| "active-active across Regions with a relational DB" | **Aurora Global Database** |
| "recreate the whole environment automatically" | **CloudFormation** |
| "large one-time data transfer from the data center" | **AWS Snowball** |
| "ongoing on-premises backup into S3" | **AWS Storage Gateway** |
| "backup from Direct Connect failure" | **Site-to-Site VPN** כגיבוי |

### טעויות נפוצות

> [!warning] מלכודת 1 — "Multi-AZ הוא פתרון DR"
> **הניסוח:** "Our RDS is Multi-AZ, so we are protected against a Region failure."
> **הטעות:** לבלבל בין HA ל-DR.
> **הנכון:** **Multi-AZ הוא HA בתוך Region אחד.** נפילת Region מפילה גם את ה-standby.
> ל-DR צריך **Cross-Region Read Replica**, **Aurora Global Database** או העתקת snapshots ל-Region אחר.

> [!warning] מלכודת 2 — snapshot cross-Region כתוכנית שלמה
> **הניסוח:** "We copy EBS snapshots to another Region, so we have a DR plan."
> **הטעות:** להתייחס לנקודת שחזור כאל פתרון failover.
> **הנכון:** snapshot נותן **נקודת שחזור** בלבד.
> בלי **CloudFormation templates, AMIs, IAM roles, secrets, מפתחות KMS ו-DNS** ב-Region המשני —
> יש לכם דיסק, ואין לכם מערכת.

> [!warning] מלכודת 3 — Warm Standby = Active-Active
> **הניסוח:** "Warm standby means both Regions serve production traffic."
> **הטעות:** לערבב בין השניים.
> **הנכון:** **Warm Standby רץ בקיבולת מינימלית ולא מקבל תעבורה.** באסון עושים **scale-out** ומפנים DNS.
> **Active-Active** רץ בקיבולת מלאה **ומשרת משתמשים כל הזמן**.

> [!warning] מלכודת 4 — RPO של אפס תמיד אפשרי
> **הניסוח:** "Set RPO to zero across Regions."
> **הטעות:** להניח שרפליקציה סינכרונית חוצת-Regions היא ברירת מחדל.
> **הנכון:** **RPO אפס דורש רפליקציה סינכרונית**, ובין Regions היא מוסיפה latency לכל כתיבה
> ולעיתים בלתי מעשית. חוצה-Regions לרוב **אסינכרונית** ⇒ RPO קטן, אבל **לא אפס**.

> [!warning] מלכודת 5 — לבחור את האסטרטגיה היקרה "ליתר ביטחון"
> **הניסוח:** "RTO is 8 hours. Which solution?" והתשובות כוללות Warm Standby.
> **הטעות:** לבחור את הפתרון החזק ביותר.
> **הנכון:** המבחן מחפש את **הזולה ביותר שעומדת בדרישה** — כאן **Backup and Restore**.
> Warm Standby תעמוד בדרישה אבל תעלה פי כמה בלי סיבה.

> [!warning] מלכודת 6 — תוכנית DR שלא תורגלה
> **הניסוח:** "The DR plan is documented and the backups run nightly."
> **הטעות:** להניח שזה מספיק.
> **הנכון:** **תוכנית שלא תורגלה היא השערה.** תרגילי failover הם מה שחושף מכסות חסרות,
> הרשאות שלא הועתקו, ותלויות נסתרות ב-Region המקורי. זו דרישת Reliability מפורשת ב-Well-Architected.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברת פינטק עם שלושה workloads שונים ב-`us-east-1`:

| Workload | דרישה עסקית |
|---|---|
| **A — מנוע התשלומים** | **RTO של דקות, RPO כמעט אפס.** כל דקת השבתה = הפסד ישיר |
| **B — פורטל לקוחות** | **RTO של 30 דקות, RPO של 5 דקות.** לא נעים אבל לא קטסטרופה |
| **C — מחסן נתונים לדוחות** | **RTO של 24 שעות, RPO של 24 שעות.** דוחות יומיים בלבד |

בנוסף: שרתי Oracle ותיקים ב-Data Center שצריך להגן עליהם מפני ransomware.

**הפתרון:**

```text
A — מנוע התשלומים  ►  Multi Site / Active-Active
    us-east-1 (ELB + ASG production)  ◄── Route 53 ──►  eu-west-1 (ELB + ASG production)
    Aurora Global Database (primary)  ──replication──►  Aurora Global (secondary)

B — פורטל הלקוחות  ►  Warm Standby
    us-east-1 (production)            ◄── Route 53 health check failover
    eu-west-1: ELB + ASG(min=1) + RDS Cross-Region Read Replica  ✅ רץ מוקטן

C — מחסן הנתונים   ►  Backup and Restore
    Snapshots יומיים → S3 → Cross-Region Replication → Glacier (lifecycle)
    CloudFormation template מוכן לשחזור הסביבה

Oracle ב-Data Center  ►  AWS Elastic Disaster Recovery (DRS)
    Replication Agent ──רפליקציה רציפה──► Staging (EC2/EBS זולים) ──failover──► Production EC2
```

**ההנמקה:**

| החלטה | למה |
|---|---|
| **Active-Active ל-A** | RTO של דקות ו-RPO כמעט אפס **מחייבים** שהאתר השני כבר יעבוד |
| **Aurora Global Database** ל-A | הדרך המעשית לרפליקציה חוצת-Regions בהשהיה של שנייה עם failover מהיר |
| **Warm Standby ל-B** | RTO של 30 דקות מאפשר scale-out; **Active-Active היה מבזבז** פי כמה |
| **ASG עם min=1** ב-B | הסביבה קיימת ומוכחת; באסון מגדילים ומפנים DNS |
| **Cross-Region Read Replica** ל-B | RPO של 5 דקות מושג ברפליקציה אסינכרונית — בלי מחיר ה-latency של סינכרוני |
| **Backup & Restore ל-C** | RTO של 24 שעות = אין שום סיבה להחזיק תשתית דולקת |
| **Lifecycle ל-Glacier** ב-C | הגיבויים הישנים יורדים לשכבה הזולה ביותר אוטומטית |
| **CloudFormation ל-C** | האוטומציה היא מה שמבטיח שה-24 שעות באמת יספיקו |
| **DRS ל-Oracle** | רפליקציה רציפה ברמת block, **RPO שניות ו-RTO דקות**, בלי לגעת באפליקציה |
| **Object Lock / Vault Lock** על הגיבויים | הגנה מ-ransomware: גיבוי שלא ניתן למחוק או להצפין |
| **Route 53 health checks** בכל השלושה | הפניית התעבורה היא חלק מהתוכנית, לא מחשבה שנייה |

**למה לא Active-Active לכל השלושה?**
כי זה כמעט מכפיל את חשבון ה-compute והרפליקציה — עבור workloads שה-RTO שלהם
מאפשר פתרונות זולים בהרבה. **בוחרים את הזולה ביותר שעומדת בדרישה.**

**למה לא Backup & Restore למנוע התשלומים?**
כי RTO של שעות מול דרישה של דקות אינו פער טכני — הוא **כשל עסקי**.

**מה עוד חייב להיות בתוכנית?**

- **מפתחות KMS ו-secrets** משוכפלים לכל Region משני — אחרת השחזור נעצר בשלב הפענוח.
- **AMIs והתבניות** זמינים בשני ה-Regions.
- **תרגיל failover רבעוני** שמאמת שה-Warm Standby באמת מסוגל לספוג את עומס הפרודקשן.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **תהליכי ההתקנה** של AWS Replication Agent ב-DRS.
- **quotas מספריים** של מספר snapshots או רפליקות — soft limits משתנים.
- **מוצרי DR של צד שלישי** ופרטי אינטגרציה איתם.
- **פרטי CloudEndure ההיסטוריים** — מספיק לדעת שהשם השתנה ל-**Elastic Disaster Recovery**.
- **תמחור מדויק בדולרים** של cross-Region transfer.
- **המימוש הפנימי** של רפליקציית Aurora Global.
- **אלגוריתמי chaos engineering** — מספיק להבין את הרעיון שמתרגלים כשלים.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **RPO = כמה נתונים איבדנו** (מסתכל **אחורה** מהאסון). מוקטן על ידי **תדירות גיבוי/רפליקציה**.
- **RTO = כמה זמן היינו למטה** (מסתכל **קדימה**). מוקטן על ידי **תשתית שכבר דולקת**.
- **ארבע האסטרטגיות, מהזולה ליקרה:** Backup & Restore → **Pilot Light** → **Warm Standby** → **Multi Site**.
- **Backup & Restore:** כלום לא רץ · RTO שעות–ימים · **הכי זול**.
- **Pilot Light:** **רק ה-DB/הליבה רצים**, ה-compute כבוי · RTO עשרות דקות.
- **Warm Standby:** **המערכת המלאה רצה בקיבולת מינימלית** ולא מקבלת תעבורה · RTO דקות.
- **Multi Site / Hot Site:** **קיבולת פרודקשן מלאה, active-active** · RTO שניות · **הכי יקר**.
- **הכלל לפתרון שאלה:** לפסול לפי ה-RTO, ואז לבחור את **הזולה ביותר** שנשארה.
- **Multi-AZ הוא HA, לא DR.** DR הוא **חוצה Regions**.
- **snapshot לבדו אינו DR** — צריך גם IaC, AMIs, IAM, secrets, KMS ו-DNS.
- **Aurora Global Database** הוא הבסיס ל-active-active חוצה-Regions.
- **Route 53 health checks** הם מנגנון ה-failover בין Regions.
- **CloudFormation / Beanstalk** = לשחזר סביבה שלמה אוטומטית — **מקצר RTO בזול**.
- **Snowball** להעברה חד-פעמית גדולה · **Storage Gateway** לגיבוי שוטף מ-on-premises.
- **S3 Lifecycle** → S3 IA → **Glacier**, ו-**Cross-Region Replication** כדי שהגיבוי לא ייפול עם ה-Region.
- **Site-to-Site VPN** הוא הגיבוי ל-**Direct Connect**.
- **AWS Elastic Disaster Recovery (DRS)** — לשעבר CloudEndure. **רפליקציה רציפה ברמת block**,
  **RPO שניות · RTO דקות**, staging זול, תומך **failback**, מגן מ-**ransomware**.
- **תוכנית DR שלא תורגלה אינה תוכנית.** תרגילי failover הם חלק מהדרישה.

---

## 11. ✅ בדיקת הבנה

1. מה ההבדל בין RPO ל-RTO, ולאיזה כיוון בציר הזמן כל אחד מסתכל?
2. גיבוי כל 12 שעות, שחזור לוקח 3 שעות. מה ה-RPO ומה ה-RTO?
3. RDS Multi-AZ. האם החברה מוגנת מנפילת Region?
4. "RTO של 20 דקות, ותקציב מוגבל." איזו אסטרטגיה?
5. מה ההבדל המדויק בין Pilot Light ל-Warm Standby?
6. מה ההבדל בין Warm Standby ל-Multi Site?
7. חברה מעתיקה EBS snapshots ל-Region אחר וקוראת לזה תוכנית DR. מה חסר?
8. צריך להגן על שרתי SQL Server ב-Data Center עם RPO של שניות. מה בוחרים?
9. למה RPO של אפס בין Regions לרוב לא מעשי?
10. שני פתרונות עומדים ב-RTO הנדרש. איך בוחרים?

<details>
<summary>תשובות</summary>

1. **RPO** מסתכל **אחורה** מרגע האסון ומודד **כמה נתונים אבדו** — נקבע לפי תדירות הגיבוי/הרפליקציה.
   **RTO** מסתכל **קדימה** ומודד **כמה זמן המערכת הייתה מושבתת** — נקבע לפי מידת המוכנות של האתר המשני.
2. **RPO = עד 12 שעות** (במקרה הגרוע האסון קורה רגע לפני הגיבוי הבא).
   **RTO = 3 שעות** (הזמן עד שהמערכת חזרה).
3. **לא.** Multi-AZ הוא **HA בתוך Region אחד** — גם ה-standby נמצא באותו Region.
   ל-DR צריך **Cross-Region Read Replica**, **Aurora Global Database**, או העתקת snapshots ל-Region אחר.
4. **Pilot Light** — ה-DB רץ ומשוכפל, שכבת ה-compute כבויה ומושקת בעת אירוע.
   נותן RTO של עשרות דקות בעלות נמוכה בהרבה מ-Warm Standby.
5. **Pilot Light:** רק **הליבה הקריטית** (בעיקר ה-DB) רצה; שכבת האפליקציה **כבויה**.
   **Warm Standby:** **המערכת המלאה** — ELB, ASG, DB — רצה, אבל **בקיבולת מינימלית**.
6. **Warm Standby ממתין** בקיבולת מינימלית ולא מקבל תעבורה; באסון עושים **scale-out** ומפנים DNS.
   **Multi Site** רץ ב**קיבולת פרודקשן מלאה ומשרת משתמשים כל הזמן** (active-active).
7. חסרים **כל שאר מרכיבי המערכת** ב-Region המשני: **CloudFormation templates, AMIs, IAM roles,
   secrets, מפתחות KMS והגדרות DNS**. Snapshot נותן **נקודת שחזור**, לא failover של אפליקציה.
   חסר גם **תרגול** שמוכיח שהשחזור עומד ב-RTO.
8. **AWS Elastic Disaster Recovery (DRS)** — רפליקציה רציפה ברמת block עם **RPO של שניות
   ו-RTO של דקות**, בלי שינוי באפליקציה, עם staging זול ו-**failback**.
9. כי **RPO אפס דורש רפליקציה סינכרונית** — כל כתיבה מחכה לאישור מה-Region המרוחק.
   המרחק מוסיף latency לכל טרנזקציה ופוגע בביצועים. לכן רפליקציה חוצת-Regions היא לרוב
   **אסינכרונית**: RPO קטן מאוד, אך לא אפס.
10. בוחרים את **הזול יותר**. המבחן מחפש את הפתרון **הזול ביותר שעומד בדרישה** —
    אסטרטגיה חזקה מהנדרש היא בזבוז, ולכן תשובה שגויה.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[35 - Backup and Data Protection]] · [[33 - High Availability and Scalability]] · [[36 - Migration and Hybrid Cloud]] · [[22 - RDS Scaling and Availability]] · [[14 - Route 53 and DNS]] · [[17 - S3 Security and Data Management]] · [[37 - Cost Optimization]] · [[39 - Architecture Decision Making]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 14710–14924
