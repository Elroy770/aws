---
lesson: 27
title: API Gateway
domain: Design Secure Architectures
services: [Amazon API Gateway, AWS Lambda, Amazon Cognito, AWS Step Functions, ACM, Route 53]
tags: [saa-c03, serverless, api, security]
---

# 27 — API Gateway

> [!abstract] בשורה אחת
> API Gateway היא הדלת המנוהלת ל-API שלך: היא מטפלת ב-HTTPS, אימות, throttling, caching וגרסאות — ומאפשרת לחשוף Lambda או כל שירות AWS כ-REST API בלי שרת אחד.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | מה חסר כשמחברים לקוח ישירות ל-Lambda |
| 2 | איך זה עובד | שלושת סוגי ה-integration, stages, WebSocket |
| 3 | פירוק מפורט | Endpoint Types, שלוש שיטות האימות, Cognito, caching, ACM |
| 4 | עלות | REST מול HTTP API, cache, data transfer |
| 5 | השוואות | API Gateway מול ALB מול CloudFront; User Pools מול Identity Pools |
| 6 | Well-Architected | מה כל pillar אומר על שכבת ה-API |
| 7 | מלכודות | API key ≠ אימות, איפה יושב ה-certificate |
| 8 | Scenario | MyTodoList — אפליקציית מובייל serverless מלאה |

**מונחי מפתח בשיעור:** `Integration Type` · `Stage` · `Edge-Optimized` · `Regional` · `Private` · `Lambda Authorizer` · `User Pool` · `Identity Pool` · `Usage Plan`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- יש לך Lambda שעושה CRUD על DynamoDB. איך לקוח מובייל קורא לה?
- לקרוא ל-Lambda ישירות דורש AWS credentials בצד הלקוח — בעיית אבטחה.
- ואפילו אם פתרת את זה, חסר לך הכול:
  - HTTPS עם domain משלך.
  - אימות משתמשים (מי אתה בכלל?).
  - **Rate limiting** — משתמש אחד לא יפיל לך את הכול.
  - גרסאות API (v1, v2) וסביבות (dev, test, prod).
  - Caching של תשובות שחוזרות על עצמן.
  - ולידציה של בקשות ותיעוד.

### מה השירות פותר

- **Lambda + API Gateway = אין שום תשתית לנהל.** זו הקומבינציה הקלאסית.
- כל מה שנספר למעלה מגיע בקופסה:
  - טיפול ב-**versioning** של ה-API.
  - הפרדת **environments** דרך Stages.
  - אבטחה — Authentication ו-Authorization.
  - **API keys** ו-request **throttling**.
  - ייבוא **Swagger / OpenAPI** להגדרה מהירה.
  - **Transform ו-validate** של requests ו-responses.
  - ייצור **SDK** ומפרטי API ללקוחות.
  - **Cache** לתשובות ה-API.
  - תמיכה בפרוטוקול **WebSocket** (ולא רק REST).

> [!tip] האנלוגיה
> Lambda היא המטבח. API Gateway הוא הדלפק והמלצר — הוא בודק מי נכנס, לא נותן ליותר מדי אנשים להיכנס בבת אחת, זוכר הזמנות חוזרות, ומגיש בצורה אחידה.

---

## 2. ⚙️ איך זה עובד

### 2.1 התמונה הבסיסית

```text
Client ──HTTPS──> API Gateway ──proxy──> Lambda ──CRUD──> DynamoDB
                       │
                       ├── אימות
                       ├── throttling
                       ├── caching
                       └── לוגים ומטריקות
```

### 2.2 שלושת סוגי ה-Integration

| Integration | מה זה עושה | למה משתמשים בזה |
|---|---|---|
| **Lambda Function** | מפעילה פונקציית Lambda | הדרך הכי פשוטה לחשוף REST API מעל Lambda |
| **HTTP** | מעביר ל-endpoint HTTP קיים בצד השרת | לחשוף ALB או **HTTP API פנימי ב-on-premises**, ולקבל מעליו rate limiting, caching, אימות ו-API keys |
| **AWS Service** | קורא ישירות ל-API של שירות AWS כלשהו | להתחיל **Step Functions workflow**, לשלוח הודעה ל-**SQS**, לכתוב ל-**Kinesis** — בלי Lambda באמצע |

