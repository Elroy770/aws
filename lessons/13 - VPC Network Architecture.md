# 13 — VPC Network Architecture

## 1. מה זה ולמה צריך?

Peering, Transit Gateway (TGW), Site-to-Site VPN ו-Direct Connect מחברים VPCs, accounts ו-on-premises. הבחירה תלויה במספר רשתות, transitive routing, encryption, latency ועלות.

## 2. איך זה עובד?

VPC Peering הוא קשר one-to-one ללא transitive routing; CIDRs אינם יכולים לחפוף. TGW הוא regional hub שמחבר VPC attachments ו-VPN/DX, עם route tables נפרדים. VPN מוצפן מעל internet באמצעות מנהרות IPsec. Direct Connect הוא קישור dedicated דרך מיקום DX; הוא אינו encrypted by default, ולכן משלבים VPN אם נדרשת הצפנה.

## 3. חובה למבחן והשוואה

| קישור | מתאים ל | עלות/trade-off |
|---|---|---|
| Peering | שני VPCs עם תעבורה ישירה | אין hub hourly אך mesh גדל מהר; אין transitive |
| TGW | הרבה VPCs, hub-and-spoke, segmentation | attachment ו-per-GB costs; חוסך mesh ותפעול |
| VPN | hybrid מוצפן ומהיר להקמה | tunnel/processing ו-internet variability |
| Direct Connect | bandwidth/latency עקביים ו-private path | port/circuit/provider charges, lead time; VPN נוסף עולה יותר |

## 4. מלכודות ו-scenario

אין routing אוטומטי בין שני VPCs המחוברים ל-VPC שלישי באמצעות peering. לעשרים accounts, חבר כל VPC ל-TGW וה-data center באמצעות VPN או DX; הפרד route tables לפי environment והגבל propagation.

## 5. AWS Well-Architected — ששת ה-pillars

- **Operational Excellence:** centralized TGW governance, IaC, route propagation review ו-monitoring של tunnels/BGP.
- **Security:** segmentation, encryption VPN, SG/NACL ו-DX עם VPN כשנדרש confidentiality.
- **Reliability:** שתי VPN tunnels, DX redundant connections ו-TGW multi-AZ design; אל תישען על circuit יחיד.
- **Performance Efficiency:** DX ל-throughput יציב, TGW במקום שרשרת NAT/peering, ו-CIDR/MTU מתוכננים.
- **Cost Optimization:** peering לעומס קטן, TGW כש-mesh יקר תפעולית, VPN במקום DX כש-latency/volume מאפשרים.
- **Sustainability:** קונסולידציה ב-TGW, circuits בגודל מתאים וכיבוי attachments זמניים.

## 6. סיכום

Peering פשוט ולא transitive; TGW הוא hub; VPN מוצפן על internet; DX dedicated אך לא encrypted כברירת מחדל. תמיד בדוק routes, CIDR ו-redundancy.
