---
lesson: 40
title: Integrated SAA Scenarios
domain: Design Resilient Architectures
services: [EC2, ALB, ASG, RDS, ElastiCache, EFS, S3, CloudFront, Lambda, API Gateway, DynamoDB, SQS, SNS, EventBridge, Kinesis, Route 53]
tags: [saa-c03, scenarios, practice, architecture, integration]
---

# 40 — Integrated SAA Scenarios

> [!abstract] בשורה אחת
> שיעור תרגול: שמונה ארכיטקטורות מלאות, כל אחת נבנית מהפתרון הנאיבי ועד לפתרון שעומד במבחן —
> ובכל אחת גם **מה עולה** וגם **מה משתנה כשהדרישה משתנה**.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה ידיעת שירותים לא מספיקה |
| 2 | איך זה עובד | **המבנה הקבוע** של פענוח תרחיש + שכבות הארכיטקטורה |
| 3 | שמונת התרחישים | stateless גלובלי · stateful · WordPress · תמונות serverless · אירועים · hybrid · DR · HPC |
| 4 | עלות | איפה הכסף בורח בכל אחד מהדפוסים |
| 5 | השוואות | הדפוסים זה מול זה |
| 6 | Well-Architected | סקירה של הדפוסים לפי ששת ה-Pillars |
| 7 | מלכודות | הטעויות שחוזרות בכל תרחיש |
| 8 | Scenario מסכם | חנות גלובלית שמשלבת את כל השמונה |

**מונחי מפתח בשיעור:** `Stateless` · `Stickiness` · `Fan-Out` · `Golden AMI` · `OAC` · `Lifecycle Hook` · `Alias Record`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- לדעת מה S3 עושה זה 20% מהעבודה. לדעת **מתי הוא לא מתאים** זה השאר.
- ארכיטקטורה נכונה בשכבה אחת יכולה **לשבור** שכבה אחרת: NAT ב-AZ אחת הוא נקודת כשל,
  DB ציבורי עם SG סגור הוא סיכון, תור בלי DLQ מאבד הודעות בשקט.
- במבחן מוצג **סיפור**, לא רשימת שירותים — וצריך לתרגם אותו לתרשים בראש תוך שניות.

### מה השיעור פותר

- **בונה שרירים** — אותם דפוסים חוזרים בעשרות שאלות. מי שראה אותם מזהה אותם מיד.
- מלמד את **ההתפתחות**: כל ארכיטקטורה מתחילה נאיבית ומשתפרת בצעדים, כמו במציאות.
- כל תרחיש נגמר ב-**וריאציה** — כי המבחן אוהב לשנות פרט אחד ולהפוך את התשובה.

> [!tip] האנלוגיה
> זה לא שיעור — זו **חדר כושר**. בשיעורים 1–39 למדתם את התרגילים.
> כאן מרימים משקל אמיתי, ומגלים איזה שריר עדיין חלש.

---

## 2. ⚙️ איך זה עובד — המבנה הקבוע

### 2.1 המבנה של כל תרחיש בשיעור

```text
הדרישה   →   אילוצים   →   דיאגרמה   →   טבלת החלטות
                                              ↓
                          למה לא X  →  מה זה עולה  →  וריאציה
```

### 2.2 שכבות הארכיטקטורה — הסדר שמציירים בו תמיד

```text
User
 └► Route 53 (DNS)                 ← איזה record? Alias או A? מה ה-TTL?
     └► CloudFront / Global Accelerator  ← יש משתמשים גלובליים?
         └► ALB / NLB / API Gateway      ← מה מסיים את החיבור?
             └► Compute  (private subnet)   ← EC2/ASG · Fargate · Lambda
                 └► Data                    ← RDS · DynamoDB · S3 · EFS
                     └► Cache               ← ElastiCache · DAX
                         └► Async           ← SQS · SNS · EventBridge · Kinesis
                             └► Observability  ← CloudWatch · CloudTrail
```

**בכל שכבה שואלים ארבע שאלות:**

1. **Public או Private?** — כמעט הכל צריך להיות private.
2. **כמה AZs?** — לפחות שתיים, תמיד.
3. **איפה ה-state?** — מי מחזיק דאטה שאסור לאבד?
4. **מה נופל אם זה נופל?** — failure domain.

### 2.3 שלושה כלים שמאיצים כל תרחיש

הבעיה: הקמת stack מלא (EC2 + EBS + RDS) לוקחת זמן — התקנת אפליקציות, הזרקת דאטה, קונפיגורציה.

| הכלי | מה עושה | מתי |
|---|---|---|
| **Golden AMI** | AMI עם האפליקציה, התלויות וה-OS **מותקנים מראש** | ההתקנה איטית; רוצים launch מהיר ועקבי ב-ASG |
| **User Data (Bootstrap)** | סקריפט שרץ בעלייה, לקונפיגורציה **דינמית** | פרמטרים שמשתנים בין סביבות |
| **היברידי** | Golden AMI + User Data — כך **Elastic Beanstalk** עובד | הכי מהיר והכי גמיש |
| **RDS Restore from Snapshot** | ה-DB עולה עם schema ודאטה מוכנים | שחזור, סביבות בדיקה |
| **EBS Restore from Snapshot** | הדיסק עולה מפורמט ועם דאטה | אותו דבר לרמת הדיסק |

---

## 3. 🔍 שמונת התרחישים

---

### 3.1 תרחיש A — אתר stateless גלובלי

**הדרישה:** שירות API פשוט שמחזיר תשובה מחושבת. אין דאטה לשמור, אין sessions.
מתחילים קטן, וצריך להגיע לזמינות גבוהה בלי downtime בשדרוגים.

**אילוצים:**

- אין database — האפליקציה **stateless** לחלוטין.
- חייב לשרוד כשל **AZ** שלמה.
- הוספה והסרה של שרתים חייבות להיות **שקופות למשתמש**.
- העלות בשעות שקטות צריכה להיות נמוכה.

**ההתפתחות — ארבעה צעדים:**

```text
צעד 1 — EC2 יחיד + Elastic IP
   User → EIP → EC2
   ❌ scale אנכי בלבד; שדרוג ל-instance גדול = downtime; SPOF

צעד 2 — כמה EC2 עם A Records ב-Route 53
   User → Route 53 (A record, TTL 1h) → EC2 × 3
   ❌ instance שנופל נשאר ב-DNS למשך ה-TTL. משתמשים מגיעים לשרת מת

צעד 3 — ELB + Health Checks
   User → Route 53 (Alias) → ALB → EC2 פרטיים × N
   ✅ health check מוציא שרת מת תוך שניות; ה-EC2 עוברים ל-private subnet

צעד 4 — ASG + Multi-AZ  (הפתרון)
   User → Route 53 (Alias) → ALB (2–3 AZs) → ASG(EC2) פרוס על 2–3 AZs
```

