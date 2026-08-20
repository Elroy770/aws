---
lesson: 17
title: S3 Security and Data Management
domain: Design Secure Architectures
services: [S3, KMS, IAM, CloudTrail, S3 Object Lambda]
tags: [saa-c03, security, s3, encryption, compliance, worm]
---

# 17 — S3 Security and Data Management

> [!abstract] בשורה אחת
> ארבע שיטות הצפנה, שכבת הרשאות אחת שגוברת על כולן (Block Public Access),
> ומודל WORM לרגולציה — זה כל מה שהמבחן שואל על אבטחת S3.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה bucket פתוח הוא הכשל הכי נפוץ בענן |
| 2 | איך זה עובד | 4 שיטות ההצפנה, TLS in transit, Default Encryption |
| 3 | פירוק מפורט | CORS, MFA Delete, Access Logs, Pre-Signed URLs, Object Lock, Access Points |
| 4 | עלות | KMS requests, לוגים, גרסאות שנשמרות בכוח |
| 5 | השוואות | **טבלת 4 שיטות ההצפנה** · Governance מול Compliance |
| 6 | Well-Architected | אבטחת S3 לפי ששת ה-Pillars |
| 7 | מלכודות | הצפנה במנוחה אינה TLS · KMS מגביל throughput |
| 8 | Scenario | bucket פיננסי עם ציות רגולטורי מלא |

**מונחי מפתח בשיעור:** `SSE-S3` · `SSE-KMS` · `SSE-C` · `Client-Side Encryption` · `aws:SecureTransport` · `MFA Delete` · `Pre-Signed URL` · `Object Lock` · `Access Point` · `WORM`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- S3 קל להפוך לציבורי בלחיצה אחת — וזה מקור עשרות דליפות נתונים מתוקשרות.
- רגולציה (SEC 17a-4, HIPAA, GDPR) דורשת שנתונים **לא יהיו ניתנים למחיקה** לתקופה מוגדרת.
- צריך לתת גישה זמנית לקובץ פרטי — בלי לחלק IAM users ובלי לפתוח את ה-bucket.
- ב-bucket אחד יושבים נתונים של חמישה צוותים, וכל אחד צריך לראות רק את ה-prefix שלו.
- ה-bucket policy מתנפחת עד שאף אחד לא מבין מה היא עושה.

### מה השירות פותר

- **הצפנה במנוחה** — ארבע שיטות שנבדלות בשאלה **מי מחזיק במפתח**.
- **הצפנה בתנועה** — HTTPS, וניתן **לכפות** אותו ב-bucket policy.
- **WORM** — Object Lock ו-Glacier Vault Lock מונעים מחיקה, גם מ-root.
- **גישה זמנית** — Pre-Signed URLs נותנים קישור עם תפוגה.
- **ביזור הרשאות** — Access Points מפצלים policy ענקית אחת לכמה קטנות וברורות.
- **ביקורת** — S3 Access Logs ו-CloudTrail מתעדים מי ניגש למה.

> [!tip] האנלוגיה
> הצפנה היא **הכספת**. ההבדל בין ארבע השיטות הוא רק **מי מחזיק במפתח**:
> AWS (SSE-S3), אתם דרך AWS (SSE-KMS), אתם לגמרי אבל AWS מסובבת (SSE-C),
> או שאתם מצפינים לפני שאתם בכלל שולחים (Client-Side).
> **Object Lock הוא ריתוך הדלת** — אפילו בעל הכספת לא יכול לפתוח עד שהזמן עובר.

---

## 2. ⚙️ איך זה עובד

### 2.1 ארבע שיטות ההצפנה — מבט על

```text
                      איפה מתבצעת ההצפנה?
                              │
              ┌───────────────┴───────────────┐
        בצד השרת (SSE)                  בצד הלקוח
              │                               │
   ┌──────────┼──────────┐            Client-Side Encryption
   │          │          │            הלקוח מצפין לפני השליחה
 SSE-S3    SSE-KMS     SSE-C
 מפתח       מפתח       מפתח הלקוח
 של AWS    ב-KMS       ב-header של כל בקשה
```

### 2.2 SSE-S3 — ברירת המחדל

- ההצפנה נעשית **בצד השרת**, במפתח ש-AWS **מייצרת, מנהלת ומחזיקה**.
- אלגוריתם: **AES-256**.
- Header נדרש: `x-amz-server-side-encryption: AES256`.
- **מופעל כברירת מחדל** על buckets חדשים ואובייקטים חדשים.
- **אין עלות נוספת** ואין מגבלת throughput.
- החיסרון: אין לכם שליטה על המפתח, אין rotation שאתם קובעים, ואין audit ייעודי לשימוש במפתח.

### 2.3 SSE-KMS — כשצריך שליטה ו-audit

- המפתח מנוהל ב-**AWS KMS**. אתם מגדירים key policy, rotation ו-grants.
- Header נדרש: `x-amz-server-side-encryption: aws:kms`.
- **היתרון הגדול:** כל שימוש במפתח נרשם ב-**CloudTrail**.
  זה מה שהופך אותו לתשובה כשהשאלה אומרת *"audit key usage"* או *"customer-managed key"*.

