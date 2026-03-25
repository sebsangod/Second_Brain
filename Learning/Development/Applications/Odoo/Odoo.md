---
aliases:
  - Odoo
tags:
  - learning
---
**Date**: 24/mar/2026
**Update**: 24/mar/2026
**Sources**: [Odoo](https://www.odoo.com/es), [Odoo Developers Guide](https://www.odoo.com/documentation/19.0/developer.html)

**Tags:** #ERP #CRM #opensource

**Related:** [[Python]], [[JavaScript]], [[XML]]

---

## Description

_Odoo_ is a suite of _open-source_ enterprise applications (`ERP`) designed to **comprehensively manage a business on a single platform**: sales, accounting, inventory, human resources, and marketing. Its modular structure allows users to activate applications based on business needs, facilitating scalability from small businesses to large corporations.

---

## Key concepts

* **All-in-One** `ERP` **Software:** Integrates all areas of the business into a single system, eliminating the need for multiple disparate software applications.
* **Modularity:** Users can start with a single application (e.g., `CRM`) and add modules (inventory, accounting, website) as the business grows.
* **Open Source:** The Community version is free, while the Enterprise version offers more features via subscription.
* **Full Integration:** Modules are interconnected. For example, a confirmed sale automatically generates the inventory delivery note and the accounting invoice.
* **Intuitive and Flexible Interface:** Designed to be user-friendly, with customizable applications that adapt to workflows in any industry.
* **Cloud-Based:** Allows access to company information and management from anywhere and on any device.


### Main Modules:
* CRM
* Sales
* Inventory
* Accounting
* Point of Sale
* Manufacturing (MRP)
* Projects
* Website
* E-commerce.

---

## Details

Write here...

---

## Utils

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

---