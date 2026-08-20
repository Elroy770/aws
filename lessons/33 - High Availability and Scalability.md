---
lesson: 33
title: High Availability and Scalability
domain: Design Resilient Architectures
services: [EC2, Auto Scaling, ELB, RDS, Aurora, ElastiCache, EFS, Route 53]
tags: [saa-c03, resilience, scalability, multi-az, architecture]
---

# 33 — High Availability and Scalability

> [!abstract] בשורה אחת
> Scalability = לטפל ביותר עומס. High Availability = לשרוד נפילה של Data Center. הם קשורים אבל **לא אותו דבר**, וזה בדיוק מה שהמבחן בודק.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה "שרת גדול יותר" זה לא HA |
| 2 | איך זה עובד | Vertical מול Horizontal, HA passive מול active, Cross-Zone LB |
| 3 | פירוק מפורט | המסע הקלאסי: שרת בודד → ELB → ASG → Multi-AZ; טיפול ב-state |
| 4 | עלות | על מה משלמים בעודפות, cross-AZ transfer, reserved minimum |
| 5 | השוואות | Scalability / Elasticity / HA / Fault Tolerance — ארבעת המונחים |
| 6 | Well-Architected | |
| 7 | מלכודות | Read Replica ≠ Multi-AZ, שני instances באותו AZ |
| 8 | Scenario | EC2 יחיד "זמין גבוה" עם Elastic IP ו-EBS |

**מונחי מפתח בשיעור:** `Scale up/down` · `Scale out/in` · `Multi-AZ` · `Stateless` · `Cross-Zone Load Balancing` · `Sticky Session` · `Lifecycle Hook`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- האתר קורס בשעות שיא. מגדילים את השרת — וזה מחזיק עוד חודש.
- הגדלת השרת דורשת **downtime** (עצירה, שינוי type, הפעלה).
- יש תקרה פיזית: אי אפשר להגדיל instance לנצח.
- שני שרתים באותו AZ — ה-AZ נופל, שניהם נופלים.
- הוספנו שרת שני, ועגלת הקניות של המשתמש נעלמה כשהוא נחת על השרת האחר.

### מה השירות פותר

- **Vertical Scaling** — instance גדול יותר. פשוט, אבל מוגבל וכרוך ב-downtime.
- **Horizontal Scaling** — עוד instances. דורש מערכת מבוזרת, אבל אין תקרה.
- **High Availability** — פריסה על **לפחות 2 AZs**, כדי לשרוד אובדן data center.
- **Auto Scaling + ELB** — מה שהופך את הרעיונות האלה לאוטומטיים ולריפוי-עצמי.

> [!tip] האנלוגיה — מוקד טלפוני
> **Vertical:** מחליפים נציג ג'וניור בנציג בכיר שעונה מהר יותר. יש גבול לכמה מהר אדם אחד יכול לדבר.
> **Horizontal:** מגייסים עוד שישה נציגים. אין גבול תיאורטי — אבל צריך מרכזייה שתחלק שיחות.
> **High Availability:** בניין אחד בניו יורק ובניין שני בסן פרנסיסקו. רעידת אדמה באחד — השירות ממשיך.

---

## 2. ⚙️ איך זה עובד

### 2.1 Vertical Scalability — scale up / scale down

- הגדלת **גודל ה-instance**: מ-`t2.micro` ל-`t2.large`, וב-EC2 עד `u-12tb1.metal` (12.3 TB RAM, 448 vCPU).
- נפוץ במיוחד ב-**מערכות לא-מבוזרות** — קודם כל **databases**.
- **RDS** ו-**ElastiCache** הם דוגמאות קלאסיות לשירותים שמתרחבים vertically.
- **יש תקרת חומרה.** מגיעים אליה, ואז חייבים לשנות ארכיטקטורה.
- כרוך ב-downtime (או ב-failover ב-Multi-AZ, שגם הוא רגע של ניתוק).

### 2.2 Horizontal Scalability — scale out / scale in

