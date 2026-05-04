---
aliases:
  - CLAUDE.md
tags:
  - dev/backend
  - dev/erp
date: 2026-05-02
---
**Sources**: [Odoo Coding Guidelines](https://www.odoo.com/documentation/19.0/contributing/development/coding_guidelines.html), [Odoo Developers](https://www.odoo.com/documentation/19.0/developer.html), [Best Practices](https://code.claude.com/docs/en/best-practices)

**Related:** [[CLAUDE File]], [[Odoo]], [[Odoo Developers]], [[Odoo Owl|Odoo Owl]]

---

# Odoo Professional Software Developer

## Project
Odoo 19 CE custom module. Python 3.12, PostgreSQL 17.
Framework: Odoo ORM + QWeb views + OWL components.

## Commands
- Start server: `python odoo-bin -c odoo.conf -d <db> --dev=all`
- Install/upgrade module: `python odoo-bin -c odoo.conf -d <db> -i <module> --stop-after-init`
- Run tests (single module): `python odoo-bin -c odoo.conf -d test_db --test-enable --stop-after-init -i <module>`
- Run specific test: `python odoo-bin -c odoo.conf -d test_db --test-tags=/<module>:<TestClass>.test_method --stop-after-init`
- Scaffold new module: `python odoo-bin scaffold <module_name> <addons_path>`
- Lint: `ruff check . --fix && ruff format .`

## Module Structure (Odoo official conventions)
```
module_name/
├── __init__.py
├── __manifest__.py
├── controllers/
│   ├── __init__.py
│   └── main.py
├── data/
│   └── module_data.xml
├── demo/
│   └── module_demo.xml
├── models/
│   ├── __init__.py
│   └── model_name.py          # one file per model
├── report/
│   ├── __init__.py
│   ├── report_model.py
│   └── report_templates.xml
├── security/
│   ├── ir.model.access.csv
│   └── security.xml    # record rules, groups
├── static/
│   └── src/
│       ├── js/
│       ├── scss/
│       ├── xml/
│       └── components/               # OWL templates
│           └── my_component/
│               ├── my_component.js
│               ├── my_component.scss
│               └── my_component.xml
├── tests/
│   ├── __init__.py
│   └── test_model_name.py
├── views/
│   ├── model_name_views.xml   # form, list, search views
│   ├── model_name_menus.xml   # menu items
│   └── actions.xml            # window actions
└── wizard/
    ├── __init__.py
    ├── wizard_name.py
    └── wizard_name_views.xml
```

## __manifest__.py Checklist
Every module MUST have a complete manifest:
- `name`, `version`, `summary`, `description`
- `author`, `website`, `category`
- `depends`: list ALL dependencies — never assume base modules
- `data`: ordered list — security files first, then data, then views
- `demo`: demo data files (loaded only in demo mode)
- `assets`: JS/SCSS/XML bundles when using OWL
- `license`: `LGPL-3` for CE modules
- `installable`: `True`

## ORM Conventions

### Model definition order (official Odoo style)
```python
class ModelName(models.Model):
    _name = "module_name.model_name"
    _description = "Human Readable Name"
    _inherit = ["mail.thread", "mail.activity.mixin"]  # mixins
    _order = "sequence, name"
    _rec_name = "name"

    # 1. Default method
    # 2. Field declarations (grouped: basic, relational, computed)
    # 3. CRUD methods (create, write, unlink)
    # 4. Compute / inverse / search methods
    # 5. Selection method and default method
    # 6. Constrains and onchange methods
    # 7. Action methods
    # 8. Business logic methods
```

### Field declarations
- Use `fields.Char()`, `fields.Integer()`, etc. — NOT Django/SQLAlchemy syntax
- Always provide a `string` parameter or first positional label for clarity
- Computed fields: always declare `compute=`, `store=` (if persistent), and `@api.depends`
- Relational fields: always define `comodel_name` explicitly, add `ondelete` for Many2one
- Use `_sql_constraints` for DB-level uniqueness and checks

### ORM methods — always use the ORM, never raw SQL
- Search: `self.env['model.name'].search([domain])`
- Read: `record.field_name` or `record.read(['field_list'])`
- Create: `self.env['model.name'].create(vals_dict)`
- Write: `record.write(vals_dict)`
- Unlink: `record.unlink()`
- When overriding CRUD: ALWAYS call `super()` — `return super().create(vals_list)`
- Batch operations: write for loops that call `create()` once with a list, not N times

### API decorators
- `@api.depends('field')` — for computed fields, triggers recompute
- `@api.constrains('field')` — validation, raises `ValidationError`
- `@api.onchange('field')` — UI-only, do NOT use for business logic
- `@api.model` — classmethod that doesn't operate on recordsets
- `@api.model_create_multi` — for overriding `create()` with list of vals
- NEVER use `@api.one` — deprecated since Odoo 13
- NEVER use `@api.multi` — deprecated since Odoo 13

### Context and environment
- Always propagate context: `self.with_context(**ctx)` or `self.env['model'].with_context()`
- Access current user: `self.env.user`
- Access current company: `self.env.company`
- Sudo: `self.sudo()` — use sparingly, document why
- Never call `self.env.cr.commit()` — Odoo manages transactions

## XML Conventions

### XML IDs naming pattern
- Views: `{module}.{model_name}_view_{type}` → `estate.property_view_form`
- Actions: `{module}.{model_name}_action` → `estate.property_action`
- Menus: `{module}.menu_{name}` → `estate.menu_root`
- Security groups: `{module}.group_{name}` → `estate.group_manager`
- Access rules: `access_{model}_{group}` → `access_estate_property_group_user`
- Record rules: `{module}.{model}_{rule_name}` → `estate.property_company_rule`

### View structure
- Form views: `<form>` → `<header>` (statusbar + buttons) → `<sheet>` → `<group>` → fields
- List views: `<list>` (not `<tree>` — deprecated in 17+)
- Use `<notebook>` + `<page>` for tabbed sections in forms
- Use `attrs` / `invisible` for conditional field visibility
- Use `decoration-*` attributes for colored list rows

### View inheritance
- Use `inherit_id` + XPath expressions to extend views
- Prefer `position="inside"`, `position="after"`, `position="before"`
- Never copy entire views to modify — always inherit

## Security

### Access rights (ir.model.access.csv) — MANDATORY for every model
```csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_module_model_group_user,module.model.user,model_module_model,base.group_user,1,1,1,0
access_module_model_group_manager,module.model.manager,model_module_model,module.group_manager,1,1,1,1
```

### Record rules for multi-company
```xml
<record id="module.model_company_rule" model="ir.rule">
    <field name="name">Model: multi-company</field>
    <field name="model_id" ref="model_module_model"/>
    <field name="domain_force">[('company_id', 'in', company_ids)]</field>
</record>
```

## OWL Components (Frontend)
- Component class extends `Component` from `@odoo/owl`
- Template naming: `{module_name}.{ComponentName}`
- Register in actions: `registry.category("actions").add("tag", Component)`
- Static props: always declare `static props = {}` for validation
- Use `@web/core/utils/hooks` for Odoo-specific hooks
- Assets go in `__manifest__.py` under `assets` key

## Testing

### Test classes
- Business logic: `TransactionCase` (rolls back after each test)
- HTTP/controllers: `HttpCase`
- UI/tours: `HttpCase` with `browser_js()`
- Common setup: override `setUpClass(cls)` with `super().setUpClass()`

### Test file naming
- `tests/test_{model_name}.py`
- Test class: `TestModelName(TransactionCase)`
- Test methods: `test_{behavior_description}`

### What to test
- CRUD operations with valid and invalid data
- Computed fields: verify recomputation on dependency changes
- Constraints: verify `ValidationError` is raised
- Workflow transitions: verify state changes and side effects
- Access rights: verify `AccessError` for unauthorized users (use `self.env(user=...)`)

## Odoo Anti-patterns (never do these)
1. Raw SQL (`self.env.cr.execute()`) — use ORM unless proven performance need
2. `self.env.cr.commit()` — Odoo manages transactions
3. `@api.one` or `@api.multi` — both deprecated
4. Forgetting `super()` in CRUD overrides
5. Creating models without `ir.model.access.csv` entries
6. Creating files without updating `__manifest__.py`
7. Hardcoding XML IDs without module prefix
8. Using `<tree>` instead of `<list>` in Odoo 17+
9. Business logic in `@api.onchange` — use `@api.constrains` or computed fields
10. Ignoring `_description` on models — Odoo logs warnings

## Translations
- Use `_()` for translatable user-facing strings in Python
- Lazy translation: `_lt()` for class-level strings (field labels are auto-translated)
- NEVER use f-strings inside `_()` — use `%` formatting: `_("Hello %s") % name`
- XML: user-facing text in views is auto-translatable

## Verification (Odoo-specific)
After every change:
1. Run `ruff check . --fix && ruff format .`
2. Run module tests: `--test-enable --stop-after-init -i <module>`
3. Verify module installs cleanly: `-i <module> --stop-after-init` (no errors)
4. Check logs for warnings about missing access rules or descriptions

---

## Claude Sessions
