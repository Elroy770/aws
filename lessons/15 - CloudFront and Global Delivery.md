---
lesson: 15
title: CloudFront and Global Delivery
domain: Design High-Performing Architectures
services: [CloudFront, Global Accelerator, S3, ALB, WAF, Shield, Lambda@Edge, ACM]
tags: [saa-c03, networking, performance, edge, cdn]
---

# 15 — CloudFront and Global Delivery

> [!abstract] בשורה אחת
> CloudFront מקרב **תוכן** למשתמש דרך cache ב-edge; Global Accelerator מקרב **את החיבור עצמו** דרך רשת AWS — והמבחן בודק שאתה יודע מתי כל אחד.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה latency גלובלי הוא בעיה שלא נפתרת בעוד שרתים |
| 2 | איך זה עובד | Edge locations, origins, cache key, TTL |
| 3 | פירוק מפורט | OAC, VPC Origins, Geo Restriction, Invalidations, Edge Functions |
| 4 | עלות | data transfer out, requests, invalidations, Price Class |
| 5 | השוואות | CloudFront מול GA מול CRR · CloudFront Functions מול Lambda@Edge |
| 6 | Well-Architected | |
| 7 | מלכודות | חסימת IP בשכבה הנכונה, GA לא עושה cache |
| 8 | Scenario | אתר מדיה גלובלי עם origin פרטי |
| 9-11 | דילוגים, Cheat Sheet, בדיקת הבנה | |

**מונחי מפתח בשיעור:** `Edge Location` · `Origin` · `OAC` · `Cache Behavior` · `Invalidation` · `Anycast IP` · `Price Class` · `Lambda@Edge`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- האפליקציה רצה ב-Region אחד. המשתמשים באוסטרליה, הודו, ברזיל ואירופה.
- כל בקשה עוברת **המון hops** ברשת הציבורית — כל hop מוסיף latency ותנודתיות.
- אותה תמונה של 2 MB נשלחת שוב ושוב מה-origin לכל משתמש — עומס ועלות egress.
- ה-origin חשוף לאינטרנט, כולל ל-DDoS.

### מה CloudFront פותר

- **CDN** עם מאות **Points of Presence** (edge locations) בעולם.
- התוכן **נשמר ב-cache ב-edge** — קריאות חוזרות לא מגיעות ל-origin בכלל.
- שיפור **read performance** וחוויית משתמש.
- **הגנת DDoS** מובנית (הפצה גלובלית) + אינטגרציה עם **AWS Shield** ו-**AWS WAF**.
- ה-origin יכול להישאר **פרטי לחלוטין**.

### מה Global Accelerator פותר

- לא כל workload הוא HTTP שאפשר לשמור ב-cache — משחקים (UDP), IoT (MQTT), VoIP.
- **Anycast IP** מקבע שתי כתובות IP סטטיות, ומכניס את התעבורה ל**רשת הפנימית של AWS** כמה שיותר מוקדם.
- משם התעבורה זורמת ברשת של AWS — לא באינטרנט הציבורי.

> [!tip] האנלוגיה
> CloudFront = **מחסן מקומי** — הסחורה כבר נמצאת ליד הלקוח.
> Global Accelerator = **נתיב מהיר** — הסחורה עדיין באה מהמפעל, אבל בכביש פרטי בלי פקקים.

---

## 2. ⚙️ איך זה עובד

### 2.1 המסלול הבסיסי

```text
Client  ──GET /beach.jpg?size=300x300──►  CloudFront Edge Location
                                                │
                                     ┌──────────┴──────────┐
                                     │  יש ב-Local Cache?  │
                                     └──────────┬──────────┘
                        HIT ◄───────────────────┴────────────────► MISS
                    מחזיר מיד                                Forward Request
                    (latency נמוך)                                │
                                                                  ▼
                                                            Origin (S3 / HTTP)
                                                                  │
                                                        נשמר ב-cache ל-TTL
```

- **Cache HIT** = תשובה מיידית מה-edge, בלי לגעת ב-origin.
- **Cache MISS** = ה-edge פונה ל-origin, מחזיר ושומר.
- **Cache Hit Ratio** הוא ה-metric שקובע גם ביצועים וגם עלות.

### 2.2 סוגי Origins

