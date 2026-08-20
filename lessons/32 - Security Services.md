---
lesson: 32
title: Security Services
domain: Design Secure Architectures
services: [KMS, CloudHSM, ACM, SSM Parameter Store, Secrets Manager, WAF, Shield, Firewall Manager, Network Firewall, GuardDuty, Inspector, Macie, Security Hub]
tags: [saa-c03, security, encryption, kms, waf, threat-detection]
---

# 32 — Security Services

> [!abstract] בשורה אחת
> ארבע שכבות אבטחה שצריך להפריד בראש: **מפתחות** (KMS/CloudHSM/ACM), **סודות** (Parameter Store/Secrets Manager), **הגנת קצה** (WAF/Shield/Firewall Manager) ו**זיהוי** (GuardDuty/Inspector/Macie/Security Hub).

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה defense in depth, ולמה כל שירות עונה על שאלה אחרת |
| 2א | הצפנה ומפתחות | in-flight / at-rest / client-side, KMS, CloudHSM, ACM |
| 2ב | ניהול סודות | Parameter Store מול Secrets Manager |
| 2ג | הגנת קצה | WAF, Shield, Firewall Manager, Network Firewall |
| 2ד | זיהוי ואיתור | GuardDuty, Inspector, Macie, Security Hub |
| 3 | פירוק מפורט | סוגי מפתחות, key policies, העתקת snapshots, MRK |
| 4 | עלות | מה חינם, מה בתשלום, ומה מתפוצץ בקנה מידה |
| 5 | השוואות | ארבע הטבלאות המכריעות של השיעור |
| 6-11 | Pillars, מלכודות, Scenario, Cheat Sheet | |

**מונחי מפתח בשיעור:** `Envelope Encryption` · `Key Policy` · `Multi-Region Key` · `SecureString` · `Web ACL` · `Rate-based rule` · `FIPS 140-2 Level 3` · `Finding`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- ה-DB password כתוב בקוד ב-Git. הוא לא הוחלף מאז 2019.
- ה-snapshot מוצפן — ואי אפשר להעתיק אותו ל-Region אחר בלי להבין למה זה נכשל.
- מישהו מריץ SQL injection על ה-login form.
- 900,000 בקשות בשנייה מגיעות מ-botnet, וה-ASG פשוט גדל ומייצר חשבונית ענק.
- יש 40 חשבונות בארגון. אף אחד לא יודע איזה bucket מכיל תעודות זהות של לקוחות.

### מה השירות פותר — ארבע שכבות

| שכבה | השאלה | השירותים |
|---|---|---|
| **הצפנה ומפתחות** | איך שומרים דאטה כך שגניבת הדיסק לא שווה כלום? | KMS, CloudHSM, ACM |
| **ניהול סודות** | איפה שומרים סיסמאות ומחליפים אותן אוטומטית? | Parameter Store, Secrets Manager |
| **הגנת קצה** | איך חוסמים תעבורה זדונית לפני שהיא מגיעה לאפליקציה? | WAF, Shield, Firewall Manager, Network Firewall |
| **זיהוי ואיתור** | איך יודעים שמשהו רע כבר קורה או שיש חשיפה? | GuardDuty, Inspector, Macie, Security Hub |

> [!tip] האנלוגיה
> KMS = הכספת של הבניין. Secrets Manager = מנהל הסיסמאות שמחליף לך קוד כל 30 יום.
> WAF/Shield = השומר בכניסה שבודק מי נכנס. GuardDuty/Inspector/Macie = מצלמות האבטחה שמזהות מה כבר קרה בפנים.

---

## 2. ⚙️ איך זה עובד

## 2א. הצפנה ומפתחות

### 2א.1 שלושת מודלי ההצפנה

| מודל | מי מצפין | מי יכול לפענח | דוגמה |
|---|---|---|---|
| **In flight (TLS/SSL)** | הלקוח לפני השליחה | הצד המקבל | HTTPS — מונע MITM |
| **Server-side at rest** | השרת אחרי הקבלה | השרת (יש לו גישה ל-key) | SSE-S3, EBS encryption, RDS encryption |
| **Client-side** | הלקוח | **רק לקוח אחר** — השרת לא יכול | Encryption SDK, DynamoDB Encryption Client |

- ב-server-side הדאטה נשמר מוצפן בעזרת **data key**, וצריך מקום לנהל את המפתחות.
- ב-client-side השרת מקבל דאטה שהוא לא מסוגל לפענח — זה מה שמגן גם מפני **DBA** ומפני AWS עצמה.
- **Envelope Encryption** — מצפינים את הדאטה עם data key, ואת ה-data key מצפינים עם KMS key. זה מה שמאפשר להצפין קבצים גדולים (KMS API מוגבל ל-4KB ישירות).

### 2א.2 AWS KMS — הליבה

- כשכתוב "encryption" ליד שירות AWS — כמעט תמיד מדובר ב-**KMS**.
- משולב לחלוטין ב-**IAM** להרשאות, וכל שימוש במפתח מתועד ב-**CloudTrail**.
- משולב מקורית ב-EBS, S3, RDS, SSM, DynamoDB, EFS ועוד.
- אפשר לקרוא ל-KMS גם ישירות דרך SDK/CLI ולשמור את ה-ciphertext בקוד או ב-environment variable.
- **אף פעם** לא שומרים secret בטקסט גלוי, ובוודאי לא בקוד.

### 2א.3 סוגי מפתחות KMS

| סוג מפתח | מי יוצר/מנהל | עלות | מתי |
|---|---|---|---|
| **AWS Owned Keys** | AWS, משותף בין לקוחות | **חינם** | ברירת מחדל של SSE-S3, SSE-SQS, SSE-DDB |
| **AWS Managed Keys** (`aws/rds`, `aws/ebs`) | AWS, פר-שירות פר-חשבון | **חינם** | הצפנה "בלחיצה" בלי דרישות מיוחדות |
| **Customer Managed Key (CMK)** — נוצר ב-KMS | אתה | חיוב חודשי קבוע לכל מפתח | כשצריך key policy, rotation נשלט, cross-account |
| **Customer Managed — Imported** | אתה מייבא key material | חיוב חודשי קבוע לכל מפתח | דרישות רגולציה של BYOK |

בנוסף: **חיוב לכל קריאת API ל-KMS** (`Encrypt`, `Decrypt`, `GenerateDataKey`).

**Symmetric מול Asymmetric:**

| | Symmetric | Asymmetric |
|---|---|---|
| אלגוריתם | AES-256 | RSA / ECC |
| מפתחות | מפתח יחיד — מצפין ומפענח | זוג public/private |
| גישה למפתח | **לעולם לא רואים אותו** — חייבים לקרוא ל-KMS API | ה-**public** ניתן להורדה; ה-private לעולם לא |
| שימושים | Encrypt/Decrypt | Encrypt/Decrypt או **Sign/Verify** |
| מי משתמש | **כל שירותי AWS המשולבים ב-KMS** | הצפנה מחוץ ל-AWS ע"י מי שלא יכול לקרוא ל-KMS API |