> [!warning] המגבלה שנשאלת: KMS quota
> כל **העלאה** קוראת ל-`GenerateDataKey`. כל **הורדה** קוראת ל-`Decrypt`.
> שתי הקריאות נספרות מול ה-**KMS quota לשנייה** — בערך **5,500 / 10,000 / 30,000 בקשות/שנייה**
> תלוי ב-Region.
> **המשמעות:** bucket עם תעבורה גבוהה מאוד ב-SSE-KMS יכול להיתקל ב-**throttling**.
> **הפתרונות:** בקשת הגדלת quota ב-Service Quotas, או שימוש ב-SSE-S3 היכן שאין דרישת audit,
> או הפעלת **S3 Bucket Keys** שמפחיתה דרמטית את מספר קריאות ה-KMS.

### 2.4 SSE-C — המפתח אצלכם, ההצפנה אצל AWS

- אתם מנהלים את המפתח **מחוץ ל-AWS** לגמרי.
- **S3 לא שומרת את המפתח שסיפקתם.** לעולם. היא משתמשת בו ומוחקת אותו מהזיכרון.
- **חובה HTTPS** — כי המפתח עצמו נשלח ב-header.
- חייבים לספק את המפתח ב-header של **כל בקשה**, גם בהעלאה וגם בהורדה.
- **לא נתמך בקונסולה** — עובדים רק דרך CLI/SDK.

> [!warning] אבדתם את המפתח ב-SSE-C?
> **הנתונים אבודים לצמיתות.** ל-AWS אין עותק. זה הסיכון המרכזי של השיטה.

### 2.5 Client-Side Encryption

- הלקוח **מצפין בעצמו לפני** השליחה, ו**מפענח בעצמו** אחרי ההורדה.
- משתמשים בספריות כמו **Amazon S3 Client-Side Encryption Library**.
- ל-AWS אין שום גישה ל-plaintext — היא רואה רק בייטים מוצפנים.
- הלקוח מנהל את **כל** מחזור החיים של המפתח.
- מתאים כשהדרישה היא *"AWS must never have access to unencrypted data"*.

### 2.6 הצפנה בתנועה (SSL/TLS)

- S3 חושפת **שני endpoints**: HTTP (לא מוצפן) ו-HTTPS (מוצפן).
- HTTPS הוא **המומלץ** ורוב ה-clients משתמשים בו כברירת מחדל.
- **HTTPS הוא חובה מוחלטת ב-SSE-C** (כי המפתח עובר ב-header).

**כפיית HTTPS — התנאי שחייבים להכיר:**

```text
Bucket Policy → Effect: Deny
                Action: s3:*
                Condition: "Bool": { "aws:SecureTransport": "false" }
```

- התנאי `aws:SecureTransport` הוא `false` כשהבקשה הגיעה ב-HTTP.
- ה-**Deny** על `false` הוא הדרך הסטנדרטית לכפות TLS.
- זו התשובה כשבשאלה כתוב *"ensure data is encrypted in transit"* או *"reject non-HTTPS requests"*.

### 2.7 Default Encryption מול Bucket Policy

| מנגנון | מה עושה |
|---|---|
| **Default Encryption** | SSE-S3 מוחל **אוטומטית** על כל אובייקט חדש. שקוף למעלה |
| **Bucket Policy כופה** | **דוחה** קריאת PUT שאין בה header הצפנה מתאים (SSE-KMS או SSE-C) |

> [!warning] סדר ההערכה
> **Bucket Policies נבדקות לפני Default Encryption.**
> כלומר אם ה-policy דורשת header של `aws:kms` והלקוח לא שלח אותו —
> הבקשה **נדחית**, ו-Default Encryption לא "מציל" אותה.
> זו הדרך לכפות SSE-KMS ספציפית ולא להסתפק ב-SSE-S3.

---

## 3. 🔍 פירוק מפורט

### 3.1 CORS — שאלה פופולרית במבחן

**CORS** = Cross-Origin Resource Sharing. מנגנון של **הדפדפן**, לא של השרת.

**Origin** = protocol + host + port:

| זוג כתובות | אותו origin? |
|---|---|
| `http://example.com/app1` ו-`http://example.com/app2` | ✅ **כן** — רק ה-path שונה |
| `http://www.example.com` ו-`http://other.example.com` | ❌ לא — host שונה |
| `http://example.com` ו-`https://example.com` | ❌ לא — protocol שונה |
| `https://example.com` ו-`https://example.com:8443` | ❌ לא — port שונה |

**איך זה עובד:**

```text
1) הדפדפן שולח  OPTIONS  (Preflight Request)
   Host: www.other.com
   Origin: https://www.example.com

2) השרת עונה עם CORS Headers:
   Access-Control-Allow-Origin: https://www.example.com
   Access-Control-Allow-Methods: GET, PUT, DELETE

3) רק אז הדפדפן שולח את הבקשה האמיתית (GET)
```

**התרחיש הקלאסי ב-S3:**

- bucket אחד מארח את ה-HTML, bucket שני מארח תמונות/assets.
- הדפדפן טוען `index.html` מה-bucket הראשון ואז מנסה למשוך `coffee.jpg` מהשני.
- **בלי CORS על bucket ה-assets — הדפדפן חוסם את הבקשה.**
- מגדירים **CORS configuration** על bucket ה-assets עם ה-origin של bucket ה-HTML,
  או `*` לכל ה-origins.

> [!tip] איך מזהים שאלת CORS
> אם בשאלה מופיעים **שני buckets**, **דפדפן**, ושגיאה של **"blocked"** או **"cross-origin"** —
> התשובה היא CORS על ה-bucket **שממנו מושכים** את המשאב.

