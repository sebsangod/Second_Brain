---
aliases:
  - Ruff
tags:
  - learning
  - dev/lang/python
date: 2026-05-05
---
**Sources**: [Ruff](https://docs.astral.sh/ruff/)

**Related:** [[Python]], [[Linter]], [[Rust]], [[pip]], [[Pyproject.toml]]

---

## Description

An extremely fast `Python` `linter` and code formatter, written in `Rust`.

---

## Key concepts

- ⚡️ 10-100x faster than existing linters (like Flake8) and formatters (like Black)
- 🐍 Installable via `pip`
- 🛠️ `pyproject.toml` support
- 📦 Built-in caching, to avoid re-analyzing unchanged files
- 🔧 Fix support, for automatic error correction (e.g., automatically remove unused imports)
- 📏 Over [800 built-in rules](https://docs.astral.sh/ruff/rules/), with native re-implementations of popular Flake8 plugins, like flake8-bugbear
- ⌨️ First-party [editor integrations](https://docs.astral.sh/ruff/editors/) for [VS Code](https://github.com/astral-sh/ruff-vscode) and [more](https://docs.astral.sh/ruff/editors/setup/)
- 🌎 Monorepo-friendly, with [hierarchical and cascading configuration](https://docs.astral.sh/ruff/configuration/#config-file-discovery)


---

## Examples


```toml title:pyproject.toml
[project]
...

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
	"C90", # Cyclomatic complexity
	"C901", # Complexity
	"ANN",  # Type hints
	"S",   # Security
]
ignore = [
	"S311",  # Predictable random numbers
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.ruff.lint.isort]
known-first-party = [
	"backend",
	"my_addons",
	"base_addons",
	"odoo19"
]

[tool.ruff.lint.per-file-ignores]
"**/__manifest__.py" = ["B018"]
"**models/*.py" = ["N806"]
"**controllers/*.py" = ["N806"]
"**wizards/*.py" = ["N806"]
"**/__init__.py" = ["F401"] # Imported but unused warning

```

---

## Claude Sessions