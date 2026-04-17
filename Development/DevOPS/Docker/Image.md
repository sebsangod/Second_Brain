---
aliases:
  - Learning
tags:
  - learning
  - dev/devops
date: 2026-03-18
---
**Sources**: 

**Related:** [[Docker]], [[Container]], [[Dockerfile]], [[Docker Hub]] [[Python]], [[Rust]], [[NodeJS]]

---

## Description

A Docker image is **a read-only template (blueprint)** used to create `containers`.

In short, an image is the **packaged, shareable snapshot** of everything your app needs — and a `container` is what you get when you actually **run** it.

---

## Key concepts

Think of an image like a **cookie cutter** and a container as the **cookie**. The cutter defines the shape, but you can stamp out as many cookies as you want from it — each one independent.

---

## Details

### What an image contains?

- A base OS layer (e.g., Ubuntu, Alpine Linux, etc.)
- Runtime (e.g., `Python`, `Rust`, `NodeJS`, etc.)
- App dependencies and libraries
- Your application code
- Instructions on what to run at startup


### Images are made of layers

Images are built in stacked layers, where each layer represents a change or instruction. For example:

```
Layer 1: Start from Ubuntu
Layer 2: Install Python
Layer 3: Copy app code
Layer 4: Install pip dependencies
Layer 5: Set startup command
```

This layered system is efficient — if two images share the same base layer (e.g., Ubuntu), **Docker only stores that layer once on disk**.


### How images are created?

You define an image using a `Dockerfile` (see the examples below).


### Where images come from?

- `Docker Hub`
- **Built locally** — from your own `Dockerfile`.
- **Private registries** — like AWS ECR or GitHub Container Registry.


### Container vs the `image`

If a container is a _running_ instance, an image is the _blueprint_. The relationship is like:

- `Image` → a recipe
- **Container** → the dish you cook from it

You can run multiple containers from the same `image` simultaneously, each isolated from the others.

|            | **Image**        | **`Container`**     |
| ---------- | ---------------- | ------------------- |
| State      | Static/read-only | Running/active      |
| Analogy    | Blueprint        | The built thing     |
| Created by | docker build     | docker run          |
| Count      | One              | Many from one image |

---

## Examples

Simple image for `Python`:

```dockerfile title:Dockerfile
FROM python:3.11
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

___

## Claude Sessions