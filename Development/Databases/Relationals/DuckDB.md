---
aliases:
  - DuckDB
tags:
  - learning
  - dev/database
date: 2026-05-08
---
**Sources**: [duckdb](https://duckdb.org/), [docs](https://duckdb.org/docs/current/)

**Related:** [[Relational Database]], [[SQLite]], [[OLAP]], [[C++]], [[Pandas]], [[DataFrame]], [[Python]], [[Spark]], [[BigQuery]], [[ETL]]

---

## Description

_DuckDB_ is an **embedded analytical database** — think of it as **"**``SQLite``**, but for analytics."**

- An in-process ``OLAP`` _(Online Analytical Processing)_ database engine
- Open-source, written in ``C++``
- Runs inside your application — no separate server needed

---

## Key strengths

- Blazing fast on analytical queries (aggregations, joins, scans over large datasets)
- Reads directly from Parquet, CSV, JSON, and Arrow files without importing them first
- Works seamlessly with ``pandas`` ``DataFrames`` and other ``Python`` data tools
- Zero-dependency setup — just _pip install duckdb_

---

## Details

### Common Use Cases

- Querying large CSV/Parquet files with ``SQL``
- Local data analysis without spinning up a data warehouse
- As a fast local alternative to ``Spark`` or ``BigQuery`` for moderate data sizes
- ``ETL`` pipelines and data engineering workflows


### How it compares

|DuckDB|SQLite|Postgres|
|---|---|---|---|
|Best for|Analytics|Small apps|OLTP/web apps|
|Server needed|No|No|Yes|
|Column storage|Yes|No|Optional|
|Speed on big scans|Very fast|Slow|Medium|

It's become a go-to tool in the data engineering and data science world, especially for anyone who wants SQL-powered analytics locally without the overhead of a full data warehouse.

---

## Examples

```python title:duck_db.py
import duckdb

# Query a Parquet file directly — no loading needed
duckdb.sql("SELECT region, SUM(sales) FROM 'data.parquet' GROUP BY region")

# Or query a pandas DataFrame
duckdb.sql("SELECT * FROM my_df WHERE amount > 1000")
```

---

## Claude Sessions
