# 23 — DynamoDB

## 1. מה זה ולמה?
DynamoDB הוא managed serverless NoSQL מסוג key-value/document ל-latency נמוך ו-scale גבוה ללא שרתים. מתכננים את הטבלה לפי access patterns, לא לפי joins של SQL.

## 2. איך זה עובד?
Partition key קובע פיזור; sort key מאפשר range Query בתוך partition. GSI מאפשר alternate key ו-LSI משתמש באותו partition key. Query ממוקד עדיף על Scan. On-demand מתאים traffic לא צפוי; provisioned עם auto scaling לעומס יציב. Streams מפרסם שינויי items, TTL מוחק expired items eventually, ו-Global Tables משכפל multi-Region. DAX הוא cache, לא source of truth.

## 3. הדברים שחייבים לדעת למבחן
- hot partition נובע מ-key לא מאוזן; Query יעיל וזול מ-Scan.
- eventual consistency זולה יותר; strong consistency כשחייבים read עדכני.
- GSI נושא storage/capacity נפרדים; retries מחייבים idempotency.

## 4. עלות, תמחור ו-trade-offs
משלמים requests/capacity, storage, indexes, Streams, PITR/backups, Global Tables ו-transfer. On-demand גמיש אך יקר יותר ל-request יציב; provisioned זול יותר כשניתן לחזות. GSI/Global Tables מוסיפים כתיבות ו-storage בכל Region. DAX מוסיף nodes וכדאי רק ב-read-heavy עם hit rate גבוה.

## 5. ההבדלים החשובים
| צורך | בחירה | trade-off |
|---|---|---|
| traffic לא צפוי | On-demand | יקר יותר לעומס קבוע |
| query חלופי | GSI | capacity/storage נוספים |
| event על שינוי | Streams | processing נוסף |
| active-active | Global Tables | replication ו-transfer |

## 6. Well-Architected view
- **Operational Excellence:** alarms על throttles/ConsumedCapacity ו-runbook ל-hot key.
- **Security:** IAM fine-grained, KMS, VPC endpoints ו-CloudTrail.
- **Reliability:** PITR, backups, retries/backoff ו-Global Tables לפי RTO.
- **Performance Efficiency:** key מפוזר, Query במקום Scan, pagination ו-DAX כשמתאים.
- **Cost Optimization:** provisioned ליציב, projection מצומצם ל-GSI ו-TTL.
- **Sustainability:** serverless ו-right-sizing מפחיתים idle ו-Scan מיותר.

## 7. מלכודות ו-Scenario
חיפוש לפי email בנוסף ל-userId הוא GSI, לא Scan. מערכת משחק גלובלית יכולה להשתמש ב-Global Tables + TTL ל-sessions, Streams ל-analytics ו-On-demand ל-launch spikes.

## 8. סיכום ובדיקת הבנה
Model לפי access pattern; Query עדיף על Scan; key מפוזר מונע hot partition; GSI ל-index; capacity לפי predictability; features מוסיפים עלות.

1. מה גורם hot partition? 2. מתי provisioned זול יותר? 3. מה ההבדל בין GSI ל-Streams?
