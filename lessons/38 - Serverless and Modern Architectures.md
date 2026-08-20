# 38 — Serverless & Modern Architectures

## 1. מה זה?
Serverless הוא שימוש בשירותים שבהם AWS מנהלת servers ו-capacity (Lambda, API Gateway, DynamoDB, Fargate). Microservices מפרקים domain ליחידות ownership/deployment; Event-driven architecture מחברת producers ו-consumers באמצעות events ו-queues.

## 2. למה צריך את זה?
הדפוסים מקטינים patching ומאפשרים scale independent, אך מוסיפים distributed failure, observability, retries ו-IAM boundaries. בוחרים complexity לפי requirements, לא לפי trend.

## 3. איך זה עובד?
Flow נפוץ: API Gateway→Lambda→DynamoDB, עם SQS עבור עבודה אסינכרונית, EventBridge ל-routing בין services, Step Functions ל-orchestration. Lambda היא stateless; state נשמר ב-DynamoDB/S3/RDS. SQS buffer סופג spike, consumer מעבד בקצב שלו, ו-DLQ שומר failures. Events צריכים idempotent handlers, retry/backoff ו-schema/versioning.

## 4. הדברים שחייבים לדעת למבחן
- “בלי לנהל servers” → Lambda/Fargate/managed database, אך עדיין מנהלים IAM, data ו-design.
- Async decoupling משפר resilience אך אינו מבטיח ordering או exactly-once.
- Lambda duration/concurrency limits ו-cold starts עשויים להשפיע על latency.
- Microservice צריך data ownership; shared DB יוצר coupling.
- API Gateway מתאים API serverless; ALB מתאים HTTP routing ל-EC2/ECS ויכול להיות פשוט/זול יותר ב-traffic רציף.
- Step Functions מתאים workflow עם retries/state, לא רק message fan-out.

## 5. עלות, תמחור ו-trade-offs
Lambda מחויבת requests ו-compute duration/memory; טובה ל-bursty workloads, אך compute רציף/צפוי עשוי להיות זול יותר ב-EC2 או containers. API Gateway מחויב requests (ולפי סוג API/features); ALB מחויב שעות ו-LCU. Fargate מחויב vCPU/memory בזמן הריצה וחוסך ניהול hosts, בעוד ECS על EC2 עשוי להיות זול יותר ב-utilization גבוה אך דורש patching/capacity.

EventBridge, SNS ו-SQS מחויבים לפי requests/throughput; retries ו-DLQ מגדילים messages. DynamoDB on-demand גמיש אך לרוב יקר יותר ל-throughput יציב מ-provisioned עם auto scaling. Distributed architecture מוסיפה logs, tracing, NAT ו-data transfer בין AZs — יש למדוד את כל ה-path.

## 6. ההבדלים החשובים
| Pattern | לבחור כש... | לא כש... |
|---|---|---|
| Serverless | variable traffic, minimum ops | צורך בשליטה מיוחדת/long-running |
| Microservices | teams/deploy/scale independent | domain קטן ו-simple |
| Event-driven | loose coupling, fan-out | צריך immediate synchronous result |
| SQS | durable work queue/backpressure | broadcast לכל subscribers |
| EventBridge | rule-based cross-service events | raw high-rate stream בלבד |

## 7. Well-Architected view
- **Operational Excellence:** IaC, versioned deployments, metrics/traces, DLQ ו-runbooks.
- **Security:** per-service IAM roles, private integrations, secret manager ו-validation של events.
- **Reliability:** queues, retries עם backoff, idempotency, timeouts ו-circuit breakers.
- **Performance Efficiency:** tune memory/concurrency, batching, caching ו-async paths.
- **Cost Optimization:** pay-per-use ל-bursty; consolidate chatty calls ולבחור Fargate/EC2 לפי utilization.
- **Sustainability:** managed/serverless ו-scale-to-zero מפחיתים idle capacity, אך הימנע מ-polling מיותר.

## 8. מלכודות במבחן
Serverless אינו “ללא cost/limits”; retry ללא idempotency יוצר duplicate orders; synchronous chain ארוך אינו באמת decoupled; microservices אינם answer אם requirement הוא minimum complexity.

## 9. Scenario מהעולם האמיתי
Order API מאשר מהר ושולח event ל-EventBridge. billing/shipping/notification services מקבלים events, ו-SQS לכל consumer מספק buffer ו-DLQ. Step Functions מנהל saga ופיצוי אם payment מצליח אך shipping נכשל. אין shared DB, וכל handler idempotent.

## 10. מה לא צריך לדעת
אין צורך לשנן framework libraries, קוד Lambda או כל integration פרטני; התמקד ב-flow, limits ו-trade-offs.

## 11. סיכום
- Serverless מפחית servers, לא אחריות.
- הפרד state מ-compute.
- Events ו-queues דורשים idempotency.
- SQS = buffer; EventBridge = routing.
- בחר complexity פרופורציונלית.

## 12. בדיקת הבנה
1. למה event consumer חייב להיות idempotent?
2. מתי Fargate עלול להיות יקר מ-EC2?
3. מתי microservices אינם הבחירה הטובה ביותר?
