---
aliases:
  - Learning
tags:
  - learning
---
**Date**: 16/mar/2026
**Update**: 16/mar/2026
**Sources**: 

**Tags:** #devops #infrastructure #deployment 

**Related:** [[Docker]], [[Image]], [[Virtual Machine]], [[Python]], [[Rust]], [[NodeJS]], [[PostgreSQL]]

---

## Description

A container is a **lightweight, isolated unit that packages an application along with everything it needs to run** — code, runtime, libraries, and configuration — **all bundled together**.

---

## Key concepts

- **Isolated** — Containers don't interfere with each other or the host system. Each has its own filesystem, network, and process space.
- **Lightweight** — Unlike `virtual machines`, containers don't include a full OS. They share the host machine's OS kernel, so they use far less memory and disk space.
- **Fast** — Containers start in seconds because there's no OS to boot up.
- **Portable** — A container built on your laptop runs identically on a server or in the cloud.

---

## Details

Think of a shipping container on a cargo ship. It doesn't matter what's inside — electronics, furniture, food — the container is a standardized box that can be loaded onto any ship, truck, or train. `Docker` containers work the same way: standardized packages that run the same on any machine.


### What's inside a container?

- Your application code
- Runtime (e.g., `Python`, `Rust`, `NodeJS`, etc.)
- Dependencies and libraries
- Environment variables and config files


### Container vs the `image`

If a container is a _running_ instance, an image is the _blueprint_. The relationship is like:

- `Image` → a recipe
- **Container** → the dish you cook from it

You can run multiple containers from the same `image` simultaneously, each isolated from the others.

|            | **Container**       | **`Image`**      |
| ---------- | ------------------- | ---------------- |
| State      | Running/active      | Static/read-only |
| Analogy    | The built thing     | Blueprint        |
| Created by | docker run          | docker build     |
| Count      | Many from one image | One              |


### Real-world example

Instead of installing `PostgreSQL` directly on your machine, you can run it in a container using a `Docker` command.

---