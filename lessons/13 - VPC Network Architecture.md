---
lesson: 13
title: VPC Network Architecture
domain: Design Cost-Optimized Architectures
services: [VPC Peering, Transit Gateway, AWS PrivateLink, Site-to-Site VPN, Direct Connect, Traffic Mirroring, VPC Endpoints]
tags: [saa-c03, networking, architecture, cost]
---

# 13 — VPC Network Architecture

> [!abstract] בשורה אחת
> זה השיעור שבו מחברים את כל רכיבי הרשת לתמונה אחת: איזו טופולוגיה בוחרים לחיבור בין רשתות, ואיפה בדיוק זורם הכסף על כל GB.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | הבעיה והפתרון | למה full-mesh peering קורס אחרי כמה VPCs |
| 2 | איך זה עובד | Transit Gateway כ-hub, route tables, ECMP, שיתוף cross-account |
| 3 | פירוק מפורט | טבלת כל אפשרויות החיבור + Traffic Mirroring + סיכום רכיבי VPC |
| 4 | עלות | טבלת עלויות רשת per GB — הסעיף הכי נבחן בשיעור הזה |
| 5 | השוואות | Peering מול TGW מול PrivateLink מול VPN/DX + decision tree |
| 6 | Well-Architected | איך נראית רשת טובה בששת ה-pillars |
| 7 | מלכודות | transitive routing, CIDR חופף, NAT מול Gateway Endpoint |
| 8 | Scenario | 20 חשבונות + data center — איך בונים את זה נכון |
| 9-11 | דילוגים, Cheat Sheet, בדיקת הבנה | |

**מונחי מפתח בשיעור:** `Transit Gateway` · `Transitive Routing` · `ECMP` · `RAM` · `Data Transfer Out` · `Gateway Endpoint` · `Traffic Mirroring`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- ארגון מתחיל עם 2–3 VPCs. VPC Peering פותר את זה יפה.
- אחרי שנה יש 20 VPCs, 4 חשבונות, data center אחד ו-2 Regions.
- Peering הוא **קשר one-to-one** ו**לא transitive** — כל זוג VPCs צריך חיבור משלו.
- מספר החיבורים גדל ריבועית: `n × (n-1) / 2`.
- כל חיבור מחייב עריכת route tables **בשני הצדדים** — התפעול הופך לבלתי אפשרי.

### מה השירות פותר

- **Transit Gateway (TGW)** הופך את ה-mesh ל-**hub-and-spoke**: כל VPC מתחבר פעם אחת ל-hub.
- מספר החיבורים יורד מ-`n(n-1)/2` ל-`n`.
- TGW הוא **transitive** — VPC A מדבר עם VPC C דרך ה-hub, בלי חיבור ישיר.
- אותו hub מחבר גם **Site-to-Site VPN** וגם **Direct Connect Gateway** — hybrid ו-cloud באותו מקום.

### הבעיה השנייה: עלות

- ברשת של AWS כמעט כל בייט שחוצה גבול (AZ / Region / אינטרנט) עולה כסף.
- ארכיטקטורה "נכונה" פונקציונלית יכולה להיות יקרה פי 10 מארכיטקטורה זהה עם החלטות topology אחרות.
- לכן הסעיף על עלויות כאן הוא לא נספח — הוא לב השיעור.

> [!tip] האנלוגיה
> Peering זה כמו לחפור מנהרה פרטית בין כל שני בתים בשכונה.
> Transit Gateway זה כביש מרכזי אחד שכל בית מתחבר אליו — וכל אחד יכול להגיע לכל אחד.

---

## 2. ⚙️ איך זה עובד

### 2.1 מדוע Peering לא מתרחב

```text
Full-mesh peering עם 4 VPCs = 6 חיבורים
   A ─── B
   │ \ / │
   │  X  │
   │ / \ │
   C ─── D

Hub-and-spoke עם TGW ו-4 VPCs = 4 attachments
   A       B
    \     /
     [ TGW ]
    /     \
   C       D
```

| מספר VPCs | חיבורי Peering (full-mesh) | Attachments ב-TGW |
|---|---|---|
| 3 | 3 | 3 |
| 5 | 10 | 5 |
| 10 | 45 | 10 |
| 20 | 190 | 20 |
| 50 | 1,225 | 50 |

**המסקנה למבחן:** ברגע שהשאלה מזכירה "עשרות VPCs" או "מספר חשבונות" — התשובה היא Transit Gateway.

### 2.2 Transit Gateway — המאפיינים שחייבים לזכור

