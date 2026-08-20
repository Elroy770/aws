---
lesson: 02
title: AWS Well-Architected Framework
domain: Design Resilient Architectures
services: [AWS Well-Architected Tool, AWS Trusted Advisor, AWS Support]
tags: [saa-c03, architecture, best-practices, pillars, governance]
---

# 02 — AWS Well-Architected Framework

> [!abstract] בשורה אחת
> ששת ה-Pillars הם המילון שבו המבחן כתוב: כל שאלה מסתירה pillar אחד דומיננטי, וזיהוי שלו מצמצם ארבע תשובות לאחת.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה צריך framework ולא רק "ניסיון" |
| 2 | איך זה עובד | General Guiding Principles ותהליך ה-review |
| 3 | פירוק מפורט | כל pillar: הגדרה, עקרונות, שירותים, ניסוחי מבחן |
| 4 | עלות | WA Tool מול Trusted Advisor — מה חינם ומה דורש Support plan |
| 5 | השוואות | WA Tool מול Trusted Advisor, ו-pillars שמתנגשים |
| 6 | Pillars על עצמם | מה כל pillar אומר על תהליך ה-review |
| 7 | מלכודות | ניסוחים שמסגירים איזה pillar נבחן |
| 8 | Scenario | review אמיתי של workload שגדל מהר מדי |
| 9–11 | סיכום ובדיקה | Cheat Sheet ושאלות |

**מונחי מפתח בשיעור:** `6 Pillars` · `Design Principles` · `Well-Architected Tool` · `Trusted Advisor` · `High-Risk Item` · `Game Day` · `Workload`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- "האם הארכיטקטורה שלנו טובה?" היא שאלה שכל אחד עונה עליה אחרת.
- ארכיטקט אחד אומר "מאובטח", השני אומר "יקר מדי", ואין שפה משותפת להכריע.
- החלטות נשענות על תחושת בטן ועל מה שעבד בפרויקט הקודם.
- בעיות מתגלות רק בייצור: אחרי הפריצה, אחרי הנפילה, או אחרי החשבון של סוף החודש.
- אין דרך שיטתית לזהות מה **חסר** בארכיטקטורה — קל לראות מה יש, קשה לראות מה אין.

### מה השירות פותר

- Framework נותן **שפה משותפת** בשישה צירים שכולם מסכימים עליהם.
- הופך שאלות מעורפלות ("זה אמין?") לשאלות מדידות ("מה ה-RTO? מה קורה כשה-AZ נופל?").
- מספק **checklist שיטתי** שמאתר פערים לפני שהם מתפוצצים.
- מאפשר לתעד סיכונים מוכרים ולהחליט במודע לקחת אותם — במקום לגלות אותם בטעות.
- זהו **כלי חשיבה ומתודולוגיה**, לא שירות שמתקן לך את הארכיטקטורה אוטומטית.

> [!tip] האנלוגיה
> זה בדיוק כמו טסט לרכב: לא מישהו שמתקן לך את האוטו, אלא רשימת בדיקות אחידה
> שמבטיחה שאף אחד לא "שכח" לבדוק את הבלמים כי הוא היה עסוק בצבע.

> [!info] נקודה שהמבחן אוהב
> ה-pillars אינם trade-off שצריך "לאזן" ביניהם — AWS מציגה אותם כ-**סינרגיה**.
> אוטומציה משפרת בו-זמנית Operational Excellence, Reliability, Security ו-Cost.

---

## 2. ⚙️ איך זה עובד

### 2.1 General Guiding Principles — עקרונות העל

אלו העקרונות שחוצים את כל ששת ה-Pillars:

| עיקרון | מה זה אומר בפועל | הניגוד שהוא מחליף |
|---|---|---|
| **Stop guessing your capacity needs** | scaling אוטומטי לפי ביקוש אמיתי | לרכוש חומרה לפי תחזית לשלוש שנים |
| **Test systems at production scale** | להרים סביבת בדיקה בגודל מלא, ואז לכבות אותה | לבדוק על שרת בודד ולקוות לטוב |
| **Automate to make experimentation easier** | IaC — לשכפל, לנסות, למחוק, לחזור | כל ניסוי דורש שבוע של הקמות ידניות |
| **Allow for evolutionary architectures** | לתכנן שהארכיטקטורה תשתנה עם הדרישות | "קפאנו את הדיזיין ב-2019" |
| **Drive architectures using data** | להחליט לפי מטריקות מ-CloudWatch ולא לפי דעה | ויכוח בחדר ישיבות ללא נתונים |
| **Improve through game days** | לסמלץ Black Friday ולסמלץ תקלות בכוונה | לגלות את החולשה בדיוק ביום הלחץ |

