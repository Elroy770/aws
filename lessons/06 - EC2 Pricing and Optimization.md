---
lesson: 06
title: EC2 Pricing and Optimization
domain: Design Cost-Optimized Architectures
services: [EC2, Savings Plans, Spot Fleet, Dedicated Hosts, Instance Scheduler, Auto Scaling]
tags: [saa-c03, compute, cost, pricing]
---

# 06 — EC2 Pricing and Optimization

> [!abstract] בשורה אחת
> אותו `m5.large` בדיוק יכול לעלות מחיר מלא או **עשירית** ממנו — ההבדל היחיד הוא **כמה גמישות ואיזו התחייבות** אתם מוכנים לתת, וזו כל השאלה במבחן.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה יש 7 דרכים לשלם על אותה מכונה |
| 2 | איך זה עובד | ציר הגמישות מול ההתחייבות; אנלוגיית המלון |
| 3 | פירוק מפורט | כל שבעת המודלים לעומק + Spot Requests ו-Spot Fleets |
| 4 | **עלות** — הסעיף המרכזי | **טבלת ההשוואה המלאה**, עלויות נסתרות, טיפים |
| 5 | השוואות | RI מול Savings Plan · Dedicated Host מול Dedicated Instance · Spot מול On-Demand |
| 6 | Well-Architected | אופטימיזציית עלות בלי לשבור reliability |
| 7 | מלכודות | "הכי זול" אינו תמיד Spot |
| 8 | Scenario | ארכיטקטורה מעורבת: baseline + burst + batch |

**מונחי מפתח בשיעור:** `On-Demand` · `Reserved Instance` · `Convertible RI` · `Savings Plan` · `Spot Price` · `Spot Fleet` · `Dedicated Host` · `Capacity Reservation` · `Instance Scheduler`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- compute הוא בדרך כלל השורה הגדולה ביותר בחשבונית AWS.
- הרצת הכול ב-On-Demand היא בזבוז ענק כשחלק מהעומס **ידוע וקבוע** מראש.
- מנגד — התחייבות ל-3 שנים על עומס שישתנה בעוד חצי שנה היא נעילה יקרה.
- יש workloads שיכולים לספוג הפסקה פתאומית, ויש כאלה שלא. אותו מחיר לשניהם זה לא הגיוני.
- רישוי תוכנה ישן (per-socket / per-core) מחייב לפעמים שליטה על השרת **הפיזי**.
- בשעת שיא (Black Friday) צריך **ודאות** שתהיה capacity — לא רק מחיר טוב.

### מה השירות פותר

AWS מציעה **7 מודלי רכישה**. כל אחד נותן משהו אחר בתמורה למשהו אחר:

| מודל | מה אתם נותנים | מה אתם מקבלים |
|---|---|---|
| **On-Demand** | כלום | גמישות מלאה, מחיר מלא |
| **Reserved Instance** | התחייבות 1/3 שנים לתצורה ספציפית | הנחה גדולה + הבטחת capacity (ב-Zonal) |
| **Convertible RI** | התחייבות 1/3 שנים, גמישה יותר | הנחה קצת קטנה יותר |
| **Savings Plan** | התחייבות ל-**סכום לשעה** | הנחה גדולה + גמישות גדולה |
| **Spot** | נכונות **לאבד את ה-instance** בכל רגע | ההנחה הגדולה ביותר שקיימת |
| **Dedicated Host** | תשלום על **שרת פיזי שלם** | שליטה בחומרה, BYOL, compliance |
| **Dedicated Instance** | תשלום עודף על בידוד חומרה | חומרה לא משותפת עם לקוחות אחרים |
| **Capacity Reservation** | תשלום מלא גם כשלא רצים | **ודאות** ש-capacity תהיה ב-AZ שבחרתם |

> [!tip] האנלוגיה — מלון נופש
> זו האנלוגיה שהקורס עצמו משתמש בה, והיא נצמדת לזיכרון:
>
> | מודל | במלון |
> |---|---|
> | **On-Demand** | מגיעים ועוזבים מתי שרוצים, משלמים מחיר מלא ללילה |
> | **Reserved Instance** | חוזה שנתי מראש — "אני בא כל שנה לחדר הזה" → הנחה יפה |
> | **Savings Plan** | מתחייבים ל-**סכום לשעה** ומקבלים כל סוג חדר — King, Suite, נוף לים |
> | **Spot** | המלון מוכר חדרים ריקים במכרז. המציע הגבוה זוכה — **ואפשר לזרוק אתכם החוצה** |
> | **Dedicated Host** | שוכרים **בניין שלם** של הנופש. אתם קובעים מי ישן איפה |
> | **Dedicated Instance** | קומה שכולה שלכם, אבל המלון מחליט באיזה חדר תשנו |
> | **Capacity Reservation** | מזמינים חדר לתקופה ומשלמים עליו במלואו — **גם אם לא הגעתם** |

---

## 2. ⚙️ איך זה עובד

### 2.1 שני הצירים שקובעים את המחיר

```text
             ההתחייבות שלכם  ────────────▶  ההנחה גדלה
             │
             │   On-Demand ──── Savings Plan ──── RI 3y All Upfront
             │   (0% הנחה)        (~72%)              (~72%)
             │
   הסבילות   │
   להפסקה    │   Spot  ← עד ~90% הנחה, אבל בלי שום הבטחה
             ▼
```

- **ציר ההתחייבות:** ככל שאתם מתחייבים ליותר זמן ומשלמים יותר מראש — ההנחה גדלה.
- **ציר הסבילות להפסקה:** אם אתם מוכנים שה-instance ייעלם ב-2 דקות התראה — Spot נותן את המקסימום.
- **ציר שלישי, הפוך:** אם אתם צריכים **ודאות capacity** — משלמים **תוספת** (Capacity Reservation, Dedicated).

### 2.2 העיקרון שמנצח כמעט תמיד — ארכיטקטורה מעורבת

