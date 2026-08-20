# 31 — Monitoring & Logging

## 1. מה זה?
Observability משלבת metrics, logs ו־traces. CloudWatch מודד ומתריע, CloudTrail מתעד AWS API activity, ו־AWS Config מתעד configuration/compliance לאורך זמן.

## 2. למה צריך את זה?
Metrics מראות מגמות ומפעילות scaling; logs מסבירים מה קרה בתוך האפליקציה; audit trail עונה מי ביצע שינוי; Config מזהה drift. אין כלי יחיד שמחליף את כולם.

## 3. איך זה עובד?
CloudWatch collects native/custom metrics, Logs ו־alarms. Alarm יכול לשלוח SNS, להפעיל Auto Scaling או פעולה אחרת; Logs Insights מאפשר query. CloudTrail מתעד management events כברירת מחדל ברמת account, ו־data events (למשל S3 object access) נבחרים לפי צורך; Trail יכול לכתוב ל־S3 ול־CloudWatch Logs. Config rules בודקים resources ומציגים history/compliance. CloudWatch Agent מוסיף OS metrics/logs מ־EC2.

## 4. הדברים שחייבים לדעת למבחן
- performance metric/alarm → CloudWatch.
- “who created/deleted/changed?” → CloudTrail.
- configuration drift/compliance → AWS Config.
- application/system logs → CloudWatch Logs.
- CloudTrail אינו app log; metric אינו זהות caller.
- לוגים ב־S3 דורשים retention, encryption והרשאות מתאימות.

## 5. עלות, תמחור ו־trade-offs
CloudWatch מחויב לפי custom metrics, metric API requests, log ingestion/storage ו־alarms; high-cardinality metrics ו־verbose logs יקרים. CloudTrail management events עשויים להיכלל ברמה בסיסית, אך trails נוספים, data events, Insights ו־delivery יכולים לעלות. Config מחויב לפי configuration items ו־rule evaluations. S3 archival זול יותר מ־CloudWatch Logs אך query פחות מיידי; retention קצר ו־subscription filters חוסכים כסף. אל תפעיל data events על כל bucket בלי צורך.

## 6. ההבדלים החשובים
| שירות | שאלה שהוא עונה עליה | שימוש |
|---|---|---|
| CloudWatch | איך המערכת מתנהגת? | metrics, logs, alarms |
| CloudTrail | מי עשה איזה API call? | audit/security investigation |
| AWS Config | האם resource compliant ומה השתנה? | drift/governance |
| X-Ray (אם נדרש) | היכן latency בין services? | distributed tracing |

## 7. Well-Architected view
- **Operational Excellence:** dashboards, actionable alarms, structured logs ו־runbooks.
- **Security:** CloudTrail ארגוני בלתי־ניתן לשינוי ב־S3, KMS, least privilege ו־alerts על API חריגים.
- **Reliability:** alarms על errors/latency, health signals ו־automated remediation שלא גורמת ל־flapping.
- **Performance Efficiency:** custom metrics רלוונטיים, tracing ו־Logs Insights לאבחון bottlenecks.
- **Cost Optimization:** log levels ו־retention לפי ערך, metric filters ו־S3 tiering.
- **Sustainability:** דגימה, batching ו־retention קצר מפחיתים ingestion/storage ו־compute.

## 8. מלכודות במבחן
CloudTrail אינו מנטר CPU או 5XX. Config אינו real-time application monitoring. Alarm על metric חסר datapoints אינו בהכרח recovery. CloudTrail data events אינם תמיד מופעלים אוטומטית. שמירת הכל ב־CloudWatch Logs לנצח היא יקרה ולא בהכרח best answer.

## 9. Scenario מהעולם האמיתי
ALB 5XX ו־latency נשלחים ל־CloudWatch Alarm שמודיע SNS ומפעיל response. CloudTrail organization trail כותב ל־S3; חקירה מגלה מי שינה Security Group. Config rule מזהה SG פתוח ל־world ומסמן noncompliant.

## 10. מה לא צריך לדעת
אין צורך לשנן כל namespace, API או pricing number. חשוב לזהות מקור signal, retention ותגובה אוטומטית.

## 11. סיכום
1. Metrics/alarms/logs = CloudWatch.
2. API audit = CloudTrail.
3. Drift/compliance = Config.
4. Logs אינם metrics.
5. Data events הם בחירה בעלות נוספת.
6. Retention ו־S3 tiering משפיעים על עלות.
7. Alarm צריך action ברור.

## 12. בדיקת הבנה
1. מי תיעד `DeleteBucket`?
2. מה מפעיל scaling לפי CPU?
3. איזה שירות מזהה SG לא־compliant?

