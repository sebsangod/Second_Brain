---
aliases:
  - Pydantic
tags:
  - learning
date: 2026-03-15
---
**Sources**: [Pydantic](https://pydantic.dev/), [Pydantic Docs](https://docs.pydantic.dev/latest/)

**Tags:** #development #data #datavalidation

**Related:** [[Python]], [[Rust]], [[Huggingface]], [[LangChain]]

---

## Description

_Pydantic_ is the most widely used data validation library for `Python`.

Fast and extensible, _Pydantic_ plays nicely with your linters/IDE/brain. Define how data should be in pure, canonical `Python` 3.9+; validate it with _Pydantic_.

Installing Pydantic is as simple as: _pip install pydantic_

---

## Key concepts

- **Powered by type hints:** with _Pydantic_, schema validation and serialization are controlled by type annotations; less to learn, less code to write, and integration with your IDE and static analysis tools.
- **Speed:** _Pydantic's_ core validation logic is written in `Rust`. As a result, _Pydantic_ is among the fastest data validation libraries for `Python`.
- **JSON Schema:** — _Pydantic_ models can emit JSON Schema, allowing for easy integration with other tools.
- **Strict and Lax mode:** _Pydantic_ can run in either _strict mode_ (where data is not converted) or _lax mode_ where _Pydantic_ tries to coerce data to the correct type where appropriate.
- **Dataclasses, TypedDicts and more:** _Pydantic_ supports validation of many standard library types including _dataclass_ and _TypedDict_
- **Customization:** _Pydantic_ allows custom validators and serializers to alter how data is processed in many powerful ways.
- **Ecosystem:** around 8,000 packages on _PyPI_ use _Pydantic_, including massively popular libraries like `FastAPI`, `huggingface`, _SQLModel_, & `LangChain`.

---

## Examples

To see _Pydantic_ at work, let's start with a simple example, creating a custom class that inherits from _BaseModel_:


### Validation Successful

```python title:models.py
from datetime import datetime

from pydantic import BaseModel, PositiveInt


class User(BaseModel):
    id: int
    name: str = "John Doe"
    signup_ts: datetime | None
    tastes: dict[str, PositiveInt]


external_data = {
    "id": 123,
    "signup_ts": "2019-06-01 12:22",
    "tastes": {
        "wine": 9,
        b"cheese": 7,
        "cabbage": '1',
    },
}

user = User(**external_data)

print(user.id)
#> 123
print(user.model_dump())
"""
{
    "id": 123,
    "name": "John Doe",
    "signup_ts": datetime.datetime(2019, 6, 1, 12, 22),
    "tastes": {"wine": 9, "cheese": 7, "cabbage": 1},
}
"""

```

If validation fails, _Pydantic_ will raise an error with a breakdown of what was wrong:

### Validation Error

```python title:models.py
# continuing the above example...

from datetime import datetime
from pydantic import BaseModel, PositiveInt, ValidationError


class User(BaseModel):
    id: int
    name: str = "John Doe"
    signup_ts: datetime | None
    tastes: dict[str, PositiveInt]


external_data = {"id": "not an int", "tastes": {}}

try:
    User(**external_data)
except ValidationError as e:
    print(e.errors())
    """
    [
        {
            "type": "int_parsing",
            "loc": ("id",),
            "msg": "Input should be a valid integer, unable to parse string as an integer",
            "input": "not an int",
            "url": "https://errors.pydantic.dev/2/v/int_parsing",
        },
        {
            "type": "missing",
            "loc": ("signup_ts",),
            "msg": "Field required",
            "input": {"id": "not an int", "tastes": {}},
            "url": "https://errors.pydantic.dev/2/v/missing",
        },
    ]
    """

```

---

## Utils

### Email validation

```python title:models.py
from datetime import datetime
from pydantic import BaseModel, EmailStr, PositiveInt


class User(BaseModel):
    id: int
    name: str = "John Doe"
    email: EmailStr = "example@gmail.com"
    signup_ts: datetime | None
    tastes: dict[str, PositiveInt]

```

## Claude Sessions
