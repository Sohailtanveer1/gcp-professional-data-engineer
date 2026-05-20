# Cheat Sheet: GCP Data Service Selector

> The fastest way to answer "which service?" on the exam. Match your scenario signals to the right service.

---

## Storage Services

| Scenario Signal | Service |
|----------------|---------|
| Petabyte-scale SQL analytics, BI reports | **BigQuery** |
| Millisecond reads, time-series, IoT, > 1 TB NoSQL | **Bigtable** |
| Global relational ACID transactions, multi-region | **Cloud Spanner** |
| Regional relational DB, MySQL/PostgreSQL/SQL Server | **Cloud SQL** |
| High-performance PostgreSQL, mixed OLAP+OLTP, > 64 TB | **AlloyDB** |
| Document model, mobile/web app, real-time sync | **Firestore** |
| Object storage, data lake, files, Dataproc input | **Cloud Storage (GCS)** |

## Processing Services

| Scenario Signal | Service |
|----------------|---------|
| New streaming pipeline, serverless, Apache Beam | **Dataflow** |
| Existing Spark/Hadoop code, lift-and-shift | **Dataproc** |
| Batch Spark, no cluster management | **Dataproc Serverless** |
| Visual ETL, Informatica/Talend replacement | **Cloud Data Fusion** |
| SQL-based ELT, all data already in BigQuery | **BigQuery SQL / dbt on BQ** |
| Complex DAG orchestration, cross-system workflows | **Cloud Composer** |
| ML pipeline orchestration, artifact tracking | **Vertex AI Pipelines** |
| Event-driven, serverless trigger between services | **Eventarc** |

## Ingestion Services

| Scenario Signal | Service |
|----------------|---------|
| Real-time pub/sub messaging, variable traffic | **Cloud Pub/Sub** |
| High-volume, predictable, cost-sensitive messaging | **Pub/Sub Lite** |
| CDC from MySQL/PostgreSQL/Oracle to GCS/BigQuery | **Datastream** |
| Scheduled data transfer (SaaS, S3, HDFS to GCS) | **BigQuery Data Transfer Service** |
| On-prem network connectivity, private transfer | **Storage Transfer Service** |

## ML Services

| Scenario Signal | Service |
|----------------|---------|
| SQL team, data in BigQuery, fast time-to-model | **BigQuery ML** |
| Custom model, Python/TF/PyTorch, full control | **Vertex AI Custom Training** |
| No code, fastest path to model | **Vertex AI AutoML** |
| Feature sharing across models, prevent train-serve skew | **Vertex AI Feature Store** |
| Hyperparameter tuning | **Vertex AI Vizier** |
| Online predictions (< 100ms) | **Vertex AI Online Prediction** |
| Bulk scoring of millions of rows, async | **Vertex AI Batch Prediction** |
| Spark MLlib, existing ML code | **Dataproc** |

## Analytics & BI

| Scenario Signal | Service |
|----------------|---------|
| Enterprise BI, governed metrics, LookML semantic layer | **Looker** |
| Free, ad-hoc dashboards, quick sharing | **Looker Studio** |
| Streaming aggregation dashboard, real-time | **Dataflow + BigQuery + Looker Studio** |

## Security & Governance

| Scenario Signal | Service |
|----------------|---------|
| Who can access what resource | **IAM** |
| Prevent data exfiltration across projects | **VPC Service Controls** |
| Control your own encryption keys, revocable | **CMEK (Cloud KMS)** |
| Keys never stored by Google | **CSEK** |
| Discover + mask PII in GCS/BigQuery | **Cloud DLP** |
| Column-level BigQuery access control | **Policy Tags (Data Catalog)** |
| Row-level BigQuery access control | **Row Access Policies** |
| Metadata search, data discovery across org | **Data Catalog** |
| Data lake organization + automated governance | **Dataplex** |
| Secrets, passwords, API keys | **Secret Manager** |
| WORM compliance for GCS objects | **Bucket Lock** |
| Audit trail of all admin actions | **Cloud Audit Logs (Admin Activity)** |
| Audit trail of data reads/writes | **Cloud Audit Logs (Data Access — enable manually)** |
| Org-wide resource constraints | **Organizational Policies** |

---

## Quick Decision Trees

### Messaging: Pub/Sub vs Pub/Sub Lite
```
Traffic spiky or unknown?         → Pub/Sub
Global consumers?                 → Pub/Sub
Ordering + regional failover?     → Pub/Sub (with ordering keys)
Predictable high-volume + cheap?  → Pub/Sub Lite
Zonal OK?                         → Pub/Sub Lite
```

### Processing: Dataflow vs Dataproc
```
New pipeline + no existing code?  → Dataflow
Existing Spark/Hive/Pig code?     → Dataproc
True streaming (per-event)?       → Dataflow
Micro-batch OK?                   → Dataproc Streaming
No cluster management?            → Dataflow (or Dataproc Serverless for batch Spark)
```

### ML: AutoML vs Custom vs BQML
```
SQL team, data in BQ?             → BigQuery ML
No ML expertise needed?           → AutoML
Need custom architecture?         → Vertex AI Custom Training
Model too big for 1 GPU?          → Custom Training + Model Parallelism
```

### Encryption: GMEK vs CMEK vs CSEK
```
Default, no requirements?         → GMEK
Need key revocation + audit?      → CMEK
Key must never touch Google infra?→ CSEK
```