### 3.2 MFA Delete

**תנאי מוקדם: Versioning חייב להיות מופעל.**

| פעולה | דורשת MFA? |
|---|---|
| **מחיקה לצמיתות** של version של אובייקט | ✅ **כן** |
| **Suspend** ל-versioning ב-bucket | ✅ **כן** |
| **הפעלת** versioning | ❌ לא |
| **רשימת** גרסאות שנמחקו | ❌ לא |

> [!warning] רק ה-root יכול
> **רק בעל ה-bucket (חשבון ה-root) יכול להפעיל או לכבות MFA Delete.**
> לא IAM user, גם לא אדמין. זו נקודה שנשאלת ישירות.
> בפועל זה גם החיסרון: זה לא ניתן לאוטומציה נוחה, ולכן הרבה ארגונים
> מעדיפים **Object Lock** לצרכי ציות.

### 3.3 S3 Access Logs

- מתעדים **כל** בקשה ל-bucket: מכל חשבון, מאושרת או נדחית.
- הלוגים נכתבים ל-**bucket אחר** וניתן לנתח אותם ב-Athena.
- **ה-bucket של הלוגים חייב להיות באותו Region** של ה-bucket שמנוטר.

> [!warning] לולאת הלוגים
> **אל תגדירו את ה-bucket כ-bucket הלוגים של עצמו.**
> כל כתיבת לוג היא PutObject, שמייצרת לוג, שהוא PutObject... — **גדילה מעריכית**
> ועלות שמתפוצצת. תמיד bucket נפרד.

**Access Logs מול CloudTrail:**

| קריטריון | S3 Access Logs | CloudTrail Data Events |
|---|---|---|
| עלות | אחסון בלבד | **בתשלום לכל אירוע** |
| זמן הופעה | דקות עד שעות (best effort) | כמעט מיידי |
| מתאים ל | ניתוח תעבורה בהיקף גדול, billing | חקירת אבטחה, התראות בזמן אמת |

### 3.4 Pre-Signed URLs

- קישור **זמני** שמאפשר GET או PUT לאובייקט ב-bucket **פרטי**.
- **ה-URL יורש את ההרשאות של מי שיצר אותו.** אם ליוצר אין הרשאה — גם ל-URL אין.
- נוצר דרך הקונסולה, ה-CLI או ה-SDK.

| דרך היצירה | טווח תפוגה |
|---|---|
| **S3 Console** | דקה אחת עד **720 דקות (12 שעות)** |
| **AWS CLI / SDK** | פרמטר `--expires-in` בשניות. ברירת מחדל **3,600** (שעה), מקסימום **604,800** (~168 שעות = 7 ימים) |

**Use cases קלאסיים:**

- להוריד וידאו פרימיום — רק למשתמשים מחוברים.
- רשימת משתמשים משתנה כל הזמן — מייצרים URL דינמית במקום לנהל IAM.
- לאפשר למשתמש **להעלות** קובץ למיקום מדויק ב-bucket, לזמן מוגבל.

> [!tip] Pre-Signed URL היא התשובה כש...
> בשאלה מופיע *"temporary access"*, *"share a private file"*, *"without making the bucket public"*,
> או *"users who don't have AWS credentials"*.

### 3.5 WORM — Glacier Vault Lock מול S3 Object Lock

**WORM** = Write Once Read Many. כותבים פעם אחת, אי אפשר לשנות או למחוק.

**S3 Glacier Vault Lock:**

- חל על **Vault** של Glacier (המנגנון הישן, נפרד מ-storage classes של S3).
- יוצרים **Vault Lock Policy** ואז **נועלים** אותה.
- אחרי הנעילה — **אי אפשר לשנות או למחוק את ה-policy לעולם**.

**S3 Object Lock — התנאי: Versioning חייב להיות מופעל.**

חוסם מחיקה של **version** של אובייקט לתקופה מוגדרת. שני מצבים:

| מצב | מי יכול לעקוף | האם ניתן לקצר תקופה |
|---|---|---|
| **Compliance** | **אף אחד — כולל root** | **לא.** גם לא לשנות את המצב |
| **Governance** | רק מי שיש לו **הרשאה מיוחדת** | כן, על ידי בעל ההרשאה |

**שני מנגנוני שמירה:**

| מנגנון | פירוט |
|---|---|
| **Retention Period** | הגנה ל-**תקופה קצובה**. ניתן **להאריך**, לא לקצר (ב-Compliance) |
| **Legal Hold** | הגנה **ללא תאריך סיום**, בלתי תלויה ב-retention period. מוסרים ומוסיפים בחופשיות עם ההרשאה `s3:PutObjectLegalHold` |

> [!tip] Compliance מול Governance — ההכרעה
> בשאלה מופיע *"not even the root user"* או *"regulatory requirement"* → **Compliance**.
> בשאלה מופיע *"administrators can override if needed"* → **Governance**.

### 3.6 S3 Access Points

**הבעיה:** bucket אחד עם prefixes של finance, sales ו-analytics
→ bucket policy אחת ענקית ובלתי ניתנת לתחזוקה.

**הפתרון:** לכל צוות **Access Point** משלו.

| לכל Access Point יש | פירוט |
|---|---|
| **שם DNS ייחודי** | Internet Origin או **VPC Origin** |
| **Access Point Policy** | דומה ל-bucket policy, אבל מצומצם ל-prefix הרלוונטי |

