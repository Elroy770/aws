---
lesson: 22
title: RDS Scaling and Availability
domain: Design Resilient Architectures
services: [RDS, Aurora, Aurora Serverless, Aurora Global Database, ElastiCache]
tags: [saa-c03, databases, high-availability, scalability, caching]
---

# 22 — RDS Scaling and Availability

> [!abstract] בשורה אחת
> ההבחנה שנשאלת יותר מכל אחרת ב-SAA-C03: **Read Replica פותרת ביצועי קריאה, Multi-AZ פותרת זמינות** — הן לא מחליפות זו את זו, וברוב הארכיטקטורות הנכונות שתיהן קיימות יחד.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | שני סוגי כאב שונים לחלוטין |
| 2 | Read Replicas | async, עד 15, use cases, ועלות הרשת |
| 3 | Multi-AZ | sync, DNS יחיד, failover אוטומטי |
| 4 | הטבלה המכריעה | Read Replica מול Multi-AZ, שורה מול שורה |
| 5 | Aurora | ארכיטקטורת 6 עותקים, endpoints, Serverless, Global |
| 6 | ElastiCache | Redis מול Memcached, דפוסי caching |
| 7 | עלות | מה מכפיל את החשבון |
| 8 | מלכודות ו-Scenario | הניסוחים שמכריעים |

**מונחי מפתח בשיעור:** `Read Replica` · `Multi-AZ` · `Writer/Reader Endpoint` · `Backtrack` · `Lazy Loading` · `Write Through`

---

## 1. 🎯 הבעיה והפתרון

### שתי בעיות שונות שקל לבלבל ביניהן

| הבעיה | הסימפטום | הפתרון |
|---|---|---|
| **ביצועים** | דוחות כבדים מאטים את מערכת ההזמנות | Read Replica |
| **זמינות** | ה-AZ שבו יושב ה-DB נופל, והמערכת מתה | Multi-AZ |

- Read Replica **לא** תעזור כשה-AZ נופל — צריך התערבות ידנית כדי לקדם אותה.
- Multi-AZ **לא** תעזור לביצועים — ה-standby לא מקבל שאילתות בכלל.

> [!tip] האנלוגיה
> Read Replica היא עובד נוסף שמקבל עותק של המסמכים כדי לענות על שאלות.
> Multi-AZ הוא גנרטור חירום — לא עושה כלום ביום-יום, ומציל אותך כשהחשמל נופל.

---

## 2. 📖 RDS Read Replicas

### 2.1 העובדות

- עד **15 Read Replicas** לכל DB instance.
- ניתן למקם אותן **באותו AZ, ב-AZ אחר, או ב-Region אחר**.
- הרפליקציה היא **ASYNC** — ולכן הקריאות הן **eventually consistent**.
- ניתן **לקדם (promote)** replica ל-DB עצמאי — פעולה ידנית, לא failover.
- **האפליקציה חייבת לעדכן את ה-connection string** כדי להשתמש בהן.
- ה-replicas מיועדות ל-**SELECT בלבד**. אין INSERT, UPDATE או DELETE.

```text
                Application
            ┌───────┴────────┐
         writes            reads
            │        ┌───────┼───────┐
            ▼        ▼       ▼       ▼
      RDS Primary  Replica Replica Replica
            └──ASYNC──┴───────┴───────┘
```

### 2.2 ה-use case הקלאסי

```text
Production App ──reads/writes──► RDS Primary
                                     │ ASYNC
Reporting App  ──reads only──►  Read Replica
```

- יש DB בייצור שנושא עומס רגיל.
- רוצים להריץ עליו אנליטיקה ודוחות כבדים.
- מקימים Read Replica ומכוונים אליה את יישום הדוחות.
- **אפליקציית הייצור לא מושפעת כלל.**

### 2.3 עלות הרשת — נקודה שנשאלת ישירות

ב-AWS יש חיוב על תעבורה שעוברת בין AZs. אבל:

| תרחיש | חיוב על התעבורה |
|---|---|
| Read Replica ב-**אותו Region**, AZ אחר | **חינם** — AWS מוותרת על העמלה |
| Read Replica ב-**Region אחר** | **בתשלום**, ועל כל בייט של רפליקציה |

זו הסיבה שהמלצה סטנדרטית היא replicas cross-AZ באותו Region — כמעט חינם.
cross-Region נעשה רק כשיש דרישת DR או latency גיאוגרפי אמיתי.

---

## 3. 🛡️ RDS Multi-AZ

