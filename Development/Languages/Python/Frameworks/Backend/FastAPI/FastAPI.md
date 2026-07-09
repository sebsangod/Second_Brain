---
aliases:
  - FastAPI
tags:
  - learning
  - dev/backend
date: 2026-07-08
---
**Sources**: [FastAPI](https://fastapi.tiangolo.com/) [docs](https://fastapi.tiangolo.com/tutorial/), [FastAPI with layered architecture](https://dev.to/markoulis/layered-architecture-dependency-injection-a-recipe-for-clean-and-testable-fastapi-code-3ioo)


**Related:** [[API Rest]], [[Python]], [[Pydantic]], [[Starlette]], [[OpenAPI]], [[NodeJS]], [[Go]], [[Beanie]], [[Database]], [[MongoDB]], [[Architecture]], [[DDD]], [[Layer]]

---

## Description

_FastAPI_ is a modern, fast (high-performance), web framework for building `APIs` with `Python` based on standard `Python` type hints.

Installing _FastAPI_ is as simple as: _pip install "FastAPI[standard]"_

---

## Key concepts

- **Fast**: Very high performance, on par with `NodeJS` and `Go` (thanks to `Starlette` and `Pydantic`). [One of the fastest `Python` frameworks available](https://fastapi.tiangolo.com/#performance).
- **Fast to code**: Increase the speed to develop features by about 200% to 300%.
- **Fewer bugs**: Reduce about 40% of human (developer) induced errors.
- **Intuitive**: Great editor support. Completion everywhere. Less time debugging.
- **Easy**: Designed to be easy to use and learn. Less time reading docs.
- **Short**: Minimize code duplication. Multiple features from each parameter declaration. Fewer bugs.
- **Robust**: Get production-ready code. With automatic interactive documentation.
- **Standards-based**: Based on (and fully compatible with) the open standards for APIs: `OpenAPI` (previously known as Swagger) and [JSON Schema](https://json-schema.org/).

---

## Snippets

### `DDD` and `Layer` `architecture` file tree

```
my_api/
├── v1/
│   ├── core/
│   │   ├── application/
│   │   │   ├── __init__.py
│   │   │   ├── emails.py
│   │   │   ├── config.py
│   │   │   └── logging.py
│   │   ├── domain/
│   │   │   ├── __init__.py
│   │   │   ├── error_handlers.py
│   │   │   └── exceptions.py
│   │   ├── infrastructure/
│   │   │   ├── __init__.py
│   │   │   ├── models.py
│   │   │   ├── mongo_conn.py
│   │   │   ├── odoo_conn.py
│   │   │   └── redis_conn.py
│   │   ├── presentation/
│   │   │   ├── __init__.py
│   │   │   ├── schemas.py
│   │   │   └── service.py
│   │   └── __init__.py
│   │
│   └── my_function/
│       ├── application/
│       │   ├── __init__.py
│       │   ├── dtos.py
│       │   ├── config.py
│       │   └── use_cases.py
│       ├── infrastructure/
│       │   ├── __init__.py
│       │   ├── models.py
│       │   └── repository.py
│       ├── presentation/
│       │   ├── __init__.py
│       │   ├── router.py
│       │   └── schemas.py
│       └── __init__.py
│
├── .env
├── .env.dev
├── .env.main
├── main.py
├── pyproject.toml
├── README.md
└── requirements.txt
```


### Global absolute paths

```python title:config.py
from os import getcwd
from os.path import join

#######################################################################################################################
################################# Project paths #######################################################################
#######################################################################################################################
PROJECT_DIR_ABSPATH: str = getcwd()
DOTENV_ABSPATH: str = join(PROJECT_DIR_ABSPATH, ".env")
V1_ABSPATH: str = join(PROJECT_DIR_ABSPATH, "v1")

```


#### Other settings inside _config.py_

```python title:config.py
from logging import INFO

#######################################################################################################################
################################# Endpoints ###########################################################################
#######################################################################################################################
PROJECT_V1_PATHS_PREFIX: str = "my_service/v1"

#######################################################################################################################
################################# Variables ###########################################################################
#######################################################################################################################
# Datetime
DATETIME_FORMAT: str = "%Y-%m-%d %H:%M:%S"

# Logging
LOGGING_PROJECT_NAME: str = "MY_SERVICE_V1"
LOGGING_LEVEL: int = INFO
LOGGING_FORMAT: str = "[%(asctime)s] %(levelname)s in %(name)s: %(message)s"
TIMEZONE: str = "America/Mexico_City"

```


### Logging

By default, _FastAPI_ blocks the simple usage of the _logging_ library because it has its own internal _Stream Handler_. So because of this, we need to create our own:

```python title:logging.py
from datetime import datetime
from logging import Formatter, Logger, LogRecord, StreamHandler, getLogger
from typing import TextIO

from pytz import timezone

from .config import (
    DATETIME_FORMAT,
    LOGGING_FORMAT,
    LOGGING_LEVEL,
    LOGGING_PROJECT_NAME,
    TIMEZONE,
)


class TZFormatter(Formatter):
    def __init__(self, fmt: str | None = None, datefmt: str | None = None, tz: str = TIMEZONE) -> None:
        super().__init__(fmt=fmt, datefmt=datefmt)
        self.tz = timezone(tz)

    def format_time(self, record: LogRecord, datefmt: str | None = None) -> str:
        dt: datetime = datetime.fromtimestamp(record.created, self.tz)
        return dt.strftime(datefmt) if datefmt else dt.isoformat()


def setup_logging() -> Logger:
    logger: Logger = getLogger(LOGGING_PROJECT_NAME)
    logger.setLevel(LOGGING_LEVEL)

    handler: StreamHandler[TextIO] = StreamHandler()
    formatter = TZFormatter(fmt=LOGGING_FORMAT, datefmt=DATETIME_FORMAT)
    handler.setFormatter(formatter)

    if not logger.hasHandlers():
        logger.addHandler(handler)

    return logger

```

And connect it with our _FastAPI_ instance:

```python title:main.py
from fastapi import FastAPI
from logging import Logger

from logging import setup_logging


logger: Logger = setup_logging()

app: FastAPI = FastAPI()

```


### Lifespan

To control what an `API` should do at the startup and shutdown execution moments, we need create and connect a _lifespan_, such as the following example:

```python title:main.py
from collections.abc import AsyncGenerator
from contextlib import asynccontextmanager

from fastapi import FastAPI
from httpx import AsyncClient


@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    # ON STARTUP
    app.httpx = AsyncClient()

    try:
        yield
    except Exception as exc:
        logger.error(exc)

    finally:
        # ON SHUTDOWN
        await app.httpx.aclose()

app: FastAPI = FastAPI(
    title="My API",
    summary="My service",
    version="1.0.0",
    docs_url="/docs",
    redoc_url=None,
    openapi_url="/openapi.json",
    lifespan=lifespan,
)

```

As we can see in the example above, the _lifespan_ function uses a _yield_ instance, this helps separating the actions to declare to be executed at startup from the ones to be executed at shutdown.

#### Access Lifespan Definitions

```python title:router.py
from fastapi import APIRouter, Request

from v1.application.use_cases import UseCase


router = APIRouter()


@router.post("/")
async def endpoint(request: Request) -> JSONResponse:
    logger.info("Received request")

    result = await UseCase(request).execute()

    return make_response(
        logger=logger,
        status=status.HTTP_201_CREATED,
        message="Request completed successfully",
    )

```

```python title:use_cases.py
class UseCase:
    def __init__(self, request: Request) -> None:
        self._request: Request = request

    async def execute(self, body: RequestChangePassword) -> str:
        response = await self._request.app.httpx_client.post(
            url="...",
            headers={
                "Content-Type": "application/json",
            },
        )
        ...

```


### Middlewares

#### CORS

```python title:main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware


app: FastAPI = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=False,
    allow_methods=["*"],
    allow_headers=["*"],
)

```


### Main.py ensambling

```python title:main.py
from atexit import register
from collections.abc import AsyncGenerator
from contextlib import asynccontextmanager
from logging import Logger

from cryptography.fernet import InvalidToken
from fastapi import FastAPI, HTTPException, status
from fastapi.exceptions import RequestValidationError
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from httpx import AsyncClient
from jose.exceptions import JWTError
from pydantic import ValidationError
from pydantic_core import ValidationError as CoreValidationError
from pymongo.errors import PyMongoError
from uvicorn import run

from v1.core.application.config import PROJECT_V1_PATHS_PREFIX
from v1.core.application.logging import setup_logging
from v1.core.application.security import Security
from v1.core.domain.error_handlers import (
    bad_gateway_error_handler,
    bad_request_error_handler,
    conflict_error_handler,
    database_error_handler,
    forbidden_error_handler,
    general_exception_handler,
    http_exception_handler,
    invalid_fernet_token_handler,
    invalid_jwt_error_handler,
    not_found_error_handler,
    too_many_requests_error_handler,
    unauthorized_error_handler,
    unprocessable_entity_error_handler,
    validation_exception_handler,
)
from v1.core.domain.exceptions import (
    BadGatewayError,
    BadRequestError,
    ConflictError,
    ForbiddenError,
    NotFoundError,
    TooManyRequestsError,
    UnauthorizedError,
    UnprocessableEntityError,
)
from v1.core.infrastructure.mongo_conn import Mongo
from v1.my_service.presentation.router import router as my_service_router

#######################################################################################################################
################################# Instances ###########################################################################
#######################################################################################################################
logger: Logger = setup_logging()


@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    # ON STARTUP
    app.httpx = AsyncClient()
    app.security = Security()
    app.mongodb = Mongo()

    await app.mongodb.init_db()

    try:
        yield
    except Exception as exc:
        logger.error(exc)

    finally:
        # ON SHUTDOWN
        await app.httpx.aclose()


app = FastAPI(
    title="My API",
    summary="""My service""",
    version="1.0.0",
    docs_url=f"/{PROJECT_V1_PATHS_PREFIX}/docs",
    redoc_url=None,
    openapi_url=f"/{PROJECT_V1_PATHS_PREFIX}/openapi.json",
    lifespan=lifespan,
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=False,
    allow_methods=["*"],
    allow_headers=["*"],
)


#######################################################################################################################
################################# Route Handlers ######################################################################
#######################################################################################################################
app.include_router(
    router=my_service_router,
    prefix=f"/{PROJECT_V1_PATHS_PREFIX}",
    tags=["My Service"],
    include_in_schema=True,
)


#######################################################################################################################
################################# Error Handlers ######################################################################
#######################################################################################################################
# Exception
app.add_exception_handler(Exception, general_exception_handler)
# HTTP
app.add_exception_handler(HTTPException, http_exception_handler)
# Pydantic
app.add_exception_handler(ValidationError, validation_exception_handler)
app.add_exception_handler(RequestValidationError, validation_exception_handler)
app.add_exception_handler(CoreValidationError, validation_exception_handler)
# JWT / Crypto
app.add_exception_handler(JWTError, invalid_jwt_error_handler)
app.add_exception_handler(InvalidToken, invalid_fernet_token_handler)
# Database (Beanie / Motor / PyMongo)
app.add_exception_handler(PyMongoError, database_error_handler)
# Domain
app.add_exception_handler(BadRequestError, bad_request_error_handler)
app.add_exception_handler(UnauthorizedError, unauthorized_error_handler)
app.add_exception_handler(ForbiddenError, forbidden_error_handler)
app.add_exception_handler(NotFoundError, not_found_error_handler)
app.add_exception_handler(ConflictError, conflict_error_handler)
app.add_exception_handler(UnprocessableEntityError, unprocessable_entity_error_handler)
app.add_exception_handler(TooManyRequestsError, too_many_requests_error_handler)
app.add_exception_handler(BadGatewayError, bad_gateway_error_handler)


#######################################################################################################################
################################# Root ################################################################################
#######################################################################################################################
@app.get(path=f"/{PROJECT_V1_PATHS_PREFIX}")
async def root() -> JSONResponse:
    return JSONResponse(
        status_code=status.HTTP_200_OK,
        content={
            "status": "Success",
            "message": """Welcome to My API!""",
            "version": app.version,
        },
    )


#######################################################################################################################
################################# Run #################################################################################
#######################################################################################################################
if __name__ == "__main__":
    run("main:app", host="0.0.0.0", port=8000, reload=True)

```


### Routers Docs' schemas

```python title:v1/core/presentation/service.py
from pydantic import BaseModel

def get_model_schema_for_docs(
    model: BaseModel, responses: dict[int, str]
) -> dict[int, dict[str, Any]]:
    """
    Utility function to get the JSON schema of a Pydantic response model for
    documentation purposes.

    Args:
        model: The Pydantic model class.
    Returns:
        dict: The JSON schema of the model.
    """
    result: dict[int, dict[str, Any]] = {}
    example: dict[str, Any] = model.model_json_schema().get("properties", {})

    for status_code, description in responses.items():
        if status_code in [204, 304]:
            result[status_code] = {"description": description}
            continue

        result[status_code] = {
            "description": description,
            "model": model,
            "content": {"application/json": {"example": example}},
        }
    return result

```

#### Usage in _routers_
```python title:router.py
from utils import get_model_schema_for_docs

from fastapi import APIRouter, Request, status
from fastapi.responses import JSONResponse


router: APIRouter = APIRouter()


@router.get(
    path="/todos",
    summary="Gets all ToDos",
    response_model=dict[str, Any],
    status_code=status.HTTP_200_OK,
    responses=get_model_schema_for_docs(
        model=BaseResponse, # A pydantic BaseModel
        responses={
            200: "ToDos retrieved successfully",
            422: "Unprocessable Entity: Validation Error",
            500: "Internal Server Error",
        },
    ),
)
async def get_todos(request: Request) -> JSONResponse:
	pass

```


### Error Handling

#### Custom Error Classes
```python title:v1/core/domain/exceptions.py
class CustomError(Exception):
    def __init__(self, msg: str) -> None:
        self.msg = msg

class BadRequestError(CustomError): ...  # 400

class UnauthorizedError(CustomError): ...  # 401

class ForbiddenError(CustomError): ...  # 403

class NotFoundError(CustomError): ...  # 404

class ConflictError(CustomError): ...  # 409

class UnprocessableEntityError(CustomError): ...  # 422

class TooManyRequestsError(CustomError): ...  # 429

class BadGatewayError(CustomError): ...  # 502

```

#### Error Handler Functions
```python title:v1/core/domain/error_handlers.py
from logging import Logger, getLogger

from cryptography.fernet import InvalidToken
from fastapi import HTTPException, Request, status
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from jose.exceptions import JWTError
from pydantic import ValidationError
from pydantic_core import ValidationError as CoreValidationError
from pymongo.errors import PyMongoError

from v1.core.config import LOGGING_PROJECT_NAME
from v1.core.domain.exceptions import (
    BadGatewayError,
    BadRequestError,
    ConflictError,
    ForbiddenError,
    NotFoundError,
    TooManyRequestsError,
    UnauthorizedError,
    UnprocessableEntityError,
)
from v1.core.presentation.service import (
    make_error_response,
    make_response,
)

logger: Logger = getLogger(f"{LOGGING_PROJECT_NAME}.core.error_handlers")


#######################################################################################################################
################################# Helpers #############################################################################
#######################################################################################################################
def _sanitize_input(value: object) -> object:
    """Convert non-JSON-serializable inputs (e.g. bytes) to their string representation so that the validation error
    response can be serialized."""
    if isinstance(value, (str, int, float, bool, type(None))):
        return value
    if isinstance(value, (list, tuple)):
        return [_sanitize_input(item) for item in value]
    if isinstance(value, dict):
        return {k: _sanitize_input(v) for k, v in value.items()}
    return repr(value)


#######################################################################################################################
################################# Exceptions ##########################################################################
#######################################################################################################################
async def general_exception_handler(request: Request, exception: Exception) -> JSONResponse:
    return make_error_response(logger=logger, type=type(exception).__name__, message=str(exception))


#######################################################################################################################
################################# HTTP ################################################################################
#######################################################################################################################
async def http_exception_handler(request: Request, exception: HTTPException) -> JSONResponse:
    return JSONResponse(
        status_code=exception.status_code,
        content={
            "success": False,
            "message": _sanitize_input(exception.detail),
        },
    )


#######################################################################################################################
################################# Pydantic ############################################################################
#######################################################################################################################
async def validation_exception_handler(
    request: Request,
    exception: CoreValidationError | ValidationError | RequestValidationError,
) -> JSONResponse:
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "success": False,
            "message": "Missing or invalid data",
            "details": list(
                map(
                    lambda error: {
                        "field": ", ".join(
                            f"type: {loc}" if i == 0 else f"field: {loc}"
                            for i, loc in enumerate(error["loc"])
                        ),
                        "message": error["msg"],
                        "error": error["type"],
                        "input": _sanitize_input(error.get("input", None)),
                    },
                    exception.errors(),
                )
            ),
        },
    )


#######################################################################################################################
################################# JWT #################################################################################
#######################################################################################################################
async def invalid_jwt_error_handler(request: Request, exception: JWTError) -> JSONResponse:
    """Handles jose JWTError: expired, malformed or unverifiable JWT tokens."""
    error_message = getattr(exception, "msg", str(exception)) or "Invalid JWT token"
    logger.warning(f"[JWT] {type(exception).__name__}: {error_message}")
    return make_response(logger=logger, status=status.HTTP_401_UNAUTHORIZED, message="Invalid token", success=False)


async def invalid_fernet_token_handler(request: Request, exception: InvalidToken
) -> JSONResponse:
    """Handles cryptography.fernet.InvalidToken: wrong key or corrupted ciphertext."""
    logger.error("[Fernet] InvalidToken:", exc_info=exception)
    return make_response(
        logger=logger,
        status=status.HTTP_401_UNAUTHORIZED,
        message="Invalid token credentials",
        success=False,
    )


#######################################################################################################################
################################# PyMongo #############################################################################
#######################################################################################################################
async def database_error_handler(request: Request, exception: PyMongoError) -> JSONResponse:
    """Handles PyMongo/Motor/Beanie infrastructure errors (connection, query, TLS, etc.)."""
    logger.error(f"[MongoDB] {type(exception).__name__}: {exception}", exc_info=exception)
    return make_response(logger=logger, status=status.HTTP_502_BAD_GATEWAY, message="Database error", success=False)


#######################################################################################################################
################################# Domain ##############################################################################
#######################################################################################################################
async def bad_request_error_handler(request: Request, exception: BadRequestError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_400_BAD_REQUEST,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )


async def unauthorized_error_handler(request: Request, exception: UnauthorizedError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_401_UNAUTHORIZED,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )


async def forbidden_error_handler(request: Request, exception: ForbiddenError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_403_FORBIDDEN,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )


async def not_found_error_handler(request: Request, exception: NotFoundError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_404_NOT_FOUND,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )


async def conflict_error_handler(request: Request, exception: ConflictError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_409_CONFLICT,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )


async def unprocessable_entity_error_handler(request: Request, exception: UnprocessableEntityError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_422_UNPROCESSABLE_ENTITY,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )


async def too_many_requests_error_handler(request: Request, exception: TooManyRequestsError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_429_TOO_MANY_REQUESTS,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )


async def bad_gateway_error_handler(request: Request, exception: BadGatewayError) -> JSONResponse:
    return make_response(
        logger=logger,
        status=status.HTTP_502_BAD_GATEWAY,
        message=getattr(exception, "msg", str(exception)),
        success=False,
    )

```

#### Integration in _main.py_
```python title:main.py
from cryptography.fernet import InvalidToken
from fastapi import FastAPI
from fastapi.exceptions import RequestValidationError
from jose.exceptions import JWTError
from pydantic import ValidationError
from pydantic_core import ValidationError as CoreValidationError
from pymongo.errors import PyMongoError

from v2.core.domain.error_handlers import (
    bad_gateway_error_handler,
    bad_request_error_handler,
    conflict_error_handler,
    forbidden_error_handler,
    general_exception_handler,
    http_exception_handler,
    invalid_jwt_error_handler,
    not_found_error_handler,
    too_many_requests_error_handler,
    unauthorized_error_handler,
    unprocessable_entity_error_handler,
    validation_exception_handler,
)
from v2.core.domain.exceptions import (
    BadGatewayError,
    BadRequestError,
    ConflictError,
    ForbiddenError,
    NotFoundError,
    TooManyRequestsError,
    UnauthorizedError,
    UnprocessableEntityError,
)


app: FastAPI = FastAPI()

###############################################################################
############################### Error Handlers ################################
###############################################################################
# Exception
app.add_exception_handler(Exception, general_exception_handler)
# HTTP
app.add_exception_handler(HTTPException, http_exception_handler)
# Pydantic
app.add_exception_handler(ValidationError, validation_exception_handler)
app.add_exception_handler(RequestValidationError, validation_exception_handler)
app.add_exception_handler(CoreValidationError, validation_exception_handler)
# JWT / Crypto
app.add_exception_handler(JWTError, invalid_jwt_error_handler)
app.add_exception_handler(InvalidToken, invalid_fernet_token_handler)
# Database (Beanie / Motor / PyMongo)
app.add_exception_handler(PyMongoError, database_error_handler)
# Domain
app.add_exception_handler(BadRequestError, bad_request_error_handler)
app.add_exception_handler(UnauthorizedError, unauthorized_error_handler)
app.add_exception_handler(ForbiddenError, forbidden_error_handler)
app.add_exception_handler(NotFoundError, not_found_error_handler)
app.add_exception_handler(ConflictError, conflict_error_handler)
app.add_exception_handler(
    UnprocessableEntityError, unprocessable_entity_error_handler
)
app.add_exception_handler(
    TooManyRequestsError, too_many_requests_error_handler
)
app.add_exception_handler(BadGatewayError, bad_gateway_error_handler)

```

___

## Utils

### _BaseResponse_ `Pydantic` Schemas

```python title:v1/core/presentation/schemas.py
from typing import Any
from uuid import uuid4

from pydantic import BaseModel, Field


class LoggerResponse(BaseModel):
    service: str = Field(
        min_length=1,
        max_length=64,
        description="Service name that is writing to Loki.",
    )
    execution_id: str = Field(
        default_factory=lambda: str(uuid4()),
        description="Execution ID for the request.",
    )


class ExecutionError(BaseModel):
    step: str = Field(
        min_length=1,
        max_length=64,
        description="The step in the process where the error occurred.",
    )
    type: str = Field(
        min_length=1,
        max_length=64,
        description="The type of error that occurred.",
    )
    message: str = Field(
        min_length=1,
        max_length=256,
        description="A description of the error that occurred.",
    )


class BaseResponse(BaseModel):
    success: bool = Field(
        description="Indicates if the request was successful."
    )
    message: str = Field(
        min_length=1,
        max_length=256,
        description="""
		A message providing additional information about the request.
		""",
    )
    data: dict[str, Any] | None = Field(
        default=None,
        description="""
		Additional data returned by the API, if any.
		""",
    )
    error: ExecutionError | None = Field(
        default=None,
        description="""
		A dict of an error encountered during the process, represented with
		"step" and "error" keys. False if no error occurred.
		""",
    )
    logger: LoggerResponse = Field(
        default_factory=LoggerResponse,
        description="Logger information.",
    )

```

```python title:v1/core/presentation/service.py
from base64 import b64encode
from json import dumps
from logging import Logger, getLogger
from typing import Any

from dotenv import load_dotenv
from fastapi import status
from fastapi.responses import JSONResponse, RedirectResponse
from pydantic import BaseModel

from v1.core.config import DOTENV_ABSPATH, LOGGING_PROJECT_NAME
from v1.core.presentation.schemas import (
    BaseResponse,
    ExecutionError,
    LokiLoggerResponse,
)

load_dotenv(DOTENV_ABSPATH)
logger: Logger = getLogger(f"{LOGGING_PROJECT_NAME}.core.service")


#######################################################################################################################
################################# Docs ################################################################################
#######################################################################################################################
def get_model_schema_for_docs(model: BaseModel, responses: dict[int, str]) -> dict[int, dict[str, Any]]:
    """
    Utility function to get the JSON schema of a Pydantic response model for documentation purposes.

    Args:
        model: The Pydantic model class.

    Returns:
        dict: The JSON schema of the model.
    """

    result: dict[int, dict[str, Any]] = {}
    example: dict[str, Any] = model.model_json_schema().get("properties", {})

    for status_code, description in responses.items():
        if status_code in [204, 304]:
            result[status_code] = {"description": description}
            continue

        result[status_code] = {
            "description": description,
            "model": model,
            "content": {"application/json": {"example": example}},
        }
    return result


#######################################################################################################################
################################# Responses ###########################################################################
#######################################################################################################################
def make_response(
    logger: Logger,
    status: str,
    message: str,
    success: bool = True,
    data: dict[str, Any] | None = None,
    error: dict[str, Any] | None = None,
    show_logger: bool = True,
) -> JSONResponse:
    response: BaseResponse = BaseResponse(
        success=success,
        message=message,
        data=data,
        error=None if not error else ExecutionError(**error),
        logger=LokiLoggerResponse(service=LOGGING_PROJECT_NAME),
    )
    response_message: str = f"Returning response: [{status}] {response.model_dump_json(indent=2)}"
    if show_logger:
        logger.info(response_message)
    return JSONResponse(status_code=status, content=response.model_dump())


def make_error_response(
    logger: Logger,
    type: str,
    message: str,
    endpoint: str | None = None,
    method: str | None = None,
    status: int = status.HTTP_500_INTERNAL_SERVER_ERROR,
) -> JSONResponse:

    if endpoint and method:
        logger.error(f"{type} during {method} {endpoint}: {message}")
    else:
        logger.error(f"{type}: {message}")

    response: BaseResponse = BaseResponse(
        success=False,
        message="An error occurred while processing the request",
        error=(
            None
            if not type
            else ExecutionError(
                type=type,
                message=message if message else "An unexpected error occurred",
            )
        ),
        logger=LokiLoggerResponse(service=LOGGING_PROJECT_NAME),
    )
    logger.error(f"Returning exception response: [{status}] {response.model_dump_json(indent=2)}")
    return JSONResponse(status_code=status, content=response.model_dump())


def make_redirection_response(logger: Logger, data: dict[str, Any], base_url: str) -> RedirectResponse:
    data: str = b64encode(dumps(data).encode("utf-8")).decode("utf-8")
    redirect_url: str = f"{base_url}?data={data}"
    logger.info(f"Redirecting to URL: {redirect_url}")
    return RedirectResponse(url=redirect_url, status_code=status.HTTP_302_FOUND)

```


### Send emails

```bash title:.env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
EMAIL="example@gmail.com"
PASSWORD="hsjd tudh ahfk etfi"
```

```python title:emails.py
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from logging import Logger, getLogger
from os import getenv
from smtplib import SMTP
from typing import Any

from dotenv import load_dotenv

from v1.config import DOTENV_ABSPATH, LOGGING_PROJECT_NAME

load_dotenv(DOTENV_ABSPATH)
logger: Logger = getLogger(f"{LOGGING_PROJECT_NAME}.{__name__.split('.')[-1]}")


class EmailClient:
    def __init__(self) -> None:
        self.server = SMTP(
            str(getenv("SMTP_HOST")),
            int(getenv("SMTP_PORT")),
        )

    def _connect(self) -> None:
	    self.server.starttls()
        self.server.login(
            user=str(getenv("EMAIL")), password=str(getenv("PASSWORD"))
        )
        logger.info("SMTP server connected successfully")

    def _disconnect(self) -> None:
        self.server.quit()
        logger.info("SMTP server disconnected")

    def _load_html_template(self, path: str) -> str | None:
        try:
            with open(path, encoding="utf-8") as file:
                return file.read()
        except Exception as e:
            logger.exception(
                f"[{type(e).__name__}] loading template: {path} - {str(e)}"
            )
            return None

    def send_email(
        self,
        email_to: str,
        template_path: str,
        subject: str,
        data: dict[str, Any] | None = None,
        cc: list[str] | None = None,
    ) -> None:

	    self._connect()

        msg = MIMEMultipart("alternative")
        msg["From"] = str(getenv("EMAIL"))
        msg["Subject"] = subject
        msg["To"] = email_to
        if cc:
            msg["Cc"] = ", ".join(cc)
            email_to: list[str] = [email_to] + cc

        email_template = self._load_html_template(template_path)
        if not email_template:
            logger.error("Email template could not be loaded")
            raise ValueError("Email template could not be loaded")

        if data:
            for key, value in data.items():
                email_template: str = email_template.replace(
                    f"{{{key}}}", str(value)
                )

        html_part = MIMEText(email_template, "html", "utf-8")

        plain_text = """
        Este es un correo en formato HTML.
        Si no puedes ver el contenido correctamente,
        tu cliente de correo no soporta HTML.
        """
        text_part = MIMEText(plain_text, "plain", "utf-8")

        msg.attach(text_part)
        msg.attach(html_part)

        text_message: str = msg.as_string()
        self.server.sendmail(
            from_addr=str(getenv("EMAIL")),
            to_addrs=email_to,
            msg=text_message,
        )

        self._disconnect()

        logger.info("Email sent successfully")

```


### Security class

```python title:security.py
from asyncio import get_event_loop
from hashlib import sha256
from json import dumps, loads
from os import getenv
from typing import Any

from cryptography.fernet import Fernet
from dotenv import load_dotenv
from passlib.context import CryptContext

from v1.config import DOTENV_ABSPATH

load_dotenv(DOTENV_ABSPATH)


class Security:
    def __init__(self) -> None:
        self.context = CryptContext(schemes=["bcrypt"], deprecated="auto")

    def to_dict(self, string: str) -> dict[str, Any]:
        return loads(string)

    def to_str(self, data: dict[str, Any]) -> str:
        return dumps(data)

    async def encrypt(self, string: str, key_name: str) -> str:
        def _encrypt() -> str:
            f = Fernet(getenv(key_name))
            pwd: bytes = f.encrypt(string.encode("utf-8"))
            return pwd.decode()

        return await get_event_loop().run_in_executor(None, _encrypt)

    async def decrypt(self, string: str, key_name: str) -> str:
        def _decrypt() -> str:
            f = Fernet(getenv(key_name))
            pwd: bytes = f.decrypt(string.encode("utf-8"))
            return pwd.decode()

        return await get_event_loop().run_in_executor(None, _decrypt)

    async def simple_hash(self, string: str, key_name: str) -> str:
        def _simple_hash() -> str:
            value = f"{string}${getenv(key_name)}"
            return sha256(value.encode("utf-8")).hexdigest()

        return await get_event_loop().run_in_executor(None, _simple_hash)

    async def hash(self, string: str) -> str:
        def _hash() -> str:
            return self.context.hash(string)

        return await get_event_loop().run_in_executor(None, _hash)

    async def verify(self, string: str, hash: str) -> bool:
        def _verify() -> bool:
            return self.context.verify(string, hash)

        return await get_event_loop().run_in_executor(None, _verify)

```


### `MongoDB` integration

#### Connection Keys
```txt title:.env
#######################################################################################################################
############################ Infrastructure ###########################################################################
#######################################################################################################################
# Mongo connection
MONGO_HOST=localhost
MONGO_PORT=27017
MONGO_USER=root
MONGO_PASSWORD="my_password"
MONGO_DB="mongo_db"
MONGO_CERTIFICATE_PEM_FILE="certs/data/mongodb.pem"
MONGO_CERTIFICATE_CRT_FILE="certs/data/mongodb.crt"
```

#### `Beanie` Models
```python title:v1/core/infrastructure/models.py
from datetime import UTC, datetime

from beanie import Document, PydanticObjectId
from bson import ObjectId
from pydantic import BaseModel, EmailStr, Field
from pymongo import ASCENDING, IndexModel


class UserDetails(BaseModel):
    email: EmailStr
    mobile: str | None = None


class User(Document):
    id: PydanticObjectId | None = Field(default_factory=ObjectId, alias="_id")
    username: str
    password: str
    roles: list[str]
    user_details: DataUserDetails
    created_at: datetime = Field(default_factory=lambda: datetime.now(UTC))

    class Settings:
        name: str = "users"  # Name of the collection in MongoDB
        
        indexes = [
            IndexModel([("expires_at", ASCENDING)], expireAfterSeconds=0),
            IndexModel([("username", ASCENDING)], unique=True),
        ]

    class Config:
        arbitrary_types_allowed = True

```

#### Connection
```python title:v1/core/infrastructure/mongo_conn.py
from os import getenv
from os.path import join
from urllib.parse import quote_plus

from beanie import init_beanie
from dotenv import load_dotenv
from motor.motor_asyncio import AsyncIOMotorClient

from v1.apikeys.infrastructure.models import ApiKey
from v1.core.application.config import (DOTENV_ABSPATH, V1_ABSPATH)
from v1.core.infrastructure.models import User

load_dotenv(DOTENV_ABSPATH)


class Mongo:
    def __init__(self) -> None:
        self._config = {
            # Connection settings
            "host": getenv("MONGO_HOST"),
            "port": getenv("MONGO_PORT"),
            "user": quote_plus(getenv("MONGO_USER")),
            "password": quote_plus(getenv("MONGO_PASSWORD")),
            # Databases
            "db": getenv("MONGO_DB"),
        }
        self._certificate_pem_path = join(V1_ABSPATH, getenv("MONGO_PRODUCT_CERTIFICATE_PEM_FILE"))
        self._certificate_crt_path = join(V1_ABSPATH, getenv("MONGO_PRODUCT_CERTIFICATE_CRT_FILE"))
        self._db_uri = self.format_uri(self._config["security_db"])

    def format_uri(self, db: str) -> str:
        return (
            f"mongodb://{self._config['user']}:{self._config['password']}@"
            f"{self._config['host']}:{self._config['port']}/"
            f"{db}?"
            #! Danger: Allow invalid certificates to tests only
            f"authSource=admin&tls=true" # &tlsAllowInvalidCertificates=true
        )

    async def init_db(self) -> None:
        client = AsyncIOMotorClient(
            self._security_db_uri,
            tls=True,
            tlsCertificateKeyFile=self._certificate_pem_path,
            tlsCAFile=self._certificate_crt_path,
        )
        db = client.get_default_database()
        await init_beanie(
            database=db,
            document_models=[User],
        )

```

#### Initializing in API Execution
```python title:main.py
from v1.core.infrastructure.mongo_conn import Mongo

@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None, None]:
    # ON STARTUP
    ...
    app.mongodb = Mongo()

    await app.mongodb.init_db()

    try:
        yield
    except Exception as exc:
        logger.error(exc)

    finally:
        # ON SHUTDOWN
        ...

```

#### `Repository` for `DDD` Queries
```python title:repository.py
from __future__ import annotations

from abc import ABC, abstractmethod
from datetime import datetime
from logging import Logger, getLogger

from beanie import PydanticObjectId

from v1.config import LOGGING_PROJECT_NAME
from v1.core.infrastructure.models import User
from v1.core.infrastructure.mongo_conn import Mongo

logger: Logger = getLogger(f"{LOGGING_PROJECT_NAME}.user.repository")


class UserRepositoryInterface(ABC):
    """Abstract repository for User entity persistence."""

    @abstractmethod
    async def get_user_by_username(self, username: str) -> User | None: ...

    @abstractmethod
    async def get_user_by_email(self, email: str) -> User | None: ...

    @abstractmethod
    async def get_user_by_id(self, user_id: str) -> User | None: ...

    @abstractmethod
    async def create(self, token: RefreshToken) -> RefreshToken: ...

    @abstractmethod
    async def delete(self, token: RefreshToken) -> None: ...


class UserRepository(UserRepositoryInterface):
    def __init__(self, session: Mongo, user_model: type[User]) -> None:
        self.session = session
        self.user_model = user_model

    async def get_user_by_username(self, username: str) -> User | None:
        logger.info("Getting user by username")
        if username.startswith("@"):
            user = await self.user_model.find_one(self.user_model.user == username)
        else:
            user = await self.user_model.find_one(self.user_model.user_details.email == username)
        return user

    async def get_user_by_email(self, email: str) -> User | None:
        logger.info("Getting user by email")
        user = await self.user_model.find_one(self.user_model.email == email)
        return user

    async def get_user_by_id(self, user_id: str) -> User | None:
        logger.info("Getting user by id")
        return await self.user_model.get(PydanticObjectId(user_id))

    async def create(self, name: str, email: str, password: str, expires_at: datetime) -> User:
        logger.info("Creating refresh token in MongoDB")
        user = User(
	        name=name,
            email=email,
            password=password,
            expires_at=expires_at
        )
        await user.insert()
        logger.info(f"User created: {user.id} in MongoDB")
        return user

    async def delete(self, user: User) -> None:
        logger.info("Deleting user in MongoDB")
        await user.delete()

```

___

## Claude Sessions