```text
Users (Finance)   → Finance   Access Point → R/W על  /finance/…
Users (Sales)     → Sales     Access Point → R/W על  /sales/…
Users (Analytics) → Analytics Access Point → Read על כל ה-bucket
                          ↓
                    S3 Bucket אחד עם Bucket Policy פשוטה
```

**VPC Origin — נעילה לרשת פרטית:**

- אפשר להגדיר Access Point שנגיש **רק מתוך ה-VPC**.
- חובה ליצור **VPC Endpoint** (Gateway או Interface) כדי להגיע אליו.
- **ה-VPC Endpoint Policy חייבת להתיר גישה גם ל-bucket וגם ל-Access Point.**
- זו שרשרת של שלוש policies: Endpoint Policy → Access Point Policy → Bucket Policy.
  ראו [[12 - VPC Private Connectivity]].

### 3.7 S3 Object Lambda

- מאפשר להריץ **Lambda function** שמשנה את האובייקט **בדרך החוצה**, לפני שהקורא מקבל אותו.
- **צריך רק bucket אחד.** מעליו יוצרים S3 Access Point ו-**Object Lambda Access Point**.
- אין שכפול נתונים — האובייקט המקורי נשאר יחיד.

| Use case | מה ה-Lambda עושה |
|---|---|
| **Redaction** | מסתירה PII לפני שנתונים מגיעים לסביבת analytics או dev |
| **המרת פורמט** | XML → JSON בזמן הקריאה |
| **עיבוד תמונה** | resize ו-watermark לפי זהות הקורא |
| **העשרה** | מוסיפה נתונים ממסד נתונים חיצוני לאובייקט המוחזר |

> [!tip] מתי זו התשובה
> כשבשאלה כתוב *"different views of the same data without duplicating it"*
> או *"redact sensitive fields for one consumer only"*.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **SSE-S3** | **0** | אין תוספת עלות להצפנה |
| **SSE-KMS — אחסון מפתח** | לחודש לכל CMK | ה-AWS managed key חינם; customer managed בתשלום |
| **SSE-KMS — קריאות API** | לכל 10,000 בקשות | `GenerateDataKey` בכל PUT, `Decrypt` בכל GET |
| **SSE-C / Client-Side** | **0** ל-AWS | העלות היא ניהול המפתחות אצלכם |
| **S3 Access Logs** | אחסון האובייקטים ב-bucket הלוגים + PUT requests | הרישום עצמו חינם |
| **CloudTrail Data Events** | **לכל אירוע** | היקר מבין אפשרויות הביקורת |
| **Object Lock / Versioning** | אחסון כל הגרסאות שנשמרות בכוח | אי אפשר למחוק → אי אפשר להוזיל |
| **Access Points** | **0** לרכיב | משלמים רק על הבקשות הרגילות |
| **Object Lambda** | הרצת Lambda + GB שעברו דרכה + בקשות S3 | שלוש שכבות חיוב |
| **Macie** | לפי GB שנסרק | רק אם מפעילים |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| **SSE-S3** | **0 תוספת** | ברירת המחדל. כשאין דרישת audit או שליטה במפתח |
| **SSE-KMS** עם AWS managed key | קריאות KMS בלבד | כשצריך audit ב-CloudTrail |
| **SSE-KMS** עם customer managed key | תשלום חודשי למפתח + קריאות | דרישה של rotation/מדיניות משלכם |
| **SSE-KMS + S3 Bucket Keys** | מוזיל את קריאות ה-KMS דרמטית | **כמעט תמיד כדאי** ב-buckets עמוסים |
| **Client-Side** | 0 ל-AWS, עלות תפעולית גבוהה | דרישה ש-AWS לא תראה plaintext |
| **Access Logs ל-S3** | זול | ניתוח בהיקף גדול |
| **CloudTrail Data Events** | יקר | רק על buckets רגישים |

### 🚩 עלויות נסתרות

- **קריאות KMS מצטברות** — bucket עם מיליוני GET ביום ב-SSE-KMS מייצר מיליוני קריאות `Decrypt`.
  **S3 Bucket Keys** מצמצמות זאת בסדרי גודל.
- **Throttling של KMS** — לא עלות ישירה, אבל בקשות שנכשלות → retries → יותר עלות ופחות ביצועים.
- **Object Lock ב-Compliance** — נתונים שנעולים ל-7 שנים **חייבים** להיות מאוחסנים 7 שנים.
  אם נעלתם אותם ב-Standard במקום ב-Glacier — שילמתם פי עשרות.
- **Versioning בלי lifecycle** — כל גרסה ישנה מחויבת. ראו [[18 - S3 Advanced Features]].
- **לולאת Access Logs** — bucket שמלוגג את עצמו גדל מעריכית עד שמישהו שם לב לחשבון.
- **CloudTrail Data Events על bucket עמוס** — יכול להיות פריט משמעותי בחשבון.
- **Object Lambda** — משלמים על Lambda **בכל GET**, לא פעם אחת. ב-traffic גבוה זה מצטבר.

### 💡 טיפים לחיסכון

