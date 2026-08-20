# 03 — IAM Fundamentals

## 1. מה זה?

IAM מנהל authentication ו-authorization באמצעות users, groups, roles ו-policies.

## 2. למה צריך את זה?

כדי לתת לכל אדם או workload רק את ההרשאות הנדרשות, ללא sharing של credentials קבועים.

## 3. איך זה עובד?

Policy JSON מגדיר Effect, Action, Resource ולעיתים Condition. משתמש אנושי מקבל הרשאות דרך group או policy; workload מקבל role ומבקש credentials זמניים. Root user אינו user רגיל ויש להגן עליו ב-MFA ולא להשתמש בו לפעילות יומיומית.

## 4. הדברים שחייבים לדעת למבחן

- העדף roles ו-temporary credentials על access keys.
- IAM policy מגדיר מי רשאי לבצע פעולה על איזה resource.
- Least privilege ו-MFA הם ברירת המחדל.
- IAM הוא שירות גלובלי; policies משפיעות על שירותים regional לפי resource.

## 5. ההבדלים החשובים

| זהות | מתי לבחור | מתי לא |
|---|---|---|
| IAM user | משתמש קבוע חריג/legacy | אפליקציה על AWS |
| IAM role | EC2, Lambda, cross-account | password אנושי קבוע |
| IAM group | ניהול הרשאות משתמשים דומים | הענקת הרשאות ל-service |

## 6. מלכודות במבחן

אם EC2 צריך לקרוא S3, התשובה היא instance role—not access key בקוד או בקובץ.

## 7. Scenario מהעולם האמיתי

Lambda מעבדת upload מ-S3 וכותבת ל-DynamoDB. צור execution role עם `s3:GetObject` על ה-bucket ו-`dynamodb:PutItem` על הטבלה בלבד.

## 8. מה לא צריך לדעת

לא לשנן את כל Action names של כל שירות; להבין את מבנה המדיניות וה-scope חשוב יותר.

## 9. סיכום

- Authenticate ואז authorize.
- Group לא כולל roles.
- Role נותן credentials זמניים.
- הענק least privilege.
- הגן על root עם MFA.

## 10. בדיקת הבנה

1. מה עדיף לאפליקציית EC2: role או access key?
2. מה ההבדל בין user ל-group?
3. מה פירוש least privilege?

## 11. הרחבה: עלויות ו-Well-Architected

IAM ו-STS עצמם בדרך כלל אינם מחליפים חיובי compute, storage או requests שהזהות מפעילה. Temporary credentials ו-roles זולים ובטוחים יותר תפעולית מ-key rotation ידני. בחן את ה-workload בשישה pillars: Operational Excellence (permission sets ו-audit אוטומטי), Security (least privilege, MFA, KMS), Reliability (הרשאות ל-health checks ול-recovery), Performance Efficiency (גישה ישירה לשירות בלי proxy מיותר), Cost Optimization (הרשאות מוגבלות ליצירת משאבים יקרים), Sustainability (הסרת משאבים לא בשימוש). אל תיתן הרשאת Admin רק כדי לחסוך זמן; blast radius ו-incident יקרים יותר.
