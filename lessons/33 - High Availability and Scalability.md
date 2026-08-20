# 33 — High Availability & Scalability

## 1. מה זה?
High Availability (HA) ממשיך שירות בעת כשל צפוי; scalability מטפל בגידול capacity; elasticity מוסיפה ומסירה capacity לפי demand; fault tolerance שואפת להמשיך כמעט ללא interruption.

## 2. למה צריך את זה?
זמינות, throughput ו־DR הן דרישות שונות. שני servers באותו AZ נותנים capacity אך לא מגנים מכשל AZ. design טוב מגדיר RTO/RPO, failure domains ו־degradation acceptable.

## 3. איך זה עובד?
פזר stateless compute בין AZs מאחורי ALB/NLB, השתמש ב־Auto Scaling Group עם health checks ו־managed data services כמו RDS Multi-AZ או DynamoDB. Scale out מוסיף instances; scale up מגדיל instance. Sessions עוברים ל־DynamoDB/ElastiCache או משתמשים ב־sticky sessions רק כשאין חלופה. Database replicas ו־queues מפרידים read/write או workload bursts.

## 4. הדברים שחייבים לדעת למבחן
- Multi-AZ מגן מכשל AZ; Multi-Region נדרש לכשל Region.
- ASG + ELB נותנים self-healing ו־scale out, אך לא בהכרח data durability.
- stateless app קל להחליף ולהרחיב.
- scale up פשוט אך מוגבל; scale out דורש stateless/partitioning.
- HA אינה DR, ו־fault tolerance יקרה יותר.
- RTO הוא זמן התאוששות מקסימלי; RPO הוא data loss מקסימלי.

## 5. עלות, תמחור ו־trade-offs
Multi-AZ משלם עבור capacity/standby נוספת, load balancer, cross-AZ data transfer ולעיתים replicas. Scale out עם instances קטנים יכול להיות זול וגמיש מ־scale up של instance גדול, אך מוסיף LB ו־orchestration. Managed serverless/DynamoDB on-demand חוסכים idle capacity אך מחיר ליחידה עלול להיות גבוה בקצב יציב. Multi-Region ו־active-active יקרים יותר בגלל duplicate compute/data replication ו־inter-Region transfer. הימנע מ־overprovisioning: target tracking ו־right-sizing מוזילים.

## 6. ההבדלים החשובים
| מושג | מטרה | trade-off |
|---|---|---|
| HA | שירות זמין בזמן כשל מקומי | capacity ועלות נוספת |
| Scalability | טיפול בעומס גדול יותר | design של state/partitioning |
| Elasticity | התאמה דינמית ל־demand | scaling delay/metrics |
| Fault tolerance | כמעט ללא interruption | duplicate resources ועלות גבוהה |
| DR | התאוששות מאירוע רחב | RTO/RPO וזמן failover |

## 7. Well-Architected view
- **Operational Excellence:** health checks, scaling policies, game days ו־runbooks.
- **Security:** לכל AZ/instance IAM role, patching, segmentation ו־encrypted data replication.
- **Reliability:** multiple AZs, self-healing, graceful degradation ו־dependency timeouts.
- **Performance Efficiency:** scale out, caching, async queues ו־load-test לפני קביעת thresholds.
- **Cost Optimization:** right-size, scheduled scaling, Savings Plans ו־avoid idle standby מעבר ל־RTO.
- **Sustainability:** elasticity, efficient instance types ו־scale-in כשעומס יורד.

## 8. מלכודות במבחן
יותר גדול אינו HA. שני instances באותו AZ אינם AZ redundancy. Read Replica משפר reads ואינו מחליף Multi-AZ failover. ELB לבדו אינו הופך stateful app ל־stateless. Multi-AZ אינו בהכרח הגנה מכשל Region.

## 9. Scenario מהעולם האמיתי
ALB ו־ASG מפזרים EC2 בשני AZs; sessions נשמרים ב־DynamoDB, ו־RDS Multi-AZ מספק failover. Target tracking מוסיף instances לפי request count. כך compute ניתן להחלפה וה־database נשאר זמין.

## 10. מה לא צריך לדעת
לא לשנן SLA percentages. הבן failure domains, scaling triggers, state placement ועלות cross-AZ.

## 11. סיכום
1. HA שונה מ־scalability.
2. פזר failure domains.
3. ASG + ELB נותנים self-healing.
4. Stateless מאפשר scale out.
5. Multi-AZ אינו Multi-Region.
6. RTO/RPO מכתיבים design.
7. elasticity חוסכת idle capacity.

## 12. בדיקת הבנה
1. מה מגן מכשל AZ?
2. מה ההבדל בין scale-up ל־scale-out?
3. מהו RPO?

