# 29 — Event-Driven Architecture

## 1. מה זה?
ב־Event-Driven Architecture רכיב מפרסם עובדה שקרתה (event), ורכיבים אחרים מגיבים אליה ללא קריאה סינכרונית ישירה. ב־AWS הבחירה המרכזית היא EventBridge ל־business events, Kinesis ל־high-volume streams ו־Step Functions ל־workflow stateful.

## 2. למה צריך את זה?
EDA מפחית coupling, סופג bursts ומאפשר להוסיף consumers בלי לשנות producer. הוא מתאים ל־orders, audit, SaaS integration ו־telemetry. המחיר: eventual consistency, duplicates וסדר עיבוד שאינו מובטח בכל שירות.

## 3. איך זה עובד?
- **EventBridge** מקבל events מ־AWS, applications או SaaS. Event bus מפעיל rules לפי pattern ומעביר ל־Lambda, SQS, SNS או Step Functions; אפשר archive/replay ו־schema discovery.
- **Kinesis Data Streams** שומר records ב־shards. partition key קובע shard ולכן משפיע על ordering ועל hot shards; כמה consumers יכולים לקרוא אותו stream.
- **Step Functions** מריץ state machine עם branching, parallelism, retries, catches ו־timeouts. Standard מתאים ל־durable workflows ארוכים; Express ל־high-volume קצר.
- Events הם בדרך כלל at-least-once. Consumers צריכים idempotency, correlation ID ו־DLQ/retry.

## 4. הדברים שחייבים לדעת למבחן
- business event routing / SaaS integration → EventBridge.
- ordered high-throughput streaming → Kinesis; ordering הוא בתוך shard.
- multi-step orchestration, retries ו־human approval → Step Functions.
- EventBridge הוא router, לא durable queue; target מסוג SQS נותן buffering.
- Kinesis retention מוגבל; S3 מתאים ל־long-term archive.
- אל תניח exactly-once: deduplicate לפי event ID או business key.

## 5. עלות, תמחור ו־trade-offs
EventBridge מחויב לפי events, ו־targets כמו Lambda, SQS ו־Step Functions מחויבים בנפרד. Kinesis מחויב לפי provisioned או on-demand capacity, shards ו־retention. Step Functions מחויב לפי transitions ב־Standard או requests/duration ב־Express. Cross-Region delivery/data transfer מוסיפים עלות.

SQS + Lambda לרוב זול ופשוט יותר מ־Kinesis כשצריך queue ולא replay/ordering של stream. Kinesis on-demand מפחית ניהול אך עלול להיות יקר יותר בקצב יציב. EventBridge חוסך custom routing code אך rules/targets רבים מגדילים עלות. Standard Step Functions יקר יותר בקצב עצום, אך נותן durability ו־audit; Express מתאים וזול יותר ל־high-volume קצר.

## 6. ההבדלים החשובים
| שירות | מתי לבחור בו | מתי לא |
|---|---|---|
| EventBridge | business events, rules, SaaS/AWS integration | stream analytics או queue עם back-pressure |
| Kinesis Data Streams | telemetry, ordering לפי partition, replay קצר | routing עסקי פשוט או archive ארוך |
| SQS | buffer ועבודה asynchronous | fan-out/rich filtering לבדו |
| Step Functions | orchestration עם state/retry/approval | message transport או stream ingestion |

## 7. Well-Architected view
- **Operational Excellence:** schemas/versioning, correlation IDs, dashboards, replay runbook ותרגול כשלי consumers.
- **Security:** IAM least privilege לכל bus/stream/target, KMS ו־validation של payload.
- **Reliability:** retries עם backoff, DLQ, idempotent consumers ו־managed multi-AZ services.
- **Performance Efficiency:** partition keys/shards נכונים, batching ו־parallel consumers; filtering מצמצם processing.
- **Cost Optimization:** filter events, לבחור SQS כשאין צורך ב־stream, ולהתאים Standard/Express.
- **Sustainability:** asynchronous batching מפחית compute idle ו־payloads מיותרים.

## 8. מלכודות במבחן
Step Functions אינו queue. EventBridge אינו מבטיח ordering גלובלי. Kinesis מבטיח ordering בתוך shard בלבד. Target לא זמין דורש retry/DLQ. אל תבחר Kinesis רק משום שכתוב “real time” אם מדובר ב־business routing.

## 9. Scenario מהעולם האמיתי
`OrderCreated` נכנס ל־EventBridge: rule אחד שולח ל־SQS עבור billing, אחר מפעיל Step Functions לאישור ומשלוח, ושלישי שולח analytics ל־Kinesis. כל consumer idempotent.

## 10. מה לא צריך לדעת
אין צורך לשנן כל integration או quota. חשוב להבין delivery model, ordering, retry, retention והבחירה בין router, queue, stream ו־orchestrator.

## 11. סיכום
1. Event הוא עובדה, לא command סינכרוני.
2. EventBridge מנתב business events.
3. Kinesis מטפל ב־ordered streams לפי shard.
4. Step Functions מנהל workflow stateful.
5. SQS מספק buffer.
6. delivery לרוב at-least-once.
7. idempotency ו־DLQ הם חלק מה־design.

## 12. בדיקת הבנה
1. איזה שירות מתאים ל־business event routing עם SaaS?
2. היכן נשמר ordering ב־Kinesis?
3. למה consumer חייב להיות idempotent?

