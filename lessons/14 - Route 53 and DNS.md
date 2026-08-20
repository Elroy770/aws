---
lesson: 14
title: Route 53 and DNS
domain: Design High-Performing Architectures
services: [Route 53, Route 53 Resolver, CloudWatch, ELB, CloudFront]
tags: [saa-c03, networking, dns, availability]
---

# 14 — Route 53 and DNS

> [!abstract] בשורה אחת
> Route 53 הוא ה-DNS המנוהל של AWS — ובמבחן הוא בעיקר **מנוע החלטה**: איזו מדיניות ניתוב בוחרים, ומתי Alias במקום CNAME.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | מה DNS פותר ולמה Route 53 מיוחד |
| 2 | איך זה עובד | היררכיית DNS, מסלול ה-query, Hosted Zones, TTL |
| 3 | פירוק מפורט | סוגי records, Alias מול CNAME, **7 מדיניות הניתוב**, Health Checks |
| 4 | עלות | על מה משלמים ב-DNS ואיפה TTL הופך לשיקול כספי |
| 5 | השוואות | Latency מול Geolocation מול Geoproximity, Multi-Value מול ELB |
| 6 | Well-Architected | |
| 7 | מלכודות | Zone Apex, "latency אינו ping", failover ו-TTL |
| 8 | Scenario | אפליקציה גלובלית עם DR ו-blue/green |
| 9-11 | דילוגים, Cheat Sheet, בדיקת הבנה | |

**מונחי מפתח בשיעור:** `Hosted Zone` · `Alias` · `TTL` · `Routing Policy` · `Health Check` · `Calculated Health Check` · `Resolver Endpoint` · `Zone Apex`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- אנשים זוכרים שמות, מכונות מדברות בכתובות IP.
- כתובות IP של משאבי AWS **משתנות** — ALB מחליף IPs, instances מוחלפות.
- יש שני Regions פעילים ורוצים לשלוח כל משתמש לזה שקרוב אליו.
- כשה-Region הראשי נופל, מישהו צריך להפסיק לשלוח אליו תעבורה — אוטומטית.

### מה השירות פותר

- **תרגום שם → כתובת**, בצורה מנוהלת, זמינה ומדרגית.
- **Authoritative DNS** — אתה בעל הרשומות ואתה מעדכן אותן.
- **Domain Registrar** — אפשר גם לקנות את הדומיין ב-AWS.
- **Health Checks** — Route 53 בודק אם היעד חי, ומפסיק להחזיר אותו אם לא.
- **Routing Policies** — הפצת תעבורה לפי משקל, latency, מיקום גיאוגרפי או failover.
- **ה-SLA היחיד ב-AWS של 100% זמינות.** זו עובדה שנשאלת.

> [!tip] למה דווקא 53?
> 53 הוא ה-port המסורתי של DNS. שם השירות הוא בדיחה פנימית של AWS.

> [!warning] נקודה שמבלבלת
> DNS **לא מנתב תעבורה**. הוא רק **עונה על שאילתות**.
> ה-client מקבל תשובה ואז מתחבר בעצמו. לכן "Routing Policy" ב-Route 53
> זה לא אותו דבר כמו routing ב-Load Balancer.

---

## 2. ⚙️ איך זה עובד

### 2.1 האנטומיה של URL

```text
http://api.www.example.com.
 │      │   │    │      │  │
 │      │   │    │      │  └─ Root (הנקודה בסוף)
 │      │   │    │      └──── TLD  — Top Level Domain (.com)
 │      │   │    └─────────── SLD  — Second Level Domain (example)
 │      │   └──────────────── Sub Domain (www)
 │      └──────────────────── Sub Domain (api)
 └─────────────────────────── Protocol

api.www.example.com = FQDN (Fully Qualified Domain Name)
```

### 2.2 מונחי היסוד

| מונח | מה זה |
|---|---|
| Domain Registrar | מי שמוכר לך את הדומיין (Route 53, GoDaddy...) |
| DNS Records | הרשומות עצמן — A, AAAA, CNAME, NS |
| Zone File | הקובץ שמכיל את הרשומות |
| Name Server | שרת שעונה על שאילתות DNS (authoritative או לא) |
| TLD | סיומת עליונה — `.com`, `.org`, `.gov` |
| SLD | הרמה מתחת — `amazon` ב-`amazon.com` |

### 2.3 מסלול ה-query — איך DNS באמת עובד

```text
1. Browser → Local DNS Server (של ה-ISP או הארגון)
              "מי זה example.com?"

2. Local → Root DNS Server            (מנוהל ע"י ICANN)
              "לך לשרת של .com"

3. Local → TLD DNS Server (.com)      (מנוהל ע"י IANA)
              "example.com נמצא ב-NS 5.6.7.8"

4. Local → SLD DNS Server             (מנוהל ע"י ה-Registrar / Route 53)
              "example.com = 9.10.11.12"

5. Local → Browser  (+ נשמר ב-cache למשך ה-TTL)

6. Browser → Web Server (9.10.11.12)   ← החיבור בפועל, לא דרך DNS
```

