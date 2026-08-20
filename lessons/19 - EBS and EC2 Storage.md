---
lesson: 19
title: EBS and EC2 Storage
domain: Design High-Performing Architectures
services: [EBS, EC2 Instance Store, EBS Snapshots, AMI, KMS]
tags: [saa-c03, storage, block-storage, ec2]
---

# 19 — EBS and EC2 Storage

> [!abstract] בשורה אחת
> EBS הוא "דיסק ברשת" שנעול ל-AZ בודד ושורד termination, בעוד Instance Store הוא דיסק פיזי מהיר שנמחק בכל stop — והמבחן בודק בעיקר את הבחירה בין סוגי ה-volumes ואת מסלול ה-snapshot.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה EC2 צריך אחסון נפרד מהמכונה |
| 2 | איך זה עובד | volume, attach/detach, AZ binding, delete on termination |
| 3 | פירוק מפורט | 6 סוגי volumes, snapshots, AMI, Instance Store, Multi-Attach, encryption |
| 4 | עלות | על מה בדיוק מחייבים ב-EBS ואיפה מתחבאות ההוצאות |
| 5 | השוואות | gp3 מול gp2, io1 מול io2, EBS מול Instance Store |
| 6 | Well-Architected | איך ששת ה-pillars נראים בהקשר אחסון block |
| 7 | מלכודות | מילות מפתח שמובילות לתשובה הנכונה |
| 8 | Scenario | תכנון אחסון ל-DB על EC2 עם DR |

**מונחי מפתח בשיעור:** `EBS Volume` · `Snapshot` · `AMI` · `Instance Store` · `IOPS` · `Throughput` · `Multi-Attach` · `FSR`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- EC2 instance הוא חומרה זמנית — אם היא מתה, כל מה שכתוב עליה מת איתה.
- Database ו-application state חייבים לשרוד restart, resize ואפילו termination של המכונה.
- צריך יכולת להחליף מכונה בלי להעביר את הדאטה ידנית.
- צריך גיבוי נקודתי שאפשר לשחזר ממנו ב-AZ אחר או ב-Region אחר.

### מה השירות פותר

- **EBS Volume** — דיסק block שמחובר דרך הרשת ל-instance, ומנותק/מחובר מחדש תוך שניות.
- **persistence** — הדאטה שורדת גם אחרי termination של ה-instance (אם לא הוגדר אחרת).
- **provisioned capacity** — אתה קובע גודל, IOPS ו-throughput, ומקבל ביצועים צפויים.
- **Snapshots** — גיבוי incremental שמאפשר להעתיק דאטה בין AZs ובין Regions.
- **AMI** — "תמונה" של מכונה שלמה (OS + software + config) שממנה מרימים instances זהים.

> [!tip] האנלוגיה
> EBS הוא "USB stick ברשת" — נשלף ממכונה אחת ונתקע במכונה אחרת, אבל רק בתוך אותו בניין (AZ).
> Instance Store הוא הדיסק המולחם לתוך המכונה — מהיר בטירוף, אבל הולך עם המכונה.

---

## 2. ⚙️ איך זה עובד

### 2.1 EBS הוא network drive, לא דיסק פיזי

- התקשורת בין ה-instance ל-volume עוברת ברשת של AWS.
- המשמעות: יש מעט latency לעומת דיסק מקומי — זה המחיר של ה-persistence.
- אפשר לנתק (detach) volume ממכונה אחת ולחבר (attach) לאחרת במהירות.
- volume אחד = instance אחד בכל רגע נתון (החריג היחיד: Multi-Attach על io1/io2).

### 2.2 EBS נעול ל-Availability Zone

זו הנקודה שהכי הרבה שאלות במבחן נשענות עליה.

```text
us-east-1a                    us-east-1b
┌──────────────┐              ┌──────────────┐
│ EC2          │              │ EC2          │
│  └─ EBS 100G │   ✗ אסור →   │  (ריק)       │
└──────────────┘              └──────────────┘

המסלול החוקי היחיד:
EBS(1a) → Snapshot → Restore → EBS(1b)
```

- volume ב-`us-east-1a` **לא יכול** להתחבר ל-instance ב-`us-east-1b`.
- כדי להעביר volume בין AZs: snapshot → יצירת volume חדש ב-AZ היעד.
- כדי להעביר בין Regions: snapshot → copy ל-Region היעד → יצירת volume.

