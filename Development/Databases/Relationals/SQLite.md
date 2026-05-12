---
aliases:
  - Learning
tags:
  - learning
  - dev/database
date: 2026-05-12
---
**Sources**: [SQLite](https://sqlite.org/)

**Related:** [[Relational Database]], [[C]], [[SQL]]

---

## Description

_SQLite_ is a `C`-language library that implements a [small](https://sqlite.org/footprint.html), [fast](https://sqlite.org/fasterthanfs.html), [self-contained](https://sqlite.org/selfcontained.html), [high-reliability](https://sqlite.org/hirely.html), [full-featured](https://sqlite.org/fullsql.html), `SQL` database _engine_.

_SQLite_ is the [most used](https://sqlite.org/mostdeployed.html) database _engine_ in the world. _SQLite_ is **built into all mobile phones** and most computers and comes bundled inside countless other applications that people use every day.

---

## Snippets

### List all tables in a `database`

```sql title:"Query console"
SELECT name
FROM sqlite_master
WHERE type = "table"
ORDER BY name;
```

---

## Claude Sessions