**הנקודה החשובה:** ה-Local DNS Server שומר ב-cache. משך ה-cache = **TTL**.
לכן שינוי record לא נכנס לתוקף מיד אצל כולם.

### 2.4 Hosted Zones

Hosted Zone = מכל של records עבור דומיין ותת-הדומיינים שלו.

| סוג | מי רואה אותו | דוגמה |
|---|---|---|
| **Public Hosted Zone** | כל האינטרנט | `application1.mypublicdomain.com` |
| **Private Hosted Zone** | רק VPCs שמקושרים אליו במפורש | `db.company.internal` |

- Private Hosted Zone מאפשר שמות פנימיים שלא נחשפים החוצה כלל.
- אפשר לקשר **כמה VPCs** לאותו Private Hosted Zone (גם cross-account).
- מחייבים **חיוב חודשי קבוע לכל hosted zone** — לא לפי שימוש.

### 2.5 TTL — הפרמטר התפעולי החשוב ביותר

| TTL | יתרון | חיסרון |
|---|---|---|
| **גבוה** (למשל 24 שעות) | פחות queries → **עלות נמוכה יותר**, פחות עומס | שינויים לוקחים עד יממה להתפשט |
| **נמוך** (למשל 60 שניות) | שינוי ו-failover מהירים | יותר queries → **עלות גבוהה יותר** |

- **TTL הוא חובה בכל record — חוץ מ-Alias.** ב-Alias אי אפשר להגדיר TTL בכלל.
- **פרקטיקה מקצועית:** לפני שינוי מתוכנן, מורידים את ה-TTL כמה שעות מראש, מבצעים את השינוי, ואז מחזירים אותו למעלה.

> [!warning] TTL ו-Failover
> גם אם ה-health check זיהה כשל תוך 30 שניות, ה-clients ימשיכו לפנות ליעד הישן
> עד שה-**TTL** יפוג אצלם. TTL גבוה = failover איטי. זו שאלת מבחן קלאסית.

---

## 3. 🔍 פירוק מפורט

### 3.1 מבנה record

כל record מכיל:

- **שם** — `example.com` או `api.example.com`
- **Type** — A / AAAA / CNAME / NS ...
- **Value** — הערך שיוחזר
- **Routing Policy** — איך Route 53 מחליט מה להחזיר
- **TTL** — כמה זמן לשמור ב-cache

### 3.2 סוגי Records

| Type | מה עושה | הערה |
|---|---|---|
| **A** | שם → כתובת **IPv4** | הבסיסי ביותר |
| **AAAA** | שם → כתובת **IPv6** | |
| **CNAME** | שם → **שם אחר** | היעד חייב להיות בעל record מסוג A/AAAA |
| **NS** | Name Servers של ה-Hosted Zone | מה שמפנה את העולם אליך |
| CAA / DS / MX / TXT / SRV / PTR / SOA / SPF / NAPTR | מתקדמים | לדעת שקיימים; לא נבחנים לעומק |

**ארבעת החובה:** A, AAAA, CNAME, NS.

### 3.3 CNAME מול Alias — ההשוואה שנשאלת כמעט תמיד

הבעיה: משאבי AWS חושפים hostname מכוער כמו
`MyALB-123456789.us-east-1.elb.amazonaws.com`,
ואתה רוצה שהמשתמשים יראו `example.com`.

| קריטריון | **CNAME** | **Alias** |
|---|---|---|
| מצביע אל | כל hostname שהוא | **רק משאבי AWS** |
| **Zone Apex** (`example.com`) | ❌ **אסור** | ✅ **מותר** |
| תת-דומיין (`www.example.com`) | ✅ | ✅ |
| עלות | נספר כ-query רגיל | **חינם** |
| Health Check מובנה | לא | **כן** (native) |
| Type של הרשומה | CNAME | תמיד **A / AAAA** |
| TTL | חובה להגדיר | **לא ניתן להגדיר** — AWS מנהל |
| מעקב אחרי שינוי IP של היעד | לא רלוונטי | **אוטומטי** |

> [!info] שורה תחתונה
> מצביעים על משאב AWS? → **תמיד Alias**. חינם, עובד ב-zone apex, ומתעדכן לבד.
> CNAME נשמר ליעדים חיצוניים ותת-דומיינים בלבד.

### 3.4 יעדים חוקיים ל-Alias

- Elastic Load Balancers (ALB / NLB / CLB)
- CloudFront Distributions
- API Gateway
- Elastic Beanstalk environments
- **S3 Websites** (רק static website endpoint, לא bucket רגיל)
- VPC **Interface** Endpoints
- Global Accelerator
- Record אחר **באותו Hosted Zone**

