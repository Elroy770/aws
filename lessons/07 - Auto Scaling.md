---
lesson: 07
title: Auto Scaling
domain: Design High-Performing Architectures
services: [EC2 Auto Scaling, CloudWatch, ELB, Application Auto Scaling, ECS]
tags: [saa-c03, compute, scalability, elasticity, high-availability]
---

# 07 — Auto Scaling

> [!abstract] בשורה אחת
> Auto Scaling Group הוא המנגנון שמחזיק את מספר ה-EC2 בדיוק על הרמה שהעומס דורש — ומחליף לבד כל instance שנפל.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה capacity קבוע הוא גם יקר וגם שביר |
| 2 | איך זה עובד | Launch Template, min/desired/max, health checks, מחזור החיים של instance |
| 3 | פירוק מפורט | כל סוגי ה-scaling policies, metrics, cooldown, termination policy |
| 4 | עלות | ASG חינם — אבל מה כן עולה כסף ואיפה מבזבזים |
| 5 | השוואות | Target Tracking מול Step מול Scheduled מול Predictive |
| 6 | Well-Architected | איך ASG משרת כל אחד מששת ה-Pillars |
| 7 | מלכודות | מילות מפתח שמסגירות את התשובה הנכונה |
| 8 | Scenario | ארכיטקטורת web מלאה עם ALB + ASG רב-AZ |

**מונחי מפתח בשיעור:** `Launch Template` · `Desired Capacity` · `Target Tracking` · `Cooldown` · `Health Check Grace Period` · `Instance Refresh` · `Lifecycle Hook`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- העומס על אפליקציה אף פעם לא קבוע — יש שיא בבוקר, שפל בלילה, ופיצוץ ביום מבצע.
- אם מקצים capacity לפי השיא — משלמים על שרתים ריקים רוב שעות היממה.
- אם מקצים capacity לפי הממוצע — האתר קורס בדיוק ברגע שהוא הכי חשוב.
- כשה-instance נופל בשלוש לפנות בוקר, מישהו צריך לקום ולהרים אותו ידנית.
- כל instance חדש צריך גם להירשם ל-load balancer — עוד צעד ידני שנשכח.

### מה השירות פותר

- **Scale out** — הוספת instances אוטומטית כשהעומס עולה.
- **Scale in** — הסרת instances כשהעומס יורד, כדי לא לשלם על אוויר.
- **גבולות בטיחות** — min ו-max מבטיחים שלא נרד מתחת לרף ולא נתפוצץ מעבר לתקציב.
- **Self-healing** — instance שנכשל ב-health check מוחלף אוטומטית באחד חדש.
- **רישום אוטומטי ל-ELB** — כל instance חדש מצטרף ל-target group בלי התערבות.

> [!tip] האנלוגיה
> ASG הוא מנהל משמרת במסעדה: הוא לא מגיש אוכל בעצמו (זה ה-load balancer),
> אבל הוא מחליט כמה מלצרים עומדים ברצפה ומחליף מיד את מי שהלך הביתה חולה.

> [!info] נקודה שמבלבלת רבים
> **ASG הוא לא load balancer.** הוא לא מנתב בקשות ולא מסתכל על HTTP.
> הוא מנהל את הצי בלבד. הניתוב הוא תמיד עבודתו של ELB — ראו [[08 - Elastic Load Balancing]].

---

## 2. ⚙️ איך זה עובד

### 2.1 שלוש המספרים שמגדירים ASG

| ערך | מה זה אומר | התנהגות |
|---|---|---|
| **Minimum Capacity** | הרצפה — ASG לעולם לא ירד מתחת לזה | scale-in נעצר כאן |
| **Desired Capacity** | הכמות שה-ASG מנסה להחזיק *כרגע* | זה הערך שה-scaling policies משנות |
| **Maximum Capacity** | התקרה — ASG לעולם לא יעלה מעל זה | הגנת תקציב מפני scaling מטורף |

- ה-desired תמיד חייב להיות בין ה-min ל-max.
- אם instance מת, ה-desired לא משתנה — ASG פשוט משיק אחד חדש כדי לחזור אליו.
- שינוי ידני של desired הוא scaling פעולה לגיטימית לחלוטין (למשל מתוך script).