- **הפעילו S3 Bucket Keys** בכל bucket שמשתמש ב-SSE-KMS. זה החיסכון הגדול והקל ביותר.
- **SSE-S3 כברירת מחדל**, SSE-KMS רק היכן שיש דרישת audit או שליטה במפתח.
- **הפעילו Object Lock על ה-storage class הנכון** — Glacier Deep Archive לשימור ארוך.
- **CloudTrail Data Events רק על ה-buckets הרגישים**, לא על כולם.
- **Lifecycle על noncurrent versions** — כדי ש-versioning לא יהפוך לחוב מצטבר.
- **Access Logs ל-bucket זול** (Standard-IA) עם lifecycle ל-Glacier אחרי תקופת הניתוח.

---

## 5. ⚖️ השוואות מכריעות

### הטבלה המרכזית — ארבע שיטות ההצפנה

| קריטריון | **SSE-S3** | **SSE-KMS** | **SSE-C** | **Client-Side** |
|---|---|---|---|---|
| **מי מחזיק במפתח** | AWS — מייצרת, מנהלת ומחזיקה | **אתם**, דרך KMS | **אתם, מחוץ ל-AWS**. S3 לא שומרת אותו | **אתם לגמרי** |
| **מי מבצע את ההצפנה** | S3 (server-side) | S3 עם מפתח מ-KMS (server-side) | S3 עם המפתח שסיפקתם (server-side) | **הלקוח**, לפני השליחה |
| **Header נדרש** | `x-amz-server-side-encryption: AES256` | `x-amz-server-side-encryption: aws:kms` | **המפתח עצמו** ב-header, **בכל בקשה** | אין — S3 לא מעורבת |
| **HTTPS חובה** | מומלץ | מומלץ | ✅ **חובה מוחלטת** | מומלץ |
| **Audit ב-CloudTrail** | ❌ אין audit על השימוש במפתח | ✅ **כן — כל שימוש נרשם** | ❌ | ❌ (מחוץ ל-AWS) |
| **מגבלת throughput** | אין | ✅ **מוגבל ב-KMS quota** (~5,500/10,000/30,000 בקשות/שנייה) | אין | אין |
| **עלות** | **0** | אחסון מפתח + קריאות API | 0 ל-AWS | 0 ל-AWS |
| **תמיכה בקונסולה** | ✅ | ✅ | ❌ CLI/SDK בלבד | ❌ |
| **מופעל כברירת מחדל** | ✅ **כן** | ❌ | ❌ | ❌ |
| **מה קורה אם איבדתם מפתח** | לא רלוונטי | ניתן לשחזור לפי מדיניות KMS | ⚠️ **הנתונים אבודים לצמיתות** | ⚠️ **הנתונים אבודים לצמיתות** |
| **מתי בוחרים** | ברירת המחדל, פשטות | *"audit key usage"*, *"customer-managed key"*, rotation | *"keys managed outside AWS"* + AWS מצפינה | *"AWS must never see plaintext"* |

### Object Lock — Compliance מול Governance

| קריטריון | Compliance Mode | Governance Mode |
|---|---|---|
| root יכול למחוק | ❌ **לא** | ✅ כן (עם הרשאה) |
| ניתן לקצר את התקופה | ❌ לא | ✅ כן |
| ניתן לשנות את המצב | ❌ לא | ✅ כן |
| מתאים ל | רגולציה קשיחה (SEC 17a-4, ציות פיננסי) | הגנה תפעולית מפני מחיקה בטעות |

### MFA Delete מול Object Lock

| קריטריון | MFA Delete | Object Lock |
|---|---|---|
| דורש versioning | ✅ | ✅ |
| מי מפעיל | **רק root** | כל מי שיש לו הרשאות מתאימות |
| ניתן לאוטומציה | ❌ קשה מאוד | ✅ |
| מונע מחיקה | דורש קוד MFA לכל מחיקה | חוסם מחיקה **לחלוטין** לתקופה |
| מתאים ל | הגנה ידנית על bucket קריטי | **ציות רגולטורי בקנה מידה** |

