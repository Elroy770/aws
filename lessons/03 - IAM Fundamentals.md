---
lesson: 03
title: IAM Fundamentals
domain: Design Secure Architectures
services: [IAM, STS, AWS CLI, AWS SDK]
tags: [saa-c03, security, identity, least-privilege]
---

# 03 — IAM Fundamentals

> [!abstract] בשורה אחת
> IAM עונה על שתי שאלות בלבד — מי אתה (authentication) ומה מותר לך (authorization) — וכמעט כל שאלת אבטחה במבחן היא וריאציה של "Role במקום Access Key".

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה root user משותף הוא אסון מתגלגל |
| 2 | איך זה עובד | Users, Groups, Policies, Roles וירושת הרשאות |
| 3 | פירוק מפורט | מבנה ה-JSON, Password Policy, MFA, Access Keys, כלי ביקורת |
| 4 | עלות | למה IAM חינם — ואיפה בכל זאת נשרף כסף |
| 5 | השוואות | User מול Role, Access Key מול Role, Credentials Report מול Access Advisor |
| 6 | Pillars | ששת העמודים בעדשת זהויות |
| 7 | מלכודות | הניסוחים שמסגירים "תשתמש ב-Role" |
| 8 | Scenario | הרשאות לאפליקציה תלת-שכבתית |
| 9–11 | סיכום ובדיקה | Cheat Sheet ושאלות |

**מונחי מפתח בשיעור:** `Root User` · `IAM User` · `Group` · `Policy` · `Role` · `Least Privilege` · `MFA` · `Access Key` · `Credentials Report` · `Access Advisor`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- יש חשבון AWS אחד, וחמישה מפתחים שצריכים גישה אליו.
- הפתרון "הקל": כולם מתחברים עם ה-root user, והסיסמה יושבת בקבוצת WhatsApp.
- מפתח עוזב את החברה — ואי אפשר לבטל לו גישה בלי להחליף סיסמה לכולם.
- מישהו מוחק בטעות production, ואין דרך לדעת מי זה היה.
- אפליקציה על EC2 צריכה לקרוא מ-S3, אז מדביקים Access Key בתוך הקוד.
- הקוד עולה ל-GitHub. המפתח נסרק תוך דקות, ומישהו מריץ כרייה על החשבון שלך.

### מה השירות פותר

- **זהות נפרדת לכל אדם** — ביטול של אחד לא נוגע באחרים.
- **הרשאות מדויקות** — כל אחד מקבל רק את מה שהוא צריך (least privilege).
- **עקיבות** — כל פעולה נרשמת עם הזהות שביצעה אותה.
- **Credentials זמניים למכונות** — Role במקום מפתח קבוע, כך שאין מה לגנוב מהקוד.
- **שכבת אימות שנייה** — MFA הופך סיסמה גנובה לחסרת ערך.

> [!tip] האנלוגיה
> Root user = המפתח הראשי של הבניין. פותח הכול, כולל חדר החשמל.
> IAM User = כרטיס מגנטי אישי. IAM Role = כרטיס אורח זמני שפג תוקפו לבד.
> אתה לא נותן לשליח את המפתח הראשי — אתה נותן לו כרטיס לחדר אחד, לשעה.

---

## 2. ⚙️ איך זה עובד

### 2.1 שתי השאלות של IAM

```text
בקשה ל-AWS API
      │
      ▼
┌─────────────────────┐
│ Authentication      │  מי אתה?
│ סיסמה / Access Key  │  → אם נכשל: אין גישה
│ / temporary token   │
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ Authorization       │  מה מותר לך?
│ הערכת כל ה-policies │  → deny by default
└──────────┬──────────┘
           ▼
     Allow / Deny
```

- **ברירת המחדל היא Deny.** אם שום policy לא נתנה Allow מפורש — הפעולה נדחית.
- **Explicit Deny תמיד גובר** על כל Allow. אין מה שיבטל אותו.
- לוגיקת ההערכה המלאה (SCP, permission boundaries, resource policies) נלמדת ב-[[04 - IAM Advanced and Organizations]].

### 2.2 ארבעת אבני הבניין

