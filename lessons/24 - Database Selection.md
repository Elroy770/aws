---
lesson: 24
title: Database Selection
domain: Design Cost-Optimized Architectures
services: [RDS, Aurora, DynamoDB, ElastiCache, Redshift, OpenSearch, DocumentDB]
tags: [saa-c03, databases, architecture-decision]
---

# 24 — Database Selection

> [!abstract] בשורה אחת
> בחר datastore לפי data model, queries, scale וlatency — לא לפי "פופולארי" או "scalable".

## 🗺️ מפת השיעור

| # | תחנה | מה תדע בסוף |
|---|---|---|
| 1 | Database Types | 8 קטגוריות וmuse case עבור כל אחת |
| 2 | Decision Tree | איך לשאול שאלות טקטיות |
| 3 | RDS/Aurora | OLTP relational, transactions, joins |
| 4 | DynamoDB | serverless key-value, massive scale, no joins |
| 5 | ElastiCache | cache layer, NOT durable storage |
| 6 | Specialized | Redshift (analytics), DocumentDB, OpenSearch |

**מונחי מפתח בשיעור:** `OLTP` · `OLAP` · `key-value` · `document` · `graph` · `time-series` · `purpose-built` · `managed`

---

## 1. 🎯 הבעיה והפתרון

### הבעיה בעולם האמיתי

- יש לך צורך לשמור נתונים ו-AWS מציע 15+ database services.
- "Scalable" אינו פתרון — DynamoDB יכול להיות יקר לכדורי קטנים, ואילו RDS טוב למשימות מוכנות.
- joins, transactions, analytics, full-text search — כל אחד דורש thinking שונה.
- מבחן שואל "מה הבחירה הטובה ביותר" בתוך constraints (עלות, latency, uptime).

### מה בחירות טובות פותרות

- **RDS/Aurora:** ACID transactions, SQL joins, schema strong, relational.
- **DynamoDB:** massive parallel reads/writes, no schema, serverless, expensive ל-complex queries.
- **ElastiCache:** sub-millisecond access, session store, not durable (losing data OK).
- **Redshift:** analytics, aggregations, NOT transactions (slow).
- **DocumentDB, DynamoDB Streams:** semi-structured data, events.
- **OpenSearch:** full-text search, logs, NOT transactions.

---

## 2. ⚙️ איך זה עובד — Decision Tree

**שאל בתדר זה:**

1. **נדרשות transactions ACID ו-joins?**
   - YES → RDS (MySQL, PostgreSQL) או Aurora.
   - NO → עבור לשאלה 2.

2. **הנתונים הם key-value, no joins, massive parallel reads?**
   - YES → DynamoDB.
   - NO → עבור לשאלה 3.

3. **זה analytics, aggregations, historical data בהיקף גדול?**
   - YES → Redshift (or S3 + Athena).
   - NO → עבור לשאלה 4.

4. **זה קטגוריה search (full-text, logs, JSON)?**
   - YES → OpenSearch או Elasticsearch.
   - NO → עבור לשאלה 5.

5. **אתה צריך cache בתוך milliseconds, לא persistent?**
   - YES → ElastiCache (Redis / Memcached).
   - NO → עבור לשאלה 6.

6. **זה semi-structured, events, documents?**
   - YES → DocumentDB (MongoDB) או Kinesis.
   - NO → S3 (object store) או Neptune (graph).

---

## 3. 🔍 Database Patterns & Use Cases

### 3.1 OLTP vs OLAP

| מימד | OLTP (RDS/Aurora) | OLAP (Redshift/Athena) |
|---|---|---|
| Use Case | Orders, Transactions, Real-time | Analytics, Aggregations, Reports |
| Volume | עשרות Mbps | Gigabytes/Terabytes |
| Queries | Simple, indexed | Complex, joins, groupby |
| Latency | sub-100ms | seconds–minutes |
| Schema | Strong, normalized | Denormalized, schema-on-read |
| Data freshness | Real-time | Daily/Hourly batches |