| Origin | מתי | הערה |
|---|---|---|
| **S3 bucket** | הפצת קבצים ו-caching ב-edge | מאובטח באמצעות **OAC**; משמש גם ל-**upload** דרך CloudFront |
| **VPC Origin** | אפליקציות ב-**private subnets** | ALB / NLB / EC2 **פרטיים** — בלי חשיפה לאינטרנט |
| **Custom Origin (HTTP)** | כל backend ציבורי | Public ALB, שרת חיצוני |
| **S3 Static Website** | כשצריכים redirects / error pages של S3 | חייב **להפעיל static website hosting** קודם — ואז זה custom origin, לא S3 origin |

> [!warning] נקודה מבלבלת
> "S3 bucket" ו-"S3 static website" הם **שני origins שונים** ב-CloudFront.
> אם השאלה מזכירה redirect rules או index documents — זה **S3 website endpoint** (custom origin).

### 2.3 Cache Behaviors ו-Cache Key

- **Behavior** מוגדר לפי **path pattern** (`/images/*`, `/api/*`, `*`).
- CloudFront בוחר את ה-behavior **הספציפי ביותר** שמתאים.
- כל behavior מגדיר: origin, cache policy, origin request policy, TTL, פרוטוקולים מותרים.
- **Cache Key** = מה שמבדיל אובייקט אחד מהשני ב-cache: URL + מה שבחרת להעביר (headers / cookies / query strings).

| החלטה | ההשפעה |
|---|---|
| Cache key מפורט מדי (הרבה headers/cookies) | **cache hit ratio נמוך** → יותר פניות ל-origin → יקר ואיטי |
| Cache key מינימלי | hit ratio גבוה, אבל אולי תוכן לא מותאם |
| API דינמי | לרוב `CachingDisabled` — אין טעם ב-cache |

### 2.4 CloudFront מול S3 Cross-Region Replication

| קריטריון | **CloudFront** | **S3 Cross-Region Replication** |
|---|---|---|
| טכנולוגיה | רשת **edge** גלובלית | העתקה בין **buckets** ב-Regions |
| רעננות | קבצים ב-cache למשך **TTL** (יכול להיות יום) | עדכון **near real-time** |
| היקף | גלובלי אוטומטית | צריך להגדיר **לכל Region** בנפרד |
| כתיבה | קריאה בעיקר | היעד הוא **read-only** |
| מתאים ל | תוכן **סטטי** שצריך להיות זמין בכל מקום | תוכן **דינמי** ב-latency נמוך במעט Regions |

---

## 3. 🔍 פירוק מפורט

### 3.1 S3 כ-Origin עם OAC

**המטרה:** ה-bucket **סגור לחלוטין** לציבור, ורק CloudFront יכול לקרוא ממנו.

```text
Public www                          Private AWS network
  User  ──►  Edge (Mumbai)  ──OAC──►  S3 Bucket  (Block Public Access = ON)
  User  ──►  Edge (São Paulo) ──OAC──►      ▲
                                            │
                              Bucket Policy מתירה רק
                              את ה-CloudFront distribution
```

- **OAC = Origin Access Control** — המנגנון המודרני (ירש את OAI הישן).
- דורש **שני חלקים**: הגדרת OAC ב-distribution **וגם** bucket policy שמתירה אותו.
- מאפשר גם **העלאת קבצים** ל-S3 דרך CloudFront.

### 3.2 VPC Origins

- מאפשר להגיש תוכן מאפליקציות ב-**private subnets**, בלי לחשוף אותן לאינטרנט.
- יעדים נתמכים: **ALB פרטי, NLB פרטי, EC2 instances פרטיים**.

**החלופה הישנה — origin ציבורי:**

```text
ALB ציבורי:  ALB חייב להיות Public
             SG של ה-ALB מתיר רק את ה-Public IPs של edge locations
             SG של ה-EC2 מתיר רק את ה-SG של ה-ALB  (EC2 יכול להישאר פרטי)

EC2 ציבורי:  ה-EC2 חייב להיות Public
             SG מתיר רק את ה-Public IPs של edge locations
```

- רשימת ה-Public IPs של edge locations מתפרסמת ע"י AWS ומשתנה — צריך אוטומציה לעדכון.
- **VPC Origins עדיף** — אין מה לפתוח ואין מה לתחזק.

### 3.3 Geo Restriction

- מגבילים **מי** רשאי לגשת ל-distribution לפי **מדינה**.
- **Allowlist** — רק המדינות ברשימה מורשות.
- **Blocklist** — המדינות ברשימה חסומות.
- המדינה נקבעת לפי **מסד נתונים Geo-IP של צד שלישי**.
- **Use case מרכזי:** חוקי זכויות יוצרים והגבלות הפצת תוכן.