> [!info] למה בכלל AWS Service Integration
> כדי לחשוף שירות AWS **בציבור** עם שכבת אימות ו-rate control מעליו, מבלי לכתוב שורת קוד ומבלי לשלם על Lambda.

### 2.3 דוגמת Kinesis (השקף)

```text
Client ──requests──> API Gateway ──records──> Kinesis Data Streams
                                                     │
                                                     ▼
                                          Kinesis Data Firehose
                                                     │
                                                     ▼
                                          Amazon S3 (קבצי .json)
```

- אין Lambda בשרשרת הזו כלל. API Gateway כותב ישירות ל-Kinesis.

### 2.4 Stages ו-Deployments

- שינוי ב-API לא נכנס לתוקף עד שעושים **Deploy** ל-**Stage**.
- Stage = סביבה: `dev`, `test`, `prod`, או גרסה: `v1`, `v2`.
- לכל Stage יש URL משלו, הגדרות throttling משלו, cache משלו ולוגים משלו.
- אפשר **Canary deployment** — לשלוח אחוז קטן מהתעבורה לגרסה חדשה.

---

## 3. 🔍 פירוק מפורט

### 3.1 שלושת סוגי ה-Endpoint

זו טבלה שנשאלת כמעט תמיד:

| Endpoint Type | איפה התעבורה עוברת | מתי בוחרים | הערה קריטית |
|---|---|---|---|
| **Edge-Optimized** (ברירת מחדל) | דרך **CloudFront Edge Locations** | לקוחות **גלובליים** — משפר latency | ה-API עצמו עדיין חי ב-**region אחד בלבד**! זה רק נתיב כניסה |
| **Regional** | ישירות ל-region | לקוחות **באותו region** | אפשר לשלב ידנית עם CloudFront ולקבל שליטה מלאה באסטרטגיית ה-caching וב-distribution |
| **Private** | רק מתוך ה-**VPC** דרך **Interface VPC Endpoint (ENI)** | API פנימי שאסור שייחשף לאינטרנט | הגישה נשלטת דרך **Resource Policy** |

> [!warning] טעות נפוצה
> Edge-Optimized **לא** מפיץ את ה-API לכמה regions. ה-API יושב באזור אחד; רק הכניסה עוברת דרך רשת ה-Edge של CloudFront.

### 3.2 שלוש שיטות האימות

| שיטה | למי מיועדת | איך זה עובד | מילת מפתח במבחן |
|---|---|---|---|
| **IAM Roles / SigV4** | אפליקציות **פנימיות**, שירותי AWS, EC2/Lambda בתוך החשבון | הבקשה חתומה ב-credentials של IAM; API Gateway מאמת מול IAM | "internal application", "another AWS service calls the API" |
| **Amazon Cognito** (User Pools) | משתמשי קצה **חיצוניים** — מובייל, web | המשתמש מתחבר ל-User Pool, מקבל token, ומצרף אותו לבקשה | "**mobile users**", "hundreds of users", "sign-up / sign-in", "SAML" |
| **Custom Authorizer** (Lambda Authorizer) | **לוגיקה משלך** | Lambda שלך מקבלת את ה-token/header ומחזירה IAM policy של הרשאה | "third-party identity provider", "custom auth logic", "OAuth provider קיים" |

- **API Keys + Usage Plans** הם משהו אחר לגמרי — הם לא אימות אלא **מדידה ומכסה** לכל לקוח.

### 3.3 Amazon Cognito — שני שירותים בשם אחד

זה הבלבול הכי גדול בנושא. **User Pools ≠ Identity Pools.**

| | **Cognito User Pools (CUP)** | **Cognito Identity Pools (Federated Identities)** |
|---|---|---|
| מה נותן | **התחברות** (sign-in) לאפליקציה | **AWS credentials זמניים** |
| מה מקבלים בסוף | Token של המשתמש | Access Key + Secret + Session Token של IAM |
| למה זה משמש | לאמת מי המשתמש מול ה-API | לתת למשתמש גישה **ישירה** ל-S3 / DynamoDB |
| משתלב עם | **API Gateway** ו-**Application Load Balancer** | S3, DynamoDB, כל שירות AWS, וגם API Gateway |
| מקור המשתמשים | משתמשים שנרשמו + Federated (Facebook, Google, SAML) | User Pools, ספקי זהות צד שלישי |
| ההרשאות נקבעות ב... | לא רלוונטי — זה רק אימות | **IAM policies שמוגדרות ב-Cognito**, וניתן להתאים אותן לפי `user_id` |

