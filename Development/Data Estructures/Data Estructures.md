---
aliases:
  - Data Structures
tags:
  - learning
  - dev/data
date: 2026-05-07
---
**Sources**: [IBM](https://www.ibm.com/think/topics/data-structure)

**Related:** [[Development]], [[Array]], [[Linked List]], [[Queue]], [[Tree]], [[Graph]], [[Python]], [[Rust]], [[JavaScript]]

---

## Description

A data structure is a way of formatting data so that it can be used by a computer program or other system.

Data structures are a fundamental component of computer science because they give form to abstract [data](https://www.ibm.com/think/topics/data) points. In this way, **they allow users and systems to efficiently organize, work with and store data.**

Data structures **combine primitive data types** such as numbers, characters, booleans and integers into a cohesive format. Alone, each of these primitive data types possesses only a single value. When they are combined in a data structure, they **enable higher-level data operations such as sorting, searching, insertion and deletion.**

---

## Key concepts

Data structures are important because **they make it easier for computers to process large, complex sets of information.**

By logically organizing data elements, data structures increase the efficiency of computer code and make the code simpler to understand.

---

## Details

### Types

Data structures are divided into **2 main categories**: linear and nonlinear.

#### Linear data structures

In a _linear data structure_, **data is arranged in a line, with each data element placed one after the other in sequence.** This arrangement makes it simple to traverse and access the elements in order.

_Linear data structures_ are considered straightforward and simple to implement. Common data structures in this category include `arrays`, `linked lists` and `queues`.


#### Nonlinear data structures

In a _nonlinear data structure_, the organizational logic is something other than a linear, sequential arrangement. For example, **data points can be hierarchically ordered or connected in a network.**

Because they are not connected to each other in a single line, **the elements in a** _nonlinear structure_ **cannot all be traversed and accessed in a single run, as they can in a** _linear data structure_. Examples of nonlinear data structures include `trees` and `graphs`.

---

## Use Cases

_Data structures_ **are critical in designing software applications because they implement the concrete forms of** _abstract data types_.

An _abstract data type_ is a mathematical model that classifies how a data type behaves and the operations that can be performed on it.

For example, the abstract data type of a `queue` defines the queue’s behavior (following the principle of FIFO). The queue data structure provides a way to format data into a queue, such that a computer program applies the FIFO principle to that data.

Many programming languages, such as `Python`, `Rust` and `JavaScript`, include built-in _data structures_ to help developers work more efficiently.

Common use cases for _data structures_ in computer programs include:

- Data storage and organization
- Indexing
- Data exchange
- Searching
- Scalability

___

## Examples

### Slows

```python title:slow.py
from time import time

numbers: list = list(range(100_000)) + [99_999]
start = time()

has_duplicated: bool = False
for i in range(len(numbers)):
	for j in range(i + 1, len(numbers)):
		if numbers[i] == numbers[j]:
			has_duplicated = True
			break
	if has_duplicated:
		break

print(f"Has duplicated?: {has_duplicated}")
print(f"Time: {time() - start:.2f}s")

```

```bash title:bash
python slow.py
# Duplicated: True
# Time: 3.14s

```

```javascript title:slow.js
const numbers = Array.from({length: 100000}, (_, i) => i);
numbers.push(99999);

const start = Date.now();

let hasDuplicated = false;
for (let i = 0; i < numbers.length; i++) {
	for (let j = i + 1; j < numbers.length; j++) {
		if (numbers[i] === numbers[j]) {
			hasDuplicated = true;
			break;
		}
	}
	if (hasDuplicated) break;
}

console.log(`Duplicated: ${hasDuplicated}`);
console.log(`Time: ${Date.now() - start}mss`);

```

```bash title:bash
node slow.js
# Duplicated: true
# Time: 3140ms

```


### Fasts

```python title:slow.py
from time import time

numbers: list = list(range(100_000)) + [99_999]
start = time()

seen = set()
has_duplicated: bool = False
for n in numbers:
	if n in seen:
		has_duplicated = True
		break
	seen.add(n)

print(f"Has duplicated?: {has_duplicated}")
print(f"Time: {(time() - start) * 1_000:.2f}ms")

```

```bash title:bash
python slow.py
# Duplicated: True
# Time: 3.12ms

```

```javascript title:slow.js
const numbers = Array.from({length: 100000}, (_, i) => i);
numbers.push(99999);

const start = Date.now();

const seen = new Set();
let hasDuplicated = false;
for (const n of numbers) {
	if (seen.has(n)) {
		hasDuplicated = true;
		break;
	}
	seen.add(n);
}

console.log(`Duplicated: ${hasDuplicated}`);
console.log(`Time: ${Date.now() - start}mss`);

```

```bash title:bash
node slow.js
# Duplicated: true
# Time: 3ms

```
---

## Claude Sessions