```text
                    Route 53 — Alias Record
                              │
                    ┌─────────▼─────────┐
                    │        ALB        │  ← Public subnets, 2+ AZs
                    └─────────┬─────────┘
              ┌───────────────┼───────────────┐
         ┌────▼────┐     ┌────▼────┐     ┌────▼────┐
         │  AZ-a   │     │  AZ-b   │     │  AZ-c   │
         │ EC2×n   │     │ EC2×n   │     │ EC2×n   │  ← Private, Auto Scaling Group
         └─────────┘     └─────────┘     └─────────┘
              SG: מקבל תעבורה רק מה-SG של ה-ALB
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **Alias Record** ולא A Record | Alias מצביע ישירות על ה-ALB, מתעדכן אוטומטית, **חינם** ב-Route 53, ואין תלות ב-TTL |
| **ALB** ולא Elastic IPs | Health Checks מוציאים instance מת מיד; ה-EC2 עוברים ל-private והופכים בלתי נגישים ישירות |
| **ASG** ולא ניהול ידני | מוסיף ומסיר instances לפי עומס; מחליף אוטומטית instance כושל |
| **לפחות 2 AZs** (עדיף 3) | ASG שכל ה-instances שלו ב-AZ אחת נופל עם ה-AZ |
| **EC2 ב-private subnet** | אין להם IP ציבורי; רק ה-ALB חשוף |
| **SG-to-SG** | ה-SG של ה-EC2 מתיר תעבורה **מה-SG של ה-ALB**, לא מ-CIDR |
| **Reserved / Savings Plans על ה-minimum capacity** | ה-ASG לעולם לא יורד מתחת ל-min — זה עומס **קבוע** וידוע. שאר ה-capacity ב-On-Demand |

**למה לא Elastic IP לכל instance?**
EIP קושרת את המשתמש ל-instance ספציפי. כשה-instance נופל או מוחלף — הקשר נשבר,
ואין שום מנגנון בריאות. בנוסף, EIP מחייבת IP ציבורי לכל שרת — יקר ומיותר.

**למה לא Route 53 עם A Records לכל השרתים?**
כי **TTL**. אם ה-TTL הוא שעה ושרת נפל, לקוחות ימשיכו לפנות אליו עד שעה.
DNS הוא לא health check. ELB כן.

**מה זה עולה:**

| רכיב | חיוב |
|---|---|
| ALB | שעות + **LCU** (לפי חיבורים, בקשות ורוחב פס) |
| EC2 ב-ASG | לפי שעה/שנייה × מספר ה-instances בפועל |
| Route 53 Alias ל-ALB | **חינם** — Alias לשירות AWS לא מחויב על שאילתות |
| Data transfer OUT | GB לאינטרנט |
| **Cross-AZ** בין ALB ל-EC2 | GB — ההיפוך של Multi-AZ |

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה בארכיטקטורה |
|---|---|
| "משתמשים בכל העולם, latency נמוך" | מוסיפים **CloudFront** לפני ה-ALB, או **Global Accelerator** לתעבורה לא-HTTP |
| "התעבורה spiky ובלתי צפויה לחלוטין" | עוברים ל-**API Gateway + Lambda** — אפס עלות ב-idle |
| "צריך IP סטטי ללקוחות שמגדירים firewall" | **NLB** (יש לו IP סטטי ל-AZ) או **Global Accelerator** (2 IPs אנייקסט) |
| "אסור בכלל downtime בפריסת גרסה" | ASG עם **Rolling with additional batch** או deployment כחול/ירוק |

---

### 3.2 תרחיש B — אפליקציה stateful עם sessions

**הדרישה:** אתר מסחר עם **עגלת קניות**. מאות משתמשים במקביל.
המשתמש לא יאבד את העגלה, והפרטים שלו (כתובת, שם) נשמרים ב-DB.

**אילוצים:**

- חייבים לשמור **scalability אופקית** — לא לקשור משתמש לשרת.
- **אסור** לאבד את העגלה כששרת מתחלף.
- קריאות ל-DB רבות בהרבה מכתיבות.
- כשל AZ לא מפיל את האתר.

**ההתפתחות — ארבע אפשרויות ל-session:**

| # | הגישה | איך | הבעיה |
|---|---|---|---|
| 1 | **ELB Stickiness** | ה-LB שולח את המשתמש תמיד לאותו שרת | השרת נופל → **העגלה אבדה**. גם עומס לא מתפזר |
| 2 | **Cookies בצד הלקוח** | שולחים את תוכן העגלה ב-cookie | בקשות HTTP **כבדות יותר**; **סיכון אבטחה** (אפשר לשנות cookie); חייבים **לאמת** את התוכן; מוגבל ל-**פחות מ-4KB** |
| 3 | **Server Session ב-ElastiCache** ✅ | ב-cookie יושב רק **session_id**; הדאטה ב-Redis | הפתרון הנכון |
| 4 | **Server Session ב-DynamoDB** ✅ | אותו רעיון, DynamoDB במקום Redis | חלופה לגיטימית; latency גבוה יותר מזיכרון |

```text
                    Route 53 → ALB (2–3 AZs)
                              │
                    ┌─────────▼─────────┐
                    │  ASG: EC2 × N     │  Private subnets, Multi-AZ
                    └──┬────────────┬───┘
      session_id       │            │      user data
           ┌───────────▼──┐      ┌──▼──────────────────┐
           │ ElastiCache  │      │  RDS Multi-AZ        │
           │ Redis Multi-AZ│      │  (writes → master)  │
           │ + cached data│      │       │              │
           └──────────────┘      │       ▼ replication  │
                                 │  Read Replicas       │
                                 └──────────────────────┘
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **session ב-ElastiCache Redis** | ה-EC2 חוזר להיות **stateless** — כל שרת יכול לשרת כל בקשה. Redis נותן latency של **תת-מילישנייה** |
| ב-cookie רק **session_id** | Cookie קטן, לא רגיש, ולא ניתן לזייף ממנו דאטה עסקי |
| **Redis** ולא Memcached ל-sessions | Redis תומך ב-**persistence, replicas ו-Multi-AZ** — session לא נעלם בנפילת node |
| **RDS** לדאטה של המשתמש | כתובת ושם הם דאטה יחסי עם schema — זה SQL קלאסי |
| **RDS Multi-AZ** | failover אוטומטי; זו דרישת ה-**זמינות** |
| **Read Replicas** | האתר read-heavy. ה-replicas מפזרות קריאות בלי לגעת ב-master |
| **ElastiCache גם כ-cache ל-RDS** (Lazy Loading) | קריאה שנמצאת ב-cache לא מגיעה ל-DB בכלל. **hit** → מהיר וזול; **miss** → קוראים מה-DB וכותבים ל-cache |
| **שרשרת SG-to-SG** | ה-ALB פתוח ל-`0.0.0.0/0` ב-HTTP/S · ה-EC2 מקבל **רק מה-SG של ה-ALB** · ה-RDS וה-ElastiCache מקבלים **רק מה-SG של ה-EC2** |

**למה לא ELB Stickiness?**
היא פותרת את הסימפטום ולא את הבעיה. השרת עדיין מחזיק state, ולכן:
נפילת שרת = אובדן עגלה, וה-ASG לא יכול לאזן עומס באמת. **Stickiness היא פתרון זמני, לא ארכיטקטורה.**

**למה לא לשים את כל העגלה ב-cookie?**
שלוש סיבות שהקורס מונה: הבקשות נהיות **כבדות**, זהו **סיכון אבטחה** כי אפשר לשנות cookie
בצד הלקוח (ולכן חובה **לאמת** אותו בשרת), והמגבלה של **פחות מ-4KB** לא תספיק לעגלה גדולה.

**למה לא Read Replica במקום Multi-AZ?**
הן פותרות בעיות שונות. **Read Replica = scaling של קריאות**, ה-failover אליה **ידני**.
**Multi-AZ = זמינות**, ה-standby **לא משרת קריאות** אבל עושה failover אוטומטי.
כאן צריך את **שניהם**.

**מה זה עולה:**

| רכיב | חיוב | הערה |
|---|---|---|
| ElastiCache | לפי node/שעה × מספר nodes | Multi-AZ מכפיל |
| RDS Multi-AZ | **פי ~2** מ-Single-AZ | משלמים על standby שלא משרת תעבורה |
| Read Replica | תשלום מלא לכל replica | כל אחת היא instance נוסף |
| Cross-AZ traffic | GB | EC2 ב-AZ אחת ל-cache באחרת |

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה |
|---|---|
| "sessions בקנה מידה עצום, serverless" | **DynamoDB** עם **TTL** אוטומטי לניקוי sessions ישנות |
| "ה-DB עדיין עמוס בכתיבות" | Read Replicas לא עוזרות. צריך **sharding**, **Aurora**, או להעביר חלק מהדאטה ל-DynamoDB |
| "צריך מספר Regions" | **Aurora Global Database** או **DynamoDB Global Tables** + Route 53 latency routing |
| "העגלה חייבת לשרוד גם נפילת Region" | DynamoDB Global Tables ל-sessions; ElastiCache הוא Regional |

---

### 3.3 תרחיש C — WordPress עם קבצים משותפים

**הדרישה:** אתר WordPress שמתרחב אופקית. משתמשים מעלים תמונות,
והתמונות חייבות להופיע **בכל שרת**. התוכן והמשתמשים ב-MySQL.

**אילוצים:**

- **כל instance** באתר חייב לראות את **כל התמונות**, מיד.
- ה-DB חייב zמינות גבוהה וקריאות מהירות.
- האפליקציה היא WordPress — **לא משנים את הקוד**.

**ההתפתחות של שכבת האחסון:**

```text
ניסיון 1 — EBS על instance אחד
   EC2(AZ-a) ──► EBS
   ✅ עובד עם שרת אחד
   ❌ EBS משרת instance אחד בכל רגע, וקשור ל-AZ

ניסיון 2 — EBS לכל instance
   EC2(AZ-a) ──► EBS-1        EC2(AZ-b) ──► EBS-2
   ❌ תמונה שהועלתה ל-AZ-a לא קיימת ב-AZ-b. משתמשים רואים 404 באקראי

פתרון — EFS
   EC2(AZ-a) ──ENI──┐
                    ├──► EFS  (Region-scope, נגיש מכל AZ)
   EC2(AZ-b) ──ENI──┘
```