**מה User Pools נותן למשתמש:**

- מסד נתונים **serverless** של משתמשים לאפליקציית web/mobile.
- התחברות פשוטה: username (או email) + סיסמה.
- איפוס סיסמה.
- אימות email ומספר טלפון.
- **MFA** (Multi-Factor Authentication).
- **Federated Identities** — התחברות דרך Facebook, Google, SAML.

**מה Identity Pools נותן:**

- ממיר token לזהות עם **credentials זמניים** של AWS.
- Roles נפרדים ל-**authenticated users** ול-**guest users**.
- שליטה עדינה: אפשר לבנות policy שמאפשרת גישה **רק לתיקייה של המשתמש** ב-S3, או **Row Level Security** ב-DynamoDB לפי ה-`user_id`.

```text
Mobile App ──login──> Cognito User Pool ──token──┐
                                                 ▼
                                     Cognito Identity Pool
                                                 │ exchange
                                                 ▼
                                   AWS credentials זמניים
                                                 │
                              ┌──────────────────┴──────────────────┐
                              ▼                                     ▼
                   S3 (רק prefix של המשתמש)              DynamoDB (רק השורות שלו)
```

> [!info] המשפט שמסכם
> **Cognito מול IAM:** אם בשאלה כתוב "hundreds of users", "mobile users" או "authenticate with SAML" — זה **Cognito**, לא IAM Users.

### 3.4 Custom Domain ו-ACM — איפה יושב ה-Certificate

זו שאלה קלאסית, ותשובתה תלויה ב-endpoint type:

| Endpoint Type | איפה חייב להיות ה-TLS Certificate | למה |
|---|---|---|
| **Edge-Optimized** | **us-east-1** | התעבורה עוברת דרך CloudFront, וה-certificate חייב לשבת באזור של CloudFront |
| **Regional** | **באותו region** של ה-API Stage | ה-certificate מיובא ישירות ל-API Gateway |

- בשני המקרים צריך רשומת **CNAME** או — עדיף — **A-Alias** ב-Route 53 שמצביעה על ה-API.
- A-Alias עדיף כי הוא חינם ב-Route 53 ועובד גם ב-apex domain.

### 3.5 Caching ב-API Gateway

- אפשר להפעיל **cache של תשובות** ברמת ה-**Stage**.
- מגדירים TTL וגודל cache.
- התוצאה: פחות קריאות ל-Lambda / ל-backend → **פחות latency ופחות עלות compute**.
- שימושי במיוחד כשקוראים הרבה יותר משכותבים.
- אפשר לאפשר ללקוח לבטל cache לבקשה ספציפית (invalidation) — עם הרשאה מתאימה.

### 3.6 Throttling ו-Usage Plans

| מנגנון | מה עושה |
|---|---|
| Account-level throttling | מגבלת ברירת מחדל של בקשות לשנייה לכל החשבון באזור |
| Stage / Method throttling | תקרת rate ו-burst ל-stage או ל-method ספציפי |
| **Usage Plan + API Key** | מכסה (quota) ו-rate לכל **לקוח** — כמה קריאות ליום/חודש |

- ה-throttling הוא מה שמגן על ה-backend: Lambda לא תיחנק ב-throttle משלה, ו-RDS מאחור לא יקרוס.

### 3.7 AWS Step Functions — הקרוב המשפחתי

- בניית **workflow ויזואלי serverless** שמתזמר פונקציות Lambda.
- תכונות: רצף, מקבילות (parallel), תנאים, timeouts, ו-error handling.
- משתלב עם EC2, ECS, שרתים ב-on-premises, API Gateway, תורי SQS ועוד.
- תומך ב-**human approval** — עצירה עד שאדם מאשר.
- Use cases: order fulfillment, עיבוד נתונים, אפליקציות web, כל workflow רב-שלבי.
- הקשר ל-API Gateway: אפשר להתחיל execution של Step Functions **ישירות** דרך AWS Service Integration.

