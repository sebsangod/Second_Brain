---
aliases:
  - Odoo Developers
tags:
  - learning
---
**Date**: 26/mar/2026
**Update**: 26/mar/2026
**Sources**: [Odoo Developers](https://www.odoo.com/documentation/19.0/developer.html)

**Tags:** #learning

**Related:** [[Odoo]], [[Multitier Architecture]], [[Database]], [[Python]]

---

## Details

### Architecture

_Odoo_ follows a [multitier architecture](https://en.wikipedia.org/wiki/Multitier_architecture), **meaning that the presentation, the business logic and the data storage are separated.** More specifically, it uses a `three-tier architecture`.


### Odoo modules

Both server and client extensions are packaged as modules which are optionally loaded in a `database`. **A module is a collection of functions and data that target a single purpose.**

_Odoo_ _modules_ can either add brand new business logic to an _Odoo_ system or alter and extend existing business logic. One _module_ can be created to add your country’s accounting rules to _Odoo’s_ generic accounting support, while a different _module_ can add support for real-time visualization of a bus fleet.

**Everything in Odoo starts and ends with** _modules_.

Terminology: developers group their business features in _Odoo_ _modules_. The main user-facing modules are flagged and exposed as _Apps_, but a majority of the _modules_ aren’t _Apps_. _Modules_ may also be referred to as _addons_ and the directories where the _Odoo server_ finds them form the _addons_path_.


#### Module structure

Each module is a directory within a _module directory_. _Module directories_ are specified by using the _--addons-path_ option.

An _Odoo_ _module_ is declared by its `__manifest__.py`

When an Odoo module includes business objects (i.e. `Python` files), they are organized as a `Python` with a `__init__.py` file. This file contains import instructions for various `Python` files in the module.

Here is a simplified module directory:

```txt title:my_module/
my_module
├── controllers
│   ├── *.py
│   └── __init__.py
├── models
│   ├── *.py
│   └── __init__.py
├── data
│   └── *.xml
├── views
│   └── *.xml
├── security
│   └── *.csv
├── __init__.py
└── __manifest__.py
```


### Access Rights

>[!info] **Reference**: the documentation related to this topic can be found in [Access Rights](https://www.odoo.com/documentation/19.0/developer/reference/backend/security.html#reference-security-acl).

When no access rights are defined on a model, _Odoo_ **determines that no users can access the data.** It is even notified in the log:

```bash title:bash
WARNING my_database odoo.modules.loading: The models ['my_module.model'] have no access rules in module estate, consider adding some, like:
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
```

_Access rights_ are defined as records of the model `ir.model.access`. **Each** _access right_ **is associated with a model, a group (or no group for global access) and a set of permissions**: create, read, write and unlink. Such access rights are usually defined in a CSV file named `ir.model.access.csv`.

Here is an example for the _test_model_:

```csv title:actions.csv
id,name,model_id/id,group_id/id,perm_read,perm_write,perm_create,perm_unlink
access_<test_model>_group_<group_name>,<test_model>,model_<test_model>,base.group_user,1,0,0,0
```

Where:
- _id_ is an [external identifier](https://www.odoo.com/documentation/19.0/developer/glossary.html#term-external-identifier).
- _name_ is the name of the `ir.model.access`.
- _model_id/id_ refers to the model which the access right applies to. The standard way to refer to the model is `model_<model_name>`, where _<model_name>_ is the `_name` of the model with the `.` replaced by `_`.
- _group_id/id_ refers to the group which the access right applies to.


### Actions

>[!info] **Reference**: the documentation related to this topic can be found in [Actions](https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html).

Actions can be triggered in three ways:
1. by clicking on menu items (linked to specific actions)
2. by clicking on buttons in views (if these are connected to actions)
3. as contextual actions on object

**The action can be viewed as the link between the menu and the model.**

A basic action for the `test_model` is:

```xml title:actions.xml
<?xml version="1.0" encoding="UTF-8"?>
<record id="test_model.test_model_action" model="ir.actions.act_window">
    <field name="name">Test action</field>
    <field name="res_model">test_model</field>
    <field name="view_mode">list,form</field>
</record>
```

Where:
- _id_ is an [external identifier](https://www.odoo.com/documentation/19.0/developer/glossary.html#term-external-identifier). It can be used to refer to the record (without knowing its in-database identifier).
- _model_ has a fixed value of [ir.actions.act_window](https://www.odoo.com/documentation/19.0/developer/reference/backend/actions.html#reference-actions-window)).
- _name_ is the name of the action.
- _res_model_ is the model which the action applies to.
- _view_mode_ are the views that will be available.


### Menus

```xml title:menus.xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
	<menuitem id="my_module.menu_root" name="Real Estate" sequence="5">
		<menuitem id="my_module.first_menu" name="Advertisements" sequence="5">
			<menuitem id="my_module.sub_menu" name="Properties" sequence="5" action="my_module.properties_action" />
		</menuitem>
	
		<menuitem id="my_module.second_menu" name="Settings" sequence="10">
			<menuitem id="my_module.first_sub_menu" name="Property Types" sequence="5" action="my_module.property_type_action" />
			<menuitem id="my_module.second_sub_menu" name="Property Tags" sequence="10" action="my_module.property_tag_action" />
		</menuitem>
	</menuitem>
</odoo>
```

This will give us the following result:

![[menu_root.png]]

![[menus.png]]

### Computed Fields and Onchanges

>[!info] The related topic can be found [here](https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101/08_compute_onchange.html)

___

## Snippets

### Catalogue List View

```xml title:my_view.xml
<record id="test_module.test_model_view_list" model="ir.ui.view">
	<field name="name">test_module.test_model_view_list</field>
	<field name="model">test.model</field>
	<field name="arch" type="xml">
		<list string="Test Module" editable="bottom">  <!-- This attribute is the important -->
			<field name="name"/>
			...
		</list>
	</field>
</record>
```


### Form Views' Widgets

#### State as a status bar 
```xml title:my_view.xml
<record id="real_estate_tutorial.estate_property_view_form" model="ir.ui.view">
	<field name="name">estate.property.view.form</field>
	<field name="model">estate.property</field>
	<field name="arch" type="xml">
		<form string="Estate Properties">
			<header>
				<field name="state" widget="statusbar" />  <!-- This attribute is the important -->
				...
			</header>
			<sheet>
				...
			</sheet>
		</form>
	</field>
</record>
```

#### Many2many Tags (Colored)
```python title:estate_property_tag.py
from odoo import fields, models


class EstatePropertyTag(models.Model):
	_name = "estate.property.tag"
	_description = "Estate Property Tag Model"
	_order = "name"
	
	
	name = fields.Char("Name", required=True)
	color = fields.Integer("Color Index")

```

```xml title:my_view.xml
<record id="real_estate_tutorial.estate_property_view_form" model="ir.ui.view">
	<field name="name">estate.property.view.form</field>
	<field name="model">estate.property</field>
	<field name="arch" type="xml">
		<form string="Estate Properties">
			<header>
				...
			</header>
			<sheet>
				<div class="oe_title mb32">
					<h1>
						<field name="name" class="mb16" />
					</h1>
					<!-- This field is a many2many related to the estate.property.tag model -->
					<field
						name="tag_ids" 
						widget="many2many_tags"
						options="{'no_create': True, 'no_edit': True, 'color_field': 'color'}"
					/>
				</div>
			</sheet>
		</form>
	</field>
</record>
```


### Add External Libraries
```python title:__manifest__.py
{
    ...,

    "assets": {
        "web.assets_frontend": [
            "my_module/static/src/components/**/*",
            "https://cdnjs.cloudflare.com/ajax/libs/font-awesome/7.0.1/css/all.min.css",
        ]
    }
}

```


### Use Alerts in Fields on List Views
```xml title:my_view.xml
<field
    name="my_field"
    widget="label_selection"
    options="{
        'classes': {
            'state.pending': 'info',
            'state.running': 'success',
            'state.eliminated': 'danger',
            'state.unknown': 'danger'
        }
    }"
/>
```

___

## Utils

### JSON Formatting in Form Views

```python title:my_model.py
from json import dumps

from odoo import fields, models


class TestModel(models.Model):
	_name: str = "test_module.test.model"
	
	my_json = fields.Text(
		"My JSON",
		help="Field to show a JSON pretty formatted",
		readonly=True,
	)
	
	def my_func(self) -> None:
		json: dict[str, Any] = {"hello": "world!"}
		self.my_json = dumps(json, indent=4)

```

```xml title:my_model_view.xml
<record id="test_module.test_model_view_form" model="ir.ui.view">
	<field name="name">test_module.test_model_view_form</field>
	<field name="model">test.model</field>
	<field name="arch" type="xml">
		<form string="Test Module">
			 <!-- Important -->
			<field name="my_json" widget="text" />
			...
		</form>
	</field>
</record>
```


### API Connection and Handling

```bash title:.env
ODOO_URL_PROTOCOL=https
ODOO_PORT=443
ODOO_PROTOCOL="jsonrpc+ssl"
ODOO_HOST="my-odoo.com"
ODOO_DB=my_odoo
ODOO_USER=admin
ODOO_PASSWORD=openpgpwd
```

```python title:odoo_conn.py
from logging import Logger, getLogger
from os import getenv
from typing import Any

import odoorpc
from dotenv import load_dotenv
from erppeek import Client

from backend.config import DOTENV_ABSPATH, LOGGING_PROJECT_NAME

load_dotenv(DOTENV_ABSPATH)
logger: Logger = getLogger(f"{LOGGING_PROJECT_NAME}.{__name__.split('.')[-1]}")


class OdooConnection:
    def __init__(self) -> None:
        logger.debug("Initializing Odoo connection...")
        if getenv("ODOO_PROTOCOL") is not None:
            self.odoo = odoorpc.ODOO(
                host=getenv("ODOO_HOST"),
                port=int(getenv("ODOO_PORT")),
                protocol=getenv("ODOO_PROTOCOL"),  #! For production only
            )
        else:
            self.odoo = odoorpc.ODOO(
                host=getenv("ODOO_HOST"),
                port=int(getenv("ODOO_PORT"))
            )

        self.odoo.login(
            db=getenv("ODOO_DB"),
            login=getenv("ODOO_USER"),
            password=getenv("ODOO_PASSWORD"),
        )
        self.erppeek: Client = Client(
            server=f"{getenv('ODOO_URL_PROTOCOL')}://{getenv('ODOO_HOST')}:{int(getenv('ODOO_PORT'))}",
            db=getenv("ODOO_DB"),
            user=getenv("ODOO_USER"),
            password=getenv("ODOO_PASSWORD"),
        )
        logger.debug("Odoo connection initialized successfully.")

    def search_read(self, model: str, domain: list[str] | None = None, fields: list[str] | None = None) -> list[dict]:
        """Searches and reads records from a model."""
        result = self.odoo.execute(model, "search_read", domain, fields)
        logger.debug(f"Search read result for model '{model}': {result}")
        return result

    def create_record(self, model: str, values: dict[str, Any]) -> int:
        """Creates a new record in a model."""
        return self.odoo.execute(model, "create", values)

    def update_record(self, model: str, record_id: int, values: dict[str, Any]) -> bool:
        """Updates an existing record in a model."""
        return self.odoo.execute(model, "write", [record_id], values)

    def delete_record(self, model: str, record_id: int) -> list[dict]:
        """Deletes a record from a model."""
        return self.odoo.execute(model, "unlink", [record_id])

    def execute_method(self, model: str, method: str, *args: Any, **kwargs: Any) -> Any:
        """Executes a custom method on a model."""
        args = args or []
        kwargs = kwargs or {}
        return self.erppeek.model(model).execute(method, args, kwargs)


odoo_conn: OdooConnection = OdooConnection()

```

---