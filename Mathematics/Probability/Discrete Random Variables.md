---
aliases:
  - Discrete Random Variables
tags:
  - learning
  - math/probability
date: 2026-08-09
---
**Sources**: [Curso de Estadística y Probabilidad](https://platzi.com/cursos/estadistica-probabilidad/), [Better Explained](https://betterexplained.com/articles/an-intuitive-and-short-explanation-of-bayes-theorem/)

**Related:** [[Probability]]

---

## Key concepts

Write here...

---

## Details

### Permutation

A _permutation_ is a _combination_ **of elements in which the order matters**. If you swap just one element, you already have a different _permutation_.

The formula tells you **how many elements** $K$ **can be arranged within a set of size** $N$:
$$_n P_k \ = \frac{n!}{(n-k)!}$$

### Combination
A _combination_ is the grouping of certain elements within a set, **regardless of the order** in which they appear.

The key point here is that order does not matter. That is why there are almost always fewer combinations than permutations within the same set.

The formula for combining $K$ elements from a set of $N$ is calculated as follows:
$$_n C_k \ = \frac{n!}{k! * (n-k)!}$$


### What is the difference?
In a _permutation_, order matters; in a _combination_, it does not. That is why the same grouping of elements counts as several permutations but as a single combination.


### When to use them?

The decision depends on a simple question: **Does the order change the result?**

- If the order matters (passwords, podium finishes, sequences), use permutations.
- If the order doesn't matter (teams, groups, hands of cards), use combinations.
- If you're not sure, consider whether rearranging the same elements creates a new case or not.

---

## Claude Sessions
