# 11 — VPC Security

## 1. מה זה ולמה צריך?

Security Groups (SG), Network ACLs (NACL) ו-VPC Flow Logs נותנים שכבות הגנה ואבחון. SG הוא בקרת משאב, NACL guardrail ברמת subnet, ו-Flow Logs הוא תיעוד metadata.

## 2. איך זה עובד?

SG מחובר ל-ENI, stateful, ומכיל allow rules בלבד; response ל-traffic מותר חוזר אוטומטית. NACL מחובר ל-subnet, stateless, בודק inbound וגם outbound לפי מספר rule ראשון שמתאים, ומכיל allow/deny. Flow Logs נוצר ברמת VPC/subnet/ENI ונשלח ל-CloudWatch Logs או S3; הוא אינו packet capture ואינו מכיל payload.

## 3. חובה למבחן

- SG: stateful, resource-level, אפשר SG-to-SG source; אין deny rule.
- NACL: stateless, subnet-level, allow/deny; צריך לאפשר ephemeral response ports.
- שכבות נבדקות יחד עם route: connection נכשל אם אחד מהם חוסם.
- Flow Logs מתאים לזיהוי `REJECT`, source/destination/port ו-troubleshooting, לא לקריאת תוכן.

## 4. עלויות והשוואה

SG ו-NACL אינם מחויבים בנפרד. Flow Logs עשוי ליצור CloudWatch ingestion/storage או S3 storage/requests; retention קצר ו-S3 lifecycle זולים יותר מארכיון Logs ארוך ב-CW, אך trade-off הוא query convenience. AWS Network Firewall/WAF מספקים inspection עמוק יותר אך מוסיפים hourly ו-processing costs.

| כלי | scope/התנהגות | בחירה |
|---|---|---|
| SG | ENI, stateful allow | ברירת המחדל ל-access של אפליקציה |
| NACL | subnet, stateless allow/deny | חסימה רחבה או compliance guardrail |
| Flow Logs | metadata | ראיות ואבחון, לא enforcement |

## 5. מלכודות ו-scenario

NACL שמאפשר inbound אך חוסם ephemeral outbound יפיל חיבור; SG אינו דורש rule תגובה. ALB מקבל 443 מהאינטרנט, ו-EC2 מאפשר 8080 רק מ-SG של ALB. אל תפתח `0.0.0.0/0` ל-database כדי “לפתור” timeout.

## 6. AWS Well-Architected — ששת ה-pillars

- **Operational Excellence:** Flow Logs, centralized log retention, automated rule review ו-runbooks.
- **Security:** least privilege, SG-to-SG, NACL deny לכתובות אסורות ו-encryption של destinations.
- **Reliability:** rules מתועדים ונבדקים ב-deployment; avoid accidental broad deny בנתיב קריטי.
- **Performance Efficiency:** rules מצומצמים ו-flow sampling/retention לפי צורך; לא inspection כבד ללא צורך.
- **Cost Optimization:** שלוט ב-CW ingestion, שלח long-term logs ל-S3 lifecycle ואל תפעיל firewall מיותר.
- **Sustainability:** retention ו-processing מדודים, פחות duplicate logs ו-rules פשוטים.

## 7. סיכום

SG = stateful resource control; NACL = stateless subnet control; Flow Logs = metadata. בדוק תמיד route, inbound ו-outbound בשני הכיוונים.
