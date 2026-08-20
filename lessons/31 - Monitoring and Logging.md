---
lesson: 31
title: Monitoring and Logging
domain: Design Resilient Architectures
services: [CloudWatch, CloudWatch Logs, CloudTrail, AWS Config, EventBridge, X-Ray]
tags: [saa-c03, observability, monitoring, audit, compliance]
---

# 31 — Monitoring and Logging

> [!abstract] בשורה אחת
> שלושה שירותים, שלוש שאלות: CloudWatch עונה "מה קורה עכשיו", CloudTrail עונה "מי עשה את זה", ו-AWS Config עונה "האם זה מוגדר נכון".

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה metric, log ו-audit trail הם שלושה דברים שונים |
| 2 | איך זה עובד | CloudWatch Metrics, Logs, Alarms; CloudTrail; Config |
| 3 | פירוק מפורט | Alarm states, מקורות לוגים, Unified Agent, Insights family |
| 4 | עלות | custom metrics, log ingestion, data events, configuration items |
| 5 | השוואות | CloudWatch vs CloudTrail vs Config — הטבלה של המבחן |
| 6 | Well-Architected | איך observability נכנס לששת ה-pillars |
| 7 | מלכודות | RAM/disk לא קיימים by default, Config לא חוסם, retention של 90 יום |
| 8 | Scenario | חקירת אירוע אמיתי מקצה לקצה |

**מונחי מפתח בשיעור:** `Namespace` · `Dimension` · `Log Group` · `Subscription Filter` · `Unified Agent` · `Composite Alarm` · `Data Events` · `Config Rule` · `Remediation`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- האפליקציה איטית. אין לך מושג אם זה CPU, דיסק, DB או הרשת.
- מישהו מחק bucket בפרודקשן. אף אחד לא מודה.
- ה-Security Group נפתח ל-`0.0.0.0/0` בשלב כלשהו. מתי? על ידי מי? וכמה זמן הוא היה פתוח?
- יש 40 שרתים. הלוגים יושבים על הדיסק המקומי — וכשהשרת מת, הלוגים מתים איתו.

### מה השירות פותר

- **CloudWatch** — אוסף מספרים (metrics) ולוגים ממרכזי, מצייר dashboards ומפעיל alarms.
- **CloudTrail** — מתעד כל קריאת API בחשבון: מי, מתי, מאיזה IP, ומה היה ה-payload.
- **AWS Config** — מצלם את ההגדרות של כל resource לאורך זמן, ובודק אותן מול חוקים.
- **EventBridge** — הדבק שהופך אירוע (מ-CloudTrail או מ-Config) לפעולה אוטומטית.

> [!tip] האנלוגיה
> CloudWatch = מדי הלוח ברכב (מהירות, טמפרטורה).
> CloudTrail = טכוגרף — מי נהג, מתי ולאן.
> Config = דוח מבחן רישוי — האם הרכב עומד בתקן, ומה השתנה בו מאז הטסט הקודם.

---

## 2. ⚙️ איך זה עובד

### 2.1 CloudWatch Metrics — המספרים

- כל שירות ב-AWS מפרסם metrics ל-CloudWatch באופן אוטומטי.
- **Metric** = משתנה שמנטרים (`CPUUtilization`, `NetworkIn`).
- **Namespace** = הקבוצה שאליה שייך ה-metric (למשל `AWS/EC2`).
- **Dimension** = תכונה שמצמצמת את ה-metric — instance id, environment, AutoScalingGroupName.
- מגבלה למבחן: **עד 30 dimensions לכל metric**.
- לכל datapoint יש timestamp; אפשר לבנות **Dashboards** שמאגדים metrics ממספר Regions.
- **Custom Metrics** — אתה דוחף בעצמך (`PutMetricData`) מה ש-AWS לא רואה: RAM, disk usage, business KPIs.

```text
EC2 / RDS / ALB / Lambda ─┐
Custom PutMetricData ─────┼→ CloudWatch Metrics → Alarm → SNS / ASG / EC2 Action
CloudWatch Agent (OS) ────┘                    └→ Dashboard
                                               └→ Metric Stream → Firehose → S3/3rd party
```