- **Regional resource** — TGW חי ב-Region אחד, אבל אפשר **לעשות peering בין TGWs בין Regions**.
- **Transitive peering** — זה בדיוק מה ש-VPC Peering לא נותן.
- **מתחבר ל:** VPCs, Site-to-Site VPN, Direct Connect Gateway.
- **Route Tables משלו** — מנגנון ה-segmentation. אפשר להגדיר ש-Prod לא יראה את Dev בכלל.
- **שיתוף cross-account דרך AWS RAM** (Resource Access Manager) — TGW אחד ב-account מרכזי, כל שאר החשבונות מחברים אליו attachments.
- **תומך ב-IP Multicast** — השירות היחיד ב-AWS שתומך בזה. זו עובדת trivia שנשאלת.

```text
              Corporate Data Center
                       │
              ┌────────┴────────┐
              │  VPN   /   DX   │
              └────────┬────────┘
                       │
                  [ Transit Gateway ]  ← RAM share
                  /    │     │    \
              VPC-A  VPC-B VPC-C  VPC-D
             (acct1)(acct1)(acct2)(acct3)

     TGW Route Table "Prod"  → רואה רק VPC-A, VPC-B, DC
     TGW Route Table "Dev"   → רואה רק VPC-C, VPC-D
```

### 2.3 ECMP — הגדלת רוחב פס של VPN

- **ECMP = Equal-Cost Multi-Path routing.**
- רעיון: לשלוח packets על **כמה מסלולים טובים במקביל** במקום לבחור אחד.
- **VPN אל Virtual Private Gateway (VGW):** לא תומך ב-ECMP → תקרת רוחב פס של חיבור VPN יחיד (~1.25 Gbps).
- **VPN אל Transit Gateway:** תומך ב-ECMP → כל חיבור VPN נוסף **מוסיף** רוחב פס.

| מספר חיבורי VPN | אל VGW | אל TGW (ECMP) |
|---|---|---|
| 1 | ~1.25 Gbps | ~2.5 Gbps (2 tunnels) |
| 2 | ~1.25 Gbps (אין צבירה) | ~5 Gbps |
| 3 | ~1.25 Gbps | ~7.5 Gbps |

> [!info] מילת המפתח במבחן
> "צריך יותר bandwidth ב-Site-to-Site VPN" → **מספר VPN connections אל Transit Gateway עם ECMP**.
> זו התשובה, לא "לשדרג את ה-VPN" (אין דבר כזה) ולא Direct Connect (זה תשובה אחרת — latency יציב).

### 2.4 שיתוף Direct Connect בין חשבונות

- Direct Connect יקר ובעל lead time ארוך — לא רוצים אחד לכל חשבון.
- הפתרון: **DX אחד → Transit VIF → Direct Connect Gateway → Transit Gateway → כל החשבונות**.
- שיתוף ה-TGW בין החשבונות נעשה שוב דרך **AWS RAM**.

```text
DC → Customer Router → DX Location → Transit VIF
   → Direct Connect Gateway → Transit Gateway (Account 1)
        ├── VPC Account 1
        └── VPC Account 2   (RAM share)
```

### 2.5 VPC Traffic Mirroring

- מעתיק תעבורת רשת אמיתית מתוך ה-VPC לצורך ניתוח — בלי agent על ה-instance.
- **Source:** ENIs.
- **Target:** ENI בודד או **Network Load Balancer** (שמפזר ל-ASG של security appliances).
- אפשר לתפוס **הכל** או לסנן לפי filters, ואפשר **לקצר (truncate)** packets.
- Source ו-Target יכולים להיות ב-VPC שונה — דרך VPC Peering.
- **Use cases:** content inspection, threat monitoring, troubleshooting.

> [!info] Traffic Mirroring מול VPC Flow Logs
> Flow Logs = **metadata** בלבד (מי דיבר עם מי, accept/reject). לא רואים תוכן.
> Traffic Mirroring = **ה-packets עצמם**. אם השאלה מבקשת "לבדוק את התוכן" → Mirroring.

---

## 3. 🔍 פירוק מפורט

### 3.1 סיכום רכיבי ה-VPC — מפת הדרכים המלאה

