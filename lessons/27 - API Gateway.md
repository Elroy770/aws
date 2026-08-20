# 27 — API Gateway

## 1. מה זה?
API Gateway הוא שירות מנוהל ליצירת APIs עם authentication, throttling, routing ו-integrations.
## 2. למה צריך את זה?
הוא מתאים ל-serverless APIs ולחשיפת backend בצורה מבוקרת ללא ניהול servers.
## 3. איך זה עובד?
REST APIs ו-HTTP APIs מקבלים requests ומעבירים ל-Lambda, HTTP backends או AWS services. ניתן להגדיר authorizers, Cognito, API keys/usage plans, caching ו-throttling. Private API נגיש דרך interface VPC endpoint.
## 4. הדברים שחייבים לדעת למבחן
- Lambda backend/API serverless → API Gateway.
- throttling מגן על backend.
- Cognito/authorizer לאימות משתמשים.
- API Gateway אינו load balancer ל-raw TCP.
## 5. ההבדלים החשובים
| שירות | BEST עבור |
|---|---|
| API Gateway | managed API/auth/throttle |
| ALB | HTTP routing ל-EC2/ECS |
| NLB | L4 TCP/UDP |
## 6. מלכודות במבחן
ALB יכול לנתב HTTP אך אינו מחליף API management כאשר נדרשים usage plans או API keys.
## 7. Scenario מהעולם האמיתי
mobile app קוראת Lambda דרך HTTP API עם JWT authorizer ו-throttling; אין EC2 לניהול.
## 8. מה לא צריך לדעת
לא לשנן כל integration type.
## 9. סיכום
- API management managed.
- מתאים במיוחד ל-Lambda.
- auth + throttling בכניסה.
- private API משתמש ב-endpoint.
- ALB אינו API product מלא.
## 10. בדיקת הבנה
1. מה מגן על Lambda מעומס?
2. מה בוחרים ל-JWT API?
3. מה מתאים ל-TCP?
+

## 5. עלות, תמחור ו-trade-offs
החיוב כולל API calls, data transfer, caching ו-backend (Lambda/EC2); REST API בדרך כלל עשיר ויקר יותר מ-HTTP API, בעוד HTTP API זול ופשוט יותר. Custom domain, logs ו-WAF עשויים להוסיף עלות. API Gateway חוסך ניהול API אך ALB יכול להיות זול יותר ל-routing רציף ל-ECS/EC2.

## 6. Well-Architected view
- **Operational Excellence:** stages, access logs, metrics, canary/deployment ו-throttling alarms.
- **Security:** IAM/Cognito/JWT authorizer, WAF, TLS ו-private API דרך VPC endpoint.
- **Reliability:** throttling, retries/timeouts, multi-AZ integrations ו-backend health.
- **Performance Efficiency:** HTTP API כשמספיק, caching ו-payload יעיל.
- **Cost Optimization:** לבחור HTTP API/ללא cache כשלא נחוץ; לשלוט ב-logs וב-data transfer.
- **Sustainability:** serverless scale-to-zero ו-caching מפחיתים compute חוזר.

## 7. מלכודות ו-Scenario
API key/usage plan אינו authentication מלא; JWT/Cognito נועדו לזה. ל-mobile API עם Lambda, JWT authorizer ו-throttling, API Gateway מתאים יותר מ-ALB.