### 3.2 Relational (RDS/Aurora)

**מתאים:**
- Orders, users, inventory (normalized).
- Complex joins, transactions, ACID compliance דרוש.
- Constraints, foreign keys, schema validation.
- Reads ו-writes balanced.

**לא מתאים:**
- Massive unstructured blob (→ S3).
- Real-time analytics ביליארד rows (→ Redshift).
- Key-lookup בלבד בקנה מידה massiv (→ DynamoDB).

**Scaling:** Read Replicas ל-reads; Multi-AZ ל-HA; Sharding (manual) ל-massive scale אך complex.

### 3.3 Key-Value (DynamoDB)

**מתאים:**
- Session store, user profiles (key = userId, value = profile).
- Shopping carts (key = cartId).
- Massive parallel reads/writes (millisecond latency).
- Serverless (pay-per-request בחינם scaling).

**לא מתאים:**
- Joins (אין תמיכה native).
- Complex queries (filter ב-application).
- Analytics, aggregations.

**Scaling:** Unlimited parallel access בתוך throughput allocation.

### 3.4 Cache (ElastiCache)

**Redis:**
- Strings, lists, sets, sorted sets.
- Session store, leaderboard, pub/sub.
- 99.99% uptime with Multi-AZ.

**Memcached:**
- Simple key-value only.
- אם data loss OK.

**לא מתאים:** durable source of truth (אבד בfailover).

### 3.5 Specialized

**Redshift:** BI, data warehouse, terabytes-petabytes, complex queries, denormalized.

**OpenSearch:** full-text search, logs, JSON nested queries, NOT transactions.

**DocumentDB (MongoDB-compatible):** semi-structured JSON, flexible schema, transactions (בגרסאות חדשות).

**Neptune:** graphs, knowledge bases, relationship queries complex.

**Timestream:** time-series, IoT, metrics, efficiently stored.

---

## 4. 💰 עלות ותמחור

### On-Demand vs Provisioned

| Service | On-Demand | Provisioned/Reserved |
|---|---|---|
| RDS | per-second billing | RI/Savings Plans (up to 72%) |
| Aurora | per-second compute + per-GB storage | Reserved (up to 66%) |
| DynamoDB | per request + per-GB stored | provisioned capacity (cheaper כש-predictable) |
| ElastiCache | per node hour | same (no on-demand) |
| Redshift | per-node hour | no flex (commit-based) |

### What You Pay For

| Service | Unit | Example |
|---|---|---|
| RDS | instance hour + storage + I/O + backup | db.t3.medium ~$60/month + storage |
| Aurora | instance hour + per-GB storage | similar to RDS, often premium |
| DynamoDB on-demand | requests (RCU/WCU) + GB stored | ~$1.25 per M requests + $0.25/GB/month |
| DynamoDB provisioned | RCU/WCU hours + GB stored | 1 RCU ~$0.0001/hour, better ל-baseline |
| ElastiCache | node hour | cache.t3.micro ~$30/month |
| Redshift | node hour | ra3.4xl+ ~$4–5/hour |

### 🚩 עלויות נסתרות

- **RDS read replicas:** cross-AZ / cross-Region transfer = ~$0.02/GB.
- **DynamoDB:** on-demand פתאום ב-spike = 10× provisioned cost.
- **ElastiCache:** eviction costs (new nodes) אם under-provisioned.
- **Redshift:** concurrency scaling בתשלום + suspend/resume complexity.
- **Data transfer:** all cross-Region = egress charges.

### 💡 Cost Comparison

| Workload | RDS | DynamoDB On-D | DynamoDB Prov | Redshift |
|---|---|---|---|---|
| Tiny app, bursty | HIGH (instance always on) | LOW (pay-per-request) | LOW | N/A |
| Steady 1000 req/s | MEDIUM | HIGH (1000 req/s = expensive) | LOW | N/A |
| Analytics terabyte | N/A | N/A | N/A | LOW (dedicated cluster) |

---