### 3.4 Cache Invalidations

- כשמעדכנים את ה-origin, CloudFront **לא יודע** — הוא ימשיך להגיש את הגרסה ב-cache עד שה-TTL יפוג.
- **Invalidation** מכריח רענון, ועוקף את ה-TTL.
- אפשר לבטל **הכול** (`*`) או **נתיב מסוים** (`/images/*`).

> [!tip] הפרקטיקה המומלצת
> **Versioned object keys** עדיפים על invalidation תכוף:
> במקום לבטל את `app.js`, מעלים `app.v42.js`.
> אין עלות invalidation, אין המתנה, וה-cache של הגרסה הישנה פשוט מזדקן לבד.

### 3.5 Unicast מול Anycast IP — הבסיס של Global Accelerator

```text
Unicast:   IP אחד → שרת אחד.
           12.34.56.78 = שרת בווירג'יניה.  משתמש בסידני חוצה חצי עולם.

Anycast:   אותו IP מוכרז מהרבה מקומות.
           12.34.56.78 = כל edge location.
           הרשת מנתבת כל client ל-edge הקרוב אליו.
```

### 3.6 AWS Global Accelerator

- יוצר **2 Anycast IPs סטטיים** לאפליקציה.
- ה-IPs מובילים ל-**edge location** הקרוב, ומשם התעבורה נוסעת ברשת **הפנימית** של AWS.
- **יעדים נתמכים:** Elastic IP, EC2 instances, ALB, NLB — **ציבוריים או פרטיים**.

**היתרונות:**

| יתרון | הסבר |
|---|---|
| **Consistent Performance** | ניתוב חכם ל-latency הנמוך ביותר; רשת AWS במקום האינטרנט |
| **אין בעיית client cache** | ה-IP **לא משתנה** — בניגוד ל-DNS failover שתלוי ב-TTL |
| **Health Checks** | GA בודק את האפליקציה; **failover אזורי בפחות מדקה** |
| **מצוין ל-DR** | הודות ל-health checks וה-failover המהיר |
| **אבטחה** | צריך להכניס ל-allowlist **רק 2 IPs**; הגנת DDoS דרך **AWS Shield** |

> [!info] למה זה חזק ל-DR
> Route 53 failover תלוי בפקיעת TTL אצל ה-clients.
> Global Accelerator לא — ה-IP נשאר זהה, וההסטה קורית **בתוך הרשת**. לכן ה-failover מהיר יותר ודטרמיניסטי.

### 3.7 Edge Functions

**הרעיון:** להריץ קוד **ב-edge**, קרוב למשתמש, בלי לנהל שרתים.

**Use cases:**

- Website security and privacy
- Dynamic web application at the edge
- SEO
- ניתוב חכם בין origins ו-data centers
- Bot mitigation ב-edge
- Real-time image transformation
- A/B testing
- User authentication and authorization
- User prioritization
- User tracking and analytics

**ארבע נקודות ההזרקה (triggers):**

```text
Client ──(1) Viewer Request──► CloudFront ──(2) Origin Request──► Origin
Client ◄─(4) Viewer Response── CloudFront ◄─(3) Origin Response── Origin

CloudFront Functions:  רק (1) ו-(4)
Lambda@Edge:           כל הארבע
```

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **Data Transfer Out to viewers** | per GB | הרכיב העיקרי; משתנה לפי **גיאוגרפיה** |
| **HTTP/HTTPS Requests** | לפי 10,000 בקשות | HTTPS יקר מעט יותר |
| **Invalidations** | מכסה חינם של נתיבים לחודש, ואז חיוב לנתיב | עוד סיבה להעדיף versioned keys |
| **Field-Level Encryption** | לפי בקשה | תוספת |
| **CloudFront Functions** | לפי מיליון הפעלות | יש **free tier**, ובערך **שישית** מהמחיר של Lambda@Edge |
| **Lambda@Edge** | לפי בקשות **ו**-duration | **אין free tier** |
| **Origin fetch מ-S3** | **0** | S3 → CloudFront לא מחויב |
| **Dedicated IP SSL** | חיוב חודשי גבוה | SNI (ברירת המחדל) הוא **חינם** |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| הגשה **ישירה מ-S3** לאינטרנט | הגבוה ביותר per GB | כמעט אף פעם, לתוכן ציבורי |
| **CloudFront** מול S3 | egress per GB **זול יותר** מ-S3 ישיר, ועלות ה-requests נמוכה משמעותית | כמעט תמיד |
| **S3 Transfer Acceleration** | תוספת **מעל** התעריף הרגיל | רק ל-**uploads** מרחוק ל-S3 |
| **Price Class All** | היקר ביותר | קהל באמת גלובלי |
| **Price Class 200 / 100** | זול יותר | כשהקהל מרוכז — במחיר latency גבוה יותר לאזורים מרוחקים |
| **CloudFront Functions** | ~1/6 מ-Lambda@Edge | לוגיקה קלה: headers, redirects, cache key |
| **Lambda@Edge** | היקר | רק כשצריך body / רשת / זמן ריצה ארוך |
| **Global Accelerator** | חיוב **שעתי קבוע** + per GB | רק כשבאמת צריך static IPs / non-HTTP / failover מהיר |

