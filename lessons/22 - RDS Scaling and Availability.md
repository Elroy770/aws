# 22 — RDS Scaling & Availability

## 1. מה זה?
RDS הוא managed relational database. **Multi-AZ** מיועד ל-HA של writer, **Read Replicas** להגדלת reads, ו-Aurora (כולל Replicas ו-Serverless) ל-scale/availability מתקדמים.

## 2. למה צריך את זה?
מסד transactional צריך להמשיך לפעול ב-AZ failure, אך דוחות עלולים להעמיס על ה-writer. Replica ו-Multi-AZ פותרים בעיות שונות.

## 3. איך זה עובד?
- Multi-AZ מפעיל standby synchronously ב-AZ אחר; הכתיבה ל-writer endpoint, ובכשל AWS מפנה אותו ל-standby. ה-standby אינו read endpoint.
- Read Replica משכפלת asynchronously. מנתבים reads ל-replica endpoint; lag אפשרי. אפשר ליצור replicas ב-Region אחר ולקדם אחת ידנית.
- Aurora משתמשת ב-cluster storage משוכפל על פני AZs, עם writer ו-reader endpoints. Aurora Replicas מתאימות ל-read scaling ול-failover.
- Aurora Serverless v2 מגדילה/מקטינה compute לקיבולת משתנה; היא אינה פותרת query לא יעיל או connection pooling.

## 4. הדברים שחייבים לדעת למבחן
- Multi-AZ = availability/failover, לא read scaling.
- Read Replica = asynchronous read scaling; lag ו-eventual consistency.
- Multi-AZ ו-Read Replica יכולים להופיע יחד.
- Promotion של replica אינו failover אוטומטי כמו Multi-AZ.
- Aurora reader endpoint מפזר reads; writer endpoint לכתיבות.
- Backups, encryption, subnet groups ו-security groups עדיין חלק מה-design.

## 5. עלות, תמחור ו-trade-offs
משלמים DB instance-hours, storage, I/O לפי storage type, backups מעבר להקצאה, snapshots ו-data transfer. Multi-AZ מייקר כי יש standby, אך נותן RTO טוב יותר. Read Replica מוסיפה instance ו-storage; cross-Region מוסיפה inter-Region transfer. Aurora לרוב יקרה יותר מ-RDS קטן אך עשויה לתת throughput/availability גבוהים; Serverless חוסכת idle capacity אך עלולה לעלות יותר בעומס יציב. Reserved Instances חוסכים ל-capacity יציבה; On-Demand גמיש.

## 6. ההבדלים החשובים
| דרישה | בחירה | trade-off |
|---|---|---|
| AZ failure עם failover | RDS Multi-AZ/Aurora writer | עלות standby; לא לקריאה |
| הרבה reads | Read Replica/Aurora Replicas | async lag ועלות instance |
| DR ב-Region אחר | Cross-Region replica/snapshot | transfer ו-RTO גבוה יותר |
| traffic לא צפוי | Aurora Serverless v2 | פחות idle, scaling/connection concerns |

## 7. Well-Architected view
- **Operational Excellence:** CloudWatch על connections/CPU/replica lag; runbook ותרגול failover.
- **Security:** private subnets, TLS, KMS, Secrets Manager ו-least privilege.
- **Reliability:** Multi-AZ, automated backups ו-test restore; replica אינה backup.
- **Performance Efficiency:** הפרדת read/write endpoints, indexes ו-replicas; לא להסתיר query בעייתי.
- **Cost Optimization:** right-size, Reserved ל-capacity יציבה, מחיקת replicas/snapshots מיותרים.
- **Sustainability:** לכבות non-production ולמנוע overprovisioning באמצעות scaling מתאים.

## 8. מלכודות במבחן
"דוחות לא יפגעו ב-production" → Read Replica. "Failover אוטומטי ל-AZ אחר" → Multi-AZ. "הכי זול ל-DB קטן ויציב" → ייתכן RDS רגיל, לא Aurora Serverless.

## 9. Scenario מהעולם האמיתי
מערכת הזמנות צריכה writer זמין ודוחות כבדים: RDS Multi-AZ ל-writer ו-Read Replica לדוחות; האפליקציה משתמשת ב-endpoints נפרדים ומטפלת ב-stale reads.

## 10. מה לא צריך לדעת
לא לשנן replication internals או מספרי pricing; חשוב לזהות sync מול async ו-RTO/RPO.

## 11. סיכום
1. Multi-AZ = HA writer. 2. Replica = reads. 3. Replica lag קיים. 4. Aurora מפרידה cluster storage/compute. 5. Serverless לקיבולת משתנה. 6. משלמים גם transfer/backups.

## 12. בדיקת הבנה
1. למה Multi-AZ לא פותר read scaling? 2. מה הסיכון ב-read replica? 3. איזה endpoint משמש לכתיבה ב-Aurora?