## 5. ⚖️ השוואות מכריעות

### RDS/Aurora vs DynamoDB

| קריטריון | RDS/Aurora | DynamoDB |
|---|---|---|
| Queries | SQL, joins, aggregations | key-lookup, no joins |
| Transactions | ACID, multi-row | single-item only |
| Schema | strong, normalized | flexible, denormalized |
| Scaling | vertical, sharding (hard) | horizontal, unlimited |
| Cost (small) | lower ($60/month baseline) | can be cheaper (pay-per-request) |
| Cost (large) | stable/predictable | can spike (on-demand) |
| Latency | 10–100ms | 1–10ms |

> [!info] שורה תחתונה
> **Joins, transactions, normalized data?** RDS/Aurora. **Key-lookup, schema-less, massive scale?** DynamoDB.

### ElastiCache vs RDS

| קריטריון | ElastiCache | RDS |
|---|---|---|
| Latency | <1ms (in-memory) | 10–100ms (disk) |
| Persistence | NO (lost on failover) | YES (durable) |
| Cost | per-node hour | instance-hours + storage |
| When | cache layer, sessions | source of truth |

> [!info] שורה תחתונה
> ElastiCache **in front of** RDS — never **instead of** RDS if data loss unacceptable.

### RDS vs Redshift

| קריטריון | RDS | Redshift |
|---|---|---|
| Use Case | OLTP transactions | OLAP analytics |
| Queries | fast, indexed | slow, complex aggregations |
| Schema | normalized | denormalized |
| Writes | per-second updates | batch loads |
| Cost | per-query cheap | per-node expensive |

> [!info] שורה תחתונה
> RDS for **application queries**; Redshift for **BI/analytics**; never use RDS ל-BI.

---

## 6. 🏛️ Well-Architected — ששת ה-Pillars

| Pillar | מה זה אומר **בנושא הזה** | פעולה קונקרטית |
|---|---|---|
| Operational Excellence | managed service, minimal toil | choose managed (RDS/Aurora/DDB) over self-managed |
| Security | encryption, IAM, least-privilege | TLS, encryption-at-rest, secrets rotation, NACLs |
| Reliability | backups, PITR, failover | Multi-AZ ל-RDS/Aurora; DynamoDB Global Tables; Redshift snapshots |
| Performance Efficiency | right-sized engine, indices, cache | RDS indexed, DDB projected attributes, ElastiCache hits |
| Cost Optimization | pick engine לפי use case | RDS steady-state, DynamoDB bursty, Redshift aggregation |
| Sustainability | avoid waste, purpose-built | minimize compute (purpose-built vs. generic RDS) |

---

## 7. 🪤 מלכודות במבחן

### מילות מפתח → תשובה

| אם בשאלה כתוב... | כנראה מתכוונים ל... |
|---|---|
| Joins, transactions | RDS / Aurora — not DynamoDB |
| Key lookup, serverless | DynamoDB — not RDS |
| Analytics, aggregations | Redshift — not RDS |
| Full-text search, logs | OpenSearch — not RDS |
| Cache, sub-millisecond | ElastiCache — not RDS |
| Structured JSON, flexible schema | DocumentDB — not RDS |

### טעויות נפוצות

> [!warning] מלכודת
> **הניסוח:** "Use DynamoDB because it's scalable."
> **הטעות:** scalability לא קריטריון עיקרי; queries ו-transactions הם.
> **הנכון:** בחר DynamoDB אם אתה **יודע שאין joins** ו-**access pattern הוא key-only** ו-**on-demand זול למונח זה**.

> [!warning] מלכודת
> **הניסוח:** "Redshift can handle OLTP."
> **הטעות:** Redshift איטי לכל query; OLTP דורש millisecond response.
> **הנכון:** RDS ל-OLTP; Redshift ל-analytics בק batches.

> [!warning] מלכודת
> **הניסוח:** "ElastiCache is a database."
> **הטעות:** ElastiCache אינו durable; data lost בfailover.
> **הנכון:** ElastiCache זה **cache layer**, תמיד מול durable DB.

