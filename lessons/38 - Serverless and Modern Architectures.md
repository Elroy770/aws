---
lesson: 38
title: Serverless and Modern Architectures
domain: Design High-Performing Architectures
services: [Lambda, API Gateway, DynamoDB, Cognito, S3, CloudFront, SES, SQS, SNS, EventBridge, Step Functions, Fargate, DAX, Aurora Serverless]
tags: [saa-c03, serverless, architecture-patterns, microservices, event-driven]
---

# 38 — Serverless and Modern Architectures

> [!abstract] בשורה אחת
> השיעור הזה מלמד את **דפוסי הארכיטקטורה ה-serverless המלאים** שחוזרים במבחן — API, אתר גלובלי, thumbnails, welcome email, אפליקציה גלובלית — כך שברגע שתזהה את התיאור, הדיאגרמה תיבנה לך בראש לבד.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | מה זה serverless באמת | ההגדרה המדויקת ומי נמצא ברשימה |
| 2 | דפוס 1 — Serverless REST API | API Gateway + Lambda + DynamoDB + Cognito |
| 3 | דפוס 2 — אתר סטטי גלובלי ומאובטח | CloudFront + S3 + OAC |
| 4 | דפוס 3 — Thumbnail Generation | S3 Event Notifications → Lambda |
| 5 | דפוס 4 — Welcome Email | DynamoDB Streams → Lambda → SES |
| 6 | דפוס 5 — אפליקציה גלובלית | Global Tables + DAX + caching רב-שכבתי |
| 7 | דפוס 6 — Offloading בלי לשנות קוד | CloudFront לפני אפליקציה קיימת |
| 8 | Microservices | סינכרוני מול אסינכרוני, והאתגרים |
| 9 | עלות ומלכודות | מתי serverless דווקא יקר יותר |

**מונחי מפתח בשיעור:** `FaaS` · `Cognito Identity Pool` · `OAC` · `DAX` · `DynamoDB Streams` · `S3 Event Notification` · `Global Tables` · `Idempotency`

---

## 1. 🎯 מה זה Serverless באמת

### ההגדרה

- Serverless **לא** אומר שאין שרתים. הוא אומר שאתה **לא מנהל, לא מקצה ולא רואה** אותם.
- בהתחלה serverless היה שם נרדף ל-**FaaS** (Function as a Service) — כלומר Lambda.
- היום המונח כולל **כל שירות מנוהל** שמתרחב לבד ומחייב לפי שימוש: databases, messaging, storage.

### מי ברשימת ה-Serverless של AWS

| קטגוריה | השירותים |
|---|---|
| Compute | **Lambda**, **Fargate** |
| API | **API Gateway** |
| Database | **DynamoDB**, **Aurora Serverless** |
| Storage | **S3** |
| Auth | **Cognito** |
| Messaging | **SNS**, **SQS**, EventBridge |
| Streaming | **Kinesis Data Firehose** |
| Orchestration | **Step Functions** |
| Email | SES |

> [!tip] האנלוגיה
> Serverless הוא כמו מונית לעומת רכב פרטי:
> אתה משלם רק על הנסיעה, לא על החניה, הביטוח והטיפולים — אבל אין לך שליטה על סוג הרכב.

### מה serverless **לא** פותר

- עדיין צריך לתכנן **IAM, data model, error handling ו-idempotency**.
- עדיין יש **מגבלות**: timeout, זיכרון, גודל payload, concurrency.
- עדיין יש **cold starts** שמשפיעים על latency בזנב.
- "בלי לנהל שרתים" ≠ "בלי אחריות ארכיטקטונית".

---

## 2. ⚙️ דפוס 1 — Serverless REST API (MyTodoList)

**הדרישה:** אפליקציית מובייל, REST API על HTTPS, ארכיטקטורה serverless, המשתמשים ניגשים ישירות לתיקייה שלהם ב-S3, אימות דרך שירות מנוהל, יותר קריאות מכתיבות.

### 2.1 הארכיטקטורה

```text
                      ┌────────────────┐
Mobile Client ──auth──►    Cognito     │  (User Pool = מי אתה)
      │                └────────┬───────┘
      │                         │ temporary credentials (Identity Pool)
      │                         ▼
      │                    ┌────────┐
      ├──REST/HTTPS───────►│  API   │
      │                    │Gateway │──invoke──► Lambda ──query──► DynamoDB
      │                    └────────┘                                 ▲
      │                     (response                                 │
      │                      caching)                              [DAX cache]
      │
      └──direct SDK, temp creds──► S3 bucket (prefix של המשתמש בלבד)
```

### 2.2 מי עושה מה ולמה

