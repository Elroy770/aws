---
lesson: 11
title: VPC Security
domain: Design Secure Architectures
services: [Security Groups, Network ACL, VPC Flow Logs, AWS Network Firewall, CloudWatch Logs]
tags: [saa-c03, networking, vpc, security]
---

# 11 — VPC Security

> [!abstract] בשורה אחת
> **Security Group הוא stateful ומכיל allow בלבד, NACL הוא stateless ומכיל גם deny** — וכל שאלת "החיבור לא עובד" נפתרת בהבנה מה משתי השכבות האלה חסם, ובאיזה כיוון.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה שתי שכבות firewall ולא אחת |
| 2 | איך זה עובד | סדר הבדיקה של חבילה, SG, NACL, Default NACL |
| 3 | פירוק מפורט | **Ephemeral Ports**, כללי NACL ל-3-tier, Flow Logs syntax, Network Firewall |
| 4 | עלות | SG/NACL חינם; Flow Logs ו-Firewall כן עולים |
| 5 | השוואות | **טבלת SG מול NACL המלאה** · Flow Logs מול packet capture |
| 6 | Well-Architected | הגנה בשכבות לפי ששת ה-Pillars |
| 7 | מלכודות | לשכוח ephemeral ports ב-NACL, לפתוח `0.0.0.0/0` כדי "לתקן" timeout |
| 8 | Scenario | אבחון תקלת חיבור מקצה לקצה עם Flow Logs |

**מונחי מפתח בשיעור:** `Stateful` · `Stateless` · `Ephemeral Port` · `Rule Number` · `Default NACL` · `Flow Log` · `ACCEPT/REJECT` · `Network Firewall`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- Route Table קובעת **לאן** חבילה יכולה ללכת — אבל לא **מי מותר** לשלוח אותה.
- שכבת הגנה אחת שנשענת על הגדרה אחת היא שברירה: טעות אנוש אחת חושפת הכול.
- צריך גם בקרה **מדויקת ברמת המשאב** (רק ה-app tier ניגש ל-DB) וגם **guardrail רחב**
  שאף אחד לא יכול לעקוף (חסימת IP זדוני לכל ה-subnet).
- כשמשהו לא עובד — צריך **ראיות**, לא ניחושים. מי ניסה, לאיזה פורט, ומה קרה.

### מה השירות פותר

| שכבה | Scope | מה נותנת |
|---|---|---|
| **Security Group** | **ENI / משאב** | בקרה מדויקת: מי מדבר עם המשאב הזה. **allow בלבד, stateful** |
| **NACL** | **Subnet** | guardrail רחב, כולל **deny** מפורש. **stateless** |
| **VPC Flow Logs** | VPC / Subnet / ENI | **ראיות** — metadata של כל flow, כולל ACCEPT/REJECT |
| **AWS Network Firewall** | **VPC שלם** | inspection עמוק Layer 3–7 בכל הכיוונים |

> [!tip] האנלוגיה
> **NACL** הוא השומר בשער השכונה — בודק כל מי שנכנס וכל מי שיוצא, יש לו רשימה שחורה,
> והוא **לא זוכר** שנתן לכם להיכנס לפני חמש דקות.
> **Security Group** הוא המנעול על דלת הדירה — מדויק, מחליט מי נכנס, **וזוכר**
> שאם הזמנתם פיצה — השליח יכול לצאת בחזרה בלי אישור נוסף.

---

## 2. ⚙️ איך זה עובד

### 2.1 המסלול של חבילה — מי בודק קודם

```text
בקשה נכנסת (Inbound)
────────────────────
  Internet
     ↓
  [1] NACL Inbound Rules    ← stateless: חייב כלל מפורש
     ↓
  [2] SG Inbound Rules      ← stateful: יזכור את החיבור
     ↓
  EC2 Instance
     ↓  (התשובה חוזרת)
  [3] SG — מותר אוטומטית    ← **stateful, אין צורך בכלל**
     ↓
  [4] NACL Outbound Rules   ← **stateless — חייב כלל ל-ephemeral ports!**
     ↓
  Internet


בקשה יוצאת (Outbound)
─────────────────────
  EC2 Instance
     ↓
  [1] SG Outbound Rules     ← ברירת מחדל: הכול מותר
     ↓
  [2] NACL Outbound Rules
     ↓
  Internet
     ↓  (התשובה חוזרת)
  [3] NACL Inbound Rules    ← **חייב כלל ל-ephemeral ports!**
     ↓
  [4] SG — מותר אוטומטית    ← stateful
     ↓
  EC2 Instance
```

> [!warning] הנקודה שכל השיעור סובב סביבה
> ב-**SG** התשובה חוזרת **תמיד** בלי כלל נוסף — הוא **stateful**.
> ב-**NACL** התשובה היא חבילה חדשה לכל דבר, והוא **חייב** כלל מפורש שמתיר אותה.
> זה בדיוק המקום שבו נכנסים **Ephemeral Ports** (סעיף 3.1).

### 2.2 Security Group — תזכורת מרוכזת

| מאפיין | ההתנהגות |
|---|---|
| Scope | **ENI / משאב** |
| סוגי כללים | **allow בלבד** — אין deny |
| State | **Stateful** — תעבורת חזרה מותרת אוטומטית |
| הערכת כללים | **כל הכללים נבדקים יחד** לפני ההחלטה |
| ברירת מחדל inbound | **הכול חסום** |
| ברירת מחדל outbound | **הכול מותר** |
| Source אפשרי | CIDR, **Security Group אחר**, prefix list |
| החלה | רק על משאבים ש**מישהו הצמיד** אליהם את ה-SG |
| כמה | SG אחד → הרבה משאבים; משאב אחד → הרבה SGs (**איחוד** הכללים) |

פירוט מלא ב-[[05 - EC2 Fundamentals]], סעיף 3.3.

### 2.3 NACL — Network Access Control List

