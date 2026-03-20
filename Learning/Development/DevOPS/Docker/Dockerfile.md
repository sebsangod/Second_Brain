---
aliases:
  - Learning
tags:
  - learning
---
**Date**: 18/mar/2026
**Update**: 18/mar/2026
**Sources**: 

**Tags:** #devops #infrastructure #deployment

**Related:** [[Docker]], [[Image]], [[Container]], [[NodeJS]], [[Python]]

---

## Description

A Dockerfile is a **plain text file containing step-by-step instructions** that tell Docker how to build an `image`.

In short, a Dockerfile is the **source of truth** for how your app's environment is built — making it reproducible, version-controllable, and shareable with your team.

---

## Key concepts

- **Every instruction creates a layer** — `Docker` caches layers, so if nothing changed in a layer, it reuses it. This makes rebuilds fast.
- **Order matters** — Put instructions that change frequently (like copying your code) near the bottom, so `Docker` can reuse cached layers above them.
- **One Dockerfile per app** — typically lives at the root of your project alongside your code.

---

## Details

Think of a Dockerfile like a **recipe**. It lists the ingredients (dependencies) and the steps (instructions) needed to prepare the dish (`image`). Anyone with the recipe can recreate the exact same dish (multiple `containers`).


### Common instructions explained
| **Instruction** | **What it does**                                                 |
| --------------- | ---------------------------------------------------------------- |
| FROM            | Sets the base image to start from (always the first line)        |
| WORKDIR         | Sets the working directory inside the container                  |
| COPY            | Copies files from your machine into the image                    |
| RUN             | Executes a command during the build (e.g., install dependencies) |
| EXPOSE          | Documents which port the app listens on                          |
| ENV             | Sets environment variables                                       |
| CMD             | The default command to run when a container starts               |
| ENTRYPOINT      | Similar to CMD, but harder to override — sets the main process   |


### How it fits into the workflow?

1. `Dockerfile` (recipe)
2. Execute: _docker build_ (cooking)
3. `Image` (dish)
4. Execute: _docker run_ (serve)
5. `Container` (served meal)

---

## Examples

A basic example (of `NodeJS`):

```dockerfile title:Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]
```

A more real-world example (of `Python`):

```dockerfile title:Dockerfile
# Start from official Python image
FROM python:3.11-slim
# Set working directory
WORKDIR /app
# Copy and install dependencies first (for caching)
COPY requirements.txt .
RUN pip install -r requirements.txt
# Then copy the rest of the code
COPY . .
# Set environment variable
ENV PORT=8080
# Expose the port
EXPOSE 8080
# Run the app
CMD ["python", "main.py"]
```

---