| רכיב | התפקיד | למה דווקא הוא |
|---|---|---|
| **Cognito User Pool** | מי המשתמש (authentication) | שירות אימות **מנוהל** — הדרישה במפורש |
| **Cognito Identity Pool** | credentials זמניים מ-IAM | מאפשר גישה **ישירה** ל-AWS מהמכשיר, ללא backend |
| **API Gateway** | חשיפת HTTPS, authorizer, throttling, caching | לא מנהלים web server |
| **Lambda** | לוגיקת ה-CRUD | scale-to-zero, תשלום לפי שימוש |
| **DynamoDB** | אחסון ה-todos | serverless, scale אוטומטי, latency single-digit ms |
| **DAX** | cache ל-DynamoDB | הדרישה "high read throughput"; latency במיקרו-שניות |
| **S3** | קבצי המשתמשים | גישה ישירה מהמכשיר עם policy מוגבלת ל-prefix |

### 2.3 הטריק המרכזי — גישה ישירה ל-S3

- ה-Identity Pool מנפיק **temporary IAM credentials** לכל משתמש מאומת.
- ה-IAM policy מוגבלת ל-prefix של אותו משתמש: `arn:aws:s3:::bucket/${cognito-identity.amazonaws.com:sub}/*`.
- התוצאה: העלאות והורדות **לא עוברות דרך ה-backend** בכלל.
- **החיסכון:** אין Lambda שמעבירה בייטים, אין תשלום duration על העברת קבצים, אין מגבלת payload של API Gateway.
- אותו דפוס עובד גם ל-DynamoDB ול-Lambda ישירות מהקליינט.

### 2.4 שתי שכבות ה-caching

| שכבה | מה נשמר | מתי כדאי |
|---|---|---|
| **API Gateway response cache** | התשובה השלמה של ה-endpoint | קריאות זהות חוזרות; חוסך גם Lambda invocations |
| **DAX** | תוצאות query מ-DynamoDB | קריאות שונות אך אותו דאטה; latency במיקרו-שניות |

> [!info] שורה תחתונה
> API Gateway cache חוסך את **כל** ה-stack מטה (Lambda + DynamoDB).
> DAX חוסך רק את ה-DynamoDB, אך מתאים כשהתשובות אינן זהות ברמת ה-HTTP.

### 2.5 החלופות ולמה הן פחות טובות

| חלופה | למה פחות טובה כאן |
|---|---|
| ALB + EC2/ECS במקום API Gateway + Lambda | יש שרתים לנהל, אין scale-to-zero, משלמים גם ב-0 תעבורה |
| RDS במקום DynamoDB | לא serverless (אלא Aurora Serverless), scaling מורכב, latency גבוה יותר |
| ElastiCache במקום DAX | דורש שינוי בקוד לניהול cache; DAX הוא **drop-in** עם אותו API של DynamoDB |
| Lambda שמעבירה קבצים ל-S3 | יקר, מגביל payload, מיותר — Cognito credentials פותרים את זה |

**מה זה עולה?** תשלום לפי requests ב-API Gateway, לפי invocations+GB-seconds ב-Lambda, ולפי RCU/WCU ב-DynamoDB. DAX הוא **cluster שמחויב לפי שעה** — הוא ההוצאה הקבועה היחידה בדפוס הזה.

---

## 3. 🌍 דפוס 2 — אתר סטטי גלובלי ומאובטח (MyBlog.com)

**הדרישה:** אתר שמתרחב גלובלית, נכתב לעיתים רחוקות ונקרא הרבה, חלקו סטטי וחלקו REST API דינמי, caching בכל מקום שאפשר.

### 3.1 הארכיטקטורה

```text
Client (עולמי)
    │
    ▼
┌──────────────────┐
│   CloudFront     │  Global distribution, edge locations
└────┬────────┬────┘
     │        │
     │ /*     │ /api/*
     ▼        ▼
┌────────┐  ┌────────────┐
│   S3   │  │ API Gateway│──► Lambda ──► DAX ──► DynamoDB
│(static)│  └────────────┘                        │
└────────┘                                  Global Tables
   ▲                                       (replicas ב-Regions)
   │
Bucket Policy: מרשה **רק** ל-CloudFront Distribution (OAC)
```

### 3.2 למה כל רכיב

| החלטה | הנימוק |
|---|---|
| CloudFront לפני S3 | תוכן סטטי מוגש מ-**edge** קרוב למשתמש → latency נמוך גלובלית |
| **OAC** (Origin Access Control) | ה-bucket נשאר **פרטי לחלוטין**; רק ה-distribution יכול לקרוא ממנו |
| bucket policy שמגבילה ל-distribution | בלי זה, אפשר לעקוף את CloudFront ולגשת ישירות ל-S3 |
| API Gateway חשוף בלי Cognito | ה-API של הבלוג הוא **ציבורי** — אין צורך באימות |
| DAX | "נכתב לעיתים רחוקות, נקרא הרבה" — cache הוא בדיוק הדפוס הזה |
| **DynamoDB Global Tables** | multi-region, **active-active** — קוראים מהעותק המקומי בכל Region |

> [!warning] OAC מול OAI
> **OAI (Origin Access Identity)** הוא המנגנון הישן.
> **OAC (Origin Access Control)** הוא היורש המומלץ: תומך בכל ה-Regions, ב-SSE-KMS וב-HTTP methods נוספים.
> אם בשאלה מופיעות שתי האפשרויות — **OAC** היא התשובה העדכנית.