### 2.2 CloudWatch Metric Streams — ייצוא רציף

- זרימה **near-real-time** של metrics ליעד חיצוני, בלי לבצע polling ב-API.
- יעדים: **Kinesis Data Firehose** (ומשם S3, Redshift, OpenSearch, Athena) או ספק צד-שלישי (Datadog, Splunk, New Relic, Dynatrace, Sumo Logic).
- אפשר לסנן ולזרום רק תת-קבוצה של metrics — חוסך תעבורה ועלות ביעד.

### 2.3 CloudWatch Logs — הטקסט

- **Log Group** — בדרך כלל אפליקציה אחת.
- **Log Stream** — instance / container / קובץ לוג בודד בתוך ה-Log Group.
- **Retention policy** — מ-יום אחד ועד 10 שנים, או "never expire". ברירת המחדל של never expire היא מלכודת עלות.
- לוגים **מוצפנים by default**; אפשר להחליף ל-KMS CMK משלך.

מקורות שכותבים ל-CloudWatch Logs:

| מקור | הערה |
|---|---|
| SDK / CloudWatch Logs Agent / **Unified Agent** | EC2 ו-on-premises |
| Elastic Beanstalk | לוגי אפליקציה |
| ECS / EKS | לוגי containers |
| Lambda | לוגי function אוטומטית (דרך ה-execution role) |
| VPC Flow Logs | תעבורת רשת ב-VPC |
| API Gateway | access/execution logs |
| CloudTrail | לפי filter |
| Route 53 | לוג של DNS queries |

### 2.4 להוציא לוגים החוצה — Export מול Subscription

| דרך | מהירות | יעדים | מתי |
|---|---|---|---|
| **S3 Export** (`CreateExportTask`) | **עד 12 שעות** עד שהדאטה זמין | S3 בלבד | ארכוב, batch analytics |
| **Subscription Filter** | real-time / near-real-time | Kinesis Data Streams, Firehose, Lambda | עיבוד חי, אגרגציה, alerting |

- זו שאלה קלאסית: אם כתוב **"real-time"** → Subscriptions. אם כתוב "archive" / "batch" → Export ל-S3.
- Subscription Filter תומך ב-**Cross-Account** ו-**Cross-Region**: מספר חשבונות שולחים ל-Kinesis יעד אחד, עם IAM Role שמאפשר `AssumeRole` ו-Destination Access Policy בצד המקבל.

```text
Account A/Region 1 ─ Subscription Filter ─┐
Account B/Region 2 ─ Subscription Filter ─┼→ Kinesis Data Streams → Firehose → S3 (אגרגציה מרכזית)
Account B/Region 3 ─ Subscription Filter ─┘
```

### 2.5 CloudWatch Logs Insights — לשאול שאלות על הלוגים

- מנוע query ייעודי מעל CloudWatch Logs (שפת שאילתה משלו).
- מגלה שדות אוטומטית מלוגי שירותי AWS ומלוגי JSON.
- מאפשר filter, aggregate, sort, limit; אפשר לשמור query ולהצמיד ל-Dashboard.
- אפשר לשאול **מספר Log Groups בכמה חשבונות** בבת אחת.
- **חשוב:** זה מנוע query, **לא** מנוע real-time. אין כאן streaming — יש חיפוש היסטורי.

### 2.6 CloudWatch Alarms

- Alarm יושב על metric ומפעיל פעולה כשהתנאי מתקיים.
- אפשר לבנות alarm גם על **Metric Filter** של Log Group (למשל: לספור "ERROR" בלוג ולהתריע מעל סף).
- **Composite Alarm** — alarm שמנטר את ה-states של alarms אחרים עם AND / OR. הפתרון ל-"alarm noise": מתריעים רק כש-CPU גבוה **וגם** IOPS גבוה.
- לבדיקה ידנית: `aws cloudwatch set-alarm-state --alarm-name X --state-value ALARM --state-reason "test"`.

### 2.7 CloudTrail — מי עשה מה