התשובה הנכונה במבחן היא כמעט אף פעם לא "הכול במודל אחד".

```text
עומס לאורך היממה
   │                        ╭──╮
   │                    ╭───╯  ╰───╮        ← Spot / On-Demand (שיא זמני)
   │              ╭─────╯          ╰────╮
   ├──────────────────────────────────────  ← Savings Plan / RI (baseline קבוע)
   └────────────────────────────────────▶ זמן

  baseline קבוע  → RI / Savings Plan
  שיא צפוי       → On-Demand
  batch גמיש     → Spot
```

---

## 3. 🔍 פירוק מפורט — שבעת מודלי הרכישה

### 3.1 On-Demand

- משלמים על מה שרץ, **בלי התחייבות ובלי תשלום מראש**.
- **Linux ו-Windows: חיוב לפי שנייה**, אחרי הדקה הראשונה.
- **כל שאר מערכות ההפעלה: חיוב לפי שעה** (שעה חלקית = שעה מלאה).
- **המחיר הגבוה ביותר לשעה** מכל המודלים.
- מומלץ ל-workload קצר, בלתי צפוי, שאסור להפסיק אותו באמצע.

> [!note] נקודת דיוק
> ה-"pay by second" חל על Linux ו-Windows. מערכת הפעלה עם רישוי לפי שעה (למשל חלק
> מהפצות מסחריות) עדיין מחויבת לפי שעה. זו דקות שהמבחן מזכיר לעיתים.

### 3.2 Reserved Instances (RI)

- **עד ~72% הנחה** לעומת On-Demand.
- מזמינים **תצורה ספציפית**: Instance Type, Region, Tenancy, מערכת הפעלה.
- **תקופה:** שנה אחת (הנחה) או **3 שנים** (הנחה גדולה יותר).
- **אופן תשלום** — משפיע ישירות על ההנחה:

| אופן תשלום | תשלום מראש | ההנחה |
|---|---|---|
| **No Upfront** | 0 | הכי קטנה מבין השלוש |
| **Partial Upfront** | חלקי | בינונית |
| **All Upfront** | הכול מראש | **הגדולה ביותר** |

- **Scope — נקודה שנשאלת:**

| Scope | מה נותן |
|---|---|
| **Regional** | הנחה בכל ה-AZs ב-Region. **לא מבטיח capacity** |
| **Zonal** | הנחה **+ הבטחת capacity ב-AZ ספציפית** |

- **מומלץ ל-**usage יציב ורציף. הדוגמה הקלאסית: **DB שרץ 24/7**.
- ניתן **לקנות ולמכור RIs ב-Reserved Instance Marketplace** — מוצא מילוט אם העומס השתנה.

**Convertible Reserved Instance:**

- **עד ~66% הנחה** — קצת פחות מ-Standard RI.
- בתמורה: אפשר **לשנות** את instance type, instance family, מערכת הפעלה, scope ו-tenancy.
- מתאים לעומס ארוך טווח שהצורה שלו עוד עשויה להשתנות.

### 3.3 Savings Plans

- **עד ~72% הנחה** — **זהה ל-RI**.
- ההבדל המהותי: **לא מתחייבים ל-instance, מתחייבים ל-סכום לשעה.**
  לדוגמה: "אני מתחייב ל-$10 לשעה של compute למשך שנה או 3 שנים".
- **כל שימוש מעבר להתחייבות** מחויב במחיר **On-Demand רגיל**. לא מפסידים, פשוט לא מוזלים.
- **EC2 Instance Savings Plan** נעול ל-**משפחה + Region** ספציפיים (למשל `M5` ב-`us-east-1`).
- **בתוך הנעילה הזו — גמיש לחלוטין:**

| ממד | גמיש? |
|---|---|
| **Instance Size** (`m5.xlarge` ↔ `m5.2xlarge`) | **כן** |
| **מערכת הפעלה** (Linux ↔ Windows) | **כן** |
| **Tenancy** (Default / Dedicated / Host) | **כן** |
| **Instance Family** (`m5` ↔ `c5`) | **לא** ב-EC2 Instance SP |
| **Region** | **לא** |

> [!tip] Compute Savings Plan מול EC2 Instance Savings Plan
> **Compute Savings Plan** — גמיש בין משפחות, בין Regions, **וגם מכסה Fargate ו-Lambda**.
> הנחה קצת קטנה יותר.
> **EC2 Instance Savings Plan** — נעול למשפחה ו-Region, **הנחה גדולה יותר**.
> במבחן: "maximum flexibility including serverless" → **Compute SP**.
> "steady EC2 usage in one family, maximum discount" → **EC2 Instance SP** או **RI**.

### 3.4 Spot Instances

- **עד ~90% הנחה** — **ה-compute הזול ביותר שקיים ב-AWS.**
- הרעיון: אתם רוכשים **capacity פנויה** ש-AWS לא הצליחה למכור.
- מגדירים **max price**. כל עוד ה-**spot price** הנוכחי נמוך ממנו — ה-instance רץ.
- ברגע ש-spot price **עולה מעל** ה-max שלכם — מקבלים **התראה של 2 דקות** ואז ה-instance נלקח.
- ה-spot price משתנה לפי היצע וביקוש באותו pool (instance type × AZ × OS).

**מתאים ל-:**

- batch jobs
- data analysis
- image / video processing
- workloads מבוזרים
- כל דבר עם **זמן התחלה וסיום גמיש**

**לא מתאים ל-:**

- jobs קריטיים
- **databases**
- כל דבר **stateful** שלא ניתן להפעיל מחדש

