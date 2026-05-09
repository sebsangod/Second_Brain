---
aliases:
  - Tree
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Structures]], [[Binary Search]], [[AVL Tree]], [[B-Tree]], [[Decision Tree]], [[Machine Learning]], [[Python]], [[Queue]]

---

> [!TIP]
> Use this ``structure`` when you need to **read more elements than insert them**

> [!NOTE] Time complexity
> O(n)

___

## Description

A _tree_ ``data structure``, sometimes called a _prefix tree_, **is useful for establishing hierarchical relationships among data elements.**

There are different classes of _trees_, such as ``binary search trees``, ``AVL trees`` and ``b-trees``, and all have different properties and support different functions.****

> [!TIP]
> Use ``treelib`` ``python`` **library ready-to-use tree implementations**

---

## Key concepts

A single parent node sits on the top of the _tree structure_, with child subnodes branching out on subsequent levels beneath it.

![[tree.png]]

---

## Use

_Trees_ are often used to **represent hierarchies in organizational maps**, **file systems**, **domain name systems**, **database indexing** and ``decision trees`` in ``machine learning`` applications.

___

## Pure ``Python`` Implementation

### Key design points

| **Component** | **Details**                                                    |
| ------------- | -------------------------------------------------------------- |
| Node          | Holds a value and a list of _children_ (**n-ary**, not binary) |
| Tree          | Wraps _root_. All operations go through it                     |
| find()        | DFS search, returns _Node_ or _None_                           |
| insert()      | Finds _parent_ by value, appends new _child_                   |
| delete()      | Removes a _node_ (and its entire _subtree_)                    |
| height()      | Recursive max _depth_ from any _node_                          |
| size()        | Total _node_ count                                             |
| bfs()         | Level-order using a ``queue``                                  |
| dfs()         | Pre-order (_root_ -> _children_ left-to-right)                 |
| display()     | Pretty-prints with _tree_-drawing characters                   |

### Implementation

```python title:main.py
class Node:
    def __init__(self, value):
        self.value = value
        self.children = []

    def add_child(self, node):
        self.children.append(node)
        return self

    def remove_child(self, value):
        self.children = [c for c in self.children if c.value != value]

    def is_leaf(self):
        return len(self.children) == 0


class Tree:
    def __init__(self, root_value):
        self.root = Node(root_value)

    def _find(self, node, value):
        if node.value == value:
            return node
        for child in node.children:
            result = self._find(child, value)
            if result:
                return result
        return None

    def find(self, value):
        return self._find(self.root, value)

    def insert(self, parent_value, child_value):
        parent = self.find(parent_value)
        if not parent:
            raise ValueError(f"Parent '{parent_value}' not found")
        parent.add_child(Node(child_value))

    def _delete(self, node, value):
        for child in node.children:
            if child.value == value:
                node.children.remove(child)
                return True
            if self._delete(child, value):
                return True
        return False

    def delete(self, value):
        if self.root.value == value:
            raise ValueError("Cannot delete root")
        self._delete(self.root, value)

    def height(self, node=None):
        node = node or self.root
        if node.is_leaf():
            return 0
        return 1 + max(self.height(c) for c in node.children)

    def size(self, node=None):
        node = node or self.root
        return 1 + sum(self.size(c) for c in node.children)

    def bfs(self):
        """Breadth-first traversal."""
        result, queue = [], [self.root]
        while queue:
            node = queue.pop(0)
            result.append(node.value)
            queue.extend(node.children)
        return result

    def dfs(self, node=None):
        """Depth-first (pre-order) traversal."""
        node = node or self.root
        result = [node.value]
        for child in node.children:
            result.extend(self.dfs(child))
        return result

    def display(self, node=None, prefix="", is_last=True):
        node = node or self.root
        connector = "└── " if is_last else "├── "
        print(prefix + (connector if prefix else "") + str(node.value))
        child_prefix = prefix + ("    " if is_last else "│   ")
        for i, child in enumerate(node.children):
            self.display(child, child_prefix, i == len(node.children) - 1)


if __name__ == "__main__":
    t = Tree("root")
    for child in ["A", "B", "C"]:
        t.insert("root", child)
    for child in ["A1", "A2"]:
        t.insert("A", child)
    for child in ["B1", "B2", "B3"]:
        t.insert("B", child)
    t.insert("A1", "A1a")

    print("Tree structure:")
    t.display()

    print(f"\nSize:   {t.size()}")
    print(f"Height: {t.height()}")
    print(f"BFS:    {t.bfs()}")
    print(f"DFS:    {t.dfs()}")

    t.delete("B2")
    print("\nAfter deleting B2:")
    t.display()

```


### Output

```bash title:output
Tree structure:
root
├── A
│   ├── A1
│   │   └── A1a
│   └── A2
├── B
│   ├── B1
│   ├── B2
│   └── B3
└── C

Size:   9
Height: 3
BFS:    ['root', 'A', 'B', 'C', 'A1', 'A2', 'B1', 'B2', 'B3', 'A1a']
DFS:    ['root', 'A', 'A1', 'A1a', 'A2', 'B', 'B1', 'B2', 'B3', 'C']

After deleting B2:
root
├── A
│   ├── A1
│   │   └── A1a
│   └── A2
├── B
│   ├── B1
│   └── B3
└── C
```

---

## Claude Sessions