### 3.3 Global Tables — למה זה הרכיב שהופך את זה לגלובלי

- Replication **active-active** בין Regions — כותבים לכל Region וקוראים מכל Region.
- ה-Lambda בכל Region עובדת מול ה-replica **המקומית** → latency נמוך לכתיבה ולקריאה.
- דורש **DynamoDB Streams מופעל**.
- החלופה: **Aurora Global Database** — אבל היא **active-passive** (כתיבה רק ב-primary Region, קריאה בכל מקום).

| | DynamoDB Global Tables | Aurora Global Database |
|---|---|---|
| מודל | active-**active** | active-**passive** |
| כתיבה | בכל Region | רק ב-primary |
| Failover | מיידי (כותבים לכל Region ממילא) | promote של secondary, ~דקה |
| מודל נתונים | NoSQL | SQL רלציוני |
| Lag בין Regions | ~שנייה | < שנייה |

### 3.4 החלופות ולמה הן פחות טובות

| חלופה | הבעיה |
|---|---|
| S3 Static Website Hosting ישירות לאינטרנט | ה-bucket חייב להיות ציבורי, אין HTTPS מותאם, אין edge caching, egress יקר יותר |
| ALB + EC2 שמגישים HTML | שרתים לנהל, לא מתרחב גלובלית בלי עבודה, יקר יותר |
| Multi-Region עם replication ידני של S3 | CloudFront כבר פותר את הגלובליות; CRR נדרש רק ל-DR |

**מה זה עולה?** התעבורה **מ-S3 ל-CloudFront חינם**, וה-egress מ-CloudFront לאינטרנט זול מ-egress ישיר מ-S3.
בנוסף, ה-cache מקטין דרסטית את מספר ה-GET requests ל-S3. זה דפוס שהוא גם מהיר יותר וגם זול יותר.

---

## 4. 🖼️ דפוס 3 — Thumbnail Generation

**הדרישה:** כל תמונה שמועלית לבלוג צריכה לקבל thumbnail אוטומטית.

### 4.1 הארכיטקטורה

```text
                Transfer Acceleration (העלאות מרחוק)
Client ──upload──► S3 (bucket תמונות מקור)
                        │
                        │ S3 Event Notification (s3:ObjectCreated:*)
                        ▼
              ┌──────────────────────┐
              │  Lambda (resize)     │───write──► S3 (bucket thumbnails)
              └──────────────────────┘
                        │
        ואפשר במקביל גם: ──► SQS  (עיבוד כבד/מבוקר בקצב)
                            ──► SNS  (fan-out לכמה צרכנים)
```

### 4.2 החלטות מפתח

| החלטה | הנימוק |
|---|---|
| S3 Event Notification ולא polling | מונע סריקת bucket יקרה ואיטית; אירוע דוחף מיידית |
| Lambda ולא EC2 | העבודה ספורדית ו-burst-י — scale-to-zero, תשלום לפי שימוש |
| **bucket יעד נפרד** ל-thumbnails | מונע **לולאה אינסופית** — אחרת ה-thumbnail עצמו מפעיל את ה-Lambda שוב |
| SQS בין S3 ל-Lambda (אופציונלי) | buffer מול spike של אלפי העלאות; DLQ להודעות שנכשלו |
| SNS כשצריך כמה צרכנים | fan-out לאותו אירוע: גם thumbnail, גם indexing, גם audit |
| S3 Transfer Acceleration | העלאות ממשתמשים רחוקים גיאוגרפית — נכנסות דרך edge location |

> [!warning] מלכודת הלולאה
> Lambda שקוראת מ-bucket X וכותבת thumbnail **לאותו bucket** מפעילה את עצמה שוב ושוב.
> הפתרונות: bucket יעד נפרד, או prefix filter ב-event notification, או שניהם.

### 4.3 החלופות

| חלופה | למה פחות טובה |
|---|---|
| EC2 שסורק את ה-bucket כל דקה | latency גבוה, עלות רצה 24/7, LIST requests יקרים |
| **EventBridge** במקום S3 Event Notification | לגיטימי לגמרי, ואף עדיף כשצריך filtering מתוחכם או ריבוי יעדים |
| Lambda@Edge | מיועד להתאמה בזמן הגשה, לא לעיבוד batch של קבצים |

**מה זה עולה?** רק invocations ו-GB-seconds של Lambda + אחסון ה-thumbnails.
כשאין העלאות — **אפס**. זו הסיבה שזה הדפוס הקנוני של serverless.

---

## 5. 📧 דפוס 4 — Welcome Email Flow

**הדרישה:** כל משתמש חדש שנרשם מקבל מייל ברוכים הבאים.

### 5.1 הארכיטקטורה

```text
API Gateway ──► Lambda ──PutItem──► DynamoDB (טבלת users)
                                        │
                                        │ DynamoDB Streams
                                        │ (רק INSERT מעניין אותנו)
                                        ▼
                                    Lambda ──[IAM Role]──► SES ──► המייל
```

### 5.2 למה דווקא Streams ולא קריאה ישירה ל-SES

