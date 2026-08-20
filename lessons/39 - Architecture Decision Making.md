# 39 — Architecture Decision Making

## 1. מה זה?
שיטה להפוך scenario לפתרון AWS שהוא BEST, לא רק technically possible. מתחילים ב-requirements ובמגבלות ורק אחר כך מתאימים services.

## 2. למה צריך את זה?
ב-SAA כמה answers עובדות חלקית. מילים כמו **most secure**, **least operational overhead**, **lowest cost**, **highly available**, **private**, **global** ו-**minimal downtime** קובעות את הבחירה.

## 3. איך זה עובד?
1. חלץ users, data flow, state ו-critical path.
2. כתוב must-have: latency, throughput, RTO/RPO, AZ/Region, security ו-budget.
3. בנה layers: edge/DNS→ingress→compute→data→async/observability.
4. פסול תשובה שמפרה constraint; השווה remaining answers לפי operations, failure mode ועלות.
5. ודא permissions, encryption, backup, scaling ו-data transfer.

## 4. הדברים שחייבים לדעת למבחן
- “least operational overhead” → managed/serverless כשעומד בדרישות.
- survive AZ failure → Multi-AZ; read scaling → read replica/cache.
- durable async → SQS; fan-out → SNS; routing לפי event → EventBridge.
- private S3/DynamoDB access → Gateway Endpoint, לא NAT.
- global low latency → CloudFront/Route 53; strict SQL → RDS/Aurora.
- אל תבחר cheapest component אם הוא פוגע ב-RTO/security.

## 5. עלות, תמחור ו-trade-offs
Managed service עשוי לעלות יותר ליחידת compute אך לחסוך patching, on-call ו-capacity planning; compare TCO. NAT מחויב hourly + data processing, בעוד Gateway Endpoint בדרך כלל חוסך processing עבור S3/DynamoDB. Multi-AZ, replicas ו-cross-Region עולים storage/compute/transfer אך מקטינים downtime. Serverless זול ב-idle/bursty; EC2/containers יכולים להיות זולים יותר בעומס קבוע. תמיד חשב requests, egress, inter-AZ traffic ו-operations.

## 6. ההבדלים החשובים
| Requirement | דפוס מתאים | בדיקת trade-off |
|---|---|---|
| read scaling | replica/cache | stale data ו-cache invalidation |
| durable async | SQS | visibility timeout/DLQ |
| fan-out | SNS | subscribers ו-retries |
| private AWS access | VPC Endpoint | endpoint type/cost |
| global delivery | CloudFront | cacheability ו-egress |
| relational transactions | RDS/Aurora | scaling/operations |

## 7. Well-Architected view
- **Operational Excellence:** ADRs, diagrams, ownership, automation ו-review של assumptions.
- **Security:** threat model, least privilege, private paths, encryption ו-audit logs.
- **Reliability:** failure domains, retries, backups, health checks ו-dependency isolation.
- **Performance Efficiency:** מדוד latency/throughput, בחר נכון בין cache/replica/compute.
- **Cost Optimization:** compare TCO, utilization, transfer ו-managed operations.
- **Sustainability:** בחר architecture יעילה, scale-to-demand והימנע מ-overprovisioning.

## 8. מלכודות במבחן
אל תבחר service נוצץ שלא פותר constraint. “Most cost-effective” אינו “cheapest”; “private” אינו בהכרח “no encryption”; Multi-AZ אינו DR מ-Region שנמחק; read replica אינה failover אוטומטי כמו Multi-AZ.

## 9. Scenario מהעולם האמיתי
Private EC2 צריך לקרוא S3 ללא Internet ובעלות נמוכה: S3 Gateway Endpoint. NAT היה עובד, אך מוסיף hourly/data-processing ו-public egress path מיותר. אם היעד היה arbitrary SaaS endpoint, NAT היה עדיין נדרש.

## 10. מה לא צריך לדעת
אין צורך לשנן service catalog מלא או כל parameter. למד לזהות requirement→constraint→best service.

## 11. סיכום
- קרא את ה-requirement לפני ה-options.
- פסול מפרי constraint.
- בדוק failure, security, cost ו-operations.
- Managed לא תמיד cheapest אך לעיתים BEST.
- endpoint, Multi-AZ ו-async נבחרים לפי flow.

## 12. בדיקת הבנה
1. מה רומז על SQS?
2. מה ההבדל בין Multi-AZ ל-read replica?
3. למה NAT אינו BEST ל-S3 פרטי?
