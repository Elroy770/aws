---
lesson: 04
title: IAM Advanced and Organizations
domain: Design Secure Architectures
services: [AWS Organizations, SCP, IAM Identity Center, AWS Directory Service, AWS Control Tower, STS]
tags: [saa-c03, security, governance, multi-account, identity]
---

# 04 — IAM Advanced and Organizations

> [!abstract] בשורה אחת
> כשעוברים מחשבון אחד לארגון שלם, IAM כבר לא מספיק — צריך גם **תקרה** שאף admin מקומי לא יכול לפרוץ (SCP), **גבול** לזהות בודדת (Permission Boundary), ו**כניסה אחת** לכל החשבונות (Identity Center).

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה חשבון אחד גדול נשבר, ומה multi-account פותר |
| 2 | איך זה עובד | מבנה Organizations, OUs, ואיך SCP יורש במורד העץ |
| 3 | פירוק מפורט | SCP, Tag Policies, Conditions, Boundaries, Identity Center, Directory Service, Control Tower |
| 4 | עלות | Consolidated Billing, volume discounts, ומה כן עולה כסף |
| 5 | השוואות | SCP מול Boundary מול Resource Policy; Role מול Resource Policy; שלושת ה-Directory Services |
| 6 | Pillars | ששת העמודים בעדשת ממשל רב-חשבוני |
| 7 | מלכודות | הניסוחים שמסגירים SCP מול Boundary מול Identity Center |
| 8 | Scenario | בניית landing zone לחברה עם 4 סביבות |
| 9–11 | סיכום ובדיקה | Cheat Sheet ושאלות |

**מונחי מפתח בשיעור:** `Management Account` · `Member Account` · `OU` · `SCP` · `Permission Boundary` · `Resource-Based Policy` · `aws:PrincipalOrgID` · `Permission Set` · `Guardrail` · `ABAC`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- החברה גדלה: dev, test, prod, וצוות אבטחה — הכול בחשבון AWS אחד.
- מפתח מריץ סקריפט ב-dev ומוחק בטעות משאב של prod. אין הפרדה פיזית.
- אי אפשר לענות על "כמה עלתה סביבת הפיתוח החודש?" — הכול חשבונית אחת מעורבבת.
- למישהו יש הרשאת admin. מבחינת IAM אין דרך למנוע ממנו לפרוס ב-Region אסור.
- כל עובד מנהל 5 סטים של credentials — אחד לכל חשבון. אנשים מתחילים לשתף.
- Onboarding של חשבון חדש = יומיים של הגדרות ידניות, ואף אחד לא זוכר את כולן.
- הרגולטור שואל "איפה ה-audit trail של כל החשבונות?" — והוא מפוזר בחמישה מקומות.

### מה השירות פותר

- **בידוד קשיח:** חשבון הוא גבול הרשאות טבעי. מה שקורה ב-dev נשאר ב-dev.
- **Guardrails שלא ניתן לעקוף:** SCP מגביל **גם** את ה-admin של החשבון החבר.
- **חשבונית מאוחדת:** תשלום אחד, שקיפות לפי חשבון, והנחות כמות מצטברות.
- **כניסה אחת:** Identity Center נותן SSO לכל החשבונות והאפליקציות.
- **הקמה אוטומטית:** Control Tower מקים סביבה מרובת חשבונות לפי best practices.

> [!tip] האנלוגיה
> IAM Policy = הרשאות שנתת לעובד.
> SCP = חוקי הבניין. גם למנכ"ל אסור לעשן במסדרון — לא משנה מה כתוב בחוזה שלו.
> Permission Boundary = חגורת בטיחות אישית: העובד יכול להעניק לעצמו הרשאות, אבל לא מעבר לגבול שהוגדר לו.

---

## 2. ⚙️ איך זה עובד

### 2.1 מבנה AWS Organizations

- Organizations הוא שירות **גלובלי**.
- יש **Management Account** אחד (החשבון שיצר את הארגון) ו-**Member Accounts** רבים.
- חשבון חבר יכול להשתייך ל-**ארגון אחד בלבד**.
- החשבונות מסודרים ב-**Organizational Units (OUs)** שיכולים להיות מקוננים.
- קיים API שמאפשר **ליצור חשבונות חדשים אוטומטית**.

```text
                    Root (OU עליון)
                          │
     ┌────────────────────┼────────────────────┐
     ▼                    ▼                    ▼
  OU: Sandbox         OU: Workloads        OU: Security
     │                    │                    │
Account A            ┌────┴────┐          Log Archive
Account B         OU: Dev   OU: Prod       Audit
                  Dev-1      Prod-1
                  Dev-2      Prod-2

Management Account — יושב מחוץ להיררכיה האפקטיבית של SCP
```

### 2.2 שלוש דרכים נפוצות לארגן OUs

| שיטה | דוגמה | מתי מתאים |
|---|---|---|
| **לפי יחידה עסקית** | Sales OU, Retail OU, Finance OU | ארגון עם חטיבות עצמאיות ותקציב נפרד |
| **לפי מחזור חיים** | Dev OU, Test OU, Prod OU | הדרישה המרכזית היא הפרדת סביבות |
| **לפי פרויקט** | Project 1 OU, Project 2 OU | עבודה בפרויקטים עם לקוחות שונים |