### 2.2 Launch Template — ה-DNA של כל instance

ה-Launch Template הוא התבנית שממנה כל instance חדש נולד. הוא כולל:

- **AMI** ו-**Instance Type**.
- **EC2 User Data** — הסקריפט שרץ באתחול.
- **EBS Volumes** — גודל, סוג, encryption.
- **Security Groups**.
- **SSH Key Pair**.
- **IAM Instance Profile** — ה-role שמעניק הרשאות לאפליקציה.
- **VPC + Subnets** — באילו AZs מותר להשיק.
- **Load Balancer / Target Group** — לאן להירשם.

> [!warning] Launch Configuration הוא deprecated
> בשקפים הישנים מופיע גם "Launch Configuration". AWS הפסיקה לתמוך בו ליצירה חדשה.
> **תמיד Launch Template** — הוא היחיד שתומך בגרסאות, ב-Mixed Instances Policy וב-Spot.

### 2.3 מחזור החיים של scaling

```text
CloudWatch Metric  →  CloudWatch Alarm  →  Scaling Policy  →  ASG משנה Desired
        ↑                                                              ↓
   מדידה מה-instances                                    Launch / Terminate Instance
                                                                       ↓
                                                          רישום / הסרה מה-Target Group
```

- ה-metric נמדד כממוצע **על כל ה-ASG**, לא על instance בודד.
- ה-alarm הוא שמפעיל את המדיניות — לא ה-metric עצמו.
- אחרי פעולת scaling נכנסים ל-cooldown כדי לאפשר ל-metrics להתייצב.

### 2.4 Health Checks — מי מחליט ש-instance חולה

| סוג | מה נבדק | מתי מספיק |
|---|---|---|
| **EC2 Status Checks** | ברירת המחדל. האם ה-VM חי, האם הרשת והדיסק תקינים | תשתית בלבד |
| **ELB Health Check** | האם האפליקציה מחזירה 200 על ה-health path | הכי חשוב לאפליקציית web |
| **Custom Health Check** | הודעה ידנית דרך API על מצב instance | לוגיקה עסקית מיוחדת |

- **הכי חשוב למבחן:** EC2 status check לא יודע שה-Java process מת. רק ELB health check יודע.
- אם התרחיש מתאר "instance חי אבל האפליקציה קרסה ואף אחד לא החליף אותו" — התשובה היא **להפעיל ELB health checks ב-ASG**.
- **Health Check Grace Period** (ברירת מחדל 300 שניות) — פרק הזמן שבו ASG מתעלם מ-health checks אחרי השקה, כדי לתת לאפליקציה להתחיל.

> [!warning] Grace Period קצר מדי = לולאת מוות
> אם האפליקציה עולה תוך 4 דקות וה-grace period הוא דקה,
> ASG יהרוג כל instance לפני שהספיק לעלות — וישיק אחד חדש, שוב ושוב.

### 2.5 פיזור בין Availability Zones

- ASG חי בתוך **Region אחד** ויכול לפרוס על מספר subnets ב-AZs שונים.
- ASG **לא** יכול לפרוס בין Regions. Multi-Region דורש ASG נפרד בכל Region.
- ASG מנסה לשמור על איזון בין ה-AZs (AZ Rebalancing) — לפעמים ישיק instance לפני שיסגור אחר.

```text
                        Users
                          ↓
              Application Load Balancer
              ↙                       ↘
      AZ-a (subnet)              AZ-b (subnet)
   ┌──────────────┐            ┌──────────────┐
   │ EC2   EC2    │            │ EC2   EC2    │
   └──────────────┘            └──────────────┘
        └────────  Auto Scaling Group  ────────┘
```

---

## 3. 🔍 פירוק מפורט

### 3.1 Scaling Policies — כל הסוגים

