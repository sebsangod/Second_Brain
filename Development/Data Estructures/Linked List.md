---
aliases:
  - Linked List
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Estructures]], [[Memory Reference]], [[Python]]

---

> [!TIP]
> Use this ``structure`` when you need to **insert or delete more elements than retrieve them**

> [!NOTE] Time complexity
> O(1)

---

## Description

_Linked lists_ **store data items in a linear order, with each item connected to the next item in the list.**

This structure makes it **easy to insert new items or delete existing items** without having to shift the entire collection of data.

![[linked_list.png]]

---

## Details

> [!WARNING]
> Retrieve any node's value is too slow because the computer needs to read every node one by one until the target node.

### Use
_Linked lists_ are often used for **frequent insertions and deletions in scenarios**, such as:
- web browser histories
- media player playlists
- undo or redo operations in applications.

---

## ``Python`` Implementation

```python title:main.py
class Node:
	def __init__(self, value):
		self.value = value
		self.next = None


class LinkedList:
	def __init__(self):
		self.head = None

	def add_at_beginning(self, value):
		new = Node(value)
		new.next = self.head
		self.head = new

```

---

## Claude Sessions