> [!warning] היוצא מן הכלל שנשאל
> **אי אפשר** ליצור Alias ל-**DNS name של EC2 instance**.
> ל-EC2 משתמשים ב-record מסוג **A** עם ה-IP, או שמים ELB לפניו.

### 3.5 שבע מדיניות הניתוב — הטבלה המרכזית

| מדיניות | מה עושה | Use case | Health Check | מילת מפתח במבחן |
|---|---|---|---|---|
| **Simple** | מחזיר ערך אחד (או כמה, וה-client בוחר **אקראית**) | יעד יחיד, בלי לוגיקה | ❌ **לא נתמך** | "single resource", "no health check" |
| **Weighted** | מפצל תעבורה לפי משקלים יחסיים | blue/green, canary, פיצול בין Regions | ✅ | "percentage", "canary", "test new version", "blue/green" |
| **Latency-based** | מחזיר את ה-Region עם ה-latency הנמוך ביותר למשתמש | חוויית משתמש גלובלית | ✅ | "lowest latency", "best performance for users" |
| **Failover** | Primary פעיל; אם נכשל — Secondary | Active-Passive DR | ✅ **חובה** | "active-passive", "disaster recovery", "standby" |
| **Geolocation** | לפי **מיקום המשתמש** — יבשת / מדינה / State בארה"ב | לוקליזציה, רגולציה, חסימת תוכן | ✅ | "based on country", "compliance", "localized content" |
| **Geoproximity** | לפי מרחק גיאוגרפי, עם **bias** להזזת תעבורה | הסטה הדרגתית בין Regions | ✅ | "bias", "shift more traffic to a region" |
| **Multi-Value** | מחזיר עד **8 רשומות בריאות** | HA פשוט ללא ELB | ✅ | "multiple healthy IPs", "client-side load balancing" |

**בונוס — IP-based Routing:** ניתוב לפי **טווחי CIDR של ה-clients**.
מגדירים CIDR Collection שממפה בלוקים ל-endpoints. Use case: ניתוב משתמשי ISP מסוים ליעד ספציפי, אופטימיזציית ביצועים והקטנת עלויות רשת.

### 3.6 פרטים לכל מדיניות

**Simple**

- אפשר לשים כמה ערכים באותו record — ה-**client** בוחר אחד אקראית.
- כשמפעילים Alias — מותר משאב AWS **אחד** בלבד.
- **לא ניתן לשייך Health Check.** זה ההבדל המבחין העיקרי.

**Weighted**

- הנוסחה: `אחוז = משקל הרשומה / סכום כל המשקלים`.
- **המשקלים לא חייבים להסתכם ל-100.**
- כל הרשומות חייבות אותו **שם** ואותו **type**.
- **משקל 0** = להפסיק לשלוח תעבורה ליעד הזה.
- אם **כל** הרשומות במשקל 0 — כולן מוחזרות **באופן שווה**. זו מלכודת.

**Latency-based**

- המדידה היא latency בין המשתמשים ל-**AWS Regions**, לפי נתונים היסטוריים של AWS.
- **לא** ping בזמן אמת של המשתמש הספציפי.
- לכן משתמש בגרמניה **יכול** להיות מנותב לארה"ב, אם שם ה-latency נמדד כנמוך יותר.

**Failover (Active-Passive)**

- מגדירים record אחד כ-**Primary** ואחד כ-**Secondary**.
- **Health Check על ה-Primary הוא חובה** — בלעדיו אין failover.
- כשה-Primary נכשל, Route 53 מתחיל להחזיר את ה-Secondary.

**Geolocation**

- **שונה מ-Latency!** מבוסס על **איפה המשתמש**, לא על כמה מהר.
- רזולוציה: **יבשת → מדינה → US State**. אם יש חפיפה — נבחר **הספציפי ביותר**.
- **חובה ליצור record מסוג "Default"** — אחרת משתמש ממיקום לא ממופה לא יקבל תשובה.

**Geoproximity**

- מנתב לפי המרחק בין המשתמש למשאב, עם אפשרות **להטות** את הגבול.
- **Bias חיובי (1 עד 99)** — מרחיב את האזור הגיאוגרפי → יותר תעבורה למשאב.
- **Bias שלילי (-1 עד -99)** — מכווץ את האזור → פחות תעבורה.
- משאבים יכולים להיות **AWS** (לפי Region) או **לא-AWS** (לפי קו רוחב/אורך).
- **דורש הפעלת Route 53 Traffic Flow.** זו מילת מפתח מזהה.

**Multi-Value**