- הגדלת **מספר ה-instances**.
- **מחייב מערכת מבוזרת** — ולכן מחייב שהאפליקציה תהיה **stateless**.
- זה מה שאפליקציות web מודרניות עושות, וזה מה שהענן הופך לזול ומיידי.
- הכלים: **Auto Scaling Group** + **Load Balancer**.
- בקורס הזה **Horizontal Scalability מזוהה עם Elasticity** — היכולת להוסיף ולהסיר בהתאם לביקוש.

### 2.3 High Availability

- HA הולך **יד ביד** עם horizontal scaling, אבל זה לא אותו דבר.
- ההגדרה המעשית: להריץ את האפליקציה **בלפחות שני data centers = שני AZs**.
- המטרה: **לשרוד אובדן של data center שלם**.
- שני סוגים:

| סוג | מה קורה בכשל | דוגמה |
|---|---|---|
| **Passive HA** | יש standby שממתין; מתבצע failover | **RDS Multi-AZ** — standby ב-AZ אחר, מקודם אוטומטית |
| **Active HA** | כל ה-nodes מקבלים תעבורה כל הזמן | **ASG + ELB** על פני כמה AZs |

### 2.4 HA ו-Scalability עבור EC2 — הסיכום

| מונח | ב-EC2 | הכלים |
|---|---|---|
| **Vertical Scaling** | הגדלת גודל ה-instance (scale up/down) | שינוי instance type |
| **Horizontal Scaling** | הגדלת מספר ה-instances (scale out/in) | **Auto Scaling Group + Load Balancer** |
| **High Availability** | אותה אפליקציה **בכמה AZs** | **ASG multi-AZ + Load Balancer multi-AZ** |

> [!warning] הנקודה הקריטית
> ASG שמוגדר על **AZ אחד** נותן scalability — **לא** high availability.
> HA מתחיל ברגע שה-ASG וה-ELB פרוסים על **2 AZs לפחות**.

### 2.5 Cross-Zone Load Balancing

מה זה עושה: קובע אם node של ה-Load Balancer ב-AZ מסוים מפזר **רק ל-targets ב-AZ שלו**, או **לכל ה-targets בכל ה-AZs**.

```text
בלי Cross-Zone (50/50 בין ה-nodes):
  AZ1 node (50%) → 2 instances  → כל אחד מקבל 25%
  AZ2 node (50%) → 8 instances  → כל אחד מקבל 6.25%   ← חלוקה לא הוגנת!

עם Cross-Zone:
  כל ה-10 instances מקבלים 10% כל אחד            ← חלוקה אחידה
```

| Load Balancer | ברירת מחדל | חיוב על תעבורה בין AZs |
|---|---|---|
| **ALB** | **מופעל** (ניתן לכבות ברמת ה-Target Group) | **ללא חיוב** |
| **NLB** / **GWLB** | **כבוי** | **בתשלום** אם מפעילים |
| **CLB** | **כבוי** | **ללא חיוב** אם מפעילים |

זו שאלה שנשאלת ישירות: "התעבורה מתחלקת לא באופן שווה בין AZs" → Cross-Zone Load Balancing.

---

## 3. 🔍 פירוק מפורט — המסע הקלאסי של ארכיטקט

### 3.1 שלב א' — אפליקציה Stateless (אתר שמחזיר את השעה)

הדרישות: אין DB, מתחילים קטן, מוכנים ל-downtime בהתחלה, ובסוף רוצים scale מלא ללא downtime.

| שלב | מה עושים | מה הבעיה שנוצרת |
|---|---|---|
| 1 | EC2 יחיד עם **Elastic IP** | נקודת כשל יחידה |
| 2 | **Scaling vertically** — מעבר ל-M5 | **downtime** בזמן השדרוג; יש תקרה |
| 3 | **Scaling horizontally** — עוד instances, כל אחד עם Elastic IP | Elastic IPs מוגבלים במספר; ניהול ידני |
| 4 | **Route 53 A Record עם TTL של שעה** לכל ה-IPs | כש-instance נעלם, לקוחות עדיין מקבלים את ה-IP המת למשך שעה |
| 5 | **ELB + Health Checks**, ה-instances עוברים ל-**subnets פרטיים** | Route 53 מצביע ב-**Alias Record** ל-ELB; הכשל מוסתר מהלקוח |
| 6 | **Auto Scaling Group** | ה-capacity מתאים את עצמו; instance כושל מוחלף אוטומטית |
| 7 | **Multi-AZ** — ASG ו-ELB על AZ 1–3 | עכשיו זה גם HA וגם scalable |
| 8 | **Reserved Instances על ה-minimum capacity** | ה-baseline תמיד רץ — אין סיבה לשלם עליו On-Demand |

