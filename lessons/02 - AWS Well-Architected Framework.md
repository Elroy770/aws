# 02 — AWS Well-Architected Framework

## 1. מה זה?

ה-AWS Well-Architected Framework בוחן workload מול שישה pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization ו-Sustainability. זהו כלי חשיבה ושאלות design, לא שירות שמתקן ארכיטקטורה אוטומטית.

## 2. למה צריך את זה?

ה-Framework מתרגם דרישות כמו "מאובטח", "זמין" או "זול" להחלטות שניתנות למדידה. ב-SAA מחפשים את התשובה שמתאימה לדרישה ול-trade-off. Review חוזר חשוב כי workload, traffic ומחירים משתנים.

## 3. איך זה עובד?

מגדירים business requirements, RTO/RPO, latency, compliance ו-budget; ממפים אותם ל-pillars; מזהים סיכונים; בוחרים controls; ומודדים לאחר deployment. Well-Architected Tool עוזר לתעד high-risk items. Multi-AZ מעלה Reliability אך גם compute/storage cost; Auto Scaling ו-serverless עשויים לשפר Cost ו-Sustainability, אך יש לבחון cold starts ו-operational fit.

## 4. עלויות ו-trade-offs

משלמים על replicas, Multi-AZ, logs, backups, cross-Region replication ו-data transfer. בדרך כלל זול יותר single Region ו-on-demand; אמין יותר אך יקר יותר להפעיל standby/replicas ב-Region נוסף. Cost Optimization אינו בחירת השירות הזול בלבד: downtime, over-operations ו-energy waste הם גם עלות. Managed services עשויים להיות יקרים ליחידה אך זולים יותר תפעולית.

## 5. ששת ה-Pillars

| Pillar | מה בודקים | דוגמאות SAA |
|---|---|---|
| Operational Excellence | שינוי, תצפית, runbooks ולמידה | IaC, CloudWatch, automation |
| Security | זהויות, detection, protection ו-data security | IAM roles, MFA, KMS, SG/WAF |
| Reliability | recovery, failure isolation ו-demand changes | Multi-AZ, backups, health checks |
| Performance Efficiency | משאבים וטכנולוגיה מתאימים | right-sizing, caching, serverless |
| Cost Optimization | ערך עסקי מול spend | budgets, lifecycle, Savings Plans |
| Sustainability | תוצאה עסקית עם פחות משאבים/אנרגיה | scaling, efficient instances, cache |

## 6. הדברים שחייבים לדעת למבחן

- Reliability היא לא Performance: recovery וזמינות שונות מ-latency/throughput.
- Security ו-Cost הם constraints; אין לבטל least privilege כדי לחסוך.
- מדידה ו-automation משרתות כמה pillars בו-זמנית.
- Most cost-effective אינו בהכרח lowest hourly price אם נדרשת זמינות.

## 7. מלכודות במבחן

Most reliable אינו בהכרח lowest cost. אם השאלה אומרת workload interruptible, Spot עשוי לנצח; אם היא דורשת zero interruption, לא. אל תבחר Multi-Region רק כי הוא נשמע הכי זמין—בדוק RTO/RPO ו-data consistency.

## 8. Scenario מהעולם האמיתי

API עם עומס משתנה משתמש ב-ALB, ASG, CloudWatch ו-managed database. Target tracking משפר Performance ו-Cost, Multi-AZ משפר Reliability, IAM role ו-KMS משפרים Security, deployment אוטומטי משפר Operational Excellence, ו-lifecycle/log retention מצמצמים Cost ו-Sustainability impact.

## 9. סיכום ובדיקת הבנה

התחל בדרישות, זהה trade-offs, מדוד ושפר ברציפות.

1. איזה pillar מטפל ביכולת recovery?
2. אילו חיובים נוספים עלולים להופיע ב-Multi-Region?
3. כיצד scaling תורם ל-Sustainability?