| רכיב | תפקיד בשורה אחת | נקודה שנשכחת |
|---|---|---|
| CIDR | טווח כתובות IP | לא ניתן לשנות; תכנן גדול מראש |
| VPC | הרשת הווירטואלית | מגדירים CIDR של IPv4 ו-IPv6 |
| Subnet | קשור ל-**AZ אחד** | AZ אחד בלבד — לא נמתח בין AZs |
| Internet Gateway | גישה לאינטרנט ברמת ה-VPC | IPv4 **וגם** IPv6 |
| Route Tables | מה שבאמת מחבר בין הרכיבים | כמעט כל תקלת רשת = route חסר |
| Bastion Host | EC2 ציבורי ל-SSH פנימה | חלופה מודרנית: SSM Session Manager |
| NAT Instance | ישן; דורש כיבוי Source/Dest check | מופיע בשאלות ישנות בלבד |
| NAT Gateway | גישה יוצאת מנוהלת ל-**IPv4** בלבד | AZ-scoped; אחד לכל AZ ל-HA |
| Egress-Only IGW | כמו NAT Gateway אבל ל-**IPv6** | ל-IPv6 אין NAT — זה המקבילה |
| NACL | **stateless**, ברמת ה-subnet | לא לשכוח Ephemeral Ports |
| Security Group | **stateful**, ברמת ה-ENI/instance | אפשר להפנות ל-SG אחר |
| VPC Peering | חיבור 1:1, **לא transitive** | CIDRs לא יכולים לחפוף |
| VPC Endpoints | גישה פרטית לשירותי AWS | Gateway (S3/DDB) מול Interface |
| VPC Flow Logs | metadata של תעבורה | VPC / Subnet / ENI; ניתוח ב-Athena |
| Site-to-Site VPN | CGW בצד שלך + VGW בצד AWS | מעל האינטרנט הציבורי, מוצפן |
| VPN CloudHub | hub-and-spoke של VPNs בין אתרים | מודל זול לחיבור סניפים |
| Direct Connect | חיבור פרטי פיזי דרך DX Location | **לא מוצפן** by default |
| Direct Connect Gateway | DX אחד → VPCs בכמה Regions | לא מחליף TGW, משלים אותו |
| PrivateLink | חשיפת שירות פרטית בין VPCs | דורש **NLB** בצד השירות |
| Transit Gateway | hub transitive לכל הרשת | Route Tables = segmentation |
| Traffic Mirroring | העתקת packets לניתוח | Target: ENI או NLB |

### 3.2 Direct Connect — רמות עמידות

| רמת עמידות | מה זה | מתי |
|---|---|---|
| Development | חיבור יחיד, DX Location יחיד | לא production |
| High Resiliency | שני חיבורים, **שני DX Locations** | workload קריטי |
| Maximum Resiliency | שני חיבורים בכל אחד משני Locations, על devices נפרדים | קריטיות מקסימלית |
| DX + VPN backup | DX ראשי, Site-to-Site VPN כגיבוי | הדרך **הזולה** לגיבוי |

> [!tip] המלכודת הקלאסית
> "DX נפל, מה הגיבוי הזול ביותר?" → **Site-to-Site VPN**, לא DX שני.
> "מה הגיבוי בעל הביצועים הטובים ביותר?" → **DX שני ב-Location אחר**.

### 3.3 מה בוחרים לפי מספר הצדדים

| תרחיש | הפתרון |
|---|---|
| 2 VPCs, תעבורה ישירה | VPC Peering |
| 3–5 VPCs, יציב, לא צומח | Peering (עדיין סביר) |
| 10+ VPCs או ריבוי accounts | **Transit Gateway** |
| חשיפת שירות אחד ללקוחות רבים | **PrivateLink** (לא peering — לא רוצים חיבור רשת מלא) |
| VPC ↔ data center, מהר וזול | Site-to-Site VPN |
| VPC ↔ data center, bandwidth/latency יציבים | Direct Connect |
| DX לכמה Regions | Direct Connect Gateway |
| הכול ביחד | TGW כ-hub + DX Gateway + VPN backup |

---

## 4. 💰 עלות ותמחור — על מה בדיוק משלמים

זה הסעיף שנשאל ישירות במבחן. הכלל הבסיסי: **ingress (פנימה ל-AWS) בדרך כלל חינם, egress עולה.**

### 4.1 טבלת עלויות רשת per GB

המספרים כאן הם **יחסים** — נניח שעלות cross-AZ = `1×`.