- מותר לשלב: OU של Prod שבתוכו OUs לפי פרויקט.
- ההיררכיה משפיעה ישירות על SCP — לכן כדאי לתכנן אותה סביב הבידוד שרוצים לאכוף.

### 2.3 יתרונות תפעוליים של multi-account

- תקן tagging אחיד לצורכי חיוב וייחוס עלויות.
- להפעיל CloudTrail בכל החשבונות ולשלוח את הלוגים ל-**bucket מרכזי בחשבון ייעודי**.
- לרכז CloudWatch Logs בחשבון logging מרכזי.
- להקים **Cross-Account Roles** לניהול, במקום users כפולים בכל חשבון.
- לאכוף אבטחה ברמת הארגון דרך SCP.

### 2.4 איך SCP עובד — הכללים שקובעים הכול

- SCP הוא מסמך בפורמט IAM policy, שמוצמד ל-**Root, ל-OU או לחשבון**.
- הוא מגביל **users ו-roles** בתוך החשבון — כולל את ה-admin שלו.
- **הוא לא מעניק הרשאות בכלל.** הוא רק קובע את **התקרה** של מה שאפשר להעניק.
- **הוא לא חל על ה-Management Account.** לחשבון הזה יש תמיד סמכות מלאה.
- כמו IAM, SCP **לא מתיר כלום כברירת מחדל**. חייב להיות `Allow` מפורש
  בכל מפרק בנתיב — מה-Root, דרך כל OU, ועד החשבון עצמו.

```text
תקרת ההרשאות בחשבון = חיתוך של כל ה-SCPs בנתיב אליו

Root      : FullAWSAccess
   │
OU Prod   : FullAWSAccess + Deny על שינוי הגדרות CloudTrail
   │
Account X : FullAWSAccess + Deny על Regions שאינם eu-central-1
   │
   ▼
תקרה בפועל: הכול, חוץ מכיבוי CloudTrail וחוץ מפעולות מחוץ ל-eu-central-1
                     ↑
        גם אם ל-user יש AdministratorAccess מלא
```

- **Deny במקום גבוה בעץ יורש למטה ולא ניתן לביטול** ברמה נמוכה יותר.
- אם ל-OU יש רק `Allow EC2`, חשבון שתחתיו לא יוכל להשתמש ב-S3 — גם אם ה-IAM מתיר.

### 2.5 שתי אסטרטגיות SCP

| אסטרטגיה | איך בנוי | יתרון | חיסרון |
|---|---|---|---|
| **Blocklist / Deny list** | להתחיל מ-`FullAWSAccess` ולהוסיף `Deny` נקודתי | קל להתחיל, שירותים חדשים זמינים אוטומטית | קל לשכוח לחסום משהו |
| **Allowlist / Allow list** | להסיר `FullAWSAccess` ולהתיר רק רשימה מפורשת | שליטה הדוקה מאוד | תחזוקה כבדה; כל שירות חדש דורש עדכון |

- Blocklist היא הגישה הנפוצה יותר בפועל.
- Allowlist מתאימה לסביבות רגולטוריות קשיחות במיוחד.

---

## 3. 🔍 פירוק מפורט

### 3.1 Tag Policies

- מטרה: לאכוף **סטנדרט תיוג אחיד** על כל משאבי הארגון.
- מגדירים אילו מפתחות tag קיימים ואילו ערכים מותרים להם.
- מונע פעולות תיוג לא-תקינות בשירותים ובמשאבים שהוגדרו.
- **אין לו השפעה על משאבים שאין להם tags בכלל** — נקודה שקל לפספס.
- מפיק דוח של משאבים תקינים ולא-תקינים.
- אפשר לנטר הפרות דרך **EventBridge** ולהריץ תיקון אוטומטי.
- תומך ישירות ב-Cost Allocation Tags וב-Attribute-Based Access Control.

### 3.2 IAM Conditions — ארבעת המפתחות שחוזרים במבחן

| Condition Key | מה הוא מגביל | תרחיש טיפוסי |
|---|---|---|
| `aws:SourceIp` | כתובת ה-IP שממנה מגיעה קריאת ה-API | לאפשר גישה רק מרשת המשרד או מ-VPN |
| `aws:RequestedRegion` | ה-Region שאליו מופנית הקריאה | לכפות עבודה ב-Regions מאושרים בלבד |
| `ec2:ResourceTag` | תגית על המשאב עצמו | לאפשר עצירה רק של instances עם `Env=Dev` |
| `aws:MultiFactorAuthPresent` | האם ה-session אומת ב-MFA | לדרוש MFA לפעולות הרסניות |

- Conditions הם הכלי המדויק ביותר ל-least privilege — עדיף עליהם מאשר לפצל policies.

### 3.3 aws:PrincipalOrgID

- Condition key שנועד ל-**resource-based policies**.
- הוא מגביל את הגישה למשאב **רק לחשבונות שהם חברים בארגון שלך**.
- הערך הוא מזהה הארגון בפורמט `o-xxxxxxxxxx`.
- למה זה חזק: אין צורך לתחזק רשימת account IDs שמשתנה כל חודש.
  חשבון חדש שנוסף לארגון מקבל גישה אוטומטית; חשבון שיצא מאבד אותה.