**Security Groups בשרשרת:** ה-ELB פתוח ל-`0.0.0.0/0` ב-80/443; ה-SG של ה-EC2 מתיר תעבורה **רק מ-SG של ה-ELB**.

### 3.2 שלב ב' — אפליקציה Stateful (חנות בגדים עם עגלת קניות)

הבעיה: יש state (עגלה) שחייב לשרוד מעבר בין instances.

| פתרון | איך עובד | חסרונות |
|---|---|---|
| **ELB Stickiness** (Session Affinity) | ה-LB מצמיד משתמש ל-instance קבוע דרך cookie | פוגע בפיזור העומס; אם ה-instance מת — ה-session אבד |
| **User Cookies** — העגלה נשמרת אצל הלקוח | האפליקציה נשארת **stateless** לגמרי | בקשות HTTP **כבדות יותר**; **סיכון אבטחה** — cookie ניתן לזיוף וחייב validation; **cookie מוגבל ל-4KB** |
| **Server Session** — cookie מכיל רק `session_id` | הדאטה נשמר ב-**ElastiCache** (או **DynamoDB** כחלופה) | הפתרון המומלץ; מוסיף רכיב לתחזוקה |

- **דאטה של המשתמש** (שם, כתובת) → **RDS**.
- **Scaling reads** → **RDS Read Replicas**, או **ElastiCache** בדפוס **Lazy Loading** (קוראים מה-cache; miss → RDS → כותבים ל-cache).
- **Multi-AZ** לכולם: ASG, ElastiCache ו-RDS.
- Security Groups מקוננים: LB → EC2 → RDS ו-EC2 → ElastiCache.

זה בדיוק ה-**3-tier architecture** שהמבחן אוהב.

### 3.3 שלב ג' — WordPress: איפה שומרים קבצים

| אחסון | מתאים ל | הבעיה |
|---|---|---|
| **EBS** | **instance יחיד** | Volume מוצמד ל-AZ אחד ול-instance אחד. עם 2 instances — התמונה שהועלתה ל-AZ1 לא נראית ב-AZ2 |
| **EFS** | **אפליקציה מבוזרת** | file system משותף עם **ENI בכל AZ** — כל ה-instances רואים את אותם קבצים |
| **S3** (הפתרון המודרני) | תמונות ו-static assets | לא נדרש file system בכלל; זול ועמיד יותר |

- שכבת ה-DB: **Aurora MySQL** נותנת Multi-AZ ו-Read Replicas בקלות רבה יותר מ-RDS.

> [!info] הכלל
> **EBS = instance יחיד. EFS = הרבה instances באותה אפליקציה.** זו שאלת מבחן ישירה על WordPress ועל CMS.

### 3.4 שלב ד' — "HA" ל-instance יחיד שאי אפשר לפזר

לפעמים יש אפליקציה legacy שרצה על instance בודד עם Elastic IP. איך בכל זאת מוסיפים חוסן?

**גישה 1 — Standby ידני:**

```text
CloudWatch Alarm על ה-instance → מפעיל instance standby → מצמיד את ה-Elastic IP אליו
```

**גישה 2 — ASG כמנגנון ריפוי (הדפוס המומלץ):**

```text
ASG: min=1, max=1, desired=1, פרוס על >= 2 AZs
EC2 User Data: בעלייה — מצמיד את ה-Elastic IP לעצמו
EC2 Instance Role: הרשאה לקרוא ל-API של הצמדת Elastic IP
```

ה-instance מת → ה-ASG מקים אחד חדש (אולי ב-AZ אחר) → ה-User Data מצמיד אליו את אותו Elastic IP. הכתובת הציבורית לא משתנה.

**גישה 3 — ASG + EBS עם Lifecycle Hooks** (כשיש דאטה מקומי לשמר):

