---
lesson: 12
title: VPC Private Connectivity
domain: Design Secure Architectures
services: [VPC Peering, VPC Endpoints, PrivateLink, Site-to-Site VPN, Direct Connect]
tags: [saa-c03, networking, private-connectivity]
---

# 12 — VPC Private Connectivity

> [!abstract] בשורה אחת
> חיבור פרטי בין VPCs, לשירותי AWS או לרשתות on-prem, בלי חשיפה לאינטרנט ציבורי.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | VPC Peering | איך מקשרים שתי VPCs ודוגמות מתי להשתמש |
| 2 | VPC Endpoints | Gateway vs Interface ואיך חוסכים NAT |
| 3 | PrivateLink | חשיפת service מפרטי VPC אחד לאחר |
| 4 | Site-to-Site VPN | VPN tunnels מ-on-prem ל-AWS |
| 5 | Direct Connect | חיבור ייעודי עם SLA וzמנים נמוכים |

**מונחי מפתח בשיעור:** `VPC Peering` · `Gateway Endpoint` · `Interface Endpoint` · `PrivateLink` · `VPN` · `Direct Connect`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- שתי VPCs צריכות לדבר — אבל יש לך חשש לגבי security ו-CIDR overlap.
- servers פרטיים בוקעים S3, DynamoDB או Secrets Manager דרך NAT Gateway (יקר ובקטל).
- פרטית on-prem רוצה להתחבר ל-AWS בלי ממשק ציבורי.
- third-party provider צריך לחשוף service בצורה מאובטחת למספר לקוחות AWS.

### מה הפתרונות פותרים

- **VPC Peering:** קישור ישיר ופרטי בין שתי VPCs עם routing ו-security groups שלהן.
- **Gateway Endpoint:** route table entry ל-S3/DynamoDB בלי NAT, לרוב חינם.
- **Interface Endpoint:** ENI פרטי (PrivateLink) לרוב שירותי AWS, עם SG ו-DNS.
- **Site-to-Site VPN:** encrypted tunnel מ-on-prem gateway ל-AWS VPN endpoint, כללי ויציב.
- **Direct Connect:** חיבור ייעודי (1 Gbps–100 Gbps) עם SLA, latency נמוך יותר.

---

## 2. ⚙️ איך זה עובד

### 2.1 VPC Peering

**הקשר:**
- ממלא route table entry בכל VPC עם CIDR של השני וציון peering connection.
- Security Groups עובדים across-VPC (reference SG ID של peered VPC בsame account + Region).
- **לא transitive:** אם VPC-A mated עם VPC-B ו-VPC-B עם VPC-C, A ו-C לא מדברות בלי peering ישיר.

**ניתן:**
- בין VPCs שונות בחשבון אחד או שונים.
- בין Regions שונים (CIDR יכולים להיות זהים).
- reference SG מ-peered VPC ב-same Region/Account בלבד.

**דוגמה:**
```text
VPC-A (10.0.0.0/16) ←→ Peering ←→ VPC-B (10.1.0.0/16)
Route: 10.1.0.0/16 via pcx-xxxxx (בRoute Table של VPC-A)
Route: 10.0.0.0/16 via pcx-xxxxx (בRoute Table של VPC-B)
```

### 2.2 Gateway Endpoint

**מנגנון:**
- AWS מוסיף entry בRoute Table: `0.0.0.0/0 via vpce-xxxxx`.
- EC2 בsubnet פרטי בוקע `s3.us-east-1.amazonaws.com` → private DNS מחזיר IP ב-VPC.
- traffic נעצר locally, לא עובר דרך IGW/NAT.
- **לרוב חינם** עבור S3 ו-DynamoDB.

**מגבלות:**
- רק S3 ו-DynamoDB זמינים כ-Gateway Endpoints.
- Endpoint policy מוגדר בקישור; עדיין נדרש IAM user policy + bucket policy.
- אינו משנה את ביטחון bucket; צריך layers auth נוספות.