```text
S3 Bucket "financial-data"
   Bucket Policy:
     Allow  s3:GetObject
     Condition: aws:PrincipalOrgID = o-yyyyyyyyyy

   ✓ user מחשבון חבר בארגון      → מותר
   ✗ user מחשבון AWS חיצוני      → נדחה, גם אם יש לו IAM Allow
```

### 3.4 IAM Roles מול Resource-Based Policies

לגישה חוצת-חשבונות יש **שתי דרכים**:

| דרך | איך זה עובד | מה קורה להרשאות המקוריות |
|---|---|---|
| **Assume Role** | ה-user בחשבון A לובש role בחשבון B | **מוותר** על ההרשאות שלו בחשבון A וזוכה בהרשאות ה-role |
| **Resource-Based Policy** | המשאב בחשבון B מתיר ישירות ל-principal מחשבון A | **שומר** על ההרשאות המקוריות שלו |

> [!info] המקרה שמכריע את השאלה
> user בחשבון A צריך לסרוק טבלת DynamoDB **בחשבון A** ולכתוב את התוצאה ל-bucket **בחשבון B**.
> אם הוא יעשה assume role לחשבון B — הוא יאבד את הגישה ל-DynamoDB בחשבון A.
> לכן התשובה היא **Bucket Policy** בחשבון B: כך הוא שומר על שתי ההרשאות בו-זמנית.

- שירותים שתומכים ב-resource-based policies: S3 buckets, SNS topics, SQS queues, Lambda, KMS ועוד.
- שירותים שאין להם resource policy מחייבים role.

### 3.5 EventBridge — איזה מנגנון הרשאה משמש

כשכלל של EventBridge מפעיל target, הוא זקוק להרשאה על ה-target:

| סוג Target | מנגנון ההרשאה |
|---|---|
| Lambda, SNS, SQS, S3, API Gateway | **Resource-based policy** על ה-target |
| EC2 Auto Scaling, SSM Run Command, ECS Task | **IAM Role** שמוצמד לכלל |

### 3.6 IAM Permission Boundaries

- Boundary היא **managed policy** שמגדירה את **ההרשאות המקסימליות** שזהות יכולה לקבל.
- נתמכת עבור **users ו-roles בלבד — לא עבור groups**.
- ההרשאות בפועל = **חיתוך** בין ה-IAM policy לבין ה-boundary.

```text
IAM Policy נותנת:      S3 + EC2 + IAM
Permission Boundary:   S3 + DynamoDB
────────────────────────────────────
הרשאות בפועל:          S3 בלבד
```

- מה זה פותר: מאפשר להאציל סמכות בלי לפתוח דלת ל-privilege escalation.
- תרחישי שימוש מהשקפים:
  - להאציל למי שאינו admin את היכולת ליצור IAM users — בתוך גבול מוגדר.
  - לתת למפתחים לנהל את ההרשאות של עצמם, בלי שיוכלו להפוך את עצמם ל-admin.
  - להגביל **user בודד**, במקום להגביל חשבון שלם דרך SCP.
- אפשר וכדאי לשלב Boundary יחד עם SCP — הם פועלים בשכבות שונות.

### 3.7 לוגיקת הערכת ההרשאות — הסדר המלא

```text
בקשה מגיעה
   │
   ├─► יש Explicit DENY במקום כלשהו?  ─── כן ──► DENY. סוף.
   │      (IAM / SCP / Resource / Boundary)
   ▼ לא
   ├─► SCP מתיר את הפעולה?            ─── לא ──► DENY
   ▼ כן
   ├─► Resource-based policy מתירה?    ─── כן ──► ALLOW
   ▼ (או שאין כזו)
   ├─► Permission Boundary מתירה?      ─── לא ──► DENY
   ▼ כן
   ├─► Identity policy מתירה?          ─── לא ──► DENY (implicit)
   ▼ כן
 ALLOW
```

שלושת הכללים שצריך לזכור:

1. **Explicit Deny גובר על הכול.** בכל שכבה, מכל מקור.
2. **ברירת המחדל היא Implicit Deny.** בלי Allow מפורש — אין גישה.
3. **כל שכבה חייבת להתיר.** SCP, Boundary ו-Identity Policy — כולן צריכות לומר "כן".

### 3.8 AWS IAM Identity Center

- היורש של AWS Single Sign-On.
- **התחברות אחת** נותנת גישה ל:
  - כל חשבונות ה-AWS בארגון,
  - אפליקציות עסקיות בענן (Salesforce, Box, Microsoft 365),
  - אפליקציות שתומכות ב-SAML 2.0,
  - EC2 Windows Instances.

**מקורות זהות אפשריים:**

| מקור | מתי מתאים |
|---|---|
| Identity Store מובנה של Identity Center | ארגון ללא ספריית זהויות קיימת |
| Active Directory (on-prem או ב-AWS) | לארגון כבר יש AD ומנהלים בו את המשתמשים |
| ספק חיצוני: Okta, OneLogin וכדומה | כבר קיים IdP ארגוני |

**שלושת המנגנונים המרכזיים:**