> [!info] נקודת העלות הכי שימושית
> **S3 → CloudFront = 0**, ו-**CloudFront → אינטרנט זול יותר מ-S3 → אינטרנט**.
> ובנוסף ה-cache חוסך גם את עלות ה-**requests** מול S3 (הפרש של פי כמה).
> לכן שימת CloudFront מול S3 היא כמעט תמיד גם שיפור ביצועים **וגם** חיסכון.

### 🚩 עלויות נסתרות

- **Cache Hit Ratio נמוך** — cache key מפורט מדי מכריח פנייה ל-origin בכל בקשה. משלמים פעמיים.
- **Cache-busting תכוף** — query string משתנה בכל בקשה מבטל את ה-cache לגמרי.
- **Invalidations מעבר למכסה** — מחויבים **לכל נתיב**, ו-`/*` על הרבה קבצים מצטבר.
- **Lambda@Edge על כל בקשה** — נעשה יקר מהר; לרוב CloudFront Function מספיקה.
- **Global Accelerator hourly** — חיוב קבוע גם כשאין תעבורה.
- **Data transfer מ-origin ב-Region מרוחק** — אם ה-origin לא באותו Region, כל MISS משלם egress.
- **Dedicated IP SSL** — חיוב חודשי משמעותי; כמעט תמיד SNI מספיק.

### 💡 טיפים לחיסכון

- **שפרו את Cache Hit Ratio** — זה החיסכון הגדול ביותר. תעבירו ל-origin רק את מה שבאמת משנה תוכן.
- **הפעילו compression** — פחות GB יוצאים.
- **TTL ארוך יותר לנכסים סטטיים**, עם **versioned keys** במקום invalidations.
- **Price Class** מצומצם כשהקהל מרוכז גיאוגרפית.
- **CloudFront Function במקום Lambda@Edge** בכל מקום שאפשר.
- **CloudFront מול S3** במקום גישה ישירה ל-bucket.
- אל תפעילו **Global Accelerator** אם CloudFront פותר — הוא מוסיף חיוב שעתי קבוע.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 CloudFront מול Global Accelerator מול S3 Cross-Region Replication

| קריטריון | **CloudFront** | **Global Accelerator** | **S3 CRR** |
|---|---|---|---|
| מה זה | CDN עם cache ב-edge | Anycast proxy לרשת AWS | שכפול אובייקטים בין Regions |
| **Cache** | **כן** — זה הלב | **לא** — רק proxy | לא (העתק מלא) |
| פרוטוקולים | **HTTP/HTTPS בלבד** | **TCP ו-UDP** | S3 API |
| כתובות IP | משתנות (DNS) | **2 Anycast IPs סטטיים** | לא רלוונטי |
| היכן מוגש התוכן | **ב-edge** | ב-Region (edge רק מנתב) | ב-Region היעד |
| Failover | לפי origin failover | **אזורי, פחות מדקה** | לא אוטומטי |
| DDoS | Shield | Shield | לא רלוונטי |
| רעננות תוכן | לפי **TTL** | תמיד חי | **near real-time** |
| מתאים ל | תמונות, וידאו, אתרים, API acceleration | **גיימינג (UDP), IoT/MQTT, VoIP**, HTTP שדורש static IP | תוכן דינמי ב-latency נמוך במעט Regions, ריבוי עותקים לרגולציה |

> [!info] שורה תחתונה
> **תוכן שאפשר לשמור ב-cache או HTTP** → CloudFront.
> **לא-HTTP, או צורך ב-IP סטטי, או failover אזורי מהיר** → Global Accelerator.
> **עותק מלא ומעודכן של הנתונים ב-Region אחר** → S3 CRR.
> שניהם הראשונים משתמשים באותה רשת גלובלית ובשניהם יש Shield.

