---
lesson: 35
title: Backup and Data Protection
domain: Design Resilient Architectures
services: [AWS Backup, EBS Snapshots, RDS Backups, S3 Versioning, S3 Object Lock, Vault Lock]
tags: [saa-c03, backup, disaster-recovery, compliance]
---

# 35 — Backup and Data Protection

> [!abstract] בשורה אחת
> Point-in-time recovery, immutability, encryption וcross-account/Region isolation — against deletion, corruption, ransomware.

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | Backup vs Snapshot vs Replica | מתי כל אחד, trade-offs |
| 2 | Native Snapshots | EBS, RDS, DynamoDB, Aurora PITR |
| 3 | AWS Backup | centralized service, policies, lifecycle |
| 4 | Immutability & Compliance | Vault Lock, Object Lock, WORM |
| 5 | Cross-Account/Region Protection | isolation נגד compromise |
| 6 | Restore Testing | proof ש-RTO/RPO feasible |

**מונחי מפתח בשיעור:** `Backup` · `Snapshot` · `Replica` · `PITR` · `Vault Lock` · `Object Lock` · `RPO` · `RTO` · `WORM`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- משהו מחק/רשע את ה-DB בטעות או בזדון.
- ransomware הצפין את כל הנתונים.
- Region כולו נפל; צריך תשחזור ל-Region אחר.
- compliance דורש שהנתונים לא יוכלו להימחק, אפילו גם בצד המנכנו.
- אתה לא יודע אם restore בהנחה שלך לפעלות בפועל.

### מה פתרונות פותרים

- **Backup:** recovery point בחזרה בזמן, זול יחסית, restore איטי.
- **Snapshot:** point-in-time של resource יחיד, מהיר יותר.
- **Replica:** online, זמינות + read-scale אך לא backup בפני compromise.
- **Vault Lock:** immutable retention — admin לא יכול למחוק עד TTL תם.
- **Cross-account/Region copy:** isolation פיזי מפני account compromise / Region outage.
- **Restore testing:** הוכחה ש-RTO/RPO achievable ו-dependencies כולם זמינים.

---

## 2. ⚙️ איך זה עובד

### 2.1 Native Snapshots (Service-Specific)

**EBS Snapshot:**
- Point-in-time של EBS volume, saved ל-S3.
- Incremental (רק changes מהsnapshot הקודם).
- AMI יכול להיות בנוי מ-snapshots.
- Automatic or manual.

**RDS Snapshot:**
- Manual (explicit) או automatic (daily retention).
- Includes data + binlogs.
- Restore תוך דקות (new DB instance).
- Cross-Region snapshots supported (via copy).

**Aurora PITR (Point-in-Time Recovery):**
- Automatic backtrack עד 35 ימים (MySQL מסגרת; PostgreSQL שונה).
- No manual snapshots needed; automatic backups + logs.
- Restore לכל second (almost) בתוך window.

**DynamoDB On-Demand Backups:**
- Manual backup (snapshot-style) full table copy.
- No impact on throughput.
- Restore ל-new table ב-seconds.

**DynamoDB PITR:**
- Automatic, 35 days window (like Aurora).
- Restore נקודה specific בקפיצות של 1 second.

### 2.2 AWS Backup (Centralized Service)

**Flow:**
1. Create Backup Plan (define schedule + retention).
2. Assign resources (by tag, manual, resource group).
3. Backup plan runs on schedule → writes to Backup Vault.
4. Vault stores backup + metadata.
5. Optionally copy ל-vault אחר (account/Region).
6. Restore from vault → create new resource.

**Advantages:**
- Central dashboard (many resources, services).
- Lifecycle: move to cold storage after N days (cheaper).
- Tag-driven governance (tag → auto-backup).
- Cross-account/Region backup (backup account separate).

**Vault Lock:**
- Enforce retention policy on vault.
- No delete, even admin.
- WORM (Write Once, Read Many).

### 2.3 S3 Versioning & Object Lock (for Data Files)

**Versioning:**
- Each PUT = new version; old versions retained.
- Restore by GET-ing specific version.
- Cost multiplier (each version billed).

