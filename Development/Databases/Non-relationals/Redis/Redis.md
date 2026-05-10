---
aliases:
  - Redis
tags:
  - learning
  - dev/database/nosql
date: 2026-05-09
---
**Sources**: [Redis](https://redis.io/), [Docker Hub Image](https://hub.docker.com/_/redis?xk=ShowRecommendedBadge&xt=Disabled), [Redis Insight GUI Docker Hub Image](https://hub.docker.com/r/redis/redisinsight)

**Related:** [[Non-Relational Database]], [[Vector Database]], [[Python]], [[Pub-Sub]], [[Lua]], [[Docker]]

---

## Description

_Redis_ is an **in-memory data store** used by millions of developers as a ``cache database``, ``vector database``, ``document database``, ``streaming engine``, and ``message broker``.

_Redis_ also provides advanced features like **transactions**, ``pub/sub`` **messaging,** ``Lua`` **scripting, and built-in replication and clustering capabilities** for better availability and scalability.

It supports complex data types:
- Strings
- Hashes
- Lists
- Sets
- Sorted sets
- JSON

___

## Key Concepts

- **Fast performance:** as _Redis_ stores data in memory, it can offer significantly faster access times compared to traditional ``relational databases`` which store data on disk. This makes it ideal for use cases that require high performance, such as caching and real-time applications.
- **Scalability:** _Redis_ is designed with scalability in mind, making it easy to handle large amounts of data without compromising performance.
- **Ease of use:** ``Python's`` simple syntax and rich library ecosystem make it easy to work with _Redis_. The official ``Redis-py`` client library also provides a user-friendly interface for interacting with _Redis_ from within your ``Python`` code.

---

## Details

### Use cases

The potential use cases for _Redis_ are vast and varied.

Some of the most common use cases in data engineering include: 

- **Caching:** Accelerate ``database`` queries and ``API`` responses.
- **Message** ``queues``: Power event-driven architectures and job processing pipelines.
- **Real-time analytics:** Process and analyze streaming data with minimal latency.
- **Leaderboards:** Efficiently rank and retrieve top-scoring entities.
- **Session storage:** Maintain fast, scalable session state management.
- **Job queue management:** Ensure efficient task scheduling and execution.
- **Geospatial Indexing:** Store and query location-based data.

___

## Claude Sessions