| מסלול התעבורה | עלות יחסית per GB | הערה |
|---|---|---|
| בתוך אותו AZ, דרך **Private IP** | **0** | הכי זול שיש |
| בתוך אותו AZ, דרך **Public/Elastic IP** | `~2×` | אותה תעבורה, IP אחר — משלמים |
| בין AZs באותו Region (Private IP) | `1×` | בשני הכיוונים |
| בין Regions | `~2×` | |
| מ-AWS לאינטרנט (egress) | `~9×` | היקר ביותר, בפער |
| מהאינטרנט ל-AWS (ingress) | **0** | תמיד חינם |
| דרך **NAT Gateway** | שעת NAT + `~4.5×` על כל GB **בנוסף** לעלות היעד | חיוב כפול |
| דרך **Gateway VPC Endpoint** (S3/DynamoDB) | **0** | אין עלות לשימוש ב-endpoint |
| S3 → אינטרנט | `~9×` | |
| S3 → CloudFront | **0** | origin fetch חינם |
| CloudFront → אינטרנט | `~8.5×` | מעט זול מ-S3 ישירות + caching |
| S3 Cross-Region Replication | `~2×` | |
| S3 Transfer Acceleration | תוספת `~4×`–`~8×` **מעל** עלות ה-transfer | תמורת 50%–500% שיפור מהירות |

> [!warning] הטעות הכי נפוצה
> "אותו Region = חינם" — **לא נכון**. cross-AZ עולה בשני הכיוונים.
> חינם זה **אותו AZ עם Private IP** בלבד.

### 4.2 NAT Gateway מול Gateway Endpoint — הדוגמה הקלאסית

תרחיש: EC2 ב-private subnet צריך לגשת ל-S3 באותו Region.

```text
דרך NAT Gateway:
  EC2 (private) → NAT GW (public subnet) → IGW → S3
  משלמים: NAT hourly + NAT data processed per GB
  (העברת הנתונים ל-S3 באותו Region עצמה = 0)

דרך Gateway VPC Endpoint:
  EC2 (private) → prefix list route → VPC Endpoint → S3
  משלמים: 0 על ה-endpoint
```

| קריטריון | NAT Gateway | Gateway VPC Endpoint |
|---|---|---|
| עלות שעתית | יש | **אין** |
| עלות per GB | יש (משמעותית) | **אין** |
| למה מיועד | כל יעד באינטרנט | **רק S3 ו-DynamoDB** |
| האם התעבורה יוצאת מרשת AWS | כן | **לא** |
| Route table | `0.0.0.0/0 → nat-id` | `pl-xxxx → vpce-id` |
| High Availability | אחד לכל AZ | מובנה |

**המסקנה:** בכל פעם ש-instance פרטי ניגש ל-S3 או DynamoDB — Gateway Endpoint. אין סיבה לשלם NAT על זה.

### 4.3 מזעור egress — עקרון העיצוב

- **תמיד עדיף להזיז את ה-compute אל הנתונים** מאשר את הנתונים אל ה-compute.
- דוגמה: query ששולח 100 MB ומחזיר 50 KB.
  - אפליקציה ב-on-prem + DB ב-AWS → 100 MB יוצאים מ-AWS? לא — אבל התוצאות והתעבורה ההפוכה כן.
  - אפליקציה ב-AWS ליד ה-DB → רק **50 KB** חוצים את הגבול.
- **DX Location שנמצא co-located באותו Region** נותן תעריף egress נמוך יותר.

### 4.4 🚩 עלויות נסתרות

- **NAT Gateway data processed** — מחייבים על כל GB שעובר, **בנוסף** לעלות היעד.
- **Public IP במקום Private IP** — אותה תעבורה בדיוק, אבל cross-AZ נחשב ומחויב.
- **Transit Gateway per-attachment hourly** — כל attachment הוא חיוב שעתי קבוע, גם אם אין תעבורה.
- **Transit Gateway data processing per GB** — נוסף על עלות ה-cross-AZ/cross-Region.
- **תעבורה שעוברת דרך TGW פעמיים** — VPC → TGW → VPC משלם processing פעם אחת, אבל בתכנון גרוע (service chaining דרך inspection VPC) אפשר לשלם 2–3 פעמים.
- **Elastic IP לא מחובר** — מחויב שעתית כשהוא idle.
- **Interface Endpoints (PrivateLink)** — חיוב **שעתי לכל ENI בכל AZ** + per GB. עם 3 AZs ו-10 שירותים זה מצטבר.

### 4.5 💡 טיפים לחיסכון

