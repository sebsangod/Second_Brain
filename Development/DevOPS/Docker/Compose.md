---
aliases:
  - Learning
tags:
  - learning
  - dev/devops
date: 2026-03-15
---
**Sources**: 

**Related:** [[Docker]], [[Container]], [[Dockerfile]], [[Image]], [[Kubernetes]], [[Python]], [[NodeJS]], [[PostgreSQL]], [[MySQL]], [[Redis]], [[RabbitMQ]]


---

## Description

Docker Compose is a tool for defining and running **multi-container `Docker` applications** using a single YAML file.

In short, Docker Compose is the **glue** that ties multiple `containers` together into a complete, runnable application — making local development and simple deployments dramatically easier.


### The problem it solves

Real apps rarely run as a single container. A typical web app might need:

- A web server (`Python`, `NodeJS`, etc.)
- A database (`PostgreSQL`, `MySQL`)
- A cache (`Redis`)
- A message queue (`RabbitMQ`)

Without Compose, you'd have to manually start each container with long _docker run_ commands, configure networking between them, and manage them individually. Compose lets you define everything in **one file** and start it all with **one command**.

---

## Key concepts

| Concept     | What it does                                     |
| ----------- | ------------------------------------------------ |
| services    | Defines each container in your app               |
| build       | Builds an image from a local `Dockerfile`        |
| image       | Uses a pre-built image (e.g., from `Docker Hub`) |
| ports       | Maps host port to `container` port               |
| environment | Sets environment variables                       |
| volumes     | Persists data beyond the `container's` lifetime  |
| depends_on  | Controls startup order between services          |
| networks    | Defines how `containers` communicate             |

---

## Details

If a `Dockerfile` is a recipe for one dish, Docker Compose is the **full dinner menu** — it coordinates multiple dishes (`containers`) to be served together as a complete meal.


### How containers talk to each other?

Compose automatically creates a shared network for all services. `Containers` can reach each other using the **service name** as the hostname:

```javascript title:javascript
// Inside the 'web' container, connect to the database like this:
const db = connect("postgres://db:5432/mydb")
//                          ^^
//                   service name = hostname
```

### How it fits into the big picture


1. _docker run_: runs a single `container`
2. `Dockerfile`: defines one `image`
3. _docker compose_: orchestrates many `containers` together


### Compose vs. `Kubernetes`
|                | Docker Compose                | Kubernetes             |
| -------------- | ----------------------------- | ---------------------- |
| Best for       | Local dev, simple deployments | Large-scale production |
| Complexity     | Simple                        | Complex                |
| Scaling        | Basic                         | Advanced               |
| Learning curve | Low                           | High                   |

---

## Examples

```yaml title:docker-compose.yaml
version: '3'
services:

  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

  redis:
    image: redis:alpine

volumes:
  db-data:
```

---

## Commands

```bash
# Start all containers in the background
docker compose up -d

# Stop all containers
docker compose down

# View logs from all containers
docker compose logs

# View logs from a specific service
docker compose logs web

# Rebuild images and restart
docker compose up --build

# List running services
docker compose ps

# Run a command inside a service
docker compose exec web bash
```

___

## Claude Sessions