- מחזיר **עד 8 רשומות בריאות** בכל תשובה.
- אפשר לשייך Health Checks — רשומות לא-בריאות **לא** מוחזרות.
- **אינו תחליף ל-ELB.** אין בו חלוקת עומס אמיתית, אין session, אין health בשכבה 7.
- זה מנגנון HA "בזול" ברמת ה-DNS.

### 3.7 Health Checks

**שלושה סוגים:**

| סוג | מה מנטר | מתי משתמשים |
|---|---|---|
| **Endpoint** | endpoint ציבורי (אפליקציה, שרת, משאב AWS) | הרגיל |
| **Calculated** | **health checks אחרים** | הרכבת לוגיקה |
| **CloudWatch Alarm** | Alarm של CloudWatch | **משאבים פרטיים!** |

**מפרט ה-Endpoint Health Check:**

| פרמטר | ערך |
|---|---|
| מספר health checkers גלובליים | ~15 |
| נחשב תקין אם | קוד תשובה **2xx** או **3xx** |
| בדיקת תוכן | לפי טקסט ב-**5120 הבייטים הראשונים** של התשובה |
| Healthy / Unhealthy Threshold | **3** (ברירת מחדל) |
| Interval | **30 שניות** (ניתן ל-**10 שניות** — בעלות גבוהה יותר) |
| פרוטוקולים נתמכים | **HTTP, HTTPS, TCP** |
| סף החלטה | אם **יותר מ-18%** מה-checkers מדווחים תקין → Healthy |
| בחירת מיקומים | אפשר לבחור מאילו אזורים לבדוק |

> [!warning] חובה לפתוח firewall
> ה-health checkers מגיעים מטווחי IP של AWS.
> אם ה-Security Group או ה-firewall חוסמים אותם — הבדיקה תיכשל תמיד.
> טווחי ה-IP מפורסמים ב-`ip-ranges.json` של AWS.

**Calculated Health Checks:**

- משלבים תוצאות של כמה health checks לאחד, עם **OR / AND / NOT**.
- עד **256 child health checks**.
- אפשר להגדיר **כמה** מהילדים צריכים לעבור כדי שההורה יעבור.
- **Use case קלאסי:** תחזוקה מתוכננת על חלק מהשרתים בלי שכל האתר ייחשב down.

**Health Checks ל-Private Hosted Zones — המלכודת:**

- ה-health checkers של Route 53 נמצאים **מחוץ ל-VPC**.
- לכן הם **לא יכולים** להגיע ל-endpoint פרטי (private subnet או on-premises).
- **הפתרון:** יוצרים CloudWatch Metric → CloudWatch Alarm → **Health Check שמנטר את ה-Alarm**.
- זו התשובה הנכונה בכל שאלה על "health check למשאב פרטי".

### 3.8 Registrar מול DNS Service

- **Registrar** = מי שמוכר את הדומיין (חיוב שנתי).
- **DNS Service** = מי שמנהל את הרשומות.
- **אלה שני דברים שונים!** אפשר לקנות ב-GoDaddy ולנהל ב-Route 53.

**איך מחברים registrar חיצוני ל-Route 53:**

```text
1. יוצרים Public Hosted Zone ב-Route 53
2. Route 53 מנפיק 4 Name Servers
3. נכנסים לממשק ה-registrar החיצוני
4. מעדכנים שם את רשומות ה-NS לשמות של Route 53
5. מרגע זה Route 53 הוא ה-authoritative DNS
```

### 3.9 Hybrid DNS ו-Resolver Endpoints

Route 53 Resolver עונה **אוטומטית** בתוך VPC על:

- שמות מקומיים של EC2 instances
- רשומות ב-Private Hosted Zones
- רשומות ציבוריות (דרך Name Servers ציבוריים)

**Hybrid DNS** = לגרום ל-VPC ול-רשת שלך לפתור שמות אחד של השני.

| סוג Endpoint | כיוון | מה מאפשר |
|---|---|---|
| **Inbound Endpoint** | on-prem → AWS | ה-DNS resolvers שלך יכולים לפתור שמות של Private Hosted Zones ו-EC2 |
| **Outbound Endpoint** | AWS → on-prem | Route 53 Resolver **מעביר (forwards)** שאילתות ל-DNS resolvers שלך |

```text
Inbound:   on-prem server → "app.aws.private?" → Inbound Endpoint → Route 53 Resolver ✔

Outbound:  EC2 → "web.onpremise.private?" → Route 53 Resolver
              → Outbound Endpoint → on-prem DNS Resolver ✔
```

