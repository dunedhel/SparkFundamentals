# Cheat Sheet & Guide: From SQL / PL-SQL to Apache Spark (PySpark)

This guide is designed for DWH analysts, data integration specialists, and SQL/PL-SQL developers transitioning to distributed data processing with PySpark.

---

## 1. Concept Mapping: SQL / DWH vs PySpark

| SQL / DWH Concept | PySpark Equivalent | Description in a Distributed Environment |
| :--- | :--- | :--- |
| **Table / View** | `DataFrame` | An immutable, distributed collection of data organized into named columns and divided into partitions. |
| **`SELECT` Query** | `df.select(...)` | A **lazy transformation** specifying which columns to project. |
| **`WHERE` / `HAVING` Clause** | `df.filter(...)` or `df.where(...)` | Row filtering. In PySpark, filter as early as possible to leverage predicate pushdown. |
| **`GROUP BY` + Aggregations** | `df.groupBy(...).agg(...)` | Grouping data. Triggers a **Shuffle** operation (redistributing data across cluster nodes). |
| **Stored Procedure / PL-SQL Cursor** | **PySpark Module + Functions** | In Spark, avoid row-by-row cursors and loops! Data is processed vectorially across entire columns/datasets. |
| **DML: `INSERT INTO / MERGE`** | `df.write.mode("append"/"overwrite")` | Writing balanced datasets to target storage (e.g., AWS S3 / Parquet) with partitioning. |
| **Data Types (VARCHAR, INT, DATE)** | `StringType`, `IntegerType`, `DateType` | PySpark SQL data types imported from `pyspark.sql.types`. |
| **Temporary View** | `df.createOrReplaceTempView("view_name")` | Allows executing standard ANSI SQL queries directly against a DataFrame within the active session. |

---

## 2. Syntax Comparison: SQL vs PySpark DataFrame API

### 2.1. Selecting Columns, Renaming, and Calculated Fields (`SELECT`, `AS`, Calc)

#### SQL:
```sql
SELECT 
    player_name AS player,
    team,
    shots,
    goals,
    (goals * 1.0 / NULLIF(shots, 0)) * 100 AS accuracy_pct
FROM match_events
WHERE minute < 90;
```

#### PySpark DataFrame API:
```python
from pyspark.sql import functions as F

result_df = df.filter(F.col("minute") < 90).select(
    F.col("player_name").alias("player"),
    F.col("team"),
    F.col("shots"),
    F.col("goals"),
    ((F.col("goals") * 1.0 / F.when(F.col("shots") == 0, None).otherwise(F.col("shots"))) * 100).alias("accuracy_pct")
)
```

---

### 2.2. Aggregations and Grouping (`GROUP BY` & `HAVING`)

#### SQL:
```sql
SELECT 
    team,
    COUNT(*) AS total_matches,
    SUM(goals) AS total_goals,
    AVG(possession) AS avg_possession
FROM team_stats
GROUP BY team
HAVING SUM(goals) > 20
ORDER BY total_goals DESC;
```

#### PySpark DataFrame API:
```python
from pyspark.sql import functions as F

result_df = (
    df.groupBy("team")
    .agg(
        F.count("*").alias("total_matches"),
        F.sum("goals").alias("total_goals"),
        F.avg("possession").alias("avg_possession")
    )
    .filter(F.col("total_goals") > 20)
    .orderBy(F.col("total_goals").desc())
)
```

---

### 2.3. Window Functions

#### SQL:
```sql
SELECT 
    player_name,
    team,
    goals,
    RANK() OVER (PARTITION BY team ORDER BY goals DESC) AS rnk,
    SUM(goals) OVER (PARTITION BY team) AS team_total_goals
FROM player_stats;
```

#### PySpark DataFrame API:
```python
from pyspark.sql import Window
from pyspark.sql import functions as F

windowSpec = Window.partitionBy("team").orderBy(F.col("goals").desc())
windowUnbounded = Window.partitionBy("team")

result_df = df.select(
    "player_name",
    "team",
    "goals",
    F.rank().over(windowSpec).alias("rnk"),
    F.sum("goals").over(windowUnbounded).alias("team_total_goals")
)
```

---

### 2.4. Conditional Logic (`CASE WHEN`)

#### SQL:
```sql
SELECT 
    player_name,
    goals,
    CASE 
        WHEN goals >= 15 THEN 'Top Scorer'
        WHEN goals >= 5  THEN 'Regular Scorer'
        ELSE 'Bench / Support'
    END AS player_category
FROM player_stats;
```

#### PySpark DataFrame API:
```python
from pyspark.sql import functions as F

result_df = df.withColumn(
    "player_category",
    F.when(F.col("goals") >= 15, "Top Scorer")
     .when(F.col("goals") >= 5, "Regular Scorer")
     .otherwise("Bench / Support")
)
```

---

### 2.5. Table Joins (`JOIN`)

#### SQL:
```sql
SELECT 
    e.match_id,
    e.player_name,
    e.team,
    f.team_budget_eur
FROM events e
LEFT JOIN financials f ON e.team = f.team_name AND e.season = f.season;
```

#### PySpark DataFrame API:
```python
result_df = events_df.join(
    financials_df,
    (events_df.team == financials_df.team_name) & (events_df.season == financials_df.season),
    how="left"
).select(
    events_df.match_id,
    events_df.player_name,
    events_df.team,
    financials_df.team_budget_eur
)
```

> [!TIP]
> **Performance Tip (Broadcast Join):**
> If the `financials` table is small (e.g., a lookup table of a few hundred rows), use a **Broadcast Join** to avoid expensive network shuffles:
> ```python
> from pyspark.sql.functions import broadcast
> result_df = events_df.join(broadcast(financials_df), "team", "left")
> ```

