---
aliases:
  - Testing
tags:
  - learning
---
**Date**: 25/mar/2026
**Update**: 25/mar/2026
**Sources**: [FastAPI Testing](https://fastapi.tiangolo.com/tutorial/testing/#using-testclient) [FastAPI Async Testing](https://fastapi.tiangolo.com/advanced/async-tests/#other-asynchronous-function-calls)

**Tags:** #development #testing

**Related:** [[FastAPI]], [[Python]], [[Starlette]], [[HTTPX]], [[Requests]], [[Pytest]], [[Database]], [[Backend]], [[Beanie]], [[MongoDB]]

---

## Description

Thanks to `Starlette`, testing _FastAPI_ applications is easy and enjoyable.

It is based on `HTTPX` (_pip install httpx_), which in turn is designed based on `Requests`, so it's very familiar and intuitive.

With it, you can use `pytest` (_pip install pytest_) directly with _FastAPI_.

---

## Examples

### Using _TestClient_

1. Create a _TestClient_ by passing your _FastAPI_ application to it.
2. Create functions with a name that starts with _test_ (this is standard `pytest` conventions).
3. Use the _TestClient_ object the same way as you do with `httpx`.
4. Write simple _assert_ statements with the standard `Python` expressions that you need to check (again, standard `pytest`).

```txt title:my_api/
.
├── __init__.py
├── main.py
└── test_main.py
```

```python title:main.py
from typing import Annotated

from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel


fake_secret_token = "my_secret_token"

fake_db = {
    "foo": {"id": "foo", "title": "Foo", "description": "There goes my hero"},
    "bar": {"id": "bar", "title": "Bar", "description": "The bartenders"},
}

app = FastAPI()


class Item(BaseModel):
    id: str
    title: str
    description: str | None = None


@app.get("/items/{item_id}", response_model=Item)
async def read_main(item_id: str, x_token: Annotated[str, Header()]):
    if x_token != fake_secret_token:
        raise HTTPException(status_code=400, detail="Invalid X-Token header")
    if item_id not in fake_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return fake_db[item_id]


@app.post("/items/")
async def create_item(item: Item, x_token: Annotated[str, Header()]) -> Item:
    if x_token != fake_secret_token:
        raise HTTPException(status_code=400, detail="Invalid X-Token header")
    if item.id in fake_db:
        raise HTTPException(status_code=409, detail="Item already exists")
    fake_db[item.id] = item.model_dump()
    return item

```

```python title:test_main.py
from fastapi.testclient import TestClient

from .main import app


client: TestClient = TestClient(app)


def test_read_item():
    response = client.get("/items/foo", headers={"X-Token": "coneofsilence"})
    assert response.status_code == 200
    assert response.json() == {
        "id": "foo",
        "title": "Foo",
        "description": "There goes my hero",
    }


def test_read_item_bad_token():
    response = client.get("/items/foo", headers={"X-Token": "hailhydra"})
    assert response.status_code == 400
    assert response.json() == {"detail": "Invalid X-Token header"}


def test_read_nonexistent_item():
    response = client.get("/items/baz", headers={"X-Token": "coneofsilence"})
    assert response.status_code == 404
    assert response.json() == {"detail": "Item not found"}


def test_create_item():
    response = client.post(
        "/items/",
        headers={"X-Token": "coneofsilence"},
        json={"id": "foobar", "title": "Foo Bar", "description": "The Foo Barters"},
    )
    assert response.status_code == 200
    assert response.json() == {
        "id": "foobar",
        "title": "Foo Bar",
        "description": "The Foo Barters",
    }


def test_create_item_bad_token():
    response = client.post(
        "/items/",
        headers={"X-Token": "hailhydra"},
        json={"id": "bazz", "title": "Bazz", "description": "Drop the bazz"},
    )
    assert response.status_code == 400
    assert response.json() == {"detail": "Invalid X-Token header"}


def test_create_existing_item():
    response = client.post(
        "/items/",
        headers={"X-Token": "coneofsilence"},
        json={
            "id": "foo",
            "title": "The Foo ID Stealers",
            "description": "There goes my stealer",
        },
    )
    assert response.status_code == 409
    assert response.json() == {"detail": "Item already exists"}

```


#### Tips

> [!tip] Tip
> Notice that the testing functions are normal _def_, not _async def_.
> 
> And the calls to the client are also normal calls, not using _await_.
> 
> This allows you to use `pytest` directly without complications.

> [!tip] Tip
> Whenever you need the client to pass information in the request and you don't know how to, you can search (Google) how to do it in **httpx**, or even how to do it with requests, as **HTTPX's** design is based on **Requests'** design.
> 
> Then you just do the same in your tests. E.g.:
> - To pass a path or query parameter, add it to the URL itself.
> - To pass a JSON body, pass a **Python** object (e.g. a dict) to the parameter json.
> - If you need to send Form Data instead of JSON, use the data parameter instead.
> - To pass headers, use a dict in the headers parameter.
> - For cookies, a dict in the cookies parameter.


#### Execution

```bash title:/home/user/my_api/app
$ pytest

================ test session starts ================  
platform linux -- Python 3.6.9, pytest-5.3.5, py-1.8.1, pluggy-0.13.1  
rootdir: /home/user/code/superawesome-cli/app  
plugins: forked-1.1.3, xdist-1.31.0, cov-2.8.1  
collected 6 items  
  
████████████████████████████████████████ 100%  
test_main.py ...... [100%]  
  
================= 1 passed in 0.03s =================

```


### Async Tests

Being able to use asynchronous functions in your tests could be useful, for example, when you're querying your `database` _asynchronously_. Imagine you want to test sending requests to your _FastAPI_ application and then verify that your `backend` successfully wrote the correct data in the `database`, while using an _async_ `database` library.


#### pytest.mark.anyio

If we want to call _asynchronous_ functions in our tests, our test functions have to be asynchronous. `AnyIO` provides a neat plugin for this, that allows us to specify that some test functions are to be called _asynchronously_.

```txt title:my_async_api/
.
├── __init__.py
├── main.py
└── test_main.py
```

```python title:main.py
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Tomato"}

```

```python title:test_main.py
import pytest
from httpx import ASGITransport, AsyncClient

from .main import app


@pytest.mark.anyio
async def test_root():
    async with AsyncClient(
        transport=ASGITransport(app=app), base_url="http://test"
    ) as ac:
        response = await ac.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Tomato"}

```


#### In detail

The marker @pytest.mark.anyio tells `pytest` that this test function should be called _asynchronously_.

Then we can create an _AsyncClient_ with the app, and send async requests to it, using _await_. This is the equivalent to:

```python
response = client.get('/')
```

...that we used to make our requests with the _TestClient_.

> [!warning] Warning
> If your application relies on lifespan events, the **AsyncClient** won't trigger these events. To ensure they are triggered, use [LifespanManager](https://github.com/florimondmanca/asgi-lifespan#usage).

---

## Utils

### Working with `Beanie` for `MongoDB`

Write here...

```python title:main.py
print("Hello world!")

```

---