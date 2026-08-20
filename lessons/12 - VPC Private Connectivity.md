# 12 — VPC Private Connectivity

## 1. מה זה ולמה צריך?

VPC Endpoints מאפשרים גישה לשירותי AWS או services פרטיים בלי IGW, NAT או public IP. כך שומרים traffic פרטי ומקטינים NAT processing.

## 2. איך זה עובד?

Gateway Endpoint מוסיף route table entries עבור S3 או DynamoDB, זמין בתוך Region ובדרך כלל ללא hourly charge. Interface Endpoint יוצר ENI פרטי ב-subnet/AZ נבחר, משתמש ב-AWS PrivateLink וב-private DNS, ורוב שירותי AWS/API משתמשים בו. PrivateLink מאפשר provider לחשוף service מאחורי NLB לצרכנים פרטיים ללא peering.

## 3. חובה למבחן והשוואה

| סוג | שימוש | עלות/trade-off |
|---|---|---|
| Gateway | S3, DynamoDB | לרוב ללא hourly; תלוי route tables ומדיניות endpoint |
| Interface | רוב AWS APIs ו-SaaS | hourly לכל ENI/AZ + per-GB; SG ו-DNS נדרשים |
| PrivateLink | provider/consumer VPCs | private exposure בלי CIDR connectivity; provider/consumer משלמים לפי המודל |

Gateway Endpoint הוא לרוב זול יותר מ-NAT ל-S3/DynamoDB. Interface endpoint בכל AZ עולה יותר מאחד מרכזי, אך משפר HA ומונע cross-AZ transfer. Endpoint אינו הופך bucket לפרטי: IAM, bucket policy ו-endpoint policy עדיין קובעים authorization.

## 4. מלכודות ו-scenario

NAT Gateway אינו נדרש ל-S3 מפרייבט subnet כאשר Gateway Endpoint מתאים. Interface endpoint דורש SG שמאפשר HTTPS מהלקוחות ו-DNS resolution. Scenario: servers פרטיים קוראים artifacts מ-S3; הוסף Gateway Endpoint ו-policy שמגבילה גישה ל-bucket/endpoint.

## 5. AWS Well-Architected — ששת ה-pillars

- **Operational Excellence:** IaC ל-endpoints, ניטור endpoint errors ו-documentation של DNS/routes.
- **Security:** endpoint policies, bucket/IAM least privilege, private DNS ו-SG מצומצם.
- **Reliability:** Interface endpoints ביותר מ-AZ אחד; Gateway associations לכל route table רלוונטי.
- **Performance Efficiency:** path פרטי עם latency עקבי, local endpoint בכל AZ וצמצום NAT hops.
- **Cost Optimization:** Gateway ל-S3/DynamoDB, Interface רק לשירותים נדרשים וניתוח per-GB.
- **Sustainability:** פחות NAT/proxy processing, endpoints משותפים במידה שאינה פוגעת ב-isolation.

## 6. סיכום

Gateway = S3/DynamoDB route; Interface = ENI + PrivateLink; policy ו-DNS חשובים כמו ה-endpoint עצמו.