---

## 3. Common Traps for SQL / PL-SQL Developers in Spark

### Trap 1: Attempting Row-by-Row Processing (Cursors / `FOR` Loops)
- **Bad Practice in Spark:** Iterating through a DataFrame using `for row in df.collect(): ...`
- **Why is it wrong?** Calling `collect()` pulls all data from executor nodes into the Driver node's RAM. This causes an **Out Of Memory (OOM)** crash and destroys parallelism.
- **Solution:** Execute all transformations vectorially across entire columns using `pyspark.sql.functions`.

### Trap 2: Misunderstanding Lazy Evaluation (Transformations vs Actions)
- **Concept:** Spark **does not execute computations** when calling `.select()`, `.filter()`, or `.join()`. It merely builds an optimized logical Execution Plan (DAG).
- **When does Spark actually compute?** Computation is triggered only when an **Action** is invoked (e.g., `.write.parquet(...)`, `.count()`, `.show()`).

### Trap 3: The Small File Problem on Cloud Storage (AWS S3)
- **In SQL / Traditional DWH:** Inserting rows appends data to existing table storage.
- **In Spark + S3:** If you have 200 partitions in memory and call `.write.parquet()`, Spark creates **200 separate Parquet files**. Running this daily results in tens of thousands of tiny files (10 KB each).
- **Solution:** 
  - Use `df.coalesce(N)` before writing to reduce the file count without a shuffle.
  - Use `df.repartition(N)` before writing if you need evenly sized files (target Parquet file size on cloud storage: **128 MB – 256 MB** per file).

---

## 4. Working with Nested Data Structures (JSON / Struct / Array)

In relational SQL, data is usually normalized. In Spark (e.g., reading API payloads, StatsBomb logs, Kafka streams), data frequently arrives as nested JSON objects and arrays.

### Flattening Arrays (`EXPLODE`)
In SQL, you use `UNNEST` or `LATERAL JOIN`. In PySpark, use `explode()`:

```python
from pyspark.sql import functions as F

# Explode lineup array from raw match JSON
flattened_df = raw_json_df.select(
    "match_id",
    "team_name",
    F.explode("tactics.lineup").alias("player_struct")
).select(
    "match_id",
    "team_name",
    F.col("player_struct.player.id").alias("player_id"),
    F.col("player_struct.player.name").alias("player_name"),
    F.col("player_struct.position.name").alias("position")
)
```

---

## 5. Production Best Practices for PySpark Frameworks

1. **Always Specify an Explicit Schema:** Avoid `inferSchema=True` on CSV/JSON files in production. Schema inference requires scanning the whole dataset before reading, causing severe startup latency.
2. **Operation Order:** Always apply `.filter()` and `.select()` early (to reduce data volume in memory) before running `.join()` or `.groupBy()`.
3. **Session Lifecycle Management:** Always release cluster resources in a `finally` block:
   ```python
   spark = SparkSession.builder.getOrCreate()
   try:
       run_pipeline(spark)
   finally:
       spark.stop()
   ```

---

## 6. DWH Orchestration Reference: Legacy Schedulers vs Apache Airflow

For engineers transitioning from traditional relational DWH schedulers (Oracle `DBMS_SCHEDULER`, Control-M, Autosys, SSIS) to cloud orchestration:

| Concept / Capability | Legacy DWH (Control-M / Autosys / PL-SQL) | Apache Airflow equivalent | PySpark / Cloud Lakehouse Context |
| :--- | :--- | :--- | :--- |
| **Pipeline Definition** | Job / Box / Plan defined in GUI or XML | **DAG** (`DAG` in Python file) | Code-defined dependency graph stored in Git repository |
| **Atomic Step** | Job Step / Executable / PL-SQL Proc | **Task / Operator** (`BashOperator`, `SparkSubmitOperator`) | Submits a PySpark job to a cluster or runs a container |
| **Event / File Trigger** | File Watcher job polling an NFS path | **Sensor** (`S3KeySensor`, `ExternalTaskSensor`) | Waits for raw JSON/CSV drops in AWS S3 before spinning up Spark |
| **Task Failure Recovery** | Manual force rerun of entire box | **Clear Task** (in Grid/Graph UI) | Re-executes only the failed step; downstream steps re-trigger automatically |
| **Data Handover** | Staging tables (`STG_...`) or temp files | **Cloud Storage (S3 / Parquet)** | ⚠️ **Never pass large DataFrames via XCom!** Airflow passes only metadata/keys |
| **Automated Retries** | Configured per job attribute | `retries=3`, `retry_delay=timedelta(...)` | Built-in transient network & cluster retry policies |
| **Run Parameterization** | Job variables / Global parameters | **Airflow Variables & Jinja Templating** | Dynamic date parameters: `{{ ds }}`, `{{ ds_nodash }}` passed to config |

### Golden Rules for Airflow + Spark
1. **Separation of Concerns:** Spark is the **compute engine** (heavy lifting in RAM); Airflow is the **conductor** (scheduling, sensing S3, alerting, retries).
2. **Idempotency:** Re-running an Airflow DAG for the same `execution_date` should produce the exact same outcome without duplicating records (use partitioned Parquet overwrite: `.mode("overwrite")`).
3. **No Heavy Work in the DAG:** Never parse files, train models, or manipulate DataFrames directly in the DAG file. The Airflow Scheduler parses DAG files continuously every few seconds—keep DAG definitions lightweight!