> [!warning] איך באמת מסיימים Spot — שאלה שנשאלת
> ה-**Spot Request** וה-**Spot Instance** הם שני דברים נפרדים.
> **ביטול ה-Spot Request לא מסיים את ה-instance!** אם רק תבטלו את הבקשה, ה-instance ימשיך לרוץ.
> **הסדר הנכון:** קודם **cancel** ל-Spot Request, ורק אז **terminate** ל-instances.
> אם תסיימו קודם את ה-instance וה-request עדיין `active` — הוא פשוט ישיק instance חדש.
> ניתן לבטל בקשות במצב `open`, `active` או `disabled` בלבד.

### 3.5 Spot Fleets

**Spot Fleet = אוסף Spot Instances + (אופציונלי) On-Demand Instances** שמנוהל כיחידה אחת.

- מגדירים **target capacity** ואילוצי מחיר, וה-fleet משיג אותה בעצמו.
- מגדירים **launch pools** — צירופים אפשריים של instance type, OS ו-AZ.
- ככל שיש **יותר pools**, ל-fleet יש יותר מקומות לברוח אליהם כשמחיר אחד קופץ.
- ה-fleet **מפסיק להשיק** כשהגיע ל-target capacity או ל-max cost שהגדרתם.

**אסטרטגיות הקצאה — טבלה שכדאי להכיר:**

| אסטרטגיה | מה עושה | מתי בוחרים |
|---|---|---|
| **lowestPrice** | לוקח מה-pool **הזול ביותר** | אופטימיזציית עלות טהורה, workload קצר |
| **diversified** | מפזר על **כל ה-pools** | זמינות — workload **ארוך** שאסור שייקטע כולו |
| **capacityOptimized** | בוחר את ה-pool עם ה-capacity **האופטימלי** | כשהעיקר הוא שה-instances באמת יעלו וישרדו |
| **priceCapacityOptimized** | קודם pools עם **capacity גבוה**, ומתוכם ה**זול ביותר** | **המומלץ של AWS** — הבחירה הטובה לרוב המקרים |

> [!tip] הזיהוי במבחן
> "**recommended**" / "best choice for most workloads" → **priceCapacityOptimized**.
> "long-running, must survive interruptions" → **diversified**.
> "absolute lowest cost, short job" → **lowestPrice**.

### 3.6 Dedicated Hosts

- **שרת פיזי שלם** שכל ה-capacity שלו מוקצית **רק לכם**.
- אתם **רואים ושולטים** במאפייני החומרה: sockets, cores, ו**היכן** כל instance יושב.
- **המודל היקר ביותר** מכל השבעה.
- אפשרויות רכישה: **On-Demand** (לפי שנייה על ה-host הפעיל) או **Reserved** ל-1/3 שנים
  (No Upfront / Partial / All Upfront) — עם הנחה משמעותית.

**למה בכלל:**

- **BYOL — Bring Your Own License.** רישיונות תוכנה ישנים שנספרים לפי **socket** או לפי **core**
  (Oracle, SQL Server, SAP) דורשים לדעת בדיוק על איזו חומרה הם רצים.
- **Compliance רגולטורי** שדורש בידוד פיזי מוכח.

### 3.7 Dedicated Instances

- ה-instances רצים על חומרה **ייעודית לכם** — לקוחות אחרים לא חולקים אותה.
- **אבל:** הם **כן** עשויים לחלוק חומרה עם instances אחרים **של אותו חשבון**.
- **אין שליטה על ה-placement.** אחרי `stop`/`start` ה-instance עלול לעבור לחומרה פיזית אחרת.
- אין גישה למאפייני החומרה, ולכן **לא פותר BYOL לפי socket/core**.

> [!warning] Dedicated Host מול Dedicated Instance — ההבחנה שנשאלת
> שניהם "חומרה לא משותפת". ההבדל הוא **הנראוּת והשליטה**:
> - **Host** — אתם רואים את השרת הפיזי, שולטים ב-placement, ומקבלים socket/core visibility → **BYOL**.
> - **Instance** — בידוד בלבד. אין שליטה, אין נראוּת, ה-placement יכול לזוז.
>
> **המילה `BYOL` או `per-socket / per-core licensing` בשאלה = Dedicated Host.**
> המילה "no other customers on our hardware" בלי דרישת רישוי = Dedicated Instance מספיק.

### 3.8 Capacity Reservations

- שומרים **capacity של On-Demand** ב-**AZ ספציפית**, **לכל משך שתרצו**.
- **אין התחייבות זמן** — יוצרים ומבטלים מתי שרוצים.
- **אין שום הנחה!** זו הנקודה הכי חשובה כאן.
- **מחויבים במחיר On-Demand בין אם ה-instances רצים ובין אם לא.**
- אפשר **לשלב** עם Regional RI או Savings Plan כדי לקבל גם את ההנחה **וגם** את הוודאות.
- מתאים ל-workload קצר, בלתי מופרע, שחייב לרוץ **ב-AZ מסוימת** (למשל ליד משאב zonal קיים).

> [!tip] ההבחנה החדה
> **Capacity Reservation = ודאות בלי הנחה.**
> **Regional RI / Savings Plan = הנחה בלי ודאות.**
> **Zonal RI = גם וגם.**

### 3.9 Instance Scheduler on AWS

- **לא שירות** — זהו **AWS Solution** שנפרס אצלכם דרך **CloudFormation**.
- מפעיל ומכבה משאבים אוטומטית לפי לוח זמנים → **חיסכון של עד ~70%**.
- הדוגמה הקלאסית: **כיבוי כל ה-EC2 של החברה מחוץ לשעות העבודה**.
- **תומך ב-:** EC2 instances, **EC2 Auto Scaling Groups**, ו-**RDS instances**.
- לוחות הזמנים מנוהלים ב-טבלת **DynamoDB**.
- מזהה משאבים לפי **tags**, ומשתמש ב-**Lambda** כדי לעצור/להפעיל.
- תומך ב-**cross-account** ו-**cross-region**.

---

## 4. 💰 עלות ותמחור — הסעיף המרכזי בשיעור

### 4.1 טבלת ההשוואה המלאה — כל שבעת המודלים

