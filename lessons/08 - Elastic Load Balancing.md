---
lesson: 08
title: Elastic Load Balancing
domain: Design High-Performing Architectures
services: [ELB, ALB, NLB, GWLB, CLB, ACM, EC2 Auto Scaling]
tags: [saa-c03, networking, high-availability, scalability]
---

# 08 — Elastic Load Balancing

> [!abstract] בשורה אחת
> ELB הוא נקודת כניסה מנוהלת אחת שמפזרת תעבורה רק ל-targets בריאים — ובמבחן השאלה תמיד היא *איזה* מארבעת הסוגים.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה DNS round-robin לא מספיק ומה LB מנוהל נותן |
| 2 | איך זה עובד | Listener, Target Group, Health Check, Security Groups |
| 3 | פירוק מפורט | ALB / NLB / GWLB / CLB — כל אחד לעומק |
| 4 | עלות | LCU, NLCU, data processing, cross-AZ |
| 5 | השוואות | **טבלת ההשוואה המרכזית** של ארבעת ה-LBs |
| 6 | Well-Architected | איך LB משרת כל Pillar |
| 7 | מלכודות | מילות מפתח → LB נכון |
| 8 | Scenario | ארכיטקטורה מלאה עם ALB + NLB |

**מונחי מפתח בשיעור:** `Listener` · `Target Group` · `Health Check` · `Cross-Zone` · `SNI` · `Sticky Session` · `Deregistration Delay` · `GENEVE`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- אם הלקוח מכיר את ה-IP של שרת מסוים — נפילת השרת הזה היא נפילת השירות.
- DNS round-robin מפזר עומס, אבל **לא יודע מי חי**. הוא ימשיך לשלוח 50% מהתעבורה לשרת מת.
- TTL של DNS מעכב כל שינוי בדקות עד שעות — לא מספיק מהר ל-scaling אמיתי.
- כל שרת צריך אישור TLS משלו, לנהל אותו ולחדש אותו.
- אין דרך נקייה להפריד תעבורה ציבורית מהשכבה הפרטית.

### מה השירות פותר

- **נקודת גישה אחת** — DNS name יציב שלא משתנה כשה-targets מתחלפים.
- **פיזור עומס** בין targets מרובים ובין AZs.
- **Health Checks** — target שנכשל מוצא מהרוטציה תוך שניות, בלי DNS.
- **SSL/TLS Termination** — ההצפנה נגמרת ב-LB, השרתים משוחררים מעומס הקריפטו.
- **Stickiness** — אותו לקוח חוזר לאותו target כשהאפליקציה שומרת session מקומי.
- **הפרדת public מ-private** — ה-LB לבד חשוף לאינטרנט, ה-targets ב-private subnets.

### למה **Elastic** Load Balancer ולא LB שאתם מרימים

- AWS אחראית לזמינות, לשדרוגים ולתחזוקה של ה-LB עצמו.
- זול יותר להרים HAProxy/nginx לבד — אבל יקר בהרבה בזמן ותפעול.
- אינטגרציה מובנית עם EC2, Auto Scaling, ECS, ACM, CloudWatch, Route 53, WAF ו-Global Accelerator.
- החיסרון: יש רק מעט "כפתורים" להגדרה. שליטה עמוקה יותר → LB עצמאי.

> [!tip] האנלוגיה
> ה-LB הוא המארח בכניסה למסעדה: יש שולחן אחד לתור (DNS אחד),
> הוא יודע איזה מלצר עמוס ואיזה יצא להפסקה (health check), והלקוח לא צריך לדעת שמות מלצרים.

---

## 2. ⚙️ איך זה עובד

### 2.1 שלושת הרכיבים

```text
Client → [Listener: HTTPS:443]
              ↓ (Rules: path / host / header / query)
         [Target Group A]        [Target Group B]
          ↓  health check         ↓  health check
        EC2  EC2  EC2           Lambda / IP / ECS task
```

| רכיב | מה זה | הערה |
|---|---|---|
| **Listener** | פרוטוקול + פורט שה-LB מאזין בו | HTTP:80, HTTPS:443, TCP:1234, UDP… |
| **Rule** | תנאי ניתוב מ-listener ל-target group | ב-ALB בלבד |
| **Target Group** | אוסף targets + הגדרות health check | ה-health check מוגדר **ברמת ה-target group** |
| **Target** | EC2 / IP פרטי / Lambda / ALB (תחת NLB) | לפי סוג ה-LB |

### 2.2 Health Checks — הלב של ה-LB

