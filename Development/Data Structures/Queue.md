---
aliases:
  - Queue
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Structures]], [[Python]]

---

> [!TIP]
> Use this ``structure`` when you need to **track changes**

> [!NOTE] Time complexity
> O(1)

___

## Description

A _queue_ ``data structure`` performs **data operations in a predetermined order called** _FIFO_ **for “first in, first out.”**

This means that **the first data item to be added will be the first to be removed.** Programmers often use this ``data structure`` to **create priority queues**, which are similar to **waiting lists**.

---

## Details

A _queue_ allows only two operations:

- **Add to the end**
- **Remove from the beginning**

### Uses

_Queue_ ``data structures`` can be used to:
- determine the next song in a playlist
- the next user to have access to a shared printer
- the next call to be answered in a call center
- order to send messages when there is Internet connection again

---

## Examples

Customers waiting to speak to a call center representative might be placed in a queue like this:

```python title:main.py
queue = [customer_1, customer_2, customer_3]

```

When a representative is available, they automatically connect with the first customer in the queue, who is then removed from the list. Now, the queue looks like this:

```python title:main.py
queue = [customer_2, customer_3]

```

---

## ``Python`` Implementation

```python title:main.py
from collections import deque

queue = deque()

queue.append("Fernanda")
queue.append("Ramirez")
queue.append("Verdeja")

print(queue) # deque(["Fernanda", "Ramirez", "Verdeja"])

next = queue.popleft()

print(next) # "Fernanda"
print(queue) # deque(["Ramirez", "Verdeja"])

```

---

## Claude Sessions
