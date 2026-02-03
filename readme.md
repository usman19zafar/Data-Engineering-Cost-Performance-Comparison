# 📊 Cost & Performance Comparison: Databricks Notebook Clusters vs ADF vs Databricks Workflows

## 🔍 Project Overview

This repository presents an **experimental and analytical comparison** of three common orchestration/execution approaches used in modern Azure data engineering architectures:

1. **Databricks Notebook Clusters (manual / interactive execution)**
2. **Azure Data Factory (ADF) → Databricks orchestration**
3. **Databricks Workflows (Jobs) orchestrating multiple notebooks, pipelines, and queries**

The goal is to answer a very practical question:

> *For a given workload, which option is more **time-efficient** and **cost-efficient**, and when is each approach justified?*

This project combines:

* Real execution-time measurements
* Controlled experimental conditions
* Cost estimation
* Scaling assumptions
* Visual evidence (screenshots)

All results are backed by **actual runs**, not theoretical benchmarks.

---

## 🧪 Experimental Setup

### Cluster Configuration

* **VM Type:** Standard_D4s_v3
* **Cores:** 4
* **Memory:** 16 GB
* **Runtime:** Databricks Runtime (Spark)
* **Pricing Assumption:** ~$0.55 per DBU-hour

### Workload Characteristics

* **Number of notebooks:** 3 notebooks in each scenario
* **Queries:** Identical PySpark queries across all notebooks
* **Data size:** Small dataset (KB-scale) to intentionally surface orchestration overhead
* **Execution mode:** Sequential execution

> ⚠️ Important: Small data is intentionally used to highlight **orchestration overhead**, which is usually hidden at large scale.

---

## ⚙️ Compared Scenarios

### 1️⃣ Notebook Cluster (Manual Execution)

* Notebooks executed sequentially on an **already-running cluster**
* Includes **extra cluster setup code** inside notebooks
* No external orchestration

### 2️⃣ Azure Data Factory → Databricks

* ADF pipeline triggers Databricks notebook execution
* Adds external orchestration, REST calls, and activity management
* Single notebook measured (ADF overhead isolated)

### 3️⃣ Databricks Workflows (Jobs)

* Databricks-native orchestration
* Sequential execution of **3 notebooks / queries / pipelines**
* Internal dependency management, monitoring, retries

---
```code
## ⏱️ Experimental Results

| Workflow                           | Execution Time | Notes                                              |
| ---------------------------------- | -------------- | -------------------------------------------------- |
| Notebook cluster (3 notebooks)     | **71 sec**     | Fastest; minimal overhead; cluster already running |
| ADF → Databricks (3 notebooks)     | **103 sec**    | ~30 sec orchestration overhead                     |
| Databricks Workflows (3 notebooks) | **167 sec**    | Sequential orchestration inside Databricks         |

> 📌 Even with **extra cluster setup code**, the notebook cluster approach was the fastest for workloads. all other codes were exactly the same.
```
---

## 💰 Cost Estimation

Using $0.55 / DBU-hour:

```code

| Workflow             | Runtime (sec) | Estimated Cost ($) |
| -------------------- | ------------- | ------------------ |
| Notebook cluster     | 71            | ~$0.011            |
| ADF → Databricks     | 103           | ~$0.016            |
| Databricks Workflows | 167           | ~$0.025            |
```

> Cost differences are small for short jobs, but **overhead compounds** at scale and with frequent runs.

---

## 📈 Visual Evidence

This repository includes screenshots proving:

* Actual Databricks job runtimes
* ADF pipeline execution times
* Workflow execution history
* Logs showing notebook start/end markers

📁 **See `/images/` folder for experimental proof.**

---

## 📊 Key Insights

### 🔹 1. There is NO hard data-size limit

* Notebooks, ADF, and Databricks pipelines can all handle **TB–PB scale**
* Limits come from **cluster size, partitioning, and design**, not tools

### 🔹 2. Orchestration overhead is real

* ADF adds ~30+ seconds per run
* Databricks Workflows add overhead per task, but avoid external calls

### 🔹 3. Small data favors simplicity

* For small or experimental workloads:

  * Notebook clusters are fastest and cheapest

### 🔹 4. Production favors orchestration

* Databricks Workflows provide:

  * Monitoring
  * Retries
  * Dependency control
* ADF is best when:

  * Multiple Azure services must be coordinated

---

## 🏗️ Architectural Guidance

```code
| Use Case                           | Recommended Tool     |
| ---------------------------------- | -------------------- |
| Ad-hoc analysis                    | Notebook cluster     |
| Pure Spark pipelines               | Databricks Workflows |
| Cross-service orchestration        | ADF                  |
| Enterprise scheduling & monitoring | ADF + Databricks     |
```

> **There is no universal winner — only the right tool for the workload.**

---

## 🚀 Why This Project Matters

This repository demonstrates:

* Real-world performance testing
* Cost-awareness in cloud data engineering
* Architectural decision-making
* Understanding of Spark vs orchestration trade-offs

It is intentionally designed to be:

* Interview-ready
* Portfolio-ready
* Reproducible

---

## 📌 Future Enhancements

* Scale tests to GB/TB data
* Autoscaling cluster comparison
* Interactive cost modeling
* Plotly dashboards
* Delta Live Tables comparison

---

## 📎 How to Reproduce

1. Use a Databricks workspace
2. Create a Standard_D4s_v3 cluster
3. Run notebooks sequentially
4. Trigger the same notebooks via ADF and Workflows
5. Capture execution times

---

## 🧠 Final Takeaway

> **Execution engines scale data. Orchestrators scale complexity.**

Understanding the difference is key to building efficient, cost-aware data platforms.

---

⭐ If this repository helped you, feel free to star it or fork it.