### 2א.4 Key Rotation

| סוג מפתח | Rotation |
|---|---|
| AWS-Managed | אוטומטי, **כל שנה** |
| Customer-Managed | אוטומטי — **צריך להפעיל**; גם on-demand |
| Imported | **ידני בלבד** — יוצרים מפתח חדש ומעבירים את ה-**alias** אליו |

### 2א.5 KMS Key Policies

- שולטות בגישה למפתח — "דומה" ל-S3 bucket policy.
- **ההבדל הקריטי:** ב-KMS **אי אפשר לתת גישה בלי key policy**. IAM policy לבדה לא מספיקה — ה-key policy חייבת לאפשר.
- **Default Key Policy** — נוצרת אם לא סיפקת אחת; נותנת גישה מלאה ל-**root user** של החשבון (כלומר לחשבון כולו, דרך IAM).
- **Custom Key Policy** — מגדירים מי משתמש במפתח, מי מנהל אותו, ובעיקר: **גישה cross-account**.

### 2א.6 העתקת Snapshots מוצפנים

**בין Regions** — המפתח שייך ל-Region:

```text
eu-west-2: EBS Volume (KMS Key A) → Snapshot (Key A)
                 ↓ copy + KMS ReEncrypt
ap-southeast-2: Snapshot (KMS Key B) → Volume (Key B)
```

מפתח KMS **לא יוצא מה-Region שלו**. העתקה בין Regions תמיד עוברת re-encryption במפתח של ה-Region היעד.

**בין Accounts** — הרצף שחייבים לזכור:

1. יוצרים snapshot מוצפן ב-**Customer Managed Key** (לא AWS-managed — אין דרך לשתף אותו).
2. מצמידים **KMS Key Policy** שמאשרת גישה cross-account.
3. משתפים את ה-snapshot המוצפן עם החשבון היעד.
4. בחשבון היעד: **מעתיקים** את ה-snapshot ומצפינים ב-CMK מקומי.
5. יוצרים volume מה-copy.

### 2א.7 KMS Multi-Region Keys (MRK)

- מפתחות **זהים** ב-Regions שונים: אותו key ID, אותו key material, אותו rotation.
- מצפינים ב-Region אחד ומפענחים באחר — **בלי re-encrypt ובלי cross-Region API call**.
- **הם לא global!** יש **Primary + Replicas**, וכל אחד **מנוהל עצמאית** (key policy נפרדת).
- Use cases: client-side encryption גלובלי, **DynamoDB Global Tables**, **Aurora Global Database**.

```text
us-east-1 (Primary MRK) ──sync──→ us-west-2 (Replica)
                        ──sync──→ eu-west-1 (Replica)
לקוח ב-eu-west-1 מפענח מקומית → latency נמוך
```

**דוגמה:** מצפינים client-side את שדה ה-SSN ב-DynamoDB עם ה-Primary MRK ב-us-east-1, Global Table משכפל את הערך המוצפן ל-ap-southeast-2, ושם הלקוח מפענח מול ה-Replica MRK המקומי. אותו pattern עובד עם Aurora Global + AWS Encryption SDK — ומגן על השדה גם מפני DBA.

### 2א.8 Encryption Considerations ב-S3 Replication

| מצב האובייקט | האם משוכפל? |
|---|---|
| לא מוצפן | **כן, by default** |
| SSE-S3 | **כן, by default** |
| SSE-C (מפתח שהלקוח מספק) | כן, ניתן לשכפל |
| **SSE-KMS** | **צריך להפעיל את האופציה במפורש** |

עבור SSE-KMS יש להגדיר: איזה KMS Key ישמש ב-bucket היעד, להתאים את ה-Key Policy של מפתח היעד, ולתת ל-IAM Role הרשאות `kms:Decrypt` על מפתח המקור ו-`kms:Encrypt` על מפתח היעד. בנפחים גבוהים ייתכנו **KMS throttling errors** — פותרים בבקשת Service Quotas increase.

> [!warning] MRK ב-S3 Replication
> S3 מתייחס למפתחות multi-Region כאל מפתחות **עצמאיים**. האובייקט יפוענח ויוצפן מחדש — אין קיצור דרך.

### 2א.9 שיתוף AMI מוצפן

1. ה-AMI בחשבון המקור מוצפן ב-KMS Key של המקור.
2. משנים את ה-image attribute ומוסיפים **Launch Permission** לחשבון היעד.
3. **משתפים את ה-KMS Key** ששימש להצפנת ה-snapshots שה-AMI מפנה אליהם.
4. ל-IAM Role/User בחשבון היעד נדרשות: `DescribeKey`, `ReEncrypt*`, `CreateGrant`, `Decrypt`.
5. בהשקה, החשבון היעד יכול לבחור **KMS key משלו** ולהצפין מחדש את ה-volumes.

### 2א.10 ACM — תעודות TLS

- ניהול, הנפקה ופריסה של TLS certificates; זו ההצפנה **in-flight** (HTTPS).
- תעודות **public — ללא עלות**. חידוש **אוטומטי** (ACM מחדש 60 יום לפני פקיעה).
- תומך גם ב-private certificates (דרך ACM Private CA).
- אינטגרציות (טעינת התעודה על): **ELB (CLB/ALB/NLB)**, **CloudFront**, **API Gateway**.

**הנפקת תעודה public:**

1. רושמים את שמות הדומיין — FQDN (`corp.example.com`) או **wildcard** (`*.example.com`).
2. בוחרים שיטת validation: **DNS Validation** (CNAME record, מועדף — מאפשר אוטומציה וחידוש) או Email Validation (למיילים מ-WHOIS).
3. האימות לוקח מספר שעות.
4. התעודה נרשמת אוטומטית לחידוש.

**ייבוא תעודה חיצונית:**

- **אין חידוש אוטומטי** — חייבים לייבא תעודה חדשה לפני הפקיעה.
- ACM שולח **אירועי expiry יומיים החל מ-45 יום לפני** (מספר הימים ניתן להגדרה); האירועים מגיעים ל-**EventBridge**.
- ל-AWS Config יש managed rule בשם `acm-certificate-expiration-check`.

**ACM עם API Gateway — הנקודה שנופלים עליה:**

| Endpoint Type | איפה חייבת להיות התעודה |
|---|---|
| **Edge-Optimized** (ברירת מחדל, לקוחות גלובליים דרך CloudFront) | **us-east-1** — כי CloudFront שם |
| **Regional** | באותו Region של ה-API Stage |
| **Private** | נגיש רק מה-VPC דרך Interface Endpoint; שליטה ב-resource policy |