- דורש קישוריות רשתית: **VPN או Direct Connect**.
- עובד גם מול VPC אחר או VPC ב-peering.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **Hosted Zone** | חיוב **חודשי קבוע** לכל zone | גם אם אין בו שאילתות בכלל |
| **DNS Queries** | לפי מיליון שאילתות | policies מתקדמות (latency/geo) יקרות יותר לשאילתה |
| **Alias queries ל-משאבי AWS** | **חינם** | זה יתרון עלותי אמיתי מול CNAME |
| **Health Checks** | חיוב חודשי לכל check | endpoints מחוץ ל-AWS יקרים יותר |
| **Health Check מהיר (10 שניות)** | תוספת | מול 30 שניות ברירת מחדל |
| **Health Check features** | תוספת | String matching, HTTPS, Latency measurement |
| **Domain Registration** | חיוב **שנתי** | תלוי TLD |
| **Traffic Flow** | חיוב חודשי לכל policy record | נדרש עבור **Geoproximity** |
| **Resolver Endpoints** | חיוב שעתי **לכל ENI** + לפי שאילתות | 2 ENIs מינימום ל-HA |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Alias ל-ALB/CloudFront | **0** על השאילתה | תמיד, כשהיעד הוא AWS |
| CNAME לאותו יעד | משלמים על השאילתה | רק כשאין ברירה |
| TTL גבוה | נמוך | תוכן יציב שלא משתנה |
| TTL נמוך | גבוה | לפני שינוי מתוכנן, או ב-DR עם RTO קצר |
| Simple policy | הזול ביותר | יעד יחיד |
| Geoproximity | היקר ביותר (דורש Traffic Flow) | רק כשבאמת צריך bias |
| Health Check interval 30s | בסיסי | ברירת המחדל |
| Health Check interval 10s | יקר יותר | רק כש-RTO ממש קצר |

### 🚩 עלויות נסתרות

- **Hosted Zones נשכחים** — כל zone מחייב חודשית גם אם הדומיין לא בשימוש. נקה zones ישנים.
- **TTL של 60 שניות על אתר עמוס** — יכול להכפיל את חשבון ה-queries פי כמה.
- **Health Checks מיותרים** — אחד לכל instance במקום אחד ל-ALB.
- **Health Checks על endpoints מחוץ ל-AWS** מתומחרים גבוה יותר.
- **Traffic Flow** — נדרש ל-Geoproximity, ומחייב לכל policy record.
- **Resolver Endpoints** — חיוב שעתי לכל ENI, ולכל endpoint צריך לפחות 2 ל-HA.

### 💡 טיפים לחיסכון

- **תמיד Alias** במקום CNAME כשהיעד הוא משאב AWS — חוסך את עלות השאילתה.
- **Health Check אחד על ה-ALB**, לא אחד לכל instance מאחוריו.
- **TTL סביר** (למשל שעה) לרשומות יציבות; מורידים זמנית רק לפני שינוי.
- **מחקו hosted zones ו-health checks שאינם בשימוש** — זה חיוב קבוע שקט.
- אל תבחרו Geoproximity אם **Geolocation** או **Latency** פותרים את הצורך — הן זולות יותר.
- Health Check מבוסס **CloudWatch Alarm** מנצל metric שכבר קיים, במקום probe חיצוני.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 Latency מול Geolocation מול Geoproximity

| קריטריון | Latency-based | Geolocation | Geoproximity |
|---|---|---|---|
| הקריטריון | **מהירות** מדודה | **מדינה/יבשת** של המשתמש | **מרחק** גיאוגרפי + bias |
| שליטה ידנית | אין | יש (מיפוי מפורש) | יש (**bias**) |
| דורש record ברירת מחדל | לא | **כן — חובה** | לא |
| דורש Traffic Flow | לא | לא | **כן** |
| Use case | ביצועים גלובליים | רגולציה, שפה, זכויות תוכן | הסטה הדרגתית בין Regions |
| מילת מפתח | "lowest latency" | "based on country / compliance" | "bias / shift traffic" |

### 5.2 Multi-Value מול ELB

| קריטריון | Multi-Value Answer | Elastic Load Balancer |
|---|---|---|
| שכבה | DNS | 4 / 7 |
| חלוקת עומס אמיתית | לא — ה-client בוחר | **כן** |
| Health Check | ברמת ה-record | ברמת ה-target |
| מספר יעדים בתשובה | עד **8** | לא רלוונטי |
| TLS termination, path routing | לא | **כן** |
| עלות | queries + health checks | חיוב שעתי + LCU |

> [!info] שורה תחתונה
> Multi-Value זה HA זול ברמת DNS. אם השאלה מבקשת load balancing אמיתי — התשובה היא **ELB**.

### 5.3 Failover מול Weighted ל-DR