- השתמש ב-**Private IP** בכל תקשורת פנימית — לא ב-Public/Elastic IP.
- **Gateway Endpoints ל-S3 ו-DynamoDB** — חינם, מבטל NAT לחלוטין עבור הגישה הזו.
- שים **CloudFront מול S3** — origin fetch חינם, ה-egress זול יותר, וה-cache מקטין גם requests.
- **אותו AZ** לתעבורה כבדה (למשל בין app ל-cache) — אבל **על חשבון HA**. זה trade-off מודע.
- **קונסולידציה ב-TGW** במקום עשרות peering — חוסך תפעול, אך שים לב ש-peering עצמו **לא** גובה per-attachment.
- אם יש **שני VPCs בלבד** עם הרבה תעבורה — **VPC Peering זול יותר מ-TGW**, כי אין hourly ואין processing fee.
- הימנע מ-**Transfer Acceleration** אלא אם באמת נדרשת מהירות — הוא תוספת מעל התעריף הרגיל.

---

## 5. ⚖️ השוואות מכריעות

### 5.1 טופולוגיות — Full-Mesh Peering מול Hub-and-Spoke TGW

| קריטריון | VPC Peering (full-mesh) | Transit Gateway (hub-and-spoke) |
|---|---|---|
| מספר חיבורים ל-n רשתות | `n(n-1)/2` | `n` |
| Transitive routing | **לא** | **כן** |
| חיוב שעתי | אין | יש, לכל attachment |
| חיוב per GB | רק data transfer רגיל | data transfer + **TGW processing** |
| Cross-Region | כן | כן (TGW peering) |
| Cross-Account | כן | כן, דרך **RAM** |
| חיבור ל-VPN / DX | לא | **כן** |
| Segmentation | דרך route tables בכל VPC | **TGW route tables** מרכזיים |
| הפניה ל-SG של הצד השני | **כן** (same-Region) | לא |
| מתי עדיף | 2–5 VPCs, תעבורה כבדה, יציב | 10+ רשתות, hybrid, ריבוי חשבונות |

### 5.2 ארבע דרכי החיבור

| קריטריון | Peering | Transit Gateway | PrivateLink | VPN / Direct Connect |
|---|---|---|---|---|
| מה מחבר | VPC ↔ VPC | הכול ↔ הכול | **צרכן ↔ שירות בודד** | AWS ↔ on-premises |
| היקף החשיפה | רשת מלאה | רשת מלאה (לפי routes) | **endpoint אחד בלבד** | רשת מלאה |
| CIDR חופף מותר | **לא** | לא | **כן** | לא |
| Transitive | לא | **כן** | לא רלוונטי | דרך TGW כן |
| דורש NLB | לא | לא | **כן** | לא |
| מדרגיות | נמוכה | גבוהה מאוד | גבוהה מאוד (אלפי צרכנים) | בינונית |
| מוצפן | תעבורה ברשת AWS | תעבורה ברשת AWS | תעבורה ברשת AWS | VPN: כן / DX: **לא** |

### 5.3 Decision Tree — איזה חיבור לבחור

```text
צריך לחבר רשתות?
│
├─ אחד הצדדים הוא on-premises / data center?
│   ├─ צריך bandwidth ו-latency עקביים, ומוכן ל-lead time?
│   │      → Direct Connect
│   │        (צריך גם הצפנה? → DX + VPN מעליו)
│   │        (צריך גישה לכמה Regions? → Direct Connect Gateway)
│   └─ צריך מהר, זול, מוצפן מהקופסה?
│          → Site-to-Site VPN
│            (צריך יותר bandwidth? → כמה VPNs אל TGW עם ECMP)
│            (הרבה סניפים ביניהם? → VPN CloudHub)
│
└─ שני הצדדים ב-AWS?
    ├─ רוצה לחשוף שירות אחד בלבד, לצרכנים רבים, אולי עם CIDR חופף?
    │      → AWS PrivateLink (VPC Endpoint Service + NLB)
    │
    ├─ יותר מ-5 רשתות, או ריבוי accounts, או גם hybrid?
    │      → Transit Gateway  (+ RAM לשיתוף)
    │
    └─ 2–5 VPCs, CIDRs שונים, תעבורה כבדה, לא צומח?
           → VPC Peering
```