| מודל | הנחה יחסית | מחויבות | הבטחת capacity | סיכון הפסקה | מתי בוחרים | מקרה מבחן טיפוסי |
|---|---|---|---|---|---|---|
| **On-Demand** | **0% — היקר ביותר לשעה** | **אין** | לא | **אין** | עומס קצר, בלתי צפוי, שאסור להפסיק | "unpredictable workload", "short-term", "cannot be interrupted" |
| **Reserved Instance (Standard)** | **עד ~72%** | 1 או 3 שנים, תצורה **קבועה** | **כן ב-Zonal**, לא ב-Regional | אין | baseline יציב וידוע שלא ישתנה | "steady-state", "database running 24/7", "commit for 3 years" |
| **Convertible RI** | **עד ~66%** | 1 או 3 שנים, אך **ניתן לשנות** type/family/OS/tenancy | דומה ל-RI | אין | ארוך טווח שהצורה שלו עוד תשתנה | "long-term but may need to change instance family" |
| **Savings Plan** | **עד ~72%** | **$/שעה** ל-1 או 3 שנים | לא | אין | הוצאה יציבה עם גמישות בגודל/OS/tenancy | "commit to a consistent amount of compute spend", "flexibility across sizes" |
| **Spot Instance** | **עד ~90% — הזול ביותר** | **אין** | **לא** | **גבוה — 2 דקות התראה** | batch, analytics, stateless, fault-tolerant | "fault-tolerant", "flexible start/end time", "batch processing", "lowest possible cost" |
| **Dedicated Host** | **יקר מהכול** (Reserved Host → עד ~70% מזה) | לפי שנייה או 1/3 שנים | כן — השרת שלכם | אין | BYOL, compliance, שליטה בחומרה | "per-socket/per-core license", "BYOL", "regulatory requires dedicated physical server" |
| **Dedicated Instance** | תוספת מעל On-Demand | אין | לא | אין | בידוד חומרה בלי דרישת רישוי | "no other customers share our hardware" |
| **Capacity Reservation** | **0% — מחיר On-Demand מלא** | אין (מבטלים מתי שרוצים) | **כן, ב-AZ שבחרתם** | אין | ודאות capacity ב-AZ ספציפית | "guarantee capacity in a specific AZ", "must be available during peak event" |

> [!info] הסדר שכדאי לזכור מהיקר לזול
> **Dedicated Host > Dedicated Instance > On-Demand = Capacity Reservation > Convertible RI > Savings Plan ≈ Standard RI > Spot**

### 4.2 מה מגדיל את ההנחה בתוך RI ו-Savings Plan

| ממד | פחות הנחה | יותר הנחה |
|---|---|---|
| **תקופה** | שנה אחת | **3 שנים** |
| **תשלום מראש** | No Upfront | Partial → **All Upfront** |
| **גמישות** | Convertible RI | **Standard RI** |
| **טווח הכיסוי** | Compute Savings Plan | **EC2 Instance Savings Plan** |

**הכלל:** כל ויתור על גמישות או על נזילות = עוד הנחה.

### 4.3 על מה מחייבים — מעבר לשעת ה-compute

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| שעת compute | לפי שנייה (Linux/Windows) או שעה | תלוי type, Region, OS |
| **רישוי OS** | כלול במחיר לשעה | Windows/RHEL יקרים מ-Linux — הפרש קבוע |
| **EBS volumes** | GB-חודש | **ממשיך לחייב גם כשה-instance stopped** |
| **Public IPv4** | לשעה, לכל כתובת | מחויב גם בשימוש |
| **Data transfer** | GB egress, cross-AZ, cross-Region | לרוב מפתיע בגודלו |
| **Capacity Reservation לא מנוצלת** | **מחיר On-Demand מלא** | משלמים על אוויר בכוונה תחילה |
| **Spot** | לפי ה-**spot price** בפועל, לא ה-max שהגדרתם | ה-max הוא תקרה, לא מחיר |

### 4.4 🚩 עלויות נסתרות

- **RI ל-3 שנים שכבר לא רלוונטי.** העומס עבר לדור חדש או ל-containers, וה-RI ממשיך לחייב.
  מוצא: **Reserved Instance Marketplace** (למכירה) או **Convertible RI** (להמרה).
- **Savings Plan שהוגדר גבוה מדי.** התחייבתם ל-$20/שעה ואתם מנצלים $12 — ההפרש נשרף.
  **תמיד מתחייבים רק על ה-baseline המדוד**, לא על הממוצע ולא על השיא.
- **Capacity Reservation שנשכחה.** ממשיכה לחייב במחיר מלא בלי שאף instance רץ שם.
  זו העלות הנסתרת הכי מסוכנת כי אין שום סימן שמשהו רץ.
- **Spot Requests יתומים.** ביטלתם instances אבל לא את ה-request — הוא מייצר חדשים.
- **Dedicated Host שמנוצל חלקית.** משלמים על השרת השלם גם אם רץ עליו instance אחד.
- **`t` instances ב-unlimited mode** בעומס מתמשך — חיוב burst מעל ה-baseline שנצבר בשקט.
- **EBS ו-EIP יתומים** אחרי termination — ראו [[05 - EC2 Fundamentals]].
- **החשבון של סביבות dev שרצות 24/7** — 168 שעות בשבוע במקום ~50 שנחוצות.

### 4.5 💡 טיפים לחיסכון

- **מדדו לפני שמתחייבים.** Cost Explorer ו-**AWS Compute Optimizer** מראים את ה-baseline האמיתי.
- **התחייבו רק על ה-baseline**, ותנו לשיא לרוץ On-Demand או Spot.
- **Savings Plan לפני RI** ברוב המקרים — אותה הנחה, הרבה יותר גמישות.
- **Spot לכל מה ש-stateless** — CI/CD runners, עיבוד תמונות, ETL, אימון ML, rendering.
- **ASG עם Mixed Instances Policy** — baseline On-Demand + שאר ה-capacity ב-Spot,
  עם **priceCapacityOptimized** ו-**הרבה instance types** ברשימה. ראו [[07 - Auto Scaling]].
