---
lesson: 10
title: VPC Internet Connectivity
domain: Design Cost-Optimized Architectures
services: [VPC, Internet Gateway, NAT Gateway, NAT Instance, Bastion Host, VPC Endpoints]
tags: [saa-c03, networking, vpc, nat, cost]
---

# 10 — VPC Internet Connectivity

> [!abstract] בשורה אחת
> IGW נותן ניתוב לאינטרנט, NAT נותן **יציאה בלבד** ל-subnet פרטי — ובמבחן זה בעיקר שאלת **עלות**: כמעט כל תעבורה שעוברת ב-NAT ולא ב-Gateway Endpoint היא כסף שנשרף.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה subnet פרטי בכל זאת צריך לצאת החוצה |
| 2 | איך זה עובד | IGW, Bastion, NAT Instance, NAT Gateway, זרימת חבילה מלאה |
| 3 | פירוק מפורט | source/destination check, HA לכל AZ, Regional NAT GW, Egress-Only IGW |
| 4 | **עלות** — הסעיף המרכזי | **טבלת data transfer יחסית**, NAT מול Gateway Endpoint, מזעור egress |
| 5 | השוואות | **NAT Gateway מול NAT Instance**, NAT מול Endpoint, Bastion מול SSM |
| 6 | Well-Architected | חיבוריות לאינטרנט לפי ששת ה-Pillars |
| 7 | מלכודות | NAT ב-AZ אחת, NAT ב-private subnet, NAT ל-inbound |
| 8 | Scenario | יציאה לאינטרנט בשתי AZs בעלות מינימלית |

**מונחי מפתח בשיעור:** `Internet Gateway` · `NAT Gateway` · `NAT Instance` · `Source/Destination Check` · `Bastion Host` · `Egress-Only IGW` · `Gateway Endpoint` · `Egress Traffic`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- שרתי האפליקציה חייבים לשבת ב-**private subnet** — אסור שיהיו נגישים מהאינטרנט.
- אבל הם **כן** צריכים לצאת החוצה: עדכוני אבטחה, `yum`/`apt`, חבילות npm/pip, קריאה ל-API חיצוני.
- IP פרטי לא ניתן לניתוב באינטרנט. חבילה שיוצאת עם `10.0.1.5` כמקור — אף שרת לא ידע לענות לה.
- צריך גם דרך **להתחבר לניהול** ל-instance פרטי בלי לחשוף אותו.
- וכל זה עולה כסף: תעבורה יוצאת מ-AWS היא אחד הסעיפים היקרים בחשבונית.

### מה השירות פותר

| רכיב | מה נותן | כיוון |
|---|---|---|
| **Internet Gateway** | ניתוב בין ה-VPC לאינטרנט | **דו-כיווני** (כניסה ויציאה) |
| **NAT Gateway / NAT Instance** | תרגום כתובת מקור, כדי ש-instance פרטי יוכל לצאת | **יציאה בלבד** |
| **Bastion Host** | נקודת כניסה מבוקרת יחידה ל-SSH פנימה | **כניסה** מבוקרת |
| **Egress-Only IGW** | המקבילה של NAT עבור **IPv6** | **יציאה בלבד** |
| **Gateway Endpoint** | מסלול פרטי ל-S3/DynamoDB **בלי לצאת בכלל** | פנימי |

> [!tip] האנלוגיה
> **IGW** היא דלת הכניסה הראשית של הבניין — נכנסים ויוצאים דרכה.
> **NAT** היא הדלפק בלובי: הוא מוציא עבורכם דואר החוצה בשם הבניין,
> אבל **אף אחד מבחוץ לא יכול ליזום ביקור** אצלכם דרכו.
> **Bastion** הוא השומר בכניסה — הדרך היחידה שמישהו מבחוץ נכנס בכלל.

---

## 2. ⚙️ איך זה עובד

### 2.1 Internet Gateway — התזכורת

- מאפשר למשאבים ב-VPC לתקשר עם האינטרנט.
- **מתרחב אופקית, זמין מאוד ומיותר (redundant)** — מנוהל במלואו על ידי AWS.
- **חייבים ליצור אותו בנפרד** מה-VPC ואז לחבר (attach).
- **1:1** — VPC אחד ל-IGW אחד, ולהפך.
- **IGW לבדו לא נותן אינטרנט.** חייבים גם **לערוך את ה-Route Table** ולהוסיף `0.0.0.0/0 → igw-id`.

פירוט מלא ב-[[09 - VPC Fundamentals]].

### 2.2 Bastion Host — כניסה מבוקרת ל-instances פרטיים

```text
   Users
     │ SSH (22)
     ▼
┌─────────────────────────────┐
│ Public Subnet               │
│   Bastion Host              │  ← SG: inbound 22 מ-CIDR **מצומצם** בלבד
│   sg-bastion                │     (למשל ה-CIDR הציבורי של המשרד)
└──────────────┬──────────────┘
               │ SSH (22)
               ▼
┌─────────────────────────────┐
│ Private Subnet              │
│   App EC2                   │  ← SG: inbound 22 **מ-sg-bastion**
│   sg-app                    │     (או מה-private IP של ה-bastion)
└─────────────────────────────┘
```

**שני כללי ה-SG שהמבחן שואל עליהם:**

1. **`sg-bastion`** — inbound על פורט 22 **מ-CIDR מוגבל** (ה-IP הציבורי של הארגון). **לא** `0.0.0.0/0`.
2. **`sg-app`** — inbound על פורט 22 **מ-`sg-bastion`** (הפניה ל-SG, לא ל-IP).

> [!tip] היום יש דרך טובה יותר
> **SSM Session Manager** נותן גישה ל-instance פרטי **בלי bastion, בלי פורט 22 פתוח,
> ובלי IP ציבורי בכלל** — עם audit trail ב-CloudTrail.
> במבחן, שאלה שמדגישה "no bastion / no open ports / auditable access" → **Session Manager**.
> Bastion עדיין נשאל כי הוא הדפוס הקלאסי, וכי לפעמים נדרש SSH ישיר.