- **firewall ברמת ה-subnet** — שולט בתעבורה **אל** ה-subnet **וממנו**.
- **NACL אחד לכל subnet.** subnet חדש מקבל אוטומטית את ה-**Default NACL**.
- אבל **NACL אחד יכול לשרת כמה subnets**.

**איך כללי NACL נבדקים:**

| עיקרון | פירוט |
|---|---|
| **מספר כלל** | **1 – 32766**. **מספר נמוך = עדיפות גבוהה** |
| **סדר הבדיקה** | **מלמטה למעלה במספרים** — הכלל **הראשון שמתאים מכריע** ועוצר |
| **הכלל האחרון** | תמיד `*` — **DENY** לכל מה שלא התאים לשום כלל |
| המלצת AWS | להוסיף כללים **בקפיצות של 100**, כדי להשאיר מקום להכנסות באמצע |
| **NACL חדש** | **חוסם הכול** — inbound ו-outbound |

**דוגמה שממחישה את "הראשון שמתאים מנצח":**

```text
כלל #100  ALLOW  10.0.0.10/32
כלל #200  DENY   10.0.0.10/32

→ הכתובת 10.0.0.10 **תותר**, כי כלל 100 נבדק ראשון והוא מכריע.
   כלל 200 לא ייבדק בכלל.
```

> [!tip] היכולת הייחודית של NACL
> **NACL הוא הדרך לחסום כתובת IP ספציפית ברמת ה-subnet.**
> Security Group **לא יכול** לעשות זאת — אין בו deny.
> שאלה שמזכירה "block a specific malicious IP address" → **NACL**, כמעט תמיד.

### 2.4 Default NACL

| מאפיין | ה-Default NACL |
|---|---|
| Inbound | **מתיר הכול** (`100 · All Traffic · 0.0.0.0/0 · ALLOW`) |
| Outbound | **מתיר הכול** |
| שורה אחרונה | `* · All Traffic · 0.0.0.0/0 · DENY` |
| שיוך | כל subnet חדש מקבל אותו אוטומטית |

**המשמעות:** בברירת מחדל, ה-NACL **לא חוסם כלום** — כל ההגנה בפועל מגיעה מה-SG.
זה בכוונה, כדי שהרשת "תעבוד" מהרגע הראשון.

> [!warning] אל תערכו את ה-Default NACL
> **ההמלצה של AWS: לא לגעת ב-Default NACL.** במקום זאת ליצור **NACL מותאם** ולשייך אליו subnets.
> שינוי ה-Default משפיע על כל subnet שלא שויך במפורש למשהו אחר —
> וזו דרך מצוינת להפיל תעבורה שלא התכוונתם אליה.

---

## 3. 🔍 פירוק מפורט

### 3.1 Ephemeral Ports — הלב של ההבדל בין SG ל-NACL

**מה זה בכלל:**

- כדי ששני צדדים יתקשרו, לכל אחד צריך פורט.
- ה-**שרת** מאזין על **פורט קבוע וידוע**: 443, 80, 3306.
- ה-**לקוח** לא. הוא בוחר פורט **זמני ואקראי** מטווח גבוה — זה ה-**ephemeral port**.
- התשובה של השרת חוזרת אל **הפורט הזמני הזה**, לא אל 443.

```text
בקשה:
  Client 11.22.33.44 : 50105   ──────▶   Server 55.66.77.88 : 443
  src port = 50105 (ephemeral)           dst port = 443 (fixed)

תשובה:
  Server 55.66.77.88 : 443     ──────▶   Client 11.22.33.44 : 50105
  src port = 443                         dst port = 50105  ← לכאן חוזרת התשובה!
```

**הטווחים לפי מערכת הפעלה:**

| מערכת | טווח ה-ephemeral ports |
|---|---|
| **IANA / Windows 10 ומעלה** | **49152 – 65535** |
| **הרבה גרעיני Linux** | **32768 – 60999** |
| הטווח הבטוח שנוהגים לפתוח ב-NACL | **1024 – 65535** |

**למה זה משנה:**

| | **Security Group** | **NACL** |
|---|---|---|
| התיר בקשה נכנסת ל-443 | התשובה יוצאת **אוטומטית** | חייב **כלל outbound** ל-`1024-65535` |
| שלח בקשה יוצאת ל-3306 | התשובה נכנסת **אוטומטית** | חייב **כלל inbound** ל-`1024-65535` |

> [!warning] המלכודת הקלאסית ביותר בנושא
> "פתחנו ב-NACL inbound 443 והאתר עדיין לא נטען."
> **הסיבה:** ה-NACL הוא **stateless**, ולכן ה-**תשובה** נחסמת ב-outbound.
> **הפתרון:** להוסיף כלל **outbound** שמתיר `1024-65535` ליעד.
> ב-SG הבעיה הזו **פשוט לא קיימת**.

### 3.2 NACL לארכיטקטורת Web + DB — הכללים המדויקים

זה הדפוס שהמבחן מצייר. Web subnet ציבורי, DB subnet פרטי, MySQL על 3306.

```text
   Client
     │
     ▼
┌────────────────────────────┐        ┌────────────────────────────┐
│ Web Subnet (Public)        │        │ DB Subnet (Private)        │
│ ┌────────── Web-NACL ─────┐│        │┌────────── DB-NACL ───────┐│
│ │ OUT: ALLOW TCP 3306     ││ ─────▶ ││ IN : ALLOW TCP 3306      ││
│ │      → DB Subnet CIDR   ││        ││      ← Web Subnet CIDR   ││
│ │                         ││        ││                          ││
│ │ IN : ALLOW TCP          ││ ◀───── ││ OUT: ALLOW TCP           ││
│ │      1024-65535         ││        ││      1024-65535          ││
│ │      ← DB Subnet CIDR   ││        ││      → Web Subnet CIDR   ││
│ └─────────────────────────┘│        │└──────────────────────────┘│
│   Web Tier EC2             │        │   DB Instance (3306)       │
└────────────────────────────┘        └────────────────────────────┘
```

**ארבעת הכללים, במילים:**