- **Instance Scheduler** לכיבוי dev/test מחוץ לשעות עבודה — עד ~70% על אותן סביבות.
- **שדרגו דור** לפני שקונים RI. אין טעם לנעול 3 שנים על דור ישן ויקר יותר.
- **מכרו RIs מיותרים** ב-Marketplace במקום לספוג אותם עד הסוף.
- **בדקו את הרישוי:** אם יש BYOL לפי core — Dedicated Host עשוי לצאת **זול יותר** בסך הכול
  למרות שהוא יקר יותר לשעה, כי הוא חוסך רכישת רישיונות חדשים.

---

## 5. ⚖️ השוואות מכריעות

### Reserved Instance מול Savings Plan

| קריטריון | **Reserved Instance** | **Savings Plan** |
|---|---|---|
| מתחייבים ל- | **instance מסוים** (type, OS, tenancy, Region) | **סכום $ לשעה** |
| הנחה מקסימלית | **עד ~72%** | **עד ~72%** (זהה) |
| גמישות בגודל | Standard: מוגבלת · Convertible: כן | **כן** |
| גמישות במשפחה | רק Convertible | Compute SP: **כן** · EC2 Instance SP: לא |
| גמישות ב-Region | רק Convertible | Compute SP: **כן** · EC2 Instance SP: לא |
| מכסה Lambda / Fargate | **לא** | **Compute SP: כן** |
| הבטחת capacity | **כן ב-Zonal RI** | **לא** |
| ניתן למכור | **כן — RI Marketplace** | **לא** |
| שימוש מעבר להתחייבות | לא רלוונטי | מחויב **On-Demand** |

> [!info] שורה תחתונה
> אותה הנחה, פחות כאב ראש → **Savings Plan** הוא ברירת המחדל המודרנית.
> **RI** נשאר עדיף כשצריך **הבטחת capacity ב-AZ** (Zonal) או **אפשרות מכירה**.

### Dedicated Host מול Dedicated Instance

| קריטריון | **Dedicated Host** | **Dedicated Instance** |
|---|---|---|
| מה מקבלים | **שרת פיזי שלם** | חומרה לא משותפת עם לקוחות אחרים |
| חולק עם instances של אותו חשבון | אתם מחליטים | **כן, ייתכן** |
| שליטה ב-placement | **כן** | **לא** — עלול לזוז ב-stop/start |
| נראוּת socket / core | **כן** | **לא** |
| **BYOL per-socket/core** | **כן — זו המטרה** | **לא** |
| מודלי רכישה | On-Demand או Reserved (1/3 שנים) | On-Demand בעיקר |
| עלות | **הגבוהה ביותר** | תוספת מעל On-Demand |

### Spot מול On-Demand

| קריטריון | **Spot** | **On-Demand** |
|---|---|---|
| מחיר | **עד ~90% זול יותר** | מחיר מלא |
| יכול להיעלם | **כן — 2 דקות התראה** | לא |
| מחיר יציב | לא — משתנה לפי היצע/ביקוש | **כן** |
| מתאים ל-DB | **לא, בשום אופן** | כן |
| מתאים ל-batch | **כן, אידיאלי** | כן אבל יקר מיותר |
| מנוהל דרך | Spot Request / **Spot Fleet** / ASG | השקה רגילה |

### Capacity Reservation מול Zonal RI

| קריטריון | **Capacity Reservation** | **Zonal RI** |
|---|---|---|
| הנחה | **אין** | **עד ~72%** |
| הבטחת capacity ב-AZ | **כן** | **כן** |
| התחייבות זמן | **אין** — ביטול מתי שרוצים | 1 או 3 שנים |
| משלמים כשלא רצים | **כן** | כן (התחייבות) |
| מתי בוחרים | אירוע קצר שדורש ודאות | baseline ארוך שדורש ודאות |

> [!info] שורה תחתונה כללית
> **הזול ביותר = Spot** (אם אפשר לספוג הפסקה).
> **הזול ביותר בלי סיכון = Savings Plan / RI** (אם העומס יציב וידוע).
> **הכי גמיש = On-Demand.**
> **ודאות capacity = Capacity Reservation או Zonal RI.**
> **רישוי/compliance = Dedicated Host.**

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בתמחור EC2** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | ההוצאה נראית, מיוחסת ומנוהלת אוטומטית | **tagging** לכל משאב לייחוס עלות; Cost Explorer ו-Budgets עם התראות; **Instance Scheduler** במקום כיבוי ידני; אוטומציה של רכישה וחידוש RI |
| **Security** | אופטימיזציית עלות לא באה על חשבון בקרות | Spot לא פוטר מ-patching ומ-IAM Roles; Dedicated Host כשה-compliance **באמת** דורש בידוד פיזי — לא כברירת מחדל יקרה |
| **Reliability** | ה-capacity לא תלויה במזל של השוק | **baseline On-Demand/RI + Spot כתוספת**, לא Spot בלבד; Spot Fleet **diversified** על הרבה pools; **Capacity Reservation** לאירוע שיא ידוע; טיפול נכון בהתראת 2 הדקות (drain, checkpoint) |
| **Performance Efficiency** | בוחרים לפי פרופיל העומס, לא לפי המחיר בלבד | right-sizing לפי CPU/RAM/network/IO; שדרוג דור לפני רכישת RI; **Compute Optimizer** להמלצות; לא לרדת בגודל עד שהאפליקציה חונקת |
| **Cost Optimization** | מתחייבים רק על מה שנמדד | Savings Plan על ה-**baseline המדוד**; Spot ל-queue ול-batch; מכירת RI מיותר ב-Marketplace; ביטול Capacity Reservations שנשכחו; ניקוי EBS/EIP יתומים |
| **Sustainability** | פחות שעות compute = פחות חומרה פועלת | כיבוי idle ב-Instance Scheduler; דורות חדשים ו-**Graviton** ליעילות אנרגטית; scaling לפי ביקוש אמיתי; **Spot מנצל capacity שכבר פועלת ממילא** |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "fault-tolerant" / "can be interrupted" / "flexible start and end time" | **Spot Instances** |
| "batch processing" / "data analysis" / "image processing" | **Spot** (או Spot Fleet) |
| "lowest possible cost" + "stateless" | **Spot** |
| "steady state" / "runs 24/7" / "predictable usage" | **Reserved Instance** או **Savings Plan** |
| "commit to a consistent amount of compute usage per hour" | **Savings Plan** |
| "flexibility across instance families and Regions, also Fargate/Lambda" | **Compute Savings Plan** |
| "may need to change the instance family later" | **Convertible Reserved Instance** |
| "guarantee capacity in a specific Availability Zone" | **Capacity Reservation** או **Zonal RI** |
| "reserve capacity, no discount needed, cancel anytime" | **Capacity Reservation** |
| "per-socket / per-core license" / "BYOL" | **Dedicated Host** |
| "regulatory requirement — dedicated physical server" | **Dedicated Host** |
| "no other customers on the same hardware" (בלי רישוי) | **Dedicated Instance** |
| "unpredictable, short-term, cannot be interrupted" | **On-Demand** |
| "stop instances outside business hours automatically" | **Instance Scheduler on AWS** |
| "recommended Spot allocation strategy" | **priceCapacityOptimized** |
| "long-running Spot workload, maximize availability" | **diversified** |
| "combine cheap capacity with a reliable baseline" | **ASG Mixed Instances**: On-Demand baseline + Spot |
| "2-minute notification" | **Spot interruption notice** |

