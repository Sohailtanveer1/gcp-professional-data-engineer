# Exam Day Cheat Sheet — GCP Professional Data Engineer

> One page. Read this the morning of your exam.

---

## The 5 Questions to Ask Before Answering

```
1. Is the workload BATCH or STREAMING?
2. Does the scenario say "existing Spark code"? → Dataproc (not Dataflow)
3. Does it say "global", "multi-region", "ACID transactions"? → Spanner
4. Does it say "millisecond latency + NoSQL + time-series"? → Bigtable
5. Does it say "prevent exfiltration" (not just "restrict access")? → VPC-SC
```

---

## Numbers to Memorize

```
BigQuery partition limit (time)    4,000
BigQuery partition limit (int)     10,000
BigQuery clustering columns        4 max
BigQuery Time Travel               7 days (default + max)
BigQuery streaming buffer → cols   ~90 minutes
BigQuery row size max              100 MB
BigQuery on-demand slots (default) 2,000 / project

Pub/Sub message retention          7 days max
Pub/Sub Lite                       Zonal only

GCS Nearline min retention         30 days
GCS Coldline min retention         90 days
GCS Archive min retention          365 days

Bigtable node QPS (reads)          ~10,000 / node
Bigtable node write rate           ~100,000 / node

DLP streaming inspection           Per-record API call
Secret Manager                     Versioned, audited, IAM-gated
```

---

## Service One-Liners

```
Pub/Sub          → Global, auto-scale, event streaming
Pub/Sub Lite     → Zonal, cheap, pre-provisioned capacity
Dataflow         → Managed Beam, batch+stream, exactly-once with Pub/Sub
Dataproc         → Managed Spark/Hadoop; use ephemeral clusters
Dataproc SLSS    → Serverless Spark; batch only, no cluster
Data Fusion      → Visual ETL; Informatica replacement; runs on Dataproc
Cloud Composer   → Managed Airflow; complex DAGs; ETL orchestration
Datastream       → CDC from MySQL/PG/Oracle; seconds latency
Eventarc         → Event routing (not messaging); GCP service triggers
BigQuery         → Petabyte analytics; columnar; decoupled storage/compute
Bigtable         → Petabyte NoSQL; ms latency; row key = only index
Spanner          → Global relational ACID; TrueTime; expensive
Cloud SQL        → Regional relational; MySQL/PG/SQL Server; < 64 TB
AlloyDB          → High-perf PostgreSQL; columnar cache; > 64 TB OK
Firestore        → Document DB; mobile/web; real-time sync
GCS              → Object storage; data lake; immutable blobs
BigQuery ML      → Train/predict in SQL; stays in BQ; ARIMA for TS
Vertex AI        → Full ML platform; custom/AutoML/pipelines/monitoring
Feature Store    → Prevent train-serve skew; online < 10ms; batch training
Vertex Pipelines → KFP-based MLOps; artifact tracking; caching
IAM              → Identity-based: who can do what
VPC-SC           → Network-based: where data can flow
CMEK             → Your key in Cloud KMS; revocable; auditable
CSEK             → You supply key per call; Google never stores
Cloud DLP        → PII discovery + transformation; inline or batch
Policy Tags      → BQ column-level security via Data Catalog
Row Access Policy→ BQ row-level security on SESSION_USER()
Dataplex         → Data lake fabric; auto-discovery; DQ rules; lineage
Data Catalog     → Metadata search + tagging; Policy Tag taxonomy
Secret Manager   → Encrypted, versioned secrets; IAM-gated
Bucket Lock      → WORM compliance; irreversible
Org Policy       → Resource constraints; preventive; org-wide
```

---

## IAM Gotchas

```
BQ query = dataViewer (dataset) + jobUser (project) — two roles required
allAuthenticatedUsers ≠ your org — it's any Google account in the world
IAM Deny > IAM Allow — evaluated first, blocks even if Allow grants it
Lower-level IAM cannot restrict higher-level grants
SA keys → avoid; use Workload Identity Federation instead
Default compute SA → too broad; create dedicated SAs per workload
```

---

## Encryption Decision

```
No requirement            → GMEK (default, free, automatic)
Need revocation + audit   → CMEK (Cloud KMS, your key)
Key must leave Google     → CSEK (you supply per call)
Revoking CMEK key         → Data inaccessible immediately
Destroying CMEK key       → Data permanently gone, no recovery
CMEK ≠ protection from Google (KMS is still Google infra)
```

---

## DLP Transformation Decision

```
One-way, no recovery      → Redaction or Masking
JOIN across anonymized    → Crypto-based replacement (deterministic)
Need to recover original  → Tokenization or FPE (keep the key)
Date privacy (healthcare) → Date shifting
Numeric ranges (age)      → Bucketing
Real-time masking         → DLP inline in Dataflow pipeline
```

---

## Streaming Pipeline Gotchas

```
Late data    → allowedLateness (not larger window)
Dedup 24h+   → State API SetState (not windowing)
Exfiltration → VPC-SC (not IAM alone)
Pub/Sub Lite → Zonal = fails HA requirements
Dataproc SS  → Batch only, no streaming
Streaming Engine → A Dataflow flag, not a GCP service
Exactly-once → Dataflow + BQ Storage Write API (not just Pub/Sub)
```

---

## ML Gotchas

```
Dropout          → Training only; OFF at inference
Model parallel.  → Model too big for 1 GPU (not data parallelism)
Traffic split    → On Endpoint (not Model Registry)
Feature Store    → Point-in-time lookup prevents data leakage
Skew             → Your pipeline bug (training ≠ serving features)
Drift            → World changed (distribution shift over time)
BQML SHAP        → ML.EXPLAIN_PREDICT() (not Vertex Explainable AI)
AutoML           → Vertex AI AutoML (not Cloud AutoML — same thing now)
```

---

## Architecture Mnemonics

```
Standard streaming:    Pub/Sub → Dataflow → BigQuery (+ Bigtable for hot path)
Standard batch ETL:    GCS → Dataflow → BigQuery
Hadoop migration:      HDFS→GCS, Oozie→Composer, Hive→BQ or Dataproc Metastore
CDC to analytics:      Source DB → Datastream → GCS/BQ → Dataflow → BQ
MLOps loop:            BQ → Feature Store → Vertex Pipelines → Model Registry
                       → Endpoint → Model Monitoring → Composer retrain
Secure data lake:      GCS(CMEK) → DLP → BQ(Policy Tags, Row Policies) + VPC-SC
Multi-tenant BQ:       Row Access Policies + Policy Tags (not separate datasets)
```

---

## Last 10 Minutes Before Exam

- Pub/Sub Lite = **Zonal** (will fail any HA scenario)
- BigQuery **partition limit = 4,000** for time-based
- BigQuery query = **dataViewer + jobUser** (two roles)
- VPC-SC **prevents exfiltration**; IAM controls **access**
- CMEK **≠** protection from Google — use CSEK for that
- Dropout is **training only**, not at inference
- Dataproc Serverless = **batch Spark only**
- Data Access logs = **must enable manually** (not on by default)
- Bigtable row key = **the only indexed dimension** — design around queries
- Point-in-time Feature Store lookup = **prevents data leakage**

---

*Good luck. You've got this.*