### 5.2 CloudFront Functions מול Lambda@Edge

| קריטריון | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| שפות | **JavaScript** בלבד | **Node.js, Python** |
| קנה מידה | **מיליוני** בקשות לשנייה | **אלפי** בקשות לשנייה |
| Triggers | Viewer Request / Viewer Response | **כל הארבע** (+ Origin Request / Response) |
| **זמן ריצה מקסימלי** | **< 1 ms** | **5–10 שניות** |
| זיכרון מקסימלי | **2 MB** | 128 MB עד 10 GB |
| גודל package | **10 KB** | 1 MB – 50 MB |
| **גישה לרשת** | **לא** | **כן** |
| גישה ל-file system | לא | **כן** |
| **גישה ל-Request Body** | **לא** | **כן** |
| היכן מפתחים | בתוך CloudFront עצמו | ב-**us-east-1**, ואז CloudFront משכפל |
| **עלות יחסית** | **~1/6**, עם free tier | היקר, **בלי** free tier |

**מתי CloudFront Functions:**

- **Cache key normalization** — נרמול headers/cookies/query strings ל-cache key אופטימלי
- **Header manipulation** — הוספה, שינוי או מחיקה של HTTP headers
- **URL rewrites / redirects**
- **Request authentication** — יצירה ואימות של tokens (למשל JWT) כדי לאשר או לדחות בקשה

**מתי Lambda@Edge:**

- זמן ריצה ארוך יותר (כמה מילישניות ומעלה)
- צורך ב-**CPU או זיכרון** מותאמים
- הקוד תלוי ב**ספריות צד שלישי** (למשל AWS SDK כדי לגשת לשירותים אחרים)
- **גישת רשת** לשירותים חיצוניים
- גישה ל-**file system** או ל-**body של הבקשה**

### 5.3 CloudFront מול S3 Transfer Acceleration

| קריטריון | CloudFront | S3 Transfer Acceleration |
|---|---|---|
| כיוון | **Download / Delivery** | **Upload** ל-S3 |
| מנגנון | cache ב-edge | edge כנקודת כניסה לרשת AWS |
| עלות | data transfer + requests | **תוספת** מעל התעריף הרגיל |

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-CloudFront ו-edge delivery** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | ה-edge הוא שכבה שקשה לדבג בלי מדידה | behaviors ו-cache policies ב-IaC; alarms על `5xxErrorRate`, `OriginLatency` ו-**CacheHitRate**; runbook ל-invalidation |
| Security | הכל נכנס דרך ה-edge — שם צריך לעצור | HTTPS בלבד (redirect-to-HTTPS); **WAF** על ה-distribution; **OAC** + Block Public Access על ה-bucket; **Signed URLs/Cookies** לתוכן בתשלום; **Geo Restriction** לרישוי |
| Reliability | CloudFront **אינו** HA ל-origin | Multi-AZ ל-origin; **Origin Groups** ל-failover בין origins; Global Accelerator כשצריך failover אזורי מהיר |
| Performance Efficiency | latency נמוך = פחות פניות ל-origin | TTL נכון; **compression**; cache key מינימלי; CloudFront Function ל-cache key normalization; HTTP/2-HTTP/3 |
| Cost Optimization | כל MISS הוא כסף | להעלות Cache Hit Ratio; **Price Class** לפי קהל; versioned keys במקום invalidations; Functions במקום Lambda@Edge |
| Sustainability | פחות עבודה ופחות בייטים על החוט | caching ו-compression מקטינים גם origin compute וגם data transfer; לא להפעיל Lambda@Edge כשלוגיקה פשוטה מספיקה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "cache static content globally" | **CloudFront** |
| "S3 bucket must not be public" | **OAC** + Block Public Access + bucket policy |
| "gaming / UDP / MQTT / VoIP" | **Global Accelerator** |
| "static IP addresses to whitelist" | **Global Accelerator** (2 Anycast IPs) |
| "fast, deterministic regional failover" | **Global Accelerator** |
| "content must be updated in near real-time in another region" | **S3 Cross-Region Replication** |
| "restrict access by country" | **CloudFront Geo Restriction** |
| "block a specific IP address" | **AWS WAF** על ה-CloudFront distribution |
| "paid content / restricted downloads" | **Signed URLs / Signed Cookies** |
| "content updated but users see old version" | **Invalidation** (או TTL גבוה מדי) |
| "modify headers at massive scale, sub-ms" | **CloudFront Functions** |
| "needs the request body / calls another AWS service" | **Lambda@Edge** |
| "application in private subnet, no public exposure" | **VPC Origins** |
| "accelerate uploads to S3" | **S3 Transfer Acceleration** (לא CloudFront) |
| "certificate for CloudFront" | ACM ב-**us-east-1** |

