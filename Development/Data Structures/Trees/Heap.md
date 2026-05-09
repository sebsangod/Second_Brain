---
aliases:
  - Heap
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [Geek for geeks](https://www.geeksforgeeks.org/dsa/heap-data-structure/), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Structures]], [[Binary Tree]], [[Queue]], [[Python]]

---

> [!NOTE] Time complexity
> O(log n)

___

## Description

A _heap_ is a complete ``binary tree`` ``data structure`` that satisfies the **heap property**:

- in a _min-heap_, the value of **each child is greater than or equal to its parent**

- and in a _max-heap_, the value of **each child is less than or equal to its parent**

_Heaps_ are commonly used to implement priority ``queues``, where the smallest (or largest) element is **always at the root**.

> [!NOTE]
> When you insert or remove a new value into the _heap_, this last one will **automatically re-order itself to respect min or max heap property.**

___

## Examples

### Min _Heap_

![[valid_min_heaps.png]]

![[invalid_min_heaps.png]]


### Max _Heap_

![[valid_max_heaps.png]]

![[invalid_max_heaps.png]]

---

## Pure ``Python`` Implementation

```python title:main.py
from heapq import

# min-heap priority queue
heap = []

# Unordered element insertions
heapq.headpush(heap, (3, "headache"))
heapq.headpush(heap, (1, "heart attack"))
heapq.headpush(heap, (2, "fracture"))
heapq.headpush(heap, (5, "cold"))

while heap:
	print(heapq.heappop(heap))

```

```bash title:output
(1, "heart attack")
(2, "fracture")
(3, "headache")
(5, "cold")
```

---

## Claude Sessions