- שני העקרונות הראשונים אפשריים **רק** בענן — ב-data center פיזי הם לא ריאליים.
- Game Day = תרגיל מכוון שבו שוברים משהו בסביבה מבוקרת כדי לבדוק שהתאוששות עובדת.

### 2.2 תהליך ה-Review

```text
1. הגדרת Workload
   └── מה הגבול? אילו רכיבים? מי הבעלים?
        │
2. איסוף דרישות עסקיות
   └── RTO / RPO / latency / compliance / budget
        │
3. מענה על שאלות ה-Framework
   └── שאלה אחר שאלה, pillar אחר pillar
        │
4. זיהוי סיכונים
   └── High Risk Items (HRI)  ו-  Medium Risk Items
        │
5. תוכנית שיפור
   └── לתעדף לפי סיכון × מאמץ
        │
6. מדידה אחרי היישום
   └── ואז חוזרים ל-3 (review הוא מחזורי, לא אירוע חד-פעמי)
```

- Review אינו חד-פעמי: workload, traffic, מחירים ודרישות משתנים כל הזמן.
- הפלט העיקרי הוא **רשימת סיכונים מתועדת**, לא ציון.

### 2.3 AWS Well-Architected Tool

- כלי **חינמי** בקונסולה שמנחה אותך דרך ה-review.
- איך עובדים איתו:
  1. מגדירים workload (שם, סביבה, Regions, בעלים).
  2. עונים על סדרת שאלות לפי ששת ה-Pillars.
  3. הכלי מסמן אילו תשובות מייצרות סיכון.
  4. מקבלים המלצות עם קישורים לתיעוד ולסרטונים.
  5. מפיקים דוח ורואים תמונת מצב ב-dashboard.
- מאפשר לשמור milestones ולעקוב אחרי שיפור לאורך זמן.

### 2.4 AWS Trusted Advisor

- **לא צריך להתקין כלום** — סורק את החשבון הקיים ומחזיר המלצות מיידיות.
- זו הערכה **ברמת החשבון**, לא ברמת ה-workload הלוגי.

שש קטגוריות הבדיקה:

| קטגוריה | דוגמאות למה שהוא מוצא |
|---|---|
| **Cost Optimization** | instances במצב idle, Elastic IP לא משויך, RI לא מנוצל |
| **Performance** | instances שמוגבלים ב-CPU, קונפיגורציה לא אופטימלית |
| **Security** | Security Group פתוח לכל האינטרנט, MFA חסר ב-root, מפתחות חשופים |
| **Fault Tolerance** | חוסר גיבויים, משאבים ב-AZ יחיד, ELB בלי redundancy |
| **Service Limits** | התקרבות למכסה של שירות (למשל מספר VPCs) |
| **Operational Excellence** | פערים בתהליכים ובתפעול |

- גישה תוכניתית: **AWS Support API**.
- Trusted Advisor יכול לפלוט findings ל-EventBridge — כך בונים אוטומציה שמגיבה לממצא חדש.

---

## 3. 🔍 פירוק מפורט — ששת ה-Pillars

### 3.1 Operational Excellence

- **הגדרה:** היכולת להריץ ולנטר מערכות כדי לספק ערך עסקי, ולשפר תהליכים ונהלים ברציפות.
- **השאלה המרכזית:** האם אנחנו יכולים לשנות את המערכת בבטחה, ולדעת מה קרה כשמשהו נשבר?

**Design Principles:**

- לבצע operations כקוד (Infrastructure as Code).
- לעשות שינויים קטנים, תכופים והפיכים.
- לשפר נהלים באופן קבוע ולתעד runbooks.
- לצפות כשלים מראש ולתרגל אותם.
- ללמוד מכל אירוע תפעולי.

| שירותים אופייניים | תפקיד |
|---|---|
| CloudFormation | תשתית כקוד, סביבות זהות |
| CloudWatch | מטריקות, logs, alarms, dashboards |
| CloudTrail | audit של כל קריאת API |
| Systems Manager | patching, automation, run commands |
| CodePipeline / CodeDeploy | deployment אוטומטי והפיך |

