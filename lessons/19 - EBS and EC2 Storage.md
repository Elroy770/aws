# 19 — EBS & EC2 Storage

## 1. מה זה?
Amazon EBS הוא block storage persistent ל-EC2 בתוך AZ, עם volumes ו-snapshots.

## 2. למה צריך את זה?
הוא מתאים ל-boot disks, databases וקבצים שדורשים block device, IOPS ו-latency צפוי.

## 3. איך זה עובד?
Volume מצורף ל-instance באותו AZ. Snapshots הם incremental ונשמרים ב-S3 managed by AWS; ניתן ליצור volume/AMI מהם ב-AZ או Region אחר לאחר copy. gp3 הוא general-purpose SSD; io2 מתאים ל-IOPS גבוהים וקריטיים; st1/sc1 מיועדים ל-throughput HDD workloads.

## 4. הדברים שחייבים לדעת למבחן
- EBS volume הוא AZ-scoped.
- EBS Multi-Attach מוגבל לסוגים/שימושים נתמכים ואינו shared filesystem כללי.
- Snapshot הוא הדרך לגיבוי/העברה, לא EBS replication אוטומטי בין AZs.
- EBS encryption כולל data, snapshots ו-data in transit בין EC2 ל-EBS.

## 5. ההבדלים החשובים
| Storage | BEST עבור |
|---|---|
| EBS | block ל-instance/AZ |
| EFS | file share multi-AZ |
| Instance store | זמני ומהיר |

## 6. מלכודות במבחן
אי אפשר לצרף volume מאותו AZ ל-instance ב-AZ אחר; צור snapshot ואז volume חדש.

## 7. Scenario מהעולם האמיתי
Database על EC2 משתמש ב-io2 לפי דרישת IOPS, snapshots אוטומטיים ו-copy cross-Region ל-DR. אל תשתף אותו בין nodes ללא cluster-aware design.

## 8. מה לא צריך לדעת
לא נדרש לשנן throughput/IOPS exact לכל volume type.

## 9. סיכום
- EBS הוא block ו-AZ-bound.
- snapshot הוא incremental.
- gp3 ברירת מחדל נפוצה.
- io2 ל-IOPS קריטי.
- EFS לשיתוף קבצים.

## 10. בדיקת הבנה
1. האם EBS חוצה AZ?
2. איך מעבירים volume ל-AZ אחר?
3. מה מתאים לקבצים משותפים?

## העמקה: volume design, snapshots ועלויות
EBS volume מחובר ל-EC2 באותו AZ; גיבוי ל-AZ/Region אחר נעשה באמצעות snapshot ויצירת volume חדש. Snapshot הוא incremental ברמת הבלוקים ונשמר באופן מנוהל, אך restore ראשון עלול להיות איטי עד שבלוקים נטענים (pre-warming לפי הצורך). ניתן להצפין volume חדש מ-snapshot, ו-KMS key חייב להיות זמין. Multi-Attach אינו shared filesystem: משתמשים בו רק עם cluster-aware coordination ותמיכה מתאימה.

gp3 מאפשר להגדיר storage, IOPS ו-throughput בנפרד, ולכן בדרך כלל cost-effective ל-general purpose. gp2 מקשר ביצועים ל-size ויכול להיות יקר יותר עבור IOPS נדרש. io2/io2 Block Express יקרים יותר אך מציעים IOPS ו-durability גבוהים ל-DB קריטי. st1 (throughput HDD) זול ל-sequential big-data; sc1 זול עוד יותר לנתונים נדירים, ואינו מתאים ל-random I/O. משלמים על provisioned GB/IOPS/throughput גם כשלא מנוצלים, ועל snapshots לפי data שנשמר.

### בדיקת workload לפי ששת ה-pillars
- **Operational Excellence:** CloudWatch על burst balance/latency, snapshot schedules, tagging ו-runbooks ל-recovery.
- **Security:** encryption, IAM least privilege, SG, ו-KMS policies; לא לשמור secrets על volume ללא צורך.
- **Reliability:** snapshots, cross-Region copies ו-Multi-AZ application design; EBS עצמו AZ-scoped.
- **Performance Efficiency:** volume type לפי random/sequential ו-IOPS/throughput, לא לפי גודל בלבד.
- **Cost Optimization:** gp3 right-sizing, מחיקת volumes מנותקים ו-snapshot lifecycle.
- **Sustainability:** מחיקת unattached storage, הקטנת provisioned capacity ו-right-sizing.
