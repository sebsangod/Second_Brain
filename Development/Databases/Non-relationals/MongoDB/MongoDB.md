---
aliases:
  - MongoDB
tags:
  - learning
  - dev/database/nosql
date: 2026-05-09
---
**Sources**: [MongoDB Docs](https://www.mongodb.com/docs/), [BSON](http://bsonspec.org/), [BSON data types](https://www.mongodb.com/docs/manual/reference/bson-types/#std-label-bson-types), [MongoDB databases](https://www.mongodb.com/docs/manual/reference/glossary/#std-term-database)

**Related:** [[Non-Relational Database]], [[Relational Database]], [[Array]]

---

## Description

_MongoDB_ is a **document-oriented, operational database** built from the ground up as an alternative to the ``relational database`` for modern applications.

Unlike relational databases, _MongoDB_ allows developers to **store rich JSON-like documents** that map naturally to the objects they use in their code:

```json
{
   firstname: "Bob",
   lastname: "Smith",
   email: "bob@smith.com",
   address: {
      street: "100 Main St",
      city: "Anytown",
      state: "MO",
      zip: "11111"
   }
}
```

---

## Details

### Documents

_MongoDB_ stores data records as _BSON documents_. _BSON_ is a **binary representation of JSON documents**, though it contains **more data types than JSON**.

The value of a field can be any of the _BSON_ data types, including other documents, ``arrays``, and ``arrays of documents``. For example, the following document contains values of varying types:

```json
{
   _id: ObjectId("5099803df3f4948bd2f98391"),
   name: { first: "Alan", last: "Turing" },
   birth: new Date('Jun 23, 1912'),
   death: new Date('Jun 07, 1954'),
   contribs: ["Turing machine", "Turing test", "Turingery"],
   views : Long(1250000)
}
```

The above fields have the following data types:

- _\_id_ holds an _ObjectId_
- _name_ holds an _embedded document_ that contains the fields _first_ and _last_.
- _birth_ and _death_ hold values of the _Date_ type.
- _contribs_ holds an ``array`` _of strings_.
- _views_ holds a value of the _NumberLong_ type.


### Databases and Collections

_MongoDB_ stores data records as _documents_ (specifically _BSON documents_) which are gathered together in collections. A _database_ stores one or more collections of documents.

_MongoDB_ stores ``documents`` in ``collections``. **Collections are analogous to tables in relational databases.**

---

## Examples

Write here...

---

## Utils

### Use case

Write here...

```python title:main.py
print("Hello world!")

```

---

## Claude Sessions