- **Permission Sets** — אוסף של policy אחת או יותר, שמוקצה ל-user או ל-group ומגדיר מה מותר לו בחשבון.
  אותו permission set אפשר להקצות למספר חשבונות.
- **Application Assignments** — SSO לאפליקציות SAML 2.0; מספקים URL, תעודות ו-metadata.
- **ABAC (Attribute-Based Access Control)** — הרשאות לפי **מאפייני המשתמש** ב-Identity Store,
  למשל cost center, תפקיד או locale.
  היתרון: מגדירים את ההרשאות פעם אחת, ומשנים גישה על ידי שינוי המאפיין בלבד.

```text
Bob (group: Developers)
  │
  ├─► Permission Set "ReadOnlyAccess"  ──► הוקצה ל-OU Development
  │                                         → Dev Account A, Dev Account B
  └─► אין הקצאה ל-OU Production        ──► אין לו גישה ל-Prod כלל
```

### 3.9 Active Directory ו-AWS Directory Service

**מה זה Active Directory:**

- שירות של Microsoft שרץ על Windows Server עם AD Domain Services.
- מסד נתונים של אובייקטים: חשבונות משתמש, מחשבים, מדפסות, שיתופי קבצים, security groups.
- ניהול אבטחה מרכזי: יצירת חשבונות והענקת הרשאות.
- האובייקטים מסודרים ב-**trees**, וקבוצת trees נקראת **forest**.

**שלוש האפשרויות ב-AWS Directory Service:**

| שירות | מה זה | AD קיים on-prem? | תמיכה ב-MFA |
|---|---|---|---|
| **AWS Managed Microsoft AD** | AD אמיתי שרץ ב-AWS; מנהלים בו משתמשים מקומית | אפשר ליצור **trust** אליו | כן |
| **AD Connector** | שער/proxy בלבד — מפנה את האימות ל-AD המקומי | **חובה** — המשתמשים מנוהלים שם | כן |
| **Simple AD** | ספרייה מנוהלת תואמת-AD ב-AWS | **לא ניתן** לחבר ל-AD מקומי | — |

> [!tip] איך בוחרים בשאלה
> "רוצים לנהל משתמשים גם ב-AWS וגם לשמור קשר ל-on-prem" → **Managed Microsoft AD** עם trust.
> "כל המשתמשים חייבים להישאר on-prem, בלי שכפול" → **AD Connector**.
> "צריך ספרייה פשוטה ב-AWS בלי on-prem בכלל" → **Simple AD**.

**חיבור Identity Center ל-AD:**

- חיבור ל-AWS Managed Microsoft AD — אינטגרציה מובנית, ללא עבודה נוספת.
- חיבור לספרייה עצמאית (self-managed) — בשתי דרכים:
  - יחסי **two-way trust** בין ה-AD המקומי לבין AWS Managed Microsoft AD.
  - או דרך **AD Connector** כ-proxy.

### 3.10 AWS Control Tower

- דרך מהירה להקים ולנהל סביבה **מרובת חשבונות** מאובטחת ותואמת, לפי best practices.
- **משתמש ב-AWS Organizations** מתחת למכסה המנוע כדי ליצור את החשבונות.
- מה הוא נותן:
  - הקמה אוטומטית של הסביבה בכמה קליקים.
  - ניהול מדיניות שוטף באמצעות **Guardrails**.
  - זיהוי הפרות מדיניות ותיקון שלהן.
  - מעקב אחר compliance ב-dashboard אינטראקטיבי.

**שני סוגי Guardrails:**

| סוג | מבוסס על | מה הוא עושה | דוגמה |
|---|---|---|---|
| **Preventive** | **SCP** | **מונע** מראש את הפעולה | לחסום פעולות מחוץ ל-Regions מאושרים |
| **Detective** | **AWS Config** | **מזהה** הפרה אחרי שקרתה ומדווח | לאתר משאבים ללא tags |

```text
Detective Guardrail בפעולה:

AWS Config מנטר Member Accounts
        │
        ▼  משאב מסומן NON_COMPLIANT
   ┌────┴────┐
   ▼         ▼
  SNS      Lambda
  │          │
Admin    תיקון אוטומטי (למשל הוספת ה-tag החסר)
```

> [!warning] ההבחנה שהמבחן אוהב
> **Preventive = SCP** — הפעולה פשוט נכשלת.
> **Detective = Config** — הפעולה מצליחה, ואז מתגלה ומתוקנת.
> אם השאלה אומרת "prevent" — זה SCP. אם היא אומרת "detect" או "report" — זה Config.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב | מודל חיוב | הערה |
|---|---|---|
| AWS Organizations | **ללא עלות** | משלמים רק על המשאבים בחשבונות |
| SCPs / Tag Policies | **ללא עלות** | מנגנון ממשל, לא שירות מחויב |
| IAM Identity Center | **ללא עלות** | ה-SSO עצמו חינם |
| AWS Control Tower | **ללא עלות ישירה** | אבל השירותים שהוא מפעיל כן עולים |
| AWS Config (Detective Guardrails) | לפי configuration items ולפי הערכות כללים | **זו העלות האמיתית של Control Tower** |
| CloudTrail — trail נוסף / data events | לפי אירועים + אחסון ב-S3 | ריכוז לוגים מכל החשבונות מצטבר |
| AWS Managed Microsoft AD | לפי שעה, לפי גודל הספרייה | הכי יקר מבין שלושת ה-Directory Services |
| AD Connector / Simple AD | לפי שעה, לפי גודל | זולים יותר מ-Managed AD |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| חשבון יחיד עם כמה VPCs | הזול תפעולית בטווח קצר | ארגון קטן מאוד, ללא דרישות בידוד |
| Organizations + SCPs | תוספת אפסית בחיוב | כמעט תמיד — אין סיבה לא |
| Control Tower מלא | תוספת בעיקר דרך Config | ארגון שמקים סביבה חדשה או צריך ממשל מוכח |
| Identity Center | 0 | תמיד עדיף על ניהול users בכל חשבון |
| Managed Microsoft AD | הגבוה מבין הספריות | כשבאמת צריך AD מלא ב-AWS |

