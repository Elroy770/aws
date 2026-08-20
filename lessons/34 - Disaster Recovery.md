# 34 — Disaster Recovery

## 1. מה זה?
DR הוא יכולת התאוששות מכשל משמעותי, לרוב Region או account, לפי RTO (זמן downtime מותר) ו־RPO (אובדן data מותר). הוא שונה מ־HA שמטפל בכשל מקומי.

## 2. למה צריך את זה?
Multi-AZ לא מגן מכשל Region, טעות אנוש רחבה או ransomware. DR הוא trade-off בין מוכנות, עלות, complexity ומהירות; צריך להגדיר business impact לפני בחירת strategy.

## 3. איך זה עובד?
- **Backup and Restore:** backups/snapshots ו־IaC משוחזרים ב־Region אחר. זול אך RTO/RPO גבוהים.
- **Pilot Light:** data/core services פעילים, compute/application מוקטן או כבוי; בעת אירוע מרחיבים.
- **Warm Standby:** environment קטן אך עובד, מוכן ל־scale ול־traffic.
- **Active-Active:** Regions משרתים traffic יחד, עם replication ו־routing; RTO/RPO הנמוכים ביותר אך complexity הגבוהה.
Route 53 health checks, ARC או Global Accelerator מסייעים routing/failover. יש לשכפל גם secrets, keys, images, DNS/configuration ולא רק database.

## 4. הדברים שחייבים לדעת למבחן
- RTO/RPO נמוכים דורשים replication, capacity ו־testing יקרים יותר.
- backup לא מספיק אם נדרש recovery מהיר.
- Active-active אינו “בחינם”: data conflicts, consistency ו־failback מורכבים.
- Cross-Region snapshot נותן נקודת recovery, לא application failover בפני עצמו.
- DR plan חייב תרגול, מדידות ו־runbooks.

## 5. עלות, תמחור ו־trade-offs
Backup/Restore הזול ביותר: storage ו־copy, אך משלמים בזמן וב־compute בעת restore. Pilot Light משלם storage/data replication ו־מעט compute. Warm Standby משלם environment קבוע, LB ו־database capacity. Active-active משלם כמעט כפול compute, replication, monitoring ו־cross-Region data transfer. Savings Plans עשויים להוזיל compute קבוע אך אינם פותרים data transfer. אל תפעיל warm/active אם business RTO מאפשר backup.

## 6. ההבדלים החשובים
| Strategy | עלות יחסית | RTO/RPO | מתאים ל־ |
|---|---:|---|---|
| Backup/Restore | נמוכה | גבוהים | workloads לא קריטיים |
| Pilot Light | בינונית | בינוניים | core data ו־recovery מהיר יותר |
| Warm Standby | גבוהה | נמוכים | שירותים קריטיים |
| Active-Active | הגבוהה | הנמוכים ביותר | mission-critical ו־near-zero interruption |

## 7. Well-Architected view
- **Operational Excellence:** documented RTO/RPO, automated IaC, failover drills ו־post-incident review.
- **Security:** isolated recovery account, immutable backups, replicated KMS strategy ו־least privilege.
- **Reliability:** cross-Region copies/replication, tested DNS/traffic failover ו־dependency inventory.
- **Performance Efficiency:** warm capacity, prebuilt AMIs ו־load tests ל־recovery surge.
- **Cost Optimization:** לבחור strategy לפי business impact, להשהות compute ב־pilot light ולבצע lifecycle.
- **Sustainability:** לא להפעיל duplicate capacity מלאה אם RTO מאפשר backup/pilot light.

## 8. מלכודות במבחן
Multi-AZ אינו regional DR. Snapshot cross-Region ללא templates, IAM, DNS ו־secrets אינו solution. Warm standby אינו active-active: הוא ממתין ומתרחב בעת failover. RPO של אפס דורש synchronous/near-real-time design שאינו תמיד אפשרי בין Regions.

## 9. Scenario מהעולם האמיתי
שירות תשלומים דורש RTO דקות ו־RPO נמוך: warm standby ב־Region שני, database replication ו־Route 53 health-check failover. צוות מתרגל failover ומאמת שה־standby מסוגל לקבל peak traffic.

## 10. מה לא צריך לדעת
לא לשנן מוצרי DR ייעודיים או quota מספרי. התמקד ב־RTO/RPO, ארבע האסטרטגיות, replication ו־testing.

## 11. סיכום
1. RTO = זמן התאוששות.
2. RPO = data loss מותר.
3. Multi-AZ אינו DR אזורי.
4. Backup זול ואיטי.
5. Warm standby מהיר ויקר יותר.
6. Active-active מורכב ויקר ביותר.
7. שכפל גם configuration/secrets.
8. Test failover.

## 12. בדיקת הבנה
1. מה strategy הזולה ביותר?
2. מה נותן RTO נמוך יותר: warm או backup?
3. האם Multi-AZ מגן מכשל Region?

