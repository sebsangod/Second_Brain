---
aliases:
  - Learning
tags:
  - learning
  - dev/devops
date: 2026-03-18
---
**Sources**: [Docker Hub](https://docs.docker.com/)

**Related:** [[Docker]], [[Image]], [[Dockerfile]], [[GitHub]], [[NodeJS]], [[PostgreSQL]]

---

## Description

Docker Hub is a **cloud-based registry where you can store, share, and discover Docker `images`** — think of it as the `GitHub` for Docker `images`.

In short, Docker Hub is the **central marketplace** for Docker `images` — saving you from building everything from scratch and making it easy to share your work with the world.

---

## Key concepts

- **Free tier**: unlimited public repositories, 1 private repo
- **Paid plans**: more private repos, team access controls
- **Webhooks**: trigger actions when an image is pushed
- **Automated builds**: connect to GitHub/GitLab to auto-build images on code push
- **Image tags**: version your images (e.g., _my-app:v1_, _my-app:latest_)

---

## Details

If a `Dockerfile` is a recipe and an image is the prepared dish, Docker Hub is the **cookbook library** where anyone can publish their recipes or grab one someone else already made.


### What you can do with Docker Hub?

- **Pull** pre-built images to use locally
- **Push** your own images to share with others
- **Store** private images for your team or organization
- **Automate** builds triggered by code changes


### How it fits into the big picture?

1. You write code
2. `Dockerfile` defines the `image`
3. _docker build_ creates the `image`
4. _docker push_ uploads to Docker Hub
5. Anyone can _docker pull_ and _run_ it anywhere

---

## Examples

For `PostgreSQL` and `NodeJS`:

```bash title:bash
# Pull and run a PostgreSQL database
docker run postgres

# Pull a specific version
docker run postgres:15

# Pull Node.js
docker run node:18
...
```

---

## Alternatives

- **AWS ECR** — Amazon's private container registry
- **Google Artifact Registry** — Google Cloud's registry
- **GitHub Container Registry** — store images alongside your `GitHub` repos
- **self-hosted** — run your own private registry

___

## Claude Sessions