**ניסוחי מבחן טיפוסיים:** "automate deployments", "reduce manual effort", "reproducible environments", "track changes over time".

### 3.2 Security

- **הגדרה:** הגנה על דאטה, מערכות ונכסים, תוך שיפור מתמיד של יכולת ההגנה והזיהוי.
- **השאלה המרכזית:** מי יכול לעשות מה, לאיזו דאטה, ואיך נדע שזה קרה?

**Design Principles:**

- לבנות בסיס זהויות חזק ולפעול לפי least privilege.
- להפעיל traceability — לתעד ולנטר כל פעולה.
- להחיל אבטחה בכל שכבה, לא רק בהיקף.
- לאוטמט best practices של אבטחה.
- להגן על דאטה במנוחה ובתנועה.
- להרחיק אנשים מהדאטה (אוטומציה במקום גישה ידנית).
- להתכונן לאירועי אבטחה מראש.

| שירותים אופייניים | תפקיד |
|---|---|
| IAM | זהויות, roles, policies, MFA |
| KMS | ניהול מפתחות והצפנה |
| Security Groups / NACL / WAF | בקרת רשת ותעבורה |
| GuardDuty / Security Hub | זיהוי איומים וריכוז ממצאים |
| Secrets Manager | ניהול סודות ורוטציה |

**ניסוחי מבחן טיפוסיים:** "least privilege", "encrypt at rest", "who accessed", "no long-term credentials", "audit trail".

### 3.3 Reliability

- **הגדרה:** היכולת של workload לבצע את הפונקציה שלו נכון ובעקביות, ולהתאושש מכשלים.
- **השאלה המרכזית:** מה קורה כשרכיב נופל, וכמה מהר חוזרים לעבוד?

**Design Principles:**

- להתאושש אוטומטית מכשל על בסיס מדדים.
- לבדוק תהליכי recovery בפועל, לא רק בתיאוריה.
- לבצע scale אופקי כדי לצמצם blast radius.
- להפסיק לנחש capacity.
- לנהל שינויים דרך אוטומציה.

| שירותים אופייניים | תפקיד |
|---|---|
| Auto Scaling | החלפת instances כושלים, התאמה לביקוש |
| ELB + Health Checks | הוצאת יעדים לא בריאים מהמערך |
| Multi-AZ (RDS, ASG) | שרידות לכשל data center |
| Route 53 Health Checks | failover ברמת DNS |
| S3 / backups / AWS Backup | שמירה על הדאטה עצמה |

**ניסוחי מבחן טיפוסיים:** "highly available", "fault tolerant", "automatically recover", "RTO / RPO", "no single point of failure".

### 3.4 Performance Efficiency

- **הגדרה:** שימוש יעיל במשאבי מחשוב כדי לעמוד בדרישות, ושמירה על היעילות כשהדרישות משתנות.
- **השאלה המרכזית:** האם בחרנו את סוג המשאב הנכון, בגודל הנכון, במקום הנכון?

**Design Principles:**

- להפוך טכנולוגיות מתקדמות לנגישות (להשתמש ב-managed services).
- להפוך פריסה גלובלית לעניין של דקות.
- להשתמש בארכיטקטורות serverless.
- להתנסות לעיתים קרובות — קל להחליף instance type בענן.
- לגלות אמפתיה מכנית: להתאים את הטכנולוגיה לבעיה (לא כל דבר הוא RDS).

| שירותים אופייניים | תפקיד |
|---|---|
| CloudFront | קירוב תוכן למשתמש |
| ElastiCache | הורדת עומס מה-DB |
| Instance types מתמחים | GPU, compute-optimized, memory-optimized |
| Lambda | scaling מיידי ללא ניהול שרתים |
| Global Accelerator | שיפור נתיב הרשת למשתמשים גלובליים |

**ניסוחי מבחן טיפוסיים:** "reduce latency", "improve read performance", "handle sudden spikes", "right-size".

### 3.5 Cost Optimization

- **הגדרה:** להפיק את הערך העסקי המרבי מכל דולר שמוצא.
- **השאלה המרכזית:** האם אנחנו משלמים על משהו שלא מייצר ערך?

**Design Principles:**

