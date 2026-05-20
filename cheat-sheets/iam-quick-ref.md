# Cheat Sheet: IAM Quick Reference

> Every predefined role you need for the exam, organized by service.

---

## BigQuery Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/bigquery.admin` | Full control: datasets, tables, jobs | Data engineers (admin tasks) |
| `roles/bigquery.dataOwner` | CRUD on datasets + tables | Dataset owners |
| `roles/bigquery.dataEditor` | Read + write tables/views | ETL service accounts |
| `roles/bigquery.dataViewer` | Read tables, views, metadata | Analysts (need jobUser too!) |
| `roles/bigquery.metadataViewer` | List datasets/tables only | Catalog tools |
| `roles/bigquery.jobUser` | Run jobs in a project | **Required for any query** |
| `roles/bigquery.user` | Run jobs + create datasets | Standard analyst role |
| `roles/bigquery.readSessionUser` | BigQuery Storage API read | Spark/pandas connectors |

> **Critical combo**: To run a query, an analyst needs `bigquery.dataViewer` on the **dataset** AND `bigquery.jobUser` on the **project**. Missing either → query fails.

---

## Cloud Storage Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/storage.admin` | Full control: buckets + objects | Storage admins |
| `roles/storage.objectAdmin` | Full object CRUD | Dataflow SA for output |
| `roles/storage.objectCreator` | Create objects only (no read/delete) | Write-only ingest SA |
| `roles/storage.objectViewer` | Read objects + list bucket | Read-only access |
| `roles/storage.legacyBucketReader` | List objects in bucket | Needed with objectViewer for gsutil ls |

---

## Pub/Sub Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/pubsub.admin` | Full control | Admins |
| `roles/pubsub.editor` | Create/delete topics + subscriptions | Pipeline builders |
| `roles/pubsub.publisher` | Publish messages to topics | Producer SA |
| `roles/pubsub.subscriber` | Create subscriptions + consume | Consumer SA |
| `roles/pubsub.viewer` | View topics + subscriptions | Monitoring |

---

## Dataflow Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/dataflow.admin` | Full Dataflow control | Data engineers |
| `roles/dataflow.developer` | Submit + manage jobs | Dataflow SA / CI-CD |
| `roles/dataflow.worker` | Used by Dataflow worker VMs | Auto-granted to Dataflow SA |
| `roles/dataflow.viewer` | View jobs + metrics | Read-only monitoring |

> **SA chain for Dataflow**: The **Dataflow Service Account** needs `dataflow.developer`. The **Controller SA** (worker VM identity) needs `dataflow.worker` + roles on all data sources/sinks (e.g., `storage.objectAdmin`, `bigquery.dataEditor`).

---

## Dataproc Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/dataproc.admin` | Full Dataproc control | Admins |
| `roles/dataproc.editor` | Create/delete clusters + jobs | Data engineers |
| `roles/dataproc.worker` | Used by cluster worker VMs | Auto-granted |
| `roles/dataproc.viewer` | View clusters + jobs | Monitoring |

---

## Bigtable Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/bigtable.admin` | Full control | Admins |
| `roles/bigtable.user` | Read + write rows | Application SA |
| `roles/bigtable.reader` | Read rows only | Read-only analytics |
| `roles/bigtable.viewer` | View instance metadata | Monitoring |

---

## Cloud Spanner Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/spanner.admin` | Full control | Admins |
| `roles/spanner.databaseAdmin` | Create/drop databases + schemas | DBAs |
| `roles/spanner.databaseUser` | Read + write data + execute SQL | Application SA |
| `roles/spanner.databaseReader` | Read-only SQL | Reporting SA |
| `roles/spanner.viewer` | View instance metadata | Monitoring |

---

## Vertex AI Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/aiplatform.admin` | Full Vertex AI control | ML admins |
| `roles/aiplatform.user` | Submit training jobs + predictions | Data scientists |
| `roles/aiplatform.viewer` | View models + jobs | Stakeholders |
| `roles/ml.admin` | Legacy AI Platform admin | Old pipelines |

---

## Cloud KMS Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/cloudkms.admin` | Full key management | Security admins |
| `roles/cloudkms.cryptoKeyEncrypterDecrypter` | Encrypt + decrypt | Service SA using CMEK |
| `roles/cloudkms.cryptoKeyEncrypter` | Encrypt only | Write-only SA |
| `roles/cloudkms.cryptoKeyDecrypter` | Decrypt only | Read-only SA |
| `roles/cloudkms.viewer` | View key metadata | Auditors |

> **CMEK pattern**: Grant `cloudkms.cryptoKeyEncrypterDecrypter` to the **service's SA** (e.g., BigQuery SA, GCS SA) on the specific CryptoKey — not a broader role.

---

## Data Catalog Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/datacatalog.admin` | Full Data Catalog control | Governance admins |
| `roles/datacatalog.tagEditor` | Create/edit tags on entries | Data stewards |
| `roles/datacatalog.viewer` | Search + view catalog | All analysts |
| `roles/datacatalog.fineGrainedReader` | Read Policy Tag-protected BQ columns | Authorized analysts |

> **Column security pattern**: Apply a Policy Tag to a BQ column → only users with `datacatalog.fineGrainedReader` on that specific Policy Tag can see the column value. Everyone else sees NULL.

---

## Secret Manager Roles

| Role | Permissions | When to Grant |
|------|-------------|--------------|
| `roles/secretmanager.admin` | Full control | Security admins |
| `roles/secretmanager.secretAccessor` | Read secret values | Application SA |
| `roles/secretmanager.secretVersionManager` | Create/destroy versions | Secret rotation SA |
| `roles/secretmanager.viewer` | View secret metadata (not values) | Auditors |

---

## IAM Policy Hierarchy Rules

```
Organization IAM
  ↓ (inherited + additive)
Folder IAM
  ↓ (inherited + additive)
Project IAM
  ↓ (inherited + additive)
Resource IAM (dataset, bucket, topic)

Rules:
✅ Lower levels can ADD permissions granted at higher levels
❌ Lower levels CANNOT REMOVE permissions granted at higher levels
✅ IAM DENY policies override Allow policies (evaluated first)
✅ Grant at the most specific resource level possible (least privilege)
```

---

## Service Account Best Practices

```
✅ One SA per workload (not one SA for everything)
✅ Grant SA roles on specific resources, not projects
✅ Use Workload Identity Federation instead of SA keys for non-GCP
✅ Audit SA key usage → Secret Manager + Cloud Audit Logs
❌ Never grant roles/owner or roles/editor to a service account
❌ Never embed SA key files in container images or source code
❌ Never use default compute SA (too broad permissions)
```