> [!info] מתי Step Functions ולא Lambda אחת גדולה
> כשיש כמה שלבים עם retry ו-error handling שונים לכל שלב, כשצריך אישור אנושי, או כשהתהליך הכולל ארוך מ-15 דקות.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| API calls | לפי מיליון קריאות | הרכיב העיקרי |
| Data transfer out | לפי GB | כרגיל |
| **Cache** | לפי **שעה** לפי גודל ה-cache שהוקצה | משלמים גם כשאין תעבורה |
| Backend | בנפרד — Lambda, EC2, Kinesis... | API Gateway היא רק השכבה הקדמית |
| CloudWatch Logs / access logs | ingestion + אחסון | לוגים מפורטים מצטברים מהר |
| WAF (אופציונלי) | לפי rules + בקשות | ראה [[32 - Security Services]] |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **HTTP API** | הזול משמעותית מבין השניים | REST פשוט, JWT authorizer, proxy ל-Lambda/HTTP |
| **REST API** | יקר יותר | צריך API keys, usage plans, caching מובנה, request validation, WebSocket |
| **ALB** | לרוב זול יותר בתעבורה רציפה גבוהה | routing ל-EC2/ECS בלי צורך בניהול API |
| API Gateway **עם cache** | מוסיף עלות שעתית קבועה | רק כשהוא באמת חוסך קריאות backend |

- הכלל: אם לא צריך את התכונות המתקדמות של REST API — **HTTP API זול יותר ומהיר יותר**.

### 🚩 עלויות נסתרות

- **Cache מוקצה** — משלמים לפי שעה גם ב-3 בלילה כשאין תעבורה. Stage dev עם cache פתוח = בזבוז שקט.
- **Access logs מפורטים** — כל בקשה נכנסת ל-CloudWatch. במיליוני בקשות זה מצטבר.
- **קריאה כפולה** — משלמים על ה-API call **וגם** על ה-Lambda invocation **וגם** על ה-duration.
- **Data transfer** ביציאה, במיוחד ב-payload גדול.
- **Custom domain + ACM** — ה-certificate חינם, אבל אם מוסיפים CloudFront משלכם הוא מחויב בנפרד.

### 💡 טיפים לחיסכון

- לבחור **HTTP API** כברירת מחדל, ולעבור ל-REST API רק כשצריך תכונה ספציפית.
- להפעיל **cache** ב-endpoints קריאים בלבד, ולכבות אותו ב-stages שאינם production.
- **throttling ו-usage plans** — מונעים גם עלות וגם עומס.
- לצמצם payloads ולהחזיר רק שדות נדרשים.
- להגדיר **retention** ללוגים במקום ברירת המחדל.
- אם התעבורה רציפה ו-גבוהה מאוד ואין צורך בניהול API — לשקול **ALB**.

---

## 5. ⚖️ השוואות מכריעות

### API Gateway מול ALB מול CloudFront

| קריטריון | API Gateway | ALB | CloudFront |
|---|---|---|---|
| מה זה בעצם | שכבת **ניהול API** | **Load Balancer** בשכבה 7 | **CDN** גלובלי |
| יעדים נתמכים | Lambda, HTTP endpoint, כל שירות AWS | EC2, ECS, IP, **Lambda** | כל origin: S3, ALB, API GW |
| אימות משתמשים | IAM / Cognito / Lambda Authorizer | Cognito או OIDC ב-listener rule | Signed URLs/Cookies, Lambda@Edge |
| Rate limiting / quotas | **כן** — throttling + Usage Plans + API keys | לא (צריך WAF) | לא (צריך WAF) |
| Caching תשובות | כן, ברמת Stage | לא | **כן** — זה עיקר תפקידו |
| Request/response transform | **כן** | לא | מוגבל (functions) |
| ייצור SDK / OpenAPI | **כן** | לא | לא |
| WebSocket | **כן** | כן | לא |
| תמחור | לפי קריאה | לפי שעה + LCU | לפי בקשות + data transfer |
| מילת מפתח | "API management", "throttle", "API keys", "serverless API" | "route HTTP to EC2/ECS", "cheaper at constant load" | "global content delivery", "cache static content" |

> [!info] שורה תחתונה
> ALB מנתב HTTP — אבל הוא **לא מוצר ניהול API**. ברגע שהשאלה מזכירה API keys, usage plans, quotas, request validation או SDK generation — התשובה היא API Gateway.

### שלוש שיטות האימות — מתי במה