> [!info] שורה תחתונה
> Peering לפשטות וזול בקטן · TGW למדרגיות ו-hybrid · PrivateLink כשרוצים לחשוף שירות ולא רשת · VPN/DX לחיבור החוצה.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בארכיטקטורת רשת** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | טופולוגיה שאפשר לתחזק ולהבין | TGW מרכזי מנוהל ב-IaC; route tables בקוד; ניטור BGP ו-tunnel state ב-CloudWatch |
| Security | הפרדה בין סביבות ובקרת מה נחשף | TGW route tables נפרדים ל-Prod/Dev; PrivateLink במקום peering לחשיפת שירות; Traffic Mirroring ל-inspection |
| Reliability | אף רכיב רשת אינו נקודת כשל | **שתי** VPN tunnels; DX ב-2 Locations או DX+VPN backup; NAT Gateway בכל AZ |
| Performance Efficiency | פחות hops, יותר רוחב פס | DX ל-latency יציב; ECMP להגדלת VPN; Gateway Endpoint במקום מסלול NAT→IGW |
| Cost Optimization | לצמצם GB שחוצים גבול | Private IP בלבד; Gateway Endpoints ל-S3/DDB; CloudFront מול S3; ביטול attachments שלא בשימוש |
| Sustainability | פחות ציוד ופחות תעבורה מיותרת | קונסולידציה של עשרות peering ל-TGW אחד; DX בגודל נכון; כיבוי סביבות dev בלילה |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| "hundreds of VPCs" / "multiple accounts" | Transit Gateway + RAM |
| "transitive" | Transit Gateway (Peering אינו transitive) |
| "increase VPN bandwidth" | כמה VPN connections אל TGW עם **ECMP** |
| "consistent / dedicated bandwidth", "no internet" | Direct Connect |
| "DX backup, lowest cost" | Site-to-Site VPN |
| "DX must be encrypted" | Site-to-Site VPN **על גבי** DX |
| "expose our SaaS to customer VPCs" | **PrivateLink** |
| "overlapping CIDRs" | PrivateLink (peering/TGW פסולים) |
| "inspect packet contents" | **VPC Traffic Mirroring** |
| "who talked to whom / rejected traffic" | VPC Flow Logs |
| "reduce data transfer cost to S3 from private subnet" | **Gateway VPC Endpoint** |
| "minimize cost of NAT" | Gateway Endpoint / VPC Endpoint |
| "IPv6 outbound only" | **Egress-Only Internet Gateway** |
| "multicast" | Transit Gateway (היחיד שתומך) |
| "DX to VPCs in several Regions" | Direct Connect Gateway |

### טעויות נפוצות

> [!warning] מלכודת 1 — Transitive Peering
> **הניסוח:** "VPC A מחובר ל-B, ו-B מחובר ל-C. האם A מגיע ל-C?"
> **הטעות:** להניח שכן — הרי יש מסלול.
> **הנכון:** **לא.** VPC Peering אינו transitive. צריך חיבור A↔C ישיר, או Transit Gateway.

> [!warning] מלכודת 2 — "אותו Region זה חינם"
> **הניסוח:** "האפליקציה ב-AZ-a והמסד ב-AZ-b, אותו Region. כמה עולה התעבורה?"
> **הטעות:** לחשוב שאין חיוב כי זה אותו Region.
> **הנכון:** cross-AZ **מחויב per GB בשני הכיוונים**. חינם זה רק אותו AZ עם Private IP.

> [!warning] מלכודת 3 — Public IP בתוך ה-VPC
> **הניסוח:** שתי instances באותו AZ מדברות דרך Elastic IP.
> **הטעות:** להניח שזה אותו דבר כמו Private IP.
> **הנכון:** שימוש ב-Public/Elastic IP **מחייב** per GB (ופוגע ב-latency). תמיד Private IP פנימית.

> [!warning] מלכודת 4 — NAT Gateway ל-S3
> **הניסוח:** "instances ב-private subnet כותבות טרהבייטים ל-S3, החשבון מטפס."
> **הטעות:** לשדרג את ה-NAT Gateway או להוסיף עוד.
> **הנכון:** **Gateway VPC Endpoint ל-S3** — עלות אפס, והתעבורה לא עוזבת את רשת AWS.

> [!warning] מלכודת 5 — Direct Connect ו-הצפנה
> **הניסוח:** "רגולציה דורשת encryption in transit ל-on-premises."
> **הטעות:** לענות "DX — זה חיבור פרטי".
> **הנכון:** DX **אינו מוצפן** by default. מריצים **Site-to-Site VPN מעל ה-DX**.

> [!warning] מלכודת 6 — NAT Gateway ל-IPv6
> **הניסוח:** instances עם IPv6 בלבד צריכות גישה יוצאת.
> **הטעות:** NAT Gateway.
> **הנכון:** **Egress-Only Internet Gateway**. NAT Gateway עובד רק על IPv4.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**
חברה עם **20 AWS accounts**, סה"כ 35 VPCs בשני Regions, ו-data center אחד.
דרישות: כל VPC של Prod צריך גישה ל-data center; Dev מבודד מ-Prod;
תעבורה ל-S3 חייבת להיות זולה; חייבת עמידות לנפילת קו; רגולציה דורשת הצפנה ל-on-prem.

