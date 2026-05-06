---
aliases:
  - Pyproject.toml
tags:
  - learning
  - dev/backend
date: 2026-05-05
---
**Sources**: [Python Packaging User Guide](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/)

**Related:** [[Python]], [[TOML]], [[Linter]], [[uv|uv]], [[pip]]

---

## Description

_pyproject.toml_ is a **configuration file** used by packaging tools, as well as other tools such as _linters_, type checkers, etc.

Written in `TOML` _(Tom’s Obvious, Minimal Language)_, **it serves as a single source of truth for your project’s build system, metadata, and tool configurations.** Think of it as the blueprint that tells tools like `pip` or `uv` **how to construct your project.**

For example, a minimal _pyproject.toml_ might look like this:

```toml
[build-system]  
requires = ["setuptools>=61.0"]  
build-backend = "setuptools.build_meta"  
  
[project]  
name = "engineering-mindset"  
version = "0.1.0"  
description = "A simple project to explore software engineering basics"  
dependencies = ["requests>=2.25.0"]

```

This file tells Python how to build your project and what it needs to run — no complex scripts required.

---

## Key concepts

Here’s a basic breakdown of its structure:

- **[build-system]**: Defines the tools needed to build your project (e.g., setuptools or flit_core).
- **[project]**: Houses metadata like your project’s name, version, and dependencies.
- **[tool]**: Configures settings for development tools like linters or formatters.

---

## Claude Sessions