| קריטריון | Failover | Weighted |
|---|---|---|
| התנהגות רגילה | 100% ל-Primary | פיצול לפי משקלים |
| שימוש טיפוסי | Active-Passive DR | Active-Active / canary / blue-green |
| Health Check | **חובה** על Primary | אופציונלי אך מומלץ |
| כיבוי יעד | אוטומטי בכשל | ידני — משקל 0 |

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-DNS ו-Route 53** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | שינוי DNS הוא שינוי מסוכן — צריך תהליך | Hosted Zones ו-records ב-IaC; **Query Logging** ל-CloudWatch; runbook שמוריד TTL לפני שינוי |
| Security | הדומיין הוא נכס שאפשר לחטוף | **Private Hosted Zones** לשמות פנימיים; **DNSSEC** לדומיינים ציבוריים; Registrar Lock; IAM least-privilege על שינוי records |
| Reliability | DNS לא יכול להיות ה-SPOF | **Failover policy** עם health checks; **Calculated Health Checks** לתחזוקה; TTL קצר ליעדים שמשתתפים ב-DR |
| Performance Efficiency | לקצר את הדרך למשתמש | **Latency-based routing**; **Alias** למשאבי AWS (מתעדכן אוטומטית); TTL שמאזן בין רעננות ל-cache |
| Cost Optimization | לצמצם queries וחיובים קבועים | Alias במקום CNAME; TTL סביר; מחיקת zones ו-health checks לא בשימוש; לא Geoproximity כשמספיק Geolocation |
| Sustainability | פחות שאילתות ופחות probes | TTL גבוה יותר לרשומות יציבות; פחות health checks (על ה-ALB ולא על כל instance); ניתוב ל-Region קרוב מקטין תעבורה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "zone apex" / "naked domain" / "example.com without www" | **Alias** (CNAME אסור) |
| "point to an ALB / CloudFront" | Alias record |
| "10% of traffic to the new version" | **Weighted** |
| "blue/green" / "canary" | Weighted |
| "route users to the closest region for performance" | **Latency-based** |
| "users in Germany must see German content" | **Geolocation** |
| "content licensing restrictions per country" | Geolocation |
| "shift more traffic to a specific region" / "bias" | **Geoproximity** (+ Traffic Flow) |
| "active-passive DR" / "standby site" | **Failover** |
| "return multiple healthy IP addresses" | **Multi-Value** |
| "health check for a private / on-premises resource" | **CloudWatch Alarm** → Health Check |
| "maintenance without failing all checks" | **Calculated Health Check** |
| "resolve on-prem names from EC2" | Resolver **Outbound** Endpoint |
| "resolve AWS private names from the data center" | Resolver **Inbound** Endpoint |
| "changes take too long to take effect" | **TTL גבוה מדי** |
| "100% availability SLA" | Route 53 |
| "route by client ISP / CIDR block" | IP-based Routing |

### טעויות נפוצות

> [!warning] מלכודת 1 — CNAME ב-Zone Apex
> **הניסוח:** "רוצים ש-`example.com` (בלי www) יצביע ל-ALB."
> **הטעות:** ליצור CNAME.
> **הנכון:** **אסור** CNAME ב-zone apex. יוצרים **Alias record** מסוג A.

> [!warning] מלכודת 2 — Latency זה לא ping
> **הניסוח:** "משתמש בגרמניה נשלח ל-us-east-1 למרות שיש Region באירלנד."
> **הטעות:** להניח שיש באג.
> **הנכון:** Latency-based מתבסס על **מדידות latency היסטוריות של AWS**, לא על ping חי.
> אם רוצים שליטה מוחלטת לפי מדינה — משתמשים ב-**Geolocation**.

> [!warning] מלכודת 3 — Simple + Health Check
> **הניסוח:** "Simple routing עם health check שיסיר יעד כושל."
> **הטעות:** לחשוב שזה אפשרי.
> **הנכון:** **Simple routing לא תומך ב-Health Checks בכלל.**
> צריך Multi-Value (ל-HA) או Failover (ל-DR).

> [!warning] מלכודת 4 — Geolocation בלי Default
> **הניסוח:** מיפו records ל-US, אירופה ואסיה. משתמש מדרום אמריקה לא מצליח להתחבר.
> **הטעות:** להוסיף עוד records ידנית לכל יבשת.
> **הנכון:** יוצרים **record מסוג "Default"** שתופס כל מיקום לא ממופה.

> [!warning] מלכודת 5 — Health Check למשאב פרטי
> **הניסוח:** "צריך health check ל-instance ב-private subnet."
> **הטעות:** להגדיר endpoint health check על ה-Private IP.
> **הנכון:** ה-health checkers **מחוץ ל-VPC** ולא יגיעו.
> יוצרים **CloudWatch Alarm** ומגדירים Health Check שמנטר את ה-Alarm.

> [!warning] מלכודת 6 — Failover איטי בגלל TTL
> **הניסוח:** "ה-health check זיהה כשל תוך דקה, אבל משתמשים המשיכו לקבל שגיאות שעה."
> **הטעות:** להאשים את ה-health check.
> **הנכון:** ה-**TTL** היה גבוה מדי — ה-resolvers שמרו את התשובה הישנה ב-cache.
> ליעדים שמשתתפים ב-failover קובעים TTL קצר (60 שניות).