בשני המקרים מגדירים Custom Domain Name ואז **A-Alias** ב-Route 53 (עדיף על CNAME).

### 2א.11 CloudHSM — חומרה ייעודית

- ההבדל בשורה אחת: **KMS = AWS מנהלת את התוכנה. CloudHSM = AWS מספקת את החומרה, אתה מנהל את המפתחות לגמרי.**
- HSM = Hardware Security Module — התקן **tamper resistant**, תקן **FIPS 140-2 Level 3**.
- Single-tenant: החומרה שלך בלבד.
- תומך symmetric ו-asymmetric (כולל מפתחות SSL/TLS), חתימה דיגיטלית ו-hashing.
- **אין free tier.** חייבים להשתמש ב-**CloudHSM Client Software**.
- **HA:** cluster שפרוס על מספר AZs — מוסיפים HSMs ב-AZs שונים.
- **אינטגרציה עם שירותי AWS:** דרך **KMS Custom Key Store** — KMS מדבר מול ה-CloudHSM cluster, וכך EBS/S3/RDS מקבלים הצפנה שמגובה בחומרה שלך.
- **Redshift** תומך ב-CloudHSM לניהול מפתחות ה-DB; אופציה טובה גם ל-**SSE-C**.
- הרשאות: IAM שולט רק ב-CRUD על ה-cluster; **המשתמשים והמפתחות מנוהלים בתוך ה-HSM**, כולל תמיכה ב-MFA.

---

## 2ב. ניהול סודות

### 2ב.1 SSM Parameter Store

- אחסון מאובטח ל-**configuration ולסודות**.
- הצפנה אופציונלית ושקופה עם **KMS** (סוג `SecureString`).
- Serverless, scalable, durable, SDK פשוט.
- **Version tracking** לכל שינוי בערך.
- אבטחה דרך **IAM**; התראות דרך **EventBridge**; אינטגרציה עם **CloudFormation**.

**היררכיה** — זה מה שהופך אותו לנוח:

```text
/my-department/
   my-app/
      dev/db-url, dev/db-password
      prod/db-url, prod/db-password
   other-app/
```

- שולפים עם `GetParameters` או `GetParametersByPath` — Lambda של dev מושכת רק את `/my-department/my-app/dev/`.
- נתיבים מיוחדים: `/aws/reference/secretsmanager/<secret_id>` — **קריאת secret מ-Secrets Manager דרך Parameter Store**.
- `/aws/service/ami-amazon-linux-latest/...` — פרמטרים ציבוריים של AWS עם ה-AMI העדכני.

**Tiers:**

| | Standard | Advanced |
|---|---|---|
| מספר פרמטרים (לחשבון/Region) | 10,000 | 100,000 |
| גודל ערך מקסימלי | **4 KB** | **8 KB** |
| Parameter Policies | ❌ | ✅ |
| עלות אחסון | **חינם** | חיוב חודשי לכל advanced parameter |

**Parameter Policies** (רק ב-Advanced) — מאפשרות **TTL** לפרמטר, כדי לכפות עדכון או מחיקה של מידע רגיש:

- `Expiration` — מוחק את הפרמטר בתאריך.
- `ExpirationNotification` — התראה ל-EventBridge לפני הפקיעה.
- `NoChangeNotification` — התראה אם הפרמטר לא השתנה X זמן.
- אפשר להצמיד כמה policies יחד.

### 2ב.2 AWS Secrets Manager

- שירות ייעודי לסודות (חדש יותר מ-Parameter Store).
- **Rotation מאולץ כל X ימים** — זו התכונה שמבדילה אותו.
- **יצירה אוטומטית של סוד חדש** בכל rotation, באמצעות **Lambda**.
- אינטגרציה native עם **RDS (MySQL, PostgreSQL) ו-Aurora** — Secrets Manager מחליף את הסיסמה גם ב-DB וגם ב-secret.
- הסודות מוצפנים ב-**KMS**.
- **Multi-Region Secrets** — שכפול הסוד ל-Regions נוספים; Secrets Manager מסנכרן את ה-replicas מול ה-primary, וניתן **לקדם replica ל-standalone**. Use cases: אפליקציות multi-Region, **DR**, DB רב-אזורי.

---

## 2ג. הגנת קצה

### 2ג.1 AWS WAF — Layer 7

- מגן על אפליקציות web מפני exploits נפוצים ב-**שכבה 7 (HTTP)**, לעומת שכבה 4 (TCP/UDP).
- נפרס על: **ALB**, **API Gateway**, **CloudFront**, **AppSync GraphQL API**, **Cognito User Pool**.
- **לא תומך ב-NLB** (שכבה 4).

**Web ACL — סוגי כללים:**

| Rule | מה עושה | מגבלה/הערה |
|---|---|---|
| **IP Set** | חסימה/אישור לפי כתובות | עד **10,000 IP** ל-set; יותר → כמה rules |
| String / Regex match | על HTTP headers, body או URI | זה מה שחוסם **SQL injection** ו-**XSS** |
| Size constraints | הגבלת גודל בקשה | נגד payloads מנופחים |
| **Geo-match** | חסימת מדינות | |
| **Rate-based rules** | ספירת אירועים בחלון זמן | **הגנת DDoS בסיסית** — חוסם IP רועש אוטומטית |
| **Rule Group** | אוסף כללים לשימוש חוזר | כולל managed rules של AWS ושל ספקים |

- **Web ACL הוא Regional** — חוץ מ-CloudFront (שם הוא global).
- ה-Web ACL חייב להיות **באותו Region של ה-ALB**.

> [!tip] WAF + IP קבוע
> צריך IP קבוע וגם WAF? NLB נותן IP קבוע אבל **לא תומך ב-WAF**.
> הפתרון: **Global Accelerator** (IPv4 קבוע) → **ALB** עם WAF.

### 2ג.2 AWS Shield — DDoS

| | **Shield Standard** | **Shield Advanced** |
|---|---|---|
| עלות | **חינם, מופעל לכל לקוח AWS אוטומטית** | מנוי חודשי קבוע ויקר, לכל הארגון |
| שכבות | 3/4 — SYN/UDP floods, reflection attacks | 3/4/7, התקפות מתוחכמות |
| משאבים מוגנים | הכל | EC2, ELB, CloudFront, **Global Accelerator**, Route 53 |
| תמיכה | — | **גישה 24/7 ל-DDoS Response Team (DRT/SRT)** |
| חשבונית | — | **הגנה מפני חיוב מוגדל** בגלל spike של התקפה |
| WAF | — | **יוצר, מעריך ופורס כללי WAF אוטומטית** נגד התקפות שכבה 7 |

### 2ג.3 AWS Firewall Manager