### 🚩 עלויות נסתרות

- **AWS Config** ב-Control Tower מחויב לפי כל שינוי קונפיגורציה — בסביבה דינמית זה מצטבר מהר.
- **ריכוז לוגים** מכל החשבונות ב-bucket מרכזי: עלות ingestion, אחסון ו-retention.
- **חשבון נוסף** אינו חינם בפועל — כל חשבון דורש baseline: CloudTrail, Config, ניטור.
- **Directory Service** מחויב לפי שעה גם כשאף אחד לא מתחבר.
- **מורכבות תפעולית** של multi-account היא עלות אמיתית בשעות אדם.

### 💡 טיפים לחיסכון

- **Consolidated Billing** אינו רק נוחות: השימוש **מצטבר** לכל הארגון,
  ולכן מגיעים למדרגות הנחת כמות מהר יותר (למשל ב-EC2 וב-S3).
- **Reserved Instances ו-Savings Plans משותפים** בין החשבונות בארגון —
  RI שנקנה בחשבון אחד יכול לכסות שימוש בחשבון אחר.
- להשתמש ב-**SCP עם `aws:RequestedRegion`** כדי למנוע פריסות יקרות ב-Regions לא מאושרים.
- **Tag Policies** הופכים cost allocation לאפשרי — בלי תיוג עקבי אין ניתוח עלויות אמיתי.
- להפעיל Config רק על סוגי משאבים רלוונטיים במקום על הכול.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 ארבעת מנגנוני ההרשאה — הטבלה המרכזית

| מנגנון | חל על | מעניק הרשאות? | מגביל? | היקף |
|---|---|---|---|---|
| **Identity Policy** | user / group / role | **כן** | כן (עם Deny) | הזהות |
| **Resource-Based Policy** | המשאב עצמו | **כן** | כן | המשאב |
| **SCP** | חשבון / OU | **לא** | **כן** | כל החשבון, כולל admin |
| **Permission Boundary** | user / role (**לא group**) | **לא** | **כן** | זהות בודדת |

> [!warning] הכלל שהמבחן בודק שוב ושוב
> SCP ו-Permission Boundary **לעולם לא מעניקים הרשאה**. הם רק מורידים תקרה.
> גם אחרי שהסרת Deny מ-SCP, עדיין חייבת להיות policy שמתירה בפועל.

### 5.2 SCP מול Permission Boundary

| קריטריון | SCP | Permission Boundary |
|---|---|---|
| היקף ההשפעה | חשבון או OU שלם | user או role בודד |
| חל על ה-root user של החשבון | כן (ה-root של חשבון חבר מוגבל) | לא רלוונטי |
| חל על Management Account | **לא** | — |
| נוצר ומנוהל ב... | Organizations | IAM |
| שאלה טיפוסית | "לאסור Region על כל החברה" | "לתת למפתח ליצור users בלי שיהפוך ל-admin" |

### 5.3 Assume Role מול Resource-Based Policy

| קריטריון | Assume Role | Resource-Based Policy |
|---|---|---|
| ההרשאות המקוריות | **אובדות** בזמן ה-session | **נשמרות** |
| נדרש תמיכה במשאב | לא | **כן** — רק שירותים מסוימים |
| מתאים לפעולה שנוגעת בשני חשבונות בו-זמנית | לא | **כן** |
| מתאים לגישה מלאה לסביבה אחרת | **כן** | פחות |

### 5.4 שלושת ה-Directory Services

| קריטריון | Managed Microsoft AD | AD Connector | Simple AD |
|---|---|---|---|
| איפה יושבים המשתמשים | ב-AWS (וניתן trust ל-on-prem) | **רק on-prem** | ב-AWS |
| AD אמיתי של Microsoft | כן | לא — proxy בלבד | לא — תואם בלבד |
| חיבור ל-on-prem AD | trust | proxy ישיר | **לא נתמך** |
| MFA | כן | כן | — |
| עלות יחסית | הגבוהה | בינונית | הנמוכה |

### 5.5 Organizations מול Control Tower

| קריטריון | AWS Organizations | AWS Control Tower |
|---|---|---|
| מה זה | המנגנון הבסיסי לניהול חשבונות | שכבת אוטומציה וממשל מעליו |
| הקמת חשבונות | ידנית או דרך API | אוטומטית עם baseline מובנה |
| Guardrails | אתה כותב SCPs בעצמך | ערכת guardrails מוכנה, preventive + detective |
| Dashboard תאימות | אין | יש |
| מתי בוחרים | יש כבר מבנה וצריך רק שליטה | מקימים סביבה חדשה ורוצים best practices מוכנות |