| החלטה | הנימוק |
|---|---|
| **DynamoDB Streams** כטריגר | הפרדה מלאה — ה-API מחזיר תשובה מיד, המייל נשלח אסינכרונית |
| Lambda שנייה ולא אותה Lambda | כשל בשליחת מייל לא יכשיל את ההרשמה עצמה |
| **IAM Role** ל-Lambda עם הרשאת SES | אין credentials בקוד; least privilege |
| **SES** ולא SMTP חיצוני | שירות מנוהל, serverless, משתלב ב-IAM |

- ה-Stream מעביר את ה-record של השינוי (INSERT/MODIFY/REMOVE) עם **StreamViewType** שנבחר.
- מסננים ב-Lambda (או ב-Event Source Mapping filter) כדי לטפל רק ב-INSERT.
- הודעות שנכשלות → **DLQ** או retry עם backoff.

> [!warning] Idempotency
> Lambda שנקראת מ-Stream עשויה לקבל את אותו record **יותר מפעם אחת** (at-least-once).
> בלי בדיקת idempotency, המשתמש יקבל שני מיילים.
> הפתרון: לשמור `messageId` שכבר טופל, או לסמן flag ב-item עצמו.

### 5.3 החלופה

- אפשר גם **Cognito Post Confirmation Lambda Trigger** — אם ההרשמה עוברת דרך Cognito.
- אפשר **EventBridge** אם צריך לנתב את "משתמש נרשם" ליותר מצרכן אחד.
- **לא** לשלוח את המייל בתוך ה-request הסינכרוני — זה מאט את המשתמש וקושר את הגורלות.

---

## 6. 🗺️ דפוס 5 — סיכום האתר הגלובלי המלא

זהו הדפוס המורכב ביותר, והוא מאחד את כל מה שראינו:

```text
                    ┌────────────────────────────┐
   Client ─────────►│        CloudFront          │
                    └──┬──────────────────────┬──┘
                   /*  │                      │ /api/*
                       ▼                      ▼
              S3 (static, פרטי)         API Gateway
              ▲ OAC + bucket policy           │ invoke
              │                               ▼
              │                            Lambda ──► DAX ──► DynamoDB
              │                               │                  │
       upload │                        IAM Role│           Streams│
              │                               ▼                  ▼
        S3 (uploads) ──event──► Lambda      SES              Lambda
                                  │       (welcome           (thumbnail
                                  ▼         email)            / index)
                            S3 (thumbnails)
                                                     DynamoDB Global Tables
                                                     → replicas ב-Regions נוספים
```

**מה למדנו מהדפוס:**

- תוכן סטטי מופץ ב-CloudFront מעל S3, עם bucket שנשאר **פרטי**.
- ה-REST API הוא serverless ו-**ציבורי** — ולכן לא נדרש Cognito כאן.
- **Global Table** משרתת את הדאטה גלובלית (אלטרנטיבה: Aurora Global Database).
- **DynamoDB Streams** מפעיל Lambda על כל שינוי.
- ל-Lambda יש **IAM Role** שמאפשר לה להשתמש ב-SES.
- **S3 יכול להפעיל SQS, SNS או Lambda** כדי לדווח על אירועים — שלושת היעדים הקלאסיים.

---

## 7. 🚀 דפוס 6 — Offloading בלי לשנות שורת קוד

**הדרישה:** אפליקציה על EC2 מפיצה עדכוני תוכנה. כשיוצא עדכון — מגיעה מפולת בקשות והתוכן מופץ בהיקף עצום. יקר מאוד ב-CPU וב-bandwidth. **אסור לשנות את האפליקציה.**

### 7.1 המצב הקיים והפתרון

```text
לפני:
Users (המונים) ──► ASG (EC2 × N, מתנפח) ──► EFS (קבצי העדכון)
   העלות: הרבה EC2, הרבה egress, ASG מתפוצץ בכל release

אחרי:
Users ──► CloudFront ──(cache miss בלבד)──► ASG (EC2 × מעט) ──► EFS
   העלות: רוב הבקשות נעצרות ב-edge, ה-ASG כמעט לא מתרחב
```

### 7.2 למה CloudFront הוא התשובה כאן

| נימוק | הסבר |
|---|---|
| **אפס שינוי בארכיטקטורה** | פשוט מציבים distribution לפני ה-ALB/EC2 הקיימים |
| הקבצים **סטטיים לחלוטין** | קובץ עדכון לא משתנה אחרי שפורסם — cache hit ratio קרוב ל-100% |
| ה-EC2 לא serverless — **CloudFront כן** | ה-edge מתרחב במקומנו, בלי שנעשה דבר |
| ה-ASG כמעט לא מתרחב | חיסכון עצום בשעות EC2 |
| egress זול יותר | CloudFront → אינטרנט זול מ-EC2 → אינטרנט |
| זמינות טובה יותר | ה-edge סופג את ה-spike במקום ה-origin |

> [!tip] הכלל למבחן
> "אפליקציה קיימת, אסור לשנות אותה, יש spike של תוכן סטטי, רוצים לחסוך" →
> **CloudFront לפני ה-origin**. זו התשובה כמעט תמיד.