```text
ASG Terminate lifecycle hook → יוצר EBS Snapshot + tags
ASG Launch  lifecycle hook  → יוצר Volume מה-Snapshot (לפי tag) ומצמיד ל-instance החדש
User Data                   → מצמיד את ה-Elastic IP
```

זה הדפוס שמופיע בשאלות "כיצד לשמר את הדאטה כשה-ASG מחליף instance".

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| Compute עודף | לכל instance-hour | HA = לפחות 2 instances גם בעומס אפס |
| Load Balancer | שעה + **LCU/NLCU** | חיוב קבוע גם בלי תעבורה |
| **Cross-AZ Data Transfer** | לכל GB בשני הכיוונים | **חינם ב-ALB Cross-Zone; בתשלום ב-NLB** |
| RDS Multi-AZ | **כפול** מ-Single-AZ | ה-standby לא מגיש תעבורה |
| Read Replicas | לכל replica כ-instance מלא | + replication traffic |
| ElastiCache Multi-AZ | לכל node | |
| EFS | לכל GB + throughput mode | יקר מ-EBS ל-GB |
| Elastic IP | חינם כשמוצמד ל-instance פעיל | **בתשלום כשלא מוצמד** |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Single-AZ, instance אחד | הזול ביותר | dev/test בלבד |
| Scale up (instance גדול אחד) | לרוב יקר יותר לאותו כוח חישוב | DB, מערכת שלא ניתן לבזר |
| Scale out (הרבה instances קטנים) | גמיש וזול יותר | אפליקציות stateless; מאפשר **Spot** |
| RDS Multi-AZ | **פי ~2** | פרודקשן — זו הדרישה |
| Read Replica | תוספת לכל replica | scaling של reads, לא HA |
| Active-Active Multi-Region | **היקר ביותר** | דרישות RTO/RPO קיצוניות בלבד |

### 🚩 עלויות נסתרות

- **Cross-AZ traffic** בין שכבות: EC2 ב-AZ1 שקורא ל-DB ב-AZ2 משלם לכל GB, בכל בקשה, לנצח.
- **NLB עם Cross-Zone מופעל** — מחייב על תעבורה בין AZs. ALB לא.
- **Elastic IP לא מוצמד** — עולה כסף על כלום.
- **ASG שלא מבצע scale-in** בגלל cooldown שגוי או health check שלא מתייצב.
- **Over-provisioning "ליתר ביטחון"** — minimum capacity שהוגדר גבוה מדי ורץ 24/7.
- **EFS** לאחסון שיכול היה להיות S3.

### 💡 טיפים לחיסכון

- **Reserved Instances / Savings Plans** על ה-**minimum capacity** של ה-ASG — זה ה-baseline שתמיד רץ.
- **Spot Instances** לשכבה ה-stateless שמעל ה-baseline (עד ~90% הנחה).
- לשמור תעבורה **בתוך אותו AZ** כשאפשר — ולפרוס בין AZs רק את מה שחייב.
- **Target Tracking** במקום סף קבוע — מונע over-provisioning.
- **Scheduled Scaling** לעומס צפוי (שעות עבודה), במקום להחזיק capacity מלא כל היום.
- לוודא ש-scale-in באמת קורה — רוב ה-over-spend הוא capacity שלא ירד.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 ארבעת המונחים שמתבלבלים — הטבלה החשובה ביותר בשיעור

| קריטריון | **Scalability** | **Elasticity** | **High Availability** | **Fault Tolerance** |
|---|---|---|---|---|
| ההגדרה | היכולת לטפל בעומס **גדול יותר** | להוסיף **ולהסיר** capacity **אוטומטית** לפי ביקוש | לשרוד **כשל של רכיב/AZ** עם הפרעה מינימלית | להמשיך לפעול **ללא הפרעה כלל** |
| השאלה | "האם נעמוד ב-10x תנועה?" | "האם נשלם רק על מה שצריך עכשיו?" | "האם AZ שנפל מפיל אותנו?" | "האם המשתמש בכלל שם לב?" |
| הכלי הטיפוסי | instance גדול יותר, או עוד instances | **Auto Scaling Group** | פריסה על **≥2 AZs**, RDS Multi-AZ | עודפות מלאה N+1/N+2, Multi-Region active-active |
| האם דורש כמה AZs? | **לא** | לא בהכרח | **כן** | כן, ולרוב גם כמה Regions |
| הפרעה מורגשת? | לא רלוונטי | לא רלוונטי | **קצרה** (failover) | **אפס** |
| עלות יחסית | תלוי | חוסכת כסף | תוספת משמעותית | **הגבוהה ביותר** |
| דוגמה | RDS מ-`db.t3` ל-`db.r5` | ASG שגדל מ-2 ל-20 בשעות שיא | ALB + ASG על 3 AZs | Route 53 active-active בין 2 Regions |

