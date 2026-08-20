# 09 — VPC Fundamentals

## 1. מה זה?

VPC הוא virtual network מבודד ברמת Region. הוא מגדיר CIDR, subnets, route tables, gateways/endpoints ובקרות traffic. VPC אינו “שרת” ואינו מקבל public access בלי רכיבים ו-routes מפורשים.

## 2. למה צריך את זה?

VPC נותן שליטה על address space, בידוד, reachability וארכיטקטורת tiers: ingress, application ו-database. הוא מאפשר לחבר workloads ל-internet, ל-AWS services ול-on-premises בצורה מבוקרת.

## 3. איך זה עובד?

VPC מקבל IPv4 CIDR (ואפשר IPv6). Subnet הוא טווח כתובות בתוך VPC ושייך ל-AZ יחיד; מחלקים workloads בין AZs לצורך availability. כל subnet משויך route table אחת, עם `local` route ל-VPC. אם subnet route table מפנה `0.0.0.0/0` ל-Internet Gateway ויש למשאב public IPv4/EIP, הוא יכול להיות public. שם כמו “private-subnet” אינו משנה reachability.

NAT Gateway, endpoints, peering ו-TGW הם רכיבים נפרדים. Security Group ו-NACL מחליטים אם traffic מותר לאחר שה-route קיים. IP ציבורי לבדו אינו מספיק ללא route ל-IGW, ו-route אינו עוקף security controls.

## 4. הדברים שחייבים לדעת למבחן

- VPC הוא regional; subnet ו-NAT Gateway הם AZ-scoped.
- Subnet אינו חוצה AZ; בנה לפחות subnet מקביל בכל AZ ל-HA.
- Route table קובע next hop; longest-prefix match מנצח.
- `local` route מאפשר traffic בתוך VPC; אין transitive routing אוטומטי.
- Public subnet = IGW route + resource עם public address + rules מתאימים.
- כתובות מסוימות בכל subnet שמורות AWS, לכן תכנן CIDR עם מרווח.
- CIDR של VPCs שעתידים להתחבר אסור שיחפוף.

## 5. עלויות ותכנון חסכוני

VPC, route tables, subnets ו-IGW אינם מחויבים בפני עצמם. העלות נוצרת מהרכיבים וה-traffic: NAT Gateway לפי שעה ו-GB processed, interface endpoints לפי שעה לכל AZ/ENI ו-data processed, Transit Gateway לפי attachments/GB, public IPv4 לפי התמחור העדכני, ו-data transfer בין AZs/Regions או החוצה.

| בחירה | trade-off עלות |
|---|---|
| יותר AZs/subnets | יותר availability אך עלות רכיבים zonal ותפעול. |
| NAT מרכזי ב-AZ אחד | זול יותר אך cross-AZ transfer ו-single-AZ failure; לא ברירת מחדל ל-HA. |
| Gateway Endpoint ל-S3/DynamoDB | בדרך כלל זול יותר מ-NAT, ללא hourly endpoint charge. |
| CIDR גדול ומתוכנן | כמעט ללא עלות ישירה, אך מונע migration/rework יקר ו-CIDR overlap. |

## 6. ההבדלים החשובים

| רכיב | Scope | תפקיד |
|---|---|---|
| VPC | Region | גבול הרשת וה-address space |
| Subnet | AZ | טווח IP ל-workloads |
| Route table | VPC/subnet association | next hop ו-reachability |
| Security Group | ENI/resource | stateful allow rules |
| NACL | Subnet | stateless allow/deny guardrail |

## 7. מלכודות במבחן

- “Public subnet” אינו checkbox; בדוק route ל-IGW וגם public address.
- private subnet עם public IP לא נהיה בטוח או private.
- VPC הוא regional, אך subnet הוא AZ-specific; אל תציע subnet אחד ל-HA רב-AZ.
- route תקין לא עוקף SG/NACL, ו-SG תקין לא יוצר route.

## 8. Scenario מהעולם האמיתי

בנה VPC עם `/16`, public subnets ל-ALB בשתי AZs, private application subnets בשתי AZs ו-private database subnets. Application מקבל outbound דרך NAT או endpoints, ו-RDS אינו מקבל public access. CIDR אינו חופף ל-on-premises כדי לאפשר TGW/DX בעתיד.

## 9. AWS Well-Architected — ששת ה-pillars

- **Operational Excellence:** IaC, tagging, diagrams, VPC Flow Logs ו-runbooks לבדיקת routes/SG/NACL.
- **Security:** private-by-default tiers, least privilege, אין `0.0.0.0/0` ל-database, והפרדה לפי trust boundary.
- **Reliability:** subnets ורכיבים בשתי AZs לפחות ופחות נקודות כשל יחידות.
- **Performance Efficiency:** CIDR/subnet sizing נכון, הימנע מ-cross-AZ hops מיותרים, ובחר connectivity לפי latency/throughput.
- **Cost Optimization:** gateway endpoints, NAT per-AZ לפי צורך, cleanup של EIP/endpoints וניתוח data transfer.
- **Sustainability:** צמצם hops ו-NAT processing, השתמש ב-private service paths וב-resource sizing מתאים.

## 10. סיכום ובדיקת הבנה

VPC הוא גבול regional; subnet הוא AZ יחיד; routes מגדירים reachability ו-controls מגדירים authorization. Public דורש IGW route וכתובת ציבורית.

1. מה הופך subnet ל-public בפועל?
2. מדוע לתכנן CIDR לא חופף?
3. איזה רכיבים מייצרים את עיקר עלות ה-VPC?
