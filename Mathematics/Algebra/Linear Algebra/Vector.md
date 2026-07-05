---
aliases:
  - Vector
tags:
  - learning
  - math/algebra/linear
date: 2026-06-20
---
**Sources**: [Curso de Álgebra Lineal: Fundamentos y Aplicaciones](https://platzi.com/cursos/algebra-lineal/)

**Related:** [[Linear Algebra]], [[Python]], [[Array]], [[NumPy]], [[Pythagorean Theorem]]

---

## Description

A _vector_ **could be just a point in a** _n-dimensional_ **space**. The _vector_ $\vec{v} = [2, 3]$ could be a point in the x, y ($R^2$ _dimensional space_) plane while the $\vec{w} = [2, 3, 4]$ could be a point in the x, y, z ($R^3$ _dimensional space_) plane.

Also, the simplest definition of a _vector_ is just to be **an arrow, indicating a direction and a magnitude (its length). This arrows always starts from the plane's origin**.

---

## Key concepts

### Operations

#### Addition

Elements from _vectors_ of the **same n-dimension** can be combined by **adding their entries element by element**. This operation between vectors is known as addition and is denoted by the operator $+$.

Thus, let $a, b, c$ be $n$-vectors in $\mathbb{R}^{n}$ such that $c = a + b$; the operation is illustrated as follows:
$$
c = a+b = \begin{bmatrix}a_{0}\\ a_{1}\\\vdots \\ a_{n-1}\end{bmatrix} + \begin{bmatrix}b_{0}\\ b_{1}\\\vdots \\ b_{n-1}\end{bmatrix} = \begin{bmatrix}a_{0} + b_{0}\\ a_{1}+b_{1}\\\vdots \\ a_{n-1}+b_{n-1}\end{bmatrix}
$$
Example in $R^3$:
$$

\begin{bmatrix} 0\\ 1\\ 2\end{bmatrix} + \begin{bmatrix}2\\ 3\\ -1\end{bmatrix} = \begin{bmatrix}0+2\\ 1+3 \\ 2-1\end{bmatrix} = \begin{bmatrix}2\\ 4 \\ 1\end{bmatrix}

$$
The above is equal to:
$$
( 1 , 2 , 3) + ( -1 ,-2 , -3) = ( 1-1 , 2-2 , 3-3) = ( 0 , 0 , 0) = 0
$$

##### Properties
The addition of vectors is called an *algebraic operation* and satisfies certain properties. Let $\displaystyle\vec{a},\displaystyle\vec{b},\displaystyle\vec{c}$ be vectors in $\mathbb{R}^n$. Then:

* Vector addition is **commutative**: $\vec{a}+\vec{b} = \vec{b}+\vec{a}$
* Vector addition is **associative**: $(\vec{a}+\vec{b})+\vec{c} = \vec{a}+(\vec{b}+\vec{c})$. Because of this, we can freely write $\vec{a}+\vec{b}+\vec{c}$ since **the result will not change depending on which pair of vectors we add first**.
* When one vector is the zero vector, adding it to another vector has no effect on the other vector: $\vec{a}+\vec{0}=\vec{0}+\vec{a}=\vec{a}$
* Adding a vector to its inverse -or, equivalently, subtracting two identical vectors—results in the zero vector: $\vec{a}-\vec{a}=\vec{0}$


##### Example in ``Python``

A _vector_ in ``Python`` is represented as a simple _list_ (``array``). Thus we cannot just do a normal ``Python`` ``addition`` because that would result in a _list concatenation_:

```python title:main.py
a: list[int] = [1, 2, 3]
b: list[int] = [4, 5, 6]

c: list[int] = a + b
print(c)  # [1, 2, 3, 4, 5, 6]

```

To solve this, we need to use ``NumPy``:

```python title:main.py
from numpy import array, ndarray


a: ndarray = array([1, 2, 3])
b: ndarray = array([4, 5, 6])

c: ndarray = a + b
print(c)  # [5 7 9]

```


#### Subtraction

The subtraction of _vectors_ follows the same process than its addition.
Let $\displaystyle\vec{a}$ and $\displaystyle\vec{b}$ be vectors in $\mathbb{R}^n$. Then the subtraction will be the sum of the negative of $\displaystyle\vec{b}$ ($-\displaystyle\vec{b}$):
$$\vec{a}-\vec{b} = \vec{a}+(-\vec{b})$$
##### Properties
The subtraction also shares the same properties of the addition, except that this is not **commutative**: $\vec{a}-\vec{b} \neq \vec{b}-\vec{a}$


#### Scaling

Also known as _Scalar_ **product or** _scalar_-_vector_ **multiplication**.

In this operation a vector is multiplied by a _scalar_. The operation is performed element by element. Let $\vec{v}$ be a vector in $\mathbb{R}^{n}$ and $\alpha \in \mathbb{R}$. If $\vec{x} = \alpha \vec{v}$, then:

$$
\vec{x} = \alpha \vec{v} = \alpha\begin{bmatrix}v_{0}\\ v_{1}\\ \vdots \\ v_{n-1}\end{bmatrix} = \begin{bmatrix}\alpha \cdot v_{0}\\ \alpha \cdot v_{1}\\ \vdots \\ \alpha \cdot v_{n-1}\end{bmatrix}
$$
For example, if $\vec{v} = (0,1,-2.3)$ and $\alpha = -1.1$ then $\alpha \vec{v} = (0,-1.1,2.53)$


##### Properties
Let $\vec{x}$ and $\vec{y}$ be two vectors and $\alpha$ and $\beta$ be any two scalars. The _scalar-vector_ product satisfies the following properties:

* **Commutativity**: $\alpha \vec{x} = \vec{x}\alpha$
* **Associativity**: $(\beta \alpha)\vec{x} = \beta (\alpha\vec{x})$
* **Distributivity over scalar addition**: $(\alpha + \beta)\vec{x} = \alpha \vec{x} + \beta\vec{x} =\beta\vec{x} + \alpha\vec{x} = \vec{x}(\alpha + \beta)$
* **Distributive property with respect to vector addition**: $\alpha (\vec{x}+\vec{y}) = \alpha\vec{x} + \alpha\vec{y}$


##### Example in `Python`

```python title:main.py
from numpy import array, ndarray


x: ndarray = array([-1, 1, -1])
a: float = 3.1

y: ndarray = x * a
print(y)  # [-3.1  3.1  -3.1]

```


##### Linear Combinations

Let $\vec{a}_{0},\vec{a}_{1},\dots,\vec{a}_{m-1}$ be _n-vectors_ and $\beta_{0},\beta_{1},\dots,\beta_{m-1}$ be _scalars_; then we can define the following _n-vector_:
$$
\beta_{0}\vec{a}_{0}+\beta_{1}\vec{a}_{1}+\cdots+\beta_{m-1}\vec{a}_{m-1}
$$

This is called a *linear combination* of the vectors $\vec{a}_{0},\vec{a}_{1},\dots,\vec{a}_{m-1}$.

The scalars $\beta_{0},\beta_{1},\dots,\beta_{m-1}$ are called the *coefficients* of the _linear combination_.


###### Linear Combinations of Unit Vectors

Any $\vec{b}$ $n$-vector can be written as a linear combination of the standard unit vectors:

$$
\vec{b} = b_{0}\hat{e}_{0}+b_{1}\hat{e}_{1} + \cdots +b_{n-1}\hat{e}_{n-1}
$$

Where $b_{i}$ are scalars and $\hat{e}_{i}$ is the $i$-th unit vector. A specific example would be:
$$

\begin{bmatrix}1\\ -1\\ 0\end{bmatrix} = (1)\begin{bmatrix}1\\ 0\\ 0\end{bmatrix} + (-1)\begin{bmatrix}0\\ 1\\ 0\end{bmatrix} + (0)\begin{bmatrix}0\\ 0\\ 1\end {bmatrix}

$$

As a final note here, observe that if the vector space has dimension $n$, then it has $n$ unit vectors $\hat{e}_{i} \ (\hat{i}, \ \hat{j}, \ \hat{k}, \ \hat{l}, \ ... \ )$


###### Linear Dependency

###### Linear Independency


#### Dot Product

The *inner product* (standard) or simply *dot product* of two _n-vectors_ $\vec{a}$ and $\vec{b}$ is defined as the _scalar_:
$$
<\vec{a},\vec{b}> = \vec{a} \cdot \vec{b} = a_{0}b_{0}+a_{1}b_{1}+\cdots+a_{n-1}b_{n-1} = \displaystyle\sum_{i=0}^{n-1} a_{i}b_{i}
$$

**Which is simply the sum of the products of their components.**

For example, if $\vec{a}=(1,2,3)$ and $\vec{b}=(4,5,6)$, then the dot product is:
$$
\vec{a}\cdot\vec{b} = (1,2,3)\cdot (4,5,6) = 1\cdot 4 + 2\cdot 5 + 3\cdot 6 = 4 + 10 + 18 = 32
$$

##### Transposition

Let's return to the vector $\vec{a}$; we will denote the _transpose_ of $\vec{a}$ as $\vec{a}^{T}$:
$$
\vec{a}^{T} = \begin{bmatrix}a_{0}\\ a_{1}\\ \vdots \\ a_{n-1}\end{bmatrix}^{T} = [a_{0}\; a_{1}\; \cdots \; a_{n-1}]
$$

The _transposition_ operation allows us to **convert column vectors into row vectors without altering the vector being transposed**. Note that it is possible to transpose a transposed vector:
$$
\left(\vec{a}^{T}\right)^{T} = \left(\begin{bmatrix}a_{0}\\ a_{1}\\ \vdots \\ a_{n-1}\end{bmatrix}^{T}\right)^{T} = [a_{0}\; a_{1}\; \cdots \; a_{n-1}]^{T} = \begin{bmatrix}a_{0}\\ a_{1}\\ \vdots \\ a_{n-1}\end{bmatrix} = \vec{a}
$$


##### Properties

Let $\vec{a}$ and $\vec{b}$ be two _n-vectors_ and $\alpha$ a _scalar_. The inner product between $\vec{a}$ and $\vec{b}$ satisfies the following properties: 

* **Commutativity**: $\vec{a}^{T}\vec{b} = \displaystyle\sum_{i=0}^{n-1} a_{i}b_{i} = \displaystyle\sum_{i=0}^{n-1} b_{i}a_{i} = \vec{b}^{T}\vec{a}$

* **Associativity** with _scalar multiplication_: $\left(\alpha \vec{a}\right)^{T}\vec{b} = \alpha\left(\vec{a}^{T}\vec{b}\right)$

* **Distributivity** over _vector addition_: $\left(\vec{a} + \vec{b}\right)^{T}\vec{c} = \vec{a}^{T}\vec{c} + \vec{b}^{T}\vec{c}$


#### Norm

The _norm_ determines the length of a _vector_ using the `Pythagorean Theorem` because this theorem is used to calculate the distance between two points.

##### Euclidean Norm (L2)

This _norm_ gives us the distance in straight line from two points. This norm is denotated and calculated as:

$$ ||\vec{v}|| = \sqrt{x^2+y^2}$$


##### Properties

- The _norm_ is **always positive**: $||\vec{v}|| \ \geq \ 0$
- **Absolute homogeneity**: $||α \ \vec{v}|| = |α| \ ||\vec{v}||$


#### Cosine

Applying the ``cosine`` function to two _vector_ is the way to calculate the exact angle they form between each other.

The cosine function comes from another formula of the _dot product_:
$$u \cdot v = ||\vec{u}|| \ ||\vec{v}|| \ cos(θ)$$

Solving for the ``cosine`` we can get:
$$cos(θ) = \frac{u \cdot v}{||\vec{u}|| \ ||\vec{v}||}$$
$$θ = \arccos{\left( \frac{u \cdot v}{||\vec{u}|| \ ||\vec{v}||} \right) }$$

The above formula can lead us to three possible results:
- 1: 0° angle. The _vectors_ are perfectly aligned
- 0: 90° angle. The _vectors_ are _orthogonals_.
- -1: 180° angle. The _vectors_ are completely opposites.

---

### Use case

Write here...

```python title:main.py
print("Hello world!")

```

---

## Claude Sessions
