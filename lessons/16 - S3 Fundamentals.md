---
lesson: 16
title: S3 Fundamentals
domain: Design Cost-Optimized Architectures
services: [S3, S3 Glacier, IAM, S3 Replication]
tags: [saa-c03, storage, s3, object-storage, storage-classes]
---

# 16 — S3 Fundamentals

> [!abstract] בשורה אחת
> S3 הוא אחסון אובייקטים אזורי עם 11 תשיעיות עמידות — ובמבחן כמעט כל שאלת S3 מסתכמת בשתי שאלות:
> *איזה storage class* ו-*מי מורשה לגשת*.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה object storage ולא disk או file share |
| 2 | איך זה עובד | Buckets, Objects, Keys, Versioning, Replication |
| 3 | פירוק מפורט | מגבלות גודל, כללי שמות, מודל ההרשאות המלא |
| 4 | עלות | **טבלת כל ה-Storage Classes** — הליבה של השיעור |
| 5 | השוואות | Standard מול IA מול Glacier, CRR מול SRR |
| 6 | Well-Architected | S3 לפי ששת ה-Pillars |
| 7 | מלכודות | "הכי זול" הוא לא תמיד התשובה |
| 8 | Scenario | data lake עם מדיניות אחסון מדורגת |

**מונחי מפתח בשיעור:** `Bucket` · `Object Key` · `Prefix` · `Storage Class` · `Versioning` · `CRR/SRR` · `Block Public Access` · `Minimum Storage Duration`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- דיסקים (EBS) קשורים ל-instance אחד, ב-AZ אחת, ובגודל שצריך להקצות מראש.
- File share (EFS) פותר שיתוף, אבל עדיין מנהלים filesystem ומשלמים יקר לכל GB.
- צריך מקום לשים **פטה-בייטים** של לוגים, גיבויים, תמונות וקבצי media — בלי לחשוב על capacity.
- חלק מהנתונים נקראים כל שנייה, וחלק פעם בשנתיים. לשלם עליהם אותו מחיר זה בזבוז.
- צריך שהנתונים ישרדו גם אם מרכז נתונים שלם נשרף.

### מה השירות פותר

- **קיבולת אינסופית לכאורה** — לא מקצים גודל, לא מרחיבים volume, לא נגמר מקום.
- **11 תשיעיות עמידות** (99.999999999%) — הנתונים מפוזרים על **לפחות 3 AZs**.
- **תשלום לפי שימוש בפועל** — GB שאוחסנו בחודש, ולא GB שהוקצו.
- **מדרג מחירים** — אותו אובייקט יכול לעלות פי עשרות מונים פחות אם הוא בארכיון.
- **אינטגרציה עם כל AWS** — CloudFront, Athena, Glue, Lambda, CloudTrail, Backup וכו'.

> [!tip] האנלוגיה
> EBS הוא **מגירה במשרד שלך** — קרובה, מהירה, שייכת רק לך, ובגודל קבוע.
> EFS הוא **ארון תיוק משותף** לכל הקומה.
> S3 הוא **מחסן ענק עם מדפים בכמה בניינים** — שמים חבילה, מקבלים מספר מדף (key),
> ומחליטים אם היא במדף הקדמי (Standard) או במרתף שלוקח 12 שעות להוציא ממנו (Deep Archive).

### על מה בונים עם S3

| Use case | למה S3 |
|---|---|
| **Backup & Storage** | עמידות גבוהה, אין תקרת גודל |
| **Disaster Recovery** | replication ל-Region אחר |
| **Archive** | Glacier classes בעלות נמוכה מאוד |
| **Hybrid Cloud Storage** | Storage Gateway חושף S3 כ-NFS/SMB/iSCSI לאתר המקומי |
| **Application / Media hosting** | קבצים סטטיים גדולים ללא שרת |
| **Data Lake & Big Data** | Athena, Redshift Spectrum, EMR, Glue קוראים ישירות מ-S3 |
| **Software delivery** | הפצת installers ועדכונים |
| **Static Website** | אחסון אתר סטטי ישירות מה-bucket |

---

## 2. ⚙️ איך זה עובד

### 2.1 Buckets — המכולה

- Bucket הוא המכולה העליונה. אובייקט תמיד יושב בתוך bucket.
- **ה-bucket נוצר ב-Region ספציפי.** S3 *נראה* גלובלי בקונסולה, אבל הנתונים יושבים ב-Region.
- **שם ה-bucket הוא ב-namespace גלובלי** — ייחודי בכל החשבונות ובכל ה-Regions בעולם.

**כללי שמות (נשאלים לפעמים ישירות):**

| כלל | דוגמה פסולה |
|---|---|
| אין אותיות גדולות | `MyBucket` |
| אין קו תחתון | `my_bucket` |
| לא בפורמט של כתובת IP | `192.168.1.1` |
| חייב להתחיל באות קטנה או ספרה | `-mybucket` |
| אסור להתחיל בקידומת `xn--` | `xn--test` |
| אסור להסתיים בסיומת `-s3alias` | `mybucket-s3alias` |
| אורך 3–63 תווים | `ab` |

> [!warning] "Bucket already exists" בחשבון אחר
> ה-namespace גלובלי. אם מישהו בעולם תפס את השם — אתם לא תקבלו אותו,
> גם אם אתם ב-Region אחר לגמרי ואין לו שום קשר אליכם.

### 2.2 Objects ו-Keys

