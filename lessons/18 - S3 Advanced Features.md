# 18 — S3 Advanced Features

## 1. מה זה?
Replication, presigned URLs, multipart upload, Transfer Acceleration ו-event notifications מרחיבים את S3.

## 2. למה צריך את זה?
כדי להפיץ data, לאפשר upload/download זמני, להעלות אובייקטים גדולים ולקשר events ל-workflows.

## 3. איך זה עובד?
CRR/SRR משכפל objects בין/בתוך Regions ודורש versioning בשני buckets; הוא מיועד בעיקר ל-objects חדשים. Presigned URL נותן הרשאה זמנית לפעולה מסוימת ללא credentials אצל המשתמש. Multipart upload מחלק אובייקט לחלקים. Event notification מפעיל SNS, SQS או Lambda; EventBridge נותן יכולות routing נוספות.

## 4. הדברים שחייבים לדעת למבחן
- CRR = DR/קרבה גלובלית; SRR = aggregation/logs/אותו Region.
- Presigned URL אינו הופך bucket לציבורי.
- large upload/retry → multipart.
- events יכולים להימסר יותר מפעם אחת: handlers צריכים idempotency.

## 5. ההבדלים החשובים
| Feature | BEST עבור |
|---|---|
| CRR | copy בין Regions |
| Presigned URL | upload זמני ישיר ל-S3 |
| Transfer Acceleration | upload מהיר מרחוק ל-bucket |
| EventBridge | rules רבים/targets רבים |

## 6. מלכודות במבחן
Replication אינה backup להגנה מ-delete אם delete markers/replication rules לא מתוכננים. Event notification לא מבטיחה exactly-once processing.

## 7. Scenario מהעולם האמיתי
לקוח מעלה וידאו עם presigned URL; S3 event שולח message ל-SQS והעובד מעבד אותו באופן idempotent. אין צורך לפתוח credentials ללקוח.

## 8. מה לא צריך לדעת
לא לשנן multipart part size או כל replication option.

## 9. סיכום
- Versioning נדרש ל-replication.
- URL חתום הוא זמני ומוגבל.
- multipart לגדולים/retry.
- events דורשים idempotency.
- בחר CRR לפי Region requirement.

## 10. בדיקת הבנה
1. מה נדרש לפני CRR?
2. מה נותן presigned URL?
3. למה consumer של S3 event צריך idempotency?

## העמקה: workflows, failure modes ועלויות
CRR/SRR הם asynchronous replication: versioning נדרש בשני buckets, והרשאות replication role ו-KMS key נפרדות. Replication אינו backup מושלם—מחיקות ו-delete markers דורשות explicit design, וצריך לבדוק replication status ו-RPO. Presigned URL יורש את הרשאות היוצר, מוגבל ב-expiration וב-method, ואינו הופך bucket לציבורי. Multipart מאפשר parallel upload/retry; יש להשלים או abort חלקים.

S3 notifications יכולות להגיע יותר מפעם אחת ובסדר לא מובטח, ולכן consumer צריך idempotency ו-DLQ/retry מתאים. SQS נותן buffering, SNS fan-out, ו-EventBridge filtering/routing רחב יותר. Transfer Acceleration משתמש ב-edge לקבלת upload מרוחק, אך מוסיף data-transfer charges; הוא משתלם רק כש-latency upload משמעותי. CRR מוסיף storage ו-cross-Region transfer, אך עשוי להצדיק את עצמו עבור DR/compliance.

### בדיקת workload לפי ששת ה-pillars
- **Operational Excellence:** metrics על replication lag, retries ו-DLQ; runbooks ו-idempotent consumers.
- **Security:** presigned URLs קצרים, least privilege, OAC/KMS והרשאות replication מינימליות.
- **Reliability:** versioning, CRR לדרישת Region ו-retry-safe processing; לא להניח exactly-once.
- **Performance Efficiency:** multipart/parallelism, Transfer Acceleration רק לנתיבים מתאימים ו-event filtering.
- **Cost Optimization:** lifecycle ל-replicas, בחינת acceleration ו-SQS במקום polling יקר.
- **Sustainability:** עיבוד event-driven, multipart נקי ו-replication רק לנתונים עם צורך עסקי.
