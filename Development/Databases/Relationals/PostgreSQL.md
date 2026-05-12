---
aliases:
  - PostgreSQL
tags:
  - learning
  - dev/database
date: 2026-03-15
---
**Sources**: [PostgreSQL](https://www.postgresql.org/)

**Related:** [[Relational Database]]

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

### List all fields within a specific table

```bash title:bash
\d+ table_name
```

---

## Snippets

### List all fields within a specific table

```sql title:"Query console"
SELECT 
    column_name,
    data_type,
    is_nullable,
    column_default
FROM information_schema.columns
WHERE table_schema = 'public'
  AND table_name   = 'table';
```

---

## Claude Sessions
