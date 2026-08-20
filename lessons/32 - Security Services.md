# 32 — Security Services

## 1. מה זה?
שירותי security מכסים שכבות שונות: KMS להצפנה ומפתחות, Secrets Manager ל־credentials, WAF לתעבורת web, Shield ל־DDoS, GuardDuty לזיהוי threats, Inspector ל־vulnerabilities, Macie ל־sensitive S3 data ו־Security Hub לאיחוד findings.

## 2. למה צריך את זה?
Defense in depth מפחית blast radius: IAM מצמצם הרשאות, encryption מגן על data at rest, WAF מסנן requests, וכלי detection מזהים התנהגות או חשיפה. Security Hub הוא מרכז ניהול findings, לא detector בפני עצמו.

## 3. איך זה עובד?
KMS מנהל customer-managed או AWS-managed keys ו־CloudTrail מתעד שימוש; envelope encryption מאפשר להצפין data באמצעות data keys. Secrets Manager שומר ומסובב passwords/API keys ומאפשר ל־RDS rotation. WAF rules מסננות SQL injection, XSS, IP/rate patterns ב־CloudFront, ALB או API Gateway. Shield Standard מספק DDoS protection בסיסי; Advanced מוסיף הגנה/visibility בעלות נוספת. GuardDuty מנתח CloudTrail, VPC Flow Logs ו־DNS signals. Inspector סורק EC2/ECR/Lambda לפי תמיכה. Macie מגלה PII ב־S3. Security Hub מרכז findings ו־standards.

## 4. הדברים שחייבים לדעת למבחן
- password rotation → Secrets Manager; לא לשים secret בקוד או user data.
- encryption key policy/audit → KMS.
- SQLi/XSS/rate limiting → WAF.
- volumetric DDoS → Shield.
- threat detection → GuardDuty.
- software/package vulnerabilities → Inspector.
- PII discovery ב־S3 → Macie.
- central findings → Security Hub.
- IAM role עדיף על long-lived access key.

## 5. עלות, תמחור ו־trade-offs
KMS מחויב לפי customer-managed key וחלק מה־API requests; key rotation ו־cross-Region replication יכולים להוסיף עלות. Secrets Manager מחויב לפי secret לחודש ולפי API calls; Parameter Store עשוי להיות זול יותר ל־non-secret configuration או tiers מסוימים, אך אינו תמיד תחליף ל־rotation. WAF מחויב לפי Web ACL/rules ו־requests, Shield Advanced לפי subscription/usage. GuardDuty, Inspector ו־Macie מחויבים לפי נפח/מקורות שנבדקו; Security Hub לפי findings/security checks. הפעלה גורפת על כל Regions ו־resources יקרה יותר, אך selective coverage עלולה להשאיר blind spots.

## 6. ההבדלים החשובים
| צורך | שירות | לא לבלבל עם |
|---|---|---|
| key management/encryption | KMS | Secrets Manager |
| credential storage/rotation | Secrets Manager | IAM policy |
| HTTP filtering | WAF | GuardDuty |
| DDoS | Shield | WAF |
| runtime threat detection | GuardDuty | Inspector |
| vulnerability scan | Inspector | Macie |
| sensitive data discovery | Macie | GuardDuty |
| findings aggregation | Security Hub | detector יחיד |

## 7. Well-Architected view
- **Operational Excellence:** central Security Hub, automated ticket/remediation ו־rotation runbooks.
- **Security:** least privilege, KMS key policies, private secrets, WAF managed/custom rules ו־MFA.
- **Reliability:** protect dependencies without blocking valid traffic; test rotation ו־fail-open/closed decisions.
- **Performance Efficiency:** WAF rule order/scope ו־caching ב־CloudFront; scan לפי risk.
- **Cost Optimization:** scope scans, log/data-event selection ו־managed rules לפי צורך.
- **Sustainability:** detection ממוקד, retention סביר ו־avoid duplicate scanning/logging.

## 8. מלכודות במבחן
WAF לא מנתח malware כללי או DDoS volumetric לבדו. GuardDuty אינו vulnerability scanner. Security Hub אינו מחליף GuardDuty. KMS encrypts keys/data but אינו database secret store. Encryption במנוחה אינו מגן על credentials שנמצאים בקוד.

## 9. Scenario מהעולם האמיתי
CloudFront ו־ALB מוגנים ב־WAF, credentials ל־RDS ב־Secrets Manager עם rotation, EBS/RDS מוצפנים ב־KMS. GuardDuty ו־Inspector מפיקים findings, ו־Security Hub מאחד אותם לתגובה מרכזית.

## 10. מה לא צריך לדעת
לא לשנן כל finding type או rule ID. צריך לזהות שכבת הגנה, מקור הנתונים, הרשאות ועלות coverage.

## 11. סיכום
1. Secrets אינם keys.
2. WAF הוא HTTP layer.
3. Shield הוא DDoS layer.
4. GuardDuty detects threats.
5. Inspector scans vulnerabilities.
6. Macie מגלה PII ב־S3.
7. Security Hub מאחד findings.

## 12. בדיקת הבנה
1. מה מסובב DB password?
2. מה חוסם SQL injection?
3. מה מזהה PII ב־S3?