- ליישם ניהול פיננסי בענן (FinOps).
- לאמץ מודל צריכה — לשלם על מה שמשתמשים.
- למדוד יעילות כוללת, לא רק מחיר לשעה.
- להפסיק להוציא על heavy lifting לא-מבדל (לעבור ל-managed).
- לנתח ולייחס הוצאה (tagging, cost allocation).

| שירותים אופייניים | תפקיד |
|---|---|
| Cost Explorer / Budgets | ניתוח והתראות |
| Savings Plans / Reserved Instances | הנחה תמורת התחייבות |
| Spot Instances | קיבולת פנויה בהנחה עמוקה |
| S3 Lifecycle / Intelligent-Tiering | הזזת דאטה לשכבה זולה |
| Auto Scaling | לא לשלם על קיבולת רדומה |

**ניסוחי מבחן טיפוסיים:** "most cost-effective", "minimize cost", "reduce spend without impacting", "steady-state workload".

> [!warning] Cost Optimization אינו "הכי זול"
> downtime עולה כסף. עבודת תפעול עולה כסף. חשבון AWS נמוך עם צוות שנשרף אינו אופטימיזציה.

### 3.6 Sustainability

- **הגדרה:** מזעור ההשפעה הסביבתית של הרצת workloads בענן.
- **השאלה המרכזית:** האם אנחנו משיגים את אותה תוצאה עסקית עם פחות משאבים ואנרגיה?
- ה-pillar השישי והחדש ביותר — נוסף אחרי חמשת הראשונים.

**Design Principles:**

- להבין את ההשפעה ולהגדיר יעדי שיפור.
- למקסם ניצולת — שרת ב-10% ניצול הוא בזבוז.
- לאמץ חומרה ותוכנה יעילות יותר כשהן מגיעות.
- להשתמש בשירותים מנוהלים (AWS מריצה אותם ביעילות גבוהה מכם).
- לצמצם את ההשפעה במורד הזרם (פחות דאטה שנשלחת ללקוח).

| פרקטיקות אופייניות | תפקיד |
|---|---|
| Right-sizing ו-Auto Scaling | פחות משאבים רדומים |
| Graviton / instances יעילים | יותר ביצועים לוואט |
| Serverless | אפס משאבים כשאין תעבורה |
| S3 Lifecycle ומחיקת דאטה מיותרת | פחות אחסון מיותר |
| Caching / CDN | פחות compute ופחות תעבורה |

**ניסוחי מבחן טיפוסיים:** "reduce environmental impact", "minimize resources provisioned", "improve utilization".

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב | מודל חיוב | הערה |
|---|---|---|
| Well-Architected Tool | **חינם** | אין עלות על ה-review עצמו |
| Trusted Advisor — בדיקות בסיסיות | **חינם** לכל חשבון | סט מצומצם, בעיקר Security ו-Service Limits |
| Trusted Advisor — סט מלא | דורש **Business או Enterprise Support** | כאן העלות האמיתית |
| AWS Support API | כלול בתוכניות התמיכה המתאימות | מאפשר אוטומציה של הממצאים |
| יישום ההמלצות | עלות המשאבים עצמם | ה-review חינם, התיקון לא |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| WA Tool בלבד | 0 | תמיד — אין סיבה לא לעשות review |
| Trusted Advisor בסיסי | 0 | תמיד — מזהה SG פתוח ו-MFA חסר |
| Business Support | תוספת חודשית לפי אחוז מהצריכה | כשיש production אמיתי ורוצים את כל הבדיקות |
| Enterprise Support | היקר ביותר | ארגונים גדולים; כולל TAM וליווי |

### 🚩 עלויות נסתרות

- **עלות אי-ביצוע ה-review:** downtime, פריצה, או חשבון מנופח — יקרים מכל Support plan.
- **תיקון High-Risk Item** יכול לדרוש שינוי ארכיטקטוני יקר (מעבר ל-Multi-AZ, למשל).
- **Service Limits** שלא נבדקו: אתה מגלה את המכסה בדיוק כשאתה צריך לגדול.
- Review שנעשה פעם אחת ונשכח — כל העלות, אפס התועלת.

### 💡 טיפים לחיסכון

- להריץ Trusted Advisor לפני כל אופטימיזציה — הוא מוצא את הפירות הנמוכים בחינם.
- לתייג משאבים כדי שניתוח העלויות יהיה בכלל אפשרי.
- להפוך ממצאי Trusted Advisor לאוטומציה דרך EventBridge במקום לבדוק ידנית.
- לעשות review לפני deployment גדול, לא אחריו.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 Well-Architected Tool מול Trusted Advisor