- מנהל כללי אבטחה **בכל החשבונות של ה-Organization**.
- **Security Policy** = אוסף כללים אחיד שנאכף רוחבית.
- מה אפשר לנהל דרכו:
  - **WAF rules** — על ALB, API Gateway, CloudFront.
  - **Shield Advanced** — על ALB, CLB, NLB, Elastic IP, CloudFront.
  - **Security Groups** — ל-EC2, ALB ו-ENI ב-VPC.
  - **AWS Network Firewall** — ברמת ה-VPC.
  - **Route 53 Resolver DNS Firewall**.
- Policies נוצרות **ברמת ה-Region**.
- **המכה החזקה:** הכללים חלים אוטומטית על **משאבים חדשים** ועל **חשבונות עתידיים** בארגון. זו התשובה ל-"compliance אוטומטי לכל מה שייווצר".

### 2ג.4 AWS Network Firewall

- הגנה על **VPC שלם**, משכבה 3 עד שכבה 7.
- בודק תעבורה **בכל כיוון**: VPC↔VPC, outbound לאינטרנט, inbound מהאינטרנט, ומול **Direct Connect** ו-**Site-to-Site VPN**.
- מבוסס פנימית על **Gateway Load Balancer**.
- אלפי כללים: IP/port, protocol (למשל חסימת SMB יוצא), **stateful domain lists** (רק `*.mycorp.com` יוצא), התאמת regex.
- פעולות: Allow / Drop / Alert; יכולת **intrusion prevention** עם בדיקת flow חיה.
- לוגים ל-S3, CloudWatch Logs או Kinesis Data Firehose.
- ניהול cross-account דרך **Firewall Manager**.

### 2ג.5 Best Practices ל-DDoS Resiliency

| שכבה | פרקטיקה |
|---|---|
| **Edge** | **CloudFront** מגיש תוכן מהקצה וסופג SYN floods/UDP reflection; **Global Accelerator** לאפליקציות שלא מתאימות ל-CloudFront; **Route 53** עמיד ב-DDoS ברמת ה-DNS |
| **Infrastructure** | **ELB** גדל עם התעבורה ומפזר; **EC2 Auto Scaling** סופג flash crowd או התקפה |
| **Application** | **WAF** מעל CloudFront/ALB עם rate-based rules ו-managed rules (IP reputation, anonymous IPs); CloudFront חוסם גיאוגרפיות; Shield Advanced פורס כללי WAF אוטומטית |
| **Attack surface** | להסתיר את ה-backend מאחורי CloudFront/API Gateway/ELB; **Security Groups + NACLs** לסינון ברמת subnet/ENI; **Elastic IP מוגן ע"י Shield Advanced**; ב-API Gateway — burst limits, סינון headers, API keys |

---

## 2ד. זיהוי ואיתור

### 2ד.1 Amazon GuardDuty — זיהוי איומים

- **Intelligent Threat Discovery** על בסיס machine learning, anomaly detection ומודיעין צד-שלישי.
- **הפעלה בלחיצה אחת, 30 יום ניסיון, אין תוכנה להתקין.**
- **מקורות הנתונים (זו השאלה במבחן):**
  - **CloudTrail Events** — קריאות API חריגות, deployments לא מורשים.
  - **CloudTrail Management Events** — יצירת subnet, יצירת trail.
  - **CloudTrail S3 Data Events** — `GetObject`, `ListObjects`, `DeleteObject`.
  - **VPC Flow Logs** — תעבורה פנימית חריגה, IP חשוד.
  - **DNS Logs** — EC2 שנפרץ ומבריח דאטה מקודד בתוך DNS queries.
  - **תוספות אופציונליות:** EKS Audit Logs ו-Runtime Monitoring, RDS & Aurora login activity, EBS volumes (malware), Lambda network activity, S3 data events.
- **findings** → **EventBridge** → Lambda / SNS.
- יש **finding ייעודי ל-CryptoCurrency mining** — סימן קלאסי ל-instance שנפרץ.

### 2ד.2 Amazon Inspector — סריקת פגיעויות

- הערכות אבטחה **אוטומטיות**, על שלושה סוגי משאבים **בלבד**:

| משאב | מה נסרק | מנגנון |
|---|---|---|
| **EC2 instances** | פגיעויות ידועות ב-OS ובחבילות + **network reachability** | דרך **SSM Agent** |
| **Container Images ב-ECR** | פגיעויות בחבילות | סריקה **בזמן ה-push** |
| **Lambda Functions** | פגיעויות בקוד ובתלויות | סריקה **בזמן ה-deploy** |

- מסתמך על מסד **CVE**; לכל ממצא מוצמד **risk score** לתעדוף.
- סריקה **רציפה, רק כשצריך** — לא סריקה תקופתית מיותרת.
- ממצאים → **Security Hub** ו-**EventBridge**.

### 2ד.3 Amazon Macie — גילוי דאטה רגיש

- שירות מנוהל ל-**data security ו-data privacy**.
- משתמש ב-ML ו-pattern matching כדי לגלות **דאטה רגיש ב-S3**.
- הפוקוס: **PII** — תעודות זהות, כרטיסי אשראי, פרטי לקוחות.
- מתריע דרך **EventBridge**.
- מגבלה חשובה: **S3 בלבד**. לא EBS, לא RDS, לא DynamoDB.

### 2ד.4 AWS Security Hub

- **מרכז findings**, לא detector בפני עצמו.
- מאגד ממצאים מ-GuardDuty, Inspector, Macie, IAM Access Analyzer, Firewall Manager ומכלים חיצוניים.
- מריץ **security standards** (CIS, PCI DSS, AWS Foundational Security Best Practices).
- נותן תמונה אחת לכל הארגון.
- **מלכודת:** הפעלת Security Hub לבדה לא מזהה כלום — היא רק מציגה. צריך שה-detectors עצמם יהיו פעילים.

---

## 3. 🔍 פירוק מפורט

### 3.1 מגבלות ומספרים שכדאי לזכור

| פריט | ערך |
|---|---|
| KMS Symmetric | AES-256 |
| KMS API — הצפנה ישירה | עד **4 KB** (מעבר לזה: Envelope Encryption) |
| AWS-Managed key rotation | כל שנה, אוטומטי |
| Parameter Store Standard | 10,000 פרמטרים, ערך עד **4 KB**, חינם |
| Parameter Store Advanced | 100,000 פרמטרים, ערך עד **8 KB**, בתשלום, תומך policies |
| WAF IP Set | עד **10,000 IP** ל-set |
| CloudHSM | **FIPS 140-2 Level 3**, single-tenant |
| ACM public certificate | חינם, חידוש אוטומטי **60 יום** לפני פקיעה |
| ACM imported certificate | התראות **יומיות מ-45 יום** לפני פקיעה |
| GuardDuty | 30 יום ניסיון |

