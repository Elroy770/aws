# 25 — Lambda

## 1. מה זה?
AWS Lambda מריץ קוד בתגובה לאירוע בלי ניהול servers.
## 2. למה צריך את זה?
הוא מפחית operations ומתאים ל-event-driven, APIs קצרים ואוטומציה עם scaling אוטומטי.
## 3. איך זה עובד?
Lambda מקבלת event, משתמשת execution role ומדווחת ל-CloudWatch Logs. Concurrency קובע parallel invocations; reserved concurrency מגן על capacity/תלות. VPC attachment נותן גישה למשאבים פרטיים; היא לא נדרשת לגישה לשירותי AWS ציבוריים בדרך כלל.
## 4. הדברים שחייבים לדעת למבחן
- Lambda stateless; שמור state ב-DynamoDB/S3/RDS.
- execution role, לא credentials בקוד.
- async events עשויים retry; כתוב idempotent.
- VPC Lambda צריכה NAT או endpoint לגישה חיצונית לפי צורך.
## 5. ההבדלים החשובים
| Compute | BEST עבור |
|---|---|
| Lambda | event, short run, no servers |
| Fargate | container/longer process |
| EC2 | OS control/custom host |
## 6. מלכודות במבחן
Lambda ב-private subnet אינה מקבלת internet access רק מעצם החיבור ל-VPC.
## 7. Scenario מהעולם האמיתי
S3 upload מפעיל Lambda ליצירת thumbnail; שמור את התוצאה ב-bucket אחר והפוך את הפעולה idempotent.
## 8. מה לא צריך לדעת
לא לשנן memory-to-vCPU mapping.
## 9. סיכום
- serverless event compute.
- role נותן permissions.
- concurrency היא design concern.
- state מחוץ לפונקציה.
- VPC רק כשצריך private access.
## 10. בדיקת הבנה
1. היכן נשמר state durable?
2. מה מאפשר Lambda לקרוא DynamoDB?
3. האם VPC attachment נותן internet?
+

## 5. עלות, תמחור ו-trade-offs
משלמים לפי requests ו-duration (memory/CPU), ולעיתים provisioned concurrency; יש גם logs, NAT, API Gateway ו-transfer. On-demand זולה ל-bursty/idle, provisioned יקרה יותר אך מצמצמת cold starts. NAT עלול לעלות יותר מהפונקציה; VPC endpoint עשוי להיות יעיל יותר לפי traffic.

## 6. Well-Architected view
- **Operational Excellence:** structured logs, metrics, tracing, versions ו-rollback.
- **Security:** least-privilege execution role, Secrets Manager ו-VPC רק כשנדרש.
- **Reliability:** retries/backoff, DLQ/destinations, idempotency ו-reserved concurrency.
- **Performance Efficiency:** memory tuning, batching ו-provisioned concurrency ל-latency קריטי.
- **Cost Optimization:** למדוד duration, right-size memory ולנקות logs/NAT מיותר.
- **Sustainability:** pay-per-use ו-scale-to-zero מצמצמים idle compute.

## 7. מלכודות ו-Scenario
Private Lambda אינה מקבלת internet מעצם ה-VPC; S3 trigger שכותב לאותו prefix עלול ליצור loop. S3 upload יכול להפעיל thumbnail Lambda ששומרת ב-prefix אחר, עם role מינימלי ו-DLQ.
