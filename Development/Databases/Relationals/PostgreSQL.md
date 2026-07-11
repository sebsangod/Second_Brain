---
aliases:
  - PostgreSQL
tags:
  - learning
  - dev/database
date: 2026-03-15
---
**Sources**: [PostgreSQL](https://www.postgresql.org/)

**Related:** [[Relational Database]], [[Docker]]

---

## Description

_PostgreSQL_ is a powerful, **open source** ``object-relational database`` system with over 35 years of active development that has earned it a strong reputation for **reliability, feature robustness, and performance**.

There is a wealth of information to be found describing how to [install](https://www.postgresql.org/download/) and [use](https://www.postgresql.org/docs/) _PostgreSQL_ through the [official documentation](https://www.postgresql.org/docs/). The [open source community](https://www.postgresql.org/community/) provides many helpful places to become familiar with PostgreSQL, discover how it works, and find career opportunities. Learn more on how to [engage with the community](https://www.postgresql.org/community/).

---

## Key concepts

Write here...

---

## Details

### Installation

Write here...

---

## PSQL Commands

```bash title:bash
# List all tables within a database's public squema
\dt
# List all tables of all squemas within a database
\dt *.*

# List all fields within a specific table
\d+ table_name
```

---

## Snippets

```sql title:"List all tables of a database"
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
  AND table_schema NOT IN ('pg_catalog', 'information_schema')
ORDER BY table_schema, table_name;
```

```sql title:"List all fields within a specific table"
SELECT 
    column_name,
    data_type,
    is_nullable,
    column_default
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name   = 'table';
```

___

## Utils

### `Docker` Execution

```bash title:bash
docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=root -d postgres:latest
```

---

## Claude Sessions
