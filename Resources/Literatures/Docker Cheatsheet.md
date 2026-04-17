---
aliases:
  - Docker Cheatsheet
tags:
  - learning
  - dev/devops
date: 2026-04-15
---
**Sources**: Docker Cheatsheet.pdf (Resources folder)

**Related:** [[Docker]], [[Container]], [[Image]], [[Dockerfiles]], [[Compose]], [[Docker Hub]]

---

## What is this resource?

A visual cheatsheet covering the most common Docker CLI commands,
grouped by lifecycle stage (build → run → manage → cleanup).

---

## Key Takeaways

- Docker workflow follows a clear pipeline: `Dockerfile` → `build` → `Image` → `run` → `Container`
- Most daily work revolves around ~15 commands
- Cleanup commands (`prune`) are essential to avoid disk bloat

---

## Quick Reference

### Build
| Command | What it does |
|---|---|
| `docker build -t name .` | Build image from Dockerfile |
| `docker build --no-cache .` | Build without layer cache |

### Run
| Command | What it does |
|---|---|
| `docker run -d -p 80:80 name` | Run detached with port mapping |
| `docker run -it name bash` | Interactive shell inside container |
| `docker run --rm name` | Auto-remove after exit |

### Manage
| Command | What it does |
|---|---|
| `docker ps` | List running containers |
| `docker logs <id>` | View container logs |
| `docker exec -it <id> bash` | Shell into running container |
| `docker stop <id>` | Graceful stop |

### Cleanup
| Command | What it does |
|---|---|
| `docker system prune` | Remove all unused data |
| `docker image prune -a` | Remove all unused images |

---

## Where I expanded on this

- Full Docker concepts → [[Docker]]
- Container deep dive → [[Container]]
- Dockerfile syntax → [[Dockerfiles]]
- Multi-container setup → [[Compose]]

___