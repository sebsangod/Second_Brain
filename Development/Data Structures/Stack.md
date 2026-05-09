---
aliases:
  - Stack
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure), [Las 8 Estructuras de Datos que TODO Programador Usa](https://www.youtube.com/watch?v=9ifwAPFxpu0)

**Related:** [[Data Structures]], [[Queue]]

---

> [!TIP]
> Use this ``structure`` when you need to **track changes**

> [!NOTE] Time complexity
> O(1)

___

## Description

Similar to ``queues``, a _stack_ ``data structure`` performs data operations in a predetermined order. However, instead of FIFO, _stacks_ **use the** _LIFO_ **format**, which stands for **“last in, first out.” The last data item to be added will be the first to be removed.**

---

## Details

A _stack_ allows only two operations:

- **Push:** Add a new element at the top
- **Pop:** Remove the top element


### Use

_Stacks_ can be used to:
- help ensure the correct opening and closing of brackets or tags in computer codes
- track recent browser history
- undo recent operations in an application

---

## Examples

Many apps use _stacks_ to keep track of user actions so they can easily be undone. For example, a text editor might keep a stack that looks like this:

```python title:main.py
recent_actions = [typing ".", space, typing "T"]

```

When a user hits the “undo” button, the most recent action in the _stack_ (“typing ‘T’”) is undone. Now, the stack looks like this:

```python title:main.py
recent_actions = [typing ".", space]

```

---

## Claude Sessions