### 3.2 מה מוצפן by default ומה לא

| שירות | מצב ברירת מחדל |
|---|---|
| S3 | **SSE-S3 by default** (הצפנה בצד השרת בכל bucket חדש) |
| CloudWatch Logs | מוצפן by default; ניתן להחליף ל-CMK |
| EBS | **לא מוצפן by default** (אלא אם הופעל encryption by default ב-account/Region) |
| RDS | לא מוצפן by default; **חייבים להפעיל בזמן היצירה** |
| DynamoDB | מוצפן at rest by default |
| SQS / SNS | SSE זמין; ברירת מחדל תלויה בהגדרה |

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| KMS — CMK | חיוב חודשי קבוע לכל מפתח (גם imported) | AWS Owned/Managed keys — **חינם** |
| KMS — API calls | לכל 10,000 קריאות | `GenerateDataKey`/`Decrypt` בקצב גבוה מצטבר |
| KMS — Multi-Region Key | כל replica מחויב **כמפתח נפרד** | |
| CloudHSM | לפי **שעת HSM** | **אין free tier**; HA דורש עוד HSM = כפול |
| ACM — public certs | **חינם** | Private CA בתשלום |
| Parameter Store Standard | **חינם** | הבחירה הזולה ביותר לקונפיגורציה |
| Parameter Store Advanced | חיוב חודשי לכל פרמטר + לכל API interaction | |
| Secrets Manager | **חיוב חודשי לכל secret** + חיוב לכל 10,000 קריאות API | תמיד יקר מ-Parameter Store Standard |
| WAF | לכל **Web ACL**, לכל **rule**, ולכל מיליון **requests** | managed rule groups מחויבים בנפרד |
| Shield Standard | **0** | אוטומטי לכולם |
| Shield Advanced | **מנוי חודשי קבוע גבוה לכל הארגון**, בהתחייבות שנתית, + data transfer | מכסה את כל החשבונות ב-Organization |
| Firewall Manager | לכל policy לכל Region | דורש Organizations + Config מופעל |
| Network Firewall | לכל **endpoint לשעה** + לכל GB שנבדק | ה-endpoint לשעה הוא הרכוב הקבוע |
| GuardDuty | לפי **נפח** CloudTrail events, VPC Flow Logs ו-DNS logs שנבדקו | תוספות (EKS/RDS/EBS/Lambda) מחויבות בנפרד |
| Inspector | לפי **instance/image/function** שנסרק | סריקה רציפה = חיוב מתמשך |
| Macie | לפי buckets שנבדקו + **GB שנסרק** | סריקת דאטה-לייק שלם = יקר מאוד |
| Security Hub | לפי **security checks** ולפי **findings ingested** | מתפוצץ בארגון עם הרבה חשבונות |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| AWS Managed Key | 0 | אין דרישות key policy או cross-account |
| Customer Managed Key | נמוכה אך לא אפסית | key policy, audit מדויק, cross-account, rotation נשלט |
| CloudHSM | **היקרה ביותר בקטגוריה** | דרישת FIPS 140-2 L3 עם single tenancy, TDE של Oracle, שליטה מלאה |
| Parameter Store Standard | **0** | קונפיגורציה וסודות פשוטים בלי rotation |
| Secrets Manager | לכל secret, לכל חודש | כשצריך **rotation אוטומטי** או אינטגרציית RDS |
| Shield Standard | 0 | רוב האפליקציות |
| Shield Advanced | הכי יקר בשכבת ההגנה | ארגון שנתקף תדיר, או שצריך הגנת חשבונית ו-SRT |

### 🚩 עלויות נסתרות

- **קריאות KMS בלולאה** — פונקציה שקוראת `Decrypt` בכל invocation במקום לשמור data key ב-cache.
- **Multi-Region Key ב-4 Regions** = 4 חיובי מפתח, לא אחד.
- **Secrets Manager עם secret לכל microservice לכל סביבה** — מאות secrets מצטברים.
- **Macie על bucket של פטה-בייטים** — התמחור לפי GB שנסרק, לא לפי bucket.
- **GuardDuty עם S3 Data Events מופעל** על bucket עמוס — קופץ בסדרי גודל.
- **Network Firewall** — חיוב לשעה לכל endpoint, גם ב-0 תעבורה, וגם צריך endpoint ל-AZ.
- **Shield Advanced** הוא התחייבות שנתית — לא מנוי שמבטלים בחודש הבא.

### 💡 טיפים לחיסכון

- להשתמש ב-**AWS Managed Keys** כברירת מחדל; CMK רק כשיש סיבה מוגדרת.
- **Envelope Encryption + caching של data key** — במקום קריאת KMS לכל אובייקט.
- קונפיגורציה שאינה סוד → **Parameter Store Standard** (חינם), לא Secrets Manager.
- אם צריך סוד ב-Secrets Manager אבל גישה נוחה — לקרוא אותו דרך `/aws/reference/secretsmanager/` ב-Parameter Store.
- **Shield Standard + WAF rate-based rules + CloudFront** מכסים את רוב תרחישי ה-DDoS בשבריר מהעלות של Shield Advanced.
- Macie: להגדיר scope לפי prefix ולתזמן sampling, לא סריקה מלאה חוזרת.
- Firewall Manager במקום כללי WAF כפולים ב-40 חשבונות.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 KMS מול CloudHSM

| Feature | **AWS KMS** | **AWS CloudHSM** |
|---|---|---|
| Tenancy | **Multi-Tenant** | **Single-Tenant** |
| תקן | FIPS 140-2 Level 3 | FIPS 140-2 Level 3 |
| סוגי מפתחות ראשיים | AWS Owned / AWS Managed / **Customer Managed** | **Customer Managed בלבד** |
| יכולות | Symmetric, Asymmetric, Digital Signing | Symmetric, Asymmetric, Signing **+ Hashing** |
| נגישות המפתח | זמין בכמה Regions, אך **לא יוצא מה-Region שבו נוצר** | פרוס ומנוהל **ב-VPC**; ניתן לשיתוף בין VPCs דרך Peering |
| האצה קריפטוגרפית | אין | **SSL/TLS Acceleration**, **Oracle TDE Acceleration** |
| הרשאות | **AWS IAM** | **אתה יוצר משתמשים ומנהל הרשאות בתוך ה-HSM** |
| High Availability | שירות מנוהל של AWS | **מוסיפים HSMs ב-AZs שונים** |
| Audit | CloudTrail + CloudWatch | CloudTrail + CloudWatch + **MFA support** |
| Free Tier | **כן** | **לא** |

> [!info] שורה תחתונה
> KMS הוא ברירת המחדל לכל דבר. עוברים ל-CloudHSM רק כשהרגולציה דורשת **single tenancy ושליטה מלאה במפתחות ללא AWS**, או כשצריך Oracle TDE / SSL acceleration. אפשר גם לשלב: **KMS Custom Key Store מעל CloudHSM** — נוחות KMS עם חומרה ייעודית.