- **מופעל by default** בכל חשבון.
- מתעד קריאות API מ-Console, CLI, SDK ומשירותי AWS עצמם.
- Trail יכול לחול על **כל ה-Regions** (ברירת המחדל, מומלץ) או על Region בודד.
- אפשר לשלוח את הלוגים ל-**S3** ו/או ל-**CloudWatch Logs**.
- כלל אצבע: **resource נמחק → הולכים ל-CloudTrail קודם.**

### 2.8 AWS Config — האם ההגדרה תקינה

- מתעד **configuration items** — צילום מצב של resource בכל שינוי — ומחזיק timeline.
- עונה על: האם יש SSH פתוח לעולם? האם ה-bucket ציבורי? איך השתנתה הגדרת ה-ALB לאורך זמן?
- שירות **per-Region**, אבל ניתן ל-aggregate בין Regions וחשבונות.
- אפשר לאחסן את הדאטה ב-S3 ולנתח ב-Athena.
- **Config Rules** — מעל 75 managed rules, או custom rule שממומש ב-Lambda.
- הפעלת חוק: על כל שינוי בהגדרה, ו/או במרווחי זמן קבועים.
- **Config לא מונע פעולות.** אין בו deny — הוא רק מסמן COMPLIANT / NON_COMPLIANT.
- **Remediation** — תיקון אוטומטי דרך **SSM Automation Document** (managed או custom, כולל הפעלת Lambda), עם Retries אם ה-resource עדיין לא compliant.
- **Notifications** — EventBridge לכל NON_COMPLIANT, או SNS לכל האירועים (ואז מסננים ב-SNS Filtering או בצד הלקוח).

---

## 3. 🔍 פירוק מפורט

### 3.1 Alarm States

| State | מה זה אומר | מלכודת |
|---|---|---|
| `OK` | ה-metric בתוך הסף | — |
| `ALARM` | הסף נחצה | זה מה שמפעיל את ה-action |
| `INSUFFICIENT_DATA` | אין מספיק datapoints לקבוע | **לא** אומר תקלה, ולא אומר שהכל תקין |

- **Period** — אורך חלון ההערכה בשניות.
- High-resolution custom metrics: period של **10 שניות, 30 שניות, או כפולות של 60**.

### 3.2 Alarm Targets — מה alarm יכול להפעיל

| Target | דוגמה |
|---|---|
| EC2 Action | Stop / Terminate / Reboot / **Recover** של instance |
| Auto Scaling Action | scale out / scale in |
| SNS | ומשם: Lambda, SQS, email, HTTP — כלומר כמעט הכל |

### 3.3 EC2 Status Checks ו-Instance Recovery

| Check | מה נבדק |
|---|---|
| **Instance status** | ה-VM עצמו (OS, רשת פנימית) |
| **System status** | החומרה הפיזית שמתחת |
| **Attached EBS status** | ה-volumes המחוברים |

- Alarm על `StatusCheckFailed_System` → **Recover** מריץ את ה-instance על חומרה אחרת.
- ב-Recovery נשמרים: **Private IP, Public IP, Elastic IP, metadata ו-placement group**.

### 3.4 Agent — מה חסר לך by default

- מ-EC2 **לא מגיעים לוגים בכלל** ל-CloudWatch בלי agent.
- ה-metrics ה-built-in של EC2 הם ברמה גבוהה: CPU, network, disk (ברמת ה-hypervisor).

| Agent | מה נותן | מגבלה |
|---|---|---|
| CloudWatch **Logs Agent** | לוגים בלבד | גרסה ישנה, לא שולח metrics |
| CloudWatch **Unified Agent** | לוגים **+ metrics ברמת OS** | הבחירה הנכונה היום |

Unified Agent אוסף: CPU מפורט (user/system/idle/steal/guest), **RAM** (free/used/cached), **Disk** (free/used/total + IOPS), Netstat (חיבורי TCP/UDP), Processes, Swap.

- קונפיגורציה מרוכזת דרך **SSM Parameter Store** — כותבים config אחד ומושכים אותו לכל השרתים.
- עובד גם על שרתי **on-premises**.

> [!warning] העובדה שהכי נשאלת
> **RAM ו-disk usage אינם metrics ברירת מחדל של EC2.** צריך Unified Agent (או custom metric).

