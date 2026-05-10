---
aliases:
  - uv
tags:
  - learning
  - dev/backend
date: 2026-04-28
---
**Sources**: [uv](https://docs.astral.sh/uv/)

**Related:** [[Python|Python]], [[Rust]], [[pyenv]]

---

## Description

An extremely fast `Python` package and project manager, written in `Rust`.

---

## Key concepts

- A single tool to replace `pip`, `pip-tools`, `pipx`, `poetry`, `pyenv`, `twine`, `virtualenv`, and more.
- [10-100x faster](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md) than `pip`.
- Provides [comprehensive project management](https://docs.astral.sh/uv/#projects), with a [universal lockfile](https://docs.astral.sh/uv/concepts/projects/layout/#the-lockfile).
- [Runs scripts](https://docs.astral.sh/uv/#scripts), with support for [inline dependency metadata](https://docs.astral.sh/uv/guides/scripts/#declaring-script-dependencies).
- [Installs and manages](https://docs.astral.sh/uv/#python-versions) Python versions.
- [Runs and installs](https://docs.astral.sh/uv/#tools) tools published as Python packages.
- Includes a [pip-compatible interface](https://docs.astral.sh/uv/#the-pip-interface) for a performance boost with a familiar CLI.
- Supports Cargo-style [workspaces](https://docs.astral.sh/uv/concepts/projects/workspaces/) for scalable projects.
- Disk-space efficient, with a [global cache](https://docs.astral.sh/uv/concepts/cache/) for dependency deduplication.
- Installable without Rust or Python via `curl` or `pip`.
- Supports macOS, Linux, and Windows.

---

## Installation

```bash title:"MacOS and Linux"
$ curl -LsSf https://astral.sh/uv/install.sh | sh
```

```bash title:Windows
$ powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Once installed, execute:

```bash
$ uv self update
```

---

## `Python` Version Management

| **Command**            | **Description**                                                               |
| ---------------------- | ----------------------------------------------------------------------------- |
| uv python install 3.12 | Downloads and installs Python 3.12 (without `pyenv`)                          |
| uv python list         | Lists installed and available versions                                        |
| uv python pin 3.12     | Fixes given Python version for the current project (creating .python-version) |

---

## Initiate a Project

| **Command**              | **Description**                                                                 |
| ------------------------ | ------------------------------------------------------------------------------- |
| uv init my_project       | Creates project folder with pyproject.toml, README and basic initial estructure |
| uv init --lib my_library | Project type library (with /src and package settings)                           |
| uv init                  | Initiates uv within an existing directory                                       |

## Dependencies Management

| **Command**                        | **Description**                                         |
| ---------------------------------- | ------------------------------------------------------- |
| uv add requests                    | Adds the dependency, updates pyproject.toml and uv.lock |
| uv add --dev pytest ruff           | Adds dependencies just for development mode             |
| uv remove requests                 | Removes the package and updates pyproject.toml          |
| uv sync                            | Syncs the environment with uv.lock                      |
| uv pip install -r requirements.txt | Pip compatibility for existing projects                 |

___

## Development Flow

| **Command**          | **Description**                                                          |
| -------------------- | ------------------------------------------------------------------------ |
| uv run main.py       | Executes the given script within the project environment (without venv)  |
| uv run pytest        | Executes any tool installed within the project environment               |
| uv ruff check .      | Executes any tool within an fleeting environment (without installing it) |
| uv tool install ruff | Installes CLI tool globally (for every project usage)                    |
| uv tree              | Shows the project's dependencies tree                                    |
| uv lock --upgrade    | Updates uv.lock with the latest compatible versions                      |

---

## Claude Sessions