- **Key** הוא הנתיב **המלא** של האובייקט בתוך ה-bucket.
- `s3://my-bucket/reports/2026/q1.pdf` → ה-key הוא `reports/2026/q1.pdf`.
- ה-key מורכב מ-**prefix** + **object name**:

```text
s3://my-bucket/ reports/2026/ q1.pdf
                └─── prefix ──┘ └ name ┘
```

> [!warning] אין תיקיות ב-S3
> ה-namespace הוא **שטוח (flat)**. אין directories.
> יש רק keys ארוכים שמכילים תווי `/`, והקונסולה **מציגה** אותם כתיקיות.
> משמעות מעשית: אין "לשנות שם לתיקייה" — צריך להעתיק כל אובייקט ל-key חדש ולמחוק את הישן.

**מה מרכיב אובייקט:**

| רכיב | פירוט |
|---|---|
| **Body** | התוכן עצמו. **גודל מקסימלי 50 TB** (50,000 GB) |
| **Key** | הנתיב המלא |
| **Metadata** | זוגות key/value של טקסט — system metadata או user metadata |
| **Tags** | זוגות Unicode key/value, **עד 10 לאובייקט**. שימושי ל-security ול-lifecycle |
| **Version ID** | קיים רק אם versioning מופעל ב-bucket |

> [!warning] המספר שנשאל: 5 GB
> העלאה של אובייקט **מעל 5 GB חייבת** להשתמש ב-**Multi-Part Upload**.
> זה נפרד מהתקרה של 50 TB לאובייקט. פירוט ב-[[18 - S3 Advanced Features]].

### 2.3 Storage Classes — שבע האפשרויות

```text
מהיר ויקר                                                    איטי וזול
────────────────────────────────────────────────────────────────────▶
Standard  →  Standard-IA  →  One Zone-IA  →  Glacier Instant
          →  Glacier Flexible  →  Glacier Deep Archive

Intelligent-Tiering — לא במדרג; הוא מנוע שמזיז אובייקטים בין הרמות אוטומטית
```

- אפשר לבחור class **בעת ההעלאה**, לשנות אותו **ידנית**, או לתת ל-**Lifecycle Rules** לעשות זאת אוטומטית.
- כללי המעבר והאוטומציה — [[18 - S3 Advanced Features]].

### 2.4 Versioning

- מופעל **ברמת ה-bucket**, לא ברמת האובייקט.
- כתיבה על אותו key **לא דורסת** — היא יוצרת version חדש: 1, 2, 3...
- מחיקה לא מוחקת באמת — היא מוסיפה **Delete Marker**. אפשר להסיר אותו ולשחזר.

| התנהגות | פירוט |
|---|---|
| קבצים שהיו ב-bucket **לפני** ההפעלה | מקבלים version ID של `null` |
| **Suspend** לוורסינג | **לא מוחק** גרסאות קיימות. רק מפסיק ליצור חדשות |
| מחיקת גרסה ספציפית (עם version ID) | **מחיקה אמיתית ובלתי הפיכה** |
| עלות | משלמים על **כל הגרסאות** — כל version הוא אובייקט מחויב |

> [!tip] למה זה best practice
> Versioning הוא ההגנה מפני **overwrite בטעות** ומפני **ransomware**.
> הוא גם **תנאי מוקדם חובה** ל-Replication ול-MFA Delete.

### 2.5 Replication — CRR ו-SRR

**תנאי סף: Versioning חייב להיות מופעל גם ב-source וגם ב-destination.**

| סוג | מלא | מתי |
|---|---|---|
| **CRR** | Cross-Region Replication | DR, ציות רגולטורי, קירוב נתונים למשתמשים ב-Region אחר |
| **SRR** | Same-Region Replication | ריכוז לוגים מכמה buckets, שכפול חי מ-prod ל-test |

**עובדות שנשאלות:**

- ההעתקה היא **אסינכרונית**. אין ערובה לזמן — יש SLA נפרד בשם S3 RTC לזמן מובטח.
- ה-buckets יכולים להיות **בחשבונות AWS שונים**.
- חייבים לתת ל-S3 **IAM Role** עם הרשאות לקרוא מהמקור ולכתוב ליעד.
- **רק אובייקטים חדשים משוכפלים** מרגע ההפעלה.
  לשכפול הקיימים — **S3 Batch Replication** (גם מטפל באובייקטים שנכשלו).

**התנהגות מחיקות — נקודה עדינה:**

| פעולה | האם משוכפל |
|---|---|
| מחיקה רגילה (יוצרת Delete Marker) | **אופציונלי** — יש הגדרה להפעיל שכפול של delete markers |
| מחיקה של version ID ספציפי | **לא משוכפל לעולם** — הגנה מפני מחיקה זדונית |

> [!warning] אין שרשור (chaining) ב-Replication
> אם bucket 1 → bucket 2, ו-bucket 2 → bucket 3,
> אובייקט שנוצר ב-1 **לא יגיע ל-3**. השכפול הוא צעד אחד בלבד.
> כדי להגיע ל-3 צריך כלל replication ישיר מ-1 ל-3.

### 2.6 Static Website Hosting

- אפשר להגיש אתר סטטי ישירות מה-bucket.
- כתובת: `http://bucket-name.s3-website-<region>.amazonaws.com`
  או `http://bucket-name.s3-website.<region>.amazonaws.com`.
