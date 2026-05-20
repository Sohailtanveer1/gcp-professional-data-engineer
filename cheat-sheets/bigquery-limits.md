# Cheat Sheet: BigQuery Limits & Gotchas

> Every number and constraint that appears on the exam. Memorize these.

---

## Hard Limits

| Resource | Limit | Notes |
|----------|-------|-------|
| Partitions per table (time-based) | **4,000** | Daily = ~11 years; hourly = ~166 days |
| Partitions per table (integer range) | **10,000** | |
| Clustering columns per table | **4** | Applied in declared order |
| Columns per table | 10,000 | |
| Max table size | Unlimited | Storage auto-scales |
| Max query result size (interactive) | 10 GB | Use destination table for larger |
| Max row size | 100 MB | |
| Max JSON/CSV cell size | 100 MB | |
| Streaming insert row size | 1 MB | |
| Streaming insert max rows/request | 50,000 | |
| Streaming insert max bytes/request | 10 MB | |
| DML concurrent mutations per table | 20 | Applies to INSERT/UPDATE/DELETE |
| Max concurrent on-demand queries | 300/project | |
| Max concurrent slots (on-demand) | 2,000/project | Soft limit, requestable increase |
| Dataset region | Immutable after creation | Cannot move a dataset to another region |
| Time Travel window | 7 days default (max) | Configurable 2–7 days; storage cost |
| Fail-safe window | 7 days | After Time Travel expires; read-only, no config |
| Max authorized views per dataset | 2,500 | |
| Max Row Access Policies per table | 100 | |

---

## Pricing Gotchas

| Scenario | Gotcha |
|----------|--------|
| `SELECT *` on a 10 TB table | Scans **all** columns → full 10 TB charged |
| Query with partition filter on clustered column | Still scans all partitions unless partition filter also present |
| Streaming inserts | Charged per row/byte, even if query is free-tier |
| DML statements | Charged for bytes processed, not just rows modified |
| `CREATE TABLE AS SELECT` | Charges for bytes processed by the SELECT |
| External tables (GCS) | Charged for bytes read from GCS, not BQ storage |
| On-demand vs capacity pricing | On-demand: $ per TB scanned. Capacity: $ per slot-hour regardless of data scanned |

---

## Partition Behavior

```
# Partition pruning ONLY happens when you filter on the partition column
SELECT * FROM table WHERE _PARTITIONDATE = '2025-01-01'  -- ✅ prunes
SELECT * FROM table WHERE other_column = 'value'          -- ❌ full scan

# Partition expiration — auto-delete old partitions
ALTER TABLE table SET OPTIONS (partition_expiration_days = 90)

# __PARTITIONS_SUMMARY__ — inspect partition metadata without scanning data
SELECT * FROM dataset.INFORMATION_SCHEMA.PARTITIONS
WHERE table_name = 'my_table'
```

---

## DML Constraints

| Operation | Constraint |
|-----------|-----------|
| `UPDATE` / `DELETE` on partitioned tables | Must include partition filter (enforced by default) |
| `MERGE` | Counts as a DML statement; subject to concurrent mutation limit |
| `INSERT INTO ... SELECT` | Fully atomic; charged for SELECT bytes |
| `TRUNCATE TABLE` | Free (metadata-only operation) |
| DML + streaming inserts on same table | ⚠️ Not recommended simultaneously — can cause inconsistency |

---

## BigQuery ML Limits

| Resource | Limit |
|----------|-------|
| Max training rows (LOGISTIC_REG, LINEAR_REG) | 1 billion |
| Max training rows (DNN) | No hard limit (time limit applies) |
| Max training columns (features) | 10,000 |
| Max model size stored in BQ | 100 MB (larger models → GCS export) |
| ARIMA_PLUS max time series per model | 1,000,000 |

---

## Cross-Region Considerations

```
Dataset region is set at CREATION and is IMMUTABLE.
You cannot:
  - Move a dataset to another region
  - Query a table in us-central1 from a job in europe-west1 (cross-region queries blocked)
  - Join tables in different regions in a single query

To copy cross-region:
  - BigQuery Data Transfer Service (dataset copy job)
  - bq cp command (requires destination dataset pre-created in target region)
```

---

## Slot Quota Summary

| Tier | Default Slots | Max (soft limit) |
|------|--------------|-----------------|
| On-demand (per project) | 2,000 concurrent | Requestable |
| Standard capacity | Per commitment purchased | Autoscale up to baseline × 4 |
| Enterprise capacity | Per commitment | Autoscale configured |

---

## Common Exam Traps — BigQuery

```
❌ "Partition on a high-cardinality STRING column"
   → BigQuery doesn't support STRING partitioning.
     Partition on DATE/TIMESTAMP/INTEGER only.

❌ "Query filters on a clustered column without a partition filter"
   → Clustering improves performance WITHIN a partition.
     Without a partition filter, all partitions still scanned.

❌ "Use Time Travel to recover from a regional outage"
   → Time Travel is within the same region. Regional outage = no access.

❌ "BigQuery automatically replicates to another region"
   → It does NOT. Schedule cross-region copies manually.

❌ "Streaming inserts are immediately queryable"
   → ✅ True, BUT they're in a streaming buffer.
     The buffer is NOT covered by Time Travel until the data is moved
     to columnar storage (usually within 90 minutes).

❌ "CMEK on BigQuery encrypts query results"
   → CMEK encrypts data AT REST. Query results in temp tables use
     Google-managed keys unless you also specify CMEK for temp tables.
```
