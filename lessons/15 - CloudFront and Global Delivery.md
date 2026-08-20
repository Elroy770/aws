# 15 — CloudFront & Global Delivery

## 1. מה זה?
CloudFront הוא CDN שמגיש תוכן מ-edge locations ומפחית latency ועומס על origin.

## 2. למה צריך את זה?
הוא מתאים לתוכן סטטי, downloads ו-dynamic HTTP עם קהל גלובלי, אבטחת edge ו-caching.

## 3. איך זה עובד?
Distribution מקשר origins כגון S3, ALB או API Gateway. Behaviors קובעים paths, cache policy ו-origin. CloudFront cache key מגדיר מה מבדיל response (headers/cookies/query strings). Origin Access Control ‏(OAC) מאפשר ל-S3 לקבל קריאות רק מ-CloudFront.

## 4. הדברים שחייבים לדעת למבחן
- CloudFront + S3: תוכן מהיר ו-bucket פרטי עם OAC.
- CloudFront אינו מחליף Multi-AZ ל-origin.
- WAF יכול להיות מחובר ל-CloudFront.
- invalidation מסיר cached objects אך design TTL נכון עדיף.

## 5. ההבדלים החשובים
| שירות | BEST עבור |
|---|---|
| CloudFront | HTTP content/cache גלובלי |
| Global Accelerator | TCP/UDP, static anycast IP, ללא cache |
| S3 Transfer Acceleration | uploads ל-S3 מרחוק |

## 6. מלכודות במבחן
Global Accelerator לא שומר cache של תמונות. CloudFront אינו פותר upload ישירות לאפליקציה אם הדרישה היא acceleration של S3 בלבד.

## 7. Scenario מהעולם האמיתי
אתר מדיה משתמש ב-S3 private origin, OAC ו-CloudFront. המשתמשים מקבלים cache מה-edge וה-bucket אינו פתוח לציבור.

## 8. מה לא צריך לדעת
אין צורך לשנן כל CloudFront header או edge location.

## 9. סיכום
- CDN מפחית latency.
- origins יכולים להיות S3/ALB/API.
- OAC סוגר את S3 לציבור.
- cache behavior חשוב.
- WAF ב-edge נותן הגנה מוקדמת.

## 10. בדיקת הבנה
1. מה מגן על S3 origin פרטי?
2. מה ההבדל מ-Global Accelerator?
3. האם CloudFront מחליף origin HA?

## העמקה: תכנון, עלויות וששת ה-Well-Architected pillars
CloudFront בוחר את ה-behavior הספציפי ביותר לפי path pattern. ה-cache policy קובעת מה נכנס ל-cache key ומה נשלח ל-origin; forwarding של cookies, headers או query strings רבים מדי מקטין cache hit ratio. ל-API דינמי מגדירים לרוב `CachingDisabled`, ומעבירים רק את המידע הדרוש. TTL הוא איזון בין freshness לבין origin requests; versioned keys כגון `app.v42.js` עדיפים על invalidation תכוף. ל-CloudFront certificate של ACM יש להשתמש באזור `us-east-1`. OAC עדיף ל-S3 origin חדש, ו-signed URLs/cookies מגנים על תוכן בתשלום.

### עלויות ו-trade-offs
משלמים על data transfer out ל-viewers, HTTP/HTTPS requests, invalidations מעבר למכסה, ו-Functions/Lambda@Edge. Cache hit ratio גבוה מוזיל origin compute ו-egress, אך cache-busting תכוף ו-cache key מפורט מעלים עלויות. Price Class מצמצם edge locations (זול יותר, אך עלול להוסיף latency). S3 Transfer Acceleration מיועד ל-upload אל S3; CloudFront מיועד בעיקר ל-delivery/download.

### בדיקת workload לפי ששת ה-pillars
- **Operational Excellence:** ניהול behaviors כ-code, dashboards/alarms על `5xxErrorRate`, latency ו-cache hit ratio, ו-runbook ל-invalidation.
- **Security:** HTTPS, WAF, OAC, bucket פרטי ו-signed URLs לפי הרשאה.
- **Reliability:** origin ב-Multi-AZ ו-failover לפי צורך; CloudFront לבדו אינו HA ל-origin.
- **Performance Efficiency:** TTL, compression ו-cache key מצמצמים latency ו-origin load.
- **Cost Optimization:** שיפור hit ratio, TTL מתאים ו-Price Class; לא להפעיל Lambda@Edge כש-policy פשוטה מספיקה.
- **Sustainability:** caching, compression והפחתת origin requests מצמצמים compute ו-data transfer.
