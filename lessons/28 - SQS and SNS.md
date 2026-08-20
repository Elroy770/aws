# 28 — SQS & SNS

## 1. מה זה?
SQS הוא queue ל-decoupling; SNS הוא pub/sub topic שמפיץ הודעות subscribers.
## 2. למה צריך את זה?
הם מבודדים producers מ-consumers, סופגים spikes ומאפשרים fan-out.
## 3. איך זה עובד?
SQS Standard נותן at-least-once ו-best-effort ordering; FIFO נותן ordering ודדופליקציה בגבולותיו. Visibility timeout מסתיר message בזמן עיבוד. DLQ שומר כישלונות. SNS דוחף ל-SQS, Lambda, HTTP, email ועוד; SNS→SQS נותן fan-out אמין.
## 4. הדברים שחייבים לדעת למבחן
- queue אחד/consumer asynchronous → SQS.
- fan-out למספר מערכות → SNS + queues.
- DLQ ל-messages שלא עובדו.
- consumers חייבים idempotency.
## 5. ההבדלים החשובים
| Service | מודל |
|---|---|
| SQS | pull queue/buffer |
| SNS | push notification/fan-out |
| EventBridge | event routing לפי rules |
## 6. מלכודות במבחן
SQS אינו broadcast: message נצרך על ידי consumer אחד. כדי שכל מערכת תקבל event, שים queue לכל subscriber דרך SNS.
## 7. Scenario מהעולם האמיתי
Order service מפרסם ל-SNS; inventory, billing ו-email מקבלים queues נפרדים. כשל email לא מעכב billing.
## 8. מה לא צריך לדעת
לא לשנן timeout limits.
## 9. סיכום
- SQS decouples/buffers.
- SNS broadcasts.
- DLQ שומר failures.
- FIFO רק כש-order קריטי.
- idempotency חיוני.
## 10. בדיקת הבנה
1. מה מתאים ל-fan-out?
2. מה עושה visibility timeout?
3. למה צריך DLQ?
+

## 5. עלות, תמחור ו-trade-offs
SQS מחויב לפי requests (כולל פעולות message) ו-data transfer; SNS לפי publish/delivery ו-type היעד. FIFO לרוב יקר/מוגבל יותר מ-Standard אך קונה ordering/deduplication. SNS→SQS מוסיף queue ועלות, אך מבודד consumers ומונע איבוד/עיכוב של subscribers אחרים. DLQ ו-retention ארוך מוסיפים storage.

## 6. Well-Architected view
- **Operational Excellence:** metrics על age/depth, DLQ alarms ו-runbook ל-re-drive.
- **Security:** IAM policies, encryption/KMS, private endpoints ו-topic/queue policies.
- **Reliability:** durable queue, visibility timeout נכון, retries ו-idempotent consumers.
- **Performance Efficiency:** batch receive/send, Standard throughput ו-FIFO רק כשנדרש.
- **Cost Optimization:** long polling מפחית empty receives; TTL/retention ו-fan-out לפי צורך.
- **Sustainability:** buffering ו-batching מפחיתים invocations ו-network overhead.

## 7. מלכודות ו-Scenario
SQS אינו broadcast: consumer אחד מקבל message; fan-out אמין הוא SNS topic עם queue נפרד לכל subscriber. Order service יכול לפרסם ל-SNS ולבודד inventory, billing ו-email queues עם DLQ לכל אחד.