> [!info] שורה תחתונה
> **SSE-S3 עד שיש סיבה.** הסיבה היא כמעט תמיד *audit* או *שליטה במפתח* → SSE-KMS.
> **SSE-C ו-Client-Side** מופיעים רק כשהשאלה אומרת במפורש שהמפתחות מנוהלים מחוץ ל-AWS.
> **Object Lock ב-Compliance** הוא התשובה לכל שאלה שאומרת *"cannot be deleted by anyone"*.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **באבטחת S3** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | ה-policies מנוהלות כקוד ונבדקות | bucket policies ב-IaC; **Access Points** במקום policy אחת ענקית; Access Analyzer for S3 להתראה על חשיפה; rotation מתוזמן למפתחות KMS |
| **Security** | הצפנה תמיד, גישה מינימלית, שום דבר לא ציבורי | Block Public Access ברמת **החשבון**; Deny על `aws:SecureTransport: false`; SSE-KMS על נתונים רגישים; ACLs מכובים; Pre-Signed URLs במקום bucket ציבורי |
| **Reliability** | טעות אנוש ומחיקה זדונית הפיכות | Versioning + **MFA Delete** על buckets קריטיים; **Object Lock** לציות; תוכנית שחזור למפתח KMS (מה קורה אם מוחקים CMK) |
| **Performance Efficiency** | ההצפנה לא הופכת לצוואר בקבוק | **S3 Bucket Keys** להפחתת קריאות KMS; לא SSE-KMS על נתיב חם ללא צורך; VPC Endpoint במקום מסלול דרך האינטרנט |
| **Cost Optimization** | משלמים על השליטה רק היכן שצריך אותה | SSE-S3 (חינם) כברירת מחדל; CloudTrail Data Events רק על buckets רגישים; lifecycle לגרסאות ישנות; Access Logs ל-class זול |
| **Sustainability** | פחות עותקים ופחות עיבוד מיותר | **Object Lambda** במקום לשכפל bucket שני לגרסה מצונזרת; retention מוגדר במקום שמירה לנצח |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "audit key usage" / "track who used the key" | **SSE-KMS** (CloudTrail) |
| "keys managed outside of AWS, AWS performs encryption" | **SSE-C** |
| "AWS must never have access to the unencrypted data" | **Client-Side Encryption** |
| "encryption with minimal effort / no extra cost" | **SSE-S3** |
| "ensure data is encrypted in transit" / "reject HTTP" | Bucket Policy Deny על **`aws:SecureTransport: false`** |
| "getting throttled during heavy uploads with encryption" | **KMS quota** → S3 Bucket Keys או הגדלת quota |
| "cannot be deleted by anyone, including root" | **Object Lock — Compliance mode** |
| "administrators may override the retention" | **Object Lock — Governance mode** |
| "protect indefinitely, no fixed end date" | **Legal Hold** |
| "share a private file temporarily" | **Pre-Signed URL** |
| "browser blocks requests from another bucket" | **CORS** |
| "require MFA before permanently deleting a version" | **MFA Delete** (root בלבד, versioning חובה) |
| "different teams need access to different prefixes at scale" | **S3 Access Points** |
| "redact PII without duplicating the data" | **S3 Object Lambda** |
| "bucket accessible only from within the VPC" | **Access Point — VPC Origin** + VPC Endpoint |
| "log every request to the bucket for audits" | **S3 Access Logs** (bucket נפרד, אותו Region) |

### טעויות נפוצות

> [!warning] מלכודת 1 — הצפנה במנוחה מחליפה TLS
> **הניסוח:** "We enabled SSE-KMS, so data is protected in transit as well."
> **הטעות:** לערבב at-rest עם in-transit.
> **הנכון:** SSE מצפינה **על הדיסק**. ההגנה בתנועה היא **HTTPS**,
> ונכפית ב-bucket policy עם `aws:SecureTransport`.

> [!warning] מלכודת 2 — SSE-KMS בלי לחשוב על throughput
> **הניסוח:** "Uploads started failing with throttling errors after enabling SSE-KMS."
> **הטעות:** לחפש את הבעיה ב-S3.
> **הנכון:** ה-quota של **KMS** נגמרה (~5,500–30,000 בקשות/שנייה לפי Region).
> הפתרונות: **S3 Bucket Keys**, הגדלת quota, או SSE-S3 היכן שאין דרישת audit.

> [!warning] מלכודת 3 — לחשוב ש-S3 שומרת את מפתח ה-SSE-C
> **הניסוח:** "We lost our SSE-C key — AWS Support can recover the objects."
> **הטעות:** להניח שיש גיבוי אצל AWS.
> **הנכון:** **S3 לא שומרת את המפתח.** אין שחזור. הנתונים אבודים.

> [!warning] מלכודת 4 — MFA Delete על ידי אדמין
> **הניסוח:** "The security admin will enable MFA Delete on the bucket."
> **הטעות:** להניח שכל אדמין יכול.
> **הנכון:** **רק חשבון ה-root של בעל ה-bucket.** ובנוסף — **versioning חייב להיות מופעל**.

> [!warning] מלכודת 5 — bucket הלוגים הוא ה-bucket עצמו
> **הניסוח:** "Enable access logging and store the logs in the same bucket for simplicity."
> **הטעות:** לבחור בזה כי זה נשמע נוח.
> **הנכון:** **לולאת לוגים** — כל לוג מייצר לוג. גדילה מעריכית. תמיד bucket נפרד,
> **באותו Region**.

> [!warning] מלכודת 6 — Pre-Signed URL עוקף הרשאות
> **הניסוח:** "Generate a pre-signed URL so users can access a bucket we have no permissions to."
> **הטעות:** להניח שה-URL מייצר הרשאה יש מאין.
> **הנכון:** ה-URL **יורש את ההרשאות של מי שיצר אותו**. אין הרשאה ליוצר → אין גישה ל-URL.

> [!warning] מלכודת 7 — Default Encryption גובר על Bucket Policy
> **הניסוח:** "Default encryption is on, so a PUT without headers will succeed and be encrypted."
> **הטעות:** להניח ש-Default Encryption "מציל" בקשה.
> **הנכון:** **Bucket Policies נבדקות לפני Default Encryption.**
> אם ה-policy דורשת header של הצפנה ואין — הבקשה **נדחית**.

> [!warning] מלכודת 8 — Governance מספיק לרגולציה
> **הניסוח:** "Regulator requires records that no one can delete for 5 years. Use Governance mode."
> **הטעות:** לבחור Governance כי הוא גמיש.
> **הנכון:** ב-Governance יש מי שיכול לעקוף. לדרישה של *"no one, not even root"* → **Compliance mode**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

בנק שומר רשומות עסקאות ב-S3.

