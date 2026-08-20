# 24 — Database Selection

## 1. מה זה?
בחירת datastore לפי data model, queries, latency, scale ו-operations.
## 2. למה צריך את זה?
אין "database הכי טוב"; service selection הוא דפוס מרכזי ב-SAA.
## 3. איך זה עובד?
RDS/Aurora מיועדים ל-relational SQL ו-transactions; DynamoDB ל-key-value massive scale; ElastiCache ל-cache; Redshift ל-analytics; OpenSearch לחיפוש. שאל תחילה: האם נדרשים joins? איזה access pattern? writes/reads? latency? global?
## 4. הדברים שחייבים לדעת למבחן
- SQL transactions/joins → RDS או Aurora.
- predictable key lookup → DynamoDB.
- cache מול DB → ElastiCache, לא source of truth.
- analytics warehouse → Redshift, לא RDS.
## 5. ההבדלים החשובים
| צורך | שירות |
|---|---|
| OLTP relational | RDS/Aurora |
| serverless key-value | DynamoDB |
| caching | ElastiCache |
| full-text search | OpenSearch |
## 6. מלכודות במבחן
אל תבחר DynamoDB רק בגלל "scalable" כאשר נדרשים joins מורכבים.
## 7. Scenario מהעולם האמיתי
e-commerce: Aurora להזמנות ותשלומים, DynamoDB לעגלת session, ElastiCache לקטלוג חם.
## 8. מה לא צריך לדעת
לא לשנן כל engine feature.
## 9. סיכום
- התחל מה-query pattern.
- relational ≠ key-value.
- cache אינו durable source.
- purpose-built בדרך כלל BEST.
- managed מפחית operational load.
## 10. בדיקת הבנה
1. מה מתאים ל-joins?
2. מה מתאים ל-session key lookup?
3. מה עושה ElastiCache?
+

## 5. עלות, תמחור ו-trade-offs
RDS/Aurora מחויבים instance-hours, storage, I/O ו-backups; Aurora לרוב premium. DynamoDB on-demand גמיש אך provisioned זול יותר בעומס יציב. ElastiCache מחויב לפי nodes אך חוסך DB compute כש-hit rate גבוה. Redshift מתאים ל-analytics אך לא OLTP; S3 זול ל-retained objects אך לא transactions. אל תשכח transfer ו-cross-Region replication.

## 6. Well-Architected view
- **Operational Excellence:** managed service, runbooks ו-monitoring מותאם workload.
- **Security:** encryption, private connectivity, IAM/DB auth ו-secrets rotation.
- **Reliability:** Multi-AZ/backups, PITR ו-restore tests.
- **Performance Efficiency:** engine לפי query, indexes, partitioning ו-cache.
- **Cost Optimization:** right-size, lifecycle/TTL ו-reserved/provisioned capacity.
- **Sustainability:** purpose-built store ו-avoid overprovisioning.

## 7. מלכודות ו-Scenario
DynamoDB אינו ל-joins, ElastiCache אינו backup, Redshift אינו transactional DB. E-commerce יכול להשתמש ב-Aurora להזמנות, DynamoDB לעגלה, ElastiCache לקטלוג, S3 לתמונות ו-Redshift לניתוח.
