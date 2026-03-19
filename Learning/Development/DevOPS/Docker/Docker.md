---
aliases:
  - Learning
tags:
  - learning
---
**Date**: 16/mar/2026
**Update**: 18/mar/2026
**Sources**: [Docker](https://www.docker.com/), [Docker Docs](https://docs.docker.com/)

**Tags:** #devops #infrastructure #deployment

**Related:** [[Container]], [[Image]], [[Dockerfile]], [[Docker Hub]], [[PostgreSQL]], [[Redis]], [[MongoDB]], [[Virtual Machine]], [[Microservices]], [[CI-CD]], [[Database]]

---

## Description

Docker is a platform for **containerization — a way to package and run applications in isolated environments** called `containers`.

In short, Docker makes software **portable, reproducible, and easy to deploy** regardless of where it runs.

---

## Key concepts

* `Container`
- `Image`
- `Dockerfile`
- `Docker Hub`

---

## Details

### What problem it solves?

Traditionally, **software that works on one machine might break on another** due to differences in OS, libraries, or configurations. Docker eliminates this with the phrase:

> _It works on my machine? Ship the machine._


### How it differs from a `virtual machine`?

|           | **Docker `Container`** | `Virtual machine` |
| --------- | ---------------------- | ----------------- |
| OS        | Shares host kernel     | Full OS per VM    |
| Size      | MBs                    | GBs               |
| Startup   | Seconds                | Minutes           |
| Isolation | Process-level          | Hardware-level    |

---

## Use cases

* Running apps consistently across dev, staging, and production
- `Microservices` architecture
- `CI/CD` pipelines
- Spinning up `databases` or services locally without installing them natively

---

## Commands

### CRUD operations
| **Resource** | **Command**                 | **Description** |
| ------------ | --------------------------- | --------------- |
| Network      | docker network ls           | Text            |
| Network      | docker network rm [id/name] | Text            |
| Volume       | docker volume ls            | Text            |
| Volume       | docker volume rm [id/name]  | Text            |
| All          | docker system prune         | Text            |


### Container builds

```bash title:bash
docker run --name <container_name> -p <container_port>:<host_port> --network <network_name> -v <host_path>:<container_database_path> -e <ENVIRONMENT_VARIABLE_KEY>=<environment_variable_value> -d <image>
```

Where _-d_ is the _detached mode_, which means the container will execute background instead of keep the terminal waiting.

___

## Examples

```bash title:redis
docker run --name redis -p 6379:6379 --network redis-network -d redis:latest
```


```bash title:redis-insight
docker run --name redis-insight -p 5540:5540 --network redis-network -d redis/redisinsight:latest
```

___