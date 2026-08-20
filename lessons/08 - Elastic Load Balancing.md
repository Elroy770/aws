# 08 — Elastic Load Balancing (ELB)

## 1. מה זה?

ELB הוא managed service שמציג endpoint אחד ומפזר בקשות אל targets בריאים: EC2, containers, IP addresses או (ב-ALB) Lambda. הוא מסיר targets לא בריאים לפי health checks ומאפשר לפרוס targets בכמה Availability Zones.

## 2. למה צריך את זה?

במקום שלקוחות יכירו כתובות שרתים, ה-load balancer מספק נקודת כניסה יציבה, מפזר עומס, מסיים TLS ומאפשר scaling בלי שינוי DNS בכל deployment. הוא גם מפריד בין ingress לבין application tier.

## 3. איך זה עובד?

Load balancer מאזין ב-listener (למשל HTTPS/443), בודק targets ב-target groups ומנתב רק ל-targets שעברו health check. ALB עובד ב-Layer 7 ויכול לנתב לפי host, path, header או query string; NLB עובד ב-Layer 4 עבור TCP/UDP/TLS ומספק ביצועים גבוהים ו-static IP; GWLB מכניס appliances כמו firewall או IDS במסלול בצורה שקופה. TLS termination ב-ALB/NLB מפחית עומס הצפנה מהשרתים; אפשר גם re-encrypt ל-backend.

## 4. הדברים שחייבים לדעת למבחן

- HTTP/HTTPS עם path/host routing או WAF integration → ALB.
- TCP/UDP/TLS, static IP לכל AZ, source IP preservation או throughput גבוה → NLB.
- פריסת security/network virtual appliances → GWLB.
- Targets צריכים להיות רשומים ב-target group; health check מוציא target לא בריא אך לא מתקן אותו.
- לפריסה עמידה בחר לפחות שתי AZs, עם targets בכל AZ.
- ALB מתאים ל-Lambda targets ול-WebSocket; NLB מתאים ל-TLS pass-through ול-long-lived connections.
- deregistration delay/connection draining מונע ניתוק בקשות בזמן deployment.

## 5. על מה משלמים ומה זול יותר?

התמחור משתנה לפי Region ובדרך כלל כולל שעות/Capacity Units (LCU ל-ALB לפי new/active connections, bytes ו-rule evaluations; NLCU ל-NLB) וכן data transfer. ב-GWLB משלמים גם על traffic processing ועל ה-appliance עצמו. אין charge נפרד עבור health-check request, אבל המשאבים והתעבורה שמאחוריו כן עשויים לעלות.

| בחירה | trade-off |
|---|---|
| ALB | routing עשיר, אך יותר LCU כשיש הרבה bytes, connections או rules. |
| NLB | יעיל ל-L4/throughput, אך static IP/TLS ונתוני traffic עדיין מחויבים. |
| GWLB | מוסיף ELB processing ורישוי/EC2 של appliance; מוצדק כשנדרשת inspection. |
| load balancer אחד מול כמה | אחד חוסך hourly cost אך מגדיל blast radius; כמה מבודדים אך יקרים יותר. |

חסוך באמצעות right-sizing, הסרת load balancers idle ו-private paths במקום NAT ל-internal traffic. אל תבחר NLB רק כי הוא “מהיר” בלי לחשב LCU ודרישות L7.

## 6. ההבדלים החשובים

| Service | מתי לבחור | מתי לא |
|---|---|---|
| ALB | Web, microservices, HTTP routing, WAF | raw UDP/TCP או static IP מובנה |
| NLB | L4, static IP, high throughput, source IP | path/content-based routing |
| GWLB | firewall/IDS/inspection fleet | load balancing רגיל של web servers |
| Route 53 | DNS-level distribution/failover | אינו per-request load balancer |

## 7. מלכודות במבחן

- “הכי מהיר” אינו בהכרח NLB: אם נדרשת `/api` מול `/images`, ALB הוא הפתרון.
- SG של ALB אינו מאפשר אוטומטית traffic ל-EC2; SG של EC2 צריך לאפשר מקור שהוא SG של ALB.
- הוספת AZ אינה מספיקה אם אין targets או routes תקינים באותה AZ.
- DNS round robin אינו health-aware כמו ELB.

## 8. Scenario מהעולם האמיתי

חנות online מפרידה `/catalog`, `/checkout` ו-`/images` לשלושה target groups. ALB מקבל HTTPS, עושה redirect מ-HTTP ומנתב לפי path. EC2 נמצאים private subnets, ו-SG שלהם מאפשר 8080 רק מ-SG של ALB. שירות payment ב-TCP עם static IP משתמש ב-NLB.

## 9. AWS Well-Architected — ששת ה-pillars

- **Operational Excellence:** metrics/alarms ל-healthy host count ו-5xx, access logs, automated target registration ו-blue/green deployment.
- **Security:** TLS עם ACM, WAF לפני ALB, SG-to-SG rules, מינימום ports ו-logs מוגנים.
- **Reliability:** שתי AZs לפחות, health checks עמוקים, draining ו-capacity מספקת; load balancer אינו תחליף ל-redundant targets.
- **Performance Efficiency:** בחר L7/L4 לפי protocol, כוונן keep-alive ו-target capacity, והסר TLS bottleneck באמצעות termination.
- **Cost Optimization:** השווה LCU/NLCU, data processing ומספר load balancers; internal traffic לא חייב לעבור NAT.
- **Sustainability:** autoscaling של targets, ביטול idle load balancers וניתוב יעיל מצמצמים compute ו-data processing.

## 10. סיכום ובדיקת הבנה

ALB = L7, NLB = L4/static IP, GWLB = appliances; כולם דורשים target groups ו-health checks.

1. איזה load balancer מתאים ל-path routing?
2. איזה רכיב מתאים ל-TCP עם static IP?
3. אילו עלויות צריך לבדוק מעבר ל-hourly charge?