### 3.5 CloudWatch Network Synthetic Monitor

- מנטר בעיות רשת בין אפליקציות ב-AWS לבין ה-data center שלך.
- מזהה packet loss, latency, jitter.
- **ללא התקנת agents**; בודק ICMP או TCP ליעדים on-premises דרך **Direct Connect** או **Site-to-Site VPN**.
- הנתונים מתפרסמים כ-CloudWatch Metrics.

### 3.6 משפחת ה-Insights — מי לאיזו שאלה

| כלי | מקור נתונים | מה נותן |
|---|---|---|
| **Logs Insights** | CloudWatch Logs | שפת query לחיפוש וניתוח לוגים |
| **Container Insights** | ECS, EKS, K8s on EC2, Fargate | metrics + logs מ-containers (ב-EKS דרך CloudWatch Agent מכולתי) |
| **Lambda Insights** | Lambda | CPU time, memory, disk, network, **cold starts** ו-worker shutdowns; מגיע כ-**Lambda Layer** |
| **Contributor Insights** | כל לוג AWS (VPC Flow Logs, DNS...) | **Top-N contributors** — ה-IP הכי רועש, ה-URL שמייצר הכי הרבה שגיאות |
| **Application Insights** | EC2 עם טכנולוגיות נבחרות (Java, .NET, IIS, DBs) | dashboards אוטומטיים לאיתור תקלות; ממצאים ל-EventBridge ו-SSM OpsCenter |

### 3.7 CloudTrail — סוגי אירועים

| סוג | מופעל by default? | מה נכלל | למה חשוב |
|---|---|---|---|
| **Management Events** | **כן** | פעולות על resources: `AttachRolePolicy`, `CreateSubnet`, `CreateTrail` | ניתן להפריד Read מ-Write |
| **Data Events** | **לא** (נפח עצום) | S3 object-level (`GetObject`, `PutObject`, `DeleteObject`), Lambda `Invoke` | מפעילים רק על מה שצריך — עולה כסף |
| **Insights Events** | **לא** | זיהוי אנומליות בפעילות write | provisioning חריג, פריצת service limits, bursts של IAM |

- **CloudTrail Insights** בונה baseline מ-management events רגילים ואז מנתח write events בזמן אמת.
- אנומליה מופיעה ב-Console, נשלחת ל-S3, ומייצרת **EventBridge event** לאוטומציה.

### 3.8 Retention — הנקודה שנופלים עליה

- אירועים נשמרים ב-CloudTrail עצמו למשך **90 יום בלבד**.
- לשמירה ארוכה: לכתוב ל-**S3** ולנתח ב-**Athena**. זו התשובה הנכונה לכל "audit trail for 7 years".

### 3.9 EventBridge + CloudTrail — לתפוס API call ולהגיב

```text
User → DeleteTable API → CloudTrail → EventBridge Rule → SNS (alert) / Lambda (auto-fix)
User → AuthorizeSecurityGroupIngress → CloudTrail → EventBridge → Lambda מחזיר את ה-SG למצב תקין
```

זה ה-pattern שמופיע בשאלות "התראה מיידית כשמישהו פותח SG לעולם".

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| Custom Metrics | לכל metric ייחודי לחודש | **כל שילוב dimensions = metric נפרד** — כאן העלות מתפוצצת |
| Metric API requests | לכל `PutMetricData` / `GetMetricData` | batching מוזיל |
| Detailed Monitoring (EC2) | דגימה כל **דקה** במקום כל **5 דקות** (basic) | basic חינם, detailed בתשלום לכל instance |
| CloudWatch Logs — Ingestion | לכל GB שנכנס | זה בדרך כלל הרכיב הגדול |
| CloudWatch Logs — Storage | לכל GB לחודש | תלוי ישירות ב-retention |
| Logs Insights queries | לכל GB שנסרק בשאילתה | שאילתה על 30 יום סורקת 30 יום |
| Alarms | לכל alarm; high-resolution יקר יותר | Composite Alarm מחויב בנפרד |
| Dashboards | לכל dashboard מעל ה-free tier | |
| CloudTrail Management Events | העתק ראשון ל-trail — ללא חיוב | **trail שני והלאה בתשלום** |
| CloudTrail Data Events / Insights | לכל אירוע | היקף עצום — מפעילים ממוקד |
| AWS Config | **לכל configuration item שנרשם** + **לכל rule evaluation** | אין free tier |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Basic Monitoring (5 דקות) | 0 | רוב העומסים היציבים |
| Detailed Monitoring (דקה) | תוספת לכל instance | ASG שצריך להגיב מהר, אבחון תקלות |
| CloudWatch Logs retention ארוך | הכי יקר לאחסון ארוך טווח | רק ללוגים חמים |
| Export / Subscription ל-S3 + Athena | זול משמעותית לאחסון | ארכיון, compliance של שנים |
| Metric Stream ל-Firehose | זול מ-polling מסיבי ב-API | הזרמה לכלי צד-שלישי |