```text
              Route 53 → ALB (Multi-AZ)
                        │
         ┌──────────────┴──────────────┐
    ┌────▼─────┐                  ┌────▼─────┐
    │ EC2 AZ-a │                  │ EC2 AZ-b │   ← ASG
    └──┬────┬──┘                  └──┬────┬──┘
       │    └──── ENI ──┐   ┌── ENI ─┘    │
       │               ┌▼───▼┐            │
       │               │ EFS │  ← התמונות │
       │               └─────┘            │
       │                                  │
       └──────► Aurora MySQL ◄────────────┘
                Multi-AZ + Read Replicas
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **EFS** ולא EBS לתמונות | EFS הוא **NFS ברמת Region** — כל ה-instances בכל ה-AZs רואים את **אותם קבצים** |
| **Aurora MySQL** ולא RDS MySQL | Multi-AZ ו-Read Replicas פשוטים בהרבה להקמה ולניהול, וביצועים גבוהים יותר |
| **Multi-AZ + Read Replicas** | Multi-AZ ל-**זמינות**, Replicas ל-**קריאות**. בלוג הוא read-heavy קיצוני |
| ASG על **2+ AZs** | הבסיס לכל ארכיטקטורה עמידה |
| **ENI לכל AZ** ל-EFS | כך EFS נגיש מקומית מכל AZ |

**למה לא EBS Multi-Attach?**
Multi-Attach קיים ב-io1/io2, אבל **רק בתוך אותה AZ** ודורש **cluster-aware filesystem**.
WordPress לא כזה — שני שרתים שכותבים במקביל ישחיתו את המערכת הקבצים.

**למה לא S3 לתמונות?**
זו למעשה **התשובה הטובה יותר** מבחינת עלות וסקלביליות — אבל היא דורשת **תוסף או שינוי קוד**
ב-WordPress כדי לכתוב ל-S3 API במקום ל-filesystem. אם השאלה אומרת **"without changing the application"** —
**EFS**. אם היא אומרת **"most cost-effective"** ולא מגבילה שינויים — **S3 + CloudFront**.

**מה זה עולה:**

| רכיב | חיוב | הערה |
|---|---|---|
| EFS | לפי **GB מאוחסן**; **יקר** יחסית ל-GB | EFS-IA ו-lifecycle מורידים משמעותית |
| Aurora | לפי instance + **אחסון שגדל אוטומטית** + I/O | Read Replica = instance נוסף במלוא המחיר |
| ALB + EC2 | כרגיל | |

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה |
|---|---|
| "מותר לשנות את האפליקציה, רוצים עלות מינימלית" | **S3** לתמונות + **CloudFront** להגשה. הרבה יותר זול מ-EFS |
| "רוב התמונות לא נצפות אחרי חודש" | **EFS Lifecycle Management** ל-EFS-IA, או S3 Lifecycle ל-IA/Glacier |
| "האתר גלובלי" | **CloudFront** לפני הכל + **Aurora Global Database** |
| "האתר קורא הרבה יותר ממה שהוא כותב, ה-DB עמוס" | **ElastiCache** לפני Aurora, ותוסף caching ב-WordPress |

---

### 3.4 תרחיש D — עיבוד תמונות serverless

**הדרישה:** משתמשים מעלים תמונות. לכל תמונה צריך **thumbnail** אוטומטי.
בנוסף, כל משתמש חדש שנרשם מקבל **מייל ברוכים הבאים**.

**אילוצים:**

- **serverless מלא** — אין שרתים לנהל.
- כמות ההעלאות בלתי צפויה לחלוטין.
- אסור לאבד תמונה גם אם העיבוד נכשל.

```text
                    Client
                      │ upload (Transfer Acceleration אופציונלי)
                      ▼
              ┌───────────────┐
              │  S3 — uploads │
              └───────┬───────┘
                      │ S3 Event Notification
                      │ (s3:ObjectCreated:*, filter: *.jpg)
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
     Lambda          SQS           SNS
   (thumbnail)   (עיבוד עמיד)   (התראות)
        │
        ▼
  ┌──────────────┐        ┌──────────────────┐
  │ S3 —thumbnails│──OAC──►│   CloudFront     │──► Client
  └──────────────┘        └──────────────────┘

  ── זרימת המייל ──
  DynamoDB (users) ──Stream──► Lambda ──IAM Role──► SES ──► המשתמש
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **S3 Event Notifications** | S3 יכול להפעיל **Lambda, SQS או SNS** ישירות. אין צורך ב-polling |
| **סינון לפי שם אובייקט** (`*.jpg`) | ה-event מופעל רק על מה שרלוונטי — חוסך הפעלות מיותרות |
| **SQS בין S3 ל-Lambda** כשצריך עמידות | אם ה-Lambda נכשלת, ההודעה חוזרת לתור ואז ל-**DLQ**. בלי תור — האירוע אבד |
| **DynamoDB Streams → Lambda → SES** | הרשמה = כתיבה ל-DynamoDB. ה-Stream מפעיל Lambda שמשתמשת ב-**IAM Role** לשליחת מייל דרך **SES** |
| **CloudFront + OAC** על ה-thumbnails | ה-bucket נשאר **פרטי**; ה-bucket policy מתירה גישה **רק מה-distribution** |
| **S3 Transfer Acceleration** להעלאות | משתמשים רחוקים מעלים דרך edge location במקום ישירות ל-Region |
| **DAX** לפני DynamoDB (אם צריך) | קריאות ב-**microseconds** לדאטה שנקרא הרבה ומשתנה מעט |
| **API Gateway caching** | תשובות זהות לא מגיעות ל-Lambda בכלל |

**למה לא EC2 עם cron שסורק את ה-bucket?**
זה polling — משלמים 24/7 גם כשאין העלאות, יש עיכוב עד הסריקה הבאה,
וצריך לנהל scaling ותחזוקה. **S3 Events דוחפים** ומשלמים רק לפי הפעלה.

**למה לא לחבר את ה-Lambda ישירות בלי SQS?**
לרוב זה בסדר — ל-Lambda יש retries מובנים ו-DLQ. אבל אם הדרישה היא
**אסור לאבד אף אירוע** או שיש **burst** שעולה על ה-concurrency, ה-SQS הוא ה-buffer שסופג.

**למה לא SNS לבד ל-fan-out לכמה מעבדים?**
SNS דוחף ו**לא שומר**. אם מנוי נפל — ההודעה אליו אבדה.
**SNS + SQS Fan-Out**: כל מנוי מקבל תור משלו, וההודעות ממתינות בו.

**מה זה עולה:**

| רכיב | חיוב | הערה |
|---|---|---|
| Lambda | **לכל בקשה** + משך × זיכרון | אפס ב-idle |
| S3 | GB + **PUT/GET requests** | ב-scale גבוה ה-requests משמעותיים |
| SQS/SNS | לכל מיליון בקשות | זול מאוד |
| CloudFront | requests + **data out** | חוסך egress מ-S3 |
| SES | לכל 1,000 מיילים | |
| DAX | לפי node/שעה | **לא serverless** — משלמים גם ב-idle |

> [!warning] נקודת עדכניות
> S3 Event Notifications מגיעים **תוך שניות**, אבל **לפעמים דקה או יותר**.
> אם השאלה דורשת התראה מיידית קשיחה — זה שיקול.

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה |
|---|---|
| "צריך סינון מתקדם לפי metadata או גודל אובייקט" | **EventBridge** במקום S3 Events — כללי JSON, יותר מ-18 יעדים, ו-**Archive/Replay** |
| "העיבוד לוקח 40 דקות" | **לא Lambda** (תקרת 15 דקות) → **Fargate** או **AWS Batch** |
| "צריך שרשרת של 6 שלבים עם תנאים ו-retries" | **Step Functions** |
| "אסור בהחלט שאירוע יאבד, נדרש audit" | EventBridge (delivery אמין + Archive) או SQS עם DLQ ו-idempotency |

---

### 3.5 תרחיש E — pipeline של אירועים בנפח גבוה

**הדרישה:** מיליוני אירועים בשנייה ממכשירי IoT. צריך: ניתוח בזמן אמת,
ארכיון ב-S3 לניתוח היסטורי, והתראה על אירועים חריגים.

**אילוצים:**

- **סדר האירועים לכל מכשיר** חייב להישמר.
- כמה צוותים שונים צריכים לקרוא את **אותו זרם** באופן עצמאי.
- צריך יכולת **להריץ מחדש** עיבוד על דאטה של אתמול אחרי תיקון באג.
- אסור לאבד אירועים.