### העובדות

- הרפליקציה היא **SYNC** — כל כתיבה מאושרת רק אחרי שנרשמה בשני העותקים.
- **שם DNS אחד בלבד** — האפליקציה לא יודעת ולא צריכה לדעת מי ה-primary.
- ה-failover **אוטומטי לחלוטין**, ללא שום התערבות באפליקציה.
- מפעיל failover ב: אובדן AZ, אובדן רשת, כשל instance או כשל storage.
- **לא משמש ל-scaling.** ה-standby לא נגיש לקריאות.
- Read Replica יכולה בעצמה להיות מוגדרת כ-Multi-AZ, לצורכי DR.

```text
Application ──► RDS endpoint (DNS יחיד)
                     │
        ┌────────────┴────────────┐
   Primary (AZ-a)  ←─SYNC─→  Standby (AZ-b)
                                לא מקבל שאילתות
```

### מעבר מ-Single-AZ ל-Multi-AZ

- **ללא downtime** — אין צורך לעצור את ה-DB, פשוט לוחצים "modify".
- מה קורה מאחורי הקלעים:

```text
1. RDS מצלם snapshot של ה-DB הקיים
2. משחזר ממנו DB חדש ב-AZ אחר
3. מקים סנכרון synchronous בין השניים
```

זו שאלה שחוזרת: התשובה הנכונה היא **zero downtime**, ולא "צריך חלון תחזוקה".

---

## 4. ⚖️ הטבלה המכריעה — Read Replica מול Multi-AZ

| קריטריון | Read Replica | Multi-AZ |
|---|---|---|
| **המטרה** | scaling של קריאות | זמינות גבוהה / DR |
| **סוג הרפליקציה** | **ASYNC** | **SYNC** |
| **עקביות** | eventually consistent (יש lag) | זהה לחלוטין |
| **זמין לקריאה?** | ✅ כן, זו כל המטרה | ❌ **לא**, ה-standby סגור |
| **Failover אוטומטי?** | ❌ נדרשת promotion ידנית | ✅ אוטומטי לחלוטין |
| **כמה** | עד 15 | standby אחד |
| **מיקום** | אותו AZ / AZ אחר / Region אחר | AZ אחר באותו Region |
| **שינוי ב-connection string** | ✅ נדרש | ❌ אותו DNS |
| **עלות data transfer** | **חינם** באותו Region; **בתשלום** cross-Region | חינם |
| **עלות instance** | instance מלא לכל replica | מכפיל את עלות ה-primary |
| **מגן מכשל AZ?** | לא באופן אוטומטי | ✅ |

> [!info] שורה תחתונה
> "דוחות מאטים את הייצור" → Read Replica.
> "המערכת חייבת לשרוד נפילת AZ בלי התערבות" → Multi-AZ.
> "גם וגם" → שתיהן, וזו התשובה הנכונה ברוב שאלות הייצור.

---

## 5. 🚀 Amazon Aurora

### 5.1 מה זה

- טכנולוגיה **קניינית** של AWS, לא open source.
- תואמת API ל-**PostgreSQL** ול-**MySQL** — הדרייברים הקיימים עובדים ללא שינוי.
- מותאמת לענן: לפי AWS, פי ~5 מ-MySQL על RDS ופי ~3 מ-PostgreSQL על RDS.
- האחסון **גדל אוטומטית בקפיצות של 10 GB** עד עשרות ומאות טרה-בייט.
  (השקפים נוקבים ב-256 TB; המגבלה השתנתה לאורך הגרסאות — למבחן חשוב העיקרון, לא המספר.)
- עד **15 Aurora Replicas**, עם replica lag של פחות מ-10ms.
- **failover כמעט מיידי** — HA היא תכונה מובנית, לא תוספת.
- עולה כ-**20% יותר מ-RDS**, אבל יעילה יותר בפועל.

### 5.2 ארכיטקטורת האחסון — הלב של Aurora

```text
        AZ-1            AZ-2            AZ-3
       [copy]          [copy]          [copy]
       [copy]          [copy]          [copy]
        └───────────────┴───────────────┘
              Shared Storage Volume
       (striped על פני מאות volumes, self-healing)
```