- ה-health check מוגדר על **פורט + נתיב** (הנפוץ: `/health`).
- תשובה שאינה `200 OK` → ה-target מסומן `unhealthy` ומוצא מהרוטציה.
- אין מדובר בתיקון — ה-LB רק **מפסיק לשלוח** תעבורה. את ההחלפה עושה ה-ASG.
- ההגדרות: interval, timeout, healthy threshold, unhealthy threshold.
- ALB בודק ב-HTTP/HTTPS; NLB תומך גם ב-TCP וגם ב-HTTP/HTTPS.

> [!warning] health check שטחי מדי
> health check שמחזיר 200 מ-nginx בלי לבדוק את ה-DB יסמן כ"בריא" שרת שלא יכול לשרת אף בקשה.
> health check עמוק (שנוגע בתלויות) הוא הבחירה הנכונה — כל עוד הוא לא יקר מדי.

### 2.3 Security Groups — הדפוס הקלאסי

```text
Internet ──443──► [ALB SG: allow 443 from 0.0.0.0/0]
                        │
                        ▼
              [App SG: allow 8080 from ALB-SG]  ← לא מ-CIDR, מ-SG!
                        │
                     EC2 ×N  (private subnets)
```

- **SG של ה-LB:** מאפשר 80/443 מכל מקום (או מ-CIDR מוגבל).
- **SG של האפליקציה:** מאפשר את פורט האפליקציה **רק כמקור ה-SG של ה-LB**.
- זה מבטיח שאף אחד לא יכול לעקוף את ה-LB ולפנות ישירות ל-EC2.
- **טעות נפוצה:** להניח שה-SG של ה-LB "פותח" אוטומטית גישה ל-EC2. הוא לא. צריך כלל נפרד ב-SG של האפליקציה.

### 2.4 Internal מול Internet-facing

| סוג | כתובות | שימוש |
|---|---|---|
| **Internet-facing** | IP ציבוריים ב-public subnets | כניסה מהאינטרנט |
| **Internal** | IP פרטיים בלבד | תעבורה בין שכבות פנימיות (web → app → data) |

- זו בחירה שנעשית **ביצירה** ולא ניתנת לשינוי אחר כך.

---

## 3. 🔍 פירוק מפורט — ארבעת סוגי ה-Load Balancer

### 3.1 Classic Load Balancer (CLB) — הדור הישן

- הדור הראשון (2009). AWS ממליצה לא להשתמש בו בפריסות חדשות.
- תומך TCP (L4) וגם HTTP/HTTPS (L7) — אבל בשטחיות.
- **תומך באישור SSL אחד בלבד** — לכמה domains צריך כמה CLBs.
- **אינו תומך ב-SNI**.
- הפיצ'ר `Connection Draining` נקרא כך רק ב-CLB.

### 3.2 Application Load Balancer (ALB) — Layer 7

**מה הוא עושה:**

- עובד ברמת ה-HTTP: רואה URL, headers, cookies, method.
- תומך **HTTP, HTTPS, HTTP/2, gRPC ו-WebSocket**.
- תומך ב-**redirect** מובנה (למשל HTTP → HTTPS) וב-fixed response.

**ניתוב (Routing Rules) — לב ה-ALB:**

| קריטריון | דוגמה |
|---|---|
| **Path** | `/users` → TG-A, `/search` → TG-B |
| **Hostname** | `one.example.com` → TG-A, `other.example.com` → TG-B |
| **Query String** | `?platform=mobile` → TG-A, `?platform=desktop` → TG-B |
| **HTTP Header** | לפי header מותאם אישית |
| **Source IP / HTTP method** | תנאים נוספים |

**Target Types של ALB:**

| Target | הערה |
|---|---|
| **EC2 instances** | לרוב מנוהלים על ידי ASG |
| **ECS tasks** | כולל **dynamic port mapping** — כמה containers על אותו host |
| **Lambda functions** | הבקשה מתורגמת ל-JSON event |
| **IP addresses** | **חייבות להיות פרטיות** — כולל on-premises דרך VPN/DX |

**מה שחייבים לזכור:**

- ה-EC2 **לא רואה את ה-IP האמיתי של הלקוח** — ALB מסיים את החיבור.
- ה-IP האמיתי מגיע ב-header **`X-Forwarded-For`**.
  בנוסף: `X-Forwarded-Port` ו-`X-Forwarded-Proto`.
- ה-hostname קבוע: `XXX.region.elb.amazonaws.com`.
- ALB הוא ההתאמה המושלמת ל-microservices ולאפליקציות container-based.
- זה ה-LB היחיד שאפשר לשים לפניו **AWS WAF**.