```text
   IoT Devices ──► API Gateway ──►  Kinesis Data Streams
                  (Service                   │  (partition key = device_id)
                   Integration)              │
                     ┌─────────────┬─────────┴─────────┬──────────────┐
                     ▼             ▼                   ▼              ▼
              Kinesis Firehose  Lambda            Analytics app   Team C app
                     │        (real-time)              │              │
                     ▼                                 ▼              ▼
                  S3 (.json)                    CloudWatch/alerts   ...

   ── בקרת שינויים בתשתית ──
   כל API Call ──► CloudTrail ──► EventBridge (rule) ──► SNS ──► alert
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **Kinesis Data Streams** ולא SQS | צריך **סדר לפי מכשיר** (partition key), **כמה consumers על אותו דאטה**, ו-**replay**. SQS לא נותן אף אחד מהשלושה |
| **partition key = device_id** | מבטיח שכל האירועים של אותו מכשיר נכנסים לאותו shard ונקראים **בסדר** |
| **API Gateway עם Service Integration** | מקבל את הבקשות ומעביר ישירות ל-Kinesis — **בלי Lambda באמצע**. פחות רכיבים, פחות latency, פחות עלות |
| **Kinesis Data Firehose → S3** | Firehose צובר ומטעין אוטומטית ל-S3 בפורמט `.json`. מנוהל לחלוטין, ללא קוד |
| **EventBridge על CloudTrail** | כל **API Call** נרשם ב-CloudTrail; EventBridge יכול **ליירט** אותו ולשלוח התראה. לדוגמה: `DeleteTable` על DynamoDB → SNS |
| **SNS להתראות** | fan-out לצוותים, למייל, ל-Lambda שמבצע תגובה אוטומטית |

**למה לא SQS?**
ב-SQS ההודעה **נמחקת** אחרי עיבוד, **consumer אחד** מקבל אותה, ואין **replay**.
כאן צריך שכמה צוותים יקראו את אותו דאטה ושאפשר יהיה לחזור אחורה.

**למה לא EventBridge לזרם עצמו?**
EventBridge מצוין ל-**ניתוב אירועים** לפי תוכן, לאינטגרציות SaaS ולתזמון —
אבל הוא לא stream עם shards, לא שומר סדר לפי key, ולא נועד למיליוני רשומות בשנייה.
כאן הוא משמש ל-**בקרה** (יירוט API calls), לא ל-data plane.

**למה לא Lambda ישירות מ-API Gateway ל-S3?**
זה יעבוד לנפח נמוך, אבל: אין buffering, אין סדר, כל אירוע הוא קובץ נפרד ב-S3
(מיליוני אובייקטים זעירים = יקר ואיטי), ואין דרך לשרת כמה consumers.

**מה זה עולה:**

| רכיב | חיוב | הערה |
|---|---|---|
| Kinesis Data Streams | לפי **shard/שעה** + PUT payload units | **משלמים על shards גם ב-idle** (במצב Provisioned) |
| Kinesis Firehose | לפי **GB שהוטען** | אין shards לנהל |
| S3 | GB + requests | Firehose מקבץ ומקטין את מספר האובייקטים |
| EventBridge | לפי מיליון אירועים מותאמים | אירועי AWS עצמם — חינם |

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה |
|---|---|
| "הנפח בלתי צפוי לגמרי" | **Kinesis On-Demand** במקום Provisioned — לא מנהלים shards |
| "צריך רק להעביר ל-S3, בלי consumers נוספים" | **Firehose לבדו** — פשוט וזול יותר |
| "סדר לא חשוב, רק לא לאבד ולעבד פעם אחת" | **SQS** — פשוט וזול משמעותית |
| "צריך שאילתות SQL על הדאטה ההיסטורי ב-S3" | **Athena** מעל ה-S3, או **Redshift** ל-BI כבד |

---

### 3.6 תרחיש F — אפליקציה hybrid עם on-premises

**הדרישה:** אפליקציה בענן שצריכה לגשת ל-ERP שנשאר ב-Data Center.
בנוסף, שרתים מקומיים צריכים לכתוב קבצים שיישמרו ב-S3.

**אילוצים:**

- התעבורה בין הענן ל-DC חייבת להיות **פרטית** — לא מעל האינטרנט הפתוח.
- ה-throughput צריך להיות **צפוי ויציב**.
- השרתים המקומיים עובדים ב-**SMB** ולא ישונו.
- הדאטה ההיסטורי (50 TB) צריך לעבור פעם אחת.

```text
  Corporate Data Center                    AWS Region
  ─────────────────────                    ──────────
  ERP Server ◄──────┐
                    │
  App Servers       │        ┌──── Direct Connect ────┐
   │ SMB            │        │  (+ Site-to-Site VPN   │
   ▼                └────────┤     כגיבוי)            ├──► VPC
  S3 File Gateway ───────────┘                        │     └► EC2 / Lambda
   │  local cache                                     │          │
   └──── HTTPS ─────────────────────────────────────► S3 ◄───────┘
                                                       ▲
  50 TB היסטורי ──► Snowball Edge ─────────────────────┘
  דאטה שוטף ─────► DataSync (יומי) ────────────────────┘
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **Direct Connect** | חיבור **פרטי** עם **throughput צפוי** — בדיוק שתי הדרישות |
| **Site-to-Site VPN כגיבוי** | DX הוא קו יחיד. VPN מעל האינטרנט הוא גיבוי זול שנכנס לפעולה מיד |
| **S3 File Gateway** | חושף את ה-bucket כ-**SMB share**. השרתים לא משתנים, וה-**cache המקומי** שומר latency נמוך |
| **Snowball ל-50 TB ההיסטוריים** | העברה חד-פעמית של 50TB דרך הקו תחנוק אותו לשבועות |
| **DataSync לדאטה השוטף** | מתוזמן, **משמר הרשאות ו-metadata**, ולא דורש שינוי בתהליכים |
| **VPC Endpoints** לשירותי AWS | גם התעבורה בתוך ה-VPC נשארת פרטית |

**למה לא רק VPN?**
VPN עובר מעל האינטרנט הציבורי — ה-throughput וה-latency **לא צפויים**.
הדרישה מדברת על "יציב וצפוי". VPN מצוין כ-**גיבוי**, לא כפתרון יחיד לתעבורה קריטית.

**למה לא DataSync לכל 50 ה-TB?**
כלל האצבע: אם ההעברה לוקחת **יותר משבוע** — Snowball.
בנוסף, זה יחנוק את ה-DX שדרוש לתעבורה השוטפת.

**למה לא להעביר את ה-ERP לענן?**
כי בשאלה זה **אילוץ**, לא בחירה. תפקיד הארכיטקט הוא לבנות סביב האילוץ.

**מה זה עולה:**

| רכיב | חיוב | הערה |
|---|---|---|
| Direct Connect | port/שעה + **data transfer OUT** בתעריף מוזל | ה-setup לוקח **יותר מחודש** |
| Site-to-Site VPN | לשעת חיבור + data transfer | זול ומיידי |
| Storage Gateway | לפי **GB שנכתב ל-AWS** + עלות ה-S3 + requests | |
| DataSync | לפי **GB שהועבר** | |
| Snowball | לפי job + ימי החזקה + משלוח | **Data IN חינם** |

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה |
|---|---|
| "צריך לחבר גם 12 סניפים, לא רק DC אחד" | **Transit Gateway** כ-hub, במקום peering מרובה |
| "יש תהליך גיבוי לקלטות פיזיות" | **Tape Gateway** (VTL) — התהליך נשאר, הקלטות ב-S3/Glacier |
| "צריך גם volumes ברמת בלוקים" | **Volume Gateway** — Cached או Stored לפי איפה יושב ה-dataset |
| "הדאטה **אסור** שיעזוב את המתקן" | **AWS Outposts** — אף פתרון רשת לא עונה על זה |
| "שותפים חיצוניים צריכים להעלות קבצים ב-SFTP" | **Transfer Family** |

---

### 3.7 תרחיש G — ארכיטקטורת DR ל-RTO נמוך

**הדרישה:** מערכת פיננסית קריטית. **RTO של 15 דקות, RPO של דקה**.
צריך לשרוד אובדן Region שלם.

**אילוצים:**

- RTO ≤ 15 דקות, RPO ≤ דקה — אלה **דרישות קשיחות**.
- העלות חשובה, אבל **משנית לדרישות**.
- ה-failover צריך להיות **אוטומטי ככל האפשר**.