> [!info] שורה תחתונה
> "לחסום פעולה בכל הארגון" → SCP. "להגביל אדם אחד" → Permission Boundary.
> "כניסה אחת לכל החשבונות" → Identity Center. "להקים הכול מאפס לפי best practices" → Control Tower.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בממשל רב-חשבוני** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | הקמת חשבון חדש חייבת להיות חוזרת וצפויה | Control Tower ל-account factory, Permission Sets במקום users ידניים |
| Security | חשבון הוא גבול הבידוד החזק ביותר ב-AWS | SCP שאוסר כיבוי CloudTrail, `aws:PrincipalOrgID` על buckets רגישים |
| Reliability | הרשאות חוצות-חשבונות ל-backup ול-DR חייבות להיות מוכנות מראש | ליצור ולבדוק את ה-roles ל-restore לפני שצריך אותם |
| Performance Efficiency | federation מוריד שכבות proxy ותהליכי אימות ידניים | Identity Center + STS במקום סטים של credentials בכל חשבון |
| Cost Optimization | Consolidated Billing מצבירה הנחות כמות ומשתפת RI/SP | Tag Policies לייחוס עלות, SCP שחוסם Regions ו-instance types יקרים |
| Sustainability | חשבונות sandbox נשכחים וממשיכים לצרוך | Detective Guardrail שמאתר משאבים לא מתויגים ורדומים ומכבה אותם |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "prevent all accounts from using Region X" | SCP (Preventive Guardrail) |
| "even account administrators must not be able to..." | SCP |
| "restrict what a single developer can grant himself" | Permission Boundary |
| "single sign-on across all AWS accounts" | IAM Identity Center |
| "detect and report non-compliant resources" | AWS Config / Detective Guardrail |
| "set up a compliant multi-account environment quickly" | AWS Control Tower |
| "allow access only to accounts in our organization" | `aws:PrincipalOrgID` ב-resource policy |
| "user must keep permissions in both accounts" | Resource-Based Policy, לא assume role |
| "users must stay in the on-premises directory" | AD Connector |
| "run a real Microsoft AD inside AWS with a trust" | AWS Managed Microsoft AD |
| "enforce consistent tag keys and values" | Tag Policies |
| "restrict API calls to the corporate IP range" | Condition עם `aws:SourceIp` |
| "one payment method and volume discounts" | Consolidated Billing |

### טעויות נפוצות

> [!warning] מלכודת 1 — SCP כמעניק הרשאות
> **הניסוח:** "Attach an SCP that allows s3:GetObject so the developers can read the bucket."
> **הטעות:** לחשוב ש-SCP מעניק גישה.
> **הנכון:** SCP רק קובע תקרה. עדיין נדרשת IAM policy שמתירה בפועל. שתי השכבות חייבות להסכים.

> [!warning] מלכודת 2 — SCP על ה-Management Account
> **הניסוח:** "Apply the SCP to the root so it restricts every account including the management account."
> **הטעות:** להניח ש-SCP חל על כולם.
> **הנכון:** SCP **אינו חל על ה-Management Account**. לכן לא מריצים בו workloads — הוא נשאר חשבון ניהול בלבד.

> [!warning] מלכודת 3 — Permission Boundary ל-Group
> **הניסוח:** "Attach a permission boundary to the Developers group."
> **הטעות:** להניח ש-boundaries עובדות כמו policies.
> **הנכון:** Permission Boundary נתמכת עבור **users ו-roles בלבד**. לא ל-groups.

> [!warning] מלכודת 4 — assume role כשצריך שתי סמכויות
> **הניסוח:** "Have the user assume a role in Account B to copy data from Account A's database."
> **הטעות:** לשכוח שב-assume role מוותרים על ההרשאות המקוריות.
> **הנכון:** אם צריך גישה בו-זמנית לשני החשבונות — resource-based policy על היעד.

> [!warning] מלכודת 5 — Tag Policy על משאבים לא מתויגים
> **הניסוח:** "Use a tag policy to force tags on all existing untagged resources."
> **הטעות:** להניח שהיא מוסיפה tags.
> **הנכון:** Tag Policy מונעת פעולות תיוג לא-תקינות, ו**אין לה השפעה על משאבים ללא tags כלל**.
> לאיתור משאבים לא מתויגים משתמשים ב-Config / דוח התאימות.

> [!warning] מלכודת 6 — Simple AD עם on-prem
> **הניסוח:** "Use Simple AD and create a trust with the on-premises Active Directory."
> **הטעות:** להניח שכל הספריות מתחברות ל-on-prem.
> **הנכון:** Simple AD **לא ניתן לחיבור** ל-AD מקומי. לכך משמשים Managed Microsoft AD (trust) או AD Connector.

> [!warning] מלכודת 7 — חשבון חבר בשני ארגונים
> **הניסוח:** "Add the account to both the security organization and the business organization."
> **הטעות:** לחשוב שאפשר להשתייך לכמה ארגונים.
> **הנכון:** חשבון חבר יכול להיות ב-**ארגון אחד בלבד**.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