### טעויות נפוצות

> [!warning] מלכודת 1 — חסימת IP בשכבה הלא נכונה
> **הניסוח:** "יש CloudFront מול ALB. איך חוסמים כתובת IP זדונית?"
> **הטעות:** NACL או Security Group על ה-ALB.
> **הנכון:** **לא עוזר.** התעבורה מגיעה ל-ALB מכתובות ה-edge של CloudFront, לא מה-client.
> החסימה חייבת להיות ב-**edge**: **AWS WAF (IP filtering)** על ה-distribution,
> או **Geo Restriction** אם החסימה היא לפי מדינה.

> [!warning] מלכודת 2 — Global Accelerator עושה cache
> **הניסוח:** "יש לנו תמונות כבדות ומשתמשים גלובליים."
> **הטעות:** Global Accelerator.
> **הנכון:** GA **לא שומר cache בכלל** — הוא proxy. לתוכן שאפשר לשמור ב-cache: **CloudFront**.

> [!warning] מלכודת 3 — CloudFront כפתרון HA
> **הניסוח:** "ה-origin שלנו נופל מדי פעם. CloudFront יפתור?"
> **הטעות:** להניח ש-CloudFront מוסיף זמינות ל-origin.
> **הנכון:** CloudFront **אינו תחליף ל-Multi-AZ**. הוא יגיש cache עד שה-TTL יפוג ואז ייכשל.
> צריך origin חסין (Multi-AZ) ו-**Origin Group** ל-failover.

> [!warning] מלכודת 4 — Cache Hit Ratio נמוך
> **הניסוח:** "הפעלנו CloudFront אבל העומס על ה-origin לא ירד והחשבון עלה."
> **הטעות:** להאשים את ה-CDN.
> **הנכון:** כנראה מעבירים ל-origin יותר מדי headers / cookies / query strings,
> וכל וריאציה יוצרת **cache entry נפרד**. מצמצמים את ה-cache key.

> [!warning] מלכודת 5 — תעודת SSL ב-Region הלא נכון
> **הניסוח:** "יצרנו certificate ב-ACM אבל CloudFront לא מציע אותו."
> **הטעות:** ליצור את התעודה ב-Region של האפליקציה.
> **הנכון:** תעודות ל-CloudFront **חייבות להיות ב-us-east-1**. (ל-ALB — באותו Region של ה-ALB.)

> [!warning] מלכודת 6 — invalidation כשגרה
> **הניסוח:** "בכל deploy אנחנו עושים invalidation ל-`/*`."
> **הטעות:** להתייחס לזה כחינם.
> **הנכון:** invalidations מעבר למכסה החינמית **מחויבים לכל נתיב**, וגם לוקחים זמן.
> **Versioned object keys** (`app.v42.js`) פותרים את זה בעלות אפס.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**
אתר מדיה שמגיש וידאו ותמונות למשתמשים ב-4 יבשות.
דרישות:
(1) latency נמוך בכל העולם;
(2) ה-bucket **לא** יהיה נגיש לציבור;
(3) תוכן פרימיום ייחשף רק למנויים;
(4) חסימת גישה ממדינות שאין בהן רישיון הפצה;
(5) חסימת כתובות IP של bots;
(6) הוספת security headers לכל תשובה;
(7) ה-API הדינמי צריך להיות מואץ, אבל לא נשמר ב-cache.