### 2.3 Delete on Termination

| Volume | ברירת מחדל | משמעות |
|---|---|---|
| Root volume | מחיקה **מופעלת** | ה-volume נמחק כשה-instance מסתיים |
| כל volume נוסף שמחובר | מחיקה **מכובה** | ה-volume שורד ונשאר unattached |

- אפשר לשנות את ההתנהגות מה-Console או מה-CLI, גם בזמן יצירת ה-instance וגם אחרי.
- Use case קלאסי: שמירת ה-root volume לצורכי forensics או debug אחרי שמכונה הסתיימה.

### 2.4 Snapshots

- גיבוי נקודתי (point-in-time) של ה-volume.
- **incremental** — רק בלוקים שהשתנו מאז ה-snapshot הקודם נשמרים בפועל.
- לא חובה לנתק את ה-volume כדי לצלם, אבל זה מומלץ לעקביות מלאה.
- אפשר להעתיק snapshots בין AZs ובין Regions — זו התשתית ל-DR.
- ה-snapshot נשמר באחסון מנוהל של AWS (עמיד ברמת Region), לא ב-bucket שלך.

### 2.5 מ-volume ל-AMI

```text
הרם EC2 → התקן והגדר → Stop (לשלמות הדאטה) → Create AMI
                                                  │
                                    ┌─────────────┴─────────────┐
                                 snapshots                  metadata
                                                  │
                              Launch instances חדשים מה-AMI
```

- AMI = OS + software + configuration ארוזים יחד.
- יצירת AMI יוצרת מאחורי הקלעים EBS snapshots של ה-volumes.
- AMI שייך ל-**Region** מסוים; להעברה ל-Region אחר צריך copy.
- שלושה מקורות: Public AMI של AWS, AMI פרטי שלך, ו-AMI מה-Marketplace.
- הרווח: זמן boot קצר יותר, כי אין צורך להתקין תוכנה בכל הרמה מחדש.

---

## 3. 🔍 פירוק מפורט

### 3.1 ששת סוגי ה-EBS Volumes

| סוג | מדיה | IOPS מקסימלי | Throughput מקס' | גודל | Boot? | Multi-Attach? | Use case | עלות יחסית |
|---|---|---|---|---|---|---|---|---|
| **gp3** | SSD | 16,000 | 1,000 MiB/s | 1 GiB – 16 TiB | ✅ | ❌ | ברירת מחדל לכל דבר | הזול ב-SSD |
| **gp2** | SSD | 16,000 | 250 MiB/s | 1 GiB – 16 TiB | ✅ | ❌ | legacy, נשאר משיקולי תאימות | ~25% יקר מ-gp3 |
| **io1** | SSD | 64,000 (Nitro) / 32,000 | 1,000 MiB/s | 4 GiB – 16 TiB | ✅ | ✅ | DB קריטי | יקר |
| **io2 Block Express** | SSD | 256,000 | 4,000 MiB/s | 4 GiB – 64 TiB | ✅ | ✅ | DB עם sub-ms latency | היקר ביותר |
| **st1** | HDD | 500 | 500 MiB/s | 125 GiB – 16 TiB | ❌ | ❌ | Big Data, logs, DWH | זול |
| **sc1** | HDD | 250 | 250 MiB/s | 125 GiB – 16 TiB | ❌ | ❌ | ארכיון שניגשים אליו נדיר | הזול ביותר |

> [!warning] שתי עובדות שנשאלות ישירות
> **רק SSD יכול להיות boot volume** — כלומר gp2/gp3/io1/io2. HDD (st1/sc1) לעולם לא.
> **HDD זה throughput, לא IOPS** — st1 נותן 500 MiB/s אבל רק 500 IOPS. random I/O ימות שם.

### 3.2 gp3 מול gp2 — למה gp3 כמעט תמיד עדיף

| היבט | gp2 | gp3 |
|---|---|---|
| קשר בין גודל ל-IOPS | **צמוד**: 3 IOPS לכל GiB | **מנותק**: קובעים בנפרד |
| baseline | תלוי גודל; volumes קטנים מתפרצים עד 3,000 | 3,000 IOPS + 125 MiB/s מובנים |
| להגיע ל-16,000 IOPS | חייב volume בגודל 5,334 GiB | אפשר גם על volume קטן |
| throughput | עד 250 MiB/s | עד 1,000 MiB/s, נקבע בנפרד |
| מחיר ל-GB | בסיס | **זול ב-~20%** מ-gp2 |