| NACL | כיוון | פורט | לאן / ממי | למה |
|---|---|---|---|---|
| **Web-NACL** | Outbound | **3306** | ל-CIDR של DB subnet | לאפשר לשלוח שאילתה |
| **DB-NACL** | Inbound | **3306** | מ-CIDR של Web subnet | לאפשר לקבל את השאילתה |
| **DB-NACL** | Outbound | **1024-65535** | ל-CIDR של Web subnet | **התשובה** יוצאת ל-ephemeral port |
| **Web-NACL** | Inbound | **1024-65535** | מ-CIDR של DB subnet | **התשובה** נכנסת ל-ephemeral port |

> [!warning] ב-Multi-AZ — כלל לכל subnet
> אם יש `Web-Subnet-A`, `Web-Subnet-B`, `DB-Subnet-A`, `DB-Subnet-B` —
> **צריך כלל לכל CIDR של subnet יעד בנפרד**. NACL מתייחס ל-CIDR, לא ל"שכבה".
> זו אחת הסיבות ש-NACL כבד לתחזוקה ושמעדיפים SG-to-SG.

**ולהשוואה — אותה ארכיטקטורה עם Security Groups בלבד:**

| SG | כלל | זהו |
|---|---|---|
| `sg-web` | Inbound 443 מ-`0.0.0.0/0` | |
| `sg-db` | Inbound 3306 **מ-`sg-web`** | **שני כללים בסך הכול.** אין ephemeral, אין CIDR, אין multi-AZ |

> [!tip] ההיגיון המנחה
> **SG הוא כלי העבודה היומיומי. NACL הוא guardrail.**
> אל תנסו לממש בקרת גישה מפורטת ב-NACL — זה כואב, שביר ולא מתעדכן עם ה-Auto Scaling.

### 3.3 VPC Flow Logs — הראיות

**מה זה תופס:**

- מידע על **תעבורת IP** שנכנסת ויוצאת מ-network interfaces.
- ניתן להפעיל בשלוש רמות: **VPC**, **Subnet**, או **ENI בודד**.
- תופס גם ממשקים **מנוהלים על ידי AWS**: **ELB, RDS, ElastiCache, Redshift, WorkSpaces,
  NAT Gateway, Transit Gateway** ועוד.

**לאן הלוגים הולכים:**

| יעד | מתי בוחרים |
|---|---|
| **CloudWatch Logs** | התראות בזמן אמת, Metric Filters, Contributor Insights |
| **S3** | ארכיון זול, ניתוח ב-**Athena**, retention ארוך |
| **Kinesis Data Firehose** | streaming ל-pipeline או לצד שלישי |

**שדות ה-Syntax — מה יש בכל שורה:**

| שדה | מה זה | למה חשוב |
|---|---|---|
| `version` | גרסת פורמט הרשומה | |
| `account-id` | החשבון | |
| `interface-id` | ה-ENI שדרכו עברה התעבורה | לאתר את המשאב המדויק |
| **`srcaddr`** | **כתובת המקור** | **לזהות מי פונה** — IP בעייתי או תוקף |
| **`dstaddr`** | **כתובת היעד** | לזהות לאן פונים |
| **`srcport`** | פורט המקור | לרוב ephemeral |
| **`dstport`** | **פורט היעד** | **לזהות איזה שירות מנסים להגיע אליו** (22, 3389...) |
| `protocol` | מספר הפרוטוקול (6=TCP, 17=UDP) | |
| `packets` | מספר חבילות | |
| `bytes` | נפח | לזהות exfiltration או תעבורה כבדה |
| `start` / `end` | חלון הזמן | |
| **`action`** | **`ACCEPT` או `REJECT`** | **השדה החשוב ביותר לאבחון** |
| `log-status` | `OK` / `NODATA` / `SKIPDATA` | |

> [!warning] מה Flow Logs **לא** נותן
> **זה לא packet capture.** אין **payload**, אין תוכן הבקשה, אין headers.
> רק **metadata** על ה-flow. לניתוח תוכן צריך **Traffic Mirroring** או פתרון בשכבה גבוהה יותר.

### 3.4 אבחון SG מול NACL באמצעות שדה ACTION

זו הטבלה שהמבחן שואל עליה ישירות. הרעיון: **SG הוא stateful — ולכן הוא אף פעם לא חוסם
תעבורת חזרה.** אם חזרה נחסמה, האשם חייב להיות ה-NACL.

| מה רואים ב-Flow Logs | מי האשם |
|---|---|
| **Inbound REJECT** | **NACL או Security Group** — אי אפשר להכריע מכאן לבד |
| **Outbound REJECT** | **NACL או Security Group** — אי אפשר להכריע מכאן לבד |
| **Inbound ACCEPT, ואז Outbound REJECT** | **NACL בוודאות** |
| **Outbound ACCEPT, ואז Inbound REJECT** | **NACL בוודאות** |

**ההיגיון בשורה אחת:**

- אם הבקשה **נכנסה** בהצלחה (`ACCEPT`), ה-SG כבר רשם את החיבור ויתיר את התשובה אוטומטית.
- ולכן אם **התשובה** נדחתה — רק ה-**NACL ה-stateless** יכול היה לדחות אותה.
- לעומת זאת, `REJECT` **בכיוון אחד בלבד**, בלי ACCEPT שקדם לו — יכול להיות כל אחד מהשניים.

> [!tip] כלל האצבע לזיכרון
> **REJECT אחרי ACCEPT באותו חיבור = NACL.**
> **REJECT לבד = יכול להיות SG או NACL.**

### 3.5 ארכיטקטורות שימוש ב-Flow Logs

```text
1. זיהוי 10 ה-IPs המובילים
   VPC Flow Logs → CloudWatch Logs → CloudWatch Contributor Insights

2. התראה על ניסיונות SSH / RDP
   VPC Flow Logs → CloudWatch Logs → Metric Filter → CloudWatch Alarm → SNS

3. ניתוח ודשבורדים
   VPC Flow Logs → S3 → Amazon Athena → Amazon QuickSight
```