### 2.3 Interface Endpoint

**מנגנון:**
- יוצר ENI פרטי בכל AZ/Subnet המחברת אותך לשירות דרך PrivateLink.
- Private DNS resolution: `ec2.amazonaws.com` → elastic IP פרטי ב-VPC.
- ניתן להחליף ל-public endpoint על ידי כיבוי private DNS.

**עלויות:**
- ~$7/חודש לכל endpoint.
- Per-GB data processing (~$0.01/GB).

**שירותים:**
- רוב AWS services: EC2, Lambda, SNS, SQS, Secrets Manager, Systems Manager וכו'.
- Third-party SaaS through Marketplace.

### 2.4 PrivateLink

**רעיון:**
- Provider מציג service דרך NLB בתוך VPC שלו.
- Consumer ב-VPC אחר יוצר Interface Endpoint שמצביע לservice.
- תעבורה fully private — אין צורך ב-peering/routing/CIDR מוסכם.

**דוגמה:**
```text
Provider VPC: NLB ← Backend Service
Consumer VPC: Interface Endpoint → NLB בחזקת provider (דרך AWS backbone)
```

### 2.5 Site-to-Site VPN

**Flow:**
1. Customer Gateway (on-prem router IP).
2. VPN Connection ב-AWS מציע שני tunnels ל-redundancy.
3. Encryption: IPSec, keys handled אוטומטית.
4. Route: static או dynamic (BGP).

**Latency:** ~50–100ms (רשת ציבורית), variable בעומס.

**Speed:** עד 1.25 Gbps per tunnel.

### 2.6 Direct Connect

**Flow:**
1. בקשה חיבור ייעודי בחשבון AWS (1-2 שבועות).
2. AWS מעניק port בדאטה סנטר קרוב אליך.
3. on-prem router מוגדר לחבור לפורט.
4. **מבודד ופיזי** — לא עובר דרך ISP כללי.

**מהירות:** 1 Gbps, 10 Gbps, 100 Gbps ports.

**Latency:** microseconds (dedicated line).

**Resilience:** יחיד port = SPOF. בדר"כ שתי פורטים או VPN כ-backup.

---

## 3. 🔍 פירוק מפורט

### VPC Peering Attributes

| פרמטר | ערך / הערה |
|---|---|
| Transitive | NO — חובה peering ישיר בין כל זוג |
| CIDR overlap | לא מותר |
| Cross-account | כן, requires accept/reject |
| Cross-region | כן, data transfer בתשלום |
| SG reference | כן ב-same Region/Account בלבד |
| Routing | manual route table updates בכל VPC |

### Endpoint Types Comparison

