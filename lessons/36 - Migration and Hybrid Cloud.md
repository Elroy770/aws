# 36 — Migration & Hybrid Cloud | Migration Strategies

## 1. מה זה?

Migration הוא העברת applications, servers, databases ו-data מ-on-premises או מענן אחר ל-AWS. Hybrid Cloud משלב workload מקומי עם AWS דרך network מאובטח, ולעיתים הוא שלב מעבר לפני decommission של ה-datacenter. ב-SAA, בוחנים יעד, downtime, RPO/RTO, compatibility ו-operational burden — לא רק את כלי ההעתקה.

## 2. למה צריך את זה?

ארגון עשוי לרצות לצאת מ-datacenter יקר, לקבל elasticity, או להוסיף DR בלי לשכתב מערכת legacy. Constraints נפוצים הם latency למערכת מקומית, data sovereignty, bandwidth מוגבל, חלון downtime קצר או database engine שאינו נתמך ביעד.

## 3. איך זה עובד?

| אסטרטגיה | מה עושים | מתי מתאים |
|---|---|---|
| Rehost | lift-and-shift כמעט ללא שינוי | migration מהיר |
| Relocate | מעבירים יחידה שלמה, למשל VMware | שומרים operational model |
| Replatform | התאמה קטנה, למשל EC2 ל-RDS | פחות patching בלי rewrite מלא |
| Repurchase | מחליפים ל-SaaS | מבטלים ניהול product |
| Refactor/Re-architect | מעצבים מחדש ל-cloud-native | מקסימום benefit, זמן/סיכון גבוהים |
| Retain / Retire | משאירים זמנית / מסירים מערכת | dependency או חיסכון מיידי |

**AWS Application Migration Service (MGN)** מתקין replication agent, משכפל block-level data ל-staging area, ומאפשר test launch ולאחר מכן cutover של שרתים פיזיים/VM/cloud. הוא מתאים ל-rehost, לא להמרת schema של DB.

**AWS Database Migration Service (DMS)** מבצע Full Load ולאחריו CDC (Change Data Capture), כך שניתן לצמצם downtime. **Schema Conversion Tool (SCT)** מסייע במעבר בין engines שונים; DMS לבדו אינו תמיד ממיר schema או stored procedures.

**Snow Family** מעבירה data offline כש-bandwidth אינו מספיק: Snowcone ל-edge/נפח קטן, Snowball Edge ל-data גדול או processing מקומי, ו-Snowmobile ל-scale עצום. **Storage Gateway** מחבר on-premises ל-AWS: File Gateway מציג NFS/SMB עם S3 backend; Volume Gateway מספק volumes ו-cache/backup; Tape Gateway מחליף tape library ב-VTL.

Hybrid connectivity משתמשת ב-Site-to-Site VPN (הקמה מהירה, מוצפן דרך Internet) או Direct Connect (private ויציב יותר, אך provisioning יקר ואיטי יותר; לרוב מוסיפים VPN כ-backup).

## 4. הדברים שחייבים לדעת למבחן

- Lift-and-shift של servers → MGN; DB עם minimal downtime → DMS + CDC.
- שינוי engine/schema → בדוק SCT; שינוי architecture → refactor.
- Bandwidth מוגבל ו-data רב → Snow.
- File access מקומי ל-S3 → File Gateway; archive tape → Tape Gateway.
- Test launch לפני cutover מגלה drivers, IP assumptions ו-dependencies.
- תכנן waves, dependency mapping, rollback ו-DNS cutover.

## 5. עלות, תמחור ו-trade-offs

משלמים על שירותי migration/replication, storage ל-staging ול-target, compute/databases, requests ו-data transfer. MGN מוסיף replication servers ו-EBS; DMS מחויב לפי replication instance ושעות ריצה; Snow כולל device/handling ו-return logistics; Storage Gateway כולל gateway, storage/requests ו-transfer.

בדרך כלל VPN זול ומהיר יותר מ-Direct Connect להתחלה, אך DX מצדיק עצמו ב-throughput קבוע ו-critical hybrid traffic. Rehost זול בזמן הפרויקט אך עלול להשאיר instances יקרים; replatform דורש עבודה אך מפחית patching. S3 Glacier זול לאחסון קר אך retrieval ו-minimum-storage duration יקרים יותר. כלול downtime, engineering hours, support ו-run-rate — לא רק מחיר ההעברה.

## 6. ההבדלים החשובים

| צורך | בחירה | לא לבחור |
|---|---|---|
| Server migration | MGN | DMS |
| Heterogeneous DB | SCT + DMS | MGN |
| Bulk data בלי bandwidth | Snow | upload ארוך דרך Internet |
| Local SMB/NFS עם S3 backend | File Gateway | mount רגיל של S3 |
| Private predictable connectivity | Direct Connect (+ VPN backup) | VPN בלבד ל-throughput קריטי |

## 7. Well-Architected view

- **Operational Excellence:** migration factory, discovery, waves, runbooks, monitoring ו-rollback.
- **Security:** least-privilege IAM, encryption in transit/at rest, private subnets ו-no public migration endpoints.
- **Reliability:** CDC, test cutover, checkpoints, retries, backup ו-VPN/DX redundancy.
- **Performance Efficiency:** התאמת replication instance/bandwidth, וקרבה בין source, target ומשתמשים.
- **Cost Optimization:** retire assets, כבה replication לאחר cutover, והשווה Snow מול network transfer.
- **Sustainability:** decommission חומרה, העבר רק data נדרש, והעדף managed services על ציוד idle.

## 8. מלכודות במבחן

- DMS אינו general server migration; MGN אינו schema converter.
- “Minimal downtime” אינו “zero downtime”: צריך CDC, validation ו-cutover.
- Snow פותר bandwidth, לא transformation או compatibility.
- Hybrid אינו מחייב public IP; VPN/DX ו-private addressing הם ברירת מחדל.

## 9. Scenario מהעולם האמיתי

חברה מעבירה 200 VMs במהירות, אבל MySQL צריך להיכנס ל-Aurora עם downtime של דקות. משתמשים ב-MGN לשרתים וב-SCT (אם נדרש) + DMS Full Load/CDC ל-DB. מריצים test launch, מאמתים data, עוצרים writes וממתינים ל-CDC lag לפני cutover. זה עדיף על rewrite מלא תחת deadline.

## 10. מה לא צריך לדעת

אין צורך לשנן capacity מדויק של כל Snow device, commands של agent או כל limitation ספציפי. חשוב לזהות כלי, זרימת cutover וה-trade-off.

## 11. סיכום

- 7R נבחרים לפי business outcome.
- MGN = servers; DMS = databases; SCT = schema.
- CDC מצמצם downtime אך דורש validation.
- Snow מתאים ל-data רב ו-network חלש.
- Storage Gateway הוא bridge ל-hybrid.
- DX יציב/private; VPN פשוט וזול יותר.
- חשב staging, transfer, operations ו-run-rate.

## 12. בדיקת הבנה

1. איזה שירות מתאים ל-lift-and-shift של VM?
2. למה מוסיפים CDC ב-DMS?
3. מתי Snow עדיף על upload דרך network?