### 5.2 Parameter Store מול Secrets Manager

| קריטריון | **SSM Parameter Store** | **AWS Secrets Manager** |
|---|---|---|
| מטרה | configuration **וגם** סודות | **סודות בלבד** |
| **Rotation אוטומטי** | **❌ אין** (אפשר לבנות ידנית עם Lambda + EventBridge) | **✅ מובנה, כל X ימים, עם Lambda** |
| אינטגרציית RDS/Aurora | ❌ | **✅ native — מחליף את הסיסמה גם ב-DB** |
| **עלות** | **Standard: חינם.** Advanced: בתשלום לפרמטר | **בתשלום לכל secret לכל חודש** + API calls |
| הצפנה | KMS (סוג `SecureString`) | KMS — **תמיד** |
| היררכיה ונתיבים | ✅ `GetParametersByPath` | תגיות, בלי היררכיית path אמיתית |
| Versioning | ✅ | ✅ (עם staging labels) |
| Multi-Region | לא מובנה | **✅ Multi-Region Secrets עם promote ל-standalone** |
| גודל ערך | 4 KB / 8 KB | גדול יותר |
| יצירת סוד אקראי | ❌ | ✅ |

> [!info] שורה תחתונה
> אם בשאלה מופיעה המילה **rotation** או **RDS credentials** → Secrets Manager. אם מופיעה **"cost-effective"** לאחסון קונפיגורציה או סוד פשוט → Parameter Store Standard.

### 5.3 WAF מול Shield Standard מול Shield Advanced מול Firewall Manager