- **קיבלתם 403 Forbidden?** ה-bucket policy לא מאפשר קריאה ציבורית.
- **הערה חשובה:** ה-endpoint הזה הוא HTTP בלבד וללא CDN.
  לפרודקשן משתמשים ב-**CloudFront + OAC** מול bucket פרטי — ראו [[15 - CloudFront and Global Delivery]].

---

## 3. 🔍 פירוק מפורט

### 3.1 מודל האבטחה — מי מחליט מי נכנס

S3 מכריע גישה משילוב של מספר מנגנונים:

| מנגנון | סוג | מה הוא עושה |
|---|---|---|
| **IAM Policy** | User-Based | אילו קריאות API מותרות ל-user/role מסוים |
| **Bucket Policy** | Resource-Based | כלל ברמת ה-bucket כולו. **מאפשר גישה cross-account** |
| **Object ACL** | Resource-Based | הרשאה עדינה לאובייקט בודד. **ניתן לכיבוי** (מומלץ) |
| **Bucket ACL** | Resource-Based | נדיר, מנגנון ישן. **ניתן לכיבוי** |
| **Block Public Access** | Guardrail | דורס הכל ומונע חשיפה ציבורית |

**כלל ההכרעה:**

```text
גישה מאושרת אם:
   ( IAM Policy מאשרת  OR  Resource Policy מאשרת )
   AND  אין שום DENY מפורש
   AND  Block Public Access לא חוסם (כשמדובר בגישה אנונימית)
```

### 3.2 סדר העדיפויות — מה גובר על מה

זו נקודה שנשאלת בניסוחים מבלבלים. הסדר:

| # | שכבה | ההשפעה |
|---|---|---|
| 1 | **Explicit DENY** (בכל policy שהיא) | **תמיד מנצח.** אין דבר שגובר עליו |
| 2 | **SCP** ב-Organizations | תקרה. אם ה-SCP לא מתיר — שום policy מתחתיו לא יעזור |
| 3 | **Block Public Access** | חוסם גישה **אנונימית/ציבורית**, גם אם ה-bucket policy מתירה במפורש |
| 4 | **Allow** מ-IAM **או** מ-Bucket Policy | מספיק שאחד מהם מתיר (באותו חשבון) |
| 5 | ברירת מחדל | **DENY** — בלי allow מפורש, אין גישה |

> [!warning] Cross-Account דורש שני צדדים
> באותו חשבון: מספיק ש-**אחד** מהשניים (IAM או Bucket Policy) מתיר.
> **בין חשבונות: צריך שניהם** — bucket policy בחשבון היעד **וגם** IAM policy בחשבון המקור.

### 3.3 Block Public Access

- נוצר אחרי גל של דליפות נתונים מ-buckets שנחשפו בטעות.
- **מופעל כברירת מחדל** ב-buckets חדשים.
- ניתן להגדיר **ברמת החשבון כולו** — ואז אף bucket בחשבון לא יכול להיחשף.
- **הכלל הפשוט:** אם ה-bucket לא אמור להיות ציבורי — משאירים דלוק. תמיד.

ארבע ההגדרות:

| הגדרה | מה חוסמת |
|---|---|
| BlockPublicAcls | יצירת ACL ציבורי חדש |
| IgnorePublicAcls | מתעלמת מ-ACLs ציבוריים קיימים |
| BlockPublicPolicy | הגדרת bucket policy ציבורית חדשה |
| RestrictPublicBuckets | מגבילה גישה דרך policy ציבורית קיימת |

### 3.4 Durability מול Availability — לא אותו דבר

| מדד | מה מודד | הערך ב-S3 |
|---|---|---|
| **Durability** | הסיכוי שהאובייקט **לא יאבד** | **99.999999999% (11 תשיעיות) — זהה בכל ה-classes** |
| **Availability** | כמה זמן השירות **זמין לגישה** | **משתנה לפי class** |

- 11 תשיעיות בפועל: אם תאחסנו **10 מיליון אובייקטים**, בממוצע תאבדו אובייקט אחד **כל 10,000 שנה**.
- Availability של 99.99% = לא זמין ~**53 דקות בשנה**.
- **One Zone-IA הוא היוצא מן הכלל המושגי:** ה-durability שלו הוא עדיין 11 תשיעיות,
  אבל **בתוך AZ אחת בלבד**. אם ה-AZ נהרסת — **הנתונים אבדו**.

### 3.5 מגבלות ומספרים לזכור

| פריט | ערך |
|---|---|
| גודל אובייקט מקסימלי | **50 TB** |
| מעל איזה גודל חובה Multi-Part Upload | **5 GB** |
| גודל מקסימלי בהעלאה יחידה (PUT) | 5 GB |
| Tags לאובייקט | **עד 10** |
| אורך שם bucket | 3–63 תווים |
| Buckets לחשבון | 100 כברירת מחדל (ניתן להגדלה עד 1,000) |
| מספר אובייקטים ב-bucket | **ללא הגבלה** |
| AZs לפיזור (רוב ה-classes) | **≥ 3** |

### 3.6 S3 Express One Zone

Storage class מיוחד לביצועים, נפרד מהמדרג הרגיל:

| מאפיין | ערך |
|---|---|
| מיקום | **AZ יחידה**, בתוך **Directory Bucket** (סוג bucket נפרד) |
| Throughput | מאות אלפי בקשות בשנייה |
| Latency | **single-digit milliseconds** |
| ביצועים מול Standard | **עד פי 10 מהר יותר** |
| עלות גישה | **נמוכה בכ-50%** לבקשה מול Standard (אך אחסון יקר יותר) |
| Durability | 99.999999999% |
| Availability | **99.95%** |
| Use cases | AI/ML training, ניתוח אינטראקטיבי, financial modeling, media processing, HPC |
| אינטגרציות | SageMaker Training, Athena, EMR, Glue |