### 3.3 Network Load Balancer (NLB) — Layer 4

- עובד ברמת TCP/UDP/TLS — לא מפרש HTTP.
- **מיליוני בקשות בשנייה** ו-latency נמוך במיוחד (ultra-low).
- **IP סטטי אחד לכל AZ**, ותומך בהצמדת **Elastic IP** — קריטי כשלקוח צריך לעשות whitelist של IP.
- **משמר את ה-IP המקורי של הלקוח** — האפליקציה רואה את ה-source IP האמיתי, בלי `X-Forwarded-For`.

**Target Types של NLB:**

| Target | הערה |
|---|---|
| **EC2 instances** | |
| **IP addresses** | חייבות להיות פרטיות |
| **Application Load Balancer** | דפוס נפוץ: NLB לפני ALB — static IP + routing של L7 ביחד |

- Health checks של NLB תומכים ב-**TCP, HTTP ו-HTTPS**.
- Stickiness ב-NLB היא לפי **source IP / flow**, לא לפי cookie.

> [!info] עדכון מול השקפים
> בשקפים מופיע ש-NLB אינו תומך ב-Security Groups.
> מאז 2023 **NLB כן תומך ב-Security Groups** — אבל חייבים לצרף אותם **ביצירה** של ה-NLB.
> הרעיון הכללי בשאלות המבחן נשאר: כשמדובר ב-NLB בודקים גם את ה-**NACL** ואת ה-SG של ה-target.

### 3.4 Gateway Load Balancer (GWLB) — Layer 3

- נועד לפריסה וניהול של **network virtual appliances** מצד שלישי:
  firewalls, IDS/IPS, Deep Packet Inspection, payload manipulation.
- עובד ב-**Layer 3** — על חבילות IP.
- משלב שני תפקידים:
  1. **Transparent Network Gateway** — נקודת כניסה/יציאה אחת לכל התעבורה.
  2. **Load Balancer** — מפזר את התעבורה בין ה-appliances.
- משתמש בפרוטוקול **GENEVE על פורט 6081** — זו מילת המפתח שמסגירה GWLB בשאלה.
- Targets: EC2 instances או IP פרטיות.
- התעבורה מנותבת אליו דרך **Route Table** ו-**GWLB Endpoint** — היישום שקוף לאפליקציה.

```text
Users → Route Table → GWLB Endpoint → GWLB → [Firewall appliances]
                                                      ↓ (traffic נקי)
                                                 Application
```

### 3.5 Sticky Sessions (Session Affinity)

- מבטיח שאותו לקוח יגיע תמיד לאותו target — כדי לא לאבד session data מקומי.
- נתמך ב-**CLB, ALB ו-NLB**.
- **החיסרון:** stickiness יוצרת **חוסר איזון** בעומס בין ה-targets.

**סוגי cookies ב-ALB:**

| סוג | מי מייצר | שם ה-cookie |
|---|---|---|
| **Duration-based** | ה-Load Balancer | `AWSALB` (ב-CLB: `AWSELB`) |
| **Application-based (LB)** | ה-Load Balancer | `AWSALBAPP` |
| **Application-based (custom)** | ה-**target** (האפליקציה) | שם מותאם, מוגדר לכל target group |

- **אסור** להשתמש בשמות השמורים: `AWSALB`, `AWSALBAPP`, `AWSALBTG`.
- ל-duration-based ול-application-based יש תוקף שאתם קובעים.

> [!tip] הפתרון הנכון לטווח ארוך
> אם התרחיש שואל "איך משפרים איזון עומס באפליקציה עם session" —
> התשובה הטובה היא **להוציא את ה-session החוצה** (ElastiCache / DynamoDB) ולא stickiness.

### 3.6 Cross-Zone Load Balancing

**בלי cross-zone:** כל node של ה-LB מפזר רק בין ה-targets **ב-AZ שלו**.
אם ב-AZ-1 יש 2 targets וב-AZ-2 יש 8, כל target ב-AZ-1 יקבל פי 4 עומס.

**עם cross-zone:** כל node מפזר בין **כל** ה-targets בכל ה-AZs — פיזור אחיד אמיתי.

| Load Balancer | ברירת מחדל | חיוב על תעבורה בין AZ |
|---|---|---|
| **ALB** | **מופעל** (אפשר לכבות ברמת target group) | **אין חיוב** |
| **NLB** | **כבוי** | **יש חיוב** אם מפעילים |
| **GWLB** | **כבוי** | **יש חיוב** אם מפעילים |
| **CLB** | **כבוי** | אין חיוב אם מפעילים |