### 2.3 NAT — מה קורה לחבילה בפועל

**NAT = Network Address Translation.** הוא **מחליף את כתובת המקור** של החבילה בכתובת שלו.

```text
1. יציאה:
   EC2 פרטי  10.0.1.10  ──▶  NAT (EIP 12.34.56.78)  ──▶  IGW  ──▶  שרת 50.60.4.10

   src: 10.0.1.10          src: 12.34.56.78
   dst: 50.60.4.10          dst: 50.60.4.10          ← המקור הוחלף!

2. חזרה:
   שרת 50.60.4.10  ──▶  IGW  ──▶  NAT  ──▶  EC2 פרטי 10.0.1.10

   src: 50.60.4.10          src: 50.60.4.10
   dst: 12.34.56.78          dst: 10.0.1.10          ← ה-NAT זוכר למי להחזיר
```

- השרת המרוחק רואה רק את ה-**Elastic IP של ה-NAT**. הוא לא יודע שקיים `10.0.1.10`.
- ולכן **אי אפשר ליזום חיבור מבחוץ** אל ה-instance הפרטי — אין לו כתובת שאפשר לפנות אליה.
- זהו בדיוק המנגנון שהופך את ה-NAT ל"יציאה בלבד".

### 2.4 שרשרת הניתוב המלאה

```text
Private Subnet Route Table          Public Subnet Route Table
──────────────────────────          ─────────────────────────
10.0.0.0/16   → local                10.0.0.0/16   → local
0.0.0.0/0     → nat-xxxx             0.0.0.0/0     → igw-xxxx

  זרימה:  Private EC2 → NAT Gateway → Internet Gateway → Internet
                          ↑
                  יושב ב-**public** subnet
```

> [!warning] שלוש הטעויות שהורגות את הזרימה הזו
> 1. **ה-NAT Gateway בטעות ב-private subnet** — אז אין לו דרך להגיע ל-IGW. הוא **חייב** לשבת ב-public subnet.
> 2. **חסר route ל-IGW ב-public subnet** — ה-NAT עצמו לא יוצא לשום מקום.
> 3. **ה-route ב-private subnet מפנה ל-IGW במקום ל-NAT** — אז ה-subnet כבר לא פרטי.

---

## 3. 🔍 פירוק מפורט

### 3.1 NAT Instance — מיושן, אבל עדיין במבחן

NAT Instance הוא **EC2 רגיל** שאתם מפעילים ומתחזקים כדי שיבצע NAT.

**התנאים המחייבים (כל אחד מהם נשאל):**

| דרישה | למה |
|---|---|
| **חייב לשבת ב-public subnet** | כדי שיוכל להגיע ל-IGW |
| **חייבת להיות לו Elastic IP** | זו הכתובת שהעולם רואה |
| **חייבים לבטל Source/Destination Check** | ראו למטה — זה הסעיף הכי מבלבל |
| **ה-Route Tables של ה-private subnets חייבות להפנות אליו** | אחרת אף אחד לא ישתמש בו |
| **חייבים לנהל לו Security Group** | בניגוד ל-NAT Gateway |

**Source / Destination Check — למה חייבים לבטל:**

- כברירת מחדל, EC2 **זורק כל חבילה** שהוא לא המקור או היעד שלה. זו הגנה בסיסית.
- אבל NAT Instance **כל תפקידו** הוא להעביר חבילות שאינן שלו.
- ולכן **חייבים לכבות את הבדיקה** — אחרת ה-instance ישתיק את כל התעבורה בשקט.
- זו טעות קלאסית בפתרון בעיות: "ה-NAT Instance רץ, ה-routes נכונים, ואין אינטרנט".

**החולשות של NAT Instance:**

- ה-AMI המוכן של Amazon הגיע **לסוף התמיכה הרשמית ב-31 בדצמבר 2020**.
- **אינו זמין מאוד out-of-the-box** — צריך לבנות ASG רב-AZ + סקריפט user-data שמטפל ב-failover.
- **רוחב הפס תלוי ב-instance type** שבחרתם. רוצים יותר — מגדילים ומשלמים.
- **אתם מנהלים את ה-SG:**

| כיוון | הכללים הנדרשים |
|---|---|
| **Inbound** | HTTP/HTTPS **מה-private subnets** · SSH מרשת הבית/המשרד |
| **Outbound** | HTTP/HTTPS **לאינטרנט** |

- אתם אחראים על patching, על ניטור ועל שדרוגי OS.

**היתרון היחיד שנשאר לו:**

- הוא **EC2 רגיל**, ולכן אפשר לשים עליו תוכנה משלכם — סינון תוכן, proxy, לוגים מותאמים.
- ואפשר להשתמש בו **גם כ-Bastion Host** — משהו ש-NAT Gateway **לא יכול** לעשות.

### 3.2 NAT Gateway — המנוהל

- **NAT מנוהל על ידי AWS** — רוחב פס גבוה, זמינות גבוהה, **אפס ניהול**.
- **משלמים לפי שעה** ולפי **כמות ה-data שעובד**.
- **נוצר ב-Availability Zone ספציפית** ומשתמש ב-**Elastic IP**.
- **רוחב פס: 5 Gbps, מתרחב אוטומטית עד 100 Gbps.**
- **דורש IGW** — הזרימה היא תמיד `Private Subnet → NAT GW → IGW`.
- **אין Security Groups בכלל** — אין מה לנהל, ואי אפשר לנהל.

> [!warning] המגבלה שנשאלת ישירות
> **NAT Gateway לא יכול לשרת instances שנמצאים ב-אותו subnet שלו.**
> הוא משרת **רק subnets אחרים**. זה הגיוני: הוא יושב ב-public subnet,
> וה-instances שצריכים אותו יושבים ב-private subnets.

### 3.3 NAT Gateway High Availability