```text
   Route 53 — Health Check + Failover Routing
        │
   ┌────┴─────────────────────────────┐
   ▼ PRIMARY                          ▼ SECONDARY (warm)
 Region A                          Region B
 ┌──────────────────┐              ┌──────────────────┐
 │ ALB              │              │ ALB              │
 │ ASG: EC2 × 10    │              │ ASG: EC2 × 2     │ ← קנה מידה מוקטן, אבל רץ
 │ Aurora (writer)  │───Global────►│ Aurora (reader)  │ ← RPO של שניות
 │ S3 bucket        │───CRR───────►│ S3 bucket        │
 └──────────────────┘              └──────────────────┘
                                    ASG max גדול — מתנפח ב-failover
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **Warm Standby** ולא Pilot Light | Pilot Light דורש הקמת שכבת האפליקציה בזמן האסון — לא נכנס ב-15 דקות. Warm Standby כבר רץ |
| **Aurora Global Database** | רפליקציה בין Regions ב-latency נמוך; ה-RPO נמדד ב-**שניות** ועומד בדרישת הדקה |
| **S3 Cross-Region Replication** | הקבצים זמינים ב-Region המשני אוטומטית |
| **Route 53 Health Check + Failover Routing** | מזהה שה-primary לא מגיב ומעביר את הרשומה ל-secondary **אוטומטית** |
| **TTL נמוך** על הרשומה | TTL גבוה = לקוחות ממשיכים לפנות ל-Region שנפל |
| ASG משני עם **min קטן ו-max גדול** | משלמים על 2 instances ביום-יום, אבל ה-ASG יכול להתנפח מיד ב-failover |
| **תרגול failover יזום** (game day) | DR שלא נבדק הוא הנחה, לא תוכנית |

**למה לא Backup & Restore?**
RTO של שעות עד ימים. **נפסל מיד** מול דרישת 15 הדקות.

**למה לא Multi-Site Active-Active?**
הוא **עומד** בדרישה — RTO ו-RPO כמעט אפס. אבל הוא גם **היקר ביותר**:
שתי סביבות בגודל מלא, וכתיבות דו-כיווניות שמחייבות טיפול בקונפליקטים.
אם הדרישה היא 15 דקות ולא אפס — **Warm Standby הוא ה-BEST**: עומד בדרישה, בעלות נמוכה יותר.

**למה לא Multi-AZ?**
**Multi-AZ אינו DR.** הוא מגן מפני כשל AZ, לא מפני אובדן Region.

**מה זה עולה:**

| רכיב | חיוב | הערה |
|---|---|---|
| Aurora Global Database | instance משני מלא + **replicated write I/O** + cross-Region transfer | ההוצאה הגדולה |
| S3 CRR | אחסון כפול + **cross-Region transfer** לכל אובייקט | |
| ASG משני מוקטן | 2 instances במקום 10 | חיסכון מול Multi-Site |
| Route 53 Health Checks | לכל health check/חודש | זניח |

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה |
|---|---|
| "RTO של 24 שעות, עלות מינימלית" | **Backup & Restore** — AWS Backup עם copy ל-Region שני |
| "RTO של שעה" | **Pilot Light** — רק שכבת הדאטה רצה; ה-compute מוקם בעת הצורך |
| "RTO ו-RPO של אפס, וזה קריטי לעסק" | **Multi-Site Active-Active** + **DynamoDB Global Tables** או Aurora Global עם write forwarding |
| "צריך לעמוד ברגולציה שדורשת הוכחת התאוששות" | תרגול תקופתי מתועד + **AWS Backup** עם audit |

---

### 3.8 תרחיש H — HPC (מחשוב עתיר ביצועים)

**הדרישה:** סימולציות מדעיות. מאות nodes שמתקשרים בצפיפות (MPI),
צורך במערכת קבצים במיליוני IOPS. העבודות רצות בגלים.

**אילוצים:**

- **latency מינימלי בין ה-nodes** — זה workload **tightly coupled**.
- מערכת קבצים משותפת ב-**מיליוני IOPS**.
- העבודות **ניתנות להרצה מחדש** אם נקטעו.
- הדאטה הגולמי כבר ב-S3.

```text
        S3 (דאטה גולמי)
             │ lazy load / export
             ▼
      ┌── FSx for Lustre ──┐   ← מיליוני IOPS, מגובה S3
      │                    │
 ┌────┴────────────────────┴────┐
 │  Cluster Placement Group     │  ← אותו rack, אותה AZ, רשת 10Gbps
 │   EC2 ── EC2 ── EC2 ── EC2   │  ← CPU/GPU optimized, Spot
 │      └──── EFA (MPI) ────┘   │  ← עוקף את ה-OS, latency נמוך
 └──────────────┬───────────────┘
                │
          AWS Batch  (multi-node parallel jobs)
                או
          AWS ParallelCluster (מקבצי קונפיגורציה)