| קריטריון | Well-Architected Tool | Trusted Advisor |
|---|---|---|
| מה נבדק | **workload** לוגי שאתה מגדיר | **החשבון** כפי שהוא כרגע |
| איך | אתה עונה על שאלות | סריקה אוטומטית של המשאבים |
| מבוסס על | ששת ה-Pillars | שש קטגוריות בדיקה |
| דורש התקנה | לא | לא |
| עלות | חינם | בסיסי חינם, סט מלא ב-Business/Enterprise |
| הפלט | דוח סיכונים ותוכנית שיפור | רשימת המלצות קונקרטיות למשאבים |
| מתי בוחרים | תכנון ובחינה ארכיטקטונית | בדיקת היגיינה שוטפת של החשבון |

### 5.2 כשה-pillars מושכים לכיוונים שונים

| מתח | הצד האחד | הצד השני | מי גובר במבחן |
|---|---|---|---|
| Reliability מול Cost | Multi-AZ, replicas | instance יחיד | לפי דרישת ה-SLA/RTO בשאלה |
| Security מול Performance | הצפנה, בדיקות | latency נמוך יותר | Security כמעט תמיד — היא constraint |
| Performance מול Cost | instance גדול יותר | right-sizing | לפי המילה "cost-effective" בשאלה |
| Operational Excellence מול מהירות | אוטומציה ותהליך | deploy ידני מהר | אוטומציה — היא משרתת כמה pillars |

> [!info] שורה תחתונה
> WA Tool = "האם הארכיטקטורה נכונה?". Trusted Advisor = "האם החשבון שלי מסודר עכשיו?".
> וכשיש התנגשות בין pillars — הדרישה העסקית שכתובה בשאלה היא שמכריעה.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

כאן ה-framework מופנה אל עצמו: מה כל pillar דורש **מתהליך ה-review**.

| Pillar | מה זה אומר **בתהליך ה-review** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | Review הוא תהליך חוזר, לא אירוע | לקבוע milestone ב-WA Tool לכל שינוי מהותי |
| Security | Review חושף ממצאי אבטחה רגישים | להגביל גישה לדוחות, ולטפל קודם ב-HRI של Security |
| Reliability | Review שלא נבדק במציאות אינו שווה כלום | לתרגם כל טענת recovery ל-Game Day בפועל |
| Performance Efficiency | תשובות ב-review חייבות להישען על מדידה | לגבות כל תשובה במטריקה מ-CloudWatch ולא בהערכה |
| Cost Optimization | ה-review עצמו חינם — התיקונים לא | לתעדף HRI לפי סיכון חלקי מאמץ ועלות |
| Sustainability | ניצולת נמוכה היא ממצא לגיטימי ב-review | למדוד utilization ולסגור משאבים רדומים כחלק מהתוכנית |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | ה-Pillar המכוון |
|---|---|
| "highly available", "fault tolerant", "automatically recover" | Reliability |
| "least privilege", "encrypt", "audit who did what" | Security |
| "most cost-effective", "minimize spend" | Cost Optimization |
| "reduce latency", "handle traffic spikes", "right-size" | Performance Efficiency |
| "automate deployment", "reduce manual toil", "runbook" | Operational Excellence |
| "reduce environmental impact", "maximize utilization" | Sustainability |
| "review my architecture against best practices" | Well-Architected Tool |
| "check my account for idle resources / open ports" | Trusted Advisor |
| "get recommendations without installing anything" | Trusted Advisor |
| "simulate a failure to validate recovery" | Game Day (Reliability) |

### טעויות נפוצות

> [!warning] מלכודת 1 — Reliability אינה Performance
> **הניסוח:** "The application must be highly available and respond quickly."
> **הטעות:** לערבב את שני ה-pillars ולבחור פתרון latency לבעיית שרידות.
> **הנכון:** Reliability = מה קורה כשנופלים. Performance = כמה מהר עונים כשהכול תקין. שני צירים נפרדים.

> [!warning] מלכודת 2 — "הכי זול" כתשובה אוטומטית
> **הניסוח:** "Choose the most cost-effective solution that meets the SLA."
> **הטעות:** לבחור את המחיר הנמוך ביותר ולהתעלם מה-SLA.
> **הנכון:** "Cost-effective" מותנה בעמידה בדרישה. פתרון שנופל בדרישה אינו מועמד בכלל.

