---
aliases:
  - Master Config API
tags:
  - project/idea
  - dev/backend
  - dev/security
date: 2026-03-21
---
**Sources**:

**Related:** [[Microservices]], [[API Rest]], [[WebSocket]], [[FastAPI]], [[Python]], [[NodeJS]]

---

## Main idea

Separate important settings from an app or a project main code.

---

## Goal

Store other project's important settings in a separate service to avoid redeployments while upgrading.

---

## Objectives

* Create a microservice to store an give back extern service settings
* Maker extern services or apps to have zero-downtime while upgrading them
* Serve those settings as fast as possible

---

## Details

Write here...

---

## Tasks

- [ ] 🔺 Implement strong authentication methods
- [ ] 🔺 Compare development stacks: `API Rest` service (`FastAPI`, `NodeJS`, etc. HTTP clients) vs `WebSockets` efficiency.

___