- **NAT Gateway עמיד בתוך ה-AZ שלו** — AWS מטפלת בכשל רכיב.
- **אבל הוא לא חוצה AZ.** אם ה-AZ שלו נופלת — כל מי שהצביע אליו מנותק.

```text
Region
├── AZ-A                                  ├── AZ-B
│   Public Subnet ── NAT Gateway A        │   Public Subnet ── NAT Gateway B
│         ▲                               │         ▲
│   Private Subnet A ──────────────┘      │   Private Subnet B ──────────────┘
│   route: 0.0.0.0/0 → nat-A              │   route: 0.0.0.0/0 → nat-B
```

**הדפוס הנכון:**

- **NAT Gateway בכל AZ** שיש בה private subnet.
- כל private subnet מפנה ל-**NAT שב-AZ שלו** — לא ל-AZ אחרת.
- זה נותן גם **עמידות** וגם **חיסכון** — אין תעבורת cross-AZ.

> [!info] למה אין צורך ב-cross-AZ failover
> אם AZ-A נפלה — ה-instances ב-AZ-A ממילא לא רצים, ואין מי שיצטרך את ה-NAT שם.
> ולכן אין טעם לנתב את AZ-A ל-NAT של AZ-B "ליתר ביטחון" — זה רק מייצר חיוב cross-AZ.

### 3.4 Regional NAT Gateway

AWS הוסיפה וריאנט **Regional** של NAT Gateway:

| מאפיין | Regional NAT Gateway |
|---|---|
| שיוך | **ל-VPC**, לא ל-AZ בודדת |
| זמינות | זמין מאוד **בין AZs**, לא רק בתוך אחת |
| Public Subnets | **לא נדרשים** כדי לארח אותו |
| Route Tables | יש לו **טבלאות ניתוב משלו** |
| AZ חדשה | **מזהה משאבים ב-AZ חדשה ומתרחב אליה אוטומטית** |
| מה זה חוסך | את הצורך לפרוס NAT נפרד בכל AZ ולנהל route לכל אחד |

**המשמעות:** במקום שלושה NAT Gateways + שלוש route tables, יש אחד שמשרת את כל ה-VPC.

> [!note] מה עדיין תקף למבחן
> רוב שאלות ה-SAA-C03 עדיין מנוסחות סביב ה-**NAT Gateway ה-zonal** ודפוס
> "NAT בכל AZ ל-high availability". זו התשובה הבטוחה כשלא מוזכר Regional NAT במפורש.
> הכירו את ה-Regional כדי לא להיתפס מופתעים אם הוא מופיע כאפשרות.

### 3.5 Egress-Only Internet Gateway — ה-NAT של IPv6

- ב-IPv6 **אין כתובות פרטיות** — כל כתובת ניתנת לניתוב באינטרנט.
- ולכן NAT (שתפקידו לתרגם פרטי↔ציבורי) **לא רלוונטי** ל-IPv6.
- **Egress-Only IGW** נותן בדיוק את התכונה שרצינו: **יציאה כן, כניסה יזומה לא**.
- ב-Route Table הפרטית: `::/0 → eigw-xxxx`.

| | IPv4 | IPv6 |
|---|---|---|
| יציאה בלבד מ-private subnet | **NAT Gateway / NAT Instance** | **Egress-Only Internet Gateway** |
| דו-כיווני | Internet Gateway | Internet Gateway |

### 3.6 טבלת ניתוב לדוגמה — VPC מלא

```text
Public Subnet RT                    Private App Subnet RT
──────────────────                  ─────────────────────
10.0.0.0/16      → local             10.0.0.0/16      → local
0.0.0.0/0        → igw-1234          0.0.0.0/0        → nat-abcd  (ה-NAT באותו AZ)
::/0             → igw-1234          ::/0             → eigw-9999
                                     pl-s3 (prefix)   → vpce-s3   ← Gateway Endpoint

Private Data Subnet RT
──────────────────────
10.0.0.0/16      → local
(אין `0.0.0.0/0` בכלל — ה-DB לא יוצא לשום מקום)
```

- ה-**prefix list** של S3 (`pl-xxxx`) הוא ה-route שמופיע אוטומטית כשיוצרים Gateway Endpoint.
- לפי **Longest Prefix Match**, ה-route ל-S3 ספציפי יותר מ-`0.0.0.0/0` ולכן **מנצח את ה-NAT**.
  זו בדיוק הסיבה שהתעבורה ל-S3 מפסיקה לעבור ב-NAT ברגע שמוסיפים Endpoint.

---

## 4. 💰 עלות ותמחור — הסעיף המרכזי בשיעור

> [!info] כלל הקריאה
> אין כאן מחירים בדולרים — הם משתנים לפי Region. מה שחשוב הוא **היחסים**,
> וזה גם מה שהמבחן בודק.

### 4.1 טבלת Data Transfer — היחסים שחייבים לזכור

| נתיב התעבורה | עלות יחסית | הערה |
|---|---|---|
| **כניסה (ingress) לתוך AWS** | **0** | תעבורה נכנסת היא כמעט תמיד חינם |
| **באותה AZ, דרך private IP** | **0** | **הזול ביותר שקיים** |
| **באותה AZ, דרך public IP / Elastic IP** | **יקר** (בערך פי 2 מ-cross-AZ) | טעות תכנון נפוצה — לדבר דרך IP ציבורי בלי סיבה |
| **בין AZs באותו Region, private IP** | **נמוך** (יחידת בסיס) | מחויב **בשני הכיוונים** |
| **בין Regions** | **גבוה** (בערך פי 2 מ-cross-AZ) | replication, cross-region backup |
| **החוצה לאינטרנט (egress)** | **הגבוה ביותר** | הסעיף שהכי כדאי לצמצם |
| **דרך NAT Gateway** | **תוספת: שעות + לכל GB מעובד** | **בנוסף** לעלות ה-egress הרגילה |
| **דרך Gateway VPC Endpoint (S3/DynamoDB)** | **0 — חינם לגמרי** | אין שעות ואין GB |
| **מ-S3 ל-CloudFront** | **0** | ולכן CloudFront לפני S3 חוסך פעמיים |

