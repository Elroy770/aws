# 40 — Integrated SAA Scenarios

## 1. מה זה?
תרגול שבו network, compute, data, security, HA, observability ועלות פועלים יחד. SAA בודק end-to-end design ולא זיהוי שירות בודד.

## 2. למה צריך את זה?
בחירה טובה בשכבה אחת עלולה ליצור failure או cost בשכבה אחרת: NAT יחיד הוא bottleneck, DB ציבורי הוא security risk, ו-queue ללא DLQ מאבד תפעוליות.

## 3. איך זה עובד?
בכל scenario צייר בראש: User → Route 53/CloudFront → ALB/API → private compute → data → async/monitoring. סמן public/private, AZ/Region, state, credentials, backup, scaling, RTO/RPO ו-data-transfer paths. לאחר מכן בדוק failure של instance, AZ, dependency ו-Region.

## 4. הדברים שחייבים לדעת למבחן
- ALB + ASG ב-private subnets בשתי AZs ל-web tier.
- RDS Multi-AZ ל-HA/failover; read replica ל-read scaling.
- CloudFront + private S3 origin/OAC לתוכן גלובלי.
- SQS סופג spikes; consumers מתרחבים; DLQ ו-idempotency.
- IAM roles במקום static credentials; Secrets Manager לסודות.
- VPC endpoints ל-private AWS access; NAT ל-general Internet egress.
- CloudWatch/CloudTrail, encryption, backups ו-health checks משלימים design.

## 5. עלות, תמחור ו-trade-offs
משלמים על ALB/NLB, NAT gateways, EC2/Fargate, RDS instances/storage/IOPS, cross-AZ ו-cross-Region transfer, CloudFront requests/egress, logs ו-backups. Multi-AZ מוסיף compute/storage אך מונע downtime; NAT בכל AZ יקר יותר אך מונע single-AZ dependency. Gateway Endpoint לרוב זול יותר ל-S3/DynamoDB. CloudFront עשוי להוסיף cost, אך חוסך origin load ו-latency באזורים רבים. SQS/Lambda טובים ל-spikes; workers תמיד-on עשויים להיות זולים יותר בעומס קבוע.

## 6. ההבדלים החשובים
| Requirement | Layer/choice | למה |
|---|---|---|
| global low latency | CloudFront + Route 53 | edge cache ו-routing |
| private AWS access | Gateway/Interface Endpoint | ללא Internet/NAT מיותר |
| queue spike | SQS + consumers | backpressure ו-decoupling |
| relational HA | RDS Multi-AZ | failover |
| read throughput | read replica/cache | scale reads |
| regional DR | backup/replication + DNS failover | recovery מ-Region |

## 7. Well-Architected view
- **Operational Excellence:** IaC, deployment strategy, dashboards, alarms, runbooks ו-game days.
- **Security:** tier isolation, SG least privilege, WAF, IAM roles, KMS ו-audit.
- **Reliability:** Multi-AZ, health checks, retries/DLQ, backups, tested restore ו-DR.
- **Performance Efficiency:** cache, right-size, async, connection pooling ו-load test.
- **Cost Optimization:** lifecycle logs, endpoints, right-size ו-scale-to-demand; בדוק transfer.
- **Sustainability:** cache/reuse, efficient instance sizes, serverless כשהמתאים והימנע מ-idle copies.

## 8. מלכודות במבחן
אל תענה רק על שכבה אחת. NAT single-AZ, DB public, credentials בקוד, no DLQ, או read replica במקום HA יכולים לפסול architecture. Multi-Region אינו נדרש אם הדרישה היא רק AZ failure.

## 9. Scenario מהעולם האמיתי
Global shop: Route 53→CloudFront→ALB→ASG בשתי AZs→RDS Multi-AZ. S3 private עם OAC; orders ל-SQS; IAM roles; CloudWatch/CloudTrail; backups cross-Region לפי RPO. בוחרים cross-Region רק אם RTO/RPO מצדיקים replication ו-transfer cost.

## 10. מה לא צריך לדעת
לא נדרש diagram גרפי מושלם או syntax של כל service; נדרש להסביר flow, failure, permissions ו-trade-offs.

## 11. סיכום
- הסתכל end-to-end.
- הפרד public, compute ו-data.
- פזר failure domains.
- decouple עבודה אסינכרונית.
- תכנן observability, backup ו-DR.
- כל HA/global feature מוסיף cost שיש להצדיק.

## 12. בדיקת הבנה
1. איפה מסתיים public traffic?
2. איזה רכיב סופג order spikes?
3. מתי cross-Region מוצדק?
