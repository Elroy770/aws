# 05 — EC2 Fundamentals

## 1. מה זה?

Amazon EC2 מספק virtual servers עם שליטה על OS, instance type, network ו-storage.

## 2. למה צריך את זה?

בחר בו כשנדרשת שליטה ברמת השרת, תוכנה קיימת, runtime מיוחד או workload שלא מתאים ל-serverless/container מנוהל.

## 3. איך זה עובד?

מפעילים instance מ-AMI, בוחרים instance type, subnet, security group ו-IAM role. EBS הוא אחסון persistent ברשת; instance store הוא זמני. User data מריץ bootstrap בעת launch, ו-instance metadata מספק מידע/credentials לתהליך על ה-instance.

## 4. הדברים שחייבים לדעת למבחן

- AMI הוא template; snapshot אינו AMI בפני עצמו.
- EBS נשמר לאחר stop כברירת מחדל; instance store אובד כש-host נכשל/instance מסתיים.
- Security Group הוא stateful ומוצמד ל-ENI.
- אל תכניס credentials ב-user data; השתמש ב-IAM role.

## 5. ההבדלים החשובים

| אחסון | מתאים ל | לא מתאים ל |
|---|---|---|
| EBS | boot volume, DB, data persistent | shared POSIX filesystem |
| Instance store | cache זמני מהיר | data שחייב לשרוד |
| EFS | קבצים משותפים לכמה instances | boot volume טיפוסי |

## 6. מלכודות במבחן

Stop/start עלול להעביר instance ל-host אחר; אל תשתמש ב-instance store כבסיס נתונים.

## 7. Scenario מהעולם האמיתי

יישום legacy דורש agent ותוכנה מותאמת. הרץ EC2 ב-ASG, שמור data ב-RDS/S3, והשתמש ב-user data כדי לבנות כל instance מחדש.

## 8. מה לא צריך לדעת

אין צורך לשנן כל משפחת instance או מהירות CPU מדויקת.

## 9. סיכום

- EC2 נותן שליטת שרת.
- AMI = template.
- EBS persistent, instance store ephemeral.
- Roles במקום keys.
- הפוך instances ל-disposable.

## 10. בדיקת הבנה

1. איזה storage אובד ב-termination?
2. מה תפקיד AMI?
3. איך EC2 מקבל גישה בטוחה ל-S3?

## 11. הרחבה: עלויות וששת ה-Pillars

משלמים לפי instance type וזמן הרצה (ולעתים גם EBS, snapshots, Elastic IP, requests ו-data transfer). EBS gp3 לרוב מאפשר להפריד volume size מ-IOPS/throughput; הוא זול וגמיש יותר כשלא צריך ביצועי io1/io2. Instance store מספק latency טוב וללא EBS volume charge, אך data אובד בכשל ולכן מתאים ל-cache בלבד. EFS יקר יותר מ-EBS ליחידת אחסון אך חוסך ניהול copies כשכמה instances צריכים filesystem משותף.

| Pillar | החלטת EC2 |
|---|---|
| Operational Excellence | AMI/launch template, User Data, Systems Manager ו-metrics |
| Security | IAM role, IMDSv2, SG מצומצם ו-EBS encryption |
| Reliability | ASG ו-Multi-AZ, EBS snapshots ו-state מחוץ ל-instance |
| Performance Efficiency | instance family/right-size, EBS tuning ו-cache |
| Cost Optimization | stop dev, delete orphaned EBS/EIP, לבחור pricing model |
| Sustainability | כיבוי idle, right-size ו-scaling לפי ביקוש |
