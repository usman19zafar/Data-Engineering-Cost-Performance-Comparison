```Code
# Azure Data Processing Cost & Runtime Benchmark
Experimental baseline comparing three orchestration and execution paths:
1. Databricks Notebook Cluster
2. Azure Data Factory → Databricks
3. Databricks Workflows
```

All tests executed on:
- Cluster: Standard_D4s_v3 (4 cores, 16 GB RAM)
- Data size: 26.66 KB
- Operation: Simple Spark transformation (no shuffle, no joins)

This repository provides:
- Experimental timing baseline
- Cost assumptions (Azure)
- 100‑hour cost comparison
- ASCII architecture diagrams
- Cost‑per‑GB model
1. Experimental Baseline (Ground Truth)

```Code
+---------------------------------------------------------------+
| Scenario                                   | Time   | Data    |
+---------------------------------------------------------------+
| Notebook cluster (multiple notebooks)      | 71 sec | 26.66KB |
| ADF → Databricks (single notebook)         | 103 sec| 26.66KB |
| Databricks Workflows (3 notebooks, etc.)   | 167 sec| 26.66KB |
+---------------------------------------------------------------+
```

2. Runtime Assumptions

```Code
+--------------------------------------------------------------+
| Spark runtime scales linearly for simple operations          |
| Orchestration overhead is fixed (startup + scheduling)       |
| Same cluster size, same quaries used across all experiments  |
| No shuffle-heavy joins or skewed aggregations                |
+--------------------------------------------------------------+
```
3. Cost Assumptions (Azure – Conservative)

```Code
+------------------------------------+-------------------------+
| Item                               | Cost                    |
+------------------------------------+-------------------------+
| Databricks DBU (Standard)          | $0.15 per DBU-hour      |
| D4s_v3 DBU usage                   | ~2 DBU per hour         |
| VM cost (D4s_v3)                   | ~$0.20 per hour         |
| Total Databricks compute cost      | ~$0.50 per hour         |
| Azure Data Factory activity        | ~$0.005 per activity    |
+------------------------------------+-------------------------+
```
4. Time Breakdown (Small Data)
```Code
+---------------------+-----------+-----------+--------------+
| Component           | Notebook  | ADF → DB  | DB Workflows |
+---------------------+-----------+-----------+--------------+
| Fixed orchestration | ~40 sec   | ~70 sec   | ~120 sec     |
| Compute portion     | ~31 sec   | ~33 sec   | ~47 sec      |
+---------------------+-----------+-----------+--------------+
```
5. Cost Comparison for 100 Hours of Processing
Assumption:
Databricks compute = $0.50 per hour  
ADF orchestration = $0.005 per activity (~$6–$10 over 100 hours)

```Code
+----------------------+---------+-------------------+-----------+
| Scenario             | Compute | Orchestration     | Total     |
+----------------------+---------+-------------------+-----------+
| Notebook cluster     | $50     | $0                | $50       |
| ADF → Databricks     | $50     | ~$6 – $10         | $56 – $60 |
| Databricks Workflows | $50     | $0                | $50       |
+----------------------+---------+-------------------+-----------+
```
6. Final Verdict Summary
Time Efficiency (Large Data)

```Code
+-----------------------------+
| All approaches equivalent   |
| Choose based on architecture|
+-----------------------------+
```

Cost Efficiency (Long Runs)

```Code
+------+---------------------------+
| Rank | Winner                    |
+------+---------------------------+
| 1    | Notebook / DB Workflows   |
| 2    | ADF → Databricks          |
+------+---------------------------+
```

7. Architectural Fit

```Code
+-------------------------------+------------------------+
| Use Case                      | Best Choice            |
+-------------------------------+------------------------+
| Pure Spark transformations    | Databricks Workflows   |
| Multi-service Azure pipelines | Azure Data Factory     |
| Experiments / demos           | Notebook clusters      |
+-------------------------------+------------------------+
```

ASCII Architecture Diagrams
A. Notebook Cluster Execution

```Code
+---------------------+
| User / Trigger      |
+----------+----------+
           |
           v
+---------------------+
| Databricks Notebook |
| (Direct Execution)  |
+----------+----------+
           |
           v
+---------------------+
| Spark Cluster       |
| (D4s_v3 Workers)    |
+---------------------+
```
B. ADF → Databricks Pipeline
```Code
+---------------------+
| Azure Data Factory  |
| (Pipeline Trigger)  |
+----------+----------+
           |
           v
+-------------------------------+
| ADF Notebook Activity         |
+----------+--------------------+
           |
           v
+-------------------------------+
| Databricks Notebook Execution |
+----------+--------------------+
           |
           v
+-------------------------------+
| Spark Cluster (D4s_v3)        |
+-------------------------------+
```
C. Databricks Workflows
```Code
+---------------------------+
| Databricks Workflow Job  |
| (Orchestration Layer)    |
+-------------+-------------+
              |
              v
+---------------------------+
| Notebook 1 → Notebook 2 →|
| Notebook 3 (Chained)     |
+-------------+-------------+
              |
              v
+---------------------------+
| Spark Cluster (D4s_v3)    |
+---------------------------+
```
Cost‑Per‑GB Model
Using the 100‑hour compute cost:

Databricks compute: $0.50/hour

100 hours: $50 total compute

If 100 hours processes X GB, then:

```Code
Cost per GB = $50 / X
From your measured throughput:
```
Notebook processes 132 MB in 100 hours

ADF processes 91 MB in 100 hours

Convert to GB:

Notebook: 0.132 GB

ADF: 0.091 GB

Cost per GB (Notebook)
```Code
$50 / 0.132 GB ≈ $378 per GB
Cost per GB (ADF → Databricks)
```
```Code
$56 – $60 / 0.091 GB ≈ $615 – $659 per GB
Mechanical Truth
```
```Code
Notebook = cheaper per GB
ADF → DB = more expensive per GB
```