| קריטריון | **WAF** | **Shield Standard** | **Shield Advanced** | **Firewall Manager** |
|---|---|---|---|---|
| מה עושה | סינון בקשות HTTP | הגנת DDoS בסיסית | הגנת DDoS מתקדמת | **ניהול מרוכז** של כל הנ"ל |
| שכבה | **7** | 3/4 | 3/4/**7** | — |
| היקף | resource בודד | חשבון (אוטומטי) | Organization | **כל ה-Organization** |
| עלות | לפי ACL/rules/requests | **חינם** | מנוי קבוע יקר | לפי policy/Region |
| חוסם SQLi/XSS | **✅** | ❌ | דרך WAF | דרך WAF |
| חוסם SYN flood | לא ישירות | **✅** | ✅ | — |
| DDoS Response Team | ❌ | ❌ | **✅ 24/7** | ❌ |
| הגנת חשבונית | ❌ | ❌ | **✅** | ❌ |
| חל אוטומטית על משאבים חדשים | ❌ | ✅ | ✅ | **✅ — זה הייחוד שלו** |

> [!info] שורה תחתונה
> הגנה גרעינית על משאב בודד → **WAF לבד**. WAF על פני הרבה חשבונות + אכיפה על משאבים עתידיים → **Firewall Manager**. נתקפים תדיר וצריך צוות תגובה והגנת חשבונית → **Shield Advanced**. שלושתם עובדים **יחד**.

### 5.4 GuardDuty מול Inspector מול Macie

| קריטריון | **GuardDuty** | **Inspector** | **Macie** |
|---|---|---|---|
| השאלה | "האם מישהו **תוקף** אותי עכשיו?" | "האם יש לי **חולשה** ידועה?" | "האם יש לי **דאטה רגיש** חשוף?" |
| **מקורות הנתונים** | CloudTrail Events + Management + S3 Data Events, **VPC Flow Logs**, **DNS Logs**; אופציונלי: EKS, RDS, EBS, Lambda | **EC2** (דרך SSM Agent), **ECR images**, **Lambda functions** | **S3 בלבד** |
| מה מוצא | קריאות API חריגות, IP זדוני, **crypto mining**, exfiltration דרך DNS | **CVEs** בחבילות, network reachability, risk score | **PII** — ת"ז, כרטיסי אשראי, מידע אישי |
| דורש agent | **לא** | **כן** ל-EC2 (SSM Agent) | לא |
| טכניקה | ML + anomaly detection + threat intel | מסד CVE + ניתוח רשת | ML + pattern matching |
| מתי רץ | **רציף** | רציף / בזמן push/deploy | לפי scope שהוגדר |

> [!info] שורה תחתונה
> **התנהגות** = GuardDuty. **חולשות בקוד/OS** = Inspector. **תוכן הדאטה ב-S3** = Macie. שלושתם מזינים את **Security Hub**, שהוא רק המרכזייה.

### 5.5 מתי כל אחד — טבלת החלטה מהירה

| הצורך | השירות |
|---|---|
| ניהול מפתחות הצפנה מנוהל | **KMS** |
| single-tenant hardware, FIPS L3, שליטה מלאה | **CloudHSM** |
| תעודת HTTPS ל-ALB/CloudFront/API GW | **ACM** |
| אחסון קונפיגורציה חינם עם היררכיה | **Parameter Store** |
| סיסמת DB שמתחלפת אוטומטית | **Secrets Manager** |
| חסימת SQLi / XSS / מדינה / rate limit | **WAF** |
| DDoS מתקדם + צוות תגובה | **Shield Advanced** |
| אכיפת כללים על כל החשבונות והמשאבים העתידיים | **Firewall Manager** |
| בדיקת תעבורה עמוקה על VPC שלם, IPS | **Network Firewall** |
| זיהוי פעילות חשודה בחשבון | **GuardDuty** |
| CVE ב-EC2/ECR/Lambda | **Inspector** |
| PII ב-S3 | **Macie** |
| תצוגה אחת לכל הממצאים + תקני compliance | **Security Hub** |

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | אבטחה שמנוהלת כקוד ולא ידנית | Firewall Manager לכללי WAF/SG בכל הארגון; Config rule ל-`acm-certificate-expiration-check`; Security Hub כמסך אחד |
| Security | defense in depth בכל השכבות | TLS דרך ACM, at-rest ב-KMS, client-side ב-Encryption SDK לשדות רגישים, key policies מצומצמות, rotation ב-Secrets Manager |
| Reliability | הגנה שלא מפילה תעבורה לגיטימית | WAF ב-Count mode לפני Block; MRK כדי שפענוח לא ייפול על Region שנפל; CloudHSM cluster על כמה AZs |
| Performance Efficiency | להצפין בלי לשלם ב-latency | Envelope Encryption + caching של data key; MRK לפענוח מקומי; CloudHSM SSL/TLS acceleration; CloudFront מקדים את WAF |
| Cost Optimization | לשלם רק על מה שבאמת נדרש | AWS Managed Keys ברירת מחדל; Parameter Store Standard במקום Secrets Manager; Shield Standard + rate-based rules במקום Advanced |
| Sustainability | פחות סריקות ופחות חישוב מיותר | Macie בסקופ מוגדר; Inspector על משאבים פעילים בלבד; חסימה בקצה (CloudFront/WAF) חוסכת compute ב-backend |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "rotate database password automatically" | **Secrets Manager** |
| "store configuration, most cost-effective" | **Parameter Store Standard** |
| "TTL / expiration on a stored parameter" | **Parameter Store Advanced + Parameter Policy** |
| "single-tenant, FIPS 140-2 Level 3, we manage keys" | **CloudHSM** |
| "encrypt in one Region, decrypt in another without re-encrypting" | **KMS Multi-Region Keys** |
| "share encrypted snapshot with another account" | CMK + **KMS Key Policy** + copy ביעד |
| "free TLS certificate with automatic renewal" | **ACM** (public) |
| "certificate for Edge-Optimized API Gateway" | ACM ב-**us-east-1** |
| "SQL injection / XSS / block a country / rate limit" | **WAF** |
| "fixed IP address but I need WAF" | **Global Accelerator + ALB + WAF** |
| "SYN flood / UDP reflection, free" | **Shield Standard** |
| "24/7 response team, cost protection during attack" | **Shield Advanced** |
| "apply WAF rules to all accounts and future resources" | **Firewall Manager** |
| "inspect all VPC traffic, block domains, IPS" | **Network Firewall** |
| "crypto mining on my EC2 / unusual API calls" | **GuardDuty** |
| "CVE in my container images / OS packages" | **Inspector** |
| "find credit card numbers in S3" | **Macie** |
| "single pane of glass for all security findings" | **Security Hub** |
| "protect a field even from the DBA" | **Client-side encryption** (Encryption SDK / DynamoDB Encryption Client) |

### טעויות נפוצות

> [!warning] מלכודת — KMS מול Secrets Manager
> **הניסוח:** "אחסון והחלפה אוטומטית של credentials ל-RDS."
> **הטעות:** לבחור KMS כי מדובר בהצפנה.
> **הנכון:** KMS מנהל **מפתחות**, לא סודות. הוא מצפין את הסוד — אבל **מי ששומר ומסובב** את הסוד הוא **Secrets Manager**. שניהם עובדים יחד.

> [!warning] מלכודת — WAF נגד DDoS volumetric
> **הניסוח:** "התקפת SYN flood בנפח 500 Gbps."
> **הטעות:** WAF.
> **הנכון:** WAF הוא **שכבה 7** בלבד — הוא לא רואה SYN floods. זו **Shield** (Standard לבסיסי, Advanced למתוחכם). WAF כן עוזר נגד **application-layer** floods דרך rate-based rules.

> [!warning] מלכודת — WAF על NLB
> **הניסוח:** "צריך IP סטטי ל-whitelist של לקוחות, וגם סינון SQLi."
> **הטעות:** NLB + WAF.
> **הנכון:** **WAF לא נתמך על NLB** (שכבה 4). הפתרון: **Global Accelerator** (IP קבוע) → **ALB** → WAF.

> [!warning] מלכודת — GuardDuty כסורק פגיעויות
> **הניסוח:** "לזהות חבילות עם CVE ידוע בשרתי ה-EC2."
> **הטעות:** GuardDuty.
> **הנכון:** GuardDuty מנתח **התנהגות** (לוגים), לא את מה שמותקן על השרת. סריקת CVE = **Inspector** (עם SSM Agent).

> [!warning] מלכודת — Security Hub כ-detector
> **הניסוח:** "הפעלנו Security Hub, האם נזהה crypto mining?"
> **הטעות:** כן.
> **הנכון:** Security Hub **מרכז ממצאים בלבד**. בלי GuardDuty פעיל, לא יגיע ממצא של crypto mining. Security Hub מוסיף ערך רק כשה-detectors מופעלים מתחתיו.

> [!warning] מלכודת — MRK הם "global"
> **הניסוח:** "יש לי multi-Region key, אז המפתח קיים בכל AWS."
> **הטעות:** להתייחס אליו כשירות global.
> **הנכון:** יש **Primary + Replicas** ב-Regions שבחרת בלבד, וכל אחד **מנוהל בנפרד** (key policy עצמאית וחיוב עצמאי). ב-**S3 Replication** הם אפילו נחשבים למפתחות עצמאיים לגמרי.

> [!warning] מלכודת — Config Rule מול Firewall Manager
> **הניסוח:** "לוודא שכל ALB חדש בכל חשבון בארגון מקבל את כללי ה-WAF."
> **הטעות:** AWS Config rule.
> **הנכון:** Config **מזהה** אי-ציות; **Firewall Manager** באמת **מחיל את הכללים** אוטומטית על משאבים וחשבונות חדשים.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** פלטפורמת fintech ב-3 Regions. חובות: HTTPS מקצה לקצה, הצפנה at-rest עם מפתחות שנשלטים על ידי הארגון, סיסמאות DB שמתחלפות כל 30 יום, הגנה מפני SQLi ו-DDoS, גילוי אוטומטי של תעודות זהות שנשמרו בטעות ב-S3, ותצוגה אחת לכל הממצאים ב-25 חשבונות.

```text
                       ┌──────────────── AWS Organizations ────────────────┐
Users → Route 53 → CloudFront (+ WAF: SQLi/XSS/geo/rate-based)
                        ↓
                 Global Accelerator → ALB (+ WAF, ACM cert)
                        ↓
                 ASG (EC2) ──→ RDS Aurora Global (KMS CMK, מוצפן)
                        │            ↑
                        │      Secrets Manager (rotation 30d, Multi-Region replica)
                        │
             KMS Multi-Region Keys (Primary us-east-1 + Replicas)
                        │
Detection:  GuardDuty + Inspector + Macie ──findings──→ Security Hub (Delegated Admin)
Governance: Firewall Manager (WAF + SG policies על כל חשבון חדש)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| ACM public certs על CloudFront ו-ALB | חינם, חידוש אוטומטי, אפס תחזוקה; ל-CloudFront התעודה ב-us-east-1 |
| **KMS Multi-Region Keys** ולא CMK נפרד לכל Region | Aurora Global + client-side encryption של שדות רגישים; פענוח מקומי בלי cross-Region API |
| **Secrets Manager** ולא Parameter Store | דרישת rotation כל 30 יום + אינטגרציה native עם Aurora; Multi-Region replica ל-DR |
| WAF גם על CloudFront וגם על ALB | חסימה בקצה חוסכת compute; ה-ALB מוגן גם מפני מי שעוקף את ה-CDN |
| **Global Accelerator** לפני ה-ALB | לקוחות ארגוניים דורשים IP סטטי ל-whitelist, ו-WAF לא נתמך על NLB |
| **Shield Standard + rate-based rules** בשלב ראשון | מכסה את רוב התרחישים; Shield Advanced רק אם יש היסטוריית תקיפות |
| **Firewall Manager** | 25 חשבונות — אכיפה על משאבים וחשבונות **עתידיים**, לא רק על מה שקיים |
| **Macie** על ה-buckets של הלקוחות | דרישת PII ספציפית ל-S3 |
| **Security Hub** כ-delegated admin | מסך אחד + תקני CIS/PCI DSS לאודיט |

**למה לא CloudHSM?** אין דרישת single-tenancy או FIPS L3 ייעודית, והעלות גבוהה משמעותית. אם הרגולטור יידרוש — עוברים ל-**KMS Custom Key Store מעל CloudHSM** בלי לשנות את האפליקציה.

**למה לא Parameter Store לסודות?** הוא חינם, אבל **אין בו rotation אוטומטי**. הדרישה של 30 יום מכריעה את ההחלטה.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- מזהי finding types של GuardDuty או rule IDs של WAF.
- תחביר JSON מלא של KMS Key Policy (רק להבין שהיא הכרחית).
- פרטי CloudHSM Client Software וניהול משתמשים ב-HSM.
- רשימת אלגוריתמים נתמכים ב-KMS מעבר ל-AES-256 / RSA / ECC.
- מחירים מדויקים בדולרים — מספיק לדעת **מה חינם** (Shield Standard, ACM public, Parameter Store Standard, AWS Managed Keys) ומה יקר.
- תצורות מתקדמות של Network Firewall Suricata rules.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **KMS = מפתחות. Secrets Manager = סודות. הם לא מחליפים זה את זה.**
- KMS: AWS Owned/Managed **חינם**; Customer Managed **בתשלום חודשי** + חיוב לכל API call.
- **KMS Key Policy הכרחית** — בלעדיה אין גישה, גם עם IAM policy מושלמת.
- KMS API מצפין ישירות **עד 4KB**; מעבר לזה → **Envelope Encryption**.
- Rotation: AWS-Managed = אוטומטי כל שנה; Customer-Managed = צריך להפעיל; Imported = **ידני עם alias**.
- מפתח KMS **לא יוצא מה-Region** — העתקת snapshot בין Regions עוברת **ReEncrypt**.
- **MRK = Primary + Replicas**, לא global; כל אחד מנוהל בנפרד.
- **CloudHSM** = single-tenant, FIPS 140-2 L3, **אין free tier**, אתה מנהל משתמשים ומפתחות, HA דרך AZs.
- **ACM public = חינם + חידוש אוטומטי**; imported = **אין חידוש**; Edge-Optimized API GW → תעודה ב-**us-east-1**.
- **Parameter Store Standard חינם** (10K, 4KB); **Advanced** (100K, 8KB, policies) בתשלום.
- **Secrets Manager = rotation + RDS native + Multi-Region**, ובתשלום לכל secret.
- **WAF = שכבה 7** על ALB / API GW / CloudFront / AppSync / Cognito — **לא NLB**.
- WAF IP Set = עד **10,000 IP**; rate-based rule = הגנת DDoS בסיסית.
- **Shield Standard חינם ואוטומטי**; **Advanced** = מנוי יקר + SRT 24/7 + הגנת חשבונית + WAF אוטומטי לשכבה 7.
- **Firewall Manager** = אכיפה על **כל החשבונות ועל משאבים עתידיים**.
- **GuardDuty** = לוגים (CloudTrail + VPC Flow + DNS). **Inspector** = EC2/ECR/Lambda + CVE. **Macie** = PII ב-**S3 בלבד**.
- **Security Hub מרכז בלבד** — הוא לא מזהה כלום לבד.

---

## 11. ✅ בדיקת הבנה

1. צריך להעתיק EBS snapshot מוצפן לחשבון אחר. מה חייב להיות נכון לגבי המפתח, ומה השלבים?
2. אפליקציה דורשת IP סטטי ל-whitelist של שותפים, וגם סינון SQL injection. מה הארכיטקטורה?
3. מתי בוחרים Parameter Store Advanced ולא Secrets Manager, ומתי ההפך?
4. הפעלנו Security Hub ולא מגיעים ממצאים על crypto mining. למה?
5. יש Aurora Global Database בשלושה Regions, ורוצים להצפין client-side שדה SSN כך שגם DBA לא יוכל לקרוא אותו, בלי latency חוצה-Regions. מה הפתרון?
6. ארגון עם 30 חשבונות רוצה שכל ALB חדש יקבל אוטומטית את אותם כללי WAF. Config rule או Firewall Manager?

<details>
<summary>תשובות</summary>

1. ה-snapshot חייב להיות מוצפן ב-**Customer Managed Key** (AWS-managed key אי אפשר לשתף). מצמידים **KMS Key Policy** שמאשרת cross-account, משתפים את ה-snapshot, ובחשבון היעד **מעתיקים** אותו ומצפינים ב-CMK מקומי, ואז יוצרים ממנו volume.
2. **Global Accelerator** (נותן IPv4 סטטי) → **ALB** → **WAF Web ACL** באותו Region של ה-ALB. לא NLB, כי **WAF לא נתמך על NLB**.
3. **Parameter Store Advanced** — כשצריך יותר מ-10,000 פרמטרים, ערכים עד 8KB, או **Parameter Policies** (TTL/expiration notification), והעלות עדיין נמוכה משמעותית. **Secrets Manager** — כשצריך **rotation אוטומטי**, יצירת סוד אקראי, אינטגרציה native עם RDS/Aurora, או Multi-Region secret.
4. **Security Hub הוא אגרגטור בלבד.** זיהוי crypto mining מגיע מ-**GuardDuty**, שצריך להיות מופעל (בכל Region רלוונטי) כדי שהממצא בכלל ייווצר ויגיע ל-Hub.
5. **KMS Multi-Region Keys**: Primary באחד ה-Regions ו-Replicas בשניים האחרים. מצפינים את ה-SSN client-side עם **AWS Encryption SDK**, Aurora Global משכפל את הערך **המוצפן**, וכל לקוח מפענח מול ה-MRK **המקומי** — latency נמוך, ו-DBA רואה רק ciphertext.
6. **Firewall Manager**. Config rule רק **מסמן** NON_COMPLIANT; Firewall Manager באמת **מחיל** את ה-Security Policy על משאבים חדשים ועל חשבונות שיצטרפו בעתיד ל-Organization.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[03 - IAM Fundamentals]] · [[04 - IAM Advanced and Organizations]] · [[11 - VPC Security]] · [[17 - S3 Security and Data Management]] · [[15 - CloudFront and Global Delivery]] · [[31 - Monitoring and Logging]] · [[08 - Elastic Load Balancing]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 11692–12721, 14660–14710