**שני הכללים שנגזרים מהטבלה:**

1. **תמיד להעדיף private IP על public IP** — חוסך כסף **וגם** משפר ביצועים.
2. **תעבורה באותה AZ היא הזולה** — אבל אל תוותרו על multi-AZ בשביל זה. זה trade-off מול זמינות.

### 4.2 NAT Gateway מול Gateway VPC Endpoint — נושא העלות הקלאסי

זו אחת השאלות הכי נפוצות בבחינה. הנה אותו תרחיש, שני מסלולים:

```text
מסלול א' — דרך NAT Gateway              מסלול ב' — דרך Gateway Endpoint
────────────────────────────             ─────────────────────────────────
EC2 (private)                            EC2 (private)
   ↓  0.0.0.0/0 → nat                       ↓  pl-s3 → vpce
NAT Gateway (public subnet)              Gateway Endpoint
   ↓                                        ↓  (נשאר ברשת AWS)
Internet Gateway                         S3 Bucket
   ↓
S3 Bucket (דרך האינטרנט הציבורי)
```

| רכיב עלות | **דרך NAT Gateway** | **דרך Gateway Endpoint** |
|---|---|---|
| חיוב לשעה | **כן**, לכל NAT בכל AZ | **0** |
| GB מעובד | **כן**, לכל GB שעובר | **0** |
| Data transfer ל-S3 **באותו Region** | 0 | **0** |
| Data transfer ל-S3 **ב-Region אחר** | **יקר** (egress מלא) | לא רלוונטי — Gateway Endpoint הוא same-Region |
| התעבורה עוברת באינטרנט הציבורי | **כן** | **לא — נשארת ברשת AWS** |
| **סה"כ** | **שעות + GB, מצטבר בלי סוף** | **0** |

> [!warning] המסקנה שהמבחן רוצה
> **Gateway Endpoint ל-S3 ול-DynamoDB הוא חינם. אין שום סיבה להעביר את התעבורה הזו ב-NAT.**
> שאלה שמתארת "instances פרטיים מעלים כמות גדולה של נתונים ל-S3 והחשבון של ה-NAT Gateway מתפוצץ"
> → התשובה היא **תמיד Gateway Endpoint**, לא NAT גדול יותר ולא instance type אחר.

### 4.3 מזעור תעבורת Egress — הדפוס ההפוך

**Egress** = תעבורה שיוצאת מ-AWS החוצה. **Ingress** = תעבורה שנכנסת, ובדרך כלל **חינם**.

הדוגמה הקלאסית: אפליקציה ו-DB, אחד ב-AWS ואחד ב-Data Center.

```text
❌ יקר                                  ✅ זול
─────────────────                       ────────────────
AWS: Database                           AWS: Application
   ↑ query 100 MB                          ↑ results 50 KB
   ↓ results 50 KB                         ↓ query 100 MB
On-prem: Application                    On-prem: Database

ה-100 MB יוצאים מ-AWS = egress יקר      ה-100 MB נכנסים ל-AWS = ingress חינם
                                        רק 50 KB יוצאים = egress זניח
```

**הכלל:** **שימו את הרכיב ש*מייצר* את התעבורה הגדולה בצד שממנו היציאה זולה.**
כאן — האפליקציה ששולחת query כבד צריכה לשבת **ליד** ה-DB.

**עוד דרכים לצמצם egress:**

- **לשמור כמה שיותר תעבורה בתוך AWS.** שירות שקורא לשירות ב-AWS לא משלם egress לאינטרנט.
- **CloudFront לפני S3** — התעבורה מ-S3 ל-CloudFront היא **חינם**, ומ-CloudFront החוצה
  היא **מעט זולה יותר** מ-S3 ישירות, עם cache שמקטין גם את מספר ה-requests ל-S3 (סדר גודל).
- **Direct Connect location שנמצא באותו Region** — נותן תעריף egress נמוך יותר. ראו [[12 - VPC Private Connectivity]].
- **דחיסה** של תגובות API ושל קבצים לפני שליחה.

### 4.4 על מה מחייבים — סיכום

| רכיב | חיוב | הערה |
|---|---|---|
| **Internet Gateway** | **0 לרכיב** | משלמים רק על ה-data שעובר בו |
| **Route Tables** | **0** | |
| **NAT Gateway** | **שעה × כל AZ + GB מעובד** | הרכיב היקר בשיעור |
| **NAT Instance** | **EC2 + EBS + data transfer** | וגם עלות תפעול אנושית |
| **Elastic IP של ה-NAT** | לשעה | כלול בחישוב |
| **Gateway Endpoint** (S3/DynamoDB) | **0** | חינם לחלוטין |
| **Interface Endpoint** | שעה × AZ + GB | ראו [[12 - VPC Private Connectivity]] |
| **Bastion Host** | EC2 + IP ציבורי | SSM Session Manager חוסך אותו |
| **Egress-Only IGW** | **0 לרכיב** | data transfer בלבד |

### 4.5 🚩 עלויות נסתרות

- **NAT Gateway שרץ 24/7 בשלוש AZs** — שלושה חיובים לשעה שנצברים גם בלי טיפת תעבורה.
- **תעבורה ל-S3 שעוברת ב-NAT** — נפוץ מאוד, וכולו מיותר. Gateway Endpoint מאפס אותו.
- **NAT Gateway ב-AZ אחת שמשרת שלוש AZs** — "חוסך" שני NATs ומשלם **cross-AZ על כל GB**,
  ובנוסף יוצר SPOF. לרוב יקר **יותר** בסך הכול.
