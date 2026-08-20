# 01 — AWS Fundamentals

## 1. מה זה?

AWS בנויה מ-**Regions**, **Availability Zones (AZs)** ו-**Edge Locations**. Region הוא אזור גאוגרפי נפרד, AZ הוא failure domain מבודד בתוך Region, ו-Edge Location הוא אתר של רשת ההפצה (CloudFront) הקרוב למשתמש. לכל שירות יש scope: IAM הוא global, בעוד EC2, VPC ו-EBS הם regional או AZ-scoped.

## 2. למה צריך את זה?

בחירת המיקום משפיעה על latency, data residency, זמינות, מחיר ו-data transfer. Multi-AZ מגן מכשל של אתר בתוך Region; Multi-Region הוא שכבת DR נפרדת, אך דורש שכפול נתונים, ניתוב ותפעול יקרים ומורכבים יותר.

## 3. איך זה עובד?

Region כולל כמה AZs המחוברים ברשת מהירה אך מופרדים פיזית בחשמל, קירור ורשת. משאבים כמו EBS volume שייכים ל-AZ, ולכן אי אפשר לצרף אותו ישירות ל-instance ב-AZ אחר. CloudFront שומר אובייקטים ב-cache ב-Edge; ב-cache miss הבקשה נשלחת ל-origin כגון S3 או ALB. AWS מאבטחת את התשתית (security of the cloud); הלקוח אחראי לדאטה, IAM, הצפנה והגדרות השירות (security in the cloud).

## 4. על מה משלמים ומה זול/יקר?

מחירי services עשויים להשתנות לפי Region. משלמים על compute/storage/requests ועל data transfer: העברה החוצה לאינטרנט ו-cross-Region בדרך כלל יקרה יותר מתעבורה פנימית, והעברה בין AZs יכולה להיות מחויבת. CloudFront מוסיף חיובי requests ו-egress, אך עשוי להוזיל origin egress כשיש cache hit. Multi-Region מכפיל משאבים, שכפול ו-operations; Multi-AZ מגדיל לרוב את מספר המשאבים, אך זול ופשוט יותר מ-DR מלא ב-Region נוסף.

## 5. הדברים שחייבים לדעת למבחן

- Multi-AZ = הגנה מפני כשל AZ; Multi-Region = הגנה מפני כשל Region/latency גלובלי.
- Edge Location אינו AZ ואינו מקום להריץ בו EC2.
- EC2/VPC/EBS הם regional/AZ-aware; IAM הוא global.
- Availability אינו DR: גיבוי ושכפול נדרשים מעבר לפריסה ב-AZ אחד.
- Shared Responsibility משתנה לפי השירות: AWS מנהלת hardware, הלקוח מנהל configuration ו-data.

## 6. ההבדלים החשובים

| בחירה | מתי לבחור | עלות/חיסרון |
|---|---|---|
| Single-AZ | dev/test או workload לא קריטי | נקודת כשל אחת, לרוב זול יותר |
| Multi-AZ | production וזמינות בתוך Region | יותר instances/storage ו-AZ transfer |
| Multi-Region | DR אזורי, compliance או משתמשים גלובליים | שכפול, routing ו-operations יקרים ומורכבים |
| CloudFront | static/cacheable content ו-latency גלובלי | חיובי requests/egress ו-cache invalidation |

## 7. ששת ה-Pillars בעדשת התשתית

| Pillar | שאלה מעשית |
|---|---|
| Operational Excellence | האם deployment, monitoring ו-runbooks מאפשרים שינוי בטוח? |
| Security | האם IAM least privilege, encryption ו-network controls מוגדרים? |
| Reliability | האם workload שורד כשל AZ ויכול להתאושש לפי RTO/RPO? |
| Performance Efficiency | האם Region/Edge, סוג משאב ו-cache מתאימים ל-latency? |
| Cost Optimization | האם נמנעים מ-Multi-Region או data transfer ללא צורך? |
| Sustainability | האם cache, right-sizing ו-scaling מצמצמים compute ואנרגיה? |

## 8. מלכודות במבחן

"Survive an AZ failure" אינו דורש Multi-Region. "Lowest latency for global users" בדרך כלל מצביע על CloudFront או Region קרוב, לא על הוספת AZs בלבד. אל תניח ש-cross-Region replication חינמי או אוטומטי.

## 9. Scenario מהעולם האמיתי

חנות ישראלית מריצה ALB ו-ASG בשני AZs, מסד נתונים Multi-AZ, וקבצים סטטיים ב-S3 דרך CloudFront. גיבוי ל-Region שני נדרש רק אם RTO/RPO ותקציב מצדיקים זאת. CloudWatch מודד latency ועלויות, ו-lifecycle/TTL מצמצמים origin traffic ואנרגיה.

## 10. סיכום ובדיקת הבנה

Region הוא גבול גאוגרפי; AZ הוא יחידת בידוד; Edge נועד למסירה קרובה. תכנן לפי failure domain, מחיר ודרישות עסקיות.

1. איזה design מגן מפני כשל AZ?
2. מי משלם בדרך כלל על cross-Region data transfer?
3. איזה pillar מצדיק cache כדי לצמצם latency ו-origin compute?