- זו טבלה שנשאלים עליה ישירות. שווה לשנן.
- **ההיגיון:** ב-ALB זה כלול במחיר ה-LCU; ב-NLB/GWLB התעבורה בין AZ מחויבת בנפרד.

### 3.7 SSL/TLS ו-SNI

- **SSL/TLS Certificate** מצפין את התעבורה בין הלקוח ל-LB (encryption in-flight).
- SSL הוא השם הישן, TLS הוא הפרוטוקול בפועל — כולם עדיין אומרים "SSL".
- אישורים ציבוריים מונפקים על ידי **Certificate Authority** ויש להם תאריך תפוגה.
- ב-AWS מנהלים אותם ב-**ACM (AWS Certificate Manager)** — כולל **חידוש אוטומטי**.
  אפשר גם להעלות אישור עצמאי.

**HTTPS Listener:**

- חובה לציין **אישור ברירת מחדל** אחד.
- אפשר להוסיף רשימת אישורים נוספים לתמיכה בכמה domains.
- אפשר להגדיר **Security Policy** כדי לתמוך בלקוחות ישנים עם TLS ישן.

**SNI — Server Name Indication:**

- פותר את הבעיה של **כמה אישורי SSL על load balancer אחד**.
- הלקוח מציין את ה-hostname כבר ב-handshake הראשוני, וה-LB בוחר את האישור המתאים.
- **עובד ב-ALB, NLB ו-CloudFront. לא עובד ב-CLB.**
- ב-CLB, כמה domains = כמה CLBs. זו מילת מפתח קלאסית לפסילת CLB.

**TLS Termination מול Pass-through:**

| דפוס | מה קורה | מתי |
|---|---|---|
| **Termination ב-LB** | ה-LB מפענח, שולח HTTP פנימה | הנפוץ. מוריד עומס מהשרתים, מאפשר routing ב-L7 |
| **Re-encryption** | ה-LB מפענח ומצפין מחדש ל-backend | דרישות רגולציה של הצפנה end-to-end |
| **TLS Pass-through** (NLB) | ה-LB לא נוגע בהצפנה כלל | כשהאישור חייב להסתיים בשרת עצמו |

### 3.8 Connection Draining / Deregistration Delay

- **שם הפיצ'ר תלוי ב-LB:** ב-CLB זה `Connection Draining`, ב-ALB/NLB זה `Deregistration Delay`.
- מה זה עושה: כש-target מוסר או מסומן unhealthy, ה-LB **מפסיק לשלוח אליו בקשות חדשות**,
  אבל נותן לבקשות שכבר בתהליך להסתיים.
- **טווח: 1–3600 שניות. ברירת מחדל: 300 שניות.**
- אפשר לכבות לגמרי בערך 0.
- **כלל אצבע:** בקשות קצרות → ערך נמוך (deployment מהיר יותר). uploads ארוכים → ערך גבוה.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **שעות LB** | לכל שעה שה-LB קיים, גם אם 0 תעבורה | LB idle עולה כסף |
| **LCU** (ALB) | המקסימום מבין: new connections, active connections, processed bytes, **rule evaluations** | הרבה rules = יותר LCU |
| **NLCU** (NLB) | new connections, active connections, processed bytes | אין rule evaluations |
| **GWLB / GWLBE** | שעות + GB מעובדים בשני הצדדים | פלוס עלות ה-appliances עצמם |
| **Data Transfer** | GB יוצא לאינטרנט; GB בין AZs | ראו [[10 - VPC Internet Connectivity]] |
| **ACM certificates** | **0** לאישורים ציבוריים בשימוש AWS | חידוש אוטומטי חינם |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **ALB אחד עם host/path routing** | הזולה ביותר לכמה שירותים | microservices מרובים תחת domain אחד |
| **ALB לכל microservice** | גבוהה — שעות LB מוכפלות | רק כשנדרשת בידוד מוחלט |
| **NLB** | לרוב זול יותר מ-ALB לאותו נפח bytes | L4 טהור, throughput גבוה |
| **NLB + cross-zone** | מוסיף חיוב תעבורה בין AZ | רק כשהאיזון באמת נדרש |
| **GWLB + appliances** | **היקרה ביותר** | רק כשיש דרישת inspection/compliance |
| **Route 53 weighted/latency** | זול מאוד (לפי query) | פיזור ברמת DNS, לא per-request |

### 🚩 עלויות נסתרות