- **6 עותקים של הדאטה על פני 3 AZs.**
- **4 מתוך 6** עותקים נדרשים כדי לאשר כתיבה.
- **3 מתוך 6** עותקים נדרשים לקריאה.
- self-healing ברפליקציה peer-to-peer — בלוק פגום מתוקן אוטומטית.
- האחסון מפוזר (striped) על פני מאות volumes.
- instance אחד בלבד מקבל כתיבות (master); failover של ה-master **תוך פחות מ-30 שניות**.
- ה-master ועד 15 replicas מגישים קריאות.
- נתמכת רפליקציה cross-Region.

### 5.3 ה-Endpoints — נושא שנשאל ישירות

| Endpoint | לאן מפנה | מתי משתמשים |
|---|---|---|
| **Writer Endpoint** | תמיד ל-master הנוכחי | כל הכתיבות; ממשיך לעבוד גם אחרי failover |
| **Reader Endpoint** | load balancing אוטומטי בין ה-replicas | קריאות רגילות |
| **Custom Endpoint** | תת-קבוצה של instances שאתה מגדיר | הפניית שאילתות אנליטיות ל-instances חזקים בלבד |

- ה-Writer Endpoint הוא הסיבה שהאפליקציה לא צריכה לדעת מי ה-master.
- ה-Reader Endpoint מבצע **connection load balancing** — לא DNS round-robin פשוט בלבד.
- **אחרי שמגדירים Custom Endpoints, בדרך כלל כבר לא משתמשים ב-Reader Endpoint** —
  מפצלים את הקריאות ל-endpoints ייעודיים לפי סוג העומס.

דוגמה: cluster עם `db.r5.2xlarge` ל-BI ו-`db.r3.large` לאפליקציה.
Custom Endpoint מכוון את השאילתות האנליטיות רק ל-instances הגדולים.

### 5.4 Aurora Replicas Auto Scaling

- כשעומס הקריאות עולה, ה-CPU של ה-replicas מטפס.
- Aurora מוסיפה replicas אוטומטית, וה-Reader Endpoint מתרחב לכלול אותן.
- כשהעומס יורד — ה-replicas מוסרות.
- זהו scaling **של הקריאות בלבד**; ה-writer נשאר אחד.

### 5.5 יכולות Aurora נוספות

| יכולת | מה זה נותן |
|---|---|
| Automatic failover | ללא הגדרה נוספת |
| Backup and Recovery | כמו RDS, אבל retention לא ניתן לכיבוי |
| Push-button scaling | שינוי גודל instance בלחיצה |
| Automated Patching with **Zero Downtime** | עדכונים בלי חלון תחזוקה |
| Advanced Monitoring | מטריקות עמוקות יותר |
| **Backtrack** | **החזרת הדאטה לנקודת זמן קודמת בלי להשתמש בגיבוי כלל** |

> [!warning] Backtrack אינו גיבוי
> Backtrack "מגלגל אחורה" את ה-cluster הקיים במקום ליצור DB חדש — מהיר מאוד לתיקון טעות אנוש.
> אבל הוא לא מחליף backups: הוא לא מגן מפני מחיקת ה-cluster או מכשל Region.

### 5.6 Aurora Serverless

- ה-DB עולה ומתכווץ אוטומטית לפי השימוש בפועל.
- **אין capacity planning** — לא בוחרים instance class.
- חיוב **לפי שנייה**.
- מתאים ל-workloads **בלתי צפויים, לסירוגין או נדירים**: dev, אפליקציות פנימיות, עונתיות.
- לא מתאים לעומס יציב 24/7 — שם instance רגיל עם Reserved יהיה זול יותר.
- מאחורי הקלעים יש **Proxy Fleet** מנוהל של Aurora, מעל אותו shared storage volume.

### 5.7 Global Aurora

יש שתי דרכים לפרוס Aurora על פני Regions:

| אפשרות | מה זה | מתי |
|---|---|---|
| **Aurora Cross-Region Read Replicas** | replica פשוטה ב-Region אחר | DR בסיסי, קל להקמה |
| **Aurora Global Database** (**המומלץ**) | ארכיטקטורה גלובלית ייעודית | DR רציני + latency גלובלי |

מה שחשוב לזכור על Global Database:

- **Region ראשי אחד** לקריאה וכתיבה.
- עד **10 Regions משניים** לקריאה בלבד.
- **replication lag של פחות משנייה** בין Regions.
- עד **16 Read Replicas בכל Region משני**.
- **קידום Region אחר ל-primary: RTO של פחות מדקה.**

```text
us-east-1 (PRIMARY)                 eu-west-1 (SECONDARY)
Applications R/W ──► Aurora ──<1s──► Aurora ──► Applications Read-Only
                                  (עד 10 Regions, 16 replicas לכל אחד)
```