| רכיב | מה זה | כלל שחייבים לזכור |
|---|---|---|
| **User** | מייצג אדם פיזי או אפליקציה חיצונית | אדם פיזי אחד = user אחד. לא לשתף |
| **Group** | אוסף של users לצורך הרשאות | **מכיל users בלבד — לא groups אחרים ולא roles** |
| **Policy** | מסמך JSON שמגדיר הרשאות | ניתן לצרף ל-user, ל-group או ל-role |
| **Role** | זהות שאפשר "ללבוש" זמנית | מיועדת לשירותים ולגישה חוצת-חשבונות; אין לה סיסמה |

- IAM הוא שירות **גלובלי** — אין לו Region, וההגדרות תקפות בכל החשבון.
- User לא חייב להשתייך לאף group, ויכול להשתייך לכמה groups במקביל.
- אין היררכיה של groups בתוך groups. זה הרבה יותר שטוח ממה שנראה.

### 2.3 ירושת הרשאות

```text
        ┌──────────────┐
Alice ──┤ Developers   │── policy: EC2 + CloudWatch read
        └──────────────┘
        ┌──────────────┐
Bob   ──┤ Developers   │
        │ Audit Team   │── policy: read-only על הכול
        └──────────────┘
   Bob מקבל את איחוד ההרשאות של שתי הקבוצות

Charles ── (לא בשום group) ── inline policy ישירות עליו
```

- ההרשאות האפקטיביות של user = **איחוד** כל ה-policies: מכל ה-groups שלו + מה שצמוד לו ישירות.
- Policy שמוצמד ישירות ל-user נקרא **inline policy** — נוח לחריגים, קשה לתחזוקה בקנה מידה.
- Best practice: הרשאות על **groups**, לא על users בודדים.

### 2.4 Root User — הזהות המיוחדת

- נוצר אוטומטית עם החשבון, ומזוהה בכתובת המייל של החשבון.
- יש לו הרשאות בלתי מוגבלות שאי אפשר לצמצם ב-IAM policy רגילה.
- **לא להשתמש בו** מעבר להקמת החשבון הראשונית ולמשימות שרק הוא יכול לבצע.
- **לא לשתף** אותו ולא ליצור עבורו Access Keys.
- לאבטח אותו ב-MFA ביום הראשון. זו הפעולה הראשונה בכל חשבון חדש.

---

## 3. 🔍 פירוק מפורט

### 3.1 מבנה ה-Policy — כל שדה ומה הוא עושה

| שדה | חובה? | מה זה | ערך אופייני |
|---|---|---|---|
| `Version` | כן | גרסת שפת ה-policy | תמיד `"2012-10-17"` |
| `Id` | לא | מזהה ל-policy כולה | טקסט חופשי |
| `Statement` | **כן** | מערך של הצהרה אחת או יותר | הלב של המסמך |
| `Sid` | לא | מזהה להצהרה בודדת | שם תיאורי |
| `Effect` | כן | `Allow` או `Deny` | ברירת מחדל אין — חייב לציין |
| `Principal` | תלוי | מי ה-policy חלה עליו | נדרש ב-resource-based policy ולא ב-identity policy |
| `Action` | כן | אילו פעולות API | `s3:GetObject`, `ec2:Describe*` |
| `Resource` | כן | על אילו משאבים (ARN) | `arn:aws:s3:::my-bucket/*` |
| `Condition` | לא | מתי ה-policy תקפה | הגבלת IP, MFA, tag |