**המלכודת המרכזית:** ב-gp2, אם צריך 10,000 IOPS אתה נאלץ לקנות ~3,334 GiB דיסק גם אם הדאטה שלך 200 GB.
ב-gp3 אתה קונה 200 GB ומוסיף IOPS בנפרד — חיסכון דרמטי.

### 3.3 Provisioned IOPS (io1 / io2)

- מיועד ל-workloads שרגישים גם ל-latency וגם ל-**עקביות** של הביצועים.
- הבחירה הטבעית ל-database שרץ על EC2 באופן עצמאי.
- **io1**: 4 GiB עד 16 TiB; עד 64,000 IOPS על instances מבוססי Nitro, 32,000 על השאר.
- **io2 Block Express**: 4 GiB עד 64 TiB; latency תת-מילישנייה; עד 256,000 IOPS.
- ב-io2 היחס המותר הוא עד 1,000 IOPS לכל GiB — הרבה יותר גמיש מ-io1.
- io2 מציע גם durability גבוה יותר מ-io1 באותו מחיר לכל GB.

### 3.4 EC2 Instance Store

- דיסק **פיזי** שמחובר ישירות לשרת שמריץ את ה-instance.
- ביצועי I/O גבוהים במיוחד — מיליוני IOPS בסוגי instances מסוימים.
- **ephemeral**: הדאטה נעלמת ב-stop, ב-hibernate ובכשל חומרה.
- לא ניתן לנתק ולחבר למכונה אחרת, ולא ניתן לגבות ב-snapshot.
- אחריות ה-backup וה-replication היא **שלך** לחלוטין.
- שימושים נכונים: cache, buffer, scratch space, temp files, replication node שיודע להתאושש.

| קריטריון | EBS | Instance Store |
|---|---|---|
| סוג חיבור | רשת (network drive) | פיזי, מחובר לשרת |
| ביצועים | טובים, עם latency רשת | הגבוהים ביותר |
| מה קורה ב-**Stop** | הדאטה נשמרת | **הדאטה נמחקת** |
| מה קורה ב-**Terminate** | תלוי ב-Delete on Termination | הדאטה נמחקת |
| מה קורה בכשל חומרה | הדאטה שורדת (replicated ב-AZ) | הדאטה אבודה |
| ניתן לניתוק/חיבור מחדש | ✅ | ❌ |
| Snapshot | ✅ | ❌ |
| שינוי גודל | ✅ בזמן ריצה | ❌ קבוע לפי סוג ה-instance |
| עלות | תשלום נפרד לכל GB provisioned | כלול במחיר ה-instance |

### 3.5 EBS Multi-Attach

- זמין **רק** למשפחת io1/io2.
- אותו volume מחובר במקביל ל-עד **16 instances**, כולם עם הרשאות read+write מלאות.
- כל ה-instances חייבים להיות ב-**אותו AZ**.
- חובה file system שהוא cluster-aware — XFS ו-EXT4 **לא מתאימים** ויגרמו לנזק.
- האפליקציה עצמה אחראית לתאם כתיבות מקבילות.
- Use case: אפליקציות Linux בקלאסטר שדורשות זמינות גבוהה (למשל Teradata).

> [!warning] Multi-Attach אינו shared file system
> אם השאלה מדברת על "מאות שרתים שקוראים וכותבים לאותם קבצים" — התשובה היא EFS, לא Multi-Attach.

### 3.6 יכולות Snapshot מתקדמות

| יכולת | מה זה עושה | פרטים שכדאי לזכור |
|---|---|---|
| **Snapshot Archive** | מעביר snapshot ל-tier ארכיוני | זול בכ-75%; שחזור לוקח **24–72 שעות** |
| **Recycle Bin** | שומר snapshots שנמחקו בטעות | retention מ-יום אחד עד שנה |
| **Fast Snapshot Restore (FSR)** | initialization מלא מראש | אין latency בגישה הראשונה; **יקר מאוד** |

- ללא FSR, volume ששוחזר מ-snapshot טוען בלוקים "בעצלתיים" מהרקע — הגישות הראשונות איטיות.
- Recycle Bin הוא התשובה לשאלות בסגנון "מישהו מחק snapshot בטעות, איך מתאוששים".

### 3.7 EBS Encryption

מה מקבלים כשמצפינים volume:

- הדאטה at rest מוצפנת בתוך ה-volume.
- הדאטה in flight בין ה-instance ל-volume מוצפנת.
- כל ה-snapshots שנוצרים ממנו מוצפנים.
- כל volume שנוצר מ-snapshot מוצפן — מוצפן גם הוא.

עוד עובדות:

- ההצפנה מבוססת KMS עם AES-256.
- הכל שקוף לאפליקציה — אין שינוי קוד, ואין השפעה משמעותית על latency.
- **snapshot של volume מוצפן הוא תמיד מוצפן.**

**איך מצפינים volume קיים שאינו מוצפן** — זה תרחיש שנשאל שוב ושוב:

```text
1. צור EBS Snapshot מה-volume הלא מוצפן
2. העתק (copy) את ה-snapshot ובחר "encrypt" בפעולת ההעתקה
3. צור volume חדש מה-snapshot המוצפן → ה-volume יוצא מוצפן
4. נתק את הישן וחבר את החדש ל-instance
```

- אין דרך "להפעיל הצפנה" על volume קיים במקום — חייבים לעבור דרך snapshot.
- אפשר להפעיל **EBS Encryption by Default** ברמת Account+Region כדי שכל volume חדש ייווצר מוצפן.

---

## 4. 💰 עלות ותמחור

### על מה מחייבים

| רכיב חיוב | איך נמדד | הערה |
|---|---|---|
| Storage | GB-month של קיבולת **provisioned** | משלמים על מה שהקצית, לא על מה שכתבת |
| IOPS | ב-io1/io2: לכל IOPS שהוקצה. ב-gp3: מעל 3,000 בלבד | ב-gp2 ה-IOPS "כלול" במחיר הגודל |
| Throughput | ב-gp3: מעל 125 MiB/s בלבד | ב-שאר הסוגים כלול |
| Snapshots | GB-month של הדאטה הייחודית שנשמרה | incremental — לא משלמים פעמיים על אותו בלוק |
| Snapshot Archive | GB-month ב-tier זול | פלוס חיוב על פעולת ה-retrieval |
| FSR | חיוב לפי שעה לכל snapshot לכל AZ | מצטבר מהר מאוד |
| Instance Store | 0 עלות נפרדת | כלול במחיר ה-instance |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| sc1 | הזול ביותר | דאטה שניגשים אליה כמה פעמים בחודש |
| st1 | זול | קריאה סדרתית כבדה: logs, Big Data, ETL |
| gp3 | בסיס SSD | **ברירת המחדל הנכונה** לכמעט הכול |
| gp2 | ~25% יקר מ-gp3 באותו גודל | אין סיבה טובה להתחיל איתו היום |
| io1 | יקר | צורך ב-IOPS מעל 16,000 |
| io2 Block Express | היקר ביותר | latency תת-מילישנייה, DB קריטי |
| Instance Store | 0 (כלול) | scratch/cache שאפשר לאבד |

### 🚩 עלויות נסתרות

- **volumes מנותקים** — volume ב-state `available` שלא מחובר לאף instance ממשיך להיות מחויב במלואו.
- **Delete on Termination מכובה** בדיסקי דאטה — אחרי מאות terminations נשארת חווה של volumes יתומים.
- **snapshots שנצברים** — בלי lifecycle policy ה-snapshots מצטברים לנצח.
- **FSR** — קל לשכוח שהוא דולק, והחיוב הוא לפי שעה × AZ × snapshot.
- **over-provisioning של IOPS** ב-io1/io2 — משלמים על IOPS שאף פעם לא מנוצלים.
- **snapshot restore ראשון** — לא עלות כספית אלא עלות ביצועים, וזו לפעמים ההפתעה היקרה יותר.

### 💡 טיפים לחיסכון

- מעבר מ-gp2 ל-gp3 הוא כמעט תמיד חיסכון ישיר של ~20% ללא פגיעה בביצועים.
- מודדים IOPS בפועל ב-CloudWatch לפני שמקצים io1/io2 — הרבה workloads מסתפקים ב-gp3.
- Data Lifecycle Manager לניהול אוטומטי של יצירה ומחיקה של snapshots.
- Snapshot Archive ל-snapshots שנשמרים לצורכי compliance ולא לשחזור מהיר.
- סורקים מדי חודש volumes ב-state `available` ומוחקים.
- ל-cache ו-scratch: להעדיף Instance Store על גבי EBS — זה "בחינם".

---

## 5. ⚖️ השוואות מכריעות

### EBS מול EFS מול Instance Store

| קריטריון | EBS | EFS | Instance Store |
|---|---|---|---|
| סוג אחסון | Block | File (NFS) | Block |
| כמה instances | 1 (או 16 ב-Multi-Attach) | אלפים במקביל | 1 בלבד |
| Scope | AZ אחד | כל ה-Region (multi-AZ) | השרת הפיזי |
| שרידות | שורד termination | שורד הכל | נמחק ב-stop |
| Windows | ✅ | ❌ (Linux/POSIX בלבד) | ✅ |
| מחיר | בינוני | גבוה (~פי 3 מ-gp2) | כלול |
| Auto-scaling של קיבולת | ❌ (שינוי ידני) | ✅ אוטומטי | ❌ |

### gp3 מול io2

| קריטריון | gp3 | io2 Block Express |
|---|---|---|
| תקרת IOPS | 16,000 | 256,000 |
| latency | מילישניות בודדות | תת-מילישנייה |
| Multi-Attach | ❌ | ✅ |
| גודל מקסימלי | 16 TiB | 64 TiB |
| מחיר | נמוך | הגבוה ביותר |

> [!info] שורה תחתונה
> מתחילים תמיד ב-**gp3**. עוברים ל-io1/io2 רק כשמוכיחים צורך במעל 16,000 IOPS או ב-latency תת-מילישנייה.
> אם צריך שיתוף בין הרבה מכונות — לא EBS בכלל, אלא EFS.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר בנושא הזה | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | גיבוי ושחזור חייבים להיות אוטומטיים ומתועדים | Data Lifecycle Manager ל-snapshots + runbook לשחזור volume ל-AZ אחר |
| Security | דאטה במנוחה ובתעבורה חייבת להיות מוצפנת | הפעלת EBS Encryption by Default + KMS key policies נפרדים לסביבות |
| Reliability | EBS הוא AZ-scoped — כשל AZ הורג את ה-volume | snapshots אוטומטיים + copy cross-Region; אפליקציה שפרוסה ב-Multi-AZ |
| Performance Efficiency | לבחור volume לפי דפוס ה-I/O, לא לפי גודל | לנטר `VolumeReadOps`/`WriteOps` ו-`BurstBalance`; לבחור st1 ל-sequential ו-io2 ל-random |
| Cost Optimization | משלמים על provisioned, לא על used | מיגרציה גורפת gp2→gp3, מחיקת volumes מנותקים, Snapshot Archive |
| Sustainability | קיבולת שלא בשימוש היא חומרה שמסתובבת לחינם | right-sizing של volumes, מחיקת AMIs ו-snapshots ישנים שאיש לא משתמש בהם |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "highest possible IOPS", "lowest latency database" | io2 Block Express |
| "cost-effective general purpose", "balanced" | gp3 |
| "need more IOPS without paying for size" | gp3 (או io1/io2 מעל 16,000) |
| "Big Data, log processing, sequential throughput" | st1 |
| "infrequently accessed, lowest cost" | sc1 |
| "temporary, scratch, cache, maximum I/O" | Instance Store |
| "shared across many instances, hundreds of servers" | EFS (לא EBS) |
| "shared between a few Linux cluster nodes, same AZ" | io2 Multi-Attach |
| "move volume to another AZ / Region" | Snapshot → Copy → Restore |
| "recover accidentally deleted snapshot" | Recycle Bin |
| "no latency on first read after restore" | Fast Snapshot Restore |
| "archive snapshots for compliance, cheapest" | Snapshot Archive |
| "encrypt an existing unencrypted volume" | Snapshot → Copy with encryption → new volume |

### טעויות נפוצות

> [!warning] מלכודת 1 — "פשוט תחבר את ה-volume ל-AZ השני"
> **הניסוח:** "The application must be moved to another Availability Zone with its existing data."
> **הטעות:** לבחור תשובה שמציעה detach ואז attach ב-AZ אחר.
> **הנכון:** volume נעול ל-AZ. חייבים snapshot ואז יצירת volume חדש ב-AZ היעד.

> [!warning] מלכודת 2 — HDD כ-boot volume
> **הניסוח:** "Lowest cost boot volume for a fleet of instances."
> **הטעות:** לבחור sc1 כי הוא הכי זול.
> **הנכון:** st1 ו-sc1 **אינם יכולים** לשמש כ-boot volume. הזול ביותר שאפשר הוא gp3.

> [!warning] מלכודת 3 — Instance Store שורד stop
> **הניסוח:** "Instance is stopped nightly to save costs; data must persist."
> **הטעות:** להניח ש-Instance Store נמחק רק ב-terminate.
> **הנכון:** Instance Store נמחק גם ב-**stop**. רק EBS שורד stop.

> [!warning] מלכודת 4 — Multi-Attach כתחליף ל-file share
> **הניסוח:** "Hundreds of web servers across three AZs must serve the same content."
> **הטעות:** לבחור EBS Multi-Attach.
> **הנכון:** Multi-Attach מוגבל ל-16 instances, ל-AZ אחד ול-io1/io2 עם cluster-aware FS. התשובה היא EFS.

> [!warning] מלכודת 5 — "מחקתי את ה-instance, למה הדיסק עדיין מחויב?"
> **הניסוח:** "Costs remain high after terminating hundreds of instances."
> **הטעות:** להניח שכל ה-volumes נמחקו יחד עם ה-instances.
> **הנכון:** רק ה-root volume נמחק כברירת מחדל. דיסקי דאטה נשארים ומחויבים במלואם.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** מסד נתונים PostgreSQL עצמאי שרץ על EC2 (מסיבות רישוי אין אפשרות לעבור ל-RDS).
דורש 20,000 IOPS יציבים, RPO של שעה, ויכולת התאוששות ב-Region אחר תוך פחות מ-4 שעות.

```text
Region A (us-east-1)                         Region B (eu-west-1)
┌──────────────────────────┐                 ┌──────────────────────────┐
│ AZ-1a                    │                 │  EBS Snapshots (copies)  │
│  EC2 (Nitro)             │                 │            │             │
│   ├─ root: gp3 (OS)      │                 │            ▼             │
│   ├─ data: io2 20k IOPS  │─── snapshot ───▶│  Restore → EC2 חדש       │
│   └─ tmp:  Instance Store│    כל שעה (DLM)  │                          │
│      (KMS encrypted)     │──── copy ───────▶                          │
└──────────────────────────┘                 └──────────────────────────┘
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| io2 ל-volume הדאטה | 20,000 IOPS עוברים את תקרת gp3 (16,000); io2 נותן גם עקביות ב-latency |
| instance מבוסס Nitro | נדרש כדי לממש IOPS גבוהים בפועל |
| gp3 ל-root volume | ה-OS לא צריך IOPS מיוחדים — לשלם על io2 שם זה בזבוז |
| Instance Store ל-temp tablespace | I/O הכי מהיר, ו-temp files ממילא ניתנים לשחזור |
| DLM עם snapshot שעתי | עומד ב-RPO של שעה בלי סקריפטים ידניים |
| Copy cross-Region של ה-snapshots | מאפשר restore ב-Region B — snapshot לבדו הוא Region-scoped |
| EBS Encryption + KMS | דאטה רגישה; ההצפנה מכסה גם את ה-snapshots אוטומטית |
| Delete on Termination מכובה על volume הדאטה | הדאטה שורדת גם אם המכונה נהרסת בטעות |

**למה לא Multi-AZ עם Multi-Attach?** Multi-Attach הוא בתוך AZ אחד בלבד ולא נותן שום הגנה מכשל AZ, וגם PostgreSQL אינו cluster-aware על block device משותף.

**למה לא FSR על ה-snapshots?** ב-RTO של 4 שעות אין צורך בשחזור מיידי, וה-FSR היה מוסיף עלות שעתית קבועה.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- שינון מדויק של כל טבלת ה-IOPS/throughput לכל סוג — מספיק לדעת את הסדר והתקרות הגסות.
- פרטי `fio` / benchmarking ו-pre-warming ידני של volumes.
- ההבדלים בין דורות של io1 לעומת io2 שאינו Block Express.
- שמות ה-device (`/dev/xvda` מול `/dev/sda1`) והבדלי mount בין distros.
- אילו סוגי instances בדיוק כוללים NVMe Instance Store ובאיזה גודל.
- מנגנוני ה-burst credit הפנימיים של gp2 ברמת הנוסחה.

---

## 10. ⚡ Cheat Sheet

- EBS = network block storage, **נעול ל-AZ אחד**, שורד termination.
- מעבר AZ או Region נעשה **רק** דרך snapshot.
- Root volume נמחק ב-termination כברירת מחדל; volumes נוספים **לא**.
- Snapshots הם incremental, ברמת Region, וניתנים ל-copy בין Regions.
- **רק SSD (gp2/gp3/io1/io2) יכול להיות boot volume.**
- gp3: 3,000 IOPS ו-125 MiB/s בסיס; IOPS מנותק מהגודל; זול ב-~20% מ-gp2.
- gp2: 3 IOPS לכל GiB; מגיע ל-16,000 IOPS רק ב-5,334 GiB.
- io2 Block Express: עד 256,000 IOPS, עד 64 TiB, latency תת-מילישנייה.
- st1 = throughput (500 MiB/s), sc1 = הכי זול; שניהם HDD ולא bootable.
- Multi-Attach: io1/io2 בלבד, עד 16 instances, אותו AZ, cluster-aware FS חובה.
- Instance Store: הכי מהיר, נמחק ב-**stop**, אין snapshot, הגיבוי באחריותך.
- הצפנת volume קיים = snapshot → copy עם encryption → volume חדש.
- Snapshot Archive זול ב-75% אך שחזור לוקח 24–72 שעות; Recycle Bin שומר מחיקות עד שנה.
- AMI הוא Region-scoped ונוצר מ-EBS snapshots.

---

## 11. ✅ בדיקת הבנה

1. יש לך volume בגודל 500 GB מסוג gp2, והאפליקציה צריכה 8,000 IOPS. מה קורה, ומה הפתרון הזול?
2. מכונה עם Instance Store נעצרת (stop) בלילה לחיסכון. מה קורה לדאטה, ולמה?
3. איך מצפינים volume שכבר קיים ואינו מוצפן, בלי לאבד את הדאטה?
4. הצוות רוצה שכל 40 שרתי ה-web ב-3 AZs יקראו ויכתבו לאותם קבצים. EBS Multi-Attach מתאים?
5. אחרי מחיקה של 300 instances, החשבון על אחסון כמעט לא ירד. למה?

<details>
<summary>תשובות</summary>

1. ב-gp2 ה-IOPS צמודים לגודל: 500 GiB × 3 = 1,500 IOPS בלבד. אין דרך להוסיף IOPS בלי להגדיל את הדיסק ל-~2,667 GiB. הפתרון הזול: מעבר ל-**gp3** — משאירים 500 GB ומקצים 8,000 IOPS בנפרד, וגם משלמים פחות לכל GB.
2. הדאטה **נמחקת**. Instance Store הוא ephemeral ונעלם בכל stop, hibernate או כשל חומרה — לא רק ב-terminate. רק EBS שורד stop.
3. אין הצפנה "במקום". יוצרים snapshot מה-volume, מבצעים **copy** של ה-snapshot ובוחרים encryption בפעולת ההעתקה, יוצרים volume חדש מה-snapshot המוצפן ומחליפים אותו ב-instance.
4. לא. Multi-Attach מוגבל ל-io1/io2, ל-16 instances לכל היותר, ולכולם באותו **AZ** — פרוסה ב-3 AZs זה פסול מיד. בנוסף נדרש cluster-aware file system. התשובה הנכונה היא **EFS**.
5. Delete on Termination מופעל כברירת מחדל רק על ה-**root** volume. כל דיסקי הדאטה שהיו מחוברים נשארו ב-state `available` וממשיכים להיות מחויבים במלוא הקיבולת ה-provisioned.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[05 - EC2 Fundamentals]] · [[06 - EC2 Pricing and Optimization]] · [[20 - EFS and File Storage]] · [[16 - S3 Fundamentals]] · [[35 - Backup and Data Protection]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 1420–1679, 1750–1798