### 5.8 Aurora Machine Learning ו-Babelfish

**Aurora Machine Learning:**

- מוסיף תחזיות ML לאפליקציה **דרך SQL רגיל** — בלי ידע ב-ML.
- משתלב עם **Amazon SageMaker** (כל מודל) ועם **Amazon Comprehend** (ניתוח סנטימנט).
- Use cases: זיהוי הונאות, מיקוד פרסומות, ניתוח סנטימנט, המלצות מוצרים.

**Babelfish for Aurora PostgreSQL:**

- מאפשר ל-Aurora PostgreSQL להבין פקודות שנכתבו ל-**Microsoft SQL Server** (T-SQL).
- אפליקציות SQL Server עובדות מול Aurora PostgreSQL **עם מעט או ללא שינויי קוד**,
  ועם אותו SQL Server client driver.
- זהו הפתרון לשאלות בסגנון "לצאת מרישוי SQL Server בלי לשכתב את האפליקציה".
- המיגרציה עצמה מתבצעת עם **AWS SCT** ו-**AWS DMS**.

---

## 6. ⚡ Amazon ElastiCache

### 6.1 מה זה ולמה

- מה ש-RDS עושה למסדי נתונים רלציוניים, ElastiCache עושה ל-**Redis** ול-**Memcached**.
- מסדי נתונים **in-memory** — ביצועים גבוהים מאוד, latency תת-מילישנייה.
- מוריד עומס מה-DB ב-workloads עתירי קריאה.
- מאפשר לאפליקציה להיות **stateless** (session store משותף).
- AWS מטפלת ב-OS maintenance, patching, אופטימיזציות, setup, ניטור, התאוששות וגיבויים.

> [!warning] המחיר האמיתי של ElastiCache
> **שימוש ב-ElastiCache דורש שינויי קוד משמעותיים באפליקציה.**
> זו מילת מפתח במבחן: אם השאלה מדגישה "no application code changes" — ElastiCache הוא לא התשובה
> (בדפוס הזה מחפשים בדרך כלל Read Replica, RDS Proxy או DAX).

### 6.2 שני דפוסי הארכיטקטורה

**דפוס 1 — DB Cache:**

```text
Application ──► ElastiCache
                  │ cache hit  → מחזיר מיד
                  │ cache miss → קורא מ-RDS, כותב ל-cache, מחזיר
                  ▼
                 RDS
```

- מוריד עומס מ-RDS באופן דרמטי.
- **חובה אסטרטגיית invalidation** — אחרת מגישים דאטה ישנה.

**דפוס 2 — User Session Store:**

```text
משתמש מתחבר לשרת A ──► כותב session ל-ElastiCache
המשתמש נשלח לשרת B ──► קורא את ה-session מ-ElastiCache → כבר מחובר
```

- זה מה שהופך את שכבת האפליקציה ל-stateless ומאפשר Auto Scaling אמיתי.

### 6.3 Redis מול Memcached — ההשוואה המכריעה

| קריטריון | **Redis** | **Memcached** |
|---|---|---|
| Multi-AZ עם auto-failover | ✅ | ❌ |
| Read Replicas | ✅ (scaling + זמינות) | ❌ |
| Sharding / חלוקת דאטה | ✅ (Cluster mode) | ✅ (multi-node partitioning) |
| High Availability / replication | ✅ | ❌ |
| Persistence | ✅ (AOF) | ❌ **לא persistent** |
| Backup & Restore | ✅ | מוגבל (בגרסת Serverless) |
| מבני נתונים מתקדמים | ✅ **Sets ו-Sorted Sets** | ❌ key-value בלבד |
| ארכיטקטורה | single-threaded | **multi-threaded** |
| מתי בוחרים | צריך HA, persistence, מבני נתונים | cache פשוט וטהור, ניצול מולטי-ליבה |

> [!info] הכלל הפשוט
> אם השאלה מזכירה **זמינות, גיבוי, failover, leaderboard או persistence** — התשובה היא **Redis**.
> אם היא מדגישה **cache פשוט, multi-threaded, ניתן לזרוק** — **Memcached**.

**Redis Sorted Sets ו-Leaderboards:** חישוב טבלת מובילים במשחק הוא יקר מאוד ב-SQL.
Sorted Sets ב-Redis מבטיחים ייחודיות **וגם** סדר: כל אלמנט חדש מדורג בזמן אמת ומוכנס במקום הנכון.
זו התשובה הקנונית ל-"real-time gaming leaderboard".