| מצב בשאלה | הבחירה |
|---|---|
| שירות AWS פנימי קורא ל-API | **IAM** |
| אפליקציית מובייל עם הרשמה והתחברות | **Cognito User Pools** |
| ספק זהות קיים / לוגיקה עסקית מותאמת | **Lambda Authorizer** |
| רוצים למדוד ולהגביל צריכה לכל לקוח | **API Key + Usage Plan** (בנוסף לאימות, לא במקומו) |

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר בנושא הזה | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | ה-API הוא חוזה שמשתנה בגרסאות | Stages נפרדים ל-dev/test/prod, Canary deployment לגרסה חדשה, access logs + מטריקות CloudWatch, ייבוא OpenAPI כמקור אמת |
| Security | הכניסה היחידה למערכת — כאן מסננים | Cognito/IAM/Lambda Authorizer לפי סוג הלקוח, WAF מול OWASP, **Private endpoint** ל-API פנימי, ACM ב-region הנכון, resource policy |
| Reliability | ה-API חייב להגן על ה-backend מפני עצמו | throttling ברמת stage ו-method, usage plans לכל לקוח, timeouts ו-retry מוגדרים, backend ב-multi-AZ |
| Performance Efficiency | latency נמדדת מהקצה, לא מה-region | **Edge-Optimized** ללקוחות גלובליים, **cache** ב-stage, HTTP API כשמספיק, payload רזה |
| Cost Optimization | לא לשלם על תכונות שלא בשימוש | HTTP API במקום REST API כשאין צורך, cache רק איפה שהוא חוסך קריאות, retention ללוגים, throttling כבלם עלות |
| Sustainability | תשובה מה-cache = compute שלא רץ | caching ב-API Gateway ו-CloudFront, scale-to-zero של Lambda מאחור, פחות קריאות מיותרות מהלקוח |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "serverless REST API" + Lambda | **API Gateway** |
| "protect the backend from too many requests" | **Throttling** ב-API Gateway |
| "limit each customer to N calls per month" | **API Key + Usage Plan** |
| "mobile users sign up and sign in" | **Cognito User Pools** |
| "users need direct access to their own S3 folder" | **Cognito Identity Pools** |
| "global clients, reduce latency" | **Edge-Optimized** endpoint |
| "API accessible only from within the VPC" | **Private** endpoint + Interface VPC Endpoint + Resource Policy |
| "custom domain, Edge-Optimized" | ACM certificate ב-**us-east-1** |
| "custom domain, Regional" | ACM certificate **באותו region** |
| "reduce load on Lambda for repeated reads" | **API Gateway Caching** (ואולי DAX ב-DynamoDB) |
| "orchestrate multiple Lambda functions with human approval" | **Step Functions** |
| "expose SQS/Kinesis/Step Functions publicly with auth" | **AWS Service Integration** ב-API Gateway |
| "third-party identity provider / custom auth logic" | **Lambda Authorizer** |
| "raw TCP / UDP" | לא API Gateway — **NLB** |

### טעויות נפוצות

> [!warning] מלכודת 1 — API Key זה לא אימות
> **הניסוח:** "רוצים לאמת את הלקוחות שקוראים ל-API."
> **הטעות:** לבחור API Keys.
> **הנכון:** API Key + Usage Plan הם **מדידה ומכסה**, לא זהות. אימות אמיתי = IAM / Cognito / Lambda Authorizer.

> [!warning] מלכודת 2 — Edge-Optimized הוא לא multi-region
> **הניסוח:** "רוצים שה-API ישרוד נפילה של region שלם."
> **הטעות:** לבחור Edge-Optimized כי הוא "גלובלי".
> **הנכון:** ה-API עדיין חי ב-**region אחד**. ל-DR צריך API בשני regions + Route 53 failover.

> [!warning] מלכודת 3 — ה-certificate ב-region הלא נכון
> **הניסוח:** "הגדרנו custom domain ב-Edge-Optimized וה-certificate לא נראה ברשימה."
> **הטעות:** לייבא אותו ל-region של ה-API.
> **הנכון:** Edge-Optimized עובר דרך CloudFront → ה-certificate **חייב** להיות ב-**us-east-1**. ב-Regional — באותו region של ה-Stage.

> [!warning] מלכודת 4 — User Pools מול Identity Pools
> **הניסוח:** "המשתמש צריך להעלות קבצים ישירות ל-S3 מהאפליקציה."
> **הטעות:** לבחור User Pools כי "זה Cognito".
> **הנכון:** User Pools נותן **token**. כדי לקבל **AWS credentials** ולגשת ישירות ל-S3 צריך **Identity Pool**.