### 🚩 עלויות נסתרות

- **Custom metric עם dimension של instance id** ב-1,000 שרתים = 1,000 metrics.
- **`DEBUG` בפרודקשן** — ingestion של GB-ים שאף אחד לא קורא.
- **Data events על כל ה-buckets** — bucket עמוס מייצר מיליוני אירועים ביום.
- **Config על כל סוגי ה-resources** בכל Region — configuration items מצטברים גם בלי שינויים משמעותיים.
- **Logs Insights** — כל הרצה של שאילתה גורפת עולה מחדש.
- **VPC Flow Logs ל-CloudWatch Logs** — נפח אדיר; ל-S3 זה זול בהרבה.

### 💡 טיפים לחיסכון

- להגדיר **retention** מפורש לכל Log Group. ברירת המחדל "never expire" היא דליפת כסף שקטה.
- לוגים ארוכי-טווח → **S3 + Lifecycle ל-Glacier**, לא CloudWatch Logs.
- לצמצם cardinality של dimensions; לאגרגט לפני שליחה.
- להפעיל data events **רק** על ה-buckets/functions הרגישים, עם prefix filters.
- ל-Config: לבחור סוגי resources ספציפיים במקום "record all".
- להשתמש ב-metric filters במקום לשמור כל שורת לוג רק כדי לספור אותה.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 הטבלה המרכזית של השיעור

| קריטריון | **CloudWatch** | **CloudTrail** | **AWS Config** |
|---|---|---|---|
| מה מנטר | ביצועים והתנהגות | קריאות API | הגדרות של resources |
| סוג הנתון | Metrics + Logs (time series / טקסט) | Event records (who/when/what/from where) | Configuration items + compliance state |
| שאלה טיפוסית | **"מה ה-CPU?"** / "כמה 5XX?" | **"מי מחק את ה-bucket?"** | **"האם ה-SG פתוח לעולם?"** |
| ציר הזמן | now + היסטוריה של metrics | 90 יום ב-service, יותר ב-S3 | timeline מלא של שינויי הגדרה |
| היקף | per-Region (dashboards cross-Region) | Global service; trail לכל ה-Regions | **per-Region**, ניתן לאגרגציה |
| פעולה אוטומטית | Alarm → SNS/ASG/EC2 | דרך EventBridge | Remediation ב-SSM Automation |
| חוסם פעולה? | לא | לא | **לא** (רק מסמן) |

> [!info] שורה תחתונה
> ביצועים/התראות → CloudWatch. חקירת "מי עשה" → CloudTrail. "האם מוגדר נכון ומה השתנה" → Config. השאלה במבחן כמעט תמיד מנוסחת באחת משלוש הצורות האלה.

### 5.2 אותו resource, שלוש זוויות — דוגמת ALB

| שירות | מה הוא נותן על ה-ALB |
|---|---|
| CloudWatch | מספר חיבורים נכנסים, אחוז קודי שגיאה לאורך זמן, dashboard ביצועים |
| Config | מעקב אחרי כללי ה-Security Group, שינויי קונפיגורציה, חוק שמוודא שתמיד מוצמד SSL certificate |
| CloudTrail | מי ביצע את השינוי ב-ALB ובאיזו קריאת API |

### 5.3 Export מול Subscription מול Metric Stream