> [!warning] מלכודת 7 — כל המשקלים 0
> **הניסוח:** "רצינו לעצור תעבורה לכולם ושמנו weight=0 בכל הרשומות."
> **הטעות:** לצפות שהתעבורה תיעצר.
> **הנכון:** כשכל המשקלים 0, Route 53 מחזיר את **כולן באופן שווה**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**
API גלובלי הרץ ב-`us-east-1` וב-`eu-west-1` מאחורי ALB בכל Region.
דרישות:
(1) כל משתמש יגיע ל-Region המהיר עבורו;
(2) אם Region נופל — הסטה אוטומטית לשני;
(3) הדומיין הראשי `example.com` בלי `www`;
(4) גרסה חדשה תיבדק על 5% מהתעבורה לפני שחרור מלא;
(5) `db.internal` צריך להיפתר רק בתוך ה-VPCs;
(6) שרתי on-prem צריכים לפתור את השמות הפנימיים האלה.

```text
                     example.com
                          │
                 Route 53 Public Hosted Zone
                          │
          ┌── Latency-based Alias records (A) ──┐
          │                                     │
   [Health Check]                        [Health Check]
          │                                     │
   ALB us-east-1                          ALB eu-west-1
          │                                     │
   ┌──────┴───────┐                             │
   │  Weighted    │                             │
   │ v1: 95       │                             │
   │ v2: 5        │  ← canary                   │
   └──────────────┘                             │

   Private Hosted Zone: company.internal
        db.company.internal → 10.0.1.20
              │
      Resolver Inbound Endpoint ← on-prem DNS (over DX/VPN)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **Latency-based routing** בין שני ה-Regions | דרישה (1) — כל משתמש ל-Region המהיר |
| **Health Check על כל ALB** | דרישה (2) — Latency policy תומך ב-health checks ומדלג על יעד כושל |
| **Alias records** (לא CNAME) | דרישה (3) — `example.com` הוא zone apex; ובנוסף חינם ומתעדכן לבד |
| **TTL קצר** (60 שניות) על הרשומות הללו | כדי שההסטה בכשל תגיע ל-clients במהירות |
| **Weighted policy** 95/5 בתוך ה-Region | דרישה (4) — canary מבוקר; להעלות בהדרגה |
| **Private Hosted Zone** `company.internal` | דרישה (5) — נפתר רק ב-VPCs המקושרים, לא נחשף לאינטרנט |
| **Route 53 Resolver Inbound Endpoint** | דרישה (6) — מאפשר ל-DNS resolvers ב-on-prem לפתור את השמות הפרטיים |
| **Calculated Health Check** לתחזוקה | מאפשר לתחזק חלק מהשרתים בלי שכל ה-Region ייחשב unhealthy |

**למה לא Failover policy?**
Failover הוא Active-**Passive** — Region אחד יושב ולא מקבל תעבורה.
כאן שני ה-Regions פעילים ורוצים ביצועים, לכן **Latency + Health Checks** נותן גם ביצועים וגם failover.

**למה לא Geolocation?**
Geolocation מנתב לפי **מדינה**, לא לפי מהירות. אין דרישת רגולציה כאן — הדרישה היא ביצועים.

**למה לא Multi-Value?**
זה היה מחזיר את שני ה-ALBs לכל משתמש והוא היה בוחר אקראית — מפספס לגמרי את דרישת ה-latency.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **תחביר zone file גולמי** (BIND) — לא נבחן.
- **רשומות מתקדמות לעומק** — CAA, NAPTR, SRV, DS. מספיק לדעת שהן קיימות.
- **מנגנון DNSSEC הפנימי** — צריך לדעת ש-Route 53 תומך ולמה זה טוב, לא איך זה עובד.
- **המספר המדויק של health checkers** — "בערך 15" מספיק.
- **טווחי ה-IP של ה-health checkers** — צריך לדעת שיש לפתוח firewall, לא את הטווחים.
- **מחירים מדויקים בדולרים** — משתנים; לדעת **מה חינם** (Alias) ומה חיוב קבוע (hosted zone).
- **Traffic Flow כמוצר** — צריך לדעת שהוא נדרש ל-Geoproximity. לא יותר.
- **תהליך העברת דומיין בין registrars** — תפעולי, לא נבחן.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- Route 53 = **Authoritative DNS + Registrar + Health Checks**. ה-SLA היחיד ב-AWS של **100%**.
- **DNS לא מנתב תעבורה** — הוא רק עונה על שאילתות.
- **CNAME אסור ב-Zone Apex. Alias מותר** — ובנוסף חינם, עם health check מובנה, ובלי TTL.
- **Alias תמיד A/AAAA**, ורק ליעדי AWS. **אין Alias ל-EC2 DNS name.**
- שבע מדיניות: **Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multi-Value** (+ IP-based).
- **רק Simple לא תומך ב-Health Checks.**
- **Geoproximity דורש Traffic Flow** ומשתמש ב-**bias** (‎-99 עד 99).
- **Geolocation חייב record מסוג Default.**
- **Multi-Value = עד 8 רשומות בריאות**, ואינו תחליף ל-ELB.
- **Failover דורש Health Check על ה-Primary.**
- Health Check: **2xx/3xx** = תקין · threshold **3** · interval **30s** (או 10s ביוקר) · **HTTP/HTTPS/TCP** · סף **18%** מה-checkers.
- **Health Check למשאב פרטי** → CloudWatch Alarm → Health Check על ה-Alarm.
- **Calculated Health Check** — עד 256 ילדים, עם OR/AND/NOT.
- **TTL גבוה = זול ואיטי. TTL נמוך = יקר ומהיר.** Alias בלי TTL.
- **Registrar ≠ DNS Service.** קונים בחוץ, מנהלים ב-Route 53 בעדכון רשומות **NS**.
- **Inbound Endpoint** = on-prem פותר שמות AWS. **Outbound Endpoint** = AWS פותר שמות on-prem.

---

## 11. ✅ בדיקת הבנה

1. למה אי אפשר להצביע מ-`example.com` ל-ALB באמצעות CNAME, ומה עושים במקום?
2. רוצים לשחרר גרסה חדשה ל-5% מהמשתמשים. איזו מדיניות, ואיך עוצרים אותה מיד?
3. יש instance ב-private subnet שצריך health check ל-failover. איך?
4. Health check זיהה כשל תוך 90 שניות, אבל המשתמשים המשיכו להיכשל 40 דקות. מה קרה?
5. מה ההבדל בין Latency-based ל-Geolocation, ומתי כל אחת?
6. שרתים ב-data center צריכים לפתור `db.company.internal` שנמצא ב-Private Hosted Zone. מה מקימים?
7. שמנו weight=0 בכל הרשומות כדי לעצור את השירות. מה יקרה בפועל?

<details>
<summary>תשובות</summary>

1. `example.com` הוא **Zone Apex**, ותקן ה-DNS אוסר CNAME בצומת העליון של namespace. הפתרון: **Alias record** מסוג A שמצביע ל-ALB. בונוס: Alias חינם, מתעדכן אוטומטית כשה-IPs של ה-ALB משתנים, ויש בו health check מובנה.

2. **Weighted routing** — למשל 95 לגרסה הישנה ו-5 לחדשה (המשקלים לא חייבים להסתכם ל-100). לעצור מיד: משנים את משקל הגרסה החדשה ל-**0**. שימו לב — זה עובד רק כל עוד לרשומה השנייה יש משקל חיובי; אם **כל** המשקלים 0, Route 53 יחזיר את כולן שווה בשווה.

3. ה-health checkers של Route 53 יושבים **מחוץ ל-VPC** ולא מגיעים ל-endpoint פרטי. יוצרים **CloudWatch Metric + Alarm** על בריאות ה-instance, ואז **Health Check מסוג "monitor a CloudWatch Alarm"**.

4. ה-**TTL** של הרשומה היה גבוה מדי. ה-DNS resolvers של המשתמשים המשיכו להחזיר את התשובה הישנה מה-cache עד שה-TTL פג. הפתרון: TTL קצר (60 שניות) לרשומות שמשתתפות ב-failover.

5. **Latency-based** בוחר לפי **מהירות** מדודה ל-AWS Regions — לכן משתמש בגרמניה יכול להישלח לארה"ב. **Geolocation** בוחר לפי **מיקום המשתמש** בפועל (יבשת/מדינה/State), ומחייב record ברירת מחדל. משתמשים ב-Latency לביצועים, וב-Geolocation לרגולציה, שפה או זכויות תוכן.

6. **Route 53 Resolver Inbound Endpoint** ב-VPC, מעל קישוריות **Direct Connect או VPN**. אז מפנים את ה-DNS resolvers ב-data center לכתובות ה-IP של ה-Inbound Endpoint. (הכיוון ההפוך — EC2 שפותר שמות on-prem — הוא **Outbound Endpoint**.)

7. השירות **לא ייעצר**. כשכל הרשומות במשקל 0, Route 53 מחזיר את כולן **באופן שווה**. כדי לעצור באמת צריך למחוק את הרשומות, או להשאיר רשומה אחת פעילה עם משקל חיובי ולאפס את השאר.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[08 - Elastic Load Balancing]] · [[13 - VPC Network Architecture]] · [[15 - CloudFront and Global Delivery]] · [[31 - Monitoring and Logging]] · [[33 - High Availability and Scalability]] · [[34 - Disaster Recovery]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 3281–4081
