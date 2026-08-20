# 37 — Cost Optimization

## 1. מה זה?
Cost Optimization הוא אספקת business value במחיר הנמוך המתאים, בלי להפר את security, availability או performance. SAA מחפש Total Cost of Ownership: resource, data transfer, requests, licenses, operations ו-downtime.

## 2. למה צריך את זה?
Cloud מאפשר elasticity אך גם יוצר idle EC2/EBS, snapshots, NAT ו-egress שלא היו גלויים. “Cheapest” אינו בהכרח “most cost-effective” אם הוא דורש ניהול רב או פוגע ב-RTO.

## 3. איך זה עובד?
אסוף בעלות באמצעות tags/accounts ו-Cost and Usage Report; נתח מגמות ב-Cost Explorer; הגדר AWS Budgets להתראות; השתמש Compute Optimizer ו-CloudWatch ל-right-sizing. Auto Scaling מתאים supply ל-demand. Savings Plans/Reserved Instances מתאימים baseline יציב; Spot ל-stateless ו-interruptible jobs. Stop schedules מונעים תשלום על non-production idle.

## 4. הדברים שחייבים לדעת למבחן
- On-Demand = גמישות מלאה; Reserved/Savings Plans = commitment תמורת מחיר נמוך יותר.
- Spot הוא הזול ביותר ל-capacity גמיש אך ניתן interrupt.
- S3 Lifecycle מעביר objects לפי access pattern; Glacier זול באחסון אך retrieval ו-minimum duration עולים.
- NAT Gateway מחויב לפי שעה ונתונים; ל-S3/DynamoDB ב-VPC Gateway Endpoint לרוב זול יותר.
- Multi-AZ, cross-Region replication ו-egress משפרים resilience אך מגדילים cost.
- כבה nonprod; אל תכבה production HA כדי לחסוך.

## 5. עלות, תמחור ו-trade-offs
EC2 On-Demand יקר ל-baseline אך ללא התחייבות; Savings Plans גמישים יותר מ-RI מבחינת סוג שימוש אך מחייבים spend; RI מתאים ל-DB/instance pattern יציב; Spot זול מאוד אך דורש retry ו-capacity diversification. Lambda מחויבת requests ו-duration, ולכן יעילה ל-bursty workloads אך לא תמיד ל-compute רציף. Fargate חוסך ניהול שרת אך יכול לעלות יותר מ-EC2 מנוצל היטב.

ב-S3 משלמים storage class, requests, retrieval ו-data transfer. Standard יקר יותר לאחסון אך מתאים ל-hot data; Standard-IA/One Zone-IA זולים יותר לאחסון עם retrieval fees (One Zone פחות עמיד); Glacier classes זולות יותר לארכיון אך איטיות/יקרות לשליפה. EBS מחויב לפי volume type/size ו-snapshots; EFS לפי data stored/accessed, וגמיש אך לעיתים יקר יותר מ-EBS.

## 6. ההבדלים החשובים
| כלי/מודל | לבחור כש... | trade-off |
|---|---|---|
| Cost Explorer | מנתחים היסטוריה ומגמות | אינו prevention |
| Budgets | רוצים threshold/alert | אינו מכבה resource לבדו |
| Savings Plan | baseline compute ידוע וגמישות | commitment |
| Reserved Instance | pattern קבוע, גם RDS | פחות גמיש |
| Spot | batch/stateless וניתן retry | interruption |
| Gateway Endpoint | S3/DynamoDB private | מוגבל לשירותים אלה |

## 7. Well-Architected view
- **Operational Excellence:** tagging, ownership, budgets, anomaly alerts ו-review מחזורי.
- **Security:** least privilege ו-encryption; לא מחליפים security controls ב-public cheap path.
- **Reliability:** baseline on-demand/commitment, multi-AZ ו-capacity buffer; Spot רק עם graceful recovery.
- **Performance Efficiency:** right-size לפי metrics, caching ו-Auto Scaling במקום overprovisioning.
- **Cost Optimization:** eliminate idle, choose pricing model לפי variability, lifecycle ו-architecture פשוטה.
- **Sustainability:** פחות idle compute/storage, autoscaling ו-efficient managed services מפחיתים energy footprint.

## 8. מלכודות במבחן
“Most cost-effective” כולל operations; Spot אינו מתאים ל-stateful primary DB; Glacier אינו זול אם שולפים כל שעה; NAT אינו תמיד נדרש ל-AWS service access; commitment לא מתאים ל-workload לא צפוי.

## 9. Scenario מהעולם האמיתי
Web fleet יציב משתמשת Savings Plan לבסיס, ASG מוסיף On-Demand/Spot ל-bursts, ו-S3 logs עוברים Standard→IA→Glacier לפי age. S3 Gateway Endpoint מונע NAT data-processing עבור private log processors. כך נשמרת HA בלי לשלם peak capacity תמיד.

## 10. מה לא צריך לדעת
אין צורך לשנן מחירים, אחוזי discount או נוסחאות billing מדויקות; נדרש לזהות מה מחויב ומהו trade-off.

## 11. סיכום
- Measure לפני right-size.
- commitment רק ל-baseline.
- Spot לגמיש וניתן interruption.
- Lifecycle ל-cold data.
- בדוק transfer/NAT/requests.
- עלות כוללת כוללת operations ו-resilience.

## 12. בדיקת הבנה
1. איזה כלי מנתח spend היסטורי?
2. מתי Spot מתאים?
3. למה Glacier לא תמיד זול בפועל?