**הרעיון המרכזי:** מקמים את ה-storage **באותה AZ** של ה-compute כדי לחסל latency.
המחיר: אין עמידות מול כשל AZ.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

זהו לב השיעור. השאלות במבחן כמעט תמיד נופלות על **minimum duration** או על **retrieval fee**.

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| **Storage** | GB-month | תלוי ב-storage class |
| **Requests** | לכל 1,000 בקשות | PUT/POST יקרים מ-GET; ב-Glacier ה-POST הכי יקר |
| **Retrieval fee** | לכל GB שנשלף | **0 ב-Standard וב-Intelligent-Tiering**. קיים בכל השאר |
| **Data Transfer Out** | GB יוצא לאינטרנט | כניסה (IN) **חינם**. יציאה בתשלום |
| **Lifecycle transitions** | לכל 1,000 אובייקטים שהועברו | מיליוני אובייקטים קטנים = מעבר יקר |
| **Replication** | אחסון ביעד + transfer בין Regions | CRR משלם גם transfer |
| **Monitoring** | ב-Intelligent-Tiering: עמלה לכל 1,000 אובייקטים | הסיבה שהוא לא מתאים לאובייקטים קטנים |
| **Minimum storage duration** | מחויבים על X ימים גם אם מחקתם קודם | המלכודת הגדולה |
| **Minimum billable object size** | אובייקט קטן מחויב כאילו הוא בגודל המינימום | 128 KB / 40 KB |

### הטבלה שנשאלת ישירות במבחן

| Storage Class | עלות אחסון יחסית | Retrieval fee | זמן שליפה | מינ' ימי אחסון | מינ' גודל מחויב | Availability | AZs | Use case |
|---|---|---|---|---|---|---|---|---|
| **Standard** | בסיס (היקר ביותר לאחסון) | **אין** | מיידי | **אין** | **אין** | 99.99% | ≥3 | גישה תכופה, big data, אתרים |
| **Intelligent-Tiering** | בין הזול ליקר, לפי הטייר בפועל + עמלת ניטור | **אין** | מיידי | **אין** | **אין** | 99.9% | ≥3 | דפוס גישה **לא ידוע/משתנה** |
| **Standard-IA** | ~½ מ-Standard | **יש** (per GB) | מיידי | **30 יום** | **128 KB** | 99.9% | ≥3 | גיבויים, DR — נדיר אך מיידי |
| **One Zone-IA** | ~⅖ מ-Standard | **יש** (per GB) | מיידי | **30 יום** | **128 KB** | **99.5%** | **1** | עותק משני, נתונים שניתן ליצור מחדש |
| **Glacier Instant Retrieval** | ~⅙ מ-Standard | **יש** (per GB) | **מילישניות** | **90 יום** | **128 KB** | 99.9% | ≥3 | ארכיון שנגיש ~פעם ברבעון **ומיד** |
| **Glacier Flexible Retrieval** | ~⅙ מ-Standard | **יש** (per GB) | **Expedited 1–5 דק' · Standard 3–5 שע' · Bulk 5–12 שע' (Bulk חינם)** | **90 יום** | **40 KB** | 99.99% | ≥3 | ארכיון שאפשר לחכות לו |
| **Glacier Deep Archive** | **הזול ביותר** (~1/23 מ-Standard) | **יש** (per GB) | **Standard 12 שע' · Bulk 48 שע'** | **180 יום** | **40 KB** | 99.99% | ≥3 | שימור 7–10 שנים, ציות רגולטורי |
| **Express One Zone** | אחסון יקר, בקשות זולות | אין | **מילישניות בודדות** | 1 שעה | 512 KB | 99.95% | **1** | compute באותה AZ, ML/HPC |

> [!info] Durability זהה בכל השורות
> **11 תשיעיות (99.999999999%) בכל ה-classes.** מה שמשתנה זה **Availability** ו-**מספר ה-AZs**.
> אל תיפלו על תשובה שטוענת ש-Glacier פחות עמיד מ-Standard.

### שלושת המספרים הקריטיים

```text
Minimum Storage Duration          Minimum Billable Size
──────────────────────            ─────────────────────
Standard             אין          Standard              אין
Intelligent-Tiering  אין          Intelligent-Tiering   אין
Standard-IA          30 יום        Standard-IA           128 KB
One Zone-IA          30 יום        One Zone-IA           128 KB
Glacier Instant      90 יום        Glacier Instant       128 KB
Glacier Flexible     90 יום        Glacier Flexible       40 KB
Glacier Deep Archive 180 יום       Glacier Deep Archive   40 KB
```

**דרך זכירה:** `30 / 30 / 90 / 90 / 180` — כלומר IA=30, Glacier=90, Deep=180.

### 🚩 עלויות נסתרות

- **Minimum storage duration** — מחקתם אובייקט מ-Glacier אחרי 10 ימים?
  **תחויבו על 90 יום מלאים.** זה מה שהופך ארכיון של נתונים קצרי-חיים ליקר יותר מ-Standard.
- **Minimum billable object size** — אובייקט של 8 KB ב-Standard-IA מחויב כאילו הוא **128 KB**.
  מיליון אובייקטים קטנים ב-IA יכולים לעלות **יותר** מאשר ב-Standard.