| מאפיין | Gateway | Interface |
|---|---|---|
| שירותים | S3, DynamoDB | רוב AWS + 3rd party |
| Placement | VPC-level, automatic | ENI, per AZ |
| Hourly cost | $0 (בד"כ) | ~$7/month |
| Data processing | $0 | ~$0.01/GB |
| Security Group | לא | כן |
| Private DNS | אוטומטי (S3/DDB) | כן, toggle-able |
| Policy | Endpoint policy | Endpoint policy + resource |

### VPN vs Direct Connect

| קריטריון | VPN | Direct Connect |
|---|---|---|
| Setup | דקות | שבועות |
| Latency | ~50–100ms | ~1–10ms |
| Bandwidth | 1.25 Gbps/tunnel | 1 Gbps–100 Gbps |
| Cost | נמוך | גבוה |
| Encryption | IPSec built-in | TLS/IPSec add-on |
| Resilience | dual tunnels | dual ports או VPN backup |
| Use case | ad-hoc, חסכוני | high-throughput, production |

---

## 4. 💰 עלות ותמחור

### על מה מחייבים

| רכיב | יחידה | הערה |
|---|---|---|
| VPC Peering | לא חייב | **חינם** in-Region; cross-region = ~$0.02/GB |
| Gateway Endpoint | לא חייב | בד"כ **חינם** (S3/DynamoDB) |
| Interface Endpoint | ENI/AZ | ~$7/month + ~$0.01/GB processing |
| VPN | connection hour | ~$0.05/hour + ~$0.09/GB egress |
| Direct Connect | port hour | ~$0.30–3/hour + initial lump sum |

### מה זול ומה יקר

| חלופה | עלות יחסית | מתי משתלם |
|---|---|---|
| VPC Peering in-Region | FREE | תמיד > NAT cross-VPC |
| Gateway Endpoint S3/DDB | FREE | תמיד > NAT |
| Interface Endpoint | $7–10/month | אם 1000+ GB/month > NAT (~$45) |
| VPN | $50–100/month + egress | temporary; ad-hoc |
| Direct Connect | $250+/month | production 24/7, high-throughput |

### 🚩 עלויות נסתרות

- **Cross-Region peering:** ~$0.02/GB (יקר למשימות גדולות).
- **Interface Endpoint per-AZ:** 3 AZs = $21/month base.
- **VPN egress:** תעבורה cross-boundary = "egress" charge.
- **NAT Gateway per-AZ:** hourly + per-GB; יכול להיות יקר 24/7.

### 💡 טיפים לחיסכון

- השתמש Gateway Endpoints **תמיד** עבור S3 ו-DynamoDB.
- In-Region VPC Peering זול יותר מ-NAT.
- Interface Endpoints: תוך השימוש ב-DNS caching בלקוח.
- Shared Services VPC + PrivateLink אחרת 100+ peering.
- DX backup ל-VPN זול יותר מdual DX ports.

---

## 5. ⚖️ השוואות מכריעות

### VPC Peering vs Endpoints vs Transit Gateway

| קריטריון | Peering | Endpoint | Transit GW |
|---|---|---|---|
| Use case | two VPCs | AWS service access | hub-and-spoke |
| CIDR overlap | NO | יכול | כן |
| Routing | manual | automatic | central |
| Cost | FREE (in-Region) | ~$7 | ~$32 |
| Scalability | difficult (10s) | ≤1000s | ≤100s |

> [!info] שורה תחתונה
> **2 VPCs:** Peering. **Many VPCs + AWS APIs:** Transit Gateway. **Private API access:** Endpoint.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | קל להתחזק ולצפות | IaC לכל connections; CloudWatch monitoring; failover testing |
| Security | private, encrypted, least-privilege | Endpoint & resource policies; VPN encryption; SG limited; NACLs whitelist |
| Reliability | high availability, redundancy | Dual VPN tunnels; Dual DX; Interface Endpoints בכמה AZs |
| Performance Efficiency | low latency, high throughput | DX למשימות גדולות; Interface Endpoint local DNS |
| Cost Optimization | הימנע מ-NAT overage | Gateway Endpoints חינם; Peering vs. NAT; consolidate (Transit GW) |
| Sustainability | צמצם compute overheads | Gateway Endpoint זול יותר מ-NAT (פחות processing) |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| Private servers pull S3 | Gateway Endpoint (free) — not NAT |
| Two VPCs need to talk | VPC Peering (not NAT) |
| On-prem needs AWS | VPN (quick) or DX (production) |
| Expose my service to other AWS accounts | PrivateLink |
| Low latency on-prem | Direct Connect (microseconds) |
| Highly available connectivity | Dual tunnels (VPN) or Dual DX + VPN backup |

### טעויות נפוצות

> [!warning] מלכודת
> **הניסוח:** "Endpoint is more secure than NAT."
> **הטעות:** Endpoint חוסך NAT אך עדיין דורש SG/policy auth.
> **הנכון:** Endpoint **פשוט יותר** (אין NAT processing) ו-**חינם** (S3/DDB); בחר לפי use case.

> [!warning] מלכודת
> **הניסוח:** "VPC Peering is transitive."
> **הטעות:** Peering **NOT transitive** — חובה ישיר peering בין כל זוג.
> **הנכון:** כל זוג צריך peering connection ישיר.

> [!warning] מלכודת
> **הניסוח:** "Endpoint makes S3 private."
> **הטעות:** Endpoint לא משנה bucket security.
> **הנכון:** Endpoint זה **routing only** — הרשאה ברמת resource.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**
- Shared services VPC (DNS, logging, egress proxy).
- 10 VPCs ארוחות צריכות access לשירותים.
- On-prem צריך להתחבר, עלות נמוכה, 200+ Mbps stable.
- Failover תוך דקות אם network disruption.

**הפתרון:**
- **Transit Gateway:** hub מרכזי, 10 VPCs קל בניהול.
- **Gateway Endpoint S3/DDB:** private subnets חוסכות NAT.
- **Interface Endpoint Secrets/Systems Manager:** private, no egress.
- **Direct Connect + VPN backup:** DX primary (200+ Mbps), VPN ב-minutes אם DX fails.

**למה לא:**
- **10 Peering:** scaling nightmare (45 connections).
- **NAT ל-S3/DDB:** cost multiplier; Gateway Endpoint חינם.
- **VPN בלבד:** 200+ Mbps יקר בegress; DX טוב יותר.
- **Single DX:** SPOF — offline שבועות אם cable fail; VPN backup essential.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- Direct Connect hosted connections topology.
- VLAN tagging, LAG configs.
- Transit Gateway Peering (multi-account advanced).
- PrivateLink protocol-level syntax.
- VPN IKE/DPD tuning (AWS managed).

---

## 10. ⚡ Cheat Sheet

- **VPC Peering:** direct, in-Region free, cross-Region charged, NOT transitive, manual routing.
- **Gateway Endpoint:** S3 + DynamoDB, free, automatic routing.
- **Interface Endpoint:** all AWS APIs, ~$7/month + $0.01/GB.
- **PrivateLink:** provider ← NLB; consumer ← Interface Endpoint; fully private.
- **VPN:** encrypted tunnels, dual HA, ~$50–100/month + egress.
- **Direct Connect:** dedicated line, 1Gbps–100Gbps, $250+/month, microsecond latency.
- **Failover:** VPN backup ל-DX; dual DX ל-HA.
- **S3/DDB:** always Gateway Endpoint.
- **Many VPCs:** Transit Gateway hub-and-spoke vs. Peering full-mesh.

---

## 11. ✅ בדיקת הבנה

1. **Two VPCs in same Region with non-overlapping CIDRs. Simplest and cheapest solution?**
   - A) VPC Peering (free in-Region) ✓
   - B) Transit Gateway ($32/month)
   - C) NAT + IGW
   - D) VPN

2. **Private EC2 servers pull large S3 objects hourly. Best cost savings?**
   - A) Interface Endpoint
   - B) Gateway Endpoint (free, no data charge) ✓
   - C) NAT Gateway
   - D) None

3. **On-prem 500 Mbps dedicated year-round, SLA required. VPN $50/month; DX $500/month. Which cheaper?**
   - A) VPN
   - B) Direct Connect (egress savings dominate) ✓
   - C) Equal cost
   - D) Depends

<details>
<summary>תשובות</summary>

1. **A) VPC Peering.** Free in-Region, CIDR check automatic, manual route table updates. Transit Gateway overkill for 2 VPCs.

2. **B) Gateway Endpoint.** S3 + DynamoDB free, no hourly/processing fee. Interface ~$7 + $0.01/GB. For high S3 traffic, Gateway saves dramatically.

3. **B) Direct Connect.** 500 Mbps × 24/7 × ~$0.09/GB egress ≈ high cost on VPN. DX $500 flat ≈ 1–2 TB threshold → DX wins.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[09 - VPC Fundamentals]] · [[11 - VPC Security]] · [[13 - VPC Network Architecture]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 13495–14166
