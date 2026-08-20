# 30 — Application Decoupling

## 1. מה זה?
Decoupling מפריד בין רכיבים כך ש־producer ו־consumer אינם תלויים בזמינות או בקצב זה של זה. AWS patterns כוללים SQS queue, SNS fan-out, EventBridge ו־DLQ.

## 2. למה צריך את זה?
שירות downstream איטי לא אמור להפיל checkout. Queue סופג spike, מאפשר retry ונותן לצרכנים scale עצמאי. trade-offs: asynchronous processing, eventual consistency, duplicates ו־poison messages.

## 3. איך זה עובד?
Producer שולח message durable ל־SQS. Consumer קורא ומקבל visibility timeout; לאחר הצלחה מוחק. כשל גורם להודעה להופיע שוב, ואחרי `maxReceiveCount` היא עוברת ל־DLQ. Lambda/ASG יכולים לגדול לפי queue depth או age of oldest message. SNS topic משכפל ל־subscriptions; SNS→SQS נותן לכל consumer buffering עצמאי. EventBridge מוסיף filtering ו־routing.

## 4. הדברים שחייבים לדעת למבחן
- “decouple”, “buffer spikes”, “process asynchronously” → SQS.
- Standard SQS הוא at-least-once; FIFO נותן ordering/dedup לפי message group אך עם throughput trade-off.
- Visibility timeout צריך להיות ארוך מזמן העיבוד עם margin; הוא אינו retention.
- DLQ מבודד poison messages; צריך לנטר ולבצע re-drive.
- SNS הוא fan-out; SNS→SQS הוא fan-out + durable retry.
- כל consumer צריך idempotency.

## 5. עלות, תמחור ו־trade-offs
SQS מחויב לפי requests (batching מצמצם requests), retention ו־data transfer; SNS מחויב לפי publish/delivery ו־payload וכל יעד בנפרד. EventBridge, Lambda, NAT ו־cross-Region delivery מוסיפים רכיבי חיוב. Standard לרוב זול מ־FIFO ומ־Kinesis כשצריך queue פשוט. FIFO עולה יותר עבור ordering/dedup. SNS→SQS יקר יותר מ־queue יחיד אך מתאים ל־fan-out. long polling ו־batching מצמצמים עלויות empty receives.

## 6. ההבדלים החשובים
| Pattern | תועלת | trade-off |
|---|---|---|
| SQS Standard | buffer זול ו־scale consumers | duplicates, no global ordering |
| SQS FIFO | ordering/dedup לפי group | throughput ומגבלות גבוהים יותר |
| SNS | fan-out מיידי | אינו queue לצרכן איטי |
| SNS→SQS | fan-out + retry עצמאי | יותר queues ו־requests |
| DLQ | isolate failures | דורש remediation |

## 7. Well-Architected view
- **Operational Excellence:** dashboards ל־queue age, DLQ ו־errors; runbook ל־re-drive.
- **Security:** IAM נפרד producer/consumer, SSE עם KMS ו־redaction של secrets.
- **Reliability:** durable queue, retries/backoff, visibility tuning ו־idempotency.
- **Performance Efficiency:** long polling, batch receive/send ו־parallel consumers.
- **Cost Optimization:** batch requests, retention מינימלי ו־avoid fan-out מיותר.
- **Sustainability:** buffering ובאצוות מונעים polling ו־compute idle.

## 8. מלכודות במבחן
Queue לא מאיץ עיבוד; הוא מפריד וסופג עומס. visibility קצר מדי יוצר duplicate processing. DLQ אינו תחליף למדיניות retry. Queue יחיד אינו fan-out אם כמה מערכות צריכות כל event.

## 9. Scenario מהעולם האמיתי
Checkout מכניס order ל־SQS Standard. Lambda workers גדלים לפי age of oldest message. Billing ו־email מקבלים subscriptions נפרדים דרך SNS; email כושל עובר ל־DLQ בלי לעכב billing. Order ID הוא idempotency key.

## 10. מה לא צריך לדעת
לא לשנן כל quota או retry number; התמקד ב־delivery semantics, visibility/retention, fan-out, DLQ ו־back-pressure.

## 11. סיכום
1. Decoupling מגן על producer.
2. SQS סופג spikes.
3. SNS מפיץ לכמה consumers.
4. Standard הוא at-least-once.
5. FIFO עבור ordering כשנדרש.
6. Visibility ו־DLQ קריטיים.
7. Scale לפי backlog/age.

## 12. בדיקת הבנה
1. מה בוחרים כדי לספוג burst?
2. למה SNS לבדו אינו buffer?
3. מה קורה לאחר כשלי עיבוד חוזרים?