| קריטריון | S3 Export | Subscription Filter | Metric Stream |
|---|---|---|---|
| מה זורם | לוגים | לוגים | metrics |
| Latency | עד 12 שעות | real-time / near-real-time | near-real-time |
| יעדים | S3 | KDS, Firehose, Lambda | Firehose, צד-שלישי |
| שימוש טיפוסי | ארכוב | עיבוד חי, אגרגציה cross-account | ניטור בכלי חיצוני |

### 5.4 CloudWatch מול X-Ray

- CloudWatch אומר **שיש** בעיה ואיפה בערך.
- **X-Ray** (ראו [[38 - Serverless and Modern Architectures]]) אומר **באיזה hop** בין ה-microservices ה-latency נוצר.
- "distributed tracing" בשאלה → X-Ray, לא CloudWatch.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | ניטור שמוביל לפעולה, לא לרעש | Composite Alarms במקום 12 alarms נפרדים; runbook לכל alarm; Logs Insights queries שמורים ב-dashboard |
| Security | audit trail שלא ניתן לשינוי + זיהוי חריגות | Organization Trail ל-S3 עם Object Lock ו-KMS; CloudTrail Insights; EventBridge על `AuthorizeSecurityGroupIngress` |
| Reliability | לזהות כשל ולהתאושש בלי אדם | Alarm על `StatusCheckFailed_System` → EC2 Recover; Config Remediation אוטומטי ל-resources לא-compliant |
| Performance Efficiency | למדוד את מה שבאמת חוסם | Unified Agent ל-RAM ו-disk; Contributor Insights לאיתור ה-Top-N; Lambda Insights ל-cold starts |
| Cost Optimization | לשלם רק על מה שקוראים | retention לכל Log Group; data events ממוקדים; Config על סוגי resources נבחרים; ארכיון ב-S3 |
| Sustainability | פחות ingestion = פחות אחסון וחישוב | הורדת log level בפרודקשן, דגימה, aggregation ב-agent לפני שליחה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "מי מחק / מי שינה / who made the API call" | **CloudTrail** |
| "compliance", "drift", "configuration history" | **AWS Config** |
| "CPU", "latency", "5XX", "threshold", "alarm" | **CloudWatch** |
| "RAM usage" / "disk space used" מ-EC2 | **CloudWatch Unified Agent** (custom metric) |
| "real-time processing of logs" | **CloudWatch Logs Subscription Filter** → KDS/Firehose/Lambda |
| "archive logs for years / analyze with SQL" | **S3 + Athena** |
| "audit events older than 90 days" | CloudTrail → **S3** (+ Athena) |
| "top talkers" / "which IP causes most traffic" | **Contributor Insights** |
| "cold starts, memory of my functions" | **Lambda Insights** |
| "automatically fix non-compliant resource" | **Config Rule + SSM Automation Remediation** |
| "reduce alarm noise" | **Composite Alarm** |
| "instance stuck due to underlying hardware" | Alarm על **StatusCheckFailed_System** → Recover |
| "network issues to on-premises, no agents" | **CloudWatch Network Synthetic Monitor** |
| "trace request across microservices" | **X-Ray** (לא CloudWatch) |

### טעויות נפוצות

> [!warning] מלכודת — CloudTrail כמנטר ביצועים
> **הניסוח:** "האפליקציה איטית, איזה שירות יראה את עומס ה-CPU?"
> **הטעות:** לבחור CloudTrail כי הוא "מתעד הכל".
> **הנכון:** CloudTrail מתעד **קריאות API בלבד**. הוא לא יודע מה ה-CPU ולא רואה 5XX. זה CloudWatch.

> [!warning] מלכודת — RAM ב-EC2
> **הניסוח:** "התראה כשה-memory utilization עובר 80%."
> **הטעות:** להניח שיש metric מוכן ולהגדיר alarm.
> **הנכון:** **אין** memory metric ב-EC2 by default. צריך להתקין CloudWatch Unified Agent ולפרסם custom metric, ורק אז alarm.