```text
                          Users (4 continents)
                                  │
                        ┌─────────▼──────────┐
                        │  CloudFront (ACM   │
                        │  cert in us-east-1)│
                        │  + AWS WAF         │
                        │  + Geo Restriction │
                        └──┬──────────────┬──┘
        CloudFront Function│              │
        (security headers, │              │
         cache key norm.)  │              │
                           │              │
        Behavior: /media/* │              │ Behavior: /api/*
        TTL ארוך            │              │ CachingDisabled
                           ▼              ▼
                    ┌──────────┐    ┌──────────────┐
                    │ S3 (OAC) │    │ VPC Origin   │
                    │ Block    │    │ private ALB  │
                    │ Public   │    │ (Multi-AZ)   │
                    │ Access   │    └──────────────┘
                    └──────────┘
        Signed URLs / Signed Cookies לתוכן פרימיום
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **CloudFront** כשכבת הכניסה היחידה | דרישה (1) — cache ב-edge בכל העולם |
| **OAC** + Block Public Access על ה-bucket | דרישה (2) — רק ה-distribution יכול לקרוא מה-bucket |
| **Signed URLs / Signed Cookies** | דרישה (3) — קישור חתום עם תפוגה לתוכן בתשלום |
| **Geo Restriction (Blocklist)** | דרישה (4) — חסימה ברמת מדינה, בדיוק למקרה של רישוי תוכן |
| **AWS WAF** על ה-distribution | דרישה (5) — חסימת IP בשכבת ה-edge; NACL על ה-ALB **לא היה עוזר** |
| **CloudFront Function** ב-Viewer Response | דרישה (6) — הזרקת headers היא בדיוק ה-use case שלה, ובעלות ~1/6 מ-Lambda@Edge |
| **שני Behaviors:** `/media/*` עם TTL ארוך, `/api/*` עם `CachingDisabled` | דרישה (7) — האצה דרך רשת AWS בלי לשמור תשובות דינמיות |
| **VPC Origin** ל-ALB פרטי | ה-ALB לא נחשף לאינטרנט ואין צורך לתחזק allowlist של edge IPs |
| **Versioned keys** לנכסים סטטיים | מונע invalidations חוזרים ועלותם |
| **ACM ב-us-east-1** | חובה ל-CloudFront |

**למה לא Global Accelerator?**
כל התעבורה כאן היא HTTP ורובה **ניתנת ל-cache**. GA לא היה שומר כלום, ה-origin היה מקבל כל בקשה, וזה גם היה יקר יותר.
GA היה מתאים אילו נדרשו **IPs סטטיים** ל-allowlist של לקוח ארגוני, או אילו הפרוטוקול היה UDP.

**למה לא S3 Cross-Region Replication?**
CRR מעתיק את הנתונים ל-Regions בודדים ומייקר את האחסון פי מספר העותקים.
זה פותר רעננות ורגולציה, לא latency גלובלי לתוכן סטטי — לזה CDN עדיף.

**למה לא Lambda@Edge ל-headers?**
זה עובד, אבל זו לוגיקה טריוויאלית ללא גישה ל-body או לרשת.
CloudFront Function עושה זאת ב-sub-ms ובכשישית מהעלות.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **רשימת ה-edge locations** ומיקומם המדויק — משתנה כל הזמן.
- **כל ה-headers של CloudFront** (`CloudFront-Viewer-Country` וכו') — לדעת שקיימים headers גיאוגרפיים מספיק.
- **תחביר מדויק של Cache Policies ו-Origin Request Policies** — לדעת מה הן שולטות בו.
- **Field-Level Encryption** לעומק — לדעת שהוא קיים להצפנת שדות רגישים.
- **כתיבת קוד Lambda@Edge** — צריך לדעת מתי בוחרים אותה, לא איך כותבים.
- **מחירים מדויקים** — לזכור את היחסים: S3→CloudFront = 0, ו-CloudFront זול מ-S3 ישיר.
- **מנגנון RTMP / streaming ישן** — הוסר.
- **תצורת Global Accelerator לעומק** (listeners, endpoint groups, traffic dial) — הרעיון מספיק.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **CloudFront = CDN עם cache ב-edge. Global Accelerator = proxy ברשת AWS, בלי cache.**
- CloudFront: **HTTP/HTTPS בלבד**. GA: **TCP ו-UDP**.
- GA נותן **2 Anycast IPs סטטיים** — מושלם ל-allowlist ול-failover אזורי **בפחות מדקה**.
- **Origins:** S3 (עם **OAC**) · **VPC Origin** (ALB/NLB/EC2 פרטיים) · Custom HTTP · S3 Static Website.
- **OAC** = ה-bucket סגור לגמרי; צריך גם bucket policy. (OAI הוא הישן.)
- **Geo Restriction** = allowlist/blocklist לפי מדינה, לפי Geo-IP של צד שלישי.
- **חסימת IP → WAF ב-edge.** NACL/SG על ה-ALB **לא יעזרו** מאחורי CloudFront.
- **Invalidation** עוקף TTL (`*` או `/images/*`) — אבל **versioned keys** עדיפים.
- **CloudFront Functions:** JavaScript · Viewer Request/Response בלבד · **<1 ms** · 2 MB · **בלי** רשת ובלי body · **~1/6 מהמחיר**.
- **Lambda@Edge:** Node.js/Python · **כל 4 ה-triggers** · **5–10 שניות** · עד 10 GB · **עם** רשת ו-body · נכתב ב-**us-east-1**.
- **תעודת ACM ל-CloudFront חייבת להיות ב-us-east-1.**
- **S3 → CloudFront = 0 עלות.** CloudFront → אינטרנט זול מ-S3 → אינטרנט.
- **Cache Hit Ratio** הוא ה-metric — cache key מפורט מדי הורס ביצועים ועלות.
- **CloudFront אינו HA ל-origin.** צריך Multi-AZ ו-Origin Groups.
- **CRR** = near real-time, לכמה Regions, read-only. **CloudFront** = TTL, גלובלי, לתוכן סטטי.
- **Transfer Acceleration = uploads ל-S3.** CloudFront = delivery.

---

## 11. ✅ בדיקת הבנה

1. איך מוודאים ש-S3 bucket משרת תוכן דרך CloudFront בלבד ואינו נגיש לציבור?
2. משחק רב-משתתפים על UDP צריך latency נמוך גלובלית. CloudFront או Global Accelerator? למה?
3. גילינו IP שמבצע scraping. שמנו אותו ב-NACL של ה-ALB וזה לא עזר. למה, ומה הפתרון?
4. הפעלנו CloudFront אבל העומס על ה-origin לא ירד. מה לבדוק ראשון?
5. צריך להזריק header לכל תשובה, במיליוני בקשות לשנייה. איזו edge function ולמה?
6. הפונקציה צריכה לקרוא את ה-body של הבקשה ולקרוא ל-DynamoDB. איזו edge function?
7. מתי בוחרים S3 Cross-Region Replication במקום CloudFront?

<details>
<summary>תשובות</summary>

1. מפעילים **Block Public Access** על ה-bucket, מגדירים **Origin Access Control (OAC)** ב-distribution, ומוסיפים **bucket policy** שמתירה גישה רק ל-distribution הספציפי. שני החלקים נדרשים — OAC לבד לא מספיק.

2. **Global Accelerator.** CloudFront תומך רק ב-HTTP/HTTPS ולא ב-UDP. GA נותן 2 Anycast IPs, מכניס את התעבורה לרשת AWS בהקדם, ומספק ביצועים עקביים ו-failover אזורי מהיר — בדיוק מה שגיימינג צריך. אין כאן תוכן שאפשר לשמור ב-cache, ולכן ה-cache של CloudFront לא רלוונטי.

3. מאחורי CloudFront, ה-ALB רואה את כתובות ה-**edge locations** של CloudFront ולא את ה-client. לכן NACL/SG לא מזהים את ה-IP הזדוני. הפתרון: **AWS WAF עם IP filtering על ה-CloudFront distribution** — חסימה בשכבת ה-edge, לפני שהבקשה בכלל מגיעה ל-origin.

4. **Cache Hit Ratio.** ככל הנראה מעבירים ל-origin יותר מדי headers, cookies או query strings, וכל וריאציה יוצרת cache entry נפרד. מצמצמים את ה-cache key (אפשר גם עם CloudFront Function ל-normalization) ומעלים TTL לנכסים סטטיים.

5. **CloudFront Function.** היא בנויה בדיוק לזה: header manipulation ב-Viewer Response, זמן ריצה תת-מילישנייה, מיליוני בקשות לשנייה, ובערך שישית מהמחיר של Lambda@Edge. אין צורך ברשת או ב-body.

6. **Lambda@Edge.** רק היא נותנת **גישה ל-request body** ו-**גישת רשת** לשירותים אחרים (AWS SDK). CloudFront Functions חסומות משניהם, ומוגבלות ל-2 MB ולפחות ממילישנייה.

7. כשצריך **עותק מלא ומעודכן near real-time** של הנתונים ב-Region אחר — למשל לדרישת ריבונות נתונים, ל-DR של הנתונים עצמם, או כשהתוכן **דינמי** ומשתנה תדיר כך ש-cache לפי TTL לא מתאים. CloudFront מתאים לתוכן **סטטי** שצריך להיות זמין בכל העולם.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[14 - Route 53 and DNS]] · [[16 - S3 Fundamentals]] · [[17 - S3 Security and Data Management]] · [[18 - S3 Advanced Features]] · [[08 - Elastic Load Balancing]] · [[32 - Security Services]] · [[33 - High Availability and Scalability]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 5962–6235, 8211–8360, 15449–15472
