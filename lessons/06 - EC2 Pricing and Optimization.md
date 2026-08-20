# 06 — EC2 Pricing & Optimization

## 1. מה זה?

מודלי תמחור EC2 והתאמת capacity ליציבות ולעלות של workload.

## 2. למה צריך את זה?

אותו compute יכול לעלות מאוד אחרת לפי גמישות, משך התחייבות וסבילות להפסקה.

## 3. איך זה עובד?

On-Demand הוא ללא התחייבות. Savings Plans נותנים הנחה בתמורה להתחייבות שימוש; Reserved Instances מספקים הנחה עבור תצורות/התחייבות מסוימות. Spot משתמש capacity פנוי ויכול להיפסק בהתראה קצרה. Dedicated Hosts מיועדים בעיקר לדרישות רישוי או compliance ייחודיות.

## 4. הדברים שחייבים לדעת למבחן

- Spot: fault-tolerant, flexible, interruptible workloads.
- Savings Plans: baseline צפוי וגמישות רחבה יותר מ-RI במקרים רבים.
- On-Demand: קצר, לא צפוי, או בלי commitment.
- Right-size לפי metrics, לא לפי ניחוש.

## 5. ההבדלים החשובים

| מודל | BEST עבור | סיכון/חיסרון |
|---|---|---|
| On-Demand | עומס לא צפוי | מחיר גבוה יותר |
| Savings Plan | שימוש יציב | התחייבות |
| Spot | batch/stateless | interruption |
| Dedicated Host | BYOL/licensing | עלות גבוהה |

## 6. מלכודות במבחן

Spot אינו הפתרון היחיד ל"הכי זול" אם workload חייב רציפות. בחר mixed instances/ASG לשילוב resilience ועלות.

## 7. Scenario מהעולם האמיתי

עיבוד nightly של תמונות ניתן ל-retry: AWS Batch או ASG עם Spot מתאים. Web tier קריטי משתמש ב-On-Demand baseline וב-Spot כתוספת כאשר ניתן.

## 8. מה לא צריך לדעת

אין צורך לשנן אחוזי הנחה או מחירים מדויקים.

## 9. סיכום

- התחייבות מתאימה לשימוש צפוי.
- Spot מתאים ל-interruptible.
- אל תסכן stateful production ב-Spot בלבד.
- מדוד utilization.
- עלות כוללת כוללת גם operations.

## 10. בדיקת הבנה

1. איזה workload מתאים ל-Spot?
2. מה היתרון של On-Demand?
3. מהו right-sizing?

## 11. הרחבה: השוואת עלות מעשית

On-Demand הוא היקר ביותר לשעה אך גמיש וללא commitment; מתאים ל-spikes ו-workload קצר. Savings Plans זולים יותר אם יש spend יציב, עם commitment לשעה; Compute Savings Plan גמיש יותר בין instance/family/Region, בעוד EC2 Instance Savings Plan נותן הנחה גדולה יותר אך פחות גמישות. RI מתאים לדפוס EC2 יציב ומוגדר, ו-Scheduled RI/commitment אינו פתרון לכל workload. Spot הוא הזול ביותר לרוב אך ניתן ל-interrupt, לכן משלבים אותו עם ASG ובסיס On-Demand. Dedicated Host יקר משמעותית, ומשתמשים בו בעיקר ל-BYOL או compliance.

## 12. ששת ה-Pillars

Operational Excellence: Cost Explorer, tagging ו-automation של purchase/termination. Security: אל תבחר Spot או discount במחיר של patching והרשאות. Reliability: mixed instances ו-capacity שלא תלויה ב-Spot בלבד. Performance Efficiency: right-size לפי CPU, memory, network ו-IO ולא לפי מחיר בלבד. Cost Optimization: מחויבות רק ל-baseline שנמדד; Spot ל-flexible queue. Sustainability: כיבוי idle, efficient architecture ו-scaling מפחיתים שעות compute.
