---
aliases:
  - Sets
tags:
  - learning
  - math/discretes
date: 2026-05-13
---
**Sources**: [Curso de Lenguaje y Notación Matemática](https://platzi.com/cursos/notacion-matematica/), [Union of sets](https://en.flamath.com/union-of-sets)

**Related:** [[Discrete Mathematics]], [[Algebra]]

---

## Description

A _set_ is a **collection of elements considered in and of itself as a element**

### Examples:

- Vowels = {a, e, i, o, u}
- Days = {Sunday, Monday, Tuesday, Wednesday, Thursday, Friday}
- Z = {..., -3, -2, -1, 0, 1, 2, 3, ...}

---

## Details

### Operations

#### Union

Formally, we define the _union of two sets_ A and B as the **set formed by the elements that belong to A, to B, or to both**. It is denoted as **A ∪ B** and expressed in set-builder notation as:

$$A \ ∪ \ B = \{ \ x \ | \ x \ ∈ \ A \ ∨ \ x ∈ \ B \ \}$$

This could also be defined as **the sum of 2 or more sets**.

![[set_union.png]]


#### Intersection

Formally, the _intersection of two sets_ A and B is defined as **the set of all elements that belong to both A and B**. It is denoted as **A ∩ B** and is expressed in set-builder notation as follows:

$$A \ ∩ \ B = \{ \ x \ | \ x \ ∈ \ A \ ∧ \ x ∈ \ B \ \}$$

This could also be defined as **the multiplication of 2 or more sets**.

![[set_intersection.png]]


#### Difference

Formally, the _difference between two sets_ A and B is defined as the **set of all elements that are in A, but not in B**. It is denoted as **A - B** and is expressed in set-builder notation as follows:

$$A \ - \ B = \{ \ x \ | \ x \ ∈ \ A \ ∧ \ x ∉ \ B \ \}$$

This could also be defined as **the difference between 2 or more sets**.

![[set_difference.png]]


#### Symmetric Difference

Formally, the _symmetric difference_ of two sets A and B is denoted as **A Δ B** and is expressed in set-builder notation as follows:

$$A ∆ B = \{ \ x \ | \ x \ ∈ \ A \ - \ B \ ∨ \ x \ ∈ \ B \ - \ A \}$$
$$A ∆ B = (A -  B) \ ∪ \ (B - A)$$

![[set_symmetric_difference.png]]


#### Complement

Formally, if we have a universal set U and a subset A ⊆ U, the _complement of A_ is defined as the **set of all elements of U that are not in A**. It is denoted as _A′_ or _A^c_ and expressed in set-builder notation as follows:

$$A' = A^c = \{ \ x \ | \ x \ ∈ \ U \ ∧ \ x ∉ \ A \ \}$$

![[set_complement.png]]


### Sets' Algebra

#### Idempotency
$$A \ U \ A \ = \ A$$
$$A \ ∩ \ A \ = \ A$$


#### Commutative
$$A \ U \ B \ = \ B \ U \ A$$
$$A \ ∩ \ B \ = \ B \ ∩ \ A$$


#### Associative
$$(A \ U \ B) \ U \ C \ = \ A \ U \ (B \ U \ C) $$
$$(A \ ∩ \ B) \ ∩ \ C \ = \ A \ ∩ \ (B \ ∩ \ C) $$


#### Distributive
$$(A \ U \ B) \ ∩ \ C \ = \ (A \ U \ B) \ ∩ \ (A \ U \ C) $$
$$(A \ ∩ \ B) \ U \ C \ = \ (A \ ∩ \ B) \ U \ (A \ ∩ \ C) $$

---

## Claude Sessions
