# 20 — EFS & File Storage

## 1. מה זה?
EFS הוא managed elastic NFS filesystem אזורי; FSx מספק file systems ייעודיים; Storage Gateway מחבר on-prem ל-AWS storage.

## 2. למה צריך את זה?
כאשר כמה Linux instances צריכים אותו filesystem, או כאשר יש protocol/application ספציפי כגון Windows SMB, Lustre או NetApp.

## 3. איך זה עובד?
EFS משתמש ב-mount targets בכל AZ ונגיש דרך NFS, גדל אוטומטית ויכול להשתמש ב-lifecycle ל-IA. FSx for Windows File Server נותן SMB/AD; FSx for Lustre מתאים HPC וקשר ל-S3. Storage Gateway מגיע ב-File, Volume ו-Tape modes עם cache מקומי.

## 4. הדברים שחייבים לדעת למבחן
- shared Linux POSIX filesystem → EFS.
- EFS Standard multi-AZ; EFS One Zone הוא עלות מול resilience.
- Windows SMB → FSx for Windows, לא EFS.
- S3 object access/hybrid file → File Gateway לפי requirement.

## 5. ההבדלים החשובים
| שירות | BEST עבור |
|---|---|
| EFS | Linux NFS shared files |
| FSx Windows | SMB/Windows workloads |
| FSx Lustre | HPC/high-performance processing |
| S3 | objects, לא mount NFS native |

## 6. מלכודות במבחן
EFS אינו block storage ל-boot volume. "Several EC2 instances share files" כמעט תמיד מכוון ל-EFS.

## 7. Scenario מהעולם האמיתי
web instances בשני AZs צריכים uploads זהים. חבר אותם ל-EFS עם mount target בכל AZ ושמור access דרך SG של ה-instances.

## 8. מה לא צריך לדעת
אין צורך לשנן NFS versions או throughput modes לעומק.

## 9. סיכום
- EFS = NFS shared elastic.
- mount target לכל AZ נדרש.
- FSx לפי protocol/workload.
- Gateway מחבר hybrid.
- אל תבלבל file עם block/object.

## 10. בדיקת הבנה
1. מה מתאים ל-shared Linux files?
2. מה מתאים ל-SMB?
3. האם EFS מתאים ל-boot disk?

## העמקה: פרוטוקולים, ביצועים ועלויות
EFS הוא NFS אזורי: mount target הוא endpoint בתוך AZ, וה-client ניגש אליו דרך DNS ו-Security Group על TCP 2049. ב-Multi-AZ יוצרים mount target בכל AZ כדי להימנע מ-latency ו-cross-AZ charges. EFS One Zone זול יותר אך AZ יחיד. Storage classes Standard/IA ו-Archive, יחד עם lifecycle לפי הגיל האחרון של הקובץ, מאפשרים לשמור קבצים נדירים בזול; throughput mode ו-performance mode צריכים להתאים לדפוס workload.

EFS מחויב לפי GB מאוחסן (ולפי class), throughput ו-data transfer; cross-AZ access עלול להוסיף עלות. EBS לרוב זול ומהיר יותר ל-disk של instance יחיד, אך אינו shared. FSx for Windows מתאים ל-SMB/Active Directory, ו-FSx for Lustre מתאים ל-HPC ויכול לקשר S3. File Gateway מוסיף cache מקומי ו-S3 backend, אך אינו POSIX filesystem מלא.

### בדיקת workload לפי ששת ה-pillars
- **Operational Excellence:** monitor throughput/bursting, mount failures ו-lifecycle; automate backups ו-access points.
- **Security:** NFS SG, IAM authorization, POSIX permissions ו-encryption in transit/at rest.
- **Reliability:** mount targets בכל AZ, backups ו-replication/restore לפי RPO.
- **Performance Efficiency:** performance/throughput mode מתאימים, locality ב-AZ ו-parallel clients.
- **Cost Optimization:** lifecycle ל-IA/Archive, EFS One Zone לנתונים שניתנים לשחזור ו-EBS ל-single-instance.
- **Sustainability:** tiering, מחיקת קבצים זמניים ו-throughput שאינו over-provisioned.
