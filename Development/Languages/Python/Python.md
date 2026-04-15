---
aliases:
  - Python
tags:
  - learning
  - claude-generated
date: 2026-04-13
---
**Date**: 13/abr/2026
**Update**: 13/abr/2026
**Sources**: Claude Code Session

**Tags:** #learning #claude-generated

**Related:** [[Claude Code]]

---

## Description

Python es un **lenguaje de programación de propósito general**, interpretado, de alto nivel y multiparadigma, creado por Guido van Rossum en 1991. Su filosofía central es que el código debe ser **legible y simple** — al punto de que el propio lenguaje te obliga a escribir código limpio mediante indentación obligatoria en lugar de llaves `{}`.

> Puedes ver su filosofía completa corriendo `import this` en cualquier intérprete.

**Por qué existe:** antes de Python, lenguajes como C o Java requerían mucho código para tareas simples. Python nació para que puedas expresar ideas complejas con pocas líneas, priorizando la productividad del programador sobre la velocidad de la máquina.

**Imagen mental:** una navaja suiza bien afilada — no es la herramienta más rápida para ninguna tarea específica, pero es suficientemente buena para casi todo: ciencia de datos, automatización, backends web, scripting de sistemas y el ecosistema de IA.

---

## Ventajas y Desventajas

| Ventaja | Por qué importa |
|---|---|
| Sintaxis natural | Se lee casi como inglés — menos tiempo descifrando código |
| Ecosistema masivo | Hay una librería para prácticamente todo |
| Multiplataforma | El mismo código corre en Windows, Mac y Linux sin cambios |
| Multiparadigma | Procedural, orientado a objetos o funcional |
| Comunidad enorme | Casi cualquier error ya fue resuelto en Stack Overflow |
| Dominante en IA/Data | Estándar de la industria para ML y análisis de datos |

| Desventaja | Contexto |
|---|---|
| Lento | Interpretado, no compilado. Se mitiga con librerías nativas (NumPy corre en C por debajo) |
| Alto consumo de memoria | No apto para sistemas embebidos o recursos muy limitados |
| GIL | El Global Interpreter Lock impide verdadero paralelismo en threads |
| Tipado dinámico (doble filo) | Acelera el desarrollo pero puede generar bugs difíciles en proyectos grandes |
| No ideal para móvil | Prácticamente no se usa para apps móviles nativas |

---

## Librerías Populares

```
Web Backend       → Django, FastAPI, Flask
Web Scraping      → BeautifulSoup, Scrapy, Playwright
Data / Analytics  → Pandas, Polars, NumPy
Visualización     → Matplotlib, Seaborn, Plotly
Machine Learning  → Scikit-learn, XGBoost
Deep Learning     → PyTorch, TensorFlow, Keras
IA / LLMs         → LangChain, Anthropic SDK, OpenAI SDK
APIs / HTTP       → Requests, HTTPX
Bases de datos    → SQLAlchemy, psycopg2, Tortoise ORM
Testing           → Pytest, unittest
CLI Tools         → Click, Typer, Rich
Automatización    → Selenium, PyAutoGUI
Validación        → Pydantic
```

---

## Roadmap por Nivel

### Principiante — "Hacer que las cosas funcionen"

- **Tipos de datos**: strings, números, listas, diccionarios, sets, tuplas y sus métodos
- **Funciones**: definición, parámetros, valores de retorno, scope
- **Manejo de errores**: `try / except / finally`, tipos de excepciones
- **Módulos y paquetes**: `import`, cómo estructurar código en archivos
- **Comprehensions**: `[x for x in lista if condición]` — uno de los patrones más usados
- **Archivos**: leer y escribir con `open()`, context managers (`with`)
- **Entornos virtuales**: `venv`, qué es y por qué cada proyecto necesita el suyo

### Intermedio — "Código reutilizable y estructurado"

- **POO**: clases, herencia, encapsulación, `__init__`, `__repr__`, dunder methods
- **Decoradores**: funciones que modifican otras funciones — la base de FastAPI y Django
- **Generadores e iteradores**: `yield`, lazy evaluation para grandes volúmenes de datos
- **Context Managers**: crear los tuyos con `__enter__ / __exit__` o `@contextmanager`
- **Type hints**: `def func(name: str) -> int` — mejora legibilidad y soporte de editores
- **Gestión de dependencias**: `pip`, `pyproject.toml`, `uv` (el gestor moderno)

### Avanzado — "Código eficiente, robusto y escalable"

- **Async / Await**: programación asíncrona con `asyncio` — fundamental para APIs e I/O intensivo
- **Concurrencia y paralelismo**: diferencia entre `threading`, `multiprocessing` y `asyncio`
- **Metaclases**: clases que definen el comportamiento de otras clases (cómo Django crea sus modelos)
- **Descriptores**: el mecanismo detrás de `@property` y ORMs
- **Testing avanzado**: mocks, fixtures, parametrización en Pytest, cobertura de código
- **Profiling y optimización**: `cProfile`, `memory_profiler`, identificar cuellos de botella
- **Empaquetado**: publicar tu propia librería en PyPI con `pyproject.toml`

### Experto — "Entender el lenguaje en profundidad"

- **CPython internals**: cómo funciona el intérprete, el bytecode y el GIL
- **Extensions en C**: módulos en C que Python puede importar (lo que hace NumPy)
- **Protocol y Abstract Base Classes**: sistema de tipos estructural de Python
- **Dataclasses y Pydantic**: modelado de datos moderno — Pydantic es la base de FastAPI
- **Design patterns** aplicados a Python: Factory, Observer, Strategy

---

## Claude Sessions

- [[2026-04-13 - Qué Es Python Explicamelo Para Entenderlo Como Fuera La]] — 13/abr/2026
- [[2026-04-14 - Qué Es Python Explicamelo Para Entenderlo Como Fuera La]] — 14/abr/2026

---