- **Contributor Insights** — מזהה מי התורמים הגדולים ביותר לתעבורה.
- **Metric Filter + Alarm + SNS** — התראה בזמן אמת על דפוס חשוד (למשל REJECT חוזר על פורט 22).
- **Athena על S3** — שאילתות SQL על ארכיון גדול, בעלות נמוכה.
- **CloudWatch Logs Insights** — שאילתות מהירות על מה שנמצא ב-CloudWatch.

### 3.6 הרשאות — נקודה שנשאלת

כדי ש-VPC Flow Logs יוכל לכתוב ל-**CloudWatch Logs**, נדרש **IAM Service Role** עם ההרשאות:

```text
logs:CreateLogGroup
logs:CreateLogStream
logs:PutLogEvents
```

- בלי ה-Role — ה-Flow Log נוצר אבל **לא כותב כלום**, וה-status יראה כשל.
- ליעד **S3** נדרשת **bucket policy** מתאימה במקום Service Role.

### 3.7 הגנת רשת ב-AWS — התמונה המלאה

| כלי | שכבה | מה נותן |
|---|---|---|
| **NACL** | Layer 3/4, subnet | allow + **deny** לפי IP/פורט |
| **Security Group** | Layer 3/4, ENI | allow, stateful |
| **AWS WAF** | **Layer 7** (HTTP/S) | הגנה מבקשות זדוניות: SQLi, XSS, rate limiting |
| **AWS Shield** | Layer 3/4 | הגנת **DDoS**. Standard חינם ואוטומטי; **Advanced** בתשלום |
| **AWS Firewall Manager** | ניהול | **ניהול מרכזי** של כללים על פני **הרבה חשבונות ו-VPCs** |
| **AWS Network Firewall** | **Layer 3 עד 7**, VPC שלם | inspection מלא בכל כיוון |

פירוט על WAF ו-Shield ב-[[32 - Security Services]].

### 3.8 AWS Network Firewall

**מה זה:** firewall מנוהל שמגן על **VPC שלם**, מ-**Layer 3 עד Layer 7**.

**מה הוא יכול לבדוק — בכל כיוון:**

- תעבורה **בין VPCs**
- תעבורה **יוצאת לאינטרנט**
- תעבורה **נכנסת מהאינטרנט**
- תעבורה **אל ומ-Direct Connect ו-Site-to-Site VPN**

**איך הוא בנוי:**

- **מתחת למכסה המנוע הוא משתמש ב-Gateway Load Balancer** — אבל הכול מנוהל על ידי AWS.
- הכללים ניתנים לניהול **מרכזי בין חשבונות** באמצעות **AWS Firewall Manager**.

**היכולות המדויקות:**

| יכולת | דוגמה |
|---|---|
| תומך ב-**אלפי כללים** | סינון של עשרות אלפי כתובות IP |
| **IP ופורט** | חסימת טווחים שלמים |
| **פרוטוקול** | לחסום **SMB** ביציאה החוצה |
| **Stateful domain list** | להתיר יציאה **רק** ל-`*.mycorp.com` או ל-repo מאושר |
| **Regex** | התאמת דפוסים כלליים |
| **פעולות** | `Allow` · `Drop` · `Alert` |
| **Active flow inspection** | הגנה מפני איומים עם יכולות **intrusion prevention** |
| **לוגים** | ל-**S3**, **CloudWatch Logs** או **Kinesis Data Firehose** |

> [!tip] מתי Network Firewall במבחן
> כשהשאלה מבקשת **סינון לפי domain ביציאה** ("allow outbound only to approved domains"),
> **חסימת פרוטוקול** ספציפי, **IPS**, או הגנה על **כל ה-VPC לרבות תעבורת VPN/DX** —
> זה **Network Firewall**. SG ו-NACL לא יודעים לעשות אף אחד מאלה.

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב | חיוב | הערה |
|---|---|---|
| **Security Groups** | **0** | כמה שרוצים |
| **NACLs** | **0** | כמה שרוצים |
| **VPC Flow Logs — הרישום עצמו** | **0** | האיסוף חינם |
| **Flow Logs → CloudWatch Logs** | **ingestion + אחסון** | ה-ingestion הוא בדרך כלל החלק היקר |
| **Flow Logs → S3** | אחסון + requests | **זול משמעותית** מ-CloudWatch לנפחים גדולים |
| **Athena** על ה-logs | לפי **TB שנסרקו** | פרטישן לפי תאריך מקטין דרמטית |
| **AWS Network Firewall** | **שעה לכל endpoint + GB שנבדקו** | הרכיב היקר בשיעור |
| **AWS WAF** | לפי Web ACL, כללים ובקשות | ראו [[32 - Security Services]] |
| **Shield Standard** | **0** | אוטומטי לכולם |
| **Shield Advanced** | מנוי חודשי גבוה | לארגונים עם חשיפת DDoS משמעותית |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| SG-to-SG במקום כללי NACL מורכבים | **0** | תמיד — גם זול וגם פחות שביר |
| Flow Logs ל-**S3** + Athena | נמוכה | ארכיון, ניתוח תקופתי, compliance |
| Flow Logs ל-**CloudWatch Logs** | **גבוהה** בנפח גדול | כשצריך **התראות בזמן אמת** |
| **Flow Logs עם filter** (רק REJECT) | חוסך ~90% מהנפח | לניטור אבטחה. לאבחון מלא צריך ALL |
| **Network Firewall** | **גבוהה** | רק כשבאמת צריך L7/domain filtering/IPS |

### 🚩 עלויות נסתרות

- **Flow Logs ברמת VPC על VPC עמוס** — נפח עצום ל-CloudWatch Logs. זו ההפתעה הנפוצה.
- **retention אינסופי ב-CloudWatch Logs** — ברירת המחדל היא "לנצח". הגדירו retention.
- **שאילתות Athena על נתונים לא מפורטשנים** — סורקות את כל הארכיון בכל שאילתה.
- **Network Firewall endpoint בכל AZ** — חיוב שעתי מוכפל במספר ה-AZs, לפני GB אחד.
- **כפילות רישום** — Flow Logs ברמת VPC **וגם** ברמת subnet על אותה תעבורה.