---

## 8. 🧩 Microservices

### 8.1 הסביבה הטיפוסית

```text
                  Route 53 (DNS)
                        │
     ┌──────────────────┼──────────────────┐
     ▼                  ▼                  ▼
service1.example.com  service2...     service3...
     │                  │                  │
    ALB              API Gateway          ALB
     │                  │                  │
    ECS               Lambda          EC2 + ASG
     │                  │                  │
  DynamoDB          ElastiCache          RDS
```

**התובנה המרכזית:** לכל microservice מותר להיות **בנוי אחרת לגמרי**. אין חובה לאחידות טכנולוגית.

### 8.2 סינכרוני מול אסינכרוני

| דפוס | הכלים | מתי |
|---|---|---|
| **סינכרוני** | API Gateway, Load Balancers | הקורא צריך תשובה **עכשיו** |
| **אסינכרוני** | SQS, SNS, Kinesis, Lambda triggers (S3) | אפשר לענות "קיבלתי" ולעבד אחר כך |

### 8.3 האתגרים של Microservices

- **overhead חוזר** בהקמת כל שירות חדש (CI/CD, ניטור, IAM, רשת).
- קושי ב-**אופטימיזציית צפיפות שרתים** — הרבה שירותים קטנים בניצול נמוך.
- **מורכבות של ריבוי גרסאות** של ריבוי שירותים במקביל.
- **התפוצצות קוד צד-לקוח** שצריך לדבר עם המון endpoints נפרדים.

### 8.4 איך Serverless מקל על האתגרים האלה

- API Gateway ו-Lambda **מתרחבים לבד** ומחייבים לפי שימוש — בעיית צפיפות השרתים נעלמת.
- קל **לשכפל API** ולשחזר סביבות שלמות.
- **SDK צד-לקוח נוצר אוטומטית** מהגדרת ה-API (דרך אינטגרציית Swagger/OpenAPI ב-API Gateway).
- Stages ו-aliases מנהלים ריבוי גרסאות בצורה מסודרת.

> [!warning] מלכודת התכנון
> אם הדרישה בשאלה היא "**minimum complexity**" או "small team" — microservices הן **לא** התשובה.
> ארכיטקטורה מבוזרת היא עלות, לא רק יתרון.

---

## 9. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב | איך נמדד | הערה |
|---|---|---|
| Lambda | invocations + GB-seconds (ב-1ms) | free tier: מיליון requests + 400,000 GB-seconds בחודש |
| API Gateway | לפי מיליון requests (+ תוספות לפי סוג API) | HTTP API זול משמעותית מ-REST API |
| API Gateway cache | **לפי שעה** לפי גודל ה-cache | הוצאה קבועה, לא לפי שימוש |
| DynamoDB | RCU/WCU (Provisioned) או per-request (On-Demand) + אחסון | Global Tables מכפילות כתיבות לפי מספר ה-Regions |
| DAX | **לפי שעה per node** | לא scale-to-zero — הוצאה קבועה |
| CloudFront | requests + data transfer out | S3→CloudFront חינם |
| S3 | GB-חודש + requests | Transfer Acceleration = תוספת מחיר |
| SES / SNS / SQS | לפי הודעות | זול מאוד; retries ו-DLQ מגדילים ספירה |
| Step Functions | **Standard**: לפי state transition; **Express**: לפי invocations+duration | Express זול משמעותית ל-workflows קצרים ובנפח גבוה |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Lambda | זול מאוד ב-burst, יקר ב-load רציף | תעבורה ספורדית או משתנה |
| Fargate | ביניים | container שרץ דקות-שעות, בלי לנהל hosts |
| EC2 + Savings Plan | הזול ליחידת compute בניצול גבוה | עומס יציב 24/7 |
| **HTTP API** ב-API Gateway | זול משמעותית מ-REST API | כשלא צריך API keys, request validation, WAF ישיר |
| ALB במקום API Gateway | זול יותר בתעבורה רציפה גבוהה | יש כבר EC2/ECS ואין צורך בפיצ'רים של API GW |

### 🚩 עלויות נסתרות

- **DAX ו-API Gateway cache** אינם scale-to-zero — הם ההוצאה הקבועה בארכיטקטורה שנראית "לפי שימוש".
- **Global Tables** מכפילות את עלות הכתיבה במספר ה-Regions (rWCU).
- **NAT Gateway** ל-Lambda בתוך VPC שיוצאת לאינטרנט — עלות נסתרת קלאסית.
- **CloudWatch Logs** של Lambda — פונקציה שרצה מיליוני פעמים ורושמת DEBUG יוצרת חשבון לוגים גדול מהחשבון compute.
- **retries ו-DLQ** מגדילים את מספר ה-invocations בפועל.
- **Step Functions Standard** על workflow עם אלפי state transitions יכול להיות יקר — לשקול Express.

### 💡 טיפים לחיסכון

