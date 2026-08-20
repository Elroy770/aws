# 21 — RDS Fundamentals

## 1. מה זה?
Amazon RDS הוא managed relational database עבור engines נפוצים.
## 2. למה צריך את זה?
כדי לקבל SQL, backups, patching ו-monitoring בלי לנהל DB server בסיסי.
## 3. איך זה עובד?
בוחרים engine, instance class, storage ו-DB subnet group בפרייבט subnets. Automated backups מאפשרים point-in-time restore; snapshots ידניים נשמרים עד מחיקה.
## 4. הדברים שחייבים לדעת למבחן
- RDS Multi-AZ מיועד ל-HA, לא לקריאות scaling.
- Read Replica מיועד לקריאות ולפעמים DR.
- DB subnet group מכסה לפחות שני AZs ל-Multi-AZ.
- RDS אינו נגיש אוטומטית לאינטרנט; SG ו-public setting קובעים.
## 5. ההבדלים החשובים
| יכולת | BEST עבור |
|---|---|
| Multi-AZ | failover וזמינות |
| Read Replica | scale reads |
| Snapshot | backup ידני/שכפול |
## 6. מלכודות במבחן
Multi-AZ standby אינו endpoint לקריאות רגילות.
## 7. Scenario מהעולם האמיתי
יישום OLTP צריך שרידות: RDS Multi-AZ ב-private subnets, backups ו-SG שמאפשר רק ל-app tier.
## 8. מה לא צריך לדעת
לא לשנן כל engine version.
## 9. סיכום
- RDS מנהל relational DB.
- Multi-AZ=HA.
- Replica=read scale.
- backups ו-snapshots שונים.
- השאר DB פרטי.
## 10. בדיקת הבנה
1. האם standby קורא queries?
2. מה נותן PITR?
3. היכן ממקמים RDS?

## העמקה: engine, storage, backups ועלויות
RDS מפעיל את ה-database engine, אך הלקוח עדיין אחראי ל-schema, indexes, query tuning ו-connection pooling. DB subnet group מצביע על subnets פרטיים במספר AZs; בדרך כלל מונעים public access ומאפשרים TCP רק מ-SG של application tier. Automated backups מאפשרים PITR בתוך retention window; manual snapshots נשמרים עד מחיקה וניתן להעתיקם ל-Region אחר. Multi-AZ הוא standby/failover, לא read endpoint. Read Replica היא asynchronous לקריאות וניתן לקדם אותה.

משלמים על instance-hours, provisioned storage, I/O או throughput לפי סוג storage, backups מעבר לנפח המוקצה, snapshots, ומספר/Region של replicas. gp3 הוא לרוב cost-effective ל-general purpose; io1/io2 יקרים יותר ל-IOPS קריטי. On-Demand גמיש אך יקר לריצה רציפה; Reserved DB Instances זולים יותר למחויבות ארוכה. הגדלת Multi-AZ/replicas משפרת זמינות או read scale אך מכפילה compute/storage ועלויות transfer אפשריות.

### בדיקת workload לפי ששת ה-pillars
- **Operational Excellence:** Performance Insights, Enhanced Monitoring, automated backups, patch windows ו-runbooks ל-failover.
- **Security:** private subnets, SG מה-app בלבד, encryption/KMS, TLS ו-Secrets Manager במקום credentials בקוד.
- **Reliability:** Multi-AZ ל-HA, PITR/snapshots ל-recovery, read replica/DR לפי RPO/RTO.
- **Performance Efficiency:** instance/storage right-sizing, indexes, pooling ו-read replicas לקריאות; לא להשתמש ב-Multi-AZ כ-read scale.
- **Cost Optimization:** לבחור engine/instance נכון, Reserved עבור baseline, gp3 ו-retention סביר; למחוק replicas/snapshots מיותרים.
- **Sustainability:** right-size, autoscale/read replicas לפי צורך והימנעו מ-idle database capacity.
