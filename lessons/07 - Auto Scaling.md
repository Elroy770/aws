# 07 — Auto Scaling

## 1. מה זה?

EC2 Auto Scaling Group ‏(ASG) שומר capacity רצוי ומחליף instances לא בריאים.

## 2. למה צריך את זה?

כדי להתמודד עם כשל ועומס משתנה בלי להחזיק שרתים מיותרים או לבצע scaling ידני.

## 3. איך זה עובד?

ASG משתמש ב-launch template, מינימום/מקסימום/desired capacity ו-scaling policies. הוא יכול להשתמש ב-EC2 health checks או ELB health checks. Target tracking מכוון metric, למשל average CPU או ALB request count per target.

## 4. הדברים שחייבים לדעת למבחן

- ASG הוא לא load balancer; משלבים אותו עם ALB/NLB.
- Multi-AZ ASG משפר זמינות.
- Launch template מגדיר AMI, type, role ו-user data.
- Health check כושל מוביל להחלפת instance.

## 5. ההבדלים החשובים

| Policy | מתי לבחור |
|---|---|
| Target tracking | metric יציב ורצון לפשטות |
| Step scaling | תגובה מדורגת ל-alarm |
| Scheduled scaling | עומס צפוי בשעה קבועה |

## 6. מלכודות במבחן

CPU נמוך לא בהכרח אומר שאין עומס; עבור web app metric של requests per target עשוי להיות מתאים יותר.

## 7. Scenario מהעולם האמיתי

אתר מכירות מאחורי ALB מפזר instances בשני AZs. ASG עם target tracking מחליף unhealthy instance ומגדיל capacity בשיא.

## 8. מה לא צריך לדעת

לא צריך לשנן cooldown values או syntax של כל policy.

## 9. סיכום

- ASG = elasticity + self-healing.
- השתמש ב-launch template.
- פזר בין AZs.
- metric נכון חשוב מה-scaling policy המורכבת.
- ALB מנתב; ASG מנהל fleet.

## 10. בדיקת הבנה

1. מי מחליף instance לא בריא?
2. האם ASG מנתב HTTP requests?
3. מתי scheduled scaling מתאים?

## 11. הרחבה: עלויות ו-trade-offs

ASG עצמו אינו התחליף לעלות ה-instances: משלמים על EC2, EBS, load balancer, data transfer ו-CloudWatch metrics/logs. מינימום גבוה מדי יקר בשעות שפל; מינימום נמוך מדי פוגע ב-Reliability ובזמן תגובה. Target tracking פשוט, אך metric לא נכון עלול להפעיל instances יקרים ללא שיפור. Scheduled scaling זול יותר תפעולית לעומס צפוי; predictive/dynamic scaling מתאים לדפוסים משתנים אך דורש metrics טובים. שילוב On-Demand baseline עם Spot capacity זול יותר, אך דורש stateless design ויכולת interruption.

## 12. בדיקה לפי ששת ה-Pillars

| Pillar | החלטה ב-ASG |
|---|---|
| Operational Excellence | launch template, lifecycle hooks, alarms ו-automated replacement |
| Security | IAM role, private subnets ו-SG; לא credentials ב-user data |
| Reliability | פיזור AZs, ELB health checks ו-capacity rebalance |
| Performance Efficiency | requests-per-target במקום CPU כשזה מדד עומס טוב יותר |
| Cost Optimization | min/desired/max לפי metrics, scale-in ו-Savings/Spot |
| Sustainability | scale-in בשפל, instances יעילים והימנעות מ-overprovisioning |
