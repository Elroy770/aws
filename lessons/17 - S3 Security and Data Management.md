# 17 — S3 Security & Data Management

## 1. מה זה?
כלי השליטה בגישה, הצפנה ושימור נתונים ב-S3.

## 2. למה צריך את זה?
S3 פשוט לפתיחה אך data exposure או מחיקה לא מכוונת הם סיכונים מרכזיים.

## 3. איך זה עובד?
Block Public Access חוסם policies/ACLs ציבוריים. IAM policies מנהלות זהות; bucket policy היא resource policy ומתאימה ל-cross-account או דרישות endpoint. SSE-S3 מצפין עם מפתחות AWS; SSE-KMS מוסיף שליטה במפתח, audit והרשאות KMS. ACLs הן legacy ברוב designs; Object Ownership יכול לבטל ACLs.

## 4. הדברים שחייבים לדעת למבחן
- חסום public access כברירת מחדל.
- SSE-KMS דורש גם S3 permissions וגם KMS permissions.
- Bucket policy יכולה להגביל לפי `aws:SourceVpce`.
- Versioning + MFA Delete/retention לפי דרישה הם controls נפרדים.

## 5. ההבדלים החשובים
| הצפנה | מתי לבחור |
|---|---|
| SSE-S3 | encryption managed ופשוט |
| SSE-KMS | audit/control key או policy דרישה |
| Client-side | הלקוח חייב להצפין לפני upload |

## 6. מלכודות במבחן
S3 encryption-at-rest אינו מחליף TLS in transit. KMS key policy לא בהכרח מאפשרת למשתמש להשתמש במפתח בלי IAM מתאים.

## 7. Scenario מהעולם האמיתי
Bucket פיננסי פרטי משתמש ב-SSE-KMS, Block Public Access, versioning ו-bucket policy שמאפשרת גישה רק דרך VPC endpoint מאושר.

## 8. מה לא צריך לדעת
לא צריך לשנן headers של כל encryption mode.

## 9. סיכום
- סגור public access.
- IAM ו-bucket policy משלימים.
- SSE-KMS מוסיף governance.
- ACLs לרוב אינן הבחירה החדשה.
- הפרד encryption, access ו-retention.

## 10. בדיקת הבנה
1. מה מוסיף SSE-KMS מעל SSE-S3?
2. איזה control חוסם public policies?
3. האם הצפנה מחליפה IAM?

## העמקה: access, encryption, retention ועלויות
בכל request יש שילוב של identity policy, resource policy, explicit deny ו-SCP; deny מפורש מנצח allow. `aws:SourceVpce` יכול להגביל bucket ל-VPC endpoint. SSE-S3 הוא server-side encryption פשוט; SSE-KMS מוסיף key policy, grants ו-CloudTrail audit אך גם KMS API usage. Client-side encryption משאירה plaintext אצל הלקוח ומעבירה אליו את ניהול המפתחות. Versioning יוצר delete marker; Object Lock, retention ו-legal hold מיועדים ל-WORM ואינם תחליף ל-replication.

SSE-S3 לרוב זול ופשוט יותר. SSE-KMS מתאים ל-customer-controlled key, rotation ו-audit, אך KMS request/key charges ומגבלות throughput עשויים להיות trade-off. Versioning ו-Object Lock עלולים להגדיל storage אם אין lifecycle לגרסאות; CloudTrail data events ו-Macie מוסיפים עלות ויש להפעילם לפי סיכון.

### בדיקת workload לפי ששת ה-pillars
- **Operational Excellence:** policy-as-code, access reviews, key rotation ו-alerts על public access.
- **Security:** least privilege, Block Public Access, SSE-KMS לפי דרישה ו-Object Lock ל-WORM.
- **Reliability:** versioning ו-retention מגנים ממחיקה; מתכננים recovery של KMS key.
- **Performance Efficiency:** לא להפעיל KMS לכל פעולה ללא צורך; endpoint/CloudFront מתאימים.
- **Cost Optimization:** SSE-S3 כשמספיק, lifecycle לגרסאות ו-logging ממוקד.
- **Sustainability:** retention ו-archive מפחיתים עותקים ונתונים מיותרים.