> [!warning] מלכודת 5 — ALB במקום API Gateway
> **הניסוח:** "צריך לחשוף API עם quotas ללקוחות שונים ולייצר SDK."
> **הטעות:** ALB, כי גם הוא מנתב HTTP ואפילו תומך ב-Lambda targets.
> **הנכון:** ALB לא נותן API keys, usage plans, request validation או ייצור SDK. זה API Gateway.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה (MyTodoList — הדוגמה מהשקפים):** אפליקציית מובייל. חשיפה כ-REST API על HTTPS. ארכיטקטורה serverless לחלוטין. המשתמשים צריכים גישה ישירה ל-**תיקייה משלהם** ב-S3. האימות דרך שירות מנוהל serverless. המשתמשים בעיקר **קוראים** to-dos, ופחות כותבים. ה-DB צריך להתרחב ולעמוד ב-read throughput גבוה.

```text
                    ┌──────────────── Amazon Cognito ────────────────┐
                    │  User Pool (sign-in) → Identity Pool (creds)   │
                    └───────┬─────────────────────────┬──────────────┘
                            │ token                   │ temp AWS credentials
                            ▼                         ▼
Mobile ──REST/HTTPS──> API Gateway            S3 (prefix של המשתמש בלבד)
Client                  │  ├── Cognito Authorizer
                        │  ├── Response Caching  ← הרבה קריאות חוזרות
                        │  └── Throttling
                        ▼
                    AWS Lambda
                        │
                        ▼
                  DynamoDB + DAX  ← caching layer ל-read throughput גבוה
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| API Gateway + Lambda | REST על HTTPS בלי שום תשתית לנהל — בדיוק הדרישה |
| **Cognito User Pool** לאימות | שירות זהות מנוהל serverless; תומך בהרשמה, MFA, איפוס סיסמה, ו-federation |
| **Cognito Identity Pool** ל-S3 | הדרישה היא גישה **ישירה** ל-S3 — לכן צריך AWS credentials זמניים, לא רק token |
| IAM policy מבוססת `user_id` | כל משתמש נוגע רק ב-prefix שלו — בידוד ברמת התיקייה |
| DynamoDB ולא RDS | scaling אוטומטי ו-read throughput גבוה בלי ניהול |
| **DAX** לפני DynamoDB | הדרישה "mostly read" + "high read throughput" — DAX מוריד latency למיקרו-שניות |
| **Caching ב-API Gateway** | שכבת cache נוספת לפני ה-Lambda — פחות invocations, פחות עלות, latency נמוך יותר |
| Throttling ב-Stage | מגן על Lambda ו-DynamoDB מפני לקוח תקול או burst |
| Edge-Optimized endpoint | משתמשי מובייל פזורים גיאוגרפית — הכניסה דרך ה-Edge מקצרת latency |

**למה לא ALB?** הוא היה מנתב את הבקשות, אבל בלי Cognito authorizer מובנה ברמת ה-API, בלי caching של תשובות, בלי throttling לכל לקוח ובלי ייצור SDK.

**למה שתי שכבות cache (API GW + DAX)?** הן פועלות ברמות שונות: API Gateway חוסך את **כל** השרשרת (Lambda + DynamoDB) לתשובות זהות; DAX חוסך את הפנייה ל-DynamoDB גם כשה-Lambda כן רצה.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- מיפוי מלא של כל **Integration Types** ותת-הסוגים (AWS_PROXY, HTTP_PROXY, MOCK...) — מספיק להבין את שלוש הקטגוריות.
- תחביר **Velocity Template Language** ל-mapping templates.
- כל השדות של Swagger/OpenAPI.
- פרטי הפרוטוקול של WebSocket API — מספיק לדעת שהוא נתמך.
- הגדרות מפורטות של Step Functions (ASL — Amazon States Language).
- זרימת ה-tokens המדויקת של OAuth/OIDC ב-Cognito.
- מספרי throttling ברירת מחדל מדויקים לכל אזור.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **API Gateway + Lambda = אין תשתית לנהל.** זה הצמד הקלאסי של serverless API.
- שלושה Integration Types: **Lambda**, **HTTP** (כולל ALB ו-on-premises), **AWS Service** (SQS, Kinesis, Step Functions).
- שלושה Endpoint Types: **Edge-Optimized** (ברירת מחדל, גלובלי דרך CloudFront), **Regional**, **Private** (VPC Endpoint + Resource Policy).
- Edge-Optimized **לא** הופך את ה-API ל-multi-region — הוא חי ב-region אחד.
- שלוש שיטות אימות: **IAM** (פנימי) · **Cognito** (חיצוני/מובייל) · **Lambda Authorizer** (לוגיקה משלך).
- **API Keys + Usage Plans** = מכסה ומדידה לכל לקוח, **לא** אימות.
- **User Pool** = התחברות ו-token. **Identity Pool** = AWS credentials זמניים לגישה ישירה ל-S3/DynamoDB.
- User Pools משתלב עם **API Gateway** ועם **ALB**.
- "hundreds of users" / "mobile users" / "SAML" → **Cognito**, לא IAM Users.
- **Certificate:** Edge-Optimized → **us-east-1**. Regional → אותו region. ואז CNAME או A-Alias ב-Route 53.
- **Caching ברמת Stage** — מוריד עומס מה-backend, מחויב לפי שעה.
- **Throttling** ברמת account / stage / method, ו-Usage Plan לכל לקוח.
- תומך ב-**WebSocket**, versioning, environments, Swagger/OpenAPI import, ייצור SDK, ולידציה ו-transform.
- **HTTP API** זול ופשוט; **REST API** עשיר ויקר יותר.
- **Step Functions** = workflow ויזואלי מעל Lambda, כולל תנאים, מקבילות, error handling ו-human approval.

---

## 11. ✅ בדיקת הבנה

1. אפליקציית מובייל צריכה להעלות קבצים ישירות ל-S3, כל משתמש לתיקייה שלו. אילו רכיבי Cognito צריך?
2. ה-API צריך להיות זמין רק מתוך ה-VPC. איזה endpoint type, ואיך שולטים בגישה?
3. הגדרת custom domain על endpoint מסוג Edge-Optimized. באיזה region חייב להיות ה-certificate ולמה?
4. איך מגבילים לקוח מסוים ל-10,000 קריאות בחודש?
5. מה ההבדל בין ALB ל-API Gateway כשצריך לחשוף API ציבורי?
6. רוצים לחשוף SQS ללקוחות חיצוניים עם אימות, בלי לכתוב קוד. איך?
7. יש שאלה עם "mostly read, high read throughput" ב-DynamoDB מאחורי API Gateway. אילו שתי שכבות cache אפשר להוסיף?

<details>
<summary>תשובות</summary>

1. **שניהם.** User Pool לאימות (sign-in ו-token), ואז **Identity Pool** שממיר את ה-token ל-**AWS credentials זמניים**. ה-IAM policy מוגדרת ב-Cognito ומותאמת לפי `user_id` כך שכל משתמש נוגע רק ב-prefix שלו.
2. **Private** endpoint. הגישה רק דרך **Interface VPC Endpoint (ENI)** בתוך ה-VPC, והשליטה מתבצעת דרך **Resource Policy** של ה-API.
3. **us-east-1**. Edge-Optimized מנתב את התעבורה דרך CloudFront Edge Locations, וה-certificate חייב לשבת ב-region של CloudFront. ב-Regional לעומת זאת — באותו region של ה-Stage.
4. **Usage Plan** משויך ל-**API Key** של אותו לקוח, עם quota חודשית ו-rate limit. זה לא מנגנון אימות — האימות עצמו נעשה בנפרד.
5. ALB מנתב HTTP ל-EC2/ECS/Lambda, אבל אינו מוצר ניהול API: אין בו API keys, usage plans, quotas, request validation, ייצור SDK או caching של תשובות. API Gateway נותן את כל אלה.
6. **AWS Service Integration** ב-API Gateway — הוא כותב ישירות ל-SQS, ומעליו מגדירים אימות (Cognito/IAM) ו-throttling. אין צורך ב-Lambda באמצע.
7. **API Gateway Caching** ברמת ה-Stage (חוסך גם את ה-Lambda וגם את ה-DB), ו-**DAX** לפני DynamoDB (מוריד latency למיקרו-שניות בקריאות שכן מגיעות ל-DB).

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[25 - Lambda]] · [[23 - DynamoDB]] · [[08 - Elastic Load Balancing]] · [[15 - CloudFront and Global Delivery]] · [[14 - Route 53 and DNS]] · [[32 - Security Services]] · [[38 - Serverless and Modern Architectures]] · [[03 - IAM Fundamentals]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 8717–8992, 12244–12290