- **LB ששכחו למחוק** — ממשיך לחייב שעות 24/7 בלי אף בקשה. הבזבוז מספר 1.
- **Rule evaluations ב-ALB** — עשרות rules מעלים את ה-LCU בלי שהתעבורה גדלה.
- **Cross-Zone ב-NLB** — כל בקשה שחוצה AZ מחויבת בשני הכיוונים.
- **Data transfer יוצא** — התשובות שה-LB מחזיר לאינטרנט מחויבות כ-egress.
- **Health checks תכופים מדי** — לא מחויבים ישירות, אבל מייצרים עומס ולוגים.
- **Access Logs ל-S3** — האחסון והבקשות מחויבים; אצל אתר עמוס זה נפח משמעותי.

### 💡 טיפים לחיסכון

- אחדו microservices מאחורי **ALB אחד** עם path/host rules במקום LB לכל שירות.
- מחקו LBs של סביבות dev/test שלא בשימוש — או כבו אותן בלילה.
- שימו **CloudFront לפני ה-ALB** — cache מוריד גם בקשות וגם data transfer יוצא. ראו [[15 - CloudFront and Global Delivery]].
- שקלו לכבות cross-zone ב-NLB אם ה-targets מאוזנים בין ה-AZs ממילא.
- השתמשו ב-**ACM** במקום לקנות אישורים — חינם ומתחדש לבד.
- אל תבחרו GWLB "ליתר ביטחון" — הוא נדרש רק לדרישת inspection אמיתית.

---

## 5. ⚖️ השוואות מכריעות

### הטבלה המרכזית — ALB מול NLB מול GWLB מול CLB

| קריטריון | **ALB** | **NLB** | **GWLB** | **CLB** (legacy) |
|---|---|---|---|---|
| **שכבת OSI** | Layer 7 (Application) | Layer 4 (Transport) | Layer 3 (Network) | L4 + L7 שטחי |
| **פרוטוקולים** | HTTP, HTTPS, HTTP/2, gRPC, WebSocket | TCP, UDP, TLS | IP packets (GENEVE 6081) | HTTP, HTTPS, TCP, SSL |
| **Target types** | EC2, ECS tasks, **Lambda**, IP פרטי | EC2, IP פרטי, **ALB** | EC2, IP פרטי | EC2 בלבד |
| **Static IP / EIP** | ❌ (DNS בלבד) | ✅ IP סטטי לכל AZ + EIP | דרך endpoint | ❌ |
| **TLS Termination** | ✅ + SNI | ✅ + SNI (או pass-through) | לא רלוונטי | ✅ אישור **אחד בלבד**, בלי SNI |
| **ניתוב לפי תוכן** | ✅ path / host / header / query | ❌ | ❌ | ❌ |
| **שימור Source IP** | ❌ (דרך `X-Forwarded-For`) | ✅ מקורי | ✅ שקוף | ❌ |
| **WAF לפניו** | ✅ | ❌ | ❌ | ❌ |
| **Cross-Zone ברירת מחדל** | ✅ מופעל, ללא חיוב | ❌ כבוי, בחיוב | ❌ כבוי, בחיוב | ❌ כבוי, ללא חיוב |
| **ביצועים** | גבוהים | **קיצוניים** — מיליוני req/s, latency מינימלי | לפי ה-appliances | בינוניים |
| **Use case** | web apps, microservices, containers | gaming (UDP), IoT, financial, whitelisting IP | firewall / IDS / DPI fleet | מערכות legacy בלבד |
| **מילות מפתח במבחן** | "path-based", "host-based", "microservices", "WAF", "Lambda target", "HTTP" | "static IP", "Elastic IP", "UDP", "millions of requests", "ultra-low latency", "preserve source IP", "TCP" | "third-party appliances", "GENEVE", "6081", "deep packet inspection", "transparent" | "single certificate", "legacy", "old generation" |

### ELB מול חלופות אחרות

| קריטריון | ELB | Route 53 | CloudFront | Global Accelerator |
|---|---|---|---|---|
| רמת פעולה | per-request | per-DNS-query | per-request, edge | per-connection, edge |
| מודע לבריאות | ✅ מיידי | ✅ עם health checks, אבל TTL מעכב | ✅ דרך origin | ✅ |
| היקף | Region אחד | גלובלי | גלובלי (cache) | גלובלי (static anycast IP) |
| מתי | פיזור בתוך Region | פיזור/failover בין Regions | תוכן סטטי ודינמי עם cache | TCP/UDP גלובלי, IP קבועים |

