---
aliases:
  - Array
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Structures]], [[Queue]], [[Stack]]

---

> [!TIP]
> Use this ``structure`` when you need to **read more elements than insert them**

> [!NOTE] Time complexity
> O(n)

___

## Description

_Arrays_ are one of the most basic and widely used types of `data structures`.

**They store data items of a similar type at adjacent memory locations.** This structure enables items of the same type to be easily located and accessed.

---

## Details

When you need to access to the Nth element of an _array_, the computer retrieves it almost **instantly**. This is because of the _array's_ usage formula:

$$value = initialMemoryAddress + (searchIndex * Element'sSize)$$

this allows us to retrieve any data in **just 1 operation**, no matter the size of the _array_.

> [!WARNING]
> Retrieve any element is fast, but **inserting one element** in the middle of the array **IS NOT**.
> 
> This is because the computer needs to move every element of the right of the index of the new element one memory address to the right to keep all elements together.

_Arrays_ can also be used as a foundation for implementing other `data structures`, such as `queues` and `stacks`.


### Uses

Common uses for _arrays_ include
- sorting
- searching and accessing data
- scores in videogames
- product catalogs
- data tables

---

## Claude Sessions