- רגולטור דורש: הרשומות **לא ניתנות למחיקה או שינוי במשך 7 שנים** — גם לא על ידי מנהל המערכת.
- כל שימוש במפתח ההצפנה חייב להיות ניתן לביקורת.
- אסור שהנתונים יעברו אי פעם ב-HTTP.
- ה-bucket חייב להיות נגיש **רק** מתוך ה-VPC של הבנק.
- צוות ה-analytics צריך את הנתונים **בלי שדות ה-PII**.
- מחלקת compliance צריכה לדעת מי ניגש לכל אובייקט.

**הארכיטקטורה:**

```text
        VPC (הבנק)
   ┌────────────────────┐
   │  EC2 / Analytics   │
   │        │           │
   │   VPC Endpoint     │  Endpoint Policy
   └────────┼───────────┘
            ▼
   ┌── Access Point (VPC Origin) ──┐
   │  Access Point Policy          │
   └────────┬──────────────────────┘
            ▼
   ┌── records-bucket ────────────────────────────┐
   │  SSE-KMS (customer managed CMK)              │
   │  Object Lock: Compliance, 7 years            │
   │  Versioning ✅   Block Public Access ✅        │
   │  Bucket Policy: Deny aws:SecureTransport=false│
   │  Lifecycle → Glacier Deep Archive             │
   └──────────┬────────────────────┬───────────────┘
              │                    │
   Object Lambda Access Point   S3 Access Logs → logs-bucket
   (redaction) → Analytics          + CloudTrail Data Events
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **Object Lock — Compliance, 7 שנים** | הדרישה אומרת *"גם לא מנהל"*. Governance מאפשר עקיפה; Compliance לא — **גם לא ל-root** |
| **Versioning מופעל** | תנאי סף ל-Object Lock. בלעדיו אי אפשר להפעיל אותו כלל |
| **SSE-KMS עם customer managed CMK** | הדרישה היא **audit על השימוש במפתח** — רק KMS נותן רישום ב-CloudTrail |
| **S3 Bucket Keys דלוקות** | מפחיתות דרמטית את מספר קריאות ה-KMS ומרחיקות מה-quota |
| **Deny על `aws:SecureTransport: false`** | חוסם כל בקשת HTTP ברמת ה-policy, לא מסתמך על הגדרת הלקוח |
| **Access Point עם VPC Origin** | ה-bucket נגיש רק דרך ה-VPC. גם credentials שדלפו לא יעזרו מבחוץ |
| **VPC Endpoint + Endpoint Policy** | חובה כדי להגיע ל-Access Point מסוג VPC Origin. שלוש policies בשרשרת |
| **Block Public Access ברמת החשבון** | ביטוח מפני bucket policy ציבורית שמישהו יגדיר בטעות |
| **S3 Object Lambda לצוות analytics** | מחזיר גרסה מצונזרת של אותו אובייקט — **בלי לשכפל את ה-bucket** |
| **Lifecycle ל-Glacier Deep Archive** | Object Lock מחייב שמירה של 7 שנים. ב-Standard זה עולה פי עשרות מונים |
| **S3 Access Logs + CloudTrail Data Events** | Access Logs לניתוח בהיקף, Data Events להתראות ולחקירה מדויקת |

**למה לא MFA Delete במקום Object Lock?**
כי MFA Delete מופעל **רק על ידי root**, לא ניתן לאוטומציה, ובעיקר —
הוא **דורש קוד MFA** למחיקה אבל **לא מונע** אותה. הרגולטור דורש מניעה מוחלטת → Compliance mode.

**למה לא SSE-S3?**
הוא חינם ופשוט, אבל **אין לו audit על השימוש במפתח**. הדרישה המפורשת מוציאה אותו מהמשחק.

**למה לא bucket שני עם עותק מצונזר ל-analytics?**
זה מכפיל אחסון, מכפיל שטח תקיפה, ומחייב סנכרון.
**Object Lambda** מחזיר view שונה מאותו אובייקט — עותק אחד, שתי תצוגות.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **התחביר המדויק** של כל header הצפנה — מספיק לדעת ש-SSE-C דורש את המפתח ב-header של כל בקשה.
- **מבנה ה-JSON המלא** של CORS configuration — מספיק להבין מתי CORS נדרש.
- **הפורמט המדויק** של שורת S3 Access Log — יש דוקומנטציה.
- **אלגוריתמי ההצפנה הפנימיים** של KMS ו-envelope encryption לעומק.
- **מספרי quota מדויקים** של KMS לכל Region — מספיק לדעת שיש quota ושהיא בסדר גודל של אלפים בשנייה.
- **Glacier Vault Lock לעומק** — Object Lock החליף אותו ברוב הארכיטקטורות החדשות.
- **ACLs לעומק** — AWS ממליצה לכבות; המבחן מתמקד ב-bucket policies וב-Access Points.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **4 שיטות הצפנה:** SSE-S3 (AWS מנהלת, ברירת מחדל, חינם) · SSE-KMS (audit ב-CloudTrail) ·
  SSE-C (המפתח שלכם ב-header, HTTPS חובה) · Client-Side (הלקוח מצפין לפני).
- **SSE-S3 header:** `AES256`. **SSE-KMS header:** `aws:kms`.
- **SSE-KMS מוגבל ב-KMS quota** (~5,500 / 10,000 / 30,000 בקשות/שנייה). הפתרון: **S3 Bucket Keys**.
- **SSE-C: HTTPS חובה, S3 לא שומרת את המפתח, אבד המפתח = אבדו הנתונים.**
- **Bucket Policy נבדקת לפני Default Encryption.**
- **כפיית TLS:** Deny על `aws:SecureTransport: false`.
- **CORS:** origin = protocol + host + port. Preflight `OPTIONS` → `Access-Control-Allow-Origin`.
- **MFA Delete:** versioning חובה · **רק root מפעיל** · נדרש למחיקת version ול-Suspend, **לא** להפעלה.
- **Access Logs:** bucket **נפרד**, **אותו Region**, אף פעם לא את עצמו (לולאה).
- **Pre-Signed URL:** יורש הרשאות של היוצר · קונסולה עד **12 שעות** · CLI עד **7 ימים** (604,800 שניות).
- **Object Lock דורש versioning.** **Compliance** = אף אחד, כולל root. **Governance** = ניתן לעקיפה.
- **Retention Period** ניתן להארכה בלבד · **Legal Hold** בלי תאריך סיום, `s3:PutObjectLegalHold`.
- **Glacier Vault Lock** = WORM על Vault; policy שננעלה **לא ניתנת לשינוי לעולם**.
- **Access Points:** DNS משלהם + policy משלהם. **VPC Origin** דורש VPC Endpoint ושלוש policies.
- **Object Lambda:** משנה את האובייקט **בזמן הקריאה**, bucket אחד בלבד. redaction / המרת פורמט / resize.
- **גישה = (IAM ALLOW או Resource ALLOW) AND אין DENY.** ראו [[16 - S3 Fundamentals]].

---

## 11. ✅ בדיקת הבנה

1. הפעלתם SSE-KMS ופתאום העלאות נכשלות ב-throttling. מה קרה ומה הפתרון?
2. איזו שיטת הצפנה מחייבת HTTPS, ולמה דווקא היא?
3. הרגולטור דורש שאף אחד — כולל root — לא יוכל למחוק רשומה במשך 5 שנים. מה מגדירים?
4. איך כופים שכל התעבורה ל-bucket תהיה מוצפנת בתנועה?
5. Default Encryption דלוק, וה-bucket policy דורשת header של `aws:kms`. לקוח שולח PUT בלי header. מה קורה?
6. מי יכול להפעיל MFA Delete, ומה תנאי הסף?
7. אתר ב-bucket A טוען תמונות מ-bucket B והדפדפן חוסם. מה הבעיה ואיפה מתקנים?
8. צריך לתת ללקוח חיצוני להוריד קובץ פרטי לשעה אחת. מה הפתרון, ומה מגביל אותו?
9. צוות analytics צריך את אותם נתונים בלי שדות PII. מה עדיף על שכפול ה-bucket?
10. מה ההבדל בין Retention Period ל-Legal Hold?

<details>
<summary>תשובות</summary>

1. כל PUT קורא ל-`GenerateDataKey` וכל GET ל-`Decrypt`, ושניהם נספרים מול **quota של KMS**
   (~5,500 / 10,000 / 30,000 בקשות/שנייה לפי Region). הפתרונות:
   **S3 Bucket Keys** (מפחיתות את מספר הקריאות דרמטית), בקשת הגדלת quota, או SSE-S3 היכן שאין דרישת audit.
2. **SSE-C.** מפתח ההצפנה עצמו נשלח ב-**HTTP header** של כל בקשה —
   שליחתו ב-HTTP לא מוצפן הייתה חושפת אותו בדרך.
3. **S3 Object Lock במצב Compliance** עם retention period של 5 שנים.
   תנאי סף: **versioning מופעל**. Governance לא מספיק — שם יש מי שיכול לעקוף.
4. **Bucket Policy** עם `Effect: Deny` על `s3:*` בתנאי `aws:SecureTransport: false`.
   כך כל בקשה שהגיעה ב-HTTP נדחית.
5. **הבקשה נדחית.** **Bucket Policies נבדקות לפני Default Encryption** —
   ה-Default לא "משלים" header חסר.
6. **רק חשבון ה-root של בעל ה-bucket.** תנאי סף: **Versioning מופעל**.
7. **CORS.** הדפדפן חוסם בקשה cross-origin ללא headers מתאימים.
   מתקנים ב-**bucket B** (ה-bucket שממנו מושכים) — מגדירים CORS עם ה-origin של bucket A או `*`.
8. **Pre-Signed URL** עם תפוגה של שעה. המגבלה: ה-URL **יורש את ההרשאות של מי שיצר אותו** —
   אם ליוצר אין `s3:GetObject`, גם ל-URL אין. בנוסף: קונסולה עד 12 שעות, CLI/SDK עד 7 ימים.
9. **S3 Object Lambda** עם Object Lambda Access Point שמריץ פונקציית redaction בזמן הקריאה.
   bucket אחד, שתי תצוגות — בלי שכפול, בלי סנכרון, בלי שטח תקיפה כפול.
10. **Retention Period** הוא לתקופה **קצובה** (ניתן להאריך, לא לקצר ב-Compliance).
    **Legal Hold** הוא **ללא תאריך סיום**, בלתי תלוי ב-retention period,
    ומוסר/מוסף בחופשיות עם ההרשאה `s3:PutObjectLegalHold`.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[16 - S3 Fundamentals]] · [[18 - S3 Advanced Features]] · [[03 - IAM Fundamentals]] · [[04 - IAM Advanced and Organizations]] · [[12 - VPC Private Connectivity]] · [[32 - Security Services]] · [[35 - Backup and Data Protection]] · [[31 - Monitoring and Logging]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 4776–4871, 5534–5962
