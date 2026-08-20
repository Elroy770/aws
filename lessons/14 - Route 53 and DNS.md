# 14 — Route 53 & DNS

## 1. מה זה ולמה צריך?

Route 53 הוא managed DNS, domain registration ו-health checks. הוא ממפה names ל-endpoints, מחלק traffic בין Regions/versions ומסיר יעד כושל בהתאם למדיניות.

## 2. איך זה עובד?

Public hosted zone עונה לאינטרנט; private hosted zone עונה רק בתוך VPCs מקושרים. Alias הוא record AWS-aware ל-ALB, CloudFront, S3 website ועוד, כולל zone apex; CNAME אינו מתאים ל-zone apex. Policies כוללות simple, weighted, latency, failover, geolocation, geoproximity ו-multivalue. TTL קובע caching אצל resolvers, ולכן משפיע על מהירות שינוי אך אינו health check.

## 3. חובה למבחן והשוואה

| Policy | מתי לבחור | עלות/trade-off |
|---|---|---|
| Weighted | blue/green או אחוזי traffic | יותר records/health checks; מאפשר ניסוי מבוקר |
| Latency | Region עם latency נמוך לפי מדידות AWS | אינו מודד כל request בזמן אמת |
| Failover | primary/secondary | דורש health evaluation נכון; recovery מושפע מ-TTL |
| Geolocation | חוק/תוכן לפי location | mapping שגוי דורש default record |
| Multivalue | כמה healthy values | אינו תחליף ל-ALB או full load balancing |

תשלום הוא לפי hosted zones, DNS queries ו-health checks/features לפי pricing plan/Region. Alias ל-AWS resource אינו דורש עלות DNS query מיוחדת במקרים רבים, אך היעד עצמו, health checks ו-CloudFront/ALB עדיין מחויבים. TTL נמוך מגביר query volume אך מקצר failover propagation; TTL גבוה זול יותר ומקטין load אך מאט שינוי.

## 4. מלכודות ו-scenario

Latency routing אינו “ping לכל משתמש” בזמן אמת. Failover לא מתקן יעד שה-health check שלו בודק רק process ולא readiness. API פעיל ב-us-east-1 וב-eu-west-1 משתמש ב-latency records וב-health checks; blue/green משתמש ב-weighted עם משקל קטן לגרסה החדשה.

## 5. AWS Well-Architected — ששת ה-pillars

- **Operational Excellence:** IaC ל-zones/records, query logs, health-check alarms ו-runbook ל-TTL.
- **Security:** private hosted zones, DNSSEC היכן מתאים, least privilege ל-record changes ו-protected registrar.
- **Reliability:** שני Regions/targets, failover health checks, default records ו-appropriate TTL.
- **Performance Efficiency:** latency routing, alias ל-AWS endpoints ו-TTL שמאזן freshness מול cache.
- **Cost Optimization:** הסר health checks/queries מיותרים, TTL סביר ו-policy המתאימה במקום צי proxy נוסף.
- **Sustainability:** caching באמצעות TTL, פחות health probes וניתוב ל-Region קרוב מצמצמים עבודה ו-data transfer.

## 6. סיכום

Route 53 = DNS + health + routing. Alias מתאים ל-AWS ול-zone apex; policy נבחרת לפי latency, failover, weights או location, ו-TTL הוא trade-off תפעולי ועלותי.