- כוונן **זיכרון Lambda** — יותר RAM = יותר CPU = ריצה קצרה יותר, לפעמים זול יותר בסך הכול.
- העבר REST API ל-**HTTP API** אם אין תלות בפיצ'רים מתקדמים.
- החזק את גישת הקליינט ל-S3 **ישירה** (Cognito credentials) במקום דרך Lambda.
- הפעל **CloudFront** לפני כל תוכן שחוזר על עצמו.
- הורד את **retention** של CloudWatch Logs ואת רמת ה-log.
- שקול **Express Workflows** ב-Step Functions לתהליכים קצרים ובנפח גבוה.

---

## 10. ⚖️ השוואות מכריעות

| קריטריון | API Gateway + Lambda | ALB + ECS/EC2 |
|---|---|---|
| תעבורה ספורדית | **מנצח** — scale to zero | משלמים גם ב-0 |
| תעבורה רציפה גבוהה | יקר יותר ליחידה | **מנצח** עם Savings Plan |
| Cold start | קיים | לא קיים |
| Timeout | 15 דקות מקסימום ב-Lambda | ללא הגבלה מעשית |
| ניהול | אפס | patching, scaling, capacity |
| פיצ'רים (throttling, API keys, usage plans) | מובנים | צריך לבנות |

| קריטריון | SQS | SNS | EventBridge |
|---|---|---|---|
| מודל | תור, צרכן אחד לוקח הודעה | broadcast לכל המנויים | ניתוב לפי **rules** על תוכן האירוע |
| Buffer/backpressure | **כן** — היתרון המרכזי | לא | לא |
| Fan-out | דרך SNS→SQS | **כן** | כן, לכמה targets |
| סינון | לא | Message Filtering לפי attributes | **Content-based filtering** מלא |
| Schema registry / archive+replay | לא | לא | **כן** |

> [!info] שורה תחתונה
> צריך **חוצץ** מול spike → SQS. צריך **לשדר לכולם** → SNS. צריך **לנתב לפי תוכן ובין שירותים** → EventBridge.

---

## 11. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | ארכיטקטורה מבוזרת בלי observability היא קופסה שחורה | X-Ray לכל invocation, structured logs, DLQ לכל צרכן אסינכרוני |
| Security | Cognito + IAM Roles מחליפים credentials בקוד | Identity Pool ל-temp credentials, OAC ל-S3, per-function IAM role |
| Reliability | Lambda נקראת at-least-once — כפילויות הן ודאות | handlers idempotent, retry עם backoff, SQS כחוצץ, bucket יעד נפרד |
| Performance Efficiency | latency הוא סכום השכבות, לא רק ה-DB | DAX + API Gateway cache + CloudFront edge; Provisioned Concurrency ל-cold start |
| Cost Optimization | serverless הוא pay-per-use, אבל לא כל רכיב | לזהות את הרכיבים שמחויבים לפי שעה (DAX, API GW cache) ולהצדיק אותם |
| Sustainability | scale-to-zero מבטל idle capacity | Lambda/Fargate במקום EC2 בטלה; להימנע מ-polling מיותר, להעדיף event-driven |

---

## 12. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "no servers to manage" + REST API | **API Gateway + Lambda** |
| "managed authentication for mobile/web users" | **Cognito** (User Pool לזהות, Identity Pool ל-credentials) |
| "users upload directly to S3 without a backend" | **Cognito Identity Pool** + pre-signed URLs |
| "microseconds latency" + DynamoDB | **DAX** |
| "milliseconds latency" + cache כללי | **ElastiCache** |
| "S3 bucket must not be publicly accessible" + CloudFront | **OAC** + bucket policy |
| "multi-region, writes in every region" | **DynamoDB Global Tables** |
| "multi-region SQL, reads everywhere, one writer" | **Aurora Global Database** |
| "process each uploaded file automatically" | **S3 Event Notification → Lambda** |
| "react to every change in the table" | **DynamoDB Streams → Lambda** |
| "send emails in a serverless way" | **SES** |
| "existing app, no code changes, reduce cost & load" | **CloudFront** לפני ה-origin |
| "buffer to absorb traffic spikes" | **SQS** |
| "notify multiple systems of the same event" | **SNS** (fan-out) |
| "orchestrate a multi-step workflow with retries" | **Step Functions** |
| "run containers without managing servers" | **Fargate** |
| "unpredictable/intermittent SQL workload" | **Aurora Serverless v2** |

### טעויות נפוצות

> [!warning] מלכודת 1 — "Serverless = ללא מגבלות"
> **הניסוח:** "העבר את עיבוד הווידאו של 45 דקות ל-Lambda."
> **הטעות:** Lambda מוגבלת ל-**15 דקות**.
> **הנכון:** Fargate, ECS, Batch, או Step Functions שמפרק לשלבים קצרים.

> [!warning] מלכודת 2 — לולאת ה-Lambda
> **הניסוח:** "Lambda מייצרת thumbnail ושומרת אותו באותו bucket."
> **הטעות:** ה-thumbnail מפעיל את ה-event שוב → לולאה אינסופית וחשבון מפוצץ.
> **הנכון:** bucket יעד נפרד, או prefix/suffix filter ב-event notification.