```text
        Corporate Data Center
        ┌──────────┬──────────┐
        │   DX     │   VPN    │  ← VPN מוצפן מעל DX + VPN גיבוי
        └────┬─────┴────┬─────┘
             │          │
      Direct Connect Gateway
             │
   ┌─────────┴──────────┐            ┌────────────────────┐
   │  TGW  (Region A)   │◄──peering──►│  TGW  (Region B)   │
   └─┬────┬────┬────┬───┘            └───┬──────┬─────────┘
     │    │    │    │                    │      │
   Prod Prod  Dev  Dev                 Prod    Dev
   VPC  VPC   VPC  VPC                 VPC     VPC
    │
    └── Gateway Endpoint → S3   (בכל VPC, עלות 0)

   TGW Route Table "Prod" → Prod VPCs + DX
   TGW Route Table "Dev"  → Dev VPCs בלבד
```

**הפתרון וההנמקה:**

| החלטה | למה |
|---|---|
| Transit Gateway בכל Region | 35 VPCs = 595 peering connections ב-full-mesh. בלתי אפשרי לתחזק |
| שיתוף ה-TGW דרך **AWS RAM** | TGW אחד בחשבון networking מרכזי, 20 חשבונות מחברים attachments |
| **TGW peering** בין ה-Regions | חיבור בין שני ה-hubs במקום peering בין כל VPC |
| **שתי TGW route tables** (Prod / Dev) | הפרדת סביבות ברמת ה-routing — Dev פשוט לא רואה את Prod |
| **Direct Connect Gateway** | DX פיזי אחד משרת VPCs בשני Regions |
| **Site-to-Site VPN מעל ה-DX** | DX לא מוצפן; הרגולציה דורשת encryption in transit |
| **VPN נפרד כ-backup** ל-DX | הגיבוי הזול; DX שני היה עולה הרבה יותר |
| **Gateway Endpoint ל-S3** בכל VPC | מבטל את עלות ה-NAT per GB לגישה ל-S3 |
| Private IP בכל תקשורת פנימית | מונע חיוב מיותר על תעבורה בתוך אותו AZ |

**למה לא VPC Peering?**
595 חיבורים, כל אחד עם עדכוני route tables בשני הצדדים, ואין דרך לחבר את ה-data center דרכם.

**למה לא PrivateLink?**
PrivateLink חושף **שירות בודד**, לא מחבר רשתות. הדרישה כאן היא קישוריות רשתית מלאה בין Prod ל-DC.

**למה לא VPN בלבד בלי DX?**
אפשרי, וזול יותר — אבל אין ערובה ל-latency ול-bandwidth כי זה רץ מעל האינטרנט הציבורי.
אם השאלה הייתה מדגישה "עלות מינימלית" ולא "ביצועים עקביים" — VPN עם ECMP היה מנצח.

---

## 9. 🚫 מה לא צריך לדעת למבחן