### טעויות נפוצות

> [!warning] מלכודת 1 — "הכי זול" הוא לא תמיד Spot
> **הניסוח:** "The company wants the lowest cost for its production payment database."
> **הטעות:** לקפוץ ל-Spot כי הוא הזול ביותר.
> **הנכון:** **Spot אינו מתאים ל-DB ולעומס קריטי.** DB שרץ 24/7 → **RI או Savings Plan**.
> "הכי זול" מוגבל תמיד בדרישת הרציפות.

> [!warning] מלכודת 2 — ביטול Spot Request ≠ סיום ה-instance
> **הניסוח:** "We cancelled the Spot Request but we're still being billed."
> **הטעות:** להניח שביטול הבקשה מכבה הכול.
> **הנכון:** אלה שני משאבים. **קודם cancel ל-request, אחר כך terminate ל-instances.**
> בסדר ההפוך — ה-request פשוט משיק instance חדש.

> [!warning] מלכודת 3 — Capacity Reservation נותנת הנחה
> **הניסוח:** "Use Capacity Reservations to reduce EC2 costs."
> **הטעות:** לחשוב שהזמנת capacity = הוזלה.
> **הנכון:** **Capacity Reservation לא נותנת שום הנחה** ומחייבת במחיר On-Demand מלא
> **גם כשאף instance לא רץ**. לחיסכון משלבים אותה עם **Regional RI או Savings Plan**.

> [!warning] מלכודת 4 — Dedicated Instance במקום Dedicated Host ל-BYOL
> **הניסוח:** "We need to bring our own per-core Oracle licenses to AWS."
> **הטעות:** לבחור Dedicated Instance כי גם הוא "חומרה ייעודית".
> **הנכון:** רישוי לפי socket/core דורש **נראוּת ושליטה בחומרה** — רק **Dedicated Host** נותן זאת.

> [!warning] מלכודת 5 — התחייבות על השיא במקום על ה-baseline
> **הניסוח:** "Peak usage is 100 instances, so we bought 100 Reserved Instances."
> **הטעות:** לכסות את השיא בהתחייבות.
> **הנכון:** מתחייבים על ה-**baseline** בלבד (מה שרץ תמיד), ואת השיא מכסים ב-On-Demand או Spot.
> RI שלא מנוצל = כסף שנשרף לשנה או שלוש.

> [!warning] מלכודת 6 — max price של Spot הוא המחיר ששילמתם
> **הניסוח:** "We set a max Spot price of $0.10, so we pay $0.10 per hour."
> **הטעות:** לבלבל בין תקרה למחיר.
> **הנכון:** משלמים את ה-**spot price בפועל**, שהוא בדרך כלל נמוך הרבה יותר.
> ה-max הוא רק הסף שמעליו ה-instance נלקח.

> [!warning] מלכודת 7 — Savings Plan "מבטיח" capacity
> **הניסוח:** "We have a Savings Plan, so capacity is guaranteed during peak."
> **הטעות:** לערבב הנחה עם הזמנה.
> **הנכון:** **Savings Plan הוא הסדר חיוב בלבד.** לוודאות capacity צריך
> **Zonal RI** או **Capacity Reservation**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברת מדיה מריצה על AWS ארבעה סוגי עומס, ורוצה לחתוך את חשבון ה-compute בלי לפגוע בשירות:

1. **Web tier** — רץ 24/7, לעולם לא יורד מ-20 instances. שיאים ל-60 בשעות הערב.
2. **Transcoding pipeline** — ממיר וידאו בלילה. ניתן לעצור ולהתחיל מחדש בלי נזק.
3. **סביבות dev/test** — 40 instances שרצות 24/7 אבל בשימוש רק 09:00–19:00 בימי חול.
4. **מערכת billing ישנה** — Oracle עם רישיון **per-core** שהחברה כבר רכשה.

בנוסף: משדרים אירוע חי גדול פעם ברבעון וחייבים ודאות שתהיה capacity ב-AZ שבה יושב ה-cache.

