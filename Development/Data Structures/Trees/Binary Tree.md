---
aliases:
  - Binary Tree
tags:
  - learning
  - dev/data
date: 2026-05-08
---
**Sources**: [Geeks for geeks](https://www.geeksforgeeks.org/dsa/binary-tree-data-structure/)

**Related:** [[Data Structures]], [[Tree]], [[Python]]

---

> [!NOTE] Time complexity
> O(n)

___

## Description

A _binary tree_ ``data structure`` is a **hierarchical data structure** in which **each node has at most two children**, referred to as the left child and the right child:

![[binary_tree.png]]

___

## Pure ``Python`` Implementation

```python title:main.py
class Node:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None

class BinaryTree:
	def __init__(self):
        self.root = None

	def _insert(self, node, value):
		if node is None:
			return Node(value)
		if value < node.value:
			node.left = self._insert(node.left, value)
		else:
			node.right = self._insert(node.right, value)
		return node

	def insert(self, value):
		self.root = self._insert(self.root, value)


if __name__ == "__main__":
	b_tree = BinaryTree()
	for v in [50, 30, 70, 20, 40, 60, 80]:
		tree.insert(v)

```

---

## Claude Sessions