- **Bastion Host שנשכח דולק** — EC2 + IP ציבורי, 24/7, בשביל התחברות פעם בשבוע.
- **Elastic IP יתומה** אחרי מחיקת NAT Instance.
- **תעבורת cross-AZ בין app ל-DB** — נגבית בשני הכיוונים ומצטברת בשקט.
- **NAT Gateway שמשמש רק ל-`yum update`** — לפעמים זול יותר לבנות AMI מעודכן מראש.

### 4.6 💡 טיפים לחיסכון

- **Gateway Endpoint ל-S3 ול-DynamoDB — תמיד.** הוא חינם, ואין תרחיש שבו NAT עדיף עליו לשירותים האלה.
- **Interface Endpoints** לשירותים כבדי-תעבורה אחרים (ECR, CloudWatch Logs) — בתשלום,
  אבל לרוב זול מ-NAT כשהנפח גדול.
- **NAT בכל AZ** — נשמע יקר יותר, אבל מבטל cross-AZ ומבטל SPOF. לרוב הזול והנכון.
- **בדקו את מדדי ה-NAT ב-CloudWatch** (`BytesOutToDestination`) כדי לזהות מי צורך.
- **כבו NAT Gateway בסביבות dev** מחוץ לשעות עבודה — הוא מחויב לשעה גם כשאף אחד לא עובד.
- **CloudFront לפני S3** לתעבורה ציבורית — ראו [[15 - CloudFront and Global Delivery]].
- **העדיפו private IP** בכל תקשורת פנימית. גם באותה AZ, שימוש ב-public IP עולה.
- **שקלו AMI מעודכן** במקום NAT שמשמש רק ל-patching.

---

## 5. ⚖️ השוואות מכריעות

### NAT Gateway מול NAT Instance — הטבלה שנשאלת

| קריטריון | **NAT Gateway** | **NAT Instance** |
|---|---|---|
| **ניהול** | **מנוהל במלואו על ידי AWS** | **אתם** — OS, patches, תוכנה |
| **זמינות** | עמיד **בתוך ה-AZ**; ל-HA יוצרים בכל AZ | דורש **script/ASG** לניהול failover ידני |
| **רוחב פס** | **5 Gbps, מתרחב אוטומטית עד 100 Gbps** | **תלוי ב-instance type** שבחרתם |
| **Security Groups** | **אין — ואי אפשר להצמיד** | **חובה לנהל** inbound ו-outbound |
| **שימוש כ-Bastion Host** | **לא** | **כן** |
| **Source/Dest Check** | לא רלוונטי | **חובה לבטל** |
| **Elastic IP** | כן, אחת | כן, חובה |
| **Public IPv4** | כן | כן |
| **Private IPv4** | כן | כן |
| **עלות** | לפי שעה + **GB מעובד** | לפי שעה של ה-EC2 (type וגודל) + עלויות רשת |
| **התאמה אישית** | **אין** | **כן** — proxy, filtering, לוגים |
| **סטטוס** | הבחירה הנכונה כברירת מחדל | **מיושן** (AMI ללא תמיכה מאז סוף 2020) |

> [!info] שורה תחתונה
> **NAT Gateway כמעט תמיד.** NAT Instance נשאר רלוונטי בשתי נקודות בלבד:
> כשצריך **תוכנה מותאמת** על נתיב היציאה, או כשרוצים **שהוא ישמש גם כ-Bastion**.
> אם השאלה מזכירה "custom filtering software on outbound traffic" — זו הרמז ל-NAT Instance.

### NAT Gateway מול Gateway Endpoint

| קריטריון | **NAT Gateway** | **Gateway Endpoint** |
|---|---|---|
| למי מיועד | **כל יעד באינטרנט** | **S3 ו-DynamoDB בלבד** |
| עלות | שעות + GB | **0** |
| התעבורה עוברת באינטרנט | **כן** | **לא** |
| דורש IGW | **כן** | **לא** |
| בקרת גישה | SG/NACL ברמת הרשת | **Endpoint Policy** |
| מנגנון | route ל-`0.0.0.0/0` | **prefix list** ב-route table |

### Bastion Host מול SSM Session Manager

| קריטריון | **Bastion Host** | **Session Manager** |
|---|---|---|
| דורש instance ייעודי | **כן** | **לא** |
| דורש פורט 22 פתוח | **כן** | **לא** |
| דורש IP ציבורי / IGW | כן ל-bastion | **לא** (עם Interface Endpoints אפשר בלי אינטרנט כלל) |
| ניהול מפתחות SSH | **כן** | **לא** |
| Audit trail | לוגים על ה-bastion | **CloudTrail + session logs** |
| עלות | EC2 + IP ציבורי | **0 לשירות** |

