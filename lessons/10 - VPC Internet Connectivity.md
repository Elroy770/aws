# 10 — VPC Internet Connectivity

## 1. מה זה ולמה צריך?

IGW, NAT Gateway ו-NAT Instance מספקים דפוסי גישה שונים בין VPC לאינטרנט. Web tier צריך inbound; private workloads צריכים לעיתים outbound ל-patches, APIs ו-repositories בלי להיות נגישים מבחוץ.

## 2. איך זה עובד?

IGW מחובר ל-VPC ו-route table מפנה אליו `0.0.0.0/0` (או IPv6). למשאב public נדרשת גם public IPv4/EIP ו-SG/NACL מתאימים. NAT Gateway נמצא ב-public subnet עם Elastic IP; private subnet מנתב אליו outbound, והוא מבצע source NAT ואינו מקבל connections initiated מהאינטרנט. NAT Instance הוא EC2 שמנהלים לבד, עם source/destination check מבוטל ו-routing מתאים.

## 3. הדברים שחייבים לדעת

- NAT Gateway חייב public subnet ו-route ל-IGW; ה-private route מצביע ל-NAT.
- ל-HA יוצרים NAT Gateway בכל AZ ומנתבים כל private subnet ל-NAT באותה AZ.
- NAT אינו נדרש ל-S3/DynamoDB כאשר Gateway Endpoint מתאים.
- IGW אינו “proxy”; הוא מאפשר routing, אך public IP ו-security rules עדיין נדרשים.
- NAT Gateway מנוהל ו-scalable יותר; NAT Instance מאפשר custom software אך הוא נקודת ניהול/כשל.

## 4. עלויות והשוואה

| רכיב | עלות/trade-off |
|---|---|
| IGW | אין hourly charge; עדיין משלמים על data transfer/public IPv4 לפי התמחור העדכני. |
| NAT Gateway | hourly לכל AZ + per-GB processed; HA עם כמה AZs יקר יותר אך מונע cross-AZ ו-failure. |
| NAT Instance | EC2, EBS ותפעול; עשוי להיות זול לעומס קטן או custom filtering, אך דורש scaling ו-HA עצמי. |
| Gateway Endpoint | בדרך כלל זול יותר מ-NAT ל-S3/DynamoDB, ללא hourly charge; מדיניות IAM עדיין חלה. |

אל תעביר traffic רב ל-S3 דרך NAT רק כדי “לפשט”: endpoint חוסך processing ונותן path פרטי.

## 5. מלכודות ו-scenario

NAT Gateway אינו מקבל inbound. NAT ב-AZ יחיד עשוי לעבוד אך יוצר SPOF ו-cross-AZ charges. Scenario: instances פרטיים בשתי AZs צריכים updates; צור NAT בכל AZ, routes מקומיים, ו-S3 Gateway Endpoint ל-artifacts.

## 6. AWS Well-Architected — ששת ה-pillars

- **Operational Excellence:** IaC, alarms ל-NAT errors/bytes, runbook לבדיקת routes ו-flow logs.
- **Security:** private subnets, least-privilege egress, NAT ללא inbound ו-endpoints עם policies.
- **Reliability:** NAT per AZ, שתי AZs ויכולת חלופית ל-IGW/routes.
- **Performance Efficiency:** הימנע מ-cross-AZ NAT, endpoint ל-AWS services ובחר IPv4/IPv6 לפי workload.
- **Cost Optimization:** נתח NAT GB, השתמש endpoints ו-NAT Instance רק כשעלות התפעול מוצדקת.
- **Sustainability:** צמצם duplicate processing ונתיבים ארוכים; כבה NAT/instances לא נחוצים בסביבות זמניות.

## 7. סיכום

IGW = internet routing, NAT = private outbound, endpoint = private AWS service path. שאל תמיד מי יוזם, לאן ה-route מצביע, ומהו failure domain.