```

**טבלת החלטות:**

| החלטה | למה |
|---|---|
| **FSx for Lustre** | מערכת הקבצים המבוזרת ל-HPC — **מיליוני IOPS**, **מגובה S3** |
| **Cluster Placement Group** | כל ה-instances על **אותו rack באותה AZ** — latency נמוך ורשת 10Gbps |
| **EFA (Elastic Fabric Adapter)** | ENA משופר ל-HPC. **עוקף את ה-Linux OS** ונותן transport אמין ב-latency נמוך למשל **MPI**. **Linux בלבד** |
| **EC2 CPU/GPU optimized** | בוחרים משפחה שמתאימה ל-bottleneck בפועל |
| **Spot Instances / Spot Fleet** | ה-jobs ניתנים להרצה מחדש — זו ההגדרה של Spot. **עד ~90% הנחה** |
| **AWS Batch** | מנהל תור, מקצה instances, מכבה בסיום. תומך ב-**multi-node parallel jobs** |
| **ParallelCluster** כחלופה | כלי open-source לניהול cluster מקבצי טקסט; **מקים VPC, subnet, cluster ו-instance types** ומאפשר להפעיל EFA |
| **Direct Connect / Snowball / DataSync** להזרמת הדאטה | לפי הנפח: GB/s → DX, PB → Snow, סנכרון → DataSync |

**למה לא EFS?**
EFS מתרחב לפי הגודל הכולל (או provisioned), אבל הוא **לא מגיע למיליוני IOPS** של HPC
ואינו מגובה S3. הוא NFS למטרות כלליות, לא filesystem מבוזר ל-HPC.

**למה לא ENA רגיל במקום EFA?**
**Enhanced Networking (SR-IOV)** עם **ENA** נותן עד **100 Gbps** — מצוין ל-throughput.
אבל ל-**inter-node communication** ב-workload tightly coupled צריך את ה-latency הנמוך של **EFA**,
שעוקף את מערכת ההפעלה. (**Intel 82599 VF** — עד 10 Gbps — הוא **legacy**.)

**למה לא לפזר על AZs לזמינות?**
כי הדרישה הקשיחה כאן היא **latency בין nodes**, ו-cross-AZ הורס אותה.
ה-jobs ניתנים להרצה מחדש, ולכן אובדן AZ הוא לא אסון.

**מה זה עולה:**

| רכיב | חיוב | הערה |
|---|---|---|
| EC2 Spot | **עד ~90% פחות** מ-On-Demand | ההחלטה החוסכת ביותר כאן |
| FSx for Lustre | לפי GB מוקצה ו-throughput | מכבים אחרי ה-job |
| S3 | GB + requests | הדאטה הקבוע |
| Batch / ParallelCluster | **חינם** | משלמים רק על ה-compute מתחת |

**וריאציה — מה משתנה אם...**

| הדרישה משתנה ל... | מה משתנה |
|---|---|
| "ה-jobs **לא** ניתנים להרצה מחדש" | **On-Demand** או Spot עם checkpointing תכוף |
| "ה-nodes עצמאיים, לא מתקשרים ביניהם" | **לא צריך EFA ולא Cluster Placement Group**. Batch רגיל על Spot מספיק |
| "צריך IOPS מקסימלי ל-scratch מקומי" | **Instance Store** — מיליוני IOPS, נמחק עם ה-instance |
| "יש PB של דאטה on-prem להעלות" | **Snowmobile / Snowball**, ולתעבורה שוטפת **Direct Connect** |

---

## 4. 💰 עלות ותמחור — איפה הכסף בורח בכל דפוס

### על מה מחייבים בכל שכבה

| שכבה | רכיב החיוב העיקרי | המלכודת |
|---|---|---|
| **DNS** | שאילתות ב-Route 53 | **Alias לשירות AWS = חינם**. A record רגיל מחויב |
| **Edge** | CloudFront: requests + **data out** | מוזיל את ה-egress הכולל, אבל מוסיף שכבה |
| **LB** | ALB/NLB: שעות + **LCU** | LB בלי targets עדיין מחויב |
| **Compute** | שעות EC2 / GB-שנייה ב-Lambda | Lambda יקרה יחסית בעומס **קבוע** גבוה |
| **Data** | RDS instance + storage + I/O | **Multi-AZ מכפיל**; כל Read Replica היא instance מלא |
| **Cache** | node/שעה | **לא serverless** — משלמים גם ב-idle. **DAX** גם כן |
| **Files** | EFS לפי GB — **יקר יחסית** | Lifecycle ל-EFS-IA מוריד משמעותית |
| **Async** | לכל מיליון בקשות | Kinesis ב-**Provisioned** מחייב על shards גם ב-idle |
| **Network** | NAT: שעות + **GB מעובד** · **Cross-AZ** | ל-S3/DynamoDB — **Gateway Endpoint בחינם** |

### מה זול ומה יקר

| דפוס | עלות יחסית | מתי משתלם |
|---|---|---|
| **S3 + CloudFront** (סטטי) | **הזול ביותר** | כל תוכן שלא משתנה |
| **Serverless** (API GW + Lambda + DynamoDB) | **0 ב-idle** | תעבורה spiky או נמוכה |
| **ALB + ASG + RDS** (3-tier) | בינונית | עומס יציב, אפליקציה קיימת |
| **EFS** לקבצים משותפים | **גבוהה ל-GB** | רק כשחייבים POSIX משותף |
| **Multi-AZ + Read Replicas** | פי 2–3 מ-single | כשזמינות וקריאות הן דרישות |
| **Warm Standby cross-Region** | גבוהה | RTO של דקות |
| **Multi-Site Active-Active** | **הגבוהה ביותר** | RTO≈0 בלבד |
| **Spot ל-HPC/Batch** | **עד ~90% הנחה** | כל עומס שסובל הפסקה |

### 🚩 עלויות נסתרות

- **CloudFront לפני EC2 שמפיץ עדכוני תוכנה** — הדוגמה מהקורס: תוכן **סטטי** שמופץ בהמוניהם.
  CloudFront מטמיע אותו בקצה, ה-ASG כמעט לא מתרחב, וחוסכים **גם EC2 וגם רוחב פס** —
  **בלי לשנות שורת קוד באפליקציה**.
- **NAT Gateway כדי להגיע ל-S3** — הטעות היקרה והנפוצה ביותר.
- **Cross-AZ בין ALB ל-EC2 ובין EC2 ל-cache** — נגבה בכל בקשה.
- **Read Replicas מיותרות** — כל אחת היא instance במחיר מלא.
- **DAX ו-ElastiCache שרצים 24/7** לעומס שהוא רק ביום.
- **מיליוני אובייקטים זעירים ב-S3** — עלות ה-requests עולה על עלות האחסון.

### 💡 טיפים לחיסכון

- **Savings Plans / RI על ה-min capacity של ה-ASG** — זה עומס קבוע וידוע.
- **CloudFront לפני כל origin** שמגיש תוכן חוזר.
- **Gateway Endpoint** ל-S3 ו-DynamoDB, תמיד.
- **Spot** לכל מה שסובל הפסקה: batch, HPC, CI, עיבוד אסינכרוני.
- **Lifecycle** על S3 ועל EFS.
- **DynamoDB On-Demand** לעומס בלתי צפוי, **Provisioned** לעומס קבוע.

---

## 5. ⚖️ השוואות מכריעות

### שלושת דפוסי ה-web זה מול זה

| קריטריון | **Stateless + ASG** | **Stateful + Session Store** | **Serverless** |
|---|---|---|---|
| איפה ה-state | **אין** | ElastiCache / DynamoDB | DynamoDB / S3 |
| Scaling | ASG לפי מטריקה | ASG + scaling ל-cache ול-DB | אוטומטי לחלוטין |
| עלות ב-idle | משלמים על ה-min | משלמים על min + cache + DB | **אפס** |
| Operational overhead | בינוני | **הגבוה מהשלושה** | **הנמוך** |
| מתי | API מחושב, אתר סטטי דינמי | מסחר, פורטלים, אפליקציות קיימות | spiky, אירועים, mobile backend |

### איפה שומרים קבצים משותפים

| | **EFS** | **S3** | **EBS** | **FSx for Lustre** |
|---|---|---|---|---|
| כמה שרתים במקביל | **הרבה** | ללא הגבלה (API) | **אחד** | הרבה |
| ממשק | POSIX / NFS | HTTP API | בלוקים | POSIX מבוזר |
| דורש שינוי קוד | **לא** | **כן** | לא | לא |
| עלות ל-GB | **גבוהה** | **נמוכה** | בינונית | גבוהה |
| מתי | WordPress, אפליקציות legacy | תמונות, גיבויים, סטטי | boot volume, DB יחיד | HPC |

### SQS מול SNS מול EventBridge מול Kinesis

| קריטריון | **SQS** | **SNS** | **EventBridge** | **Kinesis** |
|---|---|---|---|---|
| כמה מקבלים הודעה | **אחד** | הרבה | הרבה, לפי rules | הרבה, במקביל |
| נשמר אחרי קריאה | לא | לא | לא | **כן** — retention |
| Replay | לא | לא | **כן** (Archive/Replay) | **כן** |
| סדר | FIFO בלבד | לא | לא | **לפי partition key** |
| מתי בשיעור | buffer מול Lambda | fan-out לכמה מנויים | סינון מתקדם, יירוט API calls | IoT, streaming, כמה consumers |

> [!info] שורה תחתונה
> **stateless → ASG. state → מוציאים אותו החוצה. קבצים משותפים → EFS (או S3 אם מותר לשנות קוד).
> אירוע אחד לכמה גורמים → SNS+SQS. זרם עם סדר ו-replay → Kinesis.**

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars על פני התרחישים

| Pillar | מה זה אומר **בתרחישים האלה** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | הסביבה ניתנת לשחזור ולניטור | **Golden AMI + User Data** ל-launch מהיר ועקבי; IaC לכל השכבות; CloudWatch alarms על ה-ASG, על ה-DLQ ועל replication lag; **game day** לתרגול failover |
| **Security** | כל שכבה מקבלת רק ממי שמעליה | **שרשרת SG-to-SG**: ALB←כולם, EC2←ALB, RDS/Cache←EC2; EC2 ב-private subnets; **OAC** ל-S3 מאחורי CloudFront; **IAM Roles** ולא credentials בקוד; Cognito ל-credentials זמניים לאפליקציות mobile |
| **Reliability** | כשל של רכיב או AZ לא מפיל את השירות | **2+ AZs** בכל שכבה; ELB Health Checks; RDS **Multi-AZ**; **SQS+DLQ** לעיבוד אסינכרוני; Route 53 Health Check ל-failover בין Regions |
| **Performance Efficiency** | הכלי מתאים לצורה של העומס | **cache בשכבה הנכונה**: CloudFront לקצה, API GW לתשובות, ElastiCache/DAX לדאטה; Read Replicas לקריאות; FSx Lustre ו-EFA ל-HPC; partition key נכון ב-Kinesis |
| **Cost Optimization** | לא משלמים על idle ולא על תעבורה מיותרת | **RI/SP על ה-min capacity**; **Spot** ל-batch ול-HPC; CloudFront לפני origin עמוס; Gateway Endpoints; lifecycle ל-S3 ול-EFS |
| **Sustainability** | פחות עבודה כפולה ופחות תעבורה | הטמעה בקצה במקום חישוב חוזר; scale-to-demand; לא לשמור עותקים מיותרים; מכבים סביבות בדיקה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "users lose their shopping cart when an instance is replaced" | ה-state על השרת → **ElastiCache** או **DynamoDB** ל-sessions |
| "images uploaded to one server are not visible on others" | **EFS** (או S3 אם מותר לשנות קוד) |
| "instance removed from the pool but users still reach it" | הסתמכות על **DNS TTL** → צריך **ELB + Health Checks** |
| "distributing large static files is expensive" | **CloudFront** — בלי לשנות את האפליקציה |
| "generate a thumbnail when an image is uploaded" | **S3 Event Notification → Lambda** |
| "advanced filtering on object metadata or size" | **EventBridge** במקום S3 Events |
| "welcome email when a user signs up" | **DynamoDB Streams → Lambda → SES** |
| "several teams need to read the same stream independently" | **Kinesis Data Streams** |
| "reprocess yesterday's data after a bug fix" | **Kinesis** (retention) או **EventBridge Archive/Replay** |
| "mobile users need direct, restricted access to their own S3 folder" | **Cognito** ל-credentials זמניים + IAM policy מוגבלת |
| "predictable, private bandwidth to the data center" | **Direct Connect** (+ VPN כגיבוי) |
| "RTO of 15 minutes across Regions" | **Warm Standby** |
| "tightly coupled nodes with MPI" | **EFA** + **Cluster Placement Group** |
| "developers just want their code to run" | **Elastic Beanstalk** |
| "scale workers based on the number of queued items" | Beanstalk **Worker Tier** או ASG על מטריקת **SQS ApproximateNumberOfMessages** |

### טעויות נפוצות

> [!warning] מלכודת 1 — Stickiness כפתרון ל-state
> **הניסוח:** "Enable ELB sticky sessions so users don't lose their cart."
> **הטעות:** לחשוב ש-stickiness פותר את הבעיה.
> **הנכון:** היא רק **דוחה** אותה. השרת עדיין מחזיק state, ולכן נפילת שרת = אובדן עגלה,
> והעומס לא מתפזר. הפתרון הוא **להוציא את ה-state החוצה** — ElastiCache או DynamoDB.

> [!warning] מלכודת 2 — EBS לקבצים משותפים
> **הניסוח:** "Attach the same EBS volume to all instances so they share uploads."
> **הטעות:** להניח ש-EBS הוא אחסון משותף.
> **הנכון:** **EBS משרת instance אחד בכל רגע** והוא קשור ל-**AZ**.
> לקבצים משותפים בין AZs — **EFS**.

> [!warning] מלכודת 3 — לענות רק על שכבה אחת
> **הניסוח:** תשובה שמציעה "RDS Multi-AZ" לשאלה על אתר שנופל.
> **הטעות:** לפתור את מה שמכירים.
> **הנכון:** לבדוק **את כל השכבות**. NAT ב-AZ אחת, ASG ב-AZ אחת, או תור בלי DLQ —
> כל אחד מהם הופך את הארכיטקטורה ללא-עמידה, גם אם ה-DB מושלם.

> [!warning] מלכודת 4 — Read Replica במקום Multi-AZ
> **הניסוח:** "Add a read replica for high availability."
> **הטעות:** לערבב בין scaling לזמינות.
> **הנכון:** **Read Replica** נועדה ל-**scaling של קריאות**, וה-failover אליה **ידני**.
> **Multi-AZ** נועדה ל-**זמינות** עם failover **אוטומטי**, וה-standby **לא משרת קריאות**.

> [!warning] מלכודת 5 — SNS לבד ל-fan-out עמיד
> **הניסוח:** "Use SNS to deliver events to three processing services."
> **הטעות:** להניח ש-SNS מבטיח שההודעה תגיע.
> **הנכון:** SNS **דוחף ולא שומר**. אם מנוי נפל — ההודעה אליו אבדה.
> ל-fan-out **עמיד** — **SNS + SQS**, תור לכל מנוי.

> [!warning] מלכודת 6 — S3 bucket ציבורי מאחורי CloudFront
> **הניסוח:** "Make the S3 bucket public so CloudFront can serve the files."
> **הטעות:** לחשוב ש-CloudFront דורש bucket ציבורי.
> **הנכון:** ה-bucket נשאר **פרטי**. משתמשים ב-**OAC (Origin Access Control)** עם
> **bucket policy** שמתירה גישה **רק מה-distribution**. אחרת אפשר לעקוף את CloudFront לגמרי.

> [!warning] מלכודת 7 — Snowball לדאטה השוטף
> **הניסוח:** "Ship a Snowball every week to sync the new files."
> **הטעות:** להשתמש בכלי החד-פעמי כפתרון קבוע.
> **הנכון:** Snowball הוא ל-**bulk חד-פעמי**. לסנכרון חוזר — **DataSync**.

---

## 8. 🏗️ Scenario מסכם — חנות גלובלית שמשלבת הכל

**הדרישה:**

חנות מקוונת עם משתמשים בשלוש יבשות. יש עגלת קניות, קטלוג עם תמונות,
עיבוד הזמנות אסינכרוני, וזרם אירועי התנהגות לניתוח. ה-ERP נשאר on-premises.
**RTO של 30 דקות** ברמת Region.

```text
                        Route 53  (latency routing + health checks)
                                    │
                        ┌───────────▼───────────┐
                        │      CloudFront       │  OAC → S3 (סטטי + תמונות)
                        └───────────┬───────────┘
                                    │ dynamic
                        ┌───────────▼───────────┐
                        │   ALB  (public, 3 AZ) │  + WAF
                        └───────────┬───────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │  ASG: EC2 (private, 3 AZ)     │
                    └──┬──────────┬────────────┬────┘
        sessions       │          │ orders     │ events
           ┌───────────▼──┐  ┌────▼─────┐  ┌───▼──────────────┐
           │ ElastiCache  │  │   SQS    │  │ Kinesis Streams  │
           │    Redis     │  └────┬─────┘  └───┬──────────────┘
           └──────────────┘       │            │
                    │             ▼            ▼
           ┌────────▼────────┐  Lambda      Firehose ──► S3 ──► Athena
           │ Aurora Global   │  workers
           │ Multi-AZ + RR   │     │
           └────────┬────────┘     └──► SNS ──► התראות
                    │ Global Replication
                    ▼
              Region B (Warm Standby)
                                    ▲
              ERP on-premises ───────┘  Direct Connect (+VPN backup)