| Policy | איך מגדירים | דוגמה | מתי בוחרים |
|---|---|---|---|
| **Target Tracking** | metric + ערך יעד | "ממוצע CPU יישאר סביב 40%" | ברירת המחדל. הכי פשוט להגדרה |
| **Simple Scaling** | alarm אחד → פעולה אחת | "CPU > 70% → הוסף 2" | legacy, פשוט וגס |
| **Step Scaling** | alarm + מדרגות לפי עוצמת החריגה | "CPU 70–85% → +2, מעל 85% → +4" | כשצריך תגובה פרופורציונלית |
| **Scheduled Scaling** | תאריך/שעה קבועים | "כל שישי ב-17:00 min=10" | עומס **צפוי מראש** |
| **Predictive Scaling** | ML חוזה עומס ומקצה מראש | חיזוי מחזורי יומי/שבועי | עומס מחזורי עם היסטוריה |

**נקודות דיוק:**

- **Target Tracking** הוא היחיד שמנהל לבדו את ה-alarms מאחורי הקלעים — לא מגדירים אותם ידנית.
- **Step Scaling** מגיב לפי כמה חרגנו מהסף, לא רק אם חרגנו — זה ההבדל המהותי מ-Simple.
- **Predictive Scaling** לומד מהיסטוריה (עד שבועיים אחורה) ומקצה capacity **לפני** שהעומס מגיע.
  זה פותר את הבעיה הכי כואבת ב-reactive scaling: הזמן שלוקח ל-instance לעלות.
- אפשר לשלב: Predictive/Scheduled ל-baseline + Target Tracking כרשת ביטחון לחריגות.

### 3.2 באילו metrics כדאי לבחור

| Metric | מתי זה ה-metric הנכון | מלכודת |
|---|---|---|
| `CPUUtilization` | אפליקציה CPU-bound (עיבוד, קידוד) | אפליקציית web I/O-bound תיראה בטלה תחת עומס |
| `RequestCountPerTarget` | **אפליקציית web מאחורי ALB** | ה-metric הזה מגיע מה-ALB, לא מה-instance |
| `NetworkIn` / `NetworkOut` | העברת קבצים, streaming, אפליקציה network-bound | לא רלוונטי לאפליקציה חישובית |
| `ApproximateNumberOfMessagesVisible` (SQS) | worker fleet שצורך תור | ה-metric הוא של התור, לא של ה-ASG |
| Custom metric דרך CloudWatch | תור פנימי, session count, latency אפליקטיבי | דורש קוד שדוחף את ה-metric |

> [!tip] כלל אצבע למבחן
> "web application behind an ALB" + "scale appropriately" → **RequestCountPerTarget**.
> "queue backlog growing" → **ApproximateNumberOfMessagesVisible**.
> CPU היא התשובה רק כשהתרחיש באמת מתאר עומס חישובי.

### 3.3 Cooldown ו-Warm-up

| מנגנון | ברירת מחדל | מה עושה |
|---|---|---|
| **Scaling Cooldown** | 300 שניות | אחרי פעולת scaling, ASG מתעלם מבקשות scaling נוספות |
| **Instance Warm-up** | לפי הגדרה | ה-instance החדש לא נספר ב-metric עד שהתחמם |
| **Health Check Grace Period** | 300 שניות | ASG לא בודק בריאות של instance טרי |

- המטרה של cooldown: למנוע **thrashing** — הוספה והסרה של instances בלולאה.
- **Simple Scaling** נחסם על ידי cooldown ולכן הוא איטי — זו הסיבה שהוא נחשב legacy.
- **Step Scaling ו-Target Tracking** משתמשים ב-instance warm-up ולא נחסמים על ידי cooldown,
  ולכן הם מגיבים מהר יותר לעומס משתנה.
- **טיפ מהשקפים:** AMI מוכן מראש (golden AMI) מקצר את זמן ההשקה ומאפשר cooldown קצר יותר.
  אם ה-user data מתקין חבילות במשך 5 דקות — כל תגובה ל-עומס מאחרת ב-5 דקות.

### 3.4 Termination Policy — את מי ASG הורג ב-scale in

סדר ברירת המחדל, לפי עדיפות:

1. ה-AZ עם הכי הרבה instances (לשמור על איזון).
2. ה-instance עם ה-Launch Template/Configuration הישן ביותר.
3. ה-instance הקרוב ביותר לשעת חיוב מלאה.
4. אקראי.

- אפשר לסמן instance ב-**Scale-In Protection** כדי שלא ייבחר.
- אפשר להעביר instance ל-**Standby** — הוא נשאר ב-ASG אבל יוצא מה-load balancer ולא נספר כפעיל.

### 3.5 Lifecycle Hooks

Lifecycle hook עוצר את ה-instance במצב המתנה כדי לבצע פעולה לפני שהוא נכנס או יוצא מהשירות.

| Hook | מתי | שימוש טיפוסי |
|---|---|---|
| `Pending:Wait` | לפני שה-instance נכנס ל-InService | התקנה, טעינת cache, רישום ל-config server |
| `Terminating:Wait` | לפני הריגה סופית | שליפת logs, יצירת EBS snapshot, drain של חיבורים |

- שימוש קלאסי מהשקפים: לפני termination יוצרים snapshot של ה-EBS, ואחרי launch מחברים volume חדש ממנו.
- אם ה-hook לא מקבל אישור בזמן — יש timeout שמחליט אם להמשיך או להרוג.

### 3.6 Instance Refresh

- מחליף בהדרגה את כל ה-instances ב-ASG כדי לפרוס AMI או Launch Template חדשים.
- מגדירים **Minimum Healthy Percentage** — כמה מהצי חייב להישאר חי בכל רגע.
- זו הדרך המובנית לבצע rolling deployment בלי כלים חיצוניים.

### 3.7 ASG עם Spot — Mixed Instances Policy

- אפשר להגדיר ASG שמערבב On-Demand ו-Spot באותו צי.
- מגדירים **base capacity** ב-On-Demand (יציבות) ואת כל מה שמעליה כאחוז Spot (חיסכון).
- מגדירים מספר instance types כדי להגדיל את הסיכוי למצוא capacity ב-Spot.
- **Capacity Rebalancing** — ASG מקבל התראה מוקדמת על interruption ומשיק תחליף לפני ההרג.
- דורש אפליקציה **stateless** שסובלת interruption. פירוט מלא ב-[[06 - EC2 Pricing and Optimization]].

### 3.8 Auto Scaling של דברים שאינם EC2

| שירות | מה מתרחב | מנוע |
|---|---|---|
| **ECS Service Auto Scaling** | מספר ה-tasks בשירות | Application Auto Scaling |
| **ECS Capacity Provider** | ה-EC2 instances שמתחת לאשכול | ASG רגיל, מונע על ידי ECS |
| **DynamoDB Auto Scaling** | RCU/WCU | Application Auto Scaling |
| **Aurora Auto Scaling** | מספר ה-Read Replicas | Application Auto Scaling |

**ECS Service Auto Scaling — מה שחשוב:**

- מתרחב לפי `ECS Service Average CPU`, `ECS Service Average Memory` או `ALB RequestCountPerTarget`.
- תומך ב-Target Tracking, Step Scaling ו-Scheduled Scaling — בדיוק כמו EC2 ASG.
- **ECS Service Auto Scaling (רמת task) ≠ EC2 Auto Scaling (רמת instance).**
  ב-EC2 Launch Type צריך את שניהם: tasks נוספים לא יעלו אם אין EC2 פנוי.