- **Retrieval fees** — Standard-IA שקוראים ממנו כל יום יוצא יקר מ-Standard, למרות מחיר אחסון נמוך.
- **עמלת Monitoring של Intelligent-Tiering** — נגבית לכל 1,000 אובייקטים.
  עם מיליארד אובייקטים זעירים היא מבטלת את החיסכון.
- **Versioning** — כל גרסה ישנה היא אובייקט מחויב במלואו. bucket עם versioning ובלי lifecycle מתנפח.
- **Multi-Part Uploads נטושים** — חלקים שהועלו ולא הושלמו **ממשיכים להיות מחויבים** ולא נראים ברשימה.
  ראו את כלל ה-lifecycle לניקוי ב-[[18 - S3 Advanced Features]].
- **Delete Markers** — לא תופסים גודל משמעותי אבל מייצרים אובייקטים ורעש בעלות requests.
- **Data Transfer Out** — הוצאת פטה-בייט לאינטרנט יכולה לעלות יותר מהאחסון עצמו. CloudFront מוזיל זאת.
- **Cross-Region Replication** — משלמים פעמיים על אחסון **ועוד** על transfer בין Regions.

### 💡 טיפים לחיסכון

- **Lifecycle Rules** — האוטומציה שממירה "אחרי 30 יום" ל-Standard-IA ו-"אחרי 90" ל-Glacier.
- **Intelligent-Tiering** כשאין לכם מושג מה דפוס הגישה — הוא חוסך בלי סיכון של retrieval fees.
- **אל תשימו אובייקטים קטנים ב-IA/Glacier** — המינימום המחויב הורג את החיסכון.
- **הפעילו lifecycle לניקוי Multi-Part Uploads נטושים** — חיסכון "חינמי" כמעט בכל חשבון.
- **הפעילו lifecycle גם על noncurrent versions** — למחוק גרסאות ישנות אחרי X ימים.
- **CloudFront מול ה-bucket** — מוריד גם egress וגם requests.
- **Gateway VPC Endpoint ל-S3** — תעבורה מ-EC2 ל-S3 לא עוברת NAT Gateway. ראו [[12 - VPC Private Connectivity]].
- **S3 Storage Lens / Storage Class Analysis** — מזהים מה באמת לא נקרא. ראו [[18 - S3 Advanced Features]].

---

## 5. ⚖️ השוואות מכריעות

### Standard מול Standard-IA מול One Zone-IA

| קריטריון | Standard | Standard-IA | One Zone-IA |
|---|---|---|---|
| עלות אחסון | הגבוהה ביותר | נמוכה יותר | הנמוכה מבין השלוש |
| Retrieval fee | **אין** | יש | יש |
| מינימום אחסון | אין | 30 יום | 30 יום |
| Availability | 99.99% | 99.9% | **99.5%** |
| AZs | ≥3 | ≥3 | **1** |
| שורד כשל AZ | ✅ | ✅ | ❌ **הנתונים אבדו** |
| מתי בוחרים | גישה תכופה | גיבוי שנקרא נדיר אבל צריך **מיד** | עותק **משני** שאפשר ליצור מחדש |

### שלושת ה-Glacier

| קריטריון | Glacier Instant Retrieval | Glacier Flexible Retrieval | Glacier Deep Archive |
|---|---|---|---|
| זמן שליפה | **מילישניות** | דקות עד שעות | **12–48 שעות** |
| אפשרויות שליפה | אין (מיידי) | Expedited 1–5 דק' / Standard 3–5 שע' / **Bulk 5–12 שע' — חינם** | Standard 12 שע' / Bulk 48 שע' |
| מינימום אחסון | 90 יום | 90 יום | **180 יום** |
| עלות אחסון | הגבוהה מבין השלושה | באמצע | **הזולה ביותר ב-S3 כולו** |
| מתי בוחרים | ארכיון שנגיש **פעם ברבעון ומיד** | ארכיון שאפשר לחכות לו שעות | שימור 7–10 שנים, כמעט לא נוגעים |

### CRR מול SRR

| קריטריון | CRR (Cross-Region) | SRR (Same-Region) |
|---|---|---|
| יעד | Region **אחר** | **אותו** Region |
| עלות transfer | יש (בין Regions) | אין transfer בין Regions |
| Use case עיקרי | **DR**, ציות, קירוב לטובת latency | **ריכוז לוגים**, שכפול prod→test |
| דורש versioning בשני הצדדים | ✅ | ✅ |
| חוצה חשבונות | ✅ | ✅ |

### S3 מול EBS מול EFS

| קריטריון | S3 | EBS | EFS |
|---|---|---|---|
| סוג | Object | Block | File (NFS) |
| מי מתחבר | HTTP API, כל מקום | instance אחד (ברוב המקרים) | הרבה instances במקביל |
| Scope | Region | **AZ** | Region (multi-AZ) |
| קיבולת | אינסופית לכאורה | מוקצית מראש | אלסטית |
| Mount כ-filesystem | **לא** | כן | כן |
| עלות יחסית ל-GB | **הזולה ביותר** | באמצע | היקרה ביותר |

