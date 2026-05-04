---
aliases:
  - CLAUDE.md
tags:
  - dev/backend
  - dev/api
date: 2026-05-02
---
**Sources**: [FastAPI Docs](https://fastapi.tiangolo.com/), [Pydantic v2](https://docs.pydantic.dev/latest/), [Beanie ODM](https://beanie-odm.dev/)

**Related:** [[CLAUDE File]], [[FastAPI]], [[Pydantic]], [[Beanie]], [[Testing]]

---

# FastAPI Senior Backend Development

## Project
Python 3.12+ async API service.
Stack: FastAPI + Pydantic v2 + Beanie (MongoDB ODM) + pytest.

## Commands
- Dev server: `uvicorn app.main:app --reload --host 0.0.0.0 --port 8000`
- Tests: `python -m pytest tests/ -v --tb=short`
- Tests (single file): `python -m pytest tests/test_module.py -v --tb=short`
- Tests (single test): `python -m pytest tests/test_module.py::TestClass::test_method -v`
- Coverage: `python -m pytest --cov=app --cov-report=term-missing`
- Lint: `ruff check . --fix && ruff format .`
- Deps: `uv add <package>` / `uv remove <package>`
- Lock: `uv lock`

## Project Structure
```
project/
├── app/
│   ├── __init__.py
│   ├── main.py                  # FastAPI instance, lifespan, error handlers
│   ├── config.py                # Settings via pydantic-settings, paths
│   ├── dependencies.py          # Shared Depends() providers
│   ├── logger.py                # Custom TZ-aware logger setup
│   ├── exceptions.py            # Domain exception classes
│   ├── error_handlers.py        # app.add_exception_handler registrations
│   │
│   ├── domain/                  # Business entities (pure, no framework deps)
│   │   ├── __init__.py
│   │   └── models.py
│   │
│   ├── infrastructure/          # DB connections, external service clients
│   │   ├── __init__.py
│   │   ├── database.py          # MongoDB/Beanie init
│   │   └── security.py          # Hashing, encryption, JWT
│   │
│   ├── features/                # Feature modules (vertical slices)
│   │   └── {feature}/
│   │       ├── __init__.py
│   │       ├── router.py        # APIRouter endpoints
│   │       ├── schemas.py       # Request/Response Pydantic models
│   │       ├── service.py       # Business logic (use cases)
│   │       ├── repository.py    # Data access (ABC + implementation)
│   │       └── dependencies.py  # Feature-specific Depends()
│   │
│   └── shared/                  # Cross-cutting utilities
│       ├── __init__.py
│       ├── responses.py         # Standard response builders
│       └── utils.py
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py              # Shared fixtures, DB mocks
│   └── {feature}/
│       ├── __init__.py
│       └── test_{feature}.py
│
├── .env
├── .env.example
├── pyproject.toml
└── uv.lock
```

## FastAPI Application Setup

### Lifespan pattern (startup/shutdown)
```python
from collections.abc import AsyncGenerator
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    # Startup: init DB, warm caches, connect services
    await database.init()
    yield
    # Shutdown: close connections, flush logs
    await database.close()

app = FastAPI(
    title="Service Name",
    version="1.0.0",
    lifespan=lifespan,
)
```

### Error handler registration order
```python
# Most specific → least specific
app.add_exception_handler(CustomDomainError, domain_error_handler)
app.add_exception_handler(ValidationError, validation_handler)
app.add_exception_handler(RequestValidationError, validation_handler)
app.add_exception_handler(HTTPException, http_handler)
app.add_exception_handler(Exception, general_handler)
```

### Router conventions
- One `APIRouter` per feature module
- Always declare: `path`, `summary`, `response_model`, `status_code`, `responses`
- Prefix routers on include: `app.include_router(router, prefix="/v1/feature", tags=["Feature"])`
- Use `status.HTTP_xxx` constants, never raw integers

## Pydantic v2 Conventions

### Schema naming
- Request DTOs: `{Action}{Resource}Request` → `CreateUserRequest`
- Response DTOs: `{Resource}Response` → `UserResponse`
- Internal models: `{Resource}` → `User`

### Schema rules
- `model_config = ConfigDict(strict=True)` when you need strict validation
- Use `Field()` with `min_length`, `max_length`, `ge`, `le`, `pattern` — validate at the boundary
- Use `EmailStr` for email fields (install `pydantic[email]`)
- Use `model_dump()` not `.dict()` — v1 API is deprecated
- Use `model_json_schema()` not `.schema()` — v1 API is deprecated
- Separate request schemas from response schemas — never expose internal fields (passwords, tokens)
- Use `Annotated[type, Field(...)]` syntax for modern Pydantic patterns

### Settings management
```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=False,
    )
    database_url: str
    secret_key: str
    debug: bool = False

settings = Settings()
```

## Beanie / MongoDB Conventions

### Document models
- Extend `Document`, not `BaseModel`, for collections
- Always set `class Settings: name = "collection_name"`
- Use `Indexed(type)` for indexed fields
- Use `IndexModel` for compound and TTL indexes
- Init in lifespan: `await init_beanie(database=db, document_models=[...])`

### Repository pattern (mandatory)
```python
from abc import ABC, abstractmethod

class UserRepositoryInterface(ABC):
    @abstractmethod
    async def get_by_id(self, user_id: str) -> User | None: ...

    @abstractmethod
    async def get_by_email(self, email: str) -> User | None: ...

    @abstractmethod
    async def create(self, user: User) -> User: ...

class UserRepository(UserRepositoryInterface):
    async def get_by_id(self, user_id: str) -> User | None:
        return await User.get(PydanticObjectId(user_id))
```
- ABC interface defines the contract — implementation lives in infrastructure
- Inject repository via `Depends()`, never import directly in routers

## Dependency Injection

### Use `Depends()` for everything
```python
from fastapi import Depends

async def get_db() -> AsyncGenerator[Database, None]:
    yield database_instance

async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    return decode_and_verify(token)

@router.get("/me")
async def read_me(user: User = Depends(get_current_user)) -> UserResponse:
    return UserResponse.model_validate(user)
```
- Chain dependencies: `get_current_user` → `get_db` → `get_settings`
- Use `Annotated` for reusable dependency types: `CurrentUser = Annotated[User, Depends(get_current_user)]`

## Async Patterns

### Rules
- All route handlers: `async def`, never `def` (unless CPU-bound with `run_in_executor`)
- All DB operations: `await` — Beanie is fully async
- CPU-bound work: offload to `loop.run_in_executor(None, blocking_fn)`
- Never use `asyncio.get_event_loop()` — use `asyncio.get_running_loop()` in async context
- Never call blocking I/O (file reads, SMTP, bcrypt) directly in async handlers — always executor

## Error Handling

### Exception hierarchy
```python
# Base domain exception
class ServiceError(Exception):
    def __init__(self, message: str, status_code: int = 500) -> None:
        self.message = message
        self.status_code = status_code

class NotFoundError(ServiceError):
    def __init__(self, resource: str, identifier: str) -> None:
        super().__init__(f"{resource} '{identifier}' not found", 404)

class ConflictError(ServiceError):
    def __init__(self, message: str) -> None:
        super().__init__(message, 409)
```
- Domain code raises domain exceptions — never `HTTPException`
- Error handlers translate domain exceptions to HTTP responses
- Always return consistent response shape: `{"success": bool, "message": str, "data": ...}`

## Testing

### Test structure
- `conftest.py`: shared fixtures — mock DB, mock auth, test client
- Feature tests mirror feature structure: `tests/{feature}/test_{feature}.py`
- Test class per concern: `TestCreateUser`, `TestUserValidation`, `TestUserAuth`

### Fixtures
```python
@pytest.fixture
def client() -> TestClient:
    with TestClient(app) as c:
        yield c

@pytest.fixture(autouse=True)
def mock_mongodb():
    """Prevent real DB connections in all tests."""
    with patch("app.infrastructure.database.init", new_callable=AsyncMock):
        yield
```

### Sync vs async tests
- **Sync** (default): `TestClient` — wraps ASGI app, no `async def` needed
- **Async** (when required): `AsyncClient` + `@pytest.mark.anyio` + `httpx.ASGITransport`
- Prefer sync `TestClient` unless testing async-specific behavior

### What to test per endpoint
1. **Happy path**: correct status code + response shape
2. **Validation**: missing fields → 422, wrong types → 422
3. **Auth**: missing token → 401, invalid token → 401, wrong role → 403
4. **Not found**: nonexistent resource → 404
5. **Conflict**: duplicate creation → 409
6. **Edge cases**: empty strings, boundary values, Unicode

### Mocking strategy
- Mock at the **repository boundary** — never mock internal service logic
- Use `unittest.mock.patch` targeting the **import path where it's used**, not where it's defined
- Use `new_callable=AsyncMock` for all async functions
- Use `side_effect` for dynamic mock behavior, `return_value` for static

## Security Patterns
- Passwords: bcrypt via `passlib` — always hash, never store plaintext
- Tokens: JWT with expiry — short-lived access + long-lived refresh
- Secrets: Fernet for reversible encryption when needed
- All secrets from `.env` — never import from code, use `Settings` class
- Rate limiting: add middleware for auth endpoints
- CORS: configure explicitly — never `allow_origins=["*"]` in production

## Anti-patterns (never do these)
1. Business logic in route handlers — extract to service/use case layer
2. `HTTPException` in service/domain code — raise domain exceptions
3. Sync `def` route handlers that do I/O
4. `response.dict()` or `model.schema()` — use Pydantic v2 API
5. Circular imports between features — depend on shared/domain, not on each other
6. Hardcoded status codes as integers — use `status.HTTP_xxx`
7. Missing `response_model` on endpoints — always declare return schema
8. `asyncio.get_event_loop()` in async context — use `get_running_loop()`
9. Testing with a live database — always mock at repository boundary
10. Returning raw dicts from endpoints — always use Pydantic response models

## Verification (FastAPI-specific)
After every change:
1. `ruff check . --fix && ruff format .`
2. `python -m pytest tests/ -v --tb=short` — all tests pass
3. Start dev server: verify `/docs` loads, endpoints respond correctly
4. Check OpenAPI schema: `GET /openapi.json` — no warnings, schemas complete

---

## Claude Sessions