### 6.4 אבטחה ב-ElastiCache

| מנגנון | פרטים |
|---|---|
| **IAM Authentication** | נתמך ב-Redis |
| **IAM Policies** | **רק לאבטחה ברמת AWS API** — לא שולטות בגישה לדאטה עצמה |
| **Redis AUTH** | סיסמה/token שנקבע ביצירת ה-cluster; שכבה נוספת מעל Security Groups |
| **SSL / TLS** | נתמכת הצפנה in flight |
| **Memcached** | תמיכה ב-**SASL** authentication |
| **Security Groups** | קו ההגנה הבסיסי — מי בכלל מגיע לפורט |

### 6.5 דפוסי Caching

| דפוס | איך זה עובד | היתרון | החיסרון |
|---|---|---|---|
| **Lazy Loading** (cache-aside) | קוראים מה-cache; ב-miss קוראים מה-DB וכותבים ל-cache | נשמרת רק דאטה שבאמת מבוקשת | **דאטה עלולה להתיישן**; ה-miss הראשון איטי |
| **Write Through** | כל כתיבה ל-DB נכתבת גם ל-cache | **אין דאטה ישנה** | כותבים גם דאטה שלעולם לא תיקרא; כתיבה איטית יותר |
| **Session Store** | שמירת session זמני ב-cache עם TTL | stateless application | ה-session אובד אם ה-cache נופל (ב-Memcached) |

> [!tip] הציטוט שכדאי לזכור
> יש רק שני דברים קשים במדעי המחשב: cache invalidation ומתן שמות.
> במבחן, "cache invalidation strategy" היא כמעט תמיד חלק מהתשובה הנכונה כשמדובר ב-Lazy Loading.

---

## 7. 💰 עלות ותמחור

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| Read Replica | instance מלא + אחסון משלה | לכל replica בנפרד |
| Multi-AZ standby | **מכפיל** את עלות ה-instance ואת האחסון | ה-standby מחויב במלואו למרות שאינו מגיש תעבורה |
| Data transfer cross-AZ | **0 ב-RDS Read Replicas באותו Region** | חריג נדיב של AWS |
| Data transfer cross-Region | לכל GB של רפליקציה | מצטבר משמעותית ב-DB עמוס |
| Aurora storage | GB-month, גדל אוטומטית | לא מקצים מראש |
| Aurora I/O | לכל מיליון בקשות I/O (בקונפיגורציה הסטנדרטית) | ב-workload עתיר I/O זה רכיב אמיתי |
| Aurora Serverless | לפי **ACU-שנייה** | אין חיוב כשאין שימוש |
| Aurora Global Database | replicated write I/O בין Regions + instances בכל Region | לא זול |
| ElastiCache | node-hours לפי סוג node | פלוס data transfer |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| Single-AZ RDS | הזול ביותר | dev/test בלבד |
| Multi-AZ | פי ~2 | ייצור, תמיד |
| Aurora | ~20% יותר מ-RDS מקביל | כשצריך ביצועים, HA מובנה או replicas מהירות |
| Aurora Serverless | לפי שימוש | עומס לסירוגין; **יקר יותר** בעומס יציב |
| Aurora Global Database | היקר ביותר | RTO של פחות מדקה בין Regions |
| ElastiCache | node קטן זול משמעותית מ-DB גדול יותר | כשה-cache חוסך upgrade של ה-RDS |

### 🚩 עלויות נסתרות

- **כל Read Replica היא instance מלא** — 5 replicas = פי 6 מעלות ה-compute.
- **cross-Region replication** מחייבת גם על התעבורה וגם על ה-instance בצד השני.
- **Aurora I/O charges** — עומס עתיר קריאות מייצר חשבון I/O משמעותי.
- **Aurora Serverless בעומס יציב** — לרוב יקר יותר מ-instance מוזמן מראש.
- **replicas ששכחו מהן** — נשארות רצות ומחויבות אחרי שהפרויקט ננטש.
- **ElastiCache node שרץ 24/7** גם כשה-hit rate נמוך — cache לא יעיל הוא הוצאה נטו.

### 💡 טיפים לחיסכון