### 💡 טיפים לחיסכון

- **Flow Logs ל-S3 עם Lifecycle** לארכיון ארוך; CloudWatch רק למה שצריך התראה עליו.
- **סננו ל-`REJECT` בלבד** כשהמטרה היא ניטור אבטחה ולא אבחון תקלות.
- **הגדירו retention** מפורש ב-CloudWatch Log Groups — תמיד.
- **פרטישנו** את הנתונים ב-S3 לפי תאריך לפני שמריצים Athena.
- **אל תפעילו Network Firewall** כשמספיקים SG + NACL. הוא כלי לדרישות ספציפיות.
- **Flow Logs ברמה אחת** — לא VPC וגם subnet וגם ENI על אותה תעבורה.

---

## 5. ⚖️ השוואות מכריעות

### Security Group מול NACL — הטבלה המלאה

| קריטריון | **Security Group** | **NACL** |
|---|---|---|
| **רמת החלה** | **רמת ה-instance / ENI** | **רמת ה-subnet** |
| **סוגי כללים** | **allow בלבד** | **allow וגם deny** |
| **State** | **Stateful** — תעבורת חזרה מותרת **אוטומטית**, ללא קשר לכללים | **Stateless** — תעבורת חזרה **חייבת** כלל מפורש (ephemeral ports) |
| **סדר הערכה** | **כל הכללים נבדקים** ואז מוחלט | **לפי מספר, מהנמוך לגבוה — הראשון שמתאים מנצח** |
| **החלה על משאבים** | רק על משאב ש**מישהו הצמיד** לו את ה-SG | **אוטומטית על כל המשאבים ב-subnet** |
| **ברירת מחדל** | inbound חסום, outbound פתוח | **Default NACL: הכול פתוח.** NACL **חדש**: הכול חסום |
| **Source/Destination** | CIDR, **SG אחר**, prefix list | **CIDR בלבד** |
| **חסימת IP ספציפי** | **בלתי אפשרי** | **כן — זה החוזק שלו** |
| **כמה לכל אובייקט** | הרבה SGs למשאב | **NACL אחד לכל subnet** |
| **עלות** | **0** | **0** |
| **מתי משתמשים** | **ברירת המחדל** לבקרת גישה לאפליקציה | **guardrail** רחב, חסימת IP, compliance |

> [!info] שורה תחתונה
> **SG הוא "מי מותר לו". NACL הוא "מי אסור לו, בלי קשר למה שמישהו הגדיר למטה".**
> החיבור עובר **רק אם שתי השכבות מתירות** — ובנוסף רק אם ה-**Route Table** בכלל מאפשרת להגיע.

### Flow Logs מול Traffic Mirroring

| קריטריון | **VPC Flow Logs** | **Traffic Mirroring** |
|---|---|---|
| מה תופס | **metadata** של flows | **חבילות מלאות** כולל payload |
| עלות | נמוכה יחסית | גבוהה — נפח מלא |
| שימוש | אבחון SG/NACL, ניטור, compliance | ניתוח עמוק, forensics, IDS |
| רמת פירוט | 5-tuple + action + bytes | הכול |

### מתי SG, מתי NACL, מתי Network Firewall

| הדרישה | הכלי |
|---|---|
| "רק ה-app tier ייגש ל-DB" | **Security Group** (SG-to-SG) |
| "לחסום כתובת IP זדונית ספציפית" | **NACL** (deny) |
| "guardrail שאף צוות לא יוכל לעקוף" | **NACL** ברמת ה-subnet |
| "להתיר יציאה רק ל-domains מאושרים" | **AWS Network Firewall** |
| "לחסום SQL injection ב-HTTP" | **AWS WAF** ([[32 - Security Services]]) |
| "הגנת DDoS" | **AWS Shield** |
| "ניהול כללים על 40 חשבונות" | **AWS Firewall Manager** |
| "להבין למה החיבור נכשל" | **VPC Flow Logs** |

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **באבטחת VPC** | פעולה קונקרטית |
|---|---|---|
| **Operational Excellence** | הכללים בקוד, והתקלות ניתנות לאבחון תוך דקות | SG/NACL ב-IaC עם code review; **Flow Logs** מופעלים מראש; runbook: route → NACL → SG; לוגים מרוכזים לחשבון security |
| **Security** | least privilege בכל שכבה, ואף שכבה לא לבדה | **SG-to-SG** במקום CIDR רחב; **NACL deny** ל-IPs חסומים; אף פעם `0.0.0.0/0` ל-DB; **אל תערכו את ה-Default NACL**; Network Firewall/WAF כשנדרש L7 |
| **Reliability** | כלל אבטחה לא מפיל שירות בטעות | לבדוק כללים ב-staging לפני פרודקשן; לזכור ש-NACL **stateless** ודורש ephemeral; לא לשנות Default NACL שמשפיע על subnets לא מכוונים |
| **Performance Efficiency** | ההגנה לא הופכת לצוואר בקבוק | מספר כללים מצומצם וברור; **Flow Logs עם sampling/filter** במקום ALL על VPC ענק; inspection כבד רק בנתיבים שדורשים אותו |
| **Cost Optimization** | משלמים על תובנה, לא על נפח | **SG/NACL חינם — להשתמש בהם קודם**; Flow Logs ל-**S3** לארכיון; retention מוגדר; **Network Firewall רק כשיש דרישה אמיתית** |
| **Sustainability** | פחות עיבוד ואחסון מיותרים | לא לרשום Flow Logs בכמה רמות במקביל; retention קצר לנתונים תפעוליים; כללים פשוטים שדורשים פחות evaluation |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "block a specific IP address" | **NACL** — ל-SG אין deny |
| "deny rule" / "explicitly deny" | **NACL** |
| "applies to all instances in the subnet automatically" | **NACL** |
| "return traffic is automatically allowed" | **Security Group** (stateful) |
| "return traffic must be explicitly allowed" | **NACL** (stateless) → **ephemeral ports** |
| "first rule that matches wins" | **NACL** — הערכה לפי מספר |
| "all rules are evaluated" | **Security Group** |
| "connection established but response never arrives" | **NACL חסר כלל ל-ephemeral ports** |
| "reference another security group as the source" | **SG-to-SG** — הדפוס ל-tier-to-tier |
| "identify which IPs are being rejected" | **VPC Flow Logs**, שדה **`action`** |
| "inbound ACCEPT but outbound REJECT" | **NACL** בוודאות |
| "need the packet contents / payload" | **לא Flow Logs** — Traffic Mirroring |
| "allow outbound only to approved domains" | **AWS Network Firewall** (stateful domain list) |
| "block SMB protocol outbound" | **AWS Network Firewall** |
| "intrusion prevention for the whole VPC" | **AWS Network Firewall** |
| "manage firewall rules across many accounts" | **AWS Firewall Manager** |
| "protect against SQL injection / XSS" | **AWS WAF** |
| "DDoS protection" | **AWS Shield** (Standard/Advanced) |
| "query flow logs with SQL" | **Athena על S3** |
| "real-time alert on repeated SSH attempts" | **CloudWatch Logs → Metric Filter → Alarm → SNS** |