> [!info] שורה תחתונה
> **HTTP/HTTPS עם החלטת ניתוב → ALB.**
> **TCP/UDP, static IP, throughput קיצוני → NLB.**
> **appliance צד-שלישי במסלול → GWLB.**
> **CLB → כמעט אף פעם**, אלא אם התרחיש מדבר על מערכת legacy קיימת.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-ELB** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | לראות מה קורה בשכבת הכניסה לפני שהמשתמשים מתלוננים | alarms על `HealthyHostCount` ועל `HTTPCode_ELB_5XX_Count`; Access Logs ל-S3; blue/green עם שני target groups |
| **Security** | ה-LB הוא הפנים היחידות שחשופות לאינטרנט | TLS עם ACM; WAF לפני ALB; SG-to-SG במקום CIDR; targets ב-private subnets |
| **Reliability** | נפילת target או AZ שלם לא מורגשת | ≥2 AZs עם targets בכל אחת; health check עמוק; deregistration delay מכויל |
| **Performance Efficiency** | לבחור את השכבה הנכונה במקום את ה"חזק ביותר" | NLB ל-L4 טהור; HTTP/2 ו-keep-alive ב-ALB; TLS termination משחרר CPU מהשרתים |
| **Cost Optimization** | פחות LBs, פחות bytes | ALB אחד עם rules במקום LB לכל שירות; CloudFront לחיסכון ב-egress; מחיקת LBs idle |
| **Sustainability** | פחות עבודה מיותרת על החומרה | cache ב-CloudFront מונע בקשות מיותרות; scale-in של targets; health check בתדירות סבירה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "route `/api` and `/images` differently" | **ALB** — path-based routing |
| "static IP address" / "whitelist an IP in a firewall" | **NLB** (+ Elastic IP) |
| "millions of requests per second" / "ultra-low latency" | **NLB** |
| "UDP traffic" (gaming, DNS, VoIP) | **NLB** — ALB לא תומך ב-UDP |
| "third-party firewall appliances" / "GENEVE" / "port 6081" | **GWLB** |
| "protect against SQL injection / OWASP" | **AWS WAF לפני ALB** |
| "invoke a serverless function from the load balancer" | **ALB עם Lambda target** |
| "multiple SSL certificates on one load balancer" | **SNI** — ALB או NLB (לא CLB) |
| "application must see the real client IP" | **NLB** (משמר), או header `X-Forwarded-For` ב-ALB |
| "requests dropped during deployment" | **Deregistration Delay / Connection Draining** |
| "uneven load across AZs" | **Cross-Zone Load Balancing** |
| "global routing / multi-Region failover" | **Route 53** או **Global Accelerator**, לא ELB |
| "single certificate only, old generation" | **CLB** — וזו סיבה לפסול אותו |

### טעויות נפוצות

> [!warning] מלכודת 1 — "NLB כי הוא הכי מהיר"
> **הניסוח:** "The application needs high performance and routes `/api` to a different fleet."
> **הטעות:** לבחור NLB בגלל המילה performance.
> **הנכון:** NLB עובד ב-L4 ו**לא רואה URL כלל**. ניתוב לפי path מחייב **ALB**.

> [!warning] מלכודת 2 — SG של ה-LB מספיק
> **הניסוח:** "We opened 443 on the ALB security group but targets show unhealthy."
> **הטעות:** לחפש את הבעיה ב-health check path.
> **הנכון:** ה-SG של ה-**EC2** חייב כלל inbound שמאפשר את פורט האפליקציה **ממקור ה-SG של ה-ALB**.

> [!warning] מלכודת 3 — הוספת AZ בלי targets
> **הניסוח:** "We added a third AZ to the load balancer but availability didn't improve."
> **הטעות:** להניח שהוספת subnet ל-LB מספיקה.
> **הנכון:** צריך targets **בפועל** באותה AZ, וגם route table ו-health check תקינים.

> [!warning] מלכודת 4 — Stickiness כפתרון scaling
> **הניסוח:** "Enable sticky sessions to improve performance."
> **הטעות:** לחשוב ש-stickiness מייעלת פיזור.
> **הנכון:** stickiness דווקא **מקלקלת את האיזון**. הפתרון הנכון ל-session הוא externalization ל-ElastiCache/DynamoDB.

> [!warning] מלכודת 5 — ELB ל-failover בין Regions
> **הניסוח:** "Fail over to another Region if the primary is down."
> **הטעות:** לבחור ELB.
> **הנכון:** ELB הוא **תוך-Region**. Failover בין Regions הוא **Route 53** (ראו [[14 - Route 53 and DNS]]) או Global Accelerator.

