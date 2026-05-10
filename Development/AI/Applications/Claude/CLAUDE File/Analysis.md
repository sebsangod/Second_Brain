---
aliases:
  - CLAUDE File
tags:
  - dev/ai/llm
date: 2026-05-10
---
**Sources**: [Claude Code 101](https://anthropic.skilljar.com/claude-code-101)

**Related:** [[Claude Code]], [[Claude]], [[CLAUDE File]]

---
# Análisis: Estructura Óptima de CLAUDE.md para Proyectos Odoo / FastAPI / IA

## Fuentes Analizadas

| Fuente | Tipo | Contenido Clave |
|--------|------|-----------------|
| [Claude.md](file:///c:/Users/USER/Documents/Obsidian/Main/Development/AI/Applications/Claude/Claude.md) | Notas del curso Claude 101 | Prompting, Projects, Artifacts, iteración |
| [CLAUDE.md File.md](file:///c:/Users/USER/Documents/Obsidian/Main/Development/AI/Applications/Claude/Claude_md/CLAUDE.md%20File.md) | Notas del curso Claude Code 101 | Propósito, ejemplo básico, jerarquía, tips |
| [Claude Code.md](file:///c:/Users/USER/Documents/Obsidian/Main/Development/AI/Applications/Claude/Claude%20Code/Claude%20Code.md) | Notas de Claude Code in Action | Agentic loop, contexto, /init, thinking modes |
| [Commands.md](file:///c:/Users/USER/Documents/Obsidian/Main/Development/AI/Applications/Claude/Claude%20Code/Commands.md) | Notas de Claude Code in Action | /compact, /clear, custom commands |
| [Hooks.md](file:///c:/Users/USER/Documents/Obsidian/Main/Development/AI/Applications/Claude/Claude%20Code/Hooks.md) | Notas de Claude Code in Action | PreToolUse, PostToolUse, protección de .env |
| [SDKs.md](file:///c:/Users/USER/Documents/Obsidian/Main/Development/AI/Applications/Claude/Claude%20Code/SDKs.md) | Notas de Claude Code in Action | Uso programático del SDK |
| [Agustín Medina](file:///c:/Users/USER/Documents/Obsidian/Main/Development/AI/Applications/Claude/Claude_md/Examples/Agust%C3%ADn%20Medina%20-%20Agente%20de%20Automatizaci%C3%B3n%20Backend.md) | Ejemplo externo | CLAUDE.md para automatizaciones backend |
| [Docs oficiales](https://code.claude.com/docs/en/claude-md) | Documentación Anthropic | Estructura, reglas, imports, auto memory |
| [Best Practices](https://code.claude.com/docs/en/best-practices) | Documentación Anthropic | Patrones y anti-patrones |

---

## 1. Evaluación del Ejemplo de Agustín Medina

### ✅ Fortalezas

| Aspecto | Detalle |
|---------|---------|
| **Identidad clara** | Define un rol específico ("desarrollador backend de clase mundial y ejecutor de automatizaciones") — esto alinea el comportamiento de Claude desde el inicio |
| **Prioridades explícitas** | `seguridad > fiabilidad > legibilidad > rendimiento` — esto es exactamente el tipo de instrucción concreta que Anthropic recomienda |
| **Reglas absolutas** | Las 3 reglas (credenciales en .env, autocorrección, no inventar dependencias) son **concretas y verificables**, no vagas |
| **Estructura de repositorio** | Incluir el tree del proyecto es una práctica recomendada por Anthropic — le da a Claude un mapa mental de la arquitectura |
| **README como índice** | El "README maestro" como tabla de navegación es brillante para que Claude sepa qué scripts existen antes de crear nuevos |
| **TASK.md por script** | Documentación granular que permite a Claude ejecutar tareas de forma autónoma |

### ⚠️ Debilidades y Oportunidades de Mejora

| Problema | Impacto | Mejora Sugerida |
|----------|---------|-----------------|
| **Falta sección de comandos de desarrollo** | Claude no sabe cómo instalar, ejecutar tests o hacer lint | Añadir: `pip install -r requirements.txt`, `pytest`, `ruff check`, etc. |
| **No hay referencia a archivos con `@`** | Claude tiene que descubrir los archivos manualmente cada sesión — gasta contexto | Usar `@README.md`, `@.env.example`, `@requirements.txt` para pre-cargar archivos críticos |
| **No hay sección de Code Style técnico** | Las reglas de identidad son filosóficas, pero faltan convenciones de código | Añadir: indentación, naming, imports, type hints, docstrings format |
| **No usa `.claude/rules/`** | Todo está en un solo archivo monolítico | Separar reglas por dominio (seguridad, estilo, workflow) en `.claude/rules/` |
| **TASK.md incompleto** | El ejemplo de TASK.md se corta en la sección "Todos los parámetros" | El template no está finalizado |
| **No hay sección de testing** | No define cómo debe Claude verificar su trabajo | Añadir: qué framework de tests, cómo ejecutar, qué coverage esperar |
| **No hay patrones de error** | No dice qué errores comunes evitar | Anthropic recomienda incluir anti-patrones específicos del proyecto |
| **Demasiada prosa narrativa** | Frases largas y explicativas consumen contexto window innecesariamente | Anthropic recomienda bullet points concisos y directos |
| **Falta workflow de git** | No dice cómo hacer commits, branches, PRs | Añadir convenciones de branching y mensajes de commit |

### Veredicto General

> [!IMPORTANT]
> El CLAUDE.md de Agustín Medina es un **buen punto de partida temático** pero tiene una estructura más cercana a un "system prompt narrativo" que a la arquitectura modular que Anthropic recomienda en su documentación oficial. Su mayor fortaleza es la **claridad de identidad y las reglas absolutas**; su mayor debilidad es la **falta de instrucciones técnicas concretas** (comandos, style, testing) y la **no utilización de features avanzadas** como `@imports`, `.claude/rules/`, o skills.

---

## 2. Principios Oficiales de Anthropic para un CLAUDE.md Efectivo

Extraídos directamente de la documentación oficial y tus notas del curso:

### Principio 1: Concreción sobre generalización

```diff
- "Format code properly"
+ "Use 2-space indentation"

- "Test your changes"  
+ "Run npm test before committing"

- "Keep files organized"
+ "API handlers live in src/api/handlers/"
```

### Principio 2: Cuándo añadir algo al CLAUDE.md

Solo agregar cuando:
- Claude comete el **mismo error dos veces**
- Un code review detecta algo que Claude debió saber
- Tecleas la **misma corrección** que en la sesión anterior
- Un nuevo teammate necesitaría **el mismo contexto**

### Principio 3: Arquitectura de capas

| Capa | Archivo | Propósito | Compartido |
|------|---------|-----------|------------|
| Organizacional | `C:\Program Files\ClaudeCode\CLAUDE.md` | Estándares de empresa | Todos |
| Global personal | `~/.claude/CLAUDE.md` | Preferencias personales en todos los proyectos | Solo tú |
| Proyecto (equipo) | `./CLAUDE.md` o `./.claude/CLAUDE.md` | Reglas del proyecto, en git | Equipo |
| Proyecto (personal) | `./CLAUDE.local.md` | Overrides personales, en .gitignore | Solo tú |
| Por subdirectorio | `./module_name/CLAUDE.md` | Reglas específicas de un módulo | Equipo |
| Reglas temáticas | `.claude/rules/*.md` | Reglas scoped por path o dominio | Equipo |

### Principio 4: Usar `@imports` para no duplicar contenido

```markdown
See @README.md for project overview.
Database schema: @docs/schema.md
API conventions: @docs/api-conventions.md
```

### Principio 5: Mantenerlo compacto

> [!WARNING]
> El contenido del CLAUDE.md se inyecta en **cada request**. Cada línea extra consume contexto window. Si tu CLAUDE.md crece demasiado, mueve contenido a `.claude/rules/` con path-scoping, o usa `@imports` para archivos que Claude solo necesita a veces.

---

## 3. Estructura Recomendada para Tus Proyectos

### 3.1 — Proyecto Odoo + Python

```
tu-modulo-odoo/
├── CLAUDE.md                          # ← Archivo principal (proyecto, equipo)
├── CLAUDE.local.md                    # ← Tus overrides personales (.gitignore)
├── .claude/
│   ├── CLAUDE.md                      # ← Alternativa al CLAUDE.md raíz
│   ├── rules/
│   │   ├── odoo-conventions.md        # ← Reglas ORM, models, views
│   │   ├── code-style.md             # ← PEP8, naming, imports
│   │   ├── security.md               # ← Access rights, record rules
│   │   └── testing.md                # ← pytest, TransactionCase
│   ├── skills/
│   │   └── create-odoo-model/        # ← Skill para crear modelos Odoo
│   │       └── SKILL.md
│   └── settings.json
└── docs/
    ├── architecture.md                # ← Referenciado con @docs/architecture.md
    └── deployment.md
```

### 3.2 — Proyecto FastAPI Backend

```
tu-api-fastapi/
├── CLAUDE.md
├── CLAUDE.local.md
├── .claude/
│   ├── rules/
│   │   ├── api-design.md             # ← REST conventions, error responses
│   │   ├── code-style.md             # ← Type hints, Pydantic models
│   │   ├── database.md               # ← SQLAlchemy/Alembic patterns
│   │   ├── security.md               # ← Auth, CORS, secrets
│   │   └── testing.md                # ← pytest-asyncio, httpx
│   ├── skills/
│   │   └── create-endpoint/
│   │       └── SKILL.md
│   └── settings.json
└── docs/
    └── api-spec.md
```

### 3.3 — Proyecto de IA / ML

```
tu-proyecto-ia/
├── CLAUDE.md
├── .claude/
│   ├── rules/
│   │   ├── ml-conventions.md         # ← Frameworks, experiment tracking
│   │   ├── data-handling.md          # ← Pipelines, validación
│   │   ├── code-style.md
│   │   └── testing.md
│   └── settings.json
```

---

## 4. Anatomía del CLAUDE.md Principal — Template Recomendado

A continuación, la estructura sección por sección con el **por qué** de cada una. Este no es el archivo final — es la arquitectura conceptual.

### Sección 1: Identidad del Proyecto
**Por qué:** Claude necesita saber qué está construyendo para tomar decisiones de arquitectura correctas.

```
# Proyecto
Este es un módulo de Odoo 19 para [X]. Backend en Python 3.12.
Stack: Odoo 19 CE, PostgreSQL 17, Python 3.12.
```

> [!TIP]
> **Sé telegráfico.** No narrativas. Claude no necesita motivación — necesita datos.

---

### Sección 2: Comandos de Desarrollo
**Por qué:** Anthropic lo menciona como la primera cosa que debes poner. Claude los usa para verificar su propio trabajo.

```
# Comandos
- Iniciar Odoo: `python odoo-bin -c odoo.conf -d mydb --dev=all`
- Tests unitarios: `python odoo-bin -c odoo.conf -d test_db --test-enable --stop-after-init -i my_module`
- Lint: `ruff check . --fix`
- Format: `ruff format .`
- Type check: `mypy src/`
```

---

### Sección 3: Arquitectura y Archivos Clave (con @imports)
**Por qué:** Reduce la exploración que Claude tiene que hacer. Los `@imports` cargan archivos sin duplicar contenido.

```
# Arquitectura
Estructura del módulo: @docs/architecture.md
Schema de datos: @models/__init__.py

Archivos críticos:
- Models: models/
- Views: views/
- Controllers: controllers/
- Security: security/ir.model.access.csv
```

---

### Sección 4: Code Style
**Por qué:** Evita las correcciones repetitivas que Anthropic dice que son la señal #1 de que necesitas agregar algo al CLAUDE.md.

```
# Code Style
- Indentación: 4 espacios
- Strings: comillas simples para código, dobles para user-facing
- Imports: stdlib → third-party → odoo → módulo local, separados por línea en blanco
- Type hints obligatorios en funciones públicas
- Docstrings: Google style
- Nombres de modelos Odoo: snake_case con prefijo del módulo
- Nombres de campos: snake_case, descriptivos
```

---

### Sección 5: Convenciones del Framework
**Por qué:** Cada framework tiene idiosincrasias que Claude puede equivocar. Esta sección previene los errores más comunes.

Para **Odoo**:
```
# Convenciones Odoo
- Usar `fields.Char()`, no `CharField` — esto NO es Django
- Herencia: `_inherit` para extender, `_name` para nuevo modelo
- No usar `@api.one` — fue deprecado en Odoo 13
- Usar `@api.depends` para computed fields, `@api.onchange` para UI-only
- Security: siempre definir access rights en ir.model.access.csv
- Vistas: usar `<record>` para data, no `<template>` (excepto website)
```

Para **FastAPI**:
```
# Convenciones FastAPI
- Schemas Pydantic en schemas/, no en routers
- Dependency injection para DB sessions via `Depends(get_db)`
- Errores: usar HTTPException con status codes explícitos
- Background tasks: usar `BackgroundTasks`, no threading manual
- Alembic para migraciones, nunca `create_all()`
```

---

### Sección 6: Reglas Absolutas (Anti-patrones)
**Por qué:** El ejemplo de Agustín Medina hace esto bien. Reglas que Claude nunca debe romper.

```
# Reglas Absolutas
1. NUNCA hardcodear credenciales — siempre .env + python-dotenv
2. NUNCA usar sql crudo en Odoo — siempre ORM (env['model'].search/create/write)
3. NUNCA hacer commits con tests fallando
4. NUNCA crear archivos sin actualizar __manifest__.py (Odoo)
5. Si no conoces una API de Odoo, PREGUNTA antes de inventar
```

---

### Sección 7: Workflow de Verificación
**Por qué:** Anthropic dice "Give Claude a way to verify its work" como el consejo #1 de best practices.

```
# Verificación
Después de cada cambio:
1. Ejecutar `ruff check . --fix`
2. Ejecutar los tests del módulo modificado
3. Si es Odoo: verificar que el módulo se instala sin errores
4. Si es API: verificar con `curl` o `httpx` que el endpoint responde
```

---

### Sección 8: Git Workflow
**Por qué:** Estandariza cómo Claude hace commits y evita mensajes genéricos.

```
# Git
- Branch naming: feature/xxx, fix/xxx, refactor/xxx
- Commit messages: conventional commits (feat:, fix:, refactor:, docs:, test:)
- Un commit por cambio lógico, no un commit gigante
- Siempre hacer lint antes de commit
```

---

## 5. Uso de `.claude/rules/` con Path-Scoping

Esta es una feature **avanzada** que tu ejemplo de Agustín Medina no usa pero que es extremadamente útil para proyectos Odoo con múltiples módulos.

```markdown
# Archivo: .claude/rules/odoo-models.md
---
paths:
  - "models/**/*.py"
---

# Reglas para Modelos Odoo
- Todo modelo debe definir `_name`, `_description`
- Campos computed deben tener `@api.depends` explícito
- Usar `_sql_constraints` para restricciones de BD
- Métodos CRUD override: siempre llamar `super()`
```

```markdown
# Archivo: .claude/rules/odoo-security.md
---
paths:
  - "security/**"
---

# Reglas de Seguridad Odoo
- Cada modelo DEBE tener entrada en ir.model.access.csv
- Format CSV: id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
- Record rules en ir.rule para multi-company
```

```markdown
# Archivo: .claude/rules/fastapi-routes.md
---
paths:
  - "src/api/**/*.py"
  - "routers/**/*.py"
---

# Reglas para Endpoints FastAPI
- Toda ruta debe tener response_model definido
- Usar status codes: 201 para creación, 204 para delete
- Paginación obligatoria en endpoints de listado
- Documentación OpenAPI en cada endpoint
```

> [!TIP]
> Estas reglas **solo se cargan** cuando Claude trabaja con archivos que coinciden con los `paths`. Esto ahorra contexto window y mantiene las instrucciones relevantes.

---

## 6. Diferencia entre CLAUDE.md y lo que Agustín Medina Implementó

| Característica | Agustín Medina | Recomendación Anthropic | Diferencia |
|---|---|---|---|
| Identidad/Rol | ✅ Extenso, narrativo | ⚡ Breve, telegráfico | El de Agustín consume más contexto de lo necesario |
| Comandos de dev | ❌ Ausente | ✅ Primera sección recomendada | Gap crítico |
| Estructura de archivos | ✅ Tree completo | ✅ Recomendado | Alineado |
| @imports | ❌ No usa | ✅ Recomendado | Oportunidad perdida |
| .claude/rules/ | ❌ No usa | ✅ Feature clave para modularidad | Gap importante |
| Code style técnico | ❌ Ausente | ✅ Esencial | Gap crítico |
| Reglas absolutas | ✅ Bien definidas (3) | ✅ Recomendado | Bien hecho |
| Verificación/Tests | ❌ Ausente | ✅ Best practice #1 | Gap crítico |
| Git workflow | ❌ Ausente | ✅ Recomendado | Gap menor |
| Tamaño del archivo | ~127 líneas | < 100 líneas idealmente | Ligeramente largo por la prosa |

---

## 7. Recomendación Final: Estrategia de Implementación

### Paso 1: Empieza sin CLAUDE.md
Como tus propias notas del curso dicen: "*Start without one*". Trabaja 2-3 sesiones y anota dónde corriges a Claude repetidamente.

### Paso 2: Crea el CLAUDE.md con las secciones que realmente necesitas
Siguiendo la anatomía de la Sección 4 de este documento, pero **solo las secciones donde tuviste que corregir a Claude**.

### Paso 3: Modulariza con `.claude/rules/`
Mueve las reglas específicas de dominio (Odoo models, FastAPI routes, security) a archivos separados con path-scoping.

### Paso 4: Crea Skills para tareas repetitivas
Si siempre le pides a Claude "crea un modelo Odoo con sus vistas y seguridad", eso es un skill.

### Paso 5: Itera con `#` (memory mode)
Cada vez que corrijas a Claude, usa `#` para guardar la corrección en el CLAUDE.md automáticamente.