### טעויות נפוצות

> [!warning] מלכודת 1 — לשכוח ephemeral ports ב-NACL
> **הניסוח:** "The NACL allows inbound HTTPS but users still can't load the page."
> **הטעות:** להוסיף עוד כללי inbound.
> **הנכון:** ה-NACL הוא **stateless**. חייבים כלל **outbound** שמתיר `1024-65535`
> כדי שהתשובה תוכל לצאת. ב-SG הבעיה הזו לא קיימת.

> [!warning] מלכודת 2 — לחפש deny ב-Security Group
> **הניסוח:** "Add a deny rule to the security group to block the attacker's IP."
> **הטעות:** להניח ש-SG הוא firewall מלא.
> **הנכון:** **ב-Security Group אין deny — רק allow.** לחסימת IP ספציפי צריך **NACL**.

> [!warning] מלכודת 3 — לפתוח `0.0.0.0/0` כדי "לפתור" timeout
> **הניסוח:** "The app can't reach the database, so we opened 3306 to 0.0.0.0/0."
> **הטעות:** לרדוף אחרי "שיעבוד" במקום להבין מה חסם.
> **הנכון:** לפתוח 3306 **מ-`sg-app`** בלבד. פתיחה לעולם היא כשל אבטחה חמור,
> וגם לא בהכרח פותרת — אולי הבעיה ב-NACL או ב-route.

> [!warning] מלכודת 4 — לערוך את ה-Default NACL
> **הניסוח:** "Modify the default NACL to block the traffic."
> **הטעות:** לחשוב שזה הפתרון הפשוט.
> **הנכון:** **AWS ממליצה לא לגעת ב-Default NACL.** יוצרים **NACL מותאם** ומשייכים אליו subnets.
> שינוי ה-Default משפיע על כל subnet שלא שויך במפורש למשהו אחר.

> [!warning] מלכודת 5 — NACL חדש שחוסם הכול
> **הניסוח:** "We created a custom NACL and associated it — now nothing works."
> **הטעות:** להניח ש-NACL חדש מתנהג כמו ה-Default.
> **הנכון:** **NACL חדש דוחה הכול** — inbound ו-outbound. צריך להוסיף כללים מפורשים לשני הכיוונים.

> [!warning] מלכודת 6 — לצפות ל-payload ב-Flow Logs
> **הניסוח:** "Use VPC Flow Logs to see what data the attacker exfiltrated."
> **הטעות:** לבלבל בין metadata לתוכן.
> **הנכון:** Flow Logs נותן **מי, לאן, איזה פורט, כמה bytes, ACCEPT/REJECT** — **ולא את התוכן**.
> לתוכן צריך **Traffic Mirroring**.

> [!warning] מלכודת 7 — Flow Logs בלי IAM Role
> **הניסוח:** "We created a flow log to CloudWatch but no data appears."
> **הטעות:** לחפש בעיה בהגדרת ה-VPC.
> **הנכון:** נדרש **IAM Service Role** עם `logs:CreateLogGroup`, `logs:CreateLogStream`
> ו-`logs:PutLogEvents`. בלעדיו הרישום נוצר ולא כותב.

> [!warning] מלכודת 8 — NACL כתחליף ל-SG
> **הניסוח:** "Implement all access control at the NACL level for centralized management."
> **הטעות:** להניח ש-subnet-level = פשוט יותר.
> **הנכון:** NACL עובד לפי **CIDR בלבד**, דורש כלל לכל subnet יעד, ודורש טיפול ידני
> ב-ephemeral ports. עם Auto Scaling זה הופך לבלתי ניתן לתחזוקה. **SG-to-SG** הוא הכלי הנכון.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

אפליקציה תלת-שכבתית: ALB ציבורי, EC2 ב-ASG על 8080, RDS MySQL על 3306.
המצב: המשתמשים מקבלים timeout. בנוסף, צוות האבטחה דורש חסימת טווח IP שממנו הגיעו סריקות,
ותיעוד שניתן לתשאול לצורכי compliance.

```text
   Internet
      │ 443
      ▼
┌──────────────────────────────────────┐
│ Public Subnets (2 AZ)                │   Public-NACL
│   ALB    sg-alb: IN 443 ← 0.0.0.0/0  │   + DENY לטווח הסורק (מספר נמוך!)
└──────────────┬───────────────────────┘
               │ 8080
               ▼
┌──────────────────────────────────────┐
│ Private App Subnets (2 AZ)           │   App-NACL
│   EC2 ASG   sg-app: IN 8080 ← sg-alb │
└──────────────┬───────────────────────┘
               │ 3306
               ▼
┌──────────────────────────────────────┐
│ Private Data Subnets (2 AZ)          │   DB-NACL
│   RDS      sg-db: IN 3306 ← sg-app   │
└──────────────────────────────────────┘

   VPC Flow Logs ──▶ CloudWatch Logs (התראות)  +  S3 (ארכיון + Athena)
```