> [!warning] מלכודת 3 — retry בלי idempotency
> **הניסוח:** "הזמנה נשלחת ל-SQS ו-Lambda מחייבת את הכרטיס."
> **הטעות:** SQS Standard הוא at-least-once — הלקוח יחויב פעמיים.
> **הנכון:** מפתח idempotency (order ID) שנבדק לפני החיוב, או FIFO queue עם deduplication.

> [!warning] מלכודת 4 — שרשרת סינכרונית ארוכה שנקראת "decoupled"
> **הניסוח:** "Lambda A קוראת ל-Lambda B שקוראת ל-Lambda C, כולן סינכרוניות."
> **הטעות:** זו לא הפרדה — זו שרשרת שבירה שמצטברת בה latency ובכל חוליה כשל מפיל הכול.
> **הנכון:** SQS/EventBridge בין השלבים, או Step Functions לניהול ה-state.

> [!warning] מלכודת 5 — Cognito מיותר
> **הניסוח:** "בלוג ציבורי שקורא פוסטים — האם צריך Cognito?"
> **הטעות:** להוסיף אימות לתוכן ציבורי.
> **הנכון:** API ציבורי לא דורש Cognito. Cognito נדרש כשיש **משתמשים אישיים** ומשאבים פרטיים.

> [!warning] מלכודת 6 — Lambda ב-VPC "כי זה מאובטח יותר"
> **הניסוח:** "שים את ה-Lambda ב-VPC כדי לאבטח את הגישה ל-DynamoDB."
> **הטעות:** DynamoDB ו-S3 נגישים דרך ה-AWS API; Lambda ב-VPC דורשת NAT Gateway כדי לצאת — עלות ומורכבות.
> **הנכון:** Lambda ב-VPC נדרשת רק לגישה למשאבים **פרטיים ב-VPC** (RDS, ElastiCache, EC2 פנימי).

---

## 13. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** סטארטאפ מעלה פלטפורמת תמונות. משתמשים מכל העולם, תעבורה בלתי צפויה לחלוטין (יכול להיות ויראלי), תקציב קטן, צוות של 3 מפתחים. צריך: העלאה, thumbnail, פיד לצפייה, מייל ברוכים הבאים.