מבנה עקרוני:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyOnOneBucket",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::reports-bucket",
        "arn:aws:s3:::reports-bucket/*"
      ]
    }
  ]
}
```

> [!warning] שתי טעויות ARN קלאסיות ב-S3
> - פעולות על **אובייקטים** (`GetObject`) דורשות ARN עם `/*` בסוף.
> - פעולה על ה-**bucket** עצמו (`ListBucket`) דורשת ARN **בלי** `/*`.
> חסר אחד מהם — וההרשאה נראית נכונה אבל לא עובדת.

- `Resource: "*"` פירושו "כל משאב" — הפוך מ-least privilege, ולרוב התשובה הלא נכונה במבחן.
- אפשר להשתמש ב-wildcard בתוך action: `ec2:Describe*` מכסה את כל פעולות התיאור.

### 3.2 Password Policy

מה אפשר לאכוף ברמת החשבון:

| הגדרה | מה היא עושה |
|---|---|
| אורך מינימלי | קובע רצפה לאורך הסיסמה |
| דרישת תווים | אותיות גדולות, קטנות, ספרות, תווים מיוחדים |
| שינוי עצמי | מאפשר ל-IAM users לשנות את הסיסמה שלהם |
| תפוגה (expiration) | מחייב החלפת סיסמה אחרי X ימים |
| מניעת שימוש חוזר | חוסם חזרה לסיסמאות קודמות |

- Password Policy חלה על **IAM users** — לא על ה-root user.
- זו הגדרה אחת ברמת החשבון, לא per-user.

### 3.3 MFA — Multi-Factor Authentication

- הרעיון: **משהו שאתה יודע** (סיסמה) + **משהו שיש לך** (מכשיר).
- התועלת המרכזית: סיסמה שנגנבה לבדה **אינה מספיקה** כדי להיכנס.
- חובה על ה-root user. מומלץ בחום על כל user עם הרשאות משמעותיות.

| סוג מכשיר | דוגמאות | מאפיין בולט |
|---|---|---|
| **Virtual MFA device** | Google Authenticator, Authy | אפליקציה בטלפון; Authy תומכת בכמה tokens במכשיר אחד |
| **U2F Security Key** | YubiKey | מפתח פיזי; מפתח אחד יכול לשרת כמה root ו-IAM users |
| **Hardware Key Fob** | התקן של Gemalto | מכשיר ייעודי שמציג קוד |
| **Key Fob ל-GovCloud** | התקן של SurePassID | גרסה ייעודית ל-AWS GovCloud (US) |

### 3.4 שלוש הדרכים לגשת ל-AWS

| דרך גישה | מיועד ל... | מוגן ב... |
|---|---|---|
| **Management Console** | עבודה אינטראקטיבית של אדם | סיסמה + MFA |
| **CLI** | סקריפטים ואוטומציה משורת הפקודה | Access Keys |
| **SDK** | קוד אפליקציה בשפת תכנות | Access Keys (או Role) |

- ה-CLI היא כלי קוד פתוח שנותן גישה ישירה ל-API הציבורי של כל שירות.
- ה-SDK הוא ספרייה מוטמעת באפליקציה, קיים ל-JavaScript, Python, Java, .NET, Go, Node.js ועוד,
  וגם בגרסאות Mobile ו-IoT.
- עובדה מעניינת: ה-CLI עצמו בנוי מעל ה-SDK של Python.

### 3.5 Access Keys

- זוג ערכים: **Access Key ID** (מקביל לשם משתמש) ו-**Secret Access Key** (מקביל לסיסמה).
- נוצרים דרך הקונסולה; כל user מנהל את המפתחות של עצמו.
- ה-Secret מוצג **פעם אחת בלבד** בעת היצירה.
- אלו credentials **קבועים** — הם לא פגים לבד. זו בדיוק הבעיה איתם.
- לעולם לא לשתף, לא להעלות ל-repo, ולא להטמיע ב-AMI.

### 3.6 IAM Roles for Services

- שירותי AWS צריכים לפעמים לבצע פעולות **בשמך** — לכן נותנים להם Role.
- ה-Role מספק **credentials זמניים** שמתחדשים אוטומטית ופגים לבד.
- אין מה לגנוב: אין מפתח קבוע בקוד, אין קובץ קונפיגורציה עם סוד.

Roles נפוצים:

| Role | לשם מה |
|---|---|
| EC2 Instance Role | אפליקציה על EC2 שניגשת ל-S3, DynamoDB, SSM |
| Lambda Execution Role | פונקציה שכותבת ללוגים, קוראת מ-SQS, כותבת ל-DynamoDB |
| CloudFormation Role | ה-stack יוצר משאבים בשמך |

```text
EC2 Instance ──(מצורף)──► IAM Role ──► STS מנפיק credentials זמניים
      │                                        │
      └──── קורא ל-s3:GetObject ◄──────────────┘
             בלי שום מפתח מאוחסן על הדיסק
```

### 3.7 שני כלי הביקורת של IAM

| כלי | Scope | מה הוא מראה | מתי משתמשים |
|---|---|---|---|
| **IAM Credentials Report** | **חשבון** | טבלה של כל ה-users וסטטוס ה-credentials שלהם: סיסמה, MFA, מפתחות, מתי שימשו לאחרונה | ביקורת תקופתית, איתור מפתחות ישנים ו-users רדומים |
| **IAM Access Advisor** | **user בודד** | אילו הרשאות שירות ניתנו ל-user, ומתי כל שירות נגיש בפועל לאחרונה | צמצום הרשאות שלא בשימוש (least privilege אמיתי) |

> [!tip] איך לא להתבלבל
> **Credentials** = מסמכי זהות → דוח ברמת **החשבון** על כל המשתמשים.
> **Advisor** = יועץ אישי → מסתכל על **user אחד** ואומר לך מה לקצץ.

### 3.8 Best Practices — הרשימה שהמבחן אוהב

- לא להשתמש ב-root מעבר להקמת החשבון.
- אדם פיזי אחד = IAM user אחד.
- לצרף users ל-groups, ולהעניק הרשאות ל-groups.
- להגדיר Password Policy חזקה.
- לאכוף MFA, במיוחד על root ועל הרשאות רגישות.
- להשתמש ב-Roles כדי לתת הרשאות לשירותי AWS.
- להשתמש ב-Access Keys רק לגישה תוכניתית שאין לה חלופת Role.
- לבצע ביקורת קבועה עם Credentials Report ו-Access Advisor.
- לעולם לא לשתף users או Access Keys.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב | מודל חיוב | הערה |
|---|---|---|
| IAM — users, groups, roles, policies | **ללא עלות** | אין חיוב על מספר הזהויות |
| STS — הנפקת credentials זמניים | **ללא עלות** | הבסיס לכל Role |
| Credentials Report / Access Advisor | **ללא עלות** | אין סיבה לא להריץ |
| הפעולות שהזהות מבצעת | לפי השירות | כאן העלות האמיתית |
| CloudTrail (management events) | trail בסיסי ללא עלות | trails נוספים ואחסון ב-S3 מחויבים |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Role עם credentials זמניים | 0 | תמיד — גם הכי זול וגם הכי בטוח |
| Access Key + rotation ידני | 0 בחיוב, יקר בתפעול | רק כשאין חלופת Role |
| Secrets Manager לניהול סודות | חיוב לפי סוד ולפי קריאות | כשחייבים לאחסן סוד אמיתי |
| Admin לכולם "כדי לחסוך זמן" | הכי יקר בטווח הארוך | לעולם לא |

### 🚩 עלויות נסתרות

- **Access Key שדלף** = החשבון מנוצל לכרייה. זו העלות הגדולה ביותר שיש בשיעור הזה.
- **הרשאות רחבות מדי** מאפשרות ליצור בטעות משאבים יקרים (instances גדולים, Multi-Region).
- **User רדום עם מפתחות פעילים** — לא עולה כלום עד שהוא נפרץ.
- **חקירת אירוע אבטחה** צורכת שבועות של עבודה יקרה, ולעיתים דיווח רגולטורי.

### 💡 טיפים לחיסכון

- להריץ Credentials Report אחת לרבעון ולמחוק users ומפתחות שלא בשימוש.
- להשתמש ב-Access Advisor כדי לקצץ הרשאות שאף פעם לא נוצלו.
- להגביל דרך IAM מי יכול להריץ instance types יקרים או לפרוס ב-Regions לא מאושרים.
- להחליף כל Access Key שיושב בקוד ב-Role — זו גם החלטת אבטחה וגם חיסכון תפעולי.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 User מול Group מול Role

| קריטריון | IAM User | IAM Group | IAM Role |
|---|---|---|---|
| מייצג | אדם או אפליקציה חיצונית | אוסף לוגי בלבד | זהות זמנית שנלבשת |
| יש סיסמה | כן (לקונסולה) | לא — אינו זהות | לא |
| יש Access Keys | אפשרי | לא | לא — credentials זמניים בלבד |
| יכול להכיל | — | users בלבד | — |
| מתאים ל-EC2/Lambda | **לא** | לא | **כן** |
| מתאים לגישה חוצת-חשבונות | לא | לא | **כן** |

### 5.2 Access Key מול Role — ההשוואה שמכריעה שאלות

| קריטריון | Access Key | IAM Role |
|---|---|---|
| תוקף | קבוע עד שמבטלים ידנית | זמני, מתחדש אוטומטית |
| איפה מאוחסן | קובץ, משתנה סביבה, או (גרוע) בקוד | לא מאוחסן — נשלף בזמן ריצה |
| מה קורה אם דלף | תוקף מלא עד לביטול ידני | פג תוך שעות; תקיפה מוגבלת |
| Rotation | ידני, ולרוב לא נעשה | אוטומטי, שקוף |
| מתאים ל-workload על AWS | לא | **כן** |
| מתאים ל-CLI מלפטופ אישי | כן (או SSO) | דרך assume-role |

### 5.3 Credentials Report מול Access Advisor

| קריטריון | Credentials Report | Access Advisor |
|---|---|---|
| Scope | כל החשבון | user בודד |
| שאלה שהוא עונה עליה | "למי יש credentials ומה מצבם?" | "אילו הרשאות בכלל בשימוש?" |
| פורמט | דוח CSV שניתן להורדה | תצוגה בקונסולה |
| שימוש טיפוסי | היגיינה: MFA חסר, מפתחות ישנים | קיצוץ הרשאות לפי שימוש בפועל |

> [!info] שורה תחתונה
> אם ה-workload רץ **בתוך** AWS — התשובה כמעט תמיד Role.
> Access Key שמור לגישה תוכניתית מחוץ ל-AWS, וגם אז עדיף SSO אם אפשר.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בניהול זהויות** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | הרשאות שמנוהלות ידנית לכל user נשברות בקנה מידה | להעניק הרשאות ל-groups ולנהל policies כקוד |
| Security | זה ה-pillar המרכזי כאן: least privilege ו-MFA | לאכוף MFA, לבטל מפתחות root, לקצץ לפי Access Advisor |
| Reliability | הרשאה חסרה מפילה recovery בדיוק בשעת חירום | לוודא שה-roles של ASG, backup ו-failover קיימים ונבדקו |
| Performance Efficiency | Role מאפשר לאפליקציה לפנות ישירות לשירות | להימנע מ-proxy ביניים שנועד רק להחזיק מפתחות |
| Cost Optimization | הרשאה רחבה מדי מאפשרת ליצור משאבים יקרים | להגביל instance types ו-Regions דרך Condition ב-policy |
| Sustainability | הרשאות רדומות מייצרות משאבים שנשארים דולקים | ביקורת תקופתית ומחיקת users ומשאבים שאינם בשימוש |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "EC2 instance needs to access S3" | IAM Role מצורף ל-instance |
| "without storing credentials" / "no hardcoded keys" | IAM Role + temporary credentials |
| "grant the minimum permissions required" | Least privilege — policy ממוקדת ל-Action ול-Resource |
| "which users have not rotated their keys" | IAM Credentials Report |
| "identify permissions that are never used" | IAM Access Advisor |
| "protect the root account" | MFA + לא ליצור לו Access Keys |
| "enforce password complexity" | IAM Password Policy |
| "require MFA to perform this action" | `Condition` עם `aws:MultiFactorAuthPresent` |
| "one user must be able to access another account" | IAM Role עם cross-account trust ([[04 - IAM Advanced and Organizations]]) |

### טעויות נפוצות

> [!warning] מלכודת 1 — Access Key על EC2
> **הניסוח:** "Store the access keys in a config file on the EC2 instance so the app can reach S3."
> **הטעות:** לחשוב שזה פתרון לגיטימי כי "הקובץ מוגן".
> **הנכון:** IAM Role מצורף ל-instance. אין מפתח על הדיסק, וה-credentials פגים לבד.

> [!warning] מלכודת 2 — Group בתוך Group
> **הניסוח:** "Nest the Developers group inside the Engineering group to inherit permissions."
> **הטעות:** להניח היררכיה שלא קיימת.
> **הנכון:** Group מכיל **users בלבד**. אין קינון. משייכים user לכמה groups במקביל.

> [!warning] מלכודת 3 — Role ל-Group
> **הניסוח:** "Attach the IAM Role to the group so all developers get it."
> **הטעות:** לערבב בין policy ל-role.
> **הנכון:** ל-group מצרפים **policy**. Role נועד לשירותים או ל-assume, לא להצמדה ל-group.

> [!warning] מלכודת 4 — `Resource: "*"` כפתרון
> **הניסוח:** "Grant s3:* on Resource * so the application works."
> **הטעות:** לפתור בעיית הרשאות בהרחבה במקום בדיוק.
> **הנכון:** לציין את ה-Action המדויק ואת ARN המדויק. זו כמעט תמיד התשובה הנכונה במבחן.

> [!warning] מלכודת 5 — Password Policy על root
> **הניסוח:** "Set an account password policy to force the root user to rotate its password."
> **הטעות:** להניח ש-Password Policy חלה על כולם.
> **הנכון:** היא חלה על **IAM users** בלבד. את root מגנים ב-MFA ובאי-שימוש.

> [!warning] מלכודת 6 — "IAM הוא regional"
> **הניסוח:** "Create IAM users in each Region where the application runs."
> **הטעות:** להתייחס ל-IAM כשירות אזורי.
> **הנכון:** IAM הוא **גלובלי**. אותו user ואותה policy תקפים בכל ה-Regions.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברה עם 12 עובדים מקימה סביבת AWS ראשונה.

- 6 מפתחים צריכים להריץ ולנטר EC2, אך לא לגעת בהגדרות חיוב.
- 2 אנשי DevOps צריכים גישה רחבה יותר, כולל IAM.
- 4 אנשי כספים צריכים לראות דוחות עלות בלבד.
- אפליקציה על EC2 מעלה קבצים ל-S3 וכותבת רשומות ל-DynamoDB.
- דרישת ביקורת: להוכיח פעם ברבעון שאין הרשאות מיותרות.

```text
                 חשבון AWS
                      │
   ┌──────────────────┼──────────────────┐
   ▼                  ▼                  ▼
Group: Developers  Group: DevOps    Group: Finance
 6 users            2 users          4 users
 policy:            policy:          policy:
 EC2 + CloudWatch   הרחב + IAM       Billing read-only
      │                  │                │
      └────────── MFA נדרש לכולם ─────────┘

Root User ──► MFA פיזי בכספת, אין Access Keys, לא בשימוש

EC2 Instance ──► IAM Role "AppRole"
                  ├── s3:PutObject על arn:aws:s3:::app-uploads/*
                  └── dynamodb:PutItem על הטבלה הספציפית
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| שלוש groups לפי תפקיד | הרשאות מנוהלות במקום אחד; עובד חדש מצטרף ל-group ומוכן |
| user נפרד לכל אדם | ביטול גישה של עובד עוזב לא משפיע על אחרים; audit trail מזוהה |
| MFA לכל user | סיסמה גנובה לא מספיקה |
| Password Policy ברמת החשבון | אכיפה אחידה בלי להסתמך על משמעת אישית |
| Role ל-EC2, לא Access Key | אין סוד על הדיסק; credentials מתחדשים לבד |
| policy צרה ל-Role | רק PutObject על bucket אחד ורק PutItem על טבלה אחת |
| root נעול עם MFA וללא מפתחות | מצמצם את הסיכון הגדול ביותר בחשבון |
| Credentials Report רבעוני | עונה ישירות על דרישת הביקורת |
| Access Advisor לפני חידוש הרשאות | מוריד הרשאות שמעולם לא נוצלו |

**למה לא לתת ל-Developers גם IAM?**

- זו הרחבת blast radius: מי שיכול לערוך policies יכול להעניק לעצמו הכול.
- הפרדת התפקיד הזו ל-DevOps היא יישום ישיר של least privilege.

**למה לא user משותף אחד ל-Finance?**

- מאבדים עקיבות — לא תדע מי צפה במה.
- ביטול גישה לאדם אחד מחייב שינוי לכולם.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- לשנן את כל שמות ה-Actions של כל שירות. להבין את המבנה `service:Action` מספיק.
- את התחביר המדויק של כל Condition key אפשרי.
- את הפורמט המלא של קובץ ה-CSV של Credentials Report.
- מפרטים טכניים של מכשירי MFA (דגמים, יצרנים, תקנים).
- את רשימת כל שפות ה-SDK. מספיק לדעת ש-SDK הוא ספרייה בשפת התכנות שלך.
- IAM Identity Center, SCP, permission boundaries ולוגיקת הערכה מלאה — נלמדים ב-[[04 - IAM Advanced and Organizations]].

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- IAM = שירות **גלובלי**. אין לו Region.
- **Group מכיל users בלבד.** אין groups בתוך groups, ואין roles בתוך groups.
- User יכול להשתייך לכמה groups, או לאף אחד.
- הרשאות אפקטיביות = **איחוד** כל ה-policies שחלות על ה-user.
- ברירת מחדל = **Deny**. **Explicit Deny גובר תמיד.**
- Policy JSON: `Version` (תמיד `2012-10-17`), `Statement`, ובכל statement — `Effect`, `Action`, `Resource`, ואופציונלית `Sid`, `Principal`, `Condition`.
- `Principal` מופיע ב-resource-based policy, לא ב-identity policy.
- ARN של אובייקטים ב-S3 מסתיים ב-`/*`; פעולה על ה-bucket עצמו — בלי.
- שלוש דרכי גישה: Console (סיסמה + MFA), CLI (Access Keys), SDK (Access Keys/Role).
- Access Key ID ≈ שם משתמש; Secret Access Key ≈ סיסמה. ה-Secret מוצג פעם אחת.
- **Role ל-EC2/Lambda/CloudFormation** — credentials זמניים, בלי מפתחות בקוד.
- MFA = משהו שאתה יודע + משהו שיש לך. סוגים: Virtual, U2F (YubiKey), Hardware Key Fob.
- **Credentials Report = רמת חשבון.** **Access Advisor = רמת user.**
- Password Policy חלה על IAM users, לא על root.
- root: MFA, בלי Access Keys, לא לשימוש יומיומי.

---

## 11. ✅ בדיקת הבנה

1. אפליקציה על EC2 צריכה לקרוא מ-S3. Role או Access Key — ולמה?
2. האם אפשר לשים group אחד בתוך group אחר?
3. מה ההבדל בין Credentials Report ל-Access Advisor?
4. Policy אחת נותנת `Allow` על `s3:*` ואחרת נותנת `Deny` על `s3:DeleteObject`. מה התוצאה?
5. מה חסר ב-policy הבאה כדי לאפשר `ListBucket`: `Resource: "arn:aws:s3:::data/*"`?
6. למה MFA לא מונע פריצה אבל כן מונע נזק מסיסמה גנובה?
7. עובד עוזב את החברה. מה הפעולה הראשונה?

<details>
<summary>תשובות</summary>

1. **Role.** ה-credentials זמניים ומתחדשים אוטומטית, אין סוד שמאוחסן על ה-instance, ואם מישהו משיג אותם — הם פגים תוך שעות. Access Key בקוד הוא credential קבוע שדולף בקלות ונשאר תקף עד ביטול ידני.

2. **לא.** IAM Group מכיל users בלבד. אין קינון. אם צריך "ירושה", משייכים את ה-user לשתי הקבוצות.

3. **Credentials Report** הוא דוח **ברמת החשבון** על כל המשתמשים וסטטוס ה-credentials שלהם (סיסמה, MFA, מפתחות ומועד שימוש אחרון). **Access Advisor** הוא תצוגה **ברמת user בודד** שמראה אילו שירותים הוא באמת ניגש אליהם — כלי לקיצוץ הרשאות.

4. **Deny.** Explicit Deny תמיד גובר על כל Allow, בלי קשר לסדר או למקור ה-policy.

5. חסר ה-ARN של ה-**bucket עצמו**, בלי `/*`. `ListBucket` היא פעולה ברמת ה-bucket, ולכן צריך גם `arn:aws:s3:::data` וגם `arn:aws:s3:::data/*` לפעולות על אובייקטים.

6. MFA לא מונע מהתוקף לנחש או לגנוב את הסיסמה. הוא מונע ממנו **להשתמש** בה, כי חסר לו הגורם השני — המכשיר הפיזי שברשותך. זו בדיוק התועלת שהשקפים מדגישים.

7. **לבטל את ה-IAM user שלו** (או להשבית אותו) ולבטל את ה-Access Keys שלו. כאן בדיוק מתגלה הערך של user נפרד לכל אדם: אין צורך לגעת באף אחד אחר.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[01 - AWS Fundamentals]] · [[04 - IAM Advanced and Organizations]] · [[05 - EC2 Fundamentals]] · [[31 - Monitoring and Logging]] · [[32 - Security Services]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 328–592
