---
aliases:
  - TOML
tags:
  - learning
  - dev/lang/toml
date: 2026-05-12
---
**Sources**: [TOML](https://toml.io/en/)

**Related:** [[Python]], [[Pyproject.toml]]

---

## Description

_TOML_ aims to be a **minimal configuration file format** that's easy to read due to obvious semantics. _TOML_ is designed to map unambiguously to a hash table. _TOML_ should be easy to parse into data structures in a wide variety of languages.

---

## Examples

```toml title:main.toml
title = "TOML Example"

[owner]
name = "Tom Preston-Werner"
dob = 1979-05-27T07:32:00-08:00

[database]
enabled = true
ports = [ 8000, 8001, 8002 ]
data = [ ["delta", "phi"], [3.14] ]
temp_targets = { cpu = 79.5, case = 72.0 }

[servers]

[servers.alpha]
ip = "10.0.0.1"
role = "frontend"

[servers.beta]
ip = "10.0.0.2"
role = "backend"

```

---

## Claude Sessions