```text
Client ──► CloudFront ──/*────► S3 (static SPA, פרטי, OAC)
   │              └──/api/*──► API Gateway ──► Lambda ──► DAX ──► DynamoDB
   │                                                                  │
   │                                                          Streams │
   │  (Cognito temp creds)                                            ▼
   └──── PUT ישיר ──► S3 uploads ──event──► Lambda ──► S3 thumbs   Lambda ──► SES
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Serverless מקצה לקצה | תעבורה בלתי צפויה + צוות קטן = אפס ops, תשלום לפי שימוש |
| Cognito + העלאה ישירה ל-S3 | לא משלמים Lambda duration על העברת מגה-בייטים; אין מגבלת payload |
| CloudFront + OAC | גלובליות, latency נמוך, egress זול, bucket נשאר פרטי |
| S3 Event → Lambda ל-thumbnails | scale-to-zero; ויראליות מטופלת אוטומטית |
| DynamoDB On-Demand | אין היסטוריה להקצות לפיה capacity; ספייק ויראלי לא ישבור כלום |
| DAX רק אם ה-read קופץ | הוא ההוצאה הקבועה היחידה — מוסיפים כשמצדיק |
| Streams → Lambda → SES | המייל אסינכרוני; כשל בו לא מפיל הרשמה |

**למה לא ECS on EC2?**
היה זול יותר ליחידת compute בעומס גבוה קבוע — אבל התעבורה **בלתי צפויה**,
והצוות של 3 אנשים ישלם את המחיר בזמן ops, patching ו-capacity planning. Serverless מנצח.

**וריאציה — מה משתנה אם התעבורה מתייצבת על עומס גבוה קבוע?**
כדאי לבחון מעבר של שכבת ה-API ל-**Fargate/ECS מאחורי ALB** עם Compute Savings Plan,
ולעבור ב-DynamoDB ל-**Provisioned + Auto Scaling**. שכבות ה-CloudFront, S3 וה-thumbnails נשארות זהות.

---

## 14. 🚫 מה לא צריך לדעת למבחן

- אין צורך לכתוב קוד Lambda או להכיר SDK ספציפי.
- אין צורך להכיר את התחביר של SAM / CDK / Serverless Framework.
- אין צורך לדעת את פורמט ה-JSON המדויק של S3 event או DynamoDB Stream record.
- אין צורך לזכור את כל ה-Cognito triggers — רק את ההבחנה User Pool מול Identity Pool.
- אין צורך בפרטי ה-ASL (Amazon States Language) של Step Functions — רק ההבחנה Standard מול Express.

---

## 15. ⚡ Cheat Sheet — סיכום מהיר

- **Serverless** = לא מנהלים/מקצים/רואים שרתים. כולל Lambda, Fargate, DynamoDB, S3, API GW, Cognito, SNS/SQS, Firehose, Aurora Serverless, Step Functions.
- **הדפוס הבסיסי:** Client → API Gateway → Lambda → DynamoDB.
- **Cognito User Pool** = מי אתה. **Identity Pool** = credentials זמניים ל-AWS ישירות מהקליינט.
- **DAX** = cache ל-DynamoDB במיקרו-שניות, drop-in, מחויב per node לפי שעה.
- **API Gateway cache** חוסך גם Lambda וגם DynamoDB; מחויב לפי שעה לפי גודל.
- **CloudFront + S3 + OAC** = אתר סטטי גלובלי עם bucket **פרטי**. S3→CloudFront חינם.
- **DynamoDB Global Tables** = active-active. **Aurora Global Database** = active-passive.
- **S3 Event Notifications** יכולים לטרגט **SQS, SNS או Lambda**.
- **DynamoDB Streams → Lambda** = התגובה לכל שינוי בטבלה.
- **SES** = שליחת מיילים serverless, עם IAM Role ל-Lambda.
- **CloudFront לפני אפליקציה קיימת** = חיסכון עצום בלי שינוי קוד, כשהתוכן סטטי.
- Microservices: סינכרוני = API GW/ALB; אסינכרוני = SQS/SNS/Kinesis/S3 triggers.
- Lambda: **15 דקות** מקסימום, at-least-once — תמיד **idempotent**.
- HTTP API זול משמעותית מ-REST API ב-API Gateway.

---

## 16. ✅ בדיקת הבנה

1. אפליקציית מובייל צריכה לאפשר למשתמשים להעלות קבצים ל-S3, כל אחד לתיקייה שלו בלבד, בלי backend שמעביר את הבייטים. איך?
2. מה ההבדל התפקודי בין caching ב-API Gateway לבין DAX, ומתי כל אחד עדיף?
3. Lambda מייצרת thumbnails ופתאום החשבון מזנק פי 1000. מה הסיבה הסבירה ביותר?
4. אפליקציה עולמית צריכה **כתיבות** מהירות בכל יבשת. DynamoDB Global Tables או Aurora Global Database?
5. אתר קיים על EC2 מפיץ קבצי עדכון; אסור לשנות את הקוד והחשבון גבוה מדי. מה עושים?
6. למה שולחים את ה-welcome email דרך DynamoDB Streams ולא ישירות מה-Lambda שיצרה את המשתמש?

<details>
<summary>תשובות</summary>

1. **Cognito Identity Pool** שמנפיק temporary IAM credentials לכל משתמש מאומת, עם IAM policy שמגבילה את הגישה ל-prefix האישי שלו (למשל באמצעות משתנה `${cognito-identity.amazonaws.com:sub}`). הקליינט מדבר עם S3 ישירות דרך ה-SDK. חלופה נוספת: pre-signed URLs שנוצרים ב-Lambda — גם היא מונעת העברת בייטים דרך ה-backend.

2. **API Gateway cache** שומר את ה-**תשובה השלמה** של ה-endpoint, ולכן חוסך גם את ה-Lambda invocation וגם את פניית ה-DynamoDB — עדיף כשאותן בקשות בדיוק חוזרות. **DAX** שומר תוצאות query ומחזיר במיקרו-שניות — עדיף כשה-requests שונים ברמת ה-HTTP אך פונים לאותו דאטה, או כשצריך cache שקוף לכל צרכן של הטבלה.

3. **לולאה אינסופית:** ה-Lambda כותבת את ה-thumbnail לאותו bucket שמפעיל אותה, וכל thumbnail מפעיל invocation חדש. הפתרון: bucket יעד נפרד, או prefix/suffix filter ב-event notification.

4. **DynamoDB Global Tables** — הן היחידות שמאפשרות כתיבה **active-active** בכל Region. Aurora Global Database מאפשרת קריאה בכל מקום אך כתיבה רק ב-primary Region, כלומר כתיבות מיבשת מרוחקת יסבלו מ-latency.

5. **CloudFront לפני ה-origin הקיים.** קבצי העדכון סטטיים לחלוטין, ולכן ה-cache hit ratio יהיה גבוה מאוד. ה-ASG כמעט לא יתרחב, ה-egress יהיה זול יותר, וה-scalability תעלה — הכול בלי לשנות שורת קוד.

6. כדי **להפריד את הגורלות**: ה-API מחזיר תשובה מיידית למשתמש, ושליחת המייל מתרחשת אסינכרונית. אם SES נכשל או איטי — ההרשמה עצמה לא נפגעת. בנוסף, כל צרכן נוסף שירצה להגיב ל"נוצר משתמש" יכול להתחבר לאותו stream בלי לגעת בקוד ההרשמה.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[25 - Lambda]] · [[27 - API Gateway]] · [[23 - DynamoDB]] · [[15 - CloudFront and Global Delivery]] · [[28 - SQS and SNS]] · [[29 - Event-Driven Architecture]] · [[30 - Application Decoupling]] · [[26 - Containers]] · [[32 - Security Services]] · [[39 - Architecture Decision Making]] · [[40 - Integrated SAA Scenarios]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 7938–7987, 8717–8729, 8902–9267