**שלבי האבחון — הסדר הנכון:**

| שלב | מה בודקים | למה בסדר הזה |
|---|---|---|
| 1 | **Route Table** | בלי route אין תקשורת, ושום SG לא יעזור. ראו [[09 - VPC Fundamentals]] |
| 2 | **NACL** — **שני הכיוונים** | הוא stateless; חסר כלל ephemeral ב-outbound יפיל את התשובה |
| 3 | **Security Group** — inbound בלבד | outbound פתוח כברירת מחדל, ו-stateful מטפל בחזרה |
| 4 | **VPC Flow Logs**, שדה `action` | לראות בדיוק היכן ה-`REJECT` ובאיזה כיוון |
| 5 | האם האפליקציה בכלל מאזינה | `connection refused` ≠ בעיית רשת. ראו [[05 - EC2 Fundamentals]] |

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| **SG-to-SG בכל מעבר שכבה** | ב-ASG ה-IPs משתנים כל הזמן. כלל לפי CIDR יישבר או יהיה רחב מדי |
| `sg-alb`: inbound 443 מ-`0.0.0.0/0` | ה-ALB הוא הרכיב היחיד שאמור להיות חשוף |
| `sg-app`: inbound 8080 **מ-`sg-alb`** | ה-EC2 אינו נגיש מהאינטרנט, גם לא בטעות |
| `sg-db`: inbound 3306 **מ-`sg-app`** | ה-DB נגיש רק לשכבת האפליקציה |
| **NACL מותאם** לחסימת טווח הסורק, במספר **נמוך** | ב-NACL **הראשון שמתאים מנצח** — כלל DENY במספר גבוה מ-ALLOW קיים לא יעבוד |
| **לא נוגעים ב-Default NACL** | שינוי שם משפיע על subnets שלא התכוונו אליהם |
| שאר ה-NACLs נשארים **מתירניים** | ה-enforcement המפורט נעשה ב-SG. NACL הוא guardrail בלבד |
| **Flow Logs → CloudWatch Logs** | Metric Filter על `REJECT` בפורטים 22/3389 → Alarm → SNS |
| **Flow Logs → S3** במקביל, עם Lifecycle | ארכיון זול ל-compliance, ניתוח ב-**Athena** ודשבורד ב-**QuickSight** |
| **IAM Service Role** ל-Flow Logs | בלי `logs:CreateLogGroup/CreateLogStream/PutLogEvents` הלוג לא נכתב |
| **Contributor Insights** על ה-log group | לזהות את 10 ה-IPs המובילים ולהזין מהם את רשימת ה-DENY |

**מה שגילינו ב-Flow Logs:**

השורות הראו `Inbound ACCEPT` ב-8080 ואחריו `Outbound REJECT` באותו flow.
לפי הטבלה בסעיף 3.4 — **REJECT אחרי ACCEPT = NACL בוודאות**.
מישהו הידק את ה-App-NACL וחסם את ה-outbound ל-`1024-65535`,
כך שהתשובות ל-ALB לא יכלו לצאת. ה-SG היה תקין לחלוטין.

**למה לא לפתוח `0.0.0.0/0` על 3306 כדי "לבדוק אם זו הבעיה"?**
כי גם אם היה עוזר, זו חשיפה של ה-DB לכל האינטרנט. Flow Logs נותן את אותה תשובה
בלי לפתוח שום דבר — וזו בדיוק המטרה שלו.

**מתי נשקול Network Firewall כאן?**
אם תופיע דרישה כמו "היציאה לאינטרנט מותרת רק ל-domains מאושרים", או חסימת פרוטוקול,
או IPS ל-VPC כולו. SG ו-NACL לא יודעים לעבוד ברמת domain.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **הטווחים המדויקים** של ephemeral ports בכל מערכת הפעלה. מספיק לזכור
  **Windows ≈ 49152–65535**, **Linux ≈ 32768–60999**, ושבפועל פותחים **1024–65535**.
- **הפורמט המלא** של כל שדות ה-Flow Log. מספיק `srcaddr`, `dstaddr`, `srcport`, `dstport`, **`action`**.
- **תחביר כללי Suricata** ב-Network Firewall.
- **המבנה הפנימי** של Gateway Load Balancer שמאחורי Network Firewall.
- **מגבלות quota** מדויקות על מספר כללי SG/NACL — soft limits משתנים.
- **תחביר Athena/CloudWatch Logs Insights** לשאילתות. מספיק לדעת מה מתאים לאיזה יעד.
- **פרטי Traffic Mirroring** — מספיק לדעת שהוא מעתיק חבילות מלאות מ-ENI לניתוח.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **SG = רמת ה-instance/ENI. NACL = רמת ה-subnet.**
- **SG = allow בלבד. NACL = allow וגם DENY.** חסימת IP ספציפי → **רק NACL**.
- **SG = Stateful** — תעבורת חזרה מותרת אוטומטית. **NACL = Stateless** — חייב כלל מפורש.
- **ב-SG כל הכללים נבדקים. ב-NACL הכלל עם המספר הנמוך ביותר שמתאים — מנצח ועוצר.**
- **מספרי כללי NACL: 1–32766.** הכלל האחרון `*` הוא **DENY** לכל השאר. AWS ממליצה קפיצות של **100**.
- **Default NACL מתיר הכול. NACL חדש חוסם הכול.** **אל תערכו את ה-Default.**
- **NACL אחד לכל subnet**, אבל NACL יכול לשרת כמה subnets.
- **Ephemeral ports:** Windows **49152–65535** · Linux **32768–60999** · ב-NACL פותחים **1024–65535**.
- **SG חל רק על משאבים שהוצמד להם. NACL חל אוטומטית על כל ה-subnet.**
- **SG יכול להפנות ל-SG אחר. NACL — CIDR בלבד.**
- **SG ו-NACL — חינם.**
- **Flow Logs:** רמות **VPC / Subnet / ENI**; יעדים **S3, CloudWatch Logs, Kinesis Data Firehose**.
- **Flow Logs תופס גם ממשקים מנוהלים:** ELB, RDS, ElastiCache, Redshift, WorkSpaces, NAT GW, TGW.
- **Flow Logs = metadata בלבד. אין payload.**
- **`action` = ACCEPT / REJECT.** **REJECT אחרי ACCEPT באותו חיבור = NACL בוודאות.**
- **REJECT בכיוון אחד בלבד = יכול להיות SG או NACL.**
- **Flow Logs ל-CloudWatch דורש IAM Service Role** עם `logs:CreateLogGroup`, `CreateLogStream`, `PutLogEvents`.
- **ניתוח:** Athena על S3 · CloudWatch Logs Insights · Contributor Insights ל-Top-10 IPs ·
  Metric Filter → Alarm → SNS להתראות.