- לפני שמוסיפים Read Replica: לבדוק אם ElastiCache יפתור את אותו עומס בזול יותר.
- להעדיף replicas באותו Region — התעבורה חינם.
- Reserved Instances לכל DB בייצור שרץ ברציפות.
- Aurora Replicas Auto Scaling במקום להחזיק replicas קבועות לשיא העומס.
- Aurora Serverless לסביבות dev שרצות שעות בודדות ביום.
- לנטר `ReplicaLag` — replica שמפגרת קבוע היא סימן ל-under-provisioning, לא לצורך בעוד replica.

---

## 8. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר בנושא הזה | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | failover שלא תורגל הוא הימור | תרגול failover יזום; ניטור `ReplicaLag`, `DatabaseConnections`, `CPUUtilization` |
| Security | כל replica ו-cache הם עותק נוסף של הדאטה הרגישה | הצפנה גם ב-replicas, Redis AUTH + TLS, private subnets לכל node |
| Reliability | Multi-AZ הוא המינימום, לא הבונוס | Multi-AZ בייצור; Aurora Global Database כשה-RTO נמדד בדקות |
| Performance Efficiency | להפריד קריאות מכתיבות ברמת הארכיטקטורה | Reader/Custom Endpoints; ElastiCache לפני שמגדילים instance |
| Cost Optimization | כל replica מכפילה עלות compute | Aurora Auto Scaling במקום replicas קבועות; להעדיף cache על עוד replica |
| Sustainability | replicas ו-cache nodes סרק הם חומרה מבוזבזת | כיבוי סביבות non-prod; Serverless לעומס לסירוגין |

---

## 9. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "reporting queries slow down production" | Read Replica |
| "offload read traffic", "scale reads" | Read Replica / Aurora Replicas |
| "survive an AZ failure automatically", "no manual intervention" | Multi-AZ |
| "synchronous replication" | Multi-AZ |
| "asynchronous", "eventually consistent" | Read Replica |
| "no application changes to connection string" | Multi-AZ (Read Replica **כן** דורשת שינוי) |
| "sub-second replication across Regions", "RTO under one minute" | Aurora Global Database |
| "unpredictable or intermittent workload, no capacity planning" | Aurora Serverless |
| "rewind the database after a bad deployment" | Aurora Backtrack |
| "run SQL Server application on PostgreSQL" | Babelfish for Aurora PostgreSQL |
| "ML predictions using SQL" | Aurora Machine Learning |
| "microsecond/sub-millisecond latency, in-memory" | ElastiCache |
| "gaming leaderboard", "real-time ranking" | ElastiCache for Redis (Sorted Sets) |
| "cache must survive a node failure", "backup the cache" | Redis (לא Memcached) |
| "pure cache, multi-threaded, can be lost" | Memcached |
| "make the application stateless" | ElastiCache session store |
| "requires significant application code changes" | ElastiCache |

### טעויות נפוצות

> [!warning] מלכודת 1 — Multi-AZ כפתרון ביצועים
> **הניסוח:** "Read traffic is overwhelming the database. Improve performance."
> **הטעות:** לבחור "enable Multi-AZ" כי זה נשמע כמו "עוד DB".
> **הנכון:** ה-standby ב-Multi-AZ **אינו מגיש שאילתות כלל**. התשובה היא Read Replica או ElastiCache.

> [!warning] מלכודת 2 — Read Replica כפתרון זמינות
> **הניסוח:** "The database must fail over automatically if the AZ fails."
> **הטעות:** לבחור Read Replica ב-AZ אחר.
> **הנכון:** promotion של replica היא פעולה **ידנית**. failover אוטומטי = Multi-AZ.

> [!warning] מלכודת 3 — replica כגיבוי
> **הניסוח:** "Protect against accidental data deletion."
> **הטעות:** להסתמך על Read Replica.
> **הנכון:** מחיקה משתכפלת ל-replica תוך שניות. גיבוי = automated backups / snapshots / Backtrack.

> [!warning] מלכודת 4 — ElastiCache כשאסור לגעת בקוד
> **הניסוח:** "Reduce database load without modifying the application."
> **הטעות:** לבחור ElastiCache.
> **הנכון:** ElastiCache דורש שינויי קוד משמעותיים. בלי שינויי קוד — Read Replica או RDS Proxy.

> [!warning] מלכודת 5 — Aurora Serverless לעומס יציב
> **הניסוח:** "Production database with steady 24/7 traffic. Optimize cost."
> **הטעות:** לבחור Aurora Serverless כי "משלמים רק על מה שמשתמשים".
> **הנכון:** בעומס יציב, instance מוזמן מראש (Reserved) יהיה זול יותר. Serverless הוא לעומס **לסירוגין**.