> [!warning] מלכודת 6 — Cross-Zone חינם תמיד
> **הניסוח:** "Enable cross-zone load balancing on the NLB at no additional cost."
> **הטעות:** להעתיק את ההתנהגות של ALB.
> **הנכון:** ב-**ALB** cross-zone דלוק כברירת מחדל וללא חיוב. ב-**NLB/GWLB** הוא כבוי כברירת מחדל ו**מחויב** על התעבורה בין ה-AZs.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חנות online עם שלושה microservices (`/catalog`, `/checkout`, `/images`), שני domains
(`www.shop.com` ו-`api.shop.com`), הגנה מפני OWASP, ובנוסף שירות תשלומים ב-TCP
שספק חיצוני דורש להוסיף IP קבוע ל-firewall שלו.

```text
                     Route 53
                        ↓
                    AWS WAF
                        ↓
        ┌── Application Load Balancer (public subnets, 2 AZ) ──┐
        │  HTTP:80  → redirect 301 → HTTPS:443 (ACM + SNI)     │
        │  host www.shop.com  → /catalog  → TG-catalog         │
        │                     → /checkout → TG-checkout        │
        │                     → /images   → TG-images          │
        │  host api.shop.com  → TG-api                         │
        └──────────────────────────────────────────────────────┘
                        ↓ (SG-to-SG, port 8080)
              EC2 / ECS tasks in private subnets (ASG)
                        ↓
                  RDS Multi-AZ

     Payment partner ──► NLB (Elastic IP per AZ, TCP:9000) ──► payment fleet
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **ALB אחד** לכל ה-HTTP | host + path routing מחליף שלושה LBs נפרדים — חיסכון בשעות LB |
| **ACM + SNI** על listener אחד | שני domains, שני אישורים, LB אחד. CLB לא היה מסוגל |
| **Redirect HTTP→HTTPS ב-ALB** | פיצ'ר מובנה, אין צורך בקוד בשרת |
| **WAF לפני ה-ALB** | הדרישה ל-OWASP; WAF עובד רק מול ALB (ו-CloudFront/API GW) |
| **Target Group לכל microservice** | health check נפרד לכל שירות; ASG נפרד; deployment עצמאי |
| **ECS tasks כ-targets** | dynamic port mapping מאפשר כמה containers על אותו host |
| **SG-to-SG** על פורט 8080 | אף אחד לא יכול לעקוף את ה-ALB ולפנות ישירות ל-EC2 |
| **NLB עם EIP** לשירות התשלומים | הספק דורש IP קבוע ל-whitelist — רק NLB נותן את זה |
| **Deregistration delay = 30s** | הבקשות קצרות; deployment מהיר בלי לנתק לקוחות |
| **Cross-zone**: דלוק ב-ALB (חינם), כבוי ב-NLB | חיסכון בתעבורה בין AZ ב-NLB, כי ה-targets מאוזנים |

**למה לא NLB לכל הארכיטקטורה?**
כי NLB לא רואה URL. `/catalog` מול `/checkout` דורש L7.

**למה לא CLB?**
אישור SSL אחד בלבד, אין SNI, אין path routing, אין Lambda targets. פשוט לא רלוונטי.

**למה לא GWLB?**
אין כאן דרישה ל-appliance של צד שלישי. GWLB היה מוסיף עלות ומורכבות בלי ערך.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **מחירי LCU/NLCU מדויקים** — משתנים לפי Region. מספיק להבין מה משפיע עליהם.
- **הנוסחה המדויקת** של חישוב LCU (rule evaluations × בקשות). מספיק לדעת שרולים משפיעים.
- **רשימת כל ה-Security Policies** של TLS. מספיק לדעת שיש מנגנון לתמיכה בלקוחות ישנים.
- **המבנה הפנימי של פרוטוקול GENEVE** — מספיק "GENEVE, פורט 6081, GWLB".
- **תחביר של ALB rules ב-CloudFormation/CLI**.
- **הגדרות CLB לעומק** — הוא legacy; מספיק לדעת למה פוסלים אותו.
- **ערכי ברירת מחדל של health check interval/threshold** — מספיק להבין את המנגנון.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **ALB = L7** — path/host/header/query routing, HTTP/2, WebSocket, gRPC, redirects.
- **NLB = L4** — TCP/UDP/TLS, **static IP לכל AZ + EIP**, מיליוני req/s, latency מינימלי.
- **GWLB = L3** — appliances של צד שלישי, **GENEVE על פורט 6081**.
- **CLB = legacy** — אישור SSL אחד, בלי SNI, בלי path routing.
- **Target types:** ALB → EC2/ECS/**Lambda**/IP · NLB → EC2/IP/**ALB** · GWLB → EC2/IP.
- **Health check מוגדר ברמת ה-Target Group**, לא ה-LB.
- **תשובה שאינה 200 = unhealthy.** ה-LB מפסיק לשלוח; ה-ASG הוא שמחליף.
- **ALB לא משמר source IP** — הוא ב-`X-Forwarded-For`. **NLB כן משמר.**
- **Cross-Zone:** ALB דלוק+חינם · NLB/GWLB כבוי+בתשלום · CLB כבוי+חינם.
- **SNI** = כמה אישורי SSL על LB אחד — **ALB, NLB, CloudFront. לא CLB.**
- **ACM** מנפיק ומחדש אישורים ציבוריים **בחינם** לשימוש ב-ELB/CloudFront.
- **Sticky sessions:** `AWSALB` (duration) · `AWSALBAPP` (application) · שמות שמורים אסורים.
- **Deregistration Delay / Connection Draining:** 1–3600 שניות, **ברירת מחדל 300**.
- **WAF עובד רק לפני ALB** (וגם CloudFront ו-API Gateway) — לא לפני NLB.
- **ELB הוא תוך-Region.** בין Regions: Route 53 או Global Accelerator.
- **SG של ה-target חייב לאפשר תעבורה מ-SG של ה-LB** — לא מספיק לפתוח את ה-LB.

---

## 11. ✅ בדיקת הבנה

1. אפליקציה צריכה לנתב `/api` ל-fleet אחד ו-`/static` לאחר, וגם לעמוד ב-latency נמוך. איזה LB?
2. ספק חיצוני דורש שתספקו לו IP קבוע להוסיף ל-firewall שלו. מה הפתרון?
3. פתחתם 443 ב-SG של ה-ALB, אבל כל ה-targets `unhealthy`. מה כנראה חסר?
4. יש לכם שני domains ואתם רוצים LB אחד. איזה מנגנון, ואיזה LB לא יכול?
5. ב-NLB רב-AZ, ה-targets ב-AZ אחת מקבלים פי 4 עומס מהשנייה. מה גורם לזה, ומה המחיר של התיקון?
6. האפליקציה רואה תמיד את אותו IP בכל הבקשות. איזה LB עומד לפניה ואיפה ה-IP האמיתי?
7. בכל deployment חלק מהמשתמשים מקבלים שגיאת connection reset. מה מכווננים?
8. צריך לפרוס firewall של ספק צד-שלישי שיבדוק את כל התעבורה ל-VPC. מה בוחרים ואיזה פרוטוקול מסגיר את זה?

<details>
<summary>תשובות</summary>

1. **ALB.** ניתוב לפי path הוא Layer 7 ו-NLB לא רואה URL. ה-latency של ALB מספיק טוב לרוב אפליקציות ה-web; "latency נמוך" הופך למכריע רק כשמדובר ב-L4 טהור.
2. **NLB עם Elastic IP** — IP סטטי לכל AZ. ALB מספק רק DNS name שה-IP שמאחוריו משתנה.
3. חסר כלל inbound ב-**Security Group של ה-EC2** שמאפשר את פורט האפליקציה **ממקור ה-SG של ה-ALB**. ה-SG של ה-LB לא פותח כלום בצד ה-target.
4. **SNI** — הלקוח מציין את ה-hostname ב-handshake וה-LB בוחר אישור. עובד ב-**ALB ו-NLB**; **CLB לא תומך** ומוגבל לאישור אחד.
5. **Cross-Zone Load Balancing כבוי** — זו ברירת המחדל ב-NLB. הפעלה תאזן את הפיזור אבל **מוסיפה חיוב על תעבורה בין AZs**. חלופה בלי עלות: לאזן את מספר ה-targets בין ה-AZs.
6. זהו **ALB** — הוא מסיים את החיבור ולכן השרת רואה את ה-IP הפרטי שלו. ה-IP האמיתי נמצא ב-header **`X-Forwarded-For`** (ולצידו `X-Forwarded-Port` ו-`X-Forwarded-Proto`).
7. **Deregistration Delay** (ב-CLB: Connection Draining). ברירת המחדל 300 שניות; אם הוא 0 או נמוך מדי, בקשות באוויר נקטעות בזמן ה-deployment.
8. **Gateway Load Balancer**. מילות המפתח: **GENEVE על פורט 6081**, "transparent network gateway", "third-party virtual appliances".

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[07 - Auto Scaling]] · [[09 - VPC Fundamentals]] · [[11 - VPC Security]] · [[14 - Route 53 and DNS]] · [[15 - CloudFront and Global Delivery]] · [[26 - Containers]] · [[32 - Security Services]] · [[33 - High Availability and Scalability]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 1884–2389