> [!warning] מלכודת
> **הניסוח:** "DynamoDB can do joins."
> **הטעות:** אין native join support; חייב ב-application code.
> **הנכון:** DynamoDB ל-denormalized, key-value only; RDS ל-joins.

---

## 8. 🏗️ Scenario מהעולם האמיתי

**הדרישה:** e-commerce platform
- Users, orders, payments (transactional) → ACID, joins.
- Shopping cart (session) → fast, key-lookup.
- Catalog (hot data) → sub-millisecond.
- Product images → durable, object.
- BI reports → aggregations, terabytes historical.

```text
Application Layer
  ↓
  ├→ RDS Aurora (Orders, Users, Payments) — ACID, transactions
  ├→ DynamoDB (Cart sessions) — key-lookup, serverless
  ├→ ElastiCache (Catalog hot data) — <1ms cache hit
  ├→ S3 (Product images) — durable, cheap
  └→ Redshift (BI, historical analytics) — batched loads from RDS
```

**חלוקת עבודה:**

| Workload | Engine | Why |
|---|---|---|
| Order creation, payment | Aurora | transactions, joins, compliance |
| Cart lookup user:{id} → {items} | DynamoDB | key-only, serverless, millions/sec |
| Hot catalog (popular SKUs) | ElastiCache Redis | <1ms, session affinity |
| Product images, PDFs | S3 | durable, cheap, CDN integrated |
| Daily sales by region/category | Redshift | analytical queries, terabytes data |

---

## 9. 🚫 מה לא צריך לדעת למבחן

- Exact SQL syntax ל-RDS יתמך כ-feature-specific (window functions וכו').
- Redshift column vs row encoding nitty-gritty.
- DynamoDB streams latency microseconds (rare).
- DocumentDB replica lag specifics.
- OpenSearch analyzers syntax.

---

## 10. ⚡ Cheat Sheet

- **Joins + Transactions?** RDS/Aurora.
- **Key-lookup + Serverless?** DynamoDB.
- **Analytics + Petabytes?** Redshift.
- **Full-text search?** OpenSearch.
- **Cache, <1ms?** ElastiCache.
- **JSON, flexible?** DocumentDB.
- **Graph relationships?** Neptune.
- **Time-series metrics?** Timestream.
- **Never forget:** ElastiCache ≠ durable. DynamoDB ≠ joins. Redshift ≠ OLTP.
- **Multi-layered:** Aurora (durable) + ElastiCache (fast) + Redshift (BI) all in one app.

---

## 11. ✅ בדיקת הבנה

1. **Application needs ACID transactions and complex joins. What's the best choice?**
   - A) DynamoDB
   - B) RDS/Aurora ✓
   - C) Redshift
   - D) ElastiCache

2. **Key-value sessions, serverless, millions of requests per second. Best choice?**
   - A) RDS
   - B) DynamoDB ✓
   - C) ElastiCache
   - D) Redshift

3. **100TB historical data, daily aggregation queries for BI dashboards. Best choice?**
   - A) RDS
   - B) DynamoDB
   - C) Redshift ✓
   - D) ElastiCache

<details>
<summary>תשובות</summary>

1. **B) RDS/Aurora.** ACID + joins = relational database. DynamoDB can't join; Redshift too slow for OLTP; ElastiCache not durable.

2. **B) DynamoDB.** Serverless key-value = DynamoDB. RDS requires instance provisioning; ElastiCache not persistent; Redshift for analytics.

3. **C) Redshift.** 100TB analytical aggregations = Redshift (data warehouse). RDS too slow; DynamoDB no aggregation; ElastiCache not for BI.

</details>

---

## 🔗 קישורים

**שיעורים קשורים:** [[21 - RDS Fundamentals]] · [[22 - RDS Scaling and Availability]] · [[23 - DynamoDB]]

**שקפי מקור:** `AWS_Certified_Solutions_Architect_Slides.md` — שורות 9267–9766