- **תצורת BGP מפורטת** — ASN, AS-path prepending, communities. לא נבחן.
- **פרטי IPsec** — אלגוריתמים, phase 1/2, DH groups.
- **ClassicLink** — טכנולוגיה שהוסרה (EC2-Classic כבר לא קיים). מופיע בחומר ישן בלבד.
- **NAT Instance** — צריך לדעת שהוא הישן והידני מול NAT Gateway המנוהל. לא צריך את שלבי ההתקנה.
- **מספרי מחיר מדויקים בדולרים** — משתנים לפי Region. צריך לזכור את **היחסים** ואת מה שחינם.
- **קונפיגורציית Traffic Mirroring filters** — צריך לדעת מה השירות עושה ומתי, לא איך מגדירים.
- **מגבלות מדויקות של TGW** (מספר attachments מקסימלי וכו') — נדיר.

---

## 10. ⚡ Cheat Sheet — סיכום מהיר

- **VPC Peering לא transitive.** נקודה. CIDRs לא יכולים לחפוף.
- Peering full-mesh = `n(n-1)/2` חיבורים; TGW = `n` attachments.
- **Transit Gateway:** regional, transitive, מחבר VPC + VPN + DX, משותף דרך **RAM**, ה**יחיד** שתומך ב-**Multicast**.
- **TGW Route Tables** = הכלי ל-segmentation בין Prod ל-Dev.
- **ECMP עובד רק אל TGW**, לא אל VGW. עוד VPN = עוד bandwidth.
- **Direct Connect לא מוצפן** — צריך הצפנה? VPN מעל DX.
- גיבוי זול ל-DX = **Site-to-Site VPN**.
- **Ingress חינם. Egress לאינטרנט הוא היקר ביותר** (~9× מ-cross-AZ).
- **חינם = אותו AZ + Private IP.** cross-AZ מחויב, גם באותו Region.
- **Public/Elastic IP בתוך ה-VPC = משלמים.** תמיד Private IP.
- **Gateway Endpoint (S3/DynamoDB) = עלות 0**, ומחליף לגמרי NAT לגישה אליהם.
- **S3 → CloudFront = 0**, ו-CloudFront → אינטרנט זול מ-S3 → אינטרנט.
- **NAT Gateway = IPv4 בלבד.** ל-IPv6 יוצא: **Egress-Only IGW**.
- **Traffic Mirroring = packets** (Target: ENI או NLB). **Flow Logs = metadata**.
- CIDR חופף + צריך לחבר? → רק **PrivateLink**.

---

## 11. ✅ בדיקת הבנה

1. יש לך 12 VPCs שצריכים לדבר כולם עם כולם. כמה חיבורי peering נדרשים, ומה החלופה?
2. אפליקציה ב-private subnet מעלה 50 TB לחודש ל-S3 באותו Region דרך NAT Gateway. איך מקטינים את החשבון כמעט לאפס?
3. הצוות מדווח שחיבור ה-VPN ל-AWS "תקוע על 1.25 Gbps". מה הפתרון?
4. הרגולטור דורש encryption in transit בין ה-data center ל-AWS. יש Direct Connect. מספיק?
5. VPC A מחובר ב-peering ל-B, ו-B מחובר ל-C. איך מאפשרים ל-A לדבר עם C?
6. שני מיקרו-שירותים באותו Region אבל ב-AZs שונים מעבירים 10 TB ביניהם ביום. מה העלות ואיך אפשר לחסוך — ומה המחיר של החיסכון?

<details>
<summary>תשובות</summary>

1. `12 × 11 / 2 = 66` חיבורים, וכל אחד דורש עדכון route tables בשני הצדדים. החלופה: **Transit Gateway** — 12 attachments בלבד, transitive, ועם route tables מרכזיים ל-segmentation.

2. **Gateway VPC Endpoint ל-S3.** אין עלות שעתית ואין עלות per GB על ה-endpoint, והתעבורה לא יוצאת מרשת AWS. זה מבטל גם את ה-NAT data processing וגם את הצורך ב-NAT עבור המסלול הזה. מוסיפים route ל-prefix list של S3 בטבלת ה-routing של ה-private subnet.

3. חיבור VPN אל **Virtual Private Gateway** לא תומך ב-ECMP, ולכן חסום סביב 1.25 Gbps. מעבירים את ה-VPN אל **Transit Gateway** ופותחים **כמה חיבורי VPN עם ECMP** — כל חיבור מוסיף בערך 2.5 Gbps.

4. **לא.** Direct Connect הוא חיבור פרטי אבל **אינו מוצפן** by default. מקימים **Site-to-Site VPN מעל ה-DX** (public VIF) כדי לקבל IPsec.

5. אין transitive routing ב-peering. או להקים **חיבור peering ישיר A↔C** (ולעדכן route tables), או — אם צפויות עוד רשתות — לעבור ל-**Transit Gateway**.

6. cross-AZ מחויב per GB **בשני הכיוונים** (ביחסים שלנו `1×` לכל GB). אפשר לחסוך לגמרי על ידי הצבת שני השירותים **באותו AZ** עם Private IP — אבל **המחיר הוא זמינות**: נפילת AZ בודד מפילה את שניהם. זה trade-off מודע בין Cost Optimization ל-Reliability, ובדרך כלל במבחן העדיפות היא HA אלא אם נאמר במפורש "minimize cost".

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[09 - VPC Fundamentals]] · [[10 - VPC Internet Connectivity]] · [[11 - VPC Security]] · [[12 - VPC Private Connectivity]] · [[15 - CloudFront and Global Delivery]] · [[36 - Migration and Hybrid Cloud]] · [[37 - Cost Optimization]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 12725–12778, 13495–13560, 14099–14328, 14473–14660