> [!warning] מלכודת — Config חוסם
> **הניסוח:** "מנע יצירת S3 bucket ציבורי."
> **הטעות:** Config Rule.
> **הנכון:** Config **מזהה ומדווח**, לא מונע. מניעה = **SCP** ([[04 - IAM Advanced and Organizations]]) או **S3 Block Public Access** ([[17 - S3 Security and Data Management]]). Config מוסיף גילוי ותיקון בדיעבד.

> [!warning] מלכודת — Data Events "מופעלים אוטומטית"
> **הניסוח:** "צריך לדעת מי קרא אובייקט ספציפי ב-S3."
> **הטעות:** להניח ש-CloudTrail כבר מתעד את זה.
> **הנכון:** by default נרשמים **management events בלבד**. `GetObject` הוא **data event** ויש להפעיל אותו במפורש (ובעלות).

> [!warning] מלכודת — INSUFFICIENT_DATA
> **הניסוח:** "ה-alarm במצב INSUFFICIENT_DATA — האם השרת תקין?"
> **הטעות:** לפרש כ-OK או כ-ALARM.
> **הנכון:** זה מצב שלישי — אין מספיק datapoints. לרוב מעיד ש-agent הפסיק לדווח או שה-period קצר מדי ביחס לתדירות הדגימה.

> [!warning] מלכודת — Logs Insights כ-real-time
> **הניסוח:** "עיבוד זרם לוגים בזמן אמת לצורך alerting."
> **הטעות:** Logs Insights.
> **הנכון:** Logs Insights הוא **מנוע query על נתונים שכבר נשמרו**. זמן אמת = Subscription Filter.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** חברת SaaS עם 3 חשבונות AWS (dev/stage/prod) צריכה: התראה תוך דקות על עלייה ב-5XX, אפשרות לחקור מי שינה משאב, זיהוי SG שנפתח לעולם עם תיקון אוטומטי, ושמירת audit ל-7 שנים.