- ב-**Fargate** הבעיה נעלמת — אין תשתית לנהל, רק tasks. פירוט ב-[[26 - Containers]].

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **ASG עצמו** | **0** | השירות חינמי לחלוטין |
| EC2 instances | לפי שעה/שנייה × מספר instances פעילים | זה כמעט כל החשבון |
| EBS volumes | GB-month לכל volume מחובר | volume נמחק עם ה-instance רק אם `DeleteOnTermination` דלוק |
| Elastic Load Balancer | שעות LB + יחידות עיבוד | ראו [[08 - Elastic Load Balancing]] |
| Data Transfer | GB יוצא לאינטרנט ו-GB בין AZs | תעבורה בין AZ בתוך VPC מחויבת |
| CloudWatch | metrics מותאמים אישית, alarms, detailed monitoring | basic monitoring (5 דק') חינם |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Fleet קבוע בגודל השיא | **היקר ביותר** | כמעט אף פעם |
| ASG עם Target Tracking | בינונית — משלמים לפי עומס אמיתי | ברירת המחדל הנכונה |
| ASG + Scheduled Scaling | נמוכה יותר — לא ממתינים ל-alarm | עומס בשעות ידועות |
| ASG + Spot במיקס | **הנמוכה ביותר** (Spot עד ~90% הנחה) | workloads stateless, batch, workers |
| ASG + Savings Plans על ה-baseline | נמוכה (עד ~72% הנחה על ה-baseline) | חלק קבוע וידוע של הצי |

### 🚩 עלויות נסתרות

- **Minimum גבוה מדי** — משלמים 24/7 על capacity שנחוצה שעתיים ביום. זו הדליפה מספר 1.
- **Detailed Monitoring** ברזולוציית דקה עולה כסף לכל instance. Basic (5 דקות) חינם — אבל מגיב לאט.
- **Custom CloudWatch metrics ו-alarms** מחויבים לכל metric ולכל alarm.
- **Thrashing** — cooldown קצר מדי גורם ללולאת השקה/הריגה, וכל השקה משלמת מינימום חיוב.
- **EBS יתומים** — אם `DeleteOnTermination=false`, כל instance שנהרג משאיר volume שממשיך לחייב.
- **Cross-AZ data transfer** — ASG רב-AZ מאחורי NLB בלי cross-zone עלול לייצר תעבורה בין AZ בתשלום.
- **AMI כבד עם user data ארוך** — לא עלות ישירה, אבל מאריך את זמן ההגעה ל-InService, ולכן מחייב לשמור capacity עודף.

### 💡 טיפים לחיסכון

- הורידו את ה-**min** לרמת הביקוש הלילי האמיתי, לא לרמת "מה אם".
- השתמשו ב-**Scheduled Scaling** לדפוסים ידועים במקום לחכות ש-alarm יגיב.
- בנו **golden AMI** — instance שעולה ב-30 שניות במקום 5 דקות מוריד את ה-buffer הנדרש.
- **Mixed Instances Policy**: On-Demand ל-baseline, Spot לשיא.
- ודאו ש-**scale-in פעיל** — הרבה צוותים מגדירים רק scale-out ואז הצי לא יורד לעולם.
- כסו את ה-baseline ב-Savings Plan, את ה-peak תשאירו On-Demand/Spot.

---

## 5. ⚖️ השוואות מכריעות

### Scaling Policies זו מול זו

| קריטריון | Target Tracking | Simple Scaling | Step Scaling | Scheduled | Predictive |
|---|---|---|---|---|---|
| מורכבות הגדרה | הכי נמוכה | נמוכה | בינונית | נמוכה | בינונית |
| מבוסס על | ערך יעד ל-metric | alarm בודד | מדרגות alarm | שעון | חיזוי ML |
| תגובה לעוצמת החריגה | אוטומטית | לא — פעולה קבועה | כן — לפי מדרגה | לא רלוונטי | מקדים את החריגה |
| נחסם על ידי cooldown | לא (warm-up) | **כן** | לא (warm-up) | לא רלוונטי | לא |
| מתאים ל | רוב המקרים | legacy | עומס תנודתי חד | עומס צפוי בשעון | עומס מחזורי |

### ASG מול חלופות

| קריטריון | EC2 ASG | ECS Service Auto Scaling | Lambda |
|---|---|---|---|
| מה מתרחב | instances | tasks/containers | concurrency |
| זמן תגובה | דקות (boot) | שניות עד דקה | מיידי |
| ניהול תשתית | אתם | חלקי (Fargate: לא) | אין |
| מודל חיוב | לפי זמן instance | לפי task/vCPU-RAM | לפי הפעלה וזמן ריצה |

> [!info] שורה תחתונה
> ברירת המחדל היא **Target Tracking**, כי היא פשוטה ומדויקת מספיק ל-90% מהמקרים.
> עוברים ל-Step כשצריך תגובה חדה לחריגות גדולות, ל-Scheduled כשהעומס ידוע בשעון,
> ול-Predictive כשזמן ה-boot ארוך והעומס עולה מהר מדי בשביל תגובה reactive.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-Auto Scaling** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | פריסות ותחזוקה בלי downtime ידני | Instance Refresh לפריסת AMI חדש; lifecycle hooks לשליפת logs לפני termination |
| **Security** | ה-instances שנולדים כבר נולדים מאובטחים | IAM Role ב-Launch Template במקום credentials; instances ב-private subnets; SG מצומצם |
| **Reliability** | כשל של instance או של AZ שלם לא מפיל את השירות | פריסה על ≥2 AZs; ELB Health Checks במקום EC2 בלבד; min ≥ 2 |
| **Performance Efficiency** | ה-capacity מתאימה לעומס האמיתי, לא לניחוש | scaling על `RequestCountPerTarget` לאפליקציית web; golden AMI לזמן boot קצר |
| **Cost Optimization** | לא משלמים על capacity לא מנוצלת | Mixed Instances עם Spot; Scheduled Scaling לשפל הלילי; min מכויל למציאות |
| **Sustainability** | פחות שרתים דולקים = פחות חומרה ואנרגיה | scale-in אגרסיבי בשעות שפל; instance types מודרניים (Graviton) בעלי יחס ביצועים/וואט טוב |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "application crashed but instance kept running" | להחליף ל-**ELB Health Checks** ב-ASG |
| "web app behind ALB, scale correctly" | **Target Tracking על RequestCountPerTarget** |
| "traffic spikes every Friday at 5pm" | **Scheduled Scaling** |
| "load increases faster than instances can boot" | **Predictive Scaling** (או golden AMI + warm-up) |
| "instances launch and terminate repeatedly" | **Cooldown / warm-up קצר מדי** |
| "new instances killed before app starts" | **Health Check Grace Period** קצר מדי |
| "run a script before instance is terminated" | **Lifecycle Hook** (`Terminating:Wait`) |
| "deploy a new AMI with no downtime" | **Instance Refresh** |
| "minimize cost for stateless workers" | **Mixed Instances Policy עם Spot** |
| "queue keeps growing, workers idle" | scale על **ApproximateNumberOfMessagesVisible** |
| "highly available across regions" | **ASG לא חוצה Region** — צריך ASG בכל Region + Route 53 |

### טעויות נפוצות

> [!warning] מלכודת 1 — ASG כ-load balancer
> **הניסוח:** "Use an Auto Scaling Group to distribute incoming traffic across instances."
> **הטעות:** ASG לא רואה בקשות ולא מנתב כלום.
> **הנכון:** ELB מנתב, ASG מנהל את מספר ה-instances. שניהם ביחד, כל אחד בתפקידו.

> [!warning] מלכודת 2 — CPU כ-metric אוניברסלי
> **הניסוח:** "The web tier is overwhelmed but CPU stays at 15%."
> **הטעות:** להגדיל את סף ה-CPU או להוריד אותו ל-10%.
> **הנכון:** האפליקציה I/O-bound. ה-metric הנכון הוא `RequestCountPerTarget` או latency.

> [!warning] מלכודת 3 — Desired שנתקע
> **הניסוח:** "We set desired to 10 but the ASG scaled back to 4."
> **הטעות:** לחשוב שיש באג.
> **הנכון:** scaling policy פעילה דורסת שינוי ידני של desired. אם רוצים 10 קבוע — משנים את ה-**min**.

> [!warning] מלכודת 4 — Max נמוך מדי
> **הניסוח:** "Under peak load users get errors although the ASG is 'scaling'."
> **הטעות:** לחפש את הבעיה ב-policy.
> **הנכון:** ה-ASG הגיע ל-**Maximum Capacity** ואינו יכול לגדול. בודקים את התקרה ואת quota ה-EC2 בחשבון.

> [!warning] מלכודת 5 — Launch Configuration
> **הניסוח:** תשובה שמציעה "create a Launch Configuration with the new AMI".
> **הטעות:** לבחור בה כי היא נשמעת מוכרת מהחומר הישן.
> **הנכון:** Launch Configuration הוא deprecated. התשובה היא **Launch Template** (וגרסה חדשה שלו + Instance Refresh).

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

אתר מסחר אלקטרוני. תעבורה יומית משתנה פי 4 בין הלילה לצהריים,
ופעם בחודש יש מבצע מתוזמן שמכפיל את העומס פי 10 תוך 3 דקות.
נדרש: אפס downtime בכשל AZ, עלות מינימלית, ופריסות ללא הפסקת שירות.

```text
                    Route 53
                        ↓
            Application Load Balancer  (public subnets, 2 AZ)
              ↙                        ↘
     AZ-a private subnet          AZ-b private subnet
   ┌────────────────────┐       ┌────────────────────┐
   │  EC2  EC2  EC2     │       │  EC2  EC2  EC2     │
   └────────────────────┘       └────────────────────┘
        └───────  Auto Scaling Group (min 4 / max 40)  ───────┘
                        ↓
              RDS Multi-AZ  (private subnets)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| ASG על **2 AZs לפחות**, min=4 | כשל AZ שלם משאיר 2 instances חיים, ו-ASG משלים את החסר ב-AZ השנייה |
| **ELB Health Check** במקום EC2 בלבד | מזהה אפליקציה שמתה גם כשה-VM חי |
| **Target Tracking על `RequestCountPerTarget`** | מדד עומס אמיתי לאפליקציית web; CPU היה מפספס |
| **Scheduled Scaling** לפני המבצע החודשי | מרימים min ל-20 עשר דקות לפני — לא מחכים ל-alarm |
| **Golden AMI** מוכן מראש | זמן boot של 40 שניות במקום 5 דקות; מאפשר cooldown קצר |
| **Mixed Instances**: On-Demand base=8, השאר Spot | ה-baseline יציב, השיא זול |
| **Instance Refresh** לפריסות | rolling deploy עם minimum healthy 90%, בלי downtime |
| instances ב-**private subnets** | אין IP ציבורי; רק ה-ALB חשוף — ראו [[11 - VPC Security]] |

**למה לא Scheduled Scaling בלבד?**
כי המבצע ידוע מראש אבל ה"וויראליות" שלו לא. Scheduled נותן את ה-baseline, Target Tracking קולט את ההפתעות.

**למה לא להשאיר 40 instances קבוע?**
זה פותר את הביצועים ומפוצץ את התקציב פי ~10 — בדיוק מה שהדרישה "עלות מינימלית" פוסלת.

**למה לא Lambda?**
אפליקציית web מונוליטית עם session ו-runtime ארוך; המעבר ל-serverless הוא שכתוב, לא scaling. ראו [[25 - Lambda]].

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **תחביר CLI/CloudFormation מדויק** של `put-scaling-policy` — לא נשאלים על syntax.
- **ערכי ה-step adjustments המדויקים** בדוגמאות. מספיק להבין את העיקרון.
- **אלגוריתם ה-ML של Predictive Scaling** — מספיק לדעת מה הוא עושה ומתי בוחרים בו.
- **רשימת כל ה-suspended processes** (`AZRebalance`, `AlarmNotification` וכו'). מספיק לדעת שאפשר להשעות תהליכים.
- **Warm Pools** — פיצ'ר קיים ושימושי, אבל כמעט לא מופיע ב-SAA-C03.
- **מגבלות quota מספריות** על מספר ASGs לחשבון — משתנות וניתנות להגדלה.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **ASG עצמו חינם** — משלמים רק על ה-EC2, EBS, ELB ו-data transfer שמתחתיו.
- **Launch Template** בלבד. Launch Configuration הוא deprecated.
- **min / desired / max** — scaling policy משנה את ה-desired, לא את ה-min.
- **ASG ≠ Load Balancer.** ASG מנהל צי, ELB מנתב תעבורה.
- **ELB Health Check** תופס אפליקציה מתה; **EC2 Status Check** תופס רק VM מת.
- **Health Check Grace Period** ברירת מחדל **300 שניות**.
- **Cooldown** ברירת מחדל **300 שניות** — חוסם Simple Scaling, לא חוסם Target/Step.
- **Target Tracking** = ברירת המחדל; **Step** = תגובה מדורגת; **Scheduled** = שעון; **Predictive** = חיזוי מראש.
- **RequestCountPerTarget** הוא ה-metric לאפליקציית web מאחורי ALB; CPU רק ל-CPU-bound.
- **Termination:** AZ עמוס ביותר → Launch Template ישן ביותר → קרוב לשעת חיוב → אקראי.
- **Lifecycle Hooks** = פעולה לפני כניסה לשירות או לפני termination.
- **Instance Refresh** = rolling deploy של AMI חדש בלי downtime.
- **ASG חי ב-Region אחד**; פיזור בין AZs — כן, בין Regions — לא.
- **ECS Service Auto Scaling (tasks) ≠ EC2 ASG (instances)** — ב-EC2 launch type צריך את שניהם.

---

## 11. ✅ בדיקת הבנה

1. ה-instance מגיב ל-ping וה-EC2 status checks ירוקים, אבל ה-Java process קרס וכל בקשה מחזירה 502. ASG לא מחליף אותו. למה, ומה מתקנים?
2. הגדרתם desired=12 ידנית בבוקר, ואחרי חצי שעה הצי חזר ל-5. מה קרה ומה הפתרון הנכון?
3. אפליקציית web מאחורי ALB "נחנקת" אבל ה-CPU עומד על 12%. איזה scaling metric תבחרו ולמה?
4. ה-ASG משיק ומכבה instances כל שתי דקות בלולאה. מה שני ההסברים הסבירים?
5. צריך לפרוס AMI חדש ל-30 instances בלי downtime. מה הכלי המובנה?
6. יש עומס מחזורי שמכפיל את התעבורה כל בוקר ב-08:00, וזמן ה-boot של ה-instance הוא 6 דקות. איזו policy תבחרו?
7. הצי לא גדל מעבר ל-20 instances למרות ש-CPU ב-95%. מה בודקים ראשון?

<details>
<summary>תשובות</summary>

1. ה-ASG מוגדר עם **EC2 Status Checks** בלבד, שרואים רק את בריאות ה-VM. מפעילים **ELB Health Check** ב-ASG כדי שה-health path האפליקטיבי יקבע.
2. **Scaling policy פעילה דרסה את ה-desired.** שינוי ידני של desired הוא חד-פעמי. אם רוצים רצפה גבוהה יותר — מעלים את ה-**min** (או מוסיפים Scheduled Scaling).
3. **`RequestCountPerTarget`** — האפליקציה I/O-bound, ה-CPU אינו מדד העומס שלה. ה-metric מגיע מה-ALB ומודד בקשות לכל target.
4. (א) **Cooldown / instance warm-up קצר מדי** — ה-metric לא הספיק להתייצב לפני החלטת ה-scaling הבאה. (ב) **Health Check Grace Period קצר מדי** — ה-instances נהרגים לפני שהאפליקציה עולה.
5. **Instance Refresh** עם Minimum Healthy Percentage — מחליף בהדרגה, בלי כלים חיצוניים.
6. **Predictive Scaling** (או Scheduled Scaling אם השעה קבועה לחלוטין). Reactive scaling לא יספיק — עד שה-alarm יורה וה-instance יעלה, 6 דקות של שגיאות כבר קרו.
7. את ה-**Maximum Capacity** של ה-ASG — קרוב לוודאי שהוא 20. אחריו בודקים את **service quota** של EC2 בחשבון ואת זמינות ה-instance type ב-AZ.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[05 - EC2 Fundamentals]] · [[06 - EC2 Pricing and Optimization]] · [[08 - Elastic Load Balancing]] · [[26 - Containers]] · [[31 - Monitoring and Logging]] · [[33 - High Availability and Scalability]] · [[37 - Cost Optimization]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 2389–2570, 7704–7756
