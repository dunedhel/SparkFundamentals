# Apache Spark Foundation Workshop (Interactive Binder Environment)

Welcome to the **Apache Spark Foundation & Ingestion Framework** 2-Day Workshop. This environment runs interactive JupyterLab directly in your web browser with zero installation, zero setup, and **no user account required**.

---

## 🚀 Getting Started

Simply open the notebooks in numerical order from the file browser on the left:

1. **`01_spark_foundations_and_architecture.ipynb`**
   - Cluster Topology (Driver & Executors), SparkSession lifecycle, Partitions, and Ingesting StatsBomb Bundesliga JSON.
2. **`02_dataframes_complex_types_and_sql.ipynb`**
   - JSON Flattening (`explode`, `struct`), Analytical Window Functions, and Spark SQL Temp Views.
3. **`03_performance_shuffle_and_storage.ipynb`**
   - Shuffle Mechanics, Broadcast Hash Joins, Small File Problem on Cloud Storage, and Partitioned Parquet Export.
4. **`04_airflow_spark_orchestration.ipynb`**
   - Orchestrating Spark jobs with Apache Airflow DAGs, file sensors, and operational recovery (`clear task`).
5. **`05_production_framework_and_capstone.ipynb`**
   - Structured JSON logging, custom exception hierarchies, and the final Capstone KPI Pipeline.

---

## 💾 How to Save Your Work (Important!)

> ⚠️ **Note:** Binder sessions are temporary (ephemeral). If you close your browser tab or leave your computer inactive for more than 15 minutes, your session will reset.
>
> **To save your code and exercise solutions:**
> Click **File -> Download** (or right-click the notebook in the file tree and select **Download**) to save your `.ipynb` file to your local computer.

---

## 📊 Preloaded Datasets (`data/raw/`)

This environment comes pre-loaded with official **StatsBomb Open Data** from the 2023/2024 Bundesliga season:
* `data/raw/bundesliga_events.json`: 31,890 real event records (Bayer Leverkusen, Bayern Munich, Borussia Dortmund, VfB Stuttgart).
* `data/raw/statsbomb_match_3895158.json`: Multiline raw JSON match file.
* `data/raw/team_financials.csv`: Complete 18-team Bundesliga financial and stadium lookup dictionary.

---

## 📖 Cheatsheet
* [`SQL_to_PySpark_CheatSheet.md`](SQL_to_PySpark_CheatSheet.md): Quick mapping between SQL/PL-SQL syntax and PySpark functions.