```text
                    ┌─ CloudWatch Alarm (ALB 5XX + p99 latency, Composite) → SNS → PagerDuty
ALB / EC2 / RDS ────┤
                    └─ Unified Agent → CloudWatch Logs ─ Subscription Filter → KDS
                                                                                 ↓
                                            Kinesis Firehose → S3 (Log Archive account)
Org Trail (all Regions, all accounts) ────→ S3 + Object Lock + KMS ──→ Athena (7 years)
AWS Config (aggregator, כל החשבונות) ──→ Rule: restricted-ssh
                                          └ NON_COMPLIANT → SSM Automation → סוגר את ה-SG
                                          └ EventBridge → SNS (security channel)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Composite Alarm על 5XX **וגם** latency | מונע פייג'ר על ספייק רגעי בודד |
| Subscription Filter (ולא S3 Export) | Export לוקח עד 12 שעות; כאן צריך אגרגציה חיה |
| Organization Trail לכל ה-Regions | לוכד גם פעילות ב-Region שאף אחד לא משתמש בו — שם מתחילות פריצות |
| S3 + Object Lock ל-audit | CloudTrail שומר 90 יום בלבד; Object Lock מונע מחיקה גם ע"י admin |
| Athena מעל S3 | שאילתות SQL על שנים של לוגים בלי לשלם אחסון CloudWatch Logs |
| Config Aggregator | Config הוא per-Region; אגרגטור נותן תמונה אחת לכל הארגון |
| SSM Automation Remediation | סגירת SG תוך שניות, בלי להעיר אדם |

**למה לא לשלוח הכל ל-CloudWatch Logs עם retention לנצח?** כי ingestion + storage ל-7 שנים יקרים בסדר גודל מ-S3, והשאילתות ההיסטוריות ב-Athena זולות יותר מ-Logs Insights על אותו נפח.

**למה לא Config בלבד לאבטחה?** Config מסמן, לא מונע. שכבת המניעה היא SCP ו-Security Group discipline; Config הוא רשת הביטחון.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- לשנן namespaces ושמות metrics מדויקים (מלבד `CPUUtilization` ו-`StatusCheckFailed_System`).
- את התחביר של שפת Logs Insights.
- מחירים מדויקים בדולרים — מספיק להבין **על מה** מחייבים.
- קונפיגורציית JSON של ה-Unified Agent.
- רשימת 75+ ה-managed Config Rules.
- הפרטים הפנימיים של Application Insights (רק מה הוא נותן ולמי).

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **CloudWatch = ביצועים | CloudTrail = מי עשה | Config = איך מוגדר.**
- CloudTrail **מופעל by default**; Config **לא**.
- CloudTrail שומר **90 יום**; לשמירה ארוכה → **S3 + Athena**.
- **Management events** נרשמים by default; **data events** (S3 objects, Lambda Invoke) — לא.
- **אין RAM ו-disk usage** ב-EC2 בלי **Unified Agent**.
- Alarm states: `OK` / `ALARM` / `INSUFFICIENT_DATA`.
- Alarm targets: EC2 action (כולל **Recover**), Auto Scaling, SNS.
- **Recover** משמר Private IP, Public IP, Elastic IP, metadata ו-placement group.
- **Composite Alarm** = AND/OR על alarms אחרים = פחות רעש.
- Logs **real-time** → Subscription Filter; Logs ל-**archive** → S3 Export (עד 12 שעות).
- Logs Insights = **query engine**, לא real-time.
- **Contributor Insights** = Top-N. **Lambda Insights** = cold starts. **Container Insights** = ECS/EKS/Fargate.
- Config הוא **per-Region**, לא מונע כלום, ומתקן דרך **SSM Automation**.
- 30 dimensions max לכל metric; high-resolution period = 10/30 שניות.
- Detailed Monitoring = דגימה כל דקה (בתשלום) מול basic כל 5 דקות.

---

## 11. ✅ בדיקת הבנה

1. מישהו מחק טבלת DynamoDB בפרודקשן לפני חודשיים. באיזה שירות מתחילים, ומה קורה אם האירוע קרה לפני 5 חודשים?
2. צריך להתריע כשה-memory של EC2 עובר 85%. מה השלבים?
3. לוגים של אפליקציה צריכים להגיע ל-OpenSearch בזמן אמת. איזה מנגנון, ולמה לא S3 Export?
4. Config Rule מזהה שה-Security Group נפתח ל-`0.0.0.0/0`. מה יקרה למשאב, ואיך גורמים לתיקון אוטומטי?
5. יש 8 alarms שמתריעים יחד בכל אירוע ומציפים את הצוות. מה הפתרון המובנה?

<details>
<summary>תשובות</summary>

1. **CloudTrail** — הוא מתעד את קריאת ה-`DeleteTable`. אבל CloudTrail עצמו שומר **90 יום בלבד**; לאירוע בן 5 חודשים צריך שה-trail יכתוב ל-**S3**, ואז מנתחים ב-**Athena**. אם לא הוגדר trail ל-S3 מראש — האירוע אבד.
2. אין metric כזה by default. מתקינים **CloudWatch Unified Agent** על ה-instances (עם IAM Role מתאים, קונפיג מרוכז ב-SSM Parameter Store), הוא מפרסם **custom metric** של RAM, ורק אז מגדירים **CloudWatch Alarm** על ה-metric הזה עם SNS.
3. **CloudWatch Logs Subscription Filter** → Kinesis Data Firehose → OpenSearch. S3 Export לא מתאים כי הדאטה יכול לקחת **עד 12 שעות** להיות זמין — זה batch, לא real-time.
4. המשאב יסומן **NON_COMPLIANT** — Config **לא חוסם ולא מתקן מעצמו**. לתיקון אוטומטי מגדירים **Remediation Action** עם **SSM Automation Document** (managed או custom שמפעיל Lambda), אפשר עם Retries. במקביל אפשר EventBridge → SNS להתראה.
5. **Composite Alarm** — alarm שמנטר את ה-states של ה-alarms האחרים עם תנאי AND/OR, ומתריע פעם אחת רק כשהצירוף באמת מעיד על תקלה.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[29 - Event-Driven Architecture]] · [[32 - Security Services]] · [[07 - Auto Scaling]] · [[38 - Serverless and Modern Architectures]] · [[04 - IAM Advanced and Organizations]] · [[11 - VPC Security]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 10383–10713, 10822–11164
