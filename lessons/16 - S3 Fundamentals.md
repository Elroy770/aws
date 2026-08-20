# 16 — S3 Fundamentals

## 1. מה זה?
Amazon S3 הוא object storage אזורי, עמיד מאוד ובעל buckets ו-objects.

## 2. למה צריך את זה?
הוא מתאים לאחסון סטטי, backups, data lakes, logs והפצת קבצים בלי ניהול filesystem או capacity מראש.

## 3. איך זה עובד?
Object נשמר תחת key ב-bucket. S3 Standard מתאים לגישה תכופה; Intelligent-Tiering מתאים לדפוס לא ידוע; Standard-IA/One Zone-IA/Glacier classes הן trade-off של עלות, זמינות וזמן retrieval. S3 נותן strong read-after-write consistency. Bucket name הוא global namespace.

## 4. הדברים שחייבים לדעת למבחן
- S3 אינו block/file system ולא ניתן mount רגיל כמו EFS.
- storage class נבחר לפי access pattern ו-retrieval time.
- Versioning מגן מפני overwrite/delete.
- S3 הוא regional; replication חייבת להיות מוגדרת.

## 5. ההבדלים החשובים
| Class | BEST עבור |
|---|---|
| Standard | גישה תכופה |
| Intelligent-Tiering | דפוס לא צפוי |
| Standard-IA | data נדיר אך מהיר לגישה |
| Glacier | archive עם retrieval trade-off |

## 6. מלכודות במבחן
"Lowest storage cost" אינו בהכרח הפתרון אם נדרש immediate retrieval. One Zone-IA אינו מתאים לנתונים שלא ניתן ליצור מחדש.

## 7. Scenario מהעולם האמיתי
דוחות חודשיים נשמרים ב-S3, מוגנים ב-versioning ומועברים lifecycle ל-archive אחרי תקופת גישה חמה.

## 8. מה לא צריך לדעת
אין צורך לשנן זמני retrieval מדויקים או מחירי GB.

## 9. סיכום
- S3 = object storage.
- בחר class לפי access/retrieval.
- Versioning מגן מטעויות.
- bucket הוא regional, name גלובלי.
- lifecycle אוטומטי חוסך עלות.

## 10. בדיקת הבנה
1. האם S3 הוא shared POSIX filesystem?
2. מה מגן מ-overwrite?
3. מה מתאים ל-archive?

## העמקה: מודל נתונים, עלויות וששת ה-pillars
S3 הוא flat namespace: ה-folders הם prefixes בתוך key, לא directories. Object כולל data, key, metadata ו-tags, ולכן עדכון חלקי אינו כמו block write. `PUT` ו-`DELETE` נהנים מ-strong consistency. Bucket policy ו-IAM קובעים גישה; static website endpoint אינו תחליף ל-CloudFront + OAC.

### עלויות ו-trade-offs
החיוב כולל GB-month, PUT/GET/LIST requests, data transfer out, retrieval, lifecycle transitions ו-replication. Standard יקר יותר לאחסון אך מתאים לגישה תכופה וללא retrieval fee. Intelligent-Tiering גובה monitoring fee קטן וחוסך כשדפוס הגישה לא ידוע. IA זול יותר לאחסון אך כולל retrieval ו-minimum duration; One Zone-IA זול עוד יותר אך AZ יחיד ומתאים לנתונים שניתן לשחזר. Glacier Flexible Retrieval/Deep Archive זולים לארכיון ארוך, תמורת retrieval של דקות/שעות ועלויות retrieval. Lifecycle ל-abort incomplete multipart חוסך חלקים נטושים.

- **Operational Excellence:** lifecycle, inventory, tags ו-alerts על growth ו-4xx/5xx.
- **Security:** Block Public Access, least privilege, TLS ו-encryption.
- **Reliability:** versioning/replication/backups לפי RPO; region יחיד אינו DR.
- **Performance Efficiency:** multipart/parallel transfer, CloudFront ו-access patterns מתאימים.
- **Cost Optimization:** מעבר אוטומטי ל-IA/Glacier, מחיקת uploads חלקיים וניתוח egress.
- **Sustainability:** compression, formats יעילים ו-archive של נתונים לא פעילים.