```

**הפתרון וההנמקה:**

| שכבה | ההחלטה | למה |
|---|---|---|
| **DNS** | Route 53 **latency routing** + health checks | משתמשים מגיעים ל-Region הקרוב; health check מפעיל failover |
| **Edge** | CloudFront + **OAC** | תמונות וסטטי מהקצה; ה-bucket נשאר פרטי; חוסך egress ועומס origin |
| **הגנה** | **WAF** על ה-ALB (או על CloudFront) | חסימת IPs ותקיפות. **SG לא יכול לחסום** |
| **Compute** | ALB + ASG על **3 AZs**, EC2 ב-private | הבסיס. SG-to-SG בכל שכבה |
| **Sessions** | **ElastiCache Redis** | ה-EC2 נשארים stateless; latency תת-מילישנייה |
| **DB** | **Aurora Global** Multi-AZ + Read Replicas | Multi-AZ לזמינות, RR לקריאות, Global ל-DR ולקריאות אזוריות |
| **הזמנות** | **SQS + Lambda + DLQ** | ספיגת spikes; הזמנה לא אובדת; העיבוד מנותק מה-web |
| **אירועים** | **Kinesis → Firehose → S3 → Athena** | סדר לפי משתמש, כמה consumers, replay, וארכיון לשאילתות |
| **תמונות** | S3 + **S3 Event → Lambda** ל-thumbnails | serverless, אפס עלות ב-idle |
| **Hybrid** | **Direct Connect** + VPN backup ל-ERP | תעבורה פרטית וצפויה |
| **DR** | **Warm Standby** ב-Region B | RTO של 30 דקות — Pilot Light מסוכן, Multi-Site מיותר |

**למה לא הכל serverless?**
אפשר, וזו ארכיטקטורה לגיטימית. אבל אם השאלה מציינת **אפליקציה קיימת שלא משנים** —
ASG + ALB הוא ה-BEST. **הרכיבים האסינכרוניים** (SQS, Lambda, Kinesis) הם serverless בכל מקרה.

**למה לא Multi-Site Active-Active?**
RTO של **30 דקות** לא דורש אותו. Multi-Site הוא היקר ביותר ומוסיף מורכבות של כתיבות דו-כיווניות.
**Warm Standby עומד בדרישה בעלות נמוכה יותר** — ולכן הוא ה-BEST.

**מה עוד נדרש:**
ניטור ולוגים ([[31 - Monitoring and Logging]]), מדיניות גיבוי ([[35 - Backup and Data Protection]]),
ואופטימיזציית עלות אחרי הייצוב ([[37 - Cost Optimization]]).

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **דיאגרמה גרפית מושלמת.** נדרש להסביר את ה-flow, לא לצייר.
- **תחביר** של כל שירות — CLI, SDK או קבצי קונפיגורציה.
- **רשימת הפלטפורמות המלאה** של Elastic Beanstalk. מספיק לדעת שהיא רחבה
  וכוללת Docker בגרסאות single, multi ו-preconfigured.
- **פרטי ה-lifecycle hooks** של ASG. מספיק לדעת שיש hook ב-**launch** וב-**terminate**,
  ושאפשר לתלות בהם פעולות כמו יצירת volume מ-snapshot או יצירת snapshot לפני מחיקה.
- **המבנה הפנימי** של Kinesis shards או של EFA.
- **המחירים המדויקים.** רק **היחסים**.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **Stateless = ASG יכול להחליף כל שרת בכל רגע.** זו מטרת-העל של כל ארכיטקטורת web.
- **State יוצא החוצה:** sessions ל-**ElastiCache** או **DynamoDB**, קבצים ל-**EFS/S3**, דאטה ל-**RDS**.
- **Stickiness היא לא פתרון ל-state** — היא רק דוחה את הבעיה.
- **Cookies בצד הלקוח:** בקשות כבדות · ניתנים לזיוף · חובה לאמת · **פחות מ-4KB**.
- **Alias Record ל-ALB — חינם ומתעדכן אוטומטית.** A Record + TTL הוא מלכודת.
- **DNS הוא לא health check. ELB כן.**
- **EBS = instance אחד, AZ אחת. EFS = הרבה instances, כל ה-Region.**
- **תמונות: EFS אם אסור לשנות קוד · S3+CloudFront אם מותר** (וזה זול בהרבה).
- **Aurora** נותן Multi-AZ ו-Read Replicas בקלות — לכן הוא ברירת המחדל ל-MySQL/PostgreSQL מנוהל.
- **Multi-AZ = זמינות (failover אוטומטי). Read Replica = scaling של קריאות (failover ידני).**
- **שרשרת SG-to-SG:** ALB←העולם · EC2←SG של ALB · RDS/Cache←SG של EC2.
- **S3 Events מפעילים Lambda, SQS או SNS.** סינון לפי שם אובייקט (`*.jpg`) אפשרי.
- **צריך סינון לפי metadata/גודל, יעדים רבים, Archive ו-Replay → EventBridge.**
- **EventBridge יכול ליירט כל API Call דרך CloudTrail** — לדוגמה `DeleteTable` → SNS.
- **DynamoDB Streams → Lambda → SES** הוא דפוס המייל הסטנדרטי.
- **Fan-Out עמיד = SNS + SQS.** SNS לבדו לא שומר הודעות.
- **Kinesis הוא היחיד עם סדר לפי partition, כמה consumers מקבילים, ו-replay.**
- **API Gateway יכול לפנות ישירות ל-Kinesis** — בלי Lambda באמצע.
- **Cognito** נותן credentials זמניים כדי שאפליקציית mobile תיגש ישירות ל-S3/DynamoDB.
- **CloudFront + OAC** = origin פרטי. אף פעם לא bucket ציבורי.
- **CloudFront לפני EC2 שמפיץ תוכן סטטי** — חוסך EC2 ורוחב פס **בלי לשנות קוד**.
- **Golden AMI** להתקנה מראש · **User Data** לקונפיגורציה דינמית · **Beanstalk** = שניהם.
- **Beanstalk:** Application → Version → Environment. **Web Tier** מול **Worker Tier** (מתרחב לפי הודעות SQS).
  **Beanstalk עצמו חינם** — משלמים על המשאבים.
- **EC2 יחיד "זמין גבוה":** ASG עם min=max=desired=**1** על **2+ AZs**, +
  **User Data שמצמיד את ה-EIP** + IAM Role שמרשה את קריאת ה-API.
  לשמירת דאטה — **lifecycle hooks**: snapshot ב-terminate, יצירת volume מ-snapshot ב-launch.
- **HPC:** Cluster Placement Group + **EFA** (MPI, Linux) + **FSx for Lustre** + **Spot** + **Batch/ParallelCluster**.
- **DR:** Backup&Restore(שעות) · Pilot Light(עשרות דקות) · **Warm Standby(דקות)** · Multi-Site(≈0).

---

## 11. ✅ בדיקת הבנה

1. אתר מאבד עגלות קניות בכל פעם ש-instance מתחלף. מהו הפתרון, ולמה לא stickiness?
2. תמונות שמועלות לשרת אחד לא נראות בשרתים אחרים. שתי אפשרויות — ומה מכריע ביניהן?
3. למה Alias Record עדיף על A Record עם TTL של שעה מול ALB?
4. איך מייצרים thumbnail אוטומטית, ומתי מוסיפים SQS באמצע?
5. ה-DB עמוס בקריאות והאתר איטי. אילו שתי אפשרויות יש ומה ההבדל?
6. אפליקציה מפיצה עדכוני תוכנה מ-EC2 והחשבון מזנק. מה משנים **בלי לגעת באפליקציה**?
7. למה Kinesis ולא SQS כשכמה צוותים צריכים לקרוא את אותם אירועים?
8. מהי שרשרת ה-Security Groups בארכיטקטורה תלת-שכבתית?
9. RTO של 15 דקות ברמת Region. איזו אסטרטגיית DR, ולמה לא השכנות שלה?
10. איך הופכים EC2 יחיד עם Elastic IP ל"זמין גבוה" בלי להוסיף שרתים?

<details>
<summary>תשובות</summary>

1. מוציאים את ה-session **החוצה מהשרת** — ל-**ElastiCache Redis** (או **DynamoDB**),
   וב-cookie נשאר רק **session_id**. Stickiness לא פותרת: השרת עדיין מחזיק את ה-state,
   ולכן נפילתו עדיין מוחקת את העגלה, והעומס גם לא מתפזר.
2. **EFS** — קבצים משותפים ברמת Region, **בלי שינוי קוד**.
   **S3 + CloudFront** — זול ומדרג יותר, אבל **דורש שינוי באפליקציה** לכתיבה דרך API.
   **המכריע:** אם השאלה אומרת "without changing the application" → **EFS**.
   אם היא אומרת "most cost-effective" בלי הגבלת שינויים → **S3**.
3. **Alias** מצביע ישירות על ה-ALB, מתעדכן אוטומטית כשה-IPs שלו משתנים, ו**חינם** ב-Route 53.
   **A Record עם TTL של שעה** גורם ללקוחות להמשיך לפנות ליעד מת עד שעה שלמה.
4. **S3 Event Notification** על `s3:ObjectCreated` עם סינון `*.jpg` → **Lambda** שמייצרת thumbnail.
   מוסיפים **SQS** באמצע כשצריך **עמידות מוחלטת** (הודעה שנכשלה חוזרת לתור ואז ל-**DLQ**)
   או כשיש **burst** שעולה על ה-concurrency של Lambda — התור סופג אותו.
5. **Read Replicas** מפזרות קריאות ל-instances נוספים; שקוף יחסית לאפליקציה, אבל כל replica
   היא instance במחיר מלא ויש **replication lag**.
   **ElastiCache** נותן **תת-מילישנייה** וחוסך פנייה ל-DB לגמרי, אבל דורש לוגיקת cache
   (Lazy Loading: hit → מהיר; miss → קוראים מה-DB וכותבים ל-cache) וטיפול ב-invalidation.
6. **מוסיפים CloudFront לפני האפליקציה.** קבצי העדכון הם **סטטיים** ולא משתנים,
   ולכן הם מוטמעים בקצה. ה-ASG כמעט לא מתרחב, וחוסכים גם EC2 וגם רוחב פס —
   **בלי שינוי בארכיטקטורה או בקוד**.
7. כי ב-**SQS** ההודעה **נמחקת** אחרי שה-consumer עיבד אותה, ורק **אחד** מקבל אותה.
   **Kinesis** שומר את הרשומות ל-retention, מאפשר **לכמה consumers לקרוא את אותו stream במקביל**,
   שומר **סדר לפי partition key**, ומאפשר **replay** של דאטה ישן אחרי תיקון באג.
8. **ALB SG** — מתיר HTTP/HTTPS מ-`0.0.0.0/0`.
   **EC2 SG** — מתיר תעבורה **מה-SG של ה-ALB בלבד** (לא מ-CIDR).
   **RDS SG** ו-**ElastiCache SG** — מתירים **מה-SG של ה-EC2 בלבד**.
   כך אף שכבה לא נגישה למי שלא נמצא מעליה, גם אם ה-IPs משתנים.
9. **Warm Standby.** סביבה מלאה בקנה מידה מוקטן כבר רצה ב-Region המשני,
   עם **Aurora Global** ו-**S3 CRR**, ו-**Route 53 Health Check** שמעביר את התעבורה.
   **Backup & Restore** נפסל — RTO של שעות עד ימים.
   **Pilot Light** מסוכן — צריך להקים את שכבת האפליקציה בזמן אמת.
   **Multi-Site** עומד בדרישה אבל **יקר מדי** לדרישה של 15 דקות — לא ה-BEST.
10. **ASG עם min=max=desired=1** שפרוס על **לפחות 2 AZs**.
    אם ה-instance נופל, ה-ASG מקים אחד חדש — אולי ב-AZ אחרת.
    **User Data** מצמיד את ה-**Elastic IP** בעלייה (לפי tag), ו-**IAM Role** על ה-instance
    מרשה את קריאת ה-API להצמדה.
    כדי לשמר גם דאטה: **lifecycle hooks** — ב-**terminate** יוצרים **EBS Snapshot** עם tags,
    וב-**launch** יוצרים volume מה-snapshot האחרון ומצמידים אותו.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[07 - Auto Scaling]] · [[08 - Elastic Load Balancing]] · [[15 - CloudFront and Global Delivery]] · [[20 - EFS and File Storage]] · [[22 - RDS Scaling and Availability]] · [[23 - DynamoDB]] · [[28 - SQS and SNS]] · [[29 - Event-Driven Architecture]] · [[33 - High Availability and Scalability]] · [[34 - Disaster Recovery]] · [[36 - Migration and Hybrid Cloud]] · [[38 - Serverless and Modern Architectures]] · [[39 - Architecture Decision Making]] · [[41 - Final Review and Exam Strategy]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 4081–4698, 8902–9267, 15257–15627