**Object Lock (Compliance Mode):**
- WORM = no delete until retention expires (even AWS can't override).
- Governance Mode = admin can override (with special permission).
- Immutable retention = protection from ransomware.

### 2.4 RDS Multi-AZ & Aurora Global Database (Online Replication, Not Backup)

**Important:**
- Multi-AZ = automatic failover (HA), not backup.
- Lost data still lost (if DB corrupted, replica corrupted too).
- **Replicas require separate backup** for protection against data corruption.

---

## 3. 🔍 פירוק מפורט

### Backup Components

| רכיב | תפקיד |
|---|---|
| **Plan** | Schedule (daily, weekly) + retention (30 days, 1 year) |
| **Vault** | Storage container, region-specific, KMS encryption |
| **Vault Lock** | Immutable retention, WORM enforcement |
| **Lifecycle** | Move to cold storage after N days (cheaper) |
| **Copy** | Cross-account/Region redundancy |
| **Restore** | Create new resource from backup |

### Recovery Window by Mechanism

| מנגנון | Window | Speed | Cost |
|---|---|---|---|
| Automatic Snapshot (RDS/Aurora) | days–weeks | minutes | medium |
| Manual Snapshot | unlimited | minutes | medium |
| PITR (Aurora/DDB) | 35 days | second-level granularity | higher (logs) |
| AWS Backup + Vault | years (retention policy) | minutes–hours | per-storage |
| Cross-Region Backup | unlimited | hours | storage + transfer |
| S3 Versioning | unlimited | immediate | multiplier |

### Protection Levels

| Level | Protection | Trade-offs |
|---|---|---|
| **Same Region, Same Account** | Point-in-time recovery | NOT protected from account compromise / Region outage |
| **Same Region, Separate Account** | Account-level isolation | Cross-account copy = restore time, data transfer cost |
| **Cross-Region, Separate Account** | Account + Region isolation (BEST) | Highest cost, longest restore |
| **S3 Object Lock (Compliance)** | Immutable retention | No delete even by admin until TTL expires |
| **Vault Lock** | Immutable vault policies | No change to retention policy |

---

## 4. 💰 עלות ותמחור

### What You Pay For

| רכיב | יחידה | הערה |
|---|---|---|
| **EBS Snapshot storage** | per-GB/month | incremental = less storage |
| **RDS Backup storage** | per-GB/month beyond 100% DB size | automated backups free if < 100% DB |
| **AWS Backup storage** | per-GB/month | centralized, consolidated billing |
| **Vault Lock** | no extra charge | immutability enforcement |
| **Cross-Region Copy** | per-GB transfer | ~$0.02/GB inter-Region |
| **Restore** | per-GB or per-request | some services charge restore time |
| **S3 Versioning** | per-GB × # versions | storage multiplier (3 versions = 3× cost) |

### Cost Comparison

| Scenario | Method | Monthly Cost |
|---|---|---|
| 1TB RDS daily snapshot, 7-day retention | Native RDS snapshot | ~$7 (incremental) |
| Same, AWS Backup | AWS Backup | ~$7–10 (centralized overhead) |
| Same, Cross-Region copy | RDS snapshot + cross-region | ~$7 + $20 (transfer) + $7 (remote storage) |
| S3 versioning, 10 versions/object | S3 versioning (1000 objects, 10MB each) | $0.023 (standard) × 10 versions ≈ $2.30/month multiplier |

### 🚩 עלויות נסתרות

- **Snapshot copy frequency:** frequent = high transfer costs.
- **Retention too long:** archive tier cheaper per-GB אך restore איטי + minimum-duration.
- **Restore traffic:** cross-Region restore = egress charges.
- **Automated backup >= 100% DB:** billed (RDS has limit).
- **Vault storage lifecycle:** moving to cold = cheaper storage, expensive retrieval.

### 💡 Cost Optimization

- Use incremental snapshots (EBS, RDS).
- Archive old backups ל-cold storage after retention minimum.
- Delete snapshots not needed (vs. indefinite retention).
- Cross-account backup = trade cost ל-isolation (budget יעודי).
- Schedule backups during off-peak (batch window).

---

## 5. ⚖️ השוואות מכריעות

### Backup vs Replica vs Multi-AZ

| מימד | Backup | Replica (Read) | Multi-AZ |
|---|---|---|---|
| Protection from data corruption | YES | NO (replica corrupted too) | NO |
| Protection from delete | YES | NO | NO |
| Protection from Region outage | YES (if cross-region) | NO (replica same region) | NO (same region) |
| RTO | hours–days | N/A | seconds (failover) |
| RPO | hours–days | N/A | 0 (synchronous) |
| Cost | low–medium | medium | medium (instance overhead) |
| Use case | DR, compliance, retention | read-scaling, reporting | HA, failover |

> [!info] שורה תחתונה
> **Multi-AZ for HA, Replica for read-scale, Backup for DR + compliance.**

### Snapshot vs Backup

| קריטריון | Native Snapshot | AWS Backup |
|---|---|---|
| Service coverage | Service-specific (EBS/RDS/etc) | Multi-service (unified) |
| Lifecycle management | Manual | Automatic, policy-driven |
| Cross-account/Region | Manual copy | Built-in |
| Immutability (Vault Lock) | No | YES |
| Cost visibility | Scattered | Consolidated |
| Compliance workflow | Manual tracking | Audit-ready reports |

> [!info] שורה תחתונה
> **Snapshot for individual resource; Backup for organization-wide DR + compliance.**

### Object Lock vs Vault Lock

| מימד | Object Lock | Vault Lock |
|---|---|---|
| What it locks | S3 object (individual retention) | Backup Vault (entire policy) |
| Compliance Mode | No delete even by AWS | No change to vault retention |
| Governance Mode | Admin can override (with special permission) | N/A |
| Supported storage | S3 only | Backup, any service |
| Use case | file-level retention (compliance) | vault-level WORM (ransomware protection) |

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | central, automated, audited | AWS Backup policy, tag governance, runbooks, restore drills quarterly |
| Security | encrypted, isolated, immutable | KMS encryption, Vault Lock, separate backup account, MFA delete |
| Reliability | known RPO/RPO, tested recovery | cross-Region copies, documented RTO, restore testing in sandbox |
| Performance Efficiency | parallel restore, warm standby | EBS snapshots incremental, RDS read replicas לrestore testing |
| Cost Optimization | retention by policy, cold tier | lifecycle → cold after 30 days, delete old backups, incremental snapshots |
| Sustainability | avoid redundant copies, dedup | incremental snapshots, compress archives, one backup source of truth |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| Point-in-time recovery, 35 days | Aurora PITR / RDS PITR / DDB PITR — not snapshots |
| Immutable retention, no delete | Vault Lock (Backup) / Object Lock (S3) |
| Ransomware protection | Cross-account/Region backup + Vault Lock |
| Account compromise recovery | Cross-account backup (separate account) |
| RTO hours, RPO hours | backup + restore automation; not replica |
| Compliance audit trail | AWS Backup (centralized, auditable) |

### טעויות נפוצות

> [!warning] מלכודת
> **הניסוח:** "Snapshot in same account/Region protects against account compromise."
> **הטעות:** Snapshots in same account = accessible by compromised account.
> **הנכון:** **Cross-account** copy ל-backup account (separate credentials) נדרש.

> [!warning] מלכודת
> **הניסוח:** "Multi-AZ replication is a backup."
> **הטעות:** Multi-AZ = failover, not backup. If DB corrupted, replica corrupted too.
> **הנכון:** Backup ≠ Replica. Use both: replica ל-HA, backup ל-DR.

> [!warning] מלכודת
> **הניסוח:** "Backup completed successfully, so we're ready for disaster recovery."
> **הטעות:** Backup completed ≠ restore works. Missing keys, permissions, DNS, dependencies.
> **הנכון:** **Test restore** in non-production sandbox, validate RTO/RPO.

> [!warning] מלכודת
> **הניסוח:** "Vault Lock is encryption."
> **הטעות:** Vault Lock is immutability/retention enforcement, not encryption.
> **הנכון:** Vault Lock **+ KMS encryption** together = secured retention.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:**
- Production RDS cluster (orders database, critical).
- Compliance: retain backups 7 years, immutable after 30 days.
- DR: RTO = 4 hours, RPO = 1 hour (Region outage).
- Security: protect from ransomware.

**Architecture:**
```text
Production Account (us-east-1)
  ├→ RDS Aurora cluster (Multi-AZ)
  └→ AWS Backup Plan
       ├→ Daily snapshots (retention 7 years)
       ├→ Vault Lock after 30 days
       └→ Copy ל-backup account (us-west-2)

Backup Account (us-west-2) [separate credentials]
  └→ Backup Vault (immutable after 30 days)
       └→ Cold storage after 90 days
```

**Plan:**
| Step | Action | RPO/RTO |
|---|---|---|
| **Normal ops** | Daily automated snapshot | backup within 1 hour |
| **Region outage detected** | Initiate restore in us-west-2 | restore completes ~2-4 hours (restore automation, not manual) |
| **Recovery** | point-in-time (latest snapshot + logs) | RPO = 1 hour, RTO = 4 hours |
| **Validation** | Restore to sandbox in backup account, test app & connectivity | quarterly drills |
| **Ransomware scenario** | Compromised production account cannot access backup account (separate credentials) | isolated |

**Trade-offs:**
- Multi-AZ (HA) vs. backup (DR): both needed.
- Daily snapshots (hourly RPO ideal אך expensive) vs. PITR (logs retained 35 days).
- Cross-Region copy (isolation) vs. cost (data transfer).

---

## 9. 🚫 מה לא צריך לדעת למבחן

- Exact snapshot quota per Region.
- Each resource's native snapshot syntax details.
- AWS Backup + Secrets Manager integration specifics.
- Incremental snapshot algorithm details.
- PITR calculation (exact second granularity).

---

## 10. ⚡ Cheat Sheet

- **Backup** = recovery point (restore takes time).
- **Snapshot** = point-in-time of one resource.
- **Replica** = live copy, not backup.
- **Multi-AZ** = HA + failover, not DR.
- **PITR** = seconds-level granularity, up to 35 days (Aurora/DDB/RDS).
- **Vault Lock** = immutable retention (no delete until TTL).
- **Cross-account backup** = protection from account compromise.
- **Cross-Region backup** = protection from Region outage.
- **Object Lock** = S3 immutability.
- **RPO** dictates backup frequency; **RTO** dictates restore automation.
- **Always test restore** (quarterly).

---

## 11. ✅ בדיקת הבנה

1. **What protects data from account compromise?**
   - A) Same-account snapshot
   - B) Multi-AZ replication
   - C) Cross-account backup copy ✓
   - D) Vault Lock alone