> [!warning] מלכודת 3 — WA Tool מתקן לבד
> **הניסוח:** "Use the Well-Architected Tool to remediate the findings."
> **הטעות:** לחשוב שהכלי מבצע שינויים בארכיטקטורה.
> **הנכון:** הוא בודק, מדרג סיכון וממליץ. היישום הוא עליך.

> [!warning] מלכודת 4 — Trusted Advisor מלא בחינם
> **הניסוח:** "Enable the full set of Trusted Advisor checks at no cost."
> **הטעות:** להניח שכל הבדיקות זמינות בכל חשבון.
> **הנכון:** הסט המלא דורש תוכנית Business או Enterprise Support.

> [!warning] מלכודת 5 — pillars כ-trade-off בלבד
> **הניסוח:** "We must sacrifice security to reduce cost."
> **הטעות:** להציג את ה-pillars כמשחק סכום אפס.
> **הנכון:** AWS מציגה אותם כסינרגיה. אוטומציה ו-right-sizing משפרים כמה pillars בבת אחת.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**

סטארטאப SaaS שגדל מ-50 ל-5,000 לקוחות בשנה. מבקשים review לפני גיוס הון.

מצב קיים:

- EC2 בודד ב-AZ אחד, מותקן ידנית לפני שנתיים.
- MySQL מותקן על אותו instance.
- גיבוי = snapshot ידני שמישהו זוכר לעשות פעם בחודש.
- מפתחות AWS מוטמעים בקוד האפליקציה.
- אין ניטור מעבר ל"האתר עלה או לא".
- החשבון תפח פי 4, ואף אחד לא יודע למה.

```text
לפני                              אחרי
─────                             ─────
Internet                          Internet
   │                                 │
   ▼                                 ▼
EC2 יחיד (AZ-a)                  CloudFront ─► S3 (static)
 ├── App                             │
 ├── MySQL                           ▼
 └── מפתחות בקוד                   ALB
                                 ┌───┴───┐
snapshot ידני חודשי           AZ-a     AZ-b
                              EC2(ASG) EC2(ASG)  ◄── IAM Role
                                 └───┬───┘
                                RDS Multi-AZ + automated backups
                                     │
                              CloudWatch + CloudTrail + Budgets
```

**הפתרון וההנמקה:**

| ממצא | Pillar | דירוג | הפעולה |
|---|---|---|---|
| מפתחות AWS בקוד | Security | High Risk | להחליף ב-IAM Role ל-instance; לבטל את המפתחות |
| DB על אותו instance כמו האפליקציה | Reliability | High Risk | להעביר ל-RDS Multi-AZ עם גיבוי אוטומטי |
| AZ יחיד | Reliability | High Risk | ASG על 2+ AZs מאחורי ALB |
| גיבוי ידני חודשי | Reliability | High Risk | automated backups + retention מוגדר |
| התקנה ידנית | Operational Excellence | Medium | IaC + AMI/User Data, כדי שהשרת יהיה ניתן לשחזור |
| אין ניטור | Operational Excellence | Medium | CloudWatch alarms + CloudTrail |
| עלות לא מוסברת | Cost Optimization | Medium | tagging, Cost Explorer, Budgets, Trusted Advisor |
| instance בניצולת 8% | Sustainability + Cost | Low | right-sizing ו-scaling לפי ביקוש |
| תמונות מוגשות מה-EC2 | Performance Efficiency | Low | S3 + CloudFront |

**סדר הביצוע:** קודם ה-High Risk Items של Security ו-Reliability — אלו הסיכונים שיכולים לחסל את החברה.
Cost ו-Sustainability חשובים, אבל לא לפני שהמערכת בכלל שורדת נפילה.

