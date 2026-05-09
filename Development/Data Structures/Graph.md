---
aliases:
  - Graph
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Structures]], [[Search]], [[Breadth First Search]], [[Depth First Search]]

---

## Description

A _graph_ ``data structure`` **organizes the relationships between different objects by using vertices and edges.** Vertices are data points "represented" by dots, and edges are lines that connect the vertices.

![[graph.png]]


### Types

![[graph_types.png]]

---

## Examples

1. For example, on a map, the cities would be vertices and the roads that connect them would be edges.
2. On Facebook, users would be vertices and the friendships that connect them would be edges.
3. _Graph_ often used with ``search algorithms`` that seek out data within complex webs of relationships. Common examples include ``breadth-first`` and ``depth-first`` searches.

---

## Pure ``Python`` Representation

```python title:main.py
graph: dict = {
	"A": ["C", "B"],
	"B": ["C", "E", "D"],
	"C": ["E"],
	"D": ["F"],
	"E": ["D", "F"],
	"F": [],
}

```

---

## Claude Sessions