חברת fintech עוברת מחשבון AWS יחיד למבנה ארגוני.

- הרגולציה מחייבת שכל המשאבים יהיו ב-`eu-central-1` בלבד.
- אסור שמישהו — כולל admin — יוכל לכבות CloudTrail.
- לוגי audit חייבים להישמר בחשבון נפרד שלא ניתן למחוק ממנו.
- 40 עובדים; החברה כבר מנהלת משתמשים ב-Active Directory מקומי.
- מפתחים צריכים לנהל את ההרשאות של עצמם ב-sandbox, בלי להפוך ל-admin.
- bucket עם דוחות כספיים חייב להיות נגיש רק לחשבונות של החברה.
- דרישה: לדעת כמה עולה כל סביבה בנפרד.

```text
                    Management Account
                    (ניהול בלבד, בלי workloads)
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
   OU: Security          OU: Workloads         OU: Sandbox
   ├── Log Archive        ┌────┴────┐          Sandbox-Dev
   └── Audit           OU: Dev   OU: Prod           │
                        Dev-1     Prod-1      Permission Boundary
                                                על ה-users שם

SCP ב-Root:      Deny אם aws:RequestedRegion ≠ eu-central-1
SCP ב-Root:      Deny על cloudtrail:StopLogging / DeleteTrail
SCP ב-Sandbox:   Deny על instance types יקרים

Identity Center ──► AWS Managed Microsoft AD ──(two-way trust)──► on-prem AD
       │
       └── Permission Sets: ReadOnly / PowerUser / Admin לפי OU

S3 "financial-reports" (בחשבון Audit)
   Bucket Policy: Condition aws:PrincipalOrgID = o-xxxxxxxxxx
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Management Account ללא workloads | SCP לא חל עליו — כל משאב שם חסר guardrails |
| SCP ב-Root עם `aws:RequestedRegion` | Deny במקום הגבוה ביותר יורש לכל החשבונות ולא ניתן לביטול למטה |
| SCP שאוסר כיבוי CloudTrail | עונה בדיוק על "גם admin לא יוכל" — IAM לבדו לא מספיק |
| חשבון Log Archive נפרד | מפריד בין מי שמייצר לוגים למי ששומר אותם |
| Permission Boundary ב-Sandbox | מאפשר למפתחים לנהל הרשאות עצמאית בלי privilege escalation |
| Managed Microsoft AD + two-way trust | משאיר את ניהול המשתמשים ב-AD הקיים ומאפשר אינטגרציה מלאה |
| Identity Center עם Permission Sets | כניסה אחת לכל החשבונות; הרשאות מוגדרות פעם אחת ומוקצות לפי OU |
| `aws:PrincipalOrgID` על ה-bucket | חוסם כל חשבון חיצוני בלי לתחזק רשימת account IDs |
| חשבון נפרד לכל סביבה + Tag Policies | Consolidated Billing מציג עלות לפי חשבון ולפי tag |

**למה לא SCP במקום Permission Boundary ב-sandbox?**

- SCP חוסם את **כל** החשבון באופן אחיד.
- כאן רוצים לתת חופש בתוך החשבון, ולהגביל **זהות ספציפית**. זה בדיוק תפקידה של Boundary.

**למה לא AD Connector?**

- החברה רוצה גם ניהול מקומי ב-AWS וגם אינטגרציה מלאה עם Identity Center.
- AD Connector הוא proxy בלבד ואינו מאפשר לנהל אובייקטים ב-AWS.

**למה לא Control Tower?**

- זו אפשרות לגיטימית ואף מומלצת להקמה מאפס, והיא הייתה נותנת את ה-guardrails והחשבונות אוטומטית.
- החברה כאן כבר מנהלת מבנה קיים; אם השאלה הייתה "להקים סביבה חדשה במהירות לפי best practices" —
  **Control Tower הייתה התשובה**.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- לשנן את התחביר המדויק של כל SCP או את כל ה-condition keys של Organizations.
- את מסכי הקונסולה של Control Tower או של Identity Center.
- את הרשימה המלאה של guardrails מובנים ב-Control Tower.
- מבנה פנימי של Active Directory ברמת LDAP, schema או replication.
- את כל השירותים שתומכים ב-resource-based policies. מספיק לזכור S3, SNS, SQS, Lambda, KMS.
- תמחור מדויק של Directory Service או של Config.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- Organizations = שירות **גלובלי**. Management Account אחד + Member Accounts.
- **חשבון חבר שייך לארגון אחד בלבד.**
- Consolidated Billing: אמצעי תשלום אחד, **הנחות כמות מצטברות**, ו-**RI/Savings Plans משותפים** בין חשבונות.
- יש API ליצירת חשבונות אוטומטית.
- **SCP מגביל, לא מעניק.** חל על users ו-roles בחשבון — **לא על ה-Management Account**.
- SCP לא מתיר כלום כברירת מחדל: נדרש `Allow` מפורש בכל הנתיב מה-Root ועד החשבון.
- **Blocklist** = FullAWSAccess + Deny נקודתי. **Allowlist** = להסיר FullAWSAccess ולהתיר רשימה.
- **Explicit Deny > SCP > Resource Policy > Boundary > Identity Policy > Implicit Deny.**
- **Permission Boundary:** users ו-roles בלבד, **לא groups**. הרשאות בפועל = חיתוך עם ה-policy.
- **Assume Role = מוותרים** על ההרשאות המקוריות. **Resource-Based Policy = שומרים** עליהן.
- לפעולה שנוגעת בשני חשבונות בו-זמנית — resource-based policy.
- `aws:PrincipalOrgID` בתוך resource policy = גישה רק לחברי הארגון.
- Conditions לזכור: `aws:SourceIp`, `aws:RequestedRegion`, `ec2:ResourceTag`, `aws:MultiFactorAuthPresent`.
- S3: `ListBucket` על ה-bucket ARN; `GetObject`/`PutObject`/`DeleteObject` על `bucket/*`.
- **Tag Policies** אוכפות מפתחות וערכים; **אין להן השפעה על משאבים ללא tags**.
- **Identity Center** (יורש AWS SSO): SSO לחשבונות, לאפליקציות SAML 2.0 ול-EC2 Windows.
- **Permission Set** = אוסף policies שמוקצה ל-user/group. **ABAC** = הרשאות לפי מאפייני משתמש.
- **Managed Microsoft AD** = AD אמיתי ב-AWS + trust. **AD Connector** = proxy ל-on-prem. **Simple AD** = לא מתחבר ל-on-prem.
- **Control Tower** בנוי מעל Organizations. **Preventive Guardrail = SCP**, **Detective Guardrail = AWS Config**.

---

## 11. ✅ בדיקת הבנה

1. SCP מסיר Deny על S3, אבל ל-user אין IAM policy ל-S3. האם יש לו גישה?
2. חברה רוצה למנוע מכל החשבונות לפרוס מחוץ ל-eu-west-1. איזה מנגנון, ואיפה מצמידים אותו?
3. למה לא מריצים workloads ב-Management Account?
4. user בחשבון A צריך לקרוא מ-DynamoDB בחשבון A ולכתוב ל-S3 בחשבון B. Role או Bucket Policy?
5. מה ההבדל בין Preventive ל-Detective Guardrail, ועל מה כל אחד בנוי?
6. רוצים לתת למפתח ליצור IAM users, בלי שיוכל להעניק לעצמו AdministratorAccess. מה הפתרון?
7. החברה מסרבת לשכפל את ה-Active Directory שלה ל-AWS. איזו אפשרות ב-Directory Service?
8. איך נותנים גישה ל-bucket לכל חשבונות הארגון, גם לחשבונות שייווצרו בעתיד?

<details>
<summary>תשובות</summary>

1. **לא.** SCP רק מסיר את התקרה. עדיין נדרשת identity policy (או resource policy) שמתירה את הפעולה בפועל. שתי השכבות חייבות לומר "כן".

2. **SCP** עם `Deny` מותנה ב-`aws:RequestedRegion` שאינו `eu-west-1`, מוצמד ל-**Root OU** — כך הוא יורש לכל החשבונות ואי אפשר לבטל אותו ברמה נמוכה יותר. שים לב שהוא לא יחול על ה-Management Account.

3. כי **SCP אינו חל עליו**. כל משאב שירוץ שם יהיה מחוץ ל-guardrails של הארגון. מחזיקים אותו כחשבון ניהול וחיוב בלבד.

4. **Bucket Policy** בחשבון B. אם הוא יעשה assume role לחשבון B הוא **יוותר** על ההרשאות שלו בחשבון A ולא יוכל לקרוא מה-DynamoDB.

5. **Preventive** מבוסס **SCP** — הפעולה נחסמת מראש ופשוט נכשלת. **Detective** מבוסס **AWS Config** — הפעולה מצליחה, ואז מזוהה כ-NON_COMPLIANT, מדווחת (SNS) ולעיתים מתוקנת אוטומטית (Lambda).

6. **IAM Permission Boundary** על ה-user. הוא יוכל ליצור users ולנהל הרשאות, אך ההרשאות האפקטיביות תמיד יהיו החיתוך עם ה-boundary — ולכן אין דרך להסלים לאדמין.

7. **AD Connector.** הוא proxy שמפנה את האימות ל-AD המקומי; המשתמשים נשארים מנוהלים on-prem ולא משוכפלים.

8. **Bucket Policy עם Condition על `aws:PrincipalOrgID`** השווה למזהה הארגון. חשבון שיצטרף בעתיד יקבל גישה אוטומטית, וחשבון שיעזוב יאבד אותה — בלי לתחזק רשימת account IDs.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[03 - IAM Fundamentals]] · [[31 - Monitoring and Logging]] · [[32 - Security Services]] · [[36 - Migration and Hybrid Cloud]] · [[37 - Cost Optimization]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 11164–11692

> [!note] הערת דיוק
> סדר הערכת ההרשאות בסעיף 3.7 מוצג בשקפים כדיאגרמה בלבד; כאן הוא מנוסח לפי לוגיקת ההערכה הרשמית של AWS.
> AWS IAM Identity Center הוא השם הנוכחי של מה שנקרא בעבר AWS Single Sign-On — שני השמות עשויים להופיע במבחן.