**למה לא לעשות הכול בבת אחת?** כי זה מפר את העיקרון של שינויים קטנים והפיכים — ואם משהו יישבר, לא תדע מה גרם לזה.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- לשנן את הנוסח המדויק של כל שאלה ב-Well-Architected Tool.
- את המבנה המלא של ה-whitepaper או מספרי העמודים בו.
- את הרשימה המדויקת של כל בדיקה בודדת ב-Trusted Advisor.
- Lenses מתמחים (Serverless Lens, ML Lens וכדומה) — לא נדרשים ב-SAA-C03.
- מחירים מדויקים של תוכניות Support — רק לדעת שהסט המלא דורש Business/Enterprise.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- ששת ה-Pillars לפי הסדר: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability.
- AWS מציגה אותם כ-**סינרגיה**, לא כ-trade-off שצריך לאזן.
- Sustainability הוא ה-pillar השישי והחדש ביותר.
- Guiding Principles: לא לנחש capacity, לבדוק בקנה מידה של production, לאוטמט, ארכיטקטורה מתפתחת, להחליט לפי דאטה, Game Days.
- **Well-Architected Tool** = חינם, בודק **workload** מול ה-pillars, מפיק דוח והמלצות. לא מתקן לבד.
- **Trusted Advisor** = בלי התקנה, בודק את **החשבון**, 6 קטגוריות.
- 6 קטגוריות Trusted Advisor: Cost, Performance, Security, Fault Tolerance, Service Limits, Operational Excellence.
- סט הבדיקות המלא של Trusted Advisor דורש **Business או Enterprise Support**.
- גישה תוכניתית ל-Trusted Advisor דרך **AWS Support API**; ממצאים יכולים לזרום ל-EventBridge.
- Reliability = התאוששות מכשל. Performance = מהירות בזמן תקין. אל תבלבל.
- "Cost-effective" תמיד מותנה בעמידה בדרישה שבשאלה.
- Review הוא מחזורי — workload ודרישות משתנים.

---

## 11. ✅ בדיקת הבנה

1. מהו ההבדל המרכזי בין Well-Architected Tool ל-Trusted Advisor?
2. חברה רוצה את כל בדיקות Trusted Advisor. מה נדרש?
3. שאלה מציינת "must automatically recover from an AZ failure". איזה pillar נבחן, ומה הפתרון?
4. מה זה Game Day ואיזה pillar הוא משרת בעיקר?
5. מנהל אומר: "נוותר על הצפנה כדי לחסוך". מה הבעיה בטיעון?
6. באיזה pillar נטפל בשרת שרץ בניצולת 5% מסביב לשעון?
7. מה הפלט העיקרי של review — ציון או משהו אחר?

<details>
<summary>תשובות</summary>

1. **WA Tool בודק workload** שאתה מגדיר, דרך שאלון מול ששת ה-Pillars, ומפיק דוח סיכונים. **Trusted Advisor סורק את החשבון** אוטומטית ומחזיר המלצות קונקרטיות ב-6 קטגוריות.

2. **תוכנית Business או Enterprise Support.** הסט הבסיסי החינמי מכסה בעיקר Security ו-Service Limits.

3. **Reliability.** הפתרון: פריסה על 2+ AZs עם Auto Scaling ו-health checks, ו-Multi-AZ ל-DB — כך שההתאוששות אוטומטית ולא ידנית.

4. **תרגיל מכוון** שבו מסמלצים תקלה או עומס קיצוני בסביבה מבוקרת, כדי לוודא שההתאוששות באמת עובדת. משרת בעיקר **Reliability**, ובנוסף Operational Excellence.

5. הטיעון מציג את ה-pillars כ-trade-off, בעוד ש-Security היא **constraint** ולא נעלם שמתמקחים עליו. בנוסף, הצפנה ב-AWS (למשל KMS על EBS/S3) היא זולה ובעלת השפעה זניחה — החיסכון מדומה והסיכון אמיתי.

6. בעיקר **Cost Optimization** (משלמים על קיבולת שלא מייצרת ערך) וגם **Sustainability** (ניצולת נמוכה = בזבוז אנרגיה). הפתרון: right-sizing ו-Auto Scaling.

7. **רשימת סיכונים מתועדת** (High/Medium Risk Items) ותוכנית שיפור מתועדפת — לא ציון מספרי.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[01 - AWS Fundamentals]] · [[31 - Monitoring and Logging]] · [[33 - High Availability and Scalability]] · [[34 - Disaster Recovery]] · [[37 - Cost Optimization]] · [[39 - Architecture Decision Making]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 16124–16210

> [!note] הערת דיוק
> השקפים מונים את ששת ה-Pillars ואת העקרונות המנחים, אך לא מפרטים את ה-Design Principles של כל pillar.
> הפירוט בסעיף 3 מבוסס על ה-Well-Architected Framework הרשמי של AWS ומיושר לנדרש ב-SAA-C03.