> [!info] שורה תחתונה
> **"Lowest storage cost"** לבדו לא קובע. שאלו שלוש שאלות:
> **כמה זמן הנתונים יישארו?** (מינימום ימים) · **כמה מהר צריך אותם?** (retrieval time) ·
> **כמה פעמים נקרא אותם?** (retrieval fee). התשובה נופלת מאחת מהשלוש.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **ב-S3** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | ניהול אוטומטי של מחזור החיים והנראות | Lifecycle Rules במקום מחיקה ידנית; S3 Inventory לרשימת אובייקטים; Storage Lens לדשבורד; התראות CloudWatch על 4xx/5xx ועל גדילת נפח |
| **Security** | private by default, הצפנה תמיד, הרשאות מינימליות | Block Public Access ברמת **החשבון**; ACLs מכובים ו-bucket policy בלבד; SSE כברירת מחדל; תנאי `aws:SecureTransport` לכפיית HTTPS ([[17 - S3 Security and Data Management]]) |
| **Reliability** | הנתונים שורדים כשל AZ ו-Region, וטעות אנוש הפיכה | Versioning דלוק; CRR ל-Region שני ל-DR; **להימנע מ-One Zone-IA** לנתונים שאין להם מקור; MFA Delete לנתונים קריטיים |
| **Performance Efficiency** | הגישה מהירה ומקבילית | Multi-Part Upload לקבצים גדולים; Byte-Range Fetches לקריאה מקבילית; CloudFront לקריאה גלובלית; Express One Zone ל-compute באותה AZ |
| **Cost Optimization** | כל אובייקט יושב ב-class שמתאים לו | Lifecycle לירידה במדרג; Intelligent-Tiering כשהדפוס לא ידוע; ניקוי multipart נטוש וגרסאות ישנות; Storage Class Analysis לפני שמחליטים |
| **Sustainability** | פחות בייטים, פחות תנועה | דחיסה ופורמטים עמודתיים (Parquet) בדאטה לייק; ארכוב נתונים לא פעילים; CloudFront כדי לא לשלוח את אותו קובץ מה-origin שוב ושוב |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "unknown or changing access patterns" | **S3 Intelligent-Tiering** |
| "lowest cost, retrieval within 12 hours is acceptable" | **Glacier Deep Archive** |
| "archive but must be retrieved in milliseconds" | **Glacier Instant Retrieval** |
| "accessed once a month, needs immediate access" | **Standard-IA** |
| "can be easily recreated / secondary copy" | **One Zone-IA** |
| "retain for 7 years for compliance, lowest cost" | **Glacier Deep Archive** (+ Object Lock — [[17 - S3 Security and Data Management]]) |
| "protect against accidental deletion / overwrite" | **Versioning** |
| "replicate to another region for DR" | **CRR** (+ versioning בשני הצדדים) |
| "aggregate logs from multiple buckets in one region" | **SRR** |
| "prevent data leaks / bucket must never be public" | **Block Public Access** ברמת החשבון |
| "grant another AWS account access to the bucket" | **Bucket Policy** (resource-based) |
| "single-digit millisecond latency, same AZ as compute" | **S3 Express One Zone** |
| "file larger than 5 GB" | **Multi-Part Upload** |
| "objects must survive an entire AZ failure" | **לא** One Zone-IA |

### טעויות נפוצות