> [!info] שורה תחתונה
> "יציאה בלבד לאינטרנט מ-subnet פרטי" → **NAT Gateway**.
> "רק ל-S3/DynamoDB" → **Gateway Endpoint**, כי הוא חינם.
> "גישת ניהול מאובטחת ל-instance פרטי" → **Session Manager**, ו-**Bastion** רק כשנדרש SSH ישיר.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בחיבוריות לאינטרנט** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | הנתיב לאינטרנט מוגדר בקוד וניתן לאבחון | VPC + NAT + routes ב-IaC; alarms על `ErrorPortAllocation` ועל `BytesOutToDestination`; **VPC Flow Logs** לאיתור מי צורך; runbook לבדיקת route tables |
| **Security** | אף instance פרטי לא נגיש מבחוץ, והיציאה מוגבלת | private subnets לכל מה שאינו LB; **NAT לא מקבל inbound**; bastion עם SSH מ-CIDR מצומצם או **Session Manager** במקומו; **Endpoint Policy** להגבלת אילו buckets נגישים |
| **Reliability** | כשל AZ לא מנתק את שאר ה-AZs | **NAT Gateway בכל AZ**, וכל private subnet מפנה ל-NAT שלו; לא לסמוך על NAT Instance בודד; לא לנתב cross-AZ "ליתר ביטחון" |
| **Performance Efficiency** | פחות קפיצות ופחות latency | **Gateway/Interface Endpoints** במקום סיבוב דרך האינטרנט; שמירת תעבורה בתוך AZ; NAT Gateway (עד 100 Gbps) במקום NAT Instance מוגבל |
| **Cost Optimization** | לא משלמים על תעבורה שלא הייתה צריכה לצאת | **Gateway Endpoint ל-S3/DynamoDB — חינם**; מזעור egress; private IP במקום public IP; כיבוי NAT ב-dev; ניתוח GB לפי מקור |
| **Sustainability** | פחות עיבוד רשת מיותר | מסלול ישיר במקום דרך NAT ואינטרנט; CloudFront שמקטין מקור-הגשה חוזר; לא לשלוח תעבורה פנימית סיבוב דרך העולם |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "private instances need to download OS updates" | **NAT Gateway** (ב-public subnet) |
| "must not be reachable from the internet, but needs outbound" | **NAT**, לא IGW |
| "high NAT Gateway data processing charges for S3 traffic" | **Gateway VPC Endpoint ל-S3** |
| "reduce cost of accessing S3 and DynamoDB privately" | **Gateway Endpoint — חינם** |
| "NAT must survive an AZ failure" | **NAT Gateway בכל AZ** + route מקומי |
| "custom filtering software on outbound traffic" | **NAT Instance** (רק הוא ניתן להתאמה) |
| "the NAT should also serve as a bastion" | **NAT Instance** — NAT Gateway לא יכול |
| "NAT instance is running but no internet" | **Source/Destination Check** לא בוטל |
| "SSH into private instances" | **Bastion Host**, או עדיף **SSM Session Manager** |
| "no bastion, no open ports, auditable" | **Session Manager** |
| "outbound-only internet over IPv6" | **Egress-Only Internet Gateway** |
| "minimize data transfer costs between app and DB" | **אותה AZ, private IP** |
| "large query from on-prem DB to AWS app" | להעביר את ה-**DB או ה-App** כך שהתעבורה הכבדה תהיה **ingress** |
| "reduce S3 delivery cost to internet users" | **CloudFront** (S3→CloudFront חינם) |
| "instances in the same subnet as the NAT Gateway can't reach it" | **מגבלה מובנית** — NAT GW משרת רק subnets אחרים |

### טעויות נפוצות

> [!warning] מלכודת 1 — NAT Gateway ב-private subnet
> **הניסוח:** "Deploy the NAT Gateway in the private subnet to keep it secure."
> **הטעות:** לחשוב ש-NAT "פרטי" צריך לשבת ב-private subnet.
> **הנכון:** ה-NAT **חייב לשבת ב-public subnet** עם route ל-IGW — אחרת הוא עצמו לא יוצא לשום מקום.
> הוא "פרטי" בכך שהוא לא מקבל inbound, לא בכך שהוא ב-subnet פרטי.

> [!warning] מלכודת 2 — NAT יחיד ל-multi-AZ
> **הניסוח:** "One NAT Gateway serves all three private subnets to save cost."
> **הטעות:** להניח שזה חיסכון.
> **הנכון:** זה **SPOF** (כשל ב-AZ שלו מנתק את כולם) **וגם** מייצר **חיוב cross-AZ על כל GB**.
> לרוב יוצא **יקר יותר** בסך הכול. הדפוס הנכון: **NAT בכל AZ**.

> [!warning] מלכודת 3 — NAT ל-inbound
> **הניסוח:** "Use a NAT Gateway to allow users to reach the private web servers."
> **הטעות:** לחשוב על NAT כעל שער דו-כיווני.
> **הנכון:** **NAT הוא יציאה בלבד.** לכניסה מהאינטרנט צריך **ALB ב-public subnet**
> ([[08 - Elastic Load Balancing]]) או IGW + IP ציבורי.

> [!warning] מלכודת 4 — לשכוח את Source/Destination Check
> **הניסוח:** "The NAT instance is running, routes are correct, but private instances have no internet."
> **הטעות:** לחפש בעיות ב-route table או ב-SG.
> **הנכון:** ב-NAT **Instance** חייבים **לבטל את ה-Source/Destination Check**.
> בלי זה ה-EC2 זורק בשקט כל חבילה שהוא לא המקור או היעד שלה.

> [!warning] מלכודת 5 — NAT במקום Gateway Endpoint ל-S3
> **הניסוח:** "Add a larger NAT Gateway to handle the S3 backup traffic."
> **הטעות:** לפתור בעיית עלות בהגדלת הרכיב היקר.
> **הנכון:** **Gateway Endpoint ל-S3 הוא חינם** ומוציא את התעבורה הזו מה-NAT לגמרי.
> לפי Longest Prefix Match, ה-prefix list של S3 מנצח את `0.0.0.0/0`.

> [!warning] מלכודת 6 — Bastion עם SSH פתוח לעולם
> **הניסוח:** "Bastion security group allows SSH from 0.0.0.0/0."
> **הטעות:** לחשוב שזה בסדר כי "רק מי שיש לו מפתח ייכנס".
> **הנכון:** ה-SG של ה-bastion חייב להתיר 22 **מ-CIDR מצומצם** בלבד.
> וה-SG של ה-instances הפרטיים מתיר 22 **מ-SG של ה-bastion**, לא מ-IP.

> [!warning] מלכודת 7 — public IP באותה AZ "בטח חינם"
> **הניסוח:** "Both instances are in the same AZ, so traffic between them is free."
> **הטעות:** להתעלם באיזו כתובת הם משתמשים.
> **הנכון:** חינם **רק דרך private IP**. תקשורת דרך **public/Elastic IP** מחויבת —
> ודווקא בתעריף הגבוה יחסית.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברת SaaS מריצה אפליקציה ב-VPC על שתי AZs. הדרישות:

- ה-instances של האפליקציה **לא נגישים מהאינטרנט**, אבל צריכים `yum update` וקריאה ל-API של ספק תשלומים.
- הם מעלים **טרות של לוגים וגיבויים ל-S3** מדי יום.
- הם קוראים וכותבים ל-**DynamoDB** בתדירות גבוהה.
- כשל של AZ שלמה **לא יפגע** ב-AZ השנייה.
- הצוות צריך גישת ניהול ל-instances, ומבקר האבטחה דורש **audit trail**.
- החשבון של ה-NAT Gateway הוא כרגע הסעיף השני בגודלו בחשבונית.

```text
                              Internet
                                 ↕
                          Internet Gateway
                                 ↕
   ┌─────────────── AZ-A ──────────────┐  ┌────────── AZ-B ──────────┐
   │ Public Subnet                     │  │ Public Subnet            │
   │   ALB  +  NAT Gateway A           │  │   ALB  +  NAT Gateway B  │
   └────────────────┬──────────────────┘  └───────────┬──────────────┘
                    │ 0.0.0.0/0 → nat-A               │ 0.0.0.0/0 → nat-B
   ┌────────────────▼──────────────────┐  ┌───────────▼──────────────┐
   │ Private App Subnet A              │  │ Private App Subnet B     │
   │   EC2 (ASG)                       │  │   EC2 (ASG)              │
   └───────┬─────────────────┬─────────┘  └──────────────────────────┘
           │ pl-s3 → vpce    │ pl-ddb → vpce
           ▼                 ▼
   ┌──────────────┐   ┌──────────────┐
   │ S3 Gateway   │   │ DynamoDB     │      ← שניהם **חינם**
   │ Endpoint     │   │ Gateway EP   │
   └──────────────┘   └──────────────┘
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **NAT Gateway בכל AZ** | ה-NAT הוא zonal. אחד בלבד = SPOF **וגם** חיוב cross-AZ על כל GB |
| כל private subnet מפנה **ל-NAT באותו AZ** | מבטל cross-AZ; ואם ה-AZ נופלת, ה-instances שם ממילא לא רצים |
| **Gateway Endpoint ל-S3** | הטרות של הגיבויים יוצאות מה-NAT לגמרי. **עלות 0** במקום GB מעובד |
| **Gateway Endpoint ל-DynamoDB** | אותו היגיון, ובנוסף latency נמוך יותר — לא עוברים באינטרנט |
| ה-NAT **נשאר** בשביל `yum` וה-API של התשלומים | אלה יעדים באינטרנט הפתוח שאין להם Endpoint |
| **ALB ב-public subnets** לכניסה | **NAT לא מקבל inbound**. הכניסה מהעולם היא רק דרך ה-LB |
| **SSM Session Manager** לגישת ניהול | בלי bastion, בלי פורט 22, בלי IP ציבורי — ועם **audit trail ב-CloudTrail** |
| **Endpoint Policy** על ה-S3 Endpoint | מגביל את הגישה ל-buckets של החברה בלבד, ולא לכל S3 בעולם |
| **VPC Flow Logs** על ה-VPC | מזהים מי עדיין צורך NAT ומה אפשר להעביר ל-Endpoint. ראו [[11 - VPC Security]] |
| **CloudWatch alarm** על `BytesOutToDestination` | התראה כשמישהו מתחיל לשפוך תעבורה חדשה דרך ה-NAT |

**למה לא NAT Gateway אחד לשתי ה-AZs?**
נראה כמו חיסכון של חיוב שעתי אחד, אבל: (א) כשל ה-AZ שלו מנתק **את שתי** ה-AZs,
ו-(ב) כל GB מ-AZ-B ל-NAT ב-AZ-A מחויב **cross-AZ בנוסף** לחיוב ה-NAT. לרוב יוצא יקר יותר.

**למה לא Interface Endpoint ל-S3 במקום Gateway Endpoint?**
Interface Endpoint מחויב **לשעה לכל AZ ולכל GB**. Gateway Endpoint **חינם**.
ל-S3 ול-DynamoDB בתוך אותו Region — Gateway הוא הבחירה.
(Interface Endpoint ל-S3 רלוונטי רק כשצריך גישה **מ-on-prem** דרך DX/VPN — ראו [[12 - VPC Private Connectivity]].)

**למה לא NAT Instance כדי לחסוך?**
צריך לנהל אותו, לתקן אותו, לבנות לו HA ידנית, ורוחב הפס שלו תלוי ב-type.
בנפח הזה הוא גם יהיה צוואר בקבוק וגם לא באמת יחסוך.

**למה לא Bastion Host?**
EC2 נוסף שרץ 24/7 עם IP ציבורי, פורט 22 חשוף, וניהול מפתחות SSH —
בשביל מה ש-Session Manager נותן בחינם, בלי פורט פתוח ועם לוג מלא.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **המחירים המדויקים בדולרים** של NAT Gateway ושל data transfer. חשובים ה**יחסים** בלבד.
- **ההגדרה הפנימית של NAT Instance** — `iptables`, `sysctl`, סקריפטי failover.
- **פרטי ה-AMI המיושן** של NAT Instance מעבר לעובדה שהתמיכה בו הסתיימה בסוף 2020.
- **המבנה הפנימי** של ה-port allocation ב-NAT Gateway (מעבר לכך שיש metric שמתריעה על מיצוי).
- **תחביר CLI** ליצירת NAT/IGW.
- **פרטי ה-Regional NAT Gateway לעומק** — מספיק לדעת שהוא VPC-scoped, מתרחב אוטומטית ל-AZ חדשה,
  ולא דורש public subnets.
- **טבלת מחירי CloudFront** — מספיק לזכור ש-S3→CloudFront הוא **0** ושה-egress מ-CloudFront זול מעט יותר.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **IGW = דו-כיווני. NAT = יציאה בלבד. Egress-Only IGW = יציאה בלבד ל-IPv6.**
- **NAT Gateway חייב לשבת ב-public subnet** עם route ל-IGW. הזרימה: `Private → NAT → IGW → Internet`.
- **NAT Gateway לא משרת instances באותו subnet שלו** — רק subnets אחרים.
- **ל-NAT Gateway אין Security Group.** ל-NAT Instance **חייבים** לנהל אחד.
- **NAT Gateway: 5 Gbps, מתרחב אוטומטית עד 100 Gbps.** NAT Instance — לפי instance type.
- **NAT Gateway הוא zonal.** ל-HA: **אחד בכל AZ**, וכל private subnet מפנה ל-NAT שלו.
- **אין צורך ב-cross-AZ failover** — AZ שנפלה ממילא לא צריכה NAT.
- **NAT Instance: חייבים לבטל Source/Destination Check.** בלי זה — שום דבר לא עובר.
- **רק NAT Instance יכול לשמש גם כ-Bastion Host.** NAT Gateway לא.
- **Gateway Endpoint ל-S3 ול-DynamoDB — חינם לגמרי.** אין סיבה להעביר את התעבורה הזו ב-NAT.
- **Interface Endpoint — בתשלום** לשעה לכל AZ ולכל GB.
- **Ingress ל-AWS ≈ חינם. Egress לאינטרנט = היקר ביותר.**
- **אותה AZ ב-private IP = 0. אותה AZ ב-public IP = יקר.** תמיד להעדיף private IP.
- **cross-AZ מחויב בשני הכיוונים. cross-Region יקר יותר מ-cross-AZ.**
- **S3 → CloudFront = 0.** CloudFront → אינטרנט זול מעט מ-S3 ישירות, ומקטין requests.
- **הכלל למזעור egress:** לשים את הרכיב שמייצר את התעבורה הכבדה **בצד שממנו היציאה חינם**.
- **Bastion SG:** 22 מ-**CIDR מצומצם**. **SG של ה-instance הפרטי:** 22 **מ-SG של ה-bastion**.
- **SSM Session Manager** — בלי bastion, בלי פורט 22, בלי IP ציבורי, עם audit.

---

## 11. ✅ בדיקת הבנה

1. השקתם NAT Gateway ב-private subnet וה-instances עדיין בלי אינטרנט. מה הבעיה?
2. NAT Instance רץ, ה-routes נכונים, ה-SG פתוח — ואין תעבורה. מה שכחתם?
3. חשבון ה-NAT Gateway קפץ בגלל העלאות יומיות ל-S3. מה הפתרון, ולמה הוא עובד ברמת ה-route?
4. למה NAT Gateway אחד שמשרת שלוש AZs עלול לצאת **יקר יותר** משלושה?
5. שני instances באותה AZ מדברים ביניהם דרך ה-Elastic IP שלהם. מה הבעיה?
6. אפליקציה ב-on-prem שולחת query של 100 MB ל-DB ב-AWS ומקבלת 50 KB. איך משנים כדי לחסוך?
7. באיזה מקרה **NAT Instance** עדיף על NAT Gateway?
8. איך נותנים ליציאה בלבד לאינטרנט ב-IPv6 מ-subnet פרטי?

<details>
<summary>תשובות</summary>

1. **NAT Gateway חייב לשבת ב-public subnet.** ב-private subnet אין לו route ל-IGW,
   ולכן הוא עצמו לא מגיע לאינטרנט. "פרטיות" ה-NAT נובעת מכך שהוא לא מקבל inbound — לא מהמיקום שלו.
2. **לבטל את ה-Source/Destination Check** על ה-EC2 של ה-NAT Instance.
   כברירת מחדל EC2 זורק כל חבילה שהוא לא המקור או היעד שלה — וזה בדיוק מה ש-NAT עושה.
3. **Gateway VPC Endpoint ל-S3** — חינם לחלוטין. ברמת ה-route, ה-Endpoint מוסיף
   **prefix list** של S3 לטבלת הניתוב. לפי **Longest Prefix Match** היא ספציפית יותר מ-`0.0.0.0/0 → nat`,
   ולכן התעבורה ל-S3 מפסיקה לעבור ב-NAT מיד.
4. כי כל GB שמגיע מ-AZ אחרת מחויב **cross-AZ בנוסף** לחיוב ה-GB של ה-NAT עצמו.
   בנפחים גבוהים זה עולה על החיסכון בשני חיובי השעה. ובנוסף — זה **SPOF**:
   כשל ה-AZ של ה-NAT מנתק את **כל** ה-AZs.
5. תקשורת דרך **public/Elastic IP** מחויבת גם בתוך אותה AZ, ובתעריף גבוה יחסית —
   בעוד תקשורת דרך **private IP** באותה AZ היא **חינם**, וגם מהירה יותר.
6. **להעביר את האפליקציה ל-AWS ליד ה-DB** (או את ה-DB ל-on-prem ליד האפליקציה).
   הכלל: התעבורה הכבדה צריכה להיות **ingress** (חינם) ולא **egress** (יקר).
   כאן ה-100 MB ייכנסו ל-AWS חינם ורק 50 KB יצאו.
7. בשני מקרים: (א) כשצריך **תוכנה מותאמת** על נתיב היציאה — סינון תוכן, proxy, לוגים מיוחדים;
   (ב) כשרוצים שאותו instance ישמש **גם כ-Bastion Host** — יכולת ש-NAT Gateway לא נותן.
8. **Egress-Only Internet Gateway**, עם route `::/0 → eigw-xxxx` בטבלת ה-private subnet.
   NAT לא רלוונטי ל-IPv6 כי אין ב-IPv6 כתובות פרטיות שצריך לתרגם.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[09 - VPC Fundamentals]] · [[11 - VPC Security]] · [[12 - VPC Private Connectivity]] · [[13 - VPC Network Architecture]] · [[08 - Elastic Load Balancing]] · [[15 - CloudFront and Global Delivery]] · [[37 - Cost Optimization]] · [[05 - EC2 Fundamentals]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 12993–13263, 14524–14660