```text
                        ┌──────────────────────────────┐
   24/7 baseline  ──────│ 20 × Savings Plan / RI 3y    │
                        └──────────────────────────────┘
   שיא ערב        ──────│ +40 On-Demand דרך ASG         │
                        └──────────────────────────────┘
   Transcoding    ──────│ Spot Fleet, priceCapacityOpt │
                        │ הרבה pools, checkpoint לכל job│
                        └──────────────────────────────┘
   dev / test     ──────│ Instance Scheduler: 09-19 בלבד│
                        └──────────────────────────────┘
   Oracle billing ──────│ Dedicated Host (Reserved 3y)  │
                        └──────────────────────────────┘
   אירוע רבעוני   ──────│ Capacity Reservation ב-AZ-a   │
                        └──────────────────────────────┘
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **20 instances ב-Savings Plan ל-3 שנים** | זהו ה-baseline המדוד שרץ תמיד. עד ~72% הנחה על החלק היציב |
| **Savings Plan ולא Standard RI** | אותה הנחה, אבל גמיש בין גדלים ו-OS — הצוות משנה types מדי פעם |
| **שיא הערב ב-On-Demand דרך ASG** | 40 instances נוספות רק כמה שעות ביום. RI עליהן יהיה בזבוז |
| **Transcoding ב-Spot Fleet** | עומס batch, stateless, גמיש בזמן — הפרופיל המדויק ל-Spot. עד ~90% הנחה |
| **אסטרטגיה `priceCapacityOptimized`** | ההמלצה של AWS: קודם pools עם capacity, ומתוכם הזול — פחות הפסקות |
| **הרבה launch pools** ל-fleet | ככל שיש יותר צירופי type/AZ, ה-fleet בורח ממחיר שקופץ |
| **checkpoint לכל job + טיפול בהתראת 2 הדקות** | Spot נלקח בהתראה קצרה; ה-job חייב להתחיל מחדש מהנקודה ולא מהתחלה |
| **Instance Scheduler ל-dev/test** | 40 instances × 118 שעות מיותרות בשבוע. עד ~70% חיסכון על הסביבות האלה |
| **Dedicated Host ל-Oracle** | רישוי **per-core** מחייב נראוּת חומרה. Dedicated Instance **לא** יספיק |
| **Reserved Dedicated Host ל-3 שנים** | ה-host הוא יקר, אבל reservation חותכת אותו משמעותית והעומס קבוע |
| **Capacity Reservation ל-AZ-a לפני האירוע** | ודאות שתהיה capacity ליד ה-cache. משלבים עם ה-Savings Plan הקיים כדי לקבל גם הנחה |
| **מבטלים את ה-Capacity Reservation אחרי האירוע** | היא מחייבת במחיר מלא גם כשלא רצים — לא משאירים אותה פתוחה |

**למה לא הכול ב-Spot?**
ה-web tier הוא production חי. הפסקה של 2 דקות באמצע שידור = תקלת שירות.
Spot מתאים ל-transcoding בדיוק כי שם ההפסקה עולה רק זמן, לא לקוחות.

**למה לא RI ל-3 שנים על כל 60 ה-instances?**
כי 40 מהן רצות רק בערב. RI מחייב 24/7 — היינו משלמים על 40 instances מושבתות רוב היום.
**מתחייבים על ה-baseline, לא על השיא.**

**למה לא Dedicated Instances במקום Dedicated Host ל-Oracle?**
Dedicated Instance נותן בידוד אבל **לא נראוּת socket/core**, ולא מונע העברה לחומרה אחרת
ב-stop/start. רישוי per-core דורש בדיוק את מה שרק Host נותן.

**למה לא פשוט Capacity Reservation קבועה כדי להיות בטוחים?**
כי היא מחייבת במחיר On-Demand מלא **סביב השעון**, בלי הנחה. זה הפוך מהמטרה.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **אחוזי הנחה מדויקים.** AWS משנה אותם. מספיק לדעת **Spot ≈ עד 90%**, **RI/SP ≈ עד 72%**,
  **Convertible ≈ עד 66%**, ואת **סדר** היוקר בין המודלים.
- **מחירים בדולרים** לכל instance type. הטבלה בשקפים היא להמחשה בלבד.
- **הנוסחה המדויקת** שלפיה AWS מחשבת spot price.
- **מבנה ה-API/CLI** של Spot Fleet requests.
- **פרטי ה-CloudFormation** של Instance Scheduler — מספיק לדעת שהוא Solution ולא שירות,
  ושהוא מכסה EC2, ASG ו-RDS.
- **כללי הרישוי הספציפיים** של Oracle/SQL Server. מספיק לזהות `BYOL` → Dedicated Host.
- **ההיסטוריה של Spot Blocks** — הופסק לרישום חדש.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **On-Demand** — היקר ביותר לשעה, אפס התחייבות. Linux/Windows **לפי שנייה**, שאר ה-OS **לפי שעה**.
- **Reserved Instance** — עד **~72%**, 1 או 3 שנים, תצורה נעולה. **All Upfront = ההנחה הגדולה ביותר**.
- **Zonal RI = הנחה + הבטחת capacity ב-AZ. Regional RI = הנחה בלבד.**
- **Convertible RI** — עד **~66%**, אבל אפשר לשנות type, family, OS, tenancy ו-scope.
- **RI Marketplace** — אפשר **לקנות ולמכור** RIs. Savings Plan אי אפשר למכור.
- **Savings Plan** — עד **~72%**, מתחייבים ל-**$/שעה**. שימוש מעבר לכך = מחיר On-Demand.
- **EC2 Instance SP** נעול ל-**family + Region**, גמיש ב-**size, OS, tenancy**.
  **Compute SP** גמיש בין families ו-Regions ומכסה גם **Fargate ו-Lambda**.
- **Spot** — עד **~90%**, ה-compute הזול ביותר. **התראת 2 דקות** לפני שהוא נלקח.
- **משלמים את ה-spot price בפועל**, לא את ה-max שהגדרתם.
- **ביטול Spot Request לא מסיים instances.** קודם cancel לבקשה, אחר כך terminate.
- **Spot Fleet strategies:** `lowestPrice` · `diversified` (ארוך טווח) · `capacityOptimized` ·
  **`priceCapacityOptimized` = ההמלצה**.
- **Dedicated Host** — שרת פיזי שלם, שליטה ב-placement, נראוּת socket/core → **BYOL ו-compliance**.
  **המודל היקר ביותר.**
- **Dedicated Instance** — חומרה לא משותפת עם לקוחות אחרים, **בלי שליטה ב-placement**, **לא ל-BYOL**.
- **Capacity Reservation** — ודאות capacity ב-AZ, **אפס הנחה**, מחויבים **גם כשלא רצים**,
  אין התחייבות זמן. משלבים עם RI/SP לקבלת הנחה.
- **Instance Scheduler on AWS** — Solution ב-CloudFormation, tags + Lambda + DynamoDB,
  מכבה **EC2, ASG ו-RDS** מחוץ לשעות עבודה, עד **~70%** חיסכון.
- **הכלל המנצח:** **baseline** ב-RI/SP · **שיא** ב-On-Demand · **batch** ב-Spot.

---

## 11. ✅ בדיקת הבנה

1. workload רץ 24/7 בדיוק באותה תצורה כבר שנתיים. מהם שני המודלים המתאימים, ומה ההבדל ביניהם?
2. הצוות ביטל Spot Request והחיוב נמשך. מה קרה ומה הסדר הנכון?
3. חברה חייבת ודאות ש-30 instances יהיו זמינות ב-`us-east-1a` במהלך אירוע של יומיים. מה בוחרים, ומה **לא** מקבלים?
4. מהו ההבדל המעשי בין Dedicated Host ל-Dedicated Instance, ומה המילה בשאלה שמכריעה?
5. השיא הוא 100 instances וה-baseline הוא 30. כמה RIs קונים ולמה?
6. איזו אסטרטגיית Spot Fleet בוחרים ל-job שרץ 14 שעות ואסור שייקטע כולו בבת אחת? למה לא `lowestPrice`?
7. מה מקבלים מ-Savings Plan שלא מקבלים מ-RI, ומה מקבלים מ-RI שלא מקבלים מ-Savings Plan?
8. 40 סביבות dev רצות 24/7 ומשמשות רק בשעות עבודה. מה הפתרון המדויק ואיזה שירותים הוא מכסה?

<details>
<summary>תשובות</summary>

1. **Reserved Instance** ו-**Savings Plan** — שניהם עד **~72%** הנחה ל-1 או 3 שנים.
   ההבדל: RI מתחייב ל-**instance מסוים** ומאפשר **הבטחת capacity (Zonal)** ו-**מכירה ב-Marketplace**;
   Savings Plan מתחייב ל-**$ לשעה** ונותן גמישות בגודל, ב-OS וב-tenancy (וב-Compute SP גם בין families ו-Regions).
2. **ה-Spot Request וה-Spot Instance הם משאבים נפרדים.** ביטול הבקשה לא מכבה instances רצים.
   הסדר הנכון: **cancel** ל-Spot Request → ואז **terminate** ל-instances.
   בסדר ההפוך, בקשה `active` תשיק instance חדש במקום זה שסיימתם.
3. **Capacity Reservation** ב-`us-east-1a`. **לא מקבלים שום הנחה** — מחויבים במחיר On-Demand מלא,
   וגם על capacity שלא נוצלה. כדי לקבל גם הנחה משלבים עם **Regional RI או Savings Plan**.
   לזכור לבטל אחרי האירוע.
4. **Dedicated Host** = שרת פיזי שלם עם **שליטה ב-placement ונראוּת socket/core**.
   **Dedicated Instance** = בידוד מלקוחות אחרים בלבד, בלי שליטה, וה-placement עלול לזוז ב-stop/start.
   **המילה המכריעה: `BYOL` או רישוי `per-socket`/`per-core` → Dedicated Host.**
5. **30** — רק ה-**baseline**. את 70 הנוספות מכסים ב-On-Demand או Spot לפי אופי העומס.
   RI על השיא פירושו לשלם 24/7 על instances שרצות חלק קטן מהזמן.
6. **`diversified`** — מפזר על כל ה-pools, כך שקפיצת מחיר ב-pool אחד לא מפילה את כל ה-fleet.
   `lowestPrice` מרכז הכול ב-pool אחד זול, ולכן **כל ה-job עלול להיקטע יחד**.
   (`priceCapacityOptimized` היא ההמלצה הכללית של AWS לרוב המקרים.)
7. מ-**Savings Plan** מקבלים **גמישות** — שינוי גודל, OS ו-tenancy בלי לאבד את ההנחה,
   וב-Compute SP גם כיסוי ל-**Fargate ו-Lambda**.
   מ-**RI** מקבלים **הבטחת capacity ב-AZ** (Zonal) ו-**אפשרות למכור** את ההתחייבות ב-Marketplace.
8. **Instance Scheduler on AWS** — פתרון (Solution) שנפרס ב-**CloudFormation**, לא שירות.
   מזהה משאבים לפי **tags**, מנהל לוחות זמנים ב-**DynamoDB**, ומפעיל **Lambda** לכיבוי/הפעלה.
   מכסה **EC2 instances, EC2 Auto Scaling Groups ו-RDS instances**, כולל cross-account ו-cross-region.
   חיסכון של עד **~70%**.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[05 - EC2 Fundamentals]] · [[07 - Auto Scaling]] · [[37 - Cost Optimization]] · [[19 - EBS and EC2 Storage]] · [[33 - High Availability and Scalability]] · [[38 - Serverless and Modern Architectures]] · [[02 - AWS Well-Architected Framework]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 939–1140, 16105–16124