> [!warning] מלכודת 1 — "הכי זול" בלי לבדוק זמן שליפה
> **הניסוח:** "Archive data must be available for audits within one hour. Choose the lowest-cost class."
> **הטעות:** לבחור **Deep Archive** כי הוא הזול ביותר.
> **הנכון:** Deep Archive לוקח **12 שעות** מינימום. שעה אחת → **Glacier Flexible Retrieval** עם Expedited (1–5 דק'),
> או Glacier Instant Retrieval אם צריך מיידי.

> [!warning] מלכודת 2 — לשכוח את ה-Minimum Storage Duration
> **הניסוח:** "Data is kept for 20 days then deleted. Which class is cheapest?"
> **הטעות:** לבחור Standard-IA או Glacier כי מחיר ה-GB נמוך.
> **הנכון:** **S3 Standard.** ב-IA תחויבו על **30 יום** וב-Glacier על **90** — גם אם מחקתם ביום ה-20.

> [!warning] מלכודת 3 — One Zone-IA לנתונים שאין להם מקור
> **הניסוח:** "Move backups of critical customer data to the cheapest IA class."
> **הטעות:** One Zone-IA כי הוא זול יותר מ-Standard-IA.
> **הנכון:** One Zone-IA יושב ב-**AZ אחת**. כשל AZ = **אובדן מוחלט**.
> מתאים רק לעותק **משני** או לנתונים שאפשר ליצור מחדש (thumbnails, cache, transcoded media).

> [!warning] מלכודת 4 — Durability שונה בין classes
> **הניסוח:** "Which storage class provides the highest durability?"
> **הטעות:** לבחור Standard.
> **הנכון:** **כל ה-classes נותנים 11 תשיעיות durability.** מה ששונה הוא **Availability**
> ומספר ה-AZs. (One Zone-IA הוא 11 תשיעיות — אבל בתוך AZ אחת.)

> [!warning] מלכודת 5 — Replication משכפל את מה שכבר קיים
> **הניסוח:** "We enabled CRR. Why is the destination bucket still empty?"
> **הטעות:** להניח שהפעלה משכפלת רטרואקטיבית.
> **הנכון:** replication חל **רק על אובייקטים חדשים**.
> לקיימים צריך **S3 Batch Replication**.

> [!warning] מלכודת 6 — שרשור replication
> **הניסוח:** "Bucket A replicates to B, and B replicates to C. Objects in A will appear in C."
> **הטעות:** לענות "כן, זה מדורג".
> **הנכון:** **אין chaining.** אובייקט מ-A יגיע רק ל-B. צריך כלל ישיר A→C.

> [!warning] מלכודת 7 — Suspend Versioning מוחק גרסאות
> **הניסוח:** "Suspend versioning to reduce storage costs."
> **הטעות:** להניח שזה מנקה את הישן.
> **הנכון:** Suspend רק **מפסיק ליצור** גרסאות חדשות. הגרסאות הישנות **נשארות ומחויבות**.
> לניקוי צריך **Lifecycle Rule על noncurrent versions**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברת מדיה מעלה ~20 TB בחודש של קבצי וידאו גולמיים ולוגים.

- הגלם נערך בשבועיים הראשונים ואז כמעט לא נוגעים בו.
- גרסאות מעובדות (transcoded) נצרכות על ידי משתמשים בכל העולם — וניתן ליצור אותן מחדש מהגלם.
- דרישת רגולציה: לשמור את הגלם **7 שנים**.
- ביקורת פנימית מבקשת לפעמים קובץ ישן, ומוכנה לחכות **עד יום עבודה**.
- אסור שדליפה תחשוף את הגלם לאינטרנט.

**הארכיטקטורה:**

```text
                    ┌── raw-bucket (eu-west-1) ──────────────┐
  העלאת גלם   ───▶  │ Standard  (0–30 יום)                   │
  (Multi-Part)      │   ↓ lifecycle                          │
                    │ Standard-IA (30–90 יום)                │
                    │   ↓ lifecycle                          │
                    │ Glacier Flexible Retrieval (90 יום+)   │
                    │   Versioning ✅   Block Public Access ✅ │
                    └────────────┬───────────────────────────┘
                                 │ CRR (DR)
                                 ▼
                     raw-dr-bucket (us-east-1)
                     Glacier Deep Archive

  transcode job ──▶ derived-bucket  →  One Zone-IA
                          ▲
                          │ OAC (bucket פרטי)
                     CloudFront  ───▶  משתמשים בעולם
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| גלם ב-**Standard** ל-30 יום | זו תקופת העריכה הפעילה. אין retrieval fee ואין מינימום |
| מעבר ל-**Standard-IA** ביום 30 | ה-30 הוא בדיוק המינימום של Standard — המעבר חוקי וזול יותר |
| מעבר ל-**Glacier Flexible** ביום 90 | הביקורת מוכנה לחכות; Bulk retrieval **חינם** ב-5–12 שעות |
| **לא Deep Archive** לגלם הראשי | הביקורת רוצה תשובה תוך יום עבודה — 48 שעות Bulk זה גבולי מדי |
| **Deep Archive** ב-bucket ה-DR | העותק השני נועד לאסון בלבד. אף אחד לא קורא ממנו שוטף |
| **CRR** ל-Region שני | הגנה מפני אובדן Region שלם + ציות. דורש versioning בשני הצדדים |
| מעובדים ב-**One Zone-IA** | ניתן ליצור אותם מחדש מהגלם. כשל AZ = מריצים transcode שוב |
| **CloudFront + OAC** מול ה-bucket הפרטי | חוסך egress, מקצר latency גלובלי, וה-bucket נשאר לא ציבורי |
| **Versioning** על ה-raw | מגן מפני מחיקה בטעות של חומר שאי אפשר לצלם מחדש |
| **Block Public Access** ברמת החשבון | דורס כל bucket policy ציבורית שמישהו יגדיר בטעות |
| **Lifecycle לניקוי multipart נטוש** | קבצי וידאו גדולים = הרבה העלאות שנקטעות = עלות שקטה |

**למה לא Intelligent-Tiering לכל ה-bucket?**
כי דפוס הגישה כאן **ידוע ומובהק** — חם 30 יום, ואז קר.
Intelligent-Tiering נועד לחוסר ודאות; כשהדפוס ידוע, lifecycle מפורש זול יותר (אין עמלת monitoring).

**למה לא One Zone-IA לגלם?**
כי אין ממנו מקור. הוא **הנתון המקורי**. אובדן AZ = אובדן בלתי הפיך.

**מה משלים את הפתרון?**
הצפנה ומדיניות גישה ב-[[17 - S3 Security and Data Management]],
כללי ה-lifecycle וה-Event Notifications להפעלת ה-transcode ב-[[18 - S3 Advanced Features]].

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **מחירים מדויקים בדולרים** לכל GB — משתנים לפי Region. צריך לדעת את **הסדר היחסי**.
- **הגבלות quota מדויקות** על מספר buckets לחשבון — soft limits.
- **תחביר ה-JSON המלא** של bucket policy — מספיק להבין Effect / Action / Principal / Resource / Condition.
- **פרטי ה-API** של multipart upload (InitiateMultipartUpload, part numbers) — מספיק לדעת מתי משתמשים.
- **המבנה הפנימי** של איך S3 מפזר shards בין AZs.
- **S3 on Outposts** ו-**S3 Glacier Vault** הישן (המנגנון הנפרד מ-S3) — מוזכרים בקצרה בלבד.
- **ACLs לעומק** — AWS ממליצה לכבות אותם; המבחן מתמקד ב-bucket policies.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **Bucket = Region. שם ה-bucket = namespace גלובלי.** אין אותיות גדולות ואין קו תחתון.
- **אין תיקיות.** רק keys שטוחים עם `/`. prefix + object name.
- **גודל אובייקט מקסימלי 50 TB. מעל 5 GB — חובה Multi-Part Upload.**
- **עד 10 tags לאובייקט.**
- **Durability = 11 תשיעיות בכל ה-classes.** Availability הוא מה שמשתנה.
- **מינימום ימי אחסון: IA=30 · Glacier Instant/Flexible=90 · Deep Archive=180.**
- **מינימום גודל מחויב: IA + Glacier Instant = 128 KB · Flexible + Deep = 40 KB.**
- **אין retrieval fee ב-Standard וב-Intelligent-Tiering בלבד.**
- **One Zone-IA = AZ אחת.** רק לנתונים שאפשר ליצור מחדש.
- **Deep Archive: 12 שעות Standard, 48 שעות Bulk.** Flexible: 1–5 דק' / 3–5 שע' / 5–12 שע' (Bulk חינם).
- **Intelligent-Tiering:** עמלת ניטור, אין retrieval fee, מזיז אוטומטית — ל-**דפוס לא ידוע**.
- **Versioning ברמת bucket.** Suspend **לא מוחק** גרסאות. קבצים ישנים = version `null`.
- **Replication דורשת versioning בשני הצדדים**, אסינכרונית, **רק אובייקטים חדשים**, **אין chaining**.
- **מחיקה עם version ID לא משוכפלת** לעולם.
- **גישה = (IAM ALLOW או Resource ALLOW) AND אין DENY מפורש.** DENY תמיד מנצח.
- **Cross-account צריך את שני הצדדים.** Bucket policy הוא הכלי ל-cross-account.
- **Block Public Access מנצח bucket policy ציבורית.** מפעילים ברמת החשבון.
- **Express One Zone:** AZ אחת, Directory Bucket, latency של מילישניות בודדות, פי ~10 ביצועים.

---

## 11. ✅ בדיקת הבנה

1. נתונים נשמרים 20 יום בלבד ואז נמחקים. איזה storage class הכי זול בפועל?
2. ארכיון שצריך להיות זמין תוך שעה. Deep Archive או Flexible Retrieval?
3. איזה storage class נותן את ה-durability הגבוה ביותר?
4. הפעלתם CRR ובכל זאת ה-bucket ביעד ריק. מה קרה?
5. Bucket A משוכפל ל-B, ו-B משוכפל ל-C. האם אובייקט חדש ב-A יגיע ל-C?
6. bucket policy מאפשרת `s3:GetObject` לכולם, אבל הקריאה נכשלת ב-403. מה הכי סביר?
7. יש לכם 50 מיליון אובייקטים בגודל 10 KB כל אחד. האם Standard-IA יחסוך כסף?
8. מתי One Zone-IA הוא בחירה נכונה ומתי הוא פסול?
9. עשיתם Suspend ל-versioning כדי לחסוך. למה החשבון לא ירד?
10. איזו קבוצה נדרשת כדי לתת לחשבון AWS אחר גישה ל-bucket?

<details>
<summary>תשובות</summary>

1. **S3 Standard.** ל-Standard-IA יש מינימום של **30 יום** ול-Glacier **90** — תחויבו על התקופה המלאה
   גם אחרי מחיקה ביום ה-20. מחיר GB נמוך לא מנצח מינימום חיוב.
2. **Glacier Flexible Retrieval.** Deep Archive מתחיל ב-**12 שעות** (Standard) ו-48 (Bulk).
   Flexible נותן Expedited של **1–5 דקות** ו-Standard של 3–5 שעות.
3. **כולם זהים — 99.999999999% (11 תשיעיות).** ההבדל הוא ב-**Availability** ובמספר ה-AZs.
4. Replication חלה **רק על אובייקטים חדשים** מרגע ההפעלה. לשכפול הקיימים צריך **S3 Batch Replication**.
   (ואם גם חדשים לא מגיעים — לבדוק ש-versioning דלוק בשני ה-buckets ושל-S3 יש IAM Role מתאים.)
5. **לא.** אין **chaining** ב-S3 Replication. השכפול הוא צעד אחד. צריך כלל ישיר A→C.
6. **Block Public Access** מופעל ודורס את ה-policy. הוא חוסם גישה אנונימית
   גם כשה-bucket policy מתירה אותה במפורש. (אפשרות שנייה: DENY מפורש ב-policy או ב-SCP.)
7. **כנראה לא — ואולי אף תשלמו יותר.** ב-Standard-IA כל אובייקט מחויב במינימום **128 KB**.
   אובייקט של 10 KB מחויב כאילו הוא 128 KB, כלומר פי ~12.8 מהנפח האמיתי.
8. **נכון:** עותק **משני** של גיבוי on-prem, thumbnails, קבצים מעובדים שאפשר ליצור מחדש מהמקור.
   **פסול:** כל נתון שהוא המקור היחיד — אובדן ה-AZ הוא אובדן בלתי הפיך.
9. כי **Suspend לא מוחק** גרסאות קיימות; הוא רק מפסיק ליצור חדשות.
   צריך **Lifecycle Rule** שמוחקת noncurrent versions אחרי X ימים.
10. **Bucket Policy** בחשבון שמחזיק ב-bucket (resource-based, מאפשר cross-account),
    **וגם** IAM Policy אצל ה-principal בחשבון האחר. ב-cross-account צריך את **שני** הצדדים.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[17 - S3 Security and Data Management]] · [[18 - S3 Advanced Features]] · [[15 - CloudFront and Global Delivery]] · [[19 - EBS and EC2 Storage]] · [[20 - EFS and File Storage]] · [[35 - Backup and Data Protection]] · [[37 - Cost Optimization]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 4698–5157
