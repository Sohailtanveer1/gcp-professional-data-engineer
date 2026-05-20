# Cheat Sheet: Streaming vs Batch Decision Framework

> Use this decision tree for every pipeline design question on the exam.

---

## Primary Decision Tree

```
What is the acceptable latency for results?
│
├─► Seconds or less (real-time)
│     │
│     ├─► Does order matter per key?
│     │     ├─► Yes → Pub/Sub (ordering keys) → Dataflow Streaming
│     │     └─► No  → Pub/Sub → Dataflow Streaming
│     │
│     └─► Do you need exactly-once processing?
│           ├─► Yes → Pub/Sub + Dataflow (exactly-once guarantee)
│           └─► No  → Pub/Sub + Dataflow (at-least-once, dedup in BQ)
│
├─► Minutes (near-real-time / micro-batch)
│     └─► Pub/Sub + Dataflow (5–10 min fixed windows)
│         or Dataproc Structured Streaming
│
└─► Hours or days (batch)
      │
      ├─► Data already in GCS?    → Dataflow Batch or Dataproc
      ├─► Data already in BQ?     → BigQuery SQL / Scheduled Queries
      ├─► Complex Spark pipeline? → Dataproc (ephemeral cluster)
      └─► Visual ETL needed?      → Cloud Data Fusion
```

---

## Latency vs Cost Trade-off Matrix

| Approach | Latency | Cost | Complexity | Best For |
|----------|---------|------|-----------|---------|
| Dataflow Streaming (per-event) | < 1 second | High (always running) | Medium | Real-time dashboards, fraud detection |
| Dataflow Streaming (windowed) | 1–60 seconds | Medium-High | Medium | Aggregated metrics, alerting |
| Dataproc Structured Streaming | 10–30 seconds | Medium | High | Existing Spark teams |
| Dataflow Batch | Minutes–Hours | Low (pay per job) | Low | ETL, daily aggregations |
| Dataproc Batch (ephemeral) | Minutes–Hours | Lowest | Medium | Spark transformations |
| BigQuery Scheduled Queries | 15 min–24 hr | Very Low | Very Low | Reporting, aggregations already in BQ |

---

## Windowing Strategy by Use Case

| Use Case | Window Type | Config |
|----------|-------------|--------|
| Hourly report | Fixed | `FixedWindows(60 min)` |
| 5-minute rolling average | Sliding | `SlidingWindows(5 min, every 1 min)` |
| User session analytics | Session | `Sessions(gap=5 min)` |
| Count all events ever | Global | `GlobalWindows()` |
| Micro-batch (mini-batch) | Fixed | `FixedWindows(30 sec)` |

---

## Handling Late Data — Decision Matrix

| Late Data Tolerance | Strategy |
|--------------------|---------|
| None (drop late data) | Default Dataflow — no `allowedLateness` |
| Up to N minutes late | `allowedLateness(Duration.standardMinutes(N))` |
| Early results + correction | `Trigger.AfterWatermark().withEarlyFirings().withLateFirings()` |
| Stateful dedup across 24h | Dataflow State API (`SetState` + processing-time timer) |
| Historical backfill needed | Dataflow Batch over historical GCS data |

---

## Exactly-Once Guarantee Map

| Source → Sink | Exactly-Once? | How |
|--------------|--------------|-----|
| Pub/Sub → Dataflow → BigQuery | ✅ Yes | Dataflow runner + BigQuery Storage Write API |
| Pub/Sub → Dataflow → GCS | ✅ Yes | Dataflow checkpointing |
| Pub/Sub → Dataflow → Bigtable | ⚠️ At-least-once | Bigtable conditional mutations for dedup |
| Dataflow → Pub/Sub (publish) | ⚠️ At-least-once | Pub/Sub doesn't deduplicate publishes |
| Kafka → Dataproc Streaming → BQ | ⚠️ At-least-once | Must implement idempotent writes |

---

## Streaming Pipeline Failure Modes & Fixes

| Failure Mode | Symptom | Fix |
|-------------|---------|-----|
| Hotspot / backlog growing | One worker at 100%, others idle | Fix Pub/Sub key distribution or add ordering |
| Data loss on restart | Missing events | Enable checkpointing in Dataflow |
| Duplicate records in BQ | Double-counted metrics | Use BigQuery Storage Write API (exactly-once mode) |
| Memory pressure on workers | OOM errors | Enable Streaming Engine (moves state off workers) |
| Late data dropped | Inaccurate historical data | Set `allowedLateness` + accumulating trigger |
| Slow watermark progress | Results always delayed | Check for stuck keys; set processing-time triggers |

---

## Batch Pipeline Patterns

### Daily ETL Pattern
```
Cloud Scheduler (cron)
  → Cloud Composer DAG trigger
    → Dataflow Batch Job (GCS → transform → BQ)
      → BigQuery Scheduled Query (aggregation)
        → Looker Dashboard refresh
```

### Backfill Pattern
```
# Run a Dataflow batch job over a date range
python pipeline.py \
  --start_date=2025-01-01 \
  --end_date=2025-03-31 \
  --runner=DataflowRunner \
  --temp_location=gs://bucket/temp
```

### Incremental Load Pattern
```
Use BigQuery MERGE statement:
MERGE target T
USING staging S ON T.id = S.id
WHEN MATCHED THEN UPDATE SET T.col = S.col
WHEN NOT MATCHED THEN INSERT (id, col) VALUES (S.id, S.col)
WHEN NOT MATCHED BY SOURCE THEN DELETE
```

---

## Common Exam Traps

```
❌ "Use Pub/Sub Lite for a pipeline that needs to survive a regional outage"
   → Pub/Sub Lite is ZONAL. Use standard Pub/Sub for HA.

❌ "Set a larger window to absorb late data"
   → Window size doesn't affect lateness. Use allowedLateness instead.

❌ "Dataproc Serverless for streaming"
   → Dataproc Serverless is BATCH ONLY. Use Dataflow for streaming.

❌ "Dataflow exactly-once means Pub/Sub deduplicates"
   → Dataflow handles exactly-once, not Pub/Sub. Pub/Sub is at-least-once.
     The Dataflow runner deduplicates using message IDs.

❌ "Session windows are good for network-delayed events"
   → Session windows are gap-based (activity → gap → new session).
     For delayed events, use allowedLateness on fixed/sliding windows.
```