> [!info] שורה תחתונה
> Scalability עונה על **גודל**. Elasticity עונה על **התאמה אוטומטית**. HA עונה על **כשל**. Fault Tolerance היא HA בלי אפילו שנייה של הפרעה — ובמחיר בהתאם. אם השאלה אומרת "survive an AZ failure" זו **HA**, לא scalability.

### 5.2 Vertical מול Horizontal

| קריטריון | **Vertical (scale up/down)** | **Horizontal (scale out/in)** |
|---|---|---|
| מה משתנה | **גודל** ה-instance | **מספר** ה-instances |
| מתאים ל | מערכות **לא מבוזרות** — בעיקר DBs | אפליקציות **מבוזרות / stateless** |
| שירותי AWS טיפוסיים | **RDS**, **ElastiCache** (וגם RDS Read Replicas כתוספת reads) | **EC2 + ASG**, ECS, Lambda |
| תקרה | **כן — מגבלת חומרה** | פרקטית — כמעט אין |
| Downtime | לרוב **כן** (restart / failover) | **לא** |
| נותן HA? | **לא בפני עצמו** | כן, **אם** פרוס על כמה AZs |
| דרישה מהאפליקציה | אין | **stateless** או ניהול state חיצוני |

> [!info] שורה תחתונה
> DB → מתחילים vertical, ומוסיפים Read Replicas ל-reads. Web tier → תמיד horizontal עם ASG. אם השאלה כוללת "without downtime" ו-"handle unpredictable traffic" — זו תשובה horizontal.

### 5.3 HA מול DR

| קריטריון | **High Availability** | **Disaster Recovery** |
|---|---|---|
| מפני מה מגן | כשל **רכיב או AZ** | אובדן **Region** או אירוע רחב |
| היקף | בתוך Region | בין Regions |
| זמן תגובה | שניות-דקות, אוטומטי | לפי **RTO** שהוגדר |
| מדדים | uptime % | **RTO / RPO** |
| הפירוט | השיעור הזה | [[34 - Disaster Recovery]] |

### 5.4 Read Replica מול Multi-AZ

| | **Read Replica** | **Multi-AZ Standby** |
|---|---|---|
| מטרה | **scaling של reads** | **זמינות** |
| מקבל תעבורה? | **כן** — endpoint נפרד לקריאה | **לא** — יושב פסיבי |
| Replication | **אסינכרוני** | **סינכרוני** |
| Failover אוטומטי | לא (אפשר לקדם ידנית) | **כן** |
| בין AZs/Regions | אפשר גם cross-Region | AZ אחר באותו Region |

