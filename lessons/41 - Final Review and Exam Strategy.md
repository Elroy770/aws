# 41 — Final Review & Exam Strategy

## 1. מה זה?
שיטת חזרה והחלטה מהירה לקראת SAA-C03. המטרה היא לזהות דרישה, failure domain ו-trade-off ולבחור BEST תחת זמן.

## 2. למה צריך את זה?
השאלות משלבות שירותים ומסיחות עם תשובה אפשרית אך יקרה, לא מאובטחת או עתירת operations. הבנה של pattern עדיפה על שינון feature list.

## 3. איך זה עובד?
קרא קודם את סוף ה-scenario וסמן must, most, minimum, least, without managing, highly available, private, cost-effective. חלץ data type, latency, scale, RTO/RPO ו-AZ/Region; צייר data flow; פסול תשובות שמפרות security/availability; השווה cost, operations ו-complexity; בדוק state, credentials, retries ו-recovery.

## 4. הדברים שחייבים לדעת למבחן
- SG הוא stateful ו-NACL stateless; SG resource-level, NACL subnet-level.
- Multi-AZ = availability/failover; replica/cache = read scale.
- SQS = queue; SNS = fan-out; EventBridge = event routing.
- NAT = general outbound Internet; Endpoint = private AWS service access.
- ALB = Layer 7 HTTP rules; NLB = Layer 4 high performance/static IP.
- EBS = block/one AZ; EFS = shared file; S3 = durable object.
- On-Demand גמיש, Savings/RI ל-baseline, Spot ל-interruptible.

## 5. עלות, תמחור ו-trade-offs
בכל תשובה שאל על instance hours, requests, storage, IOPS, NAT processing, inter-AZ/egress, replicas, logs ו-operations. S3 Standard יקר יותר לאחסון ומתאים ל-hot data; IA/Glacier זולים יותר לאחסון אך retrieval/minimum duration מגבילים. Managed/serverless מקטינים labor ומשלמים לפי שימוש; EC2 תמיד-on עשוי לנצח ב-load רציף. Multi-AZ ו-cross-Region עולים יותר אך נדרשים אם RTO/RPO דורשים אותם. Cost-effective הוא TCO, לא שורת billing אחת.

## 6. ההבדלים החשובים
| Pair | זיכרון מהיר | מלכודת |
|---|---|---|
| Multi-AZ / Replica | HA / reads | replica אינה בהכרח failover |
| SQS / SNS | queue / fan-out | SNS אינו durable work queue לבדו |
| NAT / Endpoint | Internet / private AWS | NAT יוצר hourly + processing |
| EBS / EFS / S3 | block / file / object | EBS אינו shared multi-AZ file |
| ALB / NLB | L7 / L4 | NLB אינו path routing |

## 7. Well-Architected view
- **Operational Excellence:** automate, monitor, document decisions, test restores ו-review incidents.
- **Security:** least privilege, encryption, private subnets, WAF, secrets ו-audit.
- **Reliability:** remove single-AZ points, retries/health checks, backups ו-tested DR.
- **Performance Efficiency:** cache, right-size, choose protocol/storage לפי latency ו-throughput.
- **Cost Optimization:** measure, lifecycle, commitment ל-baseline, Spot לגמיש ובדוק transfer.
- **Sustainability:** eliminate idle, scale with demand, efficient storage ו-managed services.

## 8. מלכודות במבחן
אל תבחר answer שמוסיף servers אם managed עומד בדרישות. היזהר מ-most cost-effective, minimum operational overhead, durable, private ו-survive a Region failure. זכור ש-HA, encryption ו-backup אינם אותו דבר.

## 9. Scenario מהעולם האמיתי
Private EC2 חייב לקרוא S3 ללא Internet ובעלות הנמוכה ביותר: S3 Gateway Endpoint. אם נדרש access ל-SaaS ציבורי, NAT הוא הבחירה המתאימה. ההבדל נקבע על ידי destination.

## 10. מה לא צריך לדעת
לא לשנן UI, מחירים מדויקים, obscure limits או syntax. אל תלמד answer letters; תרגל reasoning וניתוח distractors.

## 11. סיכום
- התחל ב-requirement words.
- בדוק scope: instance/subnet/AZ/Region/global.
- הפרד HA, scaling, backup ו-DR.
- בחר managed כשזה מפחית operations ועומד בדרישות.
- חשב requests, storage ו-transfer hidden costs.
- הסבר למה כל distractor נכשל.
- תרגל sets של 10 וסווג טעויות.

## 12. בדיקת הבנה
1. Multi-AZ או replica ל-read scaling?
2. מה פותר private S3 access בזול?
3. אילו מילים מרמזות על managed solution?