2. **Aurora PITR window is 35 days. Compliance requires 7-year retention. Which mechanism?**
   - A) PITR alone
   - B) Daily snapshots + AWS Backup + lifecycle ✓
   - C) Replication
   - D) S3 versioning

3. **What's the difference between RTO and RPO?**
   - A) RTO = time to restore (Recovery Time Objective), RPO = data loss window ✓
   - B) Same thing
   - C) RTO is cost, RPO is performance
   - D) No difference, AWS says so

<details>
<summary>תשובות</summary>

1. **C) Cross-account backup copy.** Same-account snapshot accessible by compromised account. Multi-AZ replication in same account. Vault Lock ≠ account isolation. Cross-account = separate credentials.

2. **B) Daily snapshots + AWS Backup + lifecycle.** PITR only 35 days. AWS Backup stores + moves to cold tier after 30 days, cheaper long-term. Compliance dashboard auditable.

3. **A) RTO = restore time, RPO = data loss window.** RTO = how long to recover (hours), RPO = how much data you lose (hours of work). "Recovery" vs. "Recovery Point."

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[21 - RDS Fundamentals]] · [[34 - Disaster Recovery]] · [[36 - Migration and Hybrid Cloud]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 1478–1519, 3020–3060, 5823–5855, 8676–8690, 15096–15166