פירוט מלא ב-[[22 - RDS Scaling and Availability]].

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | להריץ תרגילי כשל, לא להניח | Game Day: להרוג AZ ב-staging; לוודא ש-lifecycle hooks באמת יוצרים snapshot |
| Security | עודפות לא מרחיבה את משטח התקיפה | Security Groups מקוננים (LB→EC2→RDS); instances ב-**private subnets** מאחורי ה-ELB; Instance Role במקום keys ב-User Data |
| Reliability | לפחות שני AZs, health checks אמיתיים | ELB Health Check על endpoint שבודק את ה-DB, לא רק TCP; ASG על ≥2 AZs; RDS Multi-AZ |
| Performance Efficiency | להוציא state מה-compute כדי לאפשר scale out | Session ב-ElastiCache/DynamoDB; Read Replicas + Lazy Loading; EFS במקום EBS לאפליקציה מבוזרת |
| Cost Optimization | לשלם על baseline בהתחייבות, על השיא לפי שימוש | RI/Savings Plans על ה-minimum של ה-ASG; Spot מעליו; scale-in שבאמת עובד |
| Sustainability | capacity שיושב סרק הוא בזבוז אנרגיה | Target Tracking + Scheduled Scaling; instance types מודרניים (Graviton); כיבוי סביבות dev בלילה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "survive an AZ failure" | **Multi-AZ** — ASG + ELB על ≥2 AZs |
| "handle unpredictable traffic automatically" | **Auto Scaling Group** (elasticity) |
| "no downtime while scaling" | **Horizontal** scaling, לא vertical |
| "users lose their shopping cart" | להוציא session ל-**ElastiCache / DynamoDB** |
| "traffic unevenly distributed across AZs" | **Cross-Zone Load Balancing** |
| "shared files across many instances" | **EFS** (או S3), לא EBS |
| "database read performance" | **Read Replicas** או **ElastiCache** |
| "keep the same public IP after replacement" | **Elastic IP + ASG(min=1,max=1) + User Data** להצמדה |
| "preserve data when ASG replaces an instance" | **Lifecycle Hooks** — snapshot ב-terminate, volume ב-launch |
| "minimum capacity always running, reduce cost" | **Reserved Instances / Savings Plans** על ה-baseline |
| "survive a Region failure" | זה כבר **DR** — [[34 - Disaster Recovery]] |

### טעויות נפוצות

> [!warning] מלכודת — "instance גדול יותר = זמינות גבוהה"
> **הניסוח:** "האתר נפל בשעות שיא. מה יפתור גם עומס וגם זמינות?"
> **הטעות:** לשדרג ל-instance גדול יותר.
> **הנכון:** vertical scaling נותן **capacity בלבד**, ובדרך כלל עם downtime ועם תקרת חומרה. HA דורש **מספר instances בכמה AZs מאחורי ELB**.

> [!warning] מלכודת — שני instances באותו AZ
> **הניסוח:** "יש לנו שני שרתים מאחורי ALB. האם אנחנו מוגנים?"
> **הטעות:** לענות כן.
> **הנכון:** אם שניהם ב-**אותו AZ** — נפילת ה-AZ מפילה את שניהם. **redundancy ≠ HA**. חייבים פיזור בין AZs.

> [!warning] מלכודת — Read Replica כפתרון זמינות
> **הניסוח:** "ה-DB נפל, ורוצים failover אוטומטי."
> **הטעות:** להוסיף Read Replica.
> **הנכון:** Read Replica הוא **scaling של קריאות** עם רפליקציה **אסינכרונית** ובלי failover אוטומטי. זמינות = **Multi-AZ** עם standby סינכרוני.

> [!warning] מלכודת — ELB הופך אפליקציה ל-stateless
> **הניסוח:** "הוספנו ALB, למה משתמשים עדיין מאבדים session?"
> **הטעות:** להניח שה-LB פותר את זה.
> **הנכון:** ELB מפזר בקשות — הוא לא מזיז state. הפתרונות: **Stickiness** (חלקי, מזיק לפיזור), **cookies אצל הלקוח** (עד 4KB, דורש validation), או **session store חיצוני** — וזו התשובה הנכונה.

> [!warning] מלכודת — Multi-AZ כהגנה מפני אובדן Region
> **הניסוח:** "כל ה-Region לא זמין."
> **הטעות:** RDS Multi-AZ.
> **הנכון:** Multi-AZ הוא **בתוך Region**. הגנה מפני אובדן Region = **Cross-Region Read Replica**, backups מועתקים ל-Region אחר, או Aurora Global — כלומר **DR**.

> [!warning] מלכודת — EBS משותף
> **הניסוח:** "WordPress על שני instances, תמונות שהועלו לא מופיעות."
> **הטעות:** להוסיף עוד EBS volume.
> **הנכון:** **EBS מוצמד ל-AZ אחד ול-instance אחד**. אחסון משותף = **EFS**, ולתמונות עדיף **S3**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** חנות אונליין. תנועה בלתי צפויה (מבצעים), עגלת קניות שלא נעלמת, קריאות DB כבדות בדף המוצר, העלאת תמונות ע"י מוכרים, ועמידות בפני נפילת AZ.