> [!warning] מלכודת 6 — Memcached כשצריך שרידות
> **הניסוח:** "The session store must survive a node failure."
> **הטעות:** Memcached, כי הוא מהיר ו-multi-threaded.
> **הנכון:** Memcached **אינו persistent ואין לו replication**. צריך Redis עם Multi-AZ.

---

## 10. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** פלטפורמת מסחר אלקטרוני. עומס קריאות פי 20 מהכתיבות.
הנהלה דורשת שהמערכת תשרוד נפילת AZ בלי התערבות. צוות ה-BI מריץ דוחות כבדים בסוף כל יום.
נפתח שוק באירופה ומשתמשים שם סובלים מ-latency. יש דרישה לטבלת מובילים בזמן אמת במועדון הלקוחות.

```text
us-east-1 (PRIMARY)
┌─────────────────────────────────────────────┐
│  App Tier ──► ElastiCache Redis (Multi-AZ)  │  ← session store + hot catalog
│      │              (Lazy Loading + TTL)     │
│      ├──writes──► Aurora Writer Endpoint     │
│      └──reads───► Aurora Reader Endpoint     │
│                        │                     │
│  BI Team ──────► Aurora Custom Endpoint      │  ← רק ה-instances הגדולים
│                                              │
│  Aurora: 6 עותקים על 3 AZs, Auto Scaling     │
└─────────────────┬───────────────────────────┘
                  │ Global Database (<1s lag)
        eu-west-1 (SECONDARY, read-only)
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Aurora ולא RDS רגיל | יחס קריאה/כתיבה קיצוני; replicas עם lag של פחות מ-10ms ו-failover מהיר |
| ארכיטקטורת 6 עותקים על 3 AZs | HA מובנה — נפילת AZ אינה אירוע; failover של ה-master תוך פחות מ-30 שניות |
| Reader Endpoint לאפליקציה | חלוקת עומס אוטומטית בין ה-replicas ללא לוגיקה באפליקציה |
| **Custom Endpoint** ל-BI | מבודד את הדוחות הכבדים ל-instances ייעודיים; לא פוגע בלקוחות |
| Aurora Replicas Auto Scaling | שיאי עומס בקמפיינים — לא צריך להחזיק replicas לשיא כל היום |
| **Aurora Global Database** ל-eu-west-1 | קריאות מקומיות למשתמשי אירופה; lag של פחות משנייה; RTO של פחות מדקה ל-DR |
| ElastiCache **for Redis** ב-Multi-AZ | session store ששורד נפילת node, ו-cache לקטלוג החם |
| Redis **Sorted Sets** לטבלת המובילים | דירוג בזמן אמת עם ייחודיות וסדר — בלתי אפשרי ביעילות ב-SQL |
| Lazy Loading עם TTL | שומר ב-cache רק את מה שנקרא בפועל; ה-TTL מגביל את גיל הדאטה |

**למה לא Multi-AZ קלאסי של RDS?** הוא היה נותן זמינות אבל **אפס** עזרה לעומס הקריאות — ה-standby סגור.

**למה לא Memcached?** ה-session store היה מתאדה בכל נפילת node, וכל המשתמשים היו מנותקים.

**למה לא Cross-Region Read Replica פשוטה?** היא פותרת DR בסיסי, אבל Global Database נותן lag של פחות משנייה, עד 16 replicas ב-Region המשני, ו-RTO של פחות מדקה.

---

## 11. 🚫 מה לא צריך לדעת למבחן

- ה-internals של פרוטוקול הרפליקציה של Aurora ואופן ה-quorum ברמת המימוש.
- הפרמטרים המדויקים של ACU ב-Aurora Serverless v1 מול v2.
- פקודות Redis ספציפיות ותחביר מבני הנתונים.
- הגדרות `maxmemory-policy` ומדיניות ה-eviction לפרטים.
- מספרי הביצועים המדויקים של Aurora מול MySQL — מספיק "פי כמה", לא benchmarks.
- אופן העבודה של AWS SCT ו-DMS ברמת ה-mapping — מספיק לדעת שהם כלי המיגרציה.

---

## 12. ⚡ Cheat Sheet

- **Read Replica = ביצועי קריאה. Multi-AZ = זמינות.** זו ההבחנה מספר 1 במבחן.
- Read Replica: עד **15**, **ASYNC**, eventually consistent, SELECT בלבד, נדרש שינוי connection string.
- Read Replica ניתנת ל-**promotion ידנית** — זה לא failover אוטומטי.
- Data transfer ל-Read Replica **באותו Region: חינם**. cross-Region: **בתשלום**.
- Multi-AZ: **SYNC**, DNS יחיד, failover אוטומטי, ה-standby **לא נגיש לקריאה**.
- מעבר Single-AZ ל-Multi-AZ הוא **zero downtime** (snapshot → restore ב-AZ אחר → סנכרון).
- Aurora: תואם PostgreSQL/MySQL, ~5x/3x ביצועים, עולה ~20% יותר מ-RDS.
- Aurora אחסון: **6 עותקים על 3 AZs**; **4/6 לכתיבה**, **3/6 לקריאה**; self-healing; גדל בקפיצות 10GB.
- Aurora: עד 15 replicas, lag מתחת ל-10ms, failover של master **תוך פחות מ-30 שניות**.
- Endpoints: Writer (master) · Reader (load balancing) · Custom (תת-קבוצה ייעודית).
- **Backtrack** מגלגל את ה-DB אחורה בלי גיבוי — אבל אינו תחליף לגיבוי.
- Aurora Serverless: pay per second, לעומס **בלתי צפוי או לסירוגין**, ללא capacity planning.
- **Aurora Global Database**: primary אחד, עד **10 Regions משניים**, lag **<1 שנייה**, עד **16 replicas** לכל Region משני, **RTO <1 דקה**.
- Babelfish = הרצת אפליקציות SQL Server (T-SQL) על Aurora PostgreSQL.
- ElastiCache = Redis או Memcached מנוהלים; latency תת-מילישנייה; **דורש שינויי קוד משמעותיים**.
- **Redis**: Multi-AZ + auto-failover, replicas, persistence (AOF), backup, **Sorted Sets** (leaderboards).
- **Memcached**: sharding, multi-threaded, **ללא HA, ללא persistence**.
- Lazy Loading = דאטה עלולה להתיישן. Write Through = אין דאטה ישנה, אבל כותבים גם מה שלא ייקרא.

---

## 13. ✅ בדיקת הבנה

1. צוות ה-BI מתלונן שהדוחות איטיים, וההנהלה דורשת שהמערכת תשרוד נפילת AZ. מה מקימים?
2. למה Read Replica אינה תחליף לגיבוי?
3. באיזה endpoint של Aurora משתמשים כדי לשלוח שאילתות אנליטיות רק ל-instances החזקים?
4. DB dev רץ שעתיים ביום בעומס בלתי צפוי. מה הבחירה הזולה, ומתי היא הופכת ליקרה?
5. אפליקציה שומרת sessions ב-cache. הצוות בחר Memcached כי הוא multi-threaded. מה הסיכון?

<details>
<summary>תשובות</summary>

1. **את שניהם.** Multi-AZ נותן failover אוטומטי בנפילת AZ (ה-standby אינו מגיש שאילתות), ו-**Read Replica** נפרדת מקבלת את עומס הדוחות בלי לגעת בייצור. זו התשובה שהמבחן מחפש כמעט תמיד בשאלות שמשלבות את שתי הדרישות.
2. הרפליקציה היא **ASYNC אבל מיידית**. `DROP TABLE` בטעות ב-primary משתכפל ל-replica תוך שניות. גיבוי הוא נקודת זמן שאפשר לחזור אליה — replica היא עותק חי של ההווה, כולל הטעויות.
3. **Custom Endpoint**. הוא מגדיר תת-קבוצה של instances (למשל רק ה-`db.r5.2xlarge`) ומכוון אליהם את השאילתות הכבדות. אחרי הגדרתו, בדרך כלל כבר לא משתמשים ב-Reader Endpoint.
4. **Aurora Serverless** — חיוב לפי שנייה, ללא capacity planning, מושלם לעומס לסירוגין. הוא הופך ליקר כשהעומס נעשה **יציב ורציף**; שם instance רגיל, ובוודאי Reserved Instance, יהיה זול יותר.
5. Memcached **אינו persistent ואין לו replication או Multi-AZ**. נפילת node מוחקת את כל ה-sessions וכל המשתמשים מנותקים. הבחירה הנכונה היא **Redis** עם Multi-AZ ו-auto-failover.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[21 - RDS Fundamentals]] · [[24 - Database Selection]] · [[33 - High Availability and Scalability]] · [[34 - Disaster Recovery]] · [[15 - CloudFront and Global Delivery]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 2626–3020, 3125–3281