- **Network Firewall:** VPC שלם, **L3–L7**, כל כיוון (VPC↔VPC, אינטרנט, **DX ו-VPN**),
  מבוסס **Gateway Load Balancer**, ניהול מרכזי ב-**Firewall Manager**,
  **stateful domain lists**, regex, Allow/Drop/Alert, IPS.
- **סדר האבחון: Route → NACL (שני הכיוונים) → SG (inbound) → Flow Logs → האפליקציה.**

---

## 11. ✅ בדיקת הבנה

1. פתחתם ב-NACL inbound 443 והדף עדיין לא נטען. מה חסר ולמה זה לא קורה ב-SG?
2. ב-NACL יש `#100 ALLOW 10.0.0.10/32` ו-`#200 DENY 10.0.0.10/32`. מה קורה בפועל ולמה?
3. ב-Flow Logs רואים `Inbound ACCEPT` ואחריו `Outbound REJECT`. מי האשם, ואיך אתם יודעים בוודאות?
4. צוות אבטחה דורש לחסום טווח IP תוקף. למה Security Group לא יכול, ומה כן עושים?
5. יצרתם NACL מותאם ושייכתם אותו — ומיד הכול נפל. למה?
6. אילו ארבעה כללי NACL נדרשים כדי ש-Web subnet ידבר עם DB subnet על 3306?
7. יצרתם Flow Log ל-CloudWatch ואין נתונים. מה כנראה חסר?
8. מתי SG ו-NACL כבר לא מספיקים ונדרש AWS Network Firewall?

<details>
<summary>תשובות</summary>

1. חסר כלל **outbound** ב-NACL שמתיר את **ה-ephemeral ports** (`1024–65535`) חזרה ללקוח.
   ה-NACL הוא **stateless** — התשובה היא חבילה חדשה שנבדקת מחדש.
   ב-**SG** זה לא קורה כי הוא **stateful** וזוכר את החיבור, ולכן מתיר את התשובה אוטומטית.
2. הכתובת **תותר**. ב-NACL הכללים נבדקים **לפי מספר, מהנמוך לגבוה**,
   וה-**כלל הראשון שמתאים מכריע ועוצר**. כלל 100 (ALLOW) נבדק לפני 200 — ו-200 לא ייבדק כלל.
   כדי לחסום, כלל ה-DENY חייב לקבל **מספר נמוך יותר**.
3. **NACL, בוודאות.** אם הבקשה נכנסה ב-`ACCEPT`, ה-**Security Group כבר רשם את החיבור**
   ומתיר את תעבורת החזרה אוטומטית (stateful). ולכן היחיד שיכול היה לדחות את התשובה
   הוא ה-**NACL ה-stateless** שחסר לו כלל outbound.
4. כי **ב-Security Group אין כללי deny — רק allow**. לחסימה מפורשת משתמשים ב-**NACL**
   עם כלל **DENY** למספר נמוך מספיק, ברמת ה-subnet.
5. כי **NACL חדש דוחה הכול** — גם inbound וגם outbound. בניגוד ל-**Default NACL** שמתיר הכול.
   צריך להוסיף במפורש כללי allow **בשני הכיוונים**, כולל ephemeral ports.
6. ארבעה: **Web-NACL outbound 3306** ל-CIDR של ה-DB · **DB-NACL inbound 3306** מ-CIDR של ה-Web ·
   **DB-NACL outbound 1024–65535** ל-CIDR של ה-Web (התשובה) ·
   **Web-NACL inbound 1024–65535** מ-CIDR של ה-DB (התשובה).
   ובמערך multi-AZ — כלל לכל CIDR של subnet יעד בנפרד.
7. **IAM Service Role** עם ההרשאות `logs:CreateLogGroup`, `logs:CreateLogStream`
   ו-`logs:PutLogEvents`. בלעדיו ה-Flow Log נוצר אבל לא מצליח לכתוב.
8. כשנדרשת בקרה שהם לא מסוגלים לה: **סינון לפי domain ביציאה**
   (`allow outbound only to *.mycorp.com`), **חסימת פרוטוקול** (למשל SMB),
   **התאמת regex**, **intrusion prevention**, או inspection על **כל ה-VPC כולל תעבורת
   Direct Connect ו-Site-to-Site VPN**. SG ו-NACL עובדים ב-L3/L4 לפי IP ופורט בלבד.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[09 - VPC Fundamentals]] · [[10 - VPC Internet Connectivity]] · [[12 - VPC Private Connectivity]] · [[13 - VPC Network Architecture]] · [[32 - Security Services]] · [[31 - Monitoring and Logging]] · [[05 - EC2 Fundamentals]] · [[08 - Elastic Load Balancing]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 13263–13495, 13707–13849, 14660–14710
