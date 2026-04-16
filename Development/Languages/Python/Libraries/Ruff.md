---
aliases:
  - Ruff
tags:
  - learning
date: 2026-03-25
---
**Sources**: [Ruff](https://docs.astral.sh/ruff/)

**Tags:** #learning

**Related:** [[Python]], [[Rust]]

---

## Description

Write here...

---

## Key concepts

Write here...

---

## Details

Write here...

---

## Examples

Write here...

---

## Snippets

```toml title:pyproject.toml
[tool.ruff]
line-length = 79
target-version = "py312"

[tool.ruff.lint]
select = [
	"E",    # Basic errors
	"F",    # Basic errors
	"I",    # Imports
	"UP",   # Modernize syntax
	"B",    # Bugbear
	"N",    # Naming conventions
	"C901", # Complexity
	"ANN",  # Type hints
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.ruff.lint.isort]
known-first-party = ["backend"]

```

___

## Utils

### Use case

Write here...

```python title:main.py
print("Hello world!")

```

---

## Claude Sessions
