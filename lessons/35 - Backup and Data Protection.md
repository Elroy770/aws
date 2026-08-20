# 35 — Backup & Data Protection

## 1. מה זה?
Backup הוא recovery point שניתן לשחזר ממנו; data protection כוללת retention, encryption, immutability, replication ובדיקת restore. AWS Backup מרכז policies, schedules, vaults ו־copy למשאבים נתמכים.

## 2. למה צריך את זה?
גיבוי עקבי מצמצם טעויות ידניות ומאפשר התאוששות ממחיקה, corruption או ransomware. Backup אינו HA ואינו replication online; הוא חלק מ־DR ו־compliance.

## 3. איך זה עובד?
Backup plan בוחר resources לפי tags או assignments, קובע schedule, retention ו־lifecycle ל־cold storage, וכותב ל־backup vault. Vault Lock יכול לאכוף WORM/retention. Copy ל־account ול־Region אחרים מבודד את נקודות השחזור. KMS מצפין backups; יש להגן גם על policy, access ו־recovery account. Restore testing בודק שהנתונים וה־application dependencies באמת ניתנים לשימוש.

## 4. הדברים שחייבים לדעת למבחן
- backup אינו zero-downtime ואינו failover.
- cross-account/Region copy מגן מ־account compromise או Region failure.
- Vault Lock/immutable retention מסייע נגד מחיקה זדונית.
- retention ו־lifecycle הם trade-off בין cost ל־compliance/RPO.
- RPO מכתיב schedule; RTO מכתיב restore automation ו־capacity.
- tags יכולים לפספס resources אם governance חלש; validate coverage.

## 5. עלות, תמחור ו־trade-offs
משלמים עבור backup storage לפי נפח/סוג, requests או פעולות copy/restore, ו־cross-Region data transfer. Cold/archive tiers זולים לאחסון אך restore איטי ועלול לכלול retrieval/minimum-duration costs. יותר snapshots תכופים נותנים RPO טוב יותר אך מגדילים storage. AWS Backup מפחית operational toil אך השירותים עשויים להציע native snapshots במחיר/יכולות שונים. Replicas online יקרות יותר מ־backup אך מספקות RTO נמוך יותר.

## 6. ההבדלים החשובים
| מנגנון | מטרה | עלות/מגבלה |
|---|---|---|
| Backup | recovery point לאחר כשל | זול יחסית, restore איטי |
| Snapshot | point-in-time של resource | תלוי resource ו־retention |
| Replica | זמינות/קריאה או DR מהיר | capacity ו־replication יקרים |
| Vault Lock | immutability ו־retention enforcement | מגביל מחיקה לגיטימית |
| Cross-account/Region copy | isolation ו־regional recovery | storage/transfer כפולים |

## 7. Well-Architected view
- **Operational Excellence:** central plans, tag governance, restore runbooks, audit ו־restore drills.
- **Security:** separate backup account, KMS, Vault Lock, least privilege ו־MFA delete controls.
- **Reliability:** multiple recovery points, cross-Region copies, tested dependencies ו־known RPO/RTO.
- **Performance Efficiency:** parallel restore, prebuilt templates ו־archive רק לנתונים שאינם hot.
- **Cost Optimization:** deduplication/incremental snapshots, lifecycle ל־cold tier ו־retention לפי business need.
- **Sustainability:** incremental backups, compression ו־הימנעות מהעתקות מיותרות חוסכים storage/compute.

## 8. מלכודות במבחן
Snapshot באותו account/Region אינו מגן מ־account compromise. Backup אינו מחליף Multi-AZ או replica. Vault Lock אינו encryption. “Backup succeeded” אינו מוכיח שהאפליקציה תעלה; יש לבדוק restore, permissions, keys ו־DNS.

## 9. Scenario מהעולם האמיתי
Tags מסמנים production RDS/EBS. AWS Backup מריץ daily ו־weekly plans, מעביר ישנים ל־cold tier ומעתיק ל־backup account ב־Region נוסף עם Vault Lock. רבעונית מבוצע restore לחשבון isolated ונבדק RTO.

## 10. מה לא צריך לדעת
לא לשנן כל resource type או quota נתמך. התמקד ב־retention, isolation, encryption, lifecycle, RPO/RTO ו־restore testing.

## 11. סיכום
1. Backup הוא recovery point.
2. Backup אינו HA או replication.
3. Copy לחשבון/Region אחר נותן isolation.
4. Vault Lock מונע שינוי retention.
5. RPO מכתיב תדירות.
6. RTO מכתיב restore automation.
7. lifecycle מוזיל אך מאריך restore.
8. תמיד test restore.

## 12. בדיקת הבנה
1. מה מגן מ־account compromise?
2. האם backup נותן zero downtime?
3. למה test restore?

