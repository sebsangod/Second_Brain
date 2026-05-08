---
aliases:
  - Hashed Table
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Estructures]], [[Python]], [[JavaScript]]

---


> [!NOTE]
> In `Python`, a _hashed table_ is equal to a _dictionary_.
> In ``JavaScript`` a _hashed table_ is equal to a _object_.

> [!NOTE] Time complexity
> O(1)

___

## Description

A _hash_ ``data structure``, sometimes called a _hash table_ or _hash map_, uses a _hash function_ **to store data values**. The _hash function_ **creates a hash, which is a unique digital key that corresponds to the location of a specific data value in memory.**

The hash table contains a searchable index of every hash and data value pair, which makes it quick and easy to access, add and remove data from the table.

---

## Details

### Use

_Hash_ ``data structures`` can:
- help quickly retrieve data from phonebooks, dictionaries and personnel directories.
- index databases
- store passwords
- load balance IT systems.

---

## Utils

Count words in a text

```python title:main.py
text: str = "to be or not to be"
count: dict = {}

for word in text:
	count[word] = count.get(word, 0) + 1

print(count) # {"to": 2, "be": 2, "or": 1, "not": 1}

```

---

## Claude Sessions
