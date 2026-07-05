---
aliases:
  - Matrix
tags:
  - learning
  - math/algebra
date: 2026-06-03
---
**Sources**: [Curso de álgebra y funciones](https://platzi.com/cursos/algebra/partes-de-una-expresion-algebraica-varia/), [Crammer's Rule](https://www.datacamp.com/es/tutorial/cramers-rule)

**Related:** [[Equation]], [[Development/Data Structures/Matrix|Matrix]], [[Linear Equation]], [[Vector]]

---

## Description

A matrix is the mathematical name for a array of elements, positioned alongside in rows and cols.

![[Images/Mathematics/Algebra/matrix.png]]

**The first subindex of a elements represents the row of that element, while the second subindex represents the column the element belongs to.**

---

## Operations

### Addition and Subtraction

![[matrix_addition_subtration.png]]


### Scalar Multiplication

![[matrix_scalar_multiplication.png]]


### Determinant

![[matrix_determinant_2.png]]

---

## Solve for Linear Equations

### Gauss - Jordan Method

This method is limited to the following operations rules:

1. Rows additions
2. Rows exchanges
3. Rows multiplicated by a scalar greater than 0

![[matrix_gauss_jordan.png]]

### The Crammer's Rule

For a general _n×n system_, _Cramer's rule_ states that the _i-th_ component of the solution _vector_ is:

![[matrix_crammers_rule.png]]

Where:

- $x_{i}$ is the _i-th_ _variable_ we are solving for.
- det($A_{i}$) is the determinant of matrix A with its _i-th_ column replaced by ``vector`` b.
- det(A) is the determinant of the original coefficient _matrix_.

#### Example

![[matrix_crammers_rule_for_x.png]]

![[matrix_crammers_rule_for_y.png]]

---

## Claude Sessions