```text
                Route 53 (Alias Record)
                          ↓
              ALB (Multi-AZ, Cross-Zone on)
                          ↓
        ┌──────── Auto Scaling Group ────────┐
        AZ-a: EC2         AZ-b: EC2         AZ-c: EC2      (private subnets)
        └──────────┬────────────┬────────────┘
                   ↓            ↓                ↓
        ElastiCache        RDS/Aurora          EFS / S3
        (sessions +      (Multi-AZ writer      (תמונות
         lazy-load cache) + Read Replicas)      של מוכרים)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| ASG על 3 AZs, min=2 | HA אמיתי; גם אם AZ שלם נופל, נשארת capacity |
| ALB עם Cross-Zone (ברירת מחדל, ללא חיוב) | פיזור אחיד גם כשמספר ה-instances שונה בין AZs |
| Session ב-**ElastiCache**, לא Stickiness | Stickiness מרכז עומס ומאבד session כשה-instance מת |
| **Read Replicas** + **Lazy Loading** ב-ElastiCache | מוריד עומס קריאה מה-writer בלי לשנות את שכבת הכתיבה |
| **RDS/Aurora Multi-AZ** | standby סינכרוני עם failover אוטומטי — זמינות, לא ביצועים |
| תמונות ב-**S3** (או EFS ל-legacy) | EBS לא משותף בין AZs; S3 עמיד וזול יותר |
| **RI/Savings Plans** על min=2, **Spot** מעל ה-baseline | ה-baseline קבוע ולכן מתאים להתחייבות; השיא זמני ולכן מתאים ל-Spot |
| Security Groups מקוננים | ה-EC2 מקבל תעבורה רק מה-SG של ה-ALB; ה-RDS רק מה-SG של ה-EC2 |

**למה לא instance אחד ענק?** יש תקרת חומרה, יש downtime בשדרוג, ובעיקר — **AZ אחד שנופל מפיל את כל האתר**.

**למה לא Multi-Region מההתחלה?** זה כבר DR, בעלות כפולה ובמורכבות של רפליקציה ו-DNS failover. פותרים אותו רק אם הדרישה היא **שרידות Region** — ראו [[34 - Disaster Recovery]].

---

## 9. 🚫 מה לא צריך לדעת למבחן

- אחוזי SLA מדויקים של כל שירות.
- שמות מלאים של instance types וגדלים (חוץ מהעיקרון שיש תקרה).
- הנוסחאות המדויקות של LCU/NLCU.
- אלגוריתמי הפיזור הפנימיים של ה-Load Balancer.
- פרטי הקונפיגורציה של lifecycle hooks — רק מה הם מאפשרים.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **Vertical = גודל (scale up/down). Horizontal = כמות (scale out/in).**
- **Vertical** מתאים ל-**RDS ו-ElastiCache**; יש **תקרת חומרה** ולרוב downtime.
- **Horizontal** דורש מערכת מבוזרת ואפליקציה **stateless**; הכלים = **ASG + ELB**.
- **HA = לפחות 2 AZs.** ASG על AZ אחד הוא scalability בלבד.
- HA **passive** = RDS Multi-AZ. HA **active** = ASG + ELB.
- **Scalability ≠ Elasticity ≠ HA ≠ Fault Tolerance** — ארבעה מונחים, ארבע שאלות שונות.
- **Cross-Zone LB:** ALB **מופעל** וללא חיוב; **NLB/GWLB כבוי** ובתשלום; CLB כבוי אך ללא חיוב.
- **Read Replica = reads. Multi-AZ = availability.** לא להחליף ביניהם.
- **EBS = instance יחיד ב-AZ יחיד. EFS = הרבה instances בכמה AZs. S3 = static assets.**
- Session: **ElastiCache או DynamoDB**; Stickiness רק כשאין ברירה; cookie אצל הלקוח **עד 4KB** ודורש validation.
- **Multi-AZ ≠ Multi-Region.** אובדן Region זה DR.
- **Elastic IP + ASG(1,1,1) + User Data** = "HA" ל-instance בודד ששומר על אותה כתובת.
- **Lifecycle Hooks** = snapshot ב-terminate, volume חדש ב-launch — שימור דאטה בהחלפת instance.
- **RI/Savings Plans על ה-minimum capacity**, Spot מעליו — זה דפוס החיסכון הקלאסי.
- ELB Health Check הוא מה שמסתיר instance כושל מהלקוח.

---

## 11. ✅ בדיקת הבנה

1. יש שני EC2 מאחורי ALB, שניהם ב-`us-east-1a`. מה זה נותן, ומה זה **לא** נותן?
2. אפליקציה עם עגלת קניות עוברת מ-instance אחד לשלושה, ומשתמשים מתלוננים שהעגלה נעלמת. שלוש חלופות והחיסרון של כל אחת.
3. מדוע ב-NLB התעבורה עלולה להתחלק לא באופן שווה בין AZs, ומה ההשלכה הכספית של התיקון?
4. אפליקציה legacy חייבת לרוץ על instance יחיד עם כתובת IP ציבורית קבועה, ורוצים החלפה אוטומטית בכשל. איך?
5. מה ההבדל בין Elasticity ל-High Availability במשפט אחד לכל אחד?
6. WordPress על ASG עם 3 instances. מוכרים מעלים תמונות ומשתמשים רואים "תמונה חסרה" לסירוגין. למה, ומה הפתרון?

<details>
<summary>תשובות</summary>

1. זה נותן **capacity וריפוי מכשל של instance בודד** (ה-ALB יוציא כושל מה-rotation). זה **לא** נותן **High Availability** — נפילת `us-east-1a` מפילה את שניהם. HA דורש פיזור על **לפחות 2 AZs**.
2. (א) **ELB Stickiness** — פשוט, אבל פוגע בפיזור העומס וה-session אובד כשה-instance מת. (ב) **Cookies אצל הלקוח** — האפליקציה נשארת stateless לגמרי, אבל הבקשות כבדות יותר, ה-cookie מוגבל ל-**4KB** וניתן לזיוף ולכן חייב validation. (ג) **Session store חיצוני** ב-**ElastiCache** או **DynamoDB** עם `session_id` ב-cookie — הפתרון המומלץ, במחיר רכיב נוסף.
3. ב-**NLB** ה-Cross-Zone Load Balancing **כבוי by default**, ולכן כל node מפזר רק ל-targets ב-AZ שלו; אם מספר ה-instances שונה בין AZs, החלוקה לא אחידה. הפעלת Cross-Zone מתקנת את זה — אבל ב-NLB **גובים תשלום על תעבורה בין AZs** (ב-ALB זה מופעל בברירת מחדל וללא חיוב).
4. **Auto Scaling Group עם min=1, max=1, desired=1** הפרוס על **לפחות 2 AZs**, יחד עם **Elastic IP**. ב-**User Data** ה-instance מצמיד לעצמו את ה-Elastic IP, וה-**Instance Role** נותן לו הרשאה לקרוא ל-API. אם צריך לשמר דאטה — מוסיפים **Lifecycle Hooks**: snapshot של EBS ב-terminate, ויצירת volume מה-snapshot (לפי tag) ב-launch.
5. **Elasticity** — היכולת להוסיף **ולהסיר** capacity אוטומטית לפי הביקוש, כדי לשלם רק על מה שנדרש עכשיו. **High Availability** — הפריסה על לפחות שני AZs כדי שהשירות ימשיך לפעול גם כשאחד מהם נופל.
6. כל instance כותב את התמונה ל-**EBS volume מקומי** שלו, ובקשה שנוחתת על instance אחר לא מוצאת את הקובץ. הפתרון: אחסון **משותף** — **EFS** (mount target/ENI בכל AZ) לתאימות לאחור, או עדיף **S3** לתמונות עם CloudFront מלפנים.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[07 - Auto Scaling]] · [[08 - Elastic Load Balancing]] · [[22 - RDS Scaling and Availability]] · [[20 - EFS and File Storage]] · [[19 - EBS and EC2 Storage]] · [[34 - Disaster Recovery]] · [[02 - AWS Well-Architected Framework]] · [[06 - EC2 Pricing and Optimization]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 1798–1884, 2223–2283, 4081–4531, 15558–15627
