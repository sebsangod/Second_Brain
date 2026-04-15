---
aliases:
  - Odoo Owl
tags:
  - learning
---
**Date**: 28/mar/2026
**Update**: 28/mar/2026
**Sources**: [Discover the web framework](https://www.odoo.com/documentation/19.0/developer/tutorials/discover_js_framework/01_owl_components.html)

**Tags:** #development #frontend

**Related:** [[Odoo]], [[JavaScript]], [[XML]]

---

## Description

This chapter introduces the [Owl framework](https://github.com/odoo/owl), a tailor-made component system for Odoo. The main building blocks of OWL are [components](https://github.com/odoo/owl/blob/master/doc/reference/component.md) and [templates](https://github.com/odoo/owl/blob/master/doc/reference/templates.md).

In Owl, every part of user interface is managed by a component: they hold the logic and define the templates that are used to render the user interface. In practice, a component is represented by a small JavaScript class subclassing the _Component_ class.

---

## Utils

### Mount a Owl Component on a Blank Page

```txt title:awesome_owl_tutorial
awesome_owl_tutorial/
├── controllers
│   ├── __init__.py
│   └── awesome_owl.py
├── static
│   └── src
│       ├── components
│       │   └── playground
│       │       ├── playground.js
│       │       └── playground.xml
│       └── main.js
├── views
│   └── templates.xml
├── __init__.py
└── __manifest__.py
```

```python title:__manifest__.py
{
	...
	"depends": ["base", "web"],
	"data": ["views/templates.xml"],
	"assets": {
		"awesome_owl_tutorial.assets_playground": [
			# Dependencies
			("include", "web._assets_helpers"),
			("include", "web._assets_backend_helpers"),
			"web/static/src/scss/pre_variables.scss",
			"web/static/lib/bootstrap/scss/_variables.scss",
			"web/static/lib/bootstrap/scss/_maps.scss",
			("include", "web._assets_bootstrap"),
			("include", "web._assets_core"),
			"web/static/src/libs/fontawesome/css/font-awesome.css",
			# Module files
			"awesome_owl_tutorial/static/src/main.js",
			"awesome_owl_tutorial/static/src/components/**/*",
		],
	},
	...
}
```

```python title:awesome_owl.py
from odoo import http
from odoo.http import request
  

class OwlPlayground(http.Controller):
	@http.route(["/awesome_owl"], type="http", auth="public")
	def show_playground(self) -> str:
		"""Renders the owl playground page"""
		return request.render("awesome_owl_tutorial.playground_template")

```

```xml title:templates.xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
	<template id="awesome_owl_tutorial.playground_template" name="Awesome Owl Tutorial">
		<t t-set="head">
			<link type="image/x-icon" rel="shortcut icon" href="/web/static/img/favicon.ico" />
			<t t-call-assets="awesome_owl_tutorial.assets_playground" />
		</t>
	
		<t t-set="body" />
	</template>
</odoo>
```

```js title:main.js
import { whenReady } from "@odoo/owl";
import { mountComponent } from "@web/env";
import { Playground } from "./components/playground/playground";


const config = {
	dev: true,
	name: "Awesome Owl Tutorial",
};

whenReady(() => mountComponent(Playground, document.body, config));
```

```js title:playground.js
import { Component } from "@odoo/owl";

export class Playground extends Component {
	static template = "awesome_owl_tutorial.Playground";
	static props = {};
}
```

```xml title:playground.xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
	<t t-name="awesome_owl_tutorial.Playground">
		<div class="p-3">Hello world!!!!</div>
	</t>
</templates>

```

#### Result
![[owl_blank_page.png]]


### Mount a Owl Component Inside Odoo Form View

```txt title:awesome_owl_tutorial
├── controllers
│   ├── __init__.py
│   └── dashboard_owl.py # Used to load test random data only
├── static
│   └── src
│       └── components
│           └── dashboard
│               ├── dashboard.js
│               ├── dashboard.scss
│               └── dashboard.xml
├── views
│   ├── actions.xml
│   └── menus.xml
├── __init__.py
└── __manifest__.py
```

```python title:__manifest__.py
{
	...
	"depends": ["base", "web", "mail"],
	"data": [
		"views/actions.xml",
		"views/menus.xml",
	],
	"assets": {
		"web.assets_backend": [
			"dashboard_owl_tutorial/static/src/**/*",
			("remove", "dashboard_owl_tutorial/static/src/dashboard/**/*"),
		],
		"dashboard_owl_tutorial.dashboard": [
			"dashboard_owl_tutorial/static/src/dashboard/**/*"
		],
	},
	...
}
```

```xml title:actions.xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
	<record id="dashboard_owl_tutorial.dashboard_owl_action" model="ir.actions.client">
		<field name="name">Dashboard</field>
		<field name="tag">dashboard_owl_tutorial.dashboard_owl_action</field>
	</record>
</odoo>
```

```xml title:menus.xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
	<menuitem
		id="dashboard_owl_tutorial.dashboard_owl_menu_root"
		name="Awesome Dashboard"
		sequence="1"
		groups="base.group_user"
>	
		<menuitem
			id="dashboard_owl_tutorial.dashboard_owl_menu_dashboard"
			name="Dashboard"
			action="dashboard_owl_tutorial.dashboard_owl_action"
			sequence="1"
		/>
	</menuitem>
</odoo>
```

```js title:playground.js
import { Component } from "@odoo/owl";

class AwesomeDashboard extends Component {
	static template = "dashboard_owl_tutorial.AwesomeDashboard";
	
	static props = {}
	
	...
}

// This is the important part
// Used to link the component to the action defined in actions.xml
registry.category("actions").add(
    "dashboard_owl_tutorial.dashboard_owl_action",
    AwesomeDashboard
);
```

#### Result
![[owl_view_form.png]]

---