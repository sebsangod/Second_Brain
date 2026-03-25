---
aliases:
  - Beanie
tags:
  - learning
---
**Date**: 15/mar/2026
**Update**: 15/mar/2026
**Sources**: [Beanie]([https://beanie-odm.dev/](https://github.com/BeanieODM/beanie))

**Tags:** #development #data #datavalidation

**Related:** [[Python]], [[ODM]], [[MongoDB]], [[Pydantic]], [[Database]], [[Collection]], [[Bunnet]], [[PyTest]]

---

## Description

_Beanie_ - is an asynchronous `Python` object-document mapper (`ODM`) for `MongoDB`. Data models are based on `Pydantic`.

When using _Beanie_ each `database` `collection` has a corresponding _Document_ that is used to interact with that `collection`. **In addition to retrieving data,** _Beanie_ **allows you to add, update, or delete documents from the collection as well.**

Installing _Beanie_ is as simple as: _pip install beanie_

---

## Key concepts

* _Beanie_ saves you time by removing boilerplate code, and it helps you focus on the parts of your app that actually matter.
* Data and schema migrations are supported by _Beanie_ out of the box.
* There is a synchronous version of Beanie ODM: `Bunnet`

---

## Examples

```python title:beanie_connection.py
import asyncio
from typing import Optional

from beanie import Document, Indexed, init_beanie

from pymongo import AsyncMongoClient
from pydantic import BaseModel


class Category(BaseModel):
    name: str
    description: str

class Product(Document):
    name: str # You can use normal types just like in pydantic
    description: Optional[str] = None
    price: Indexed(float) # You can also specify that a field should correspond to an index
    category: Category # You can include pydantic models as well

# This is an asynchronous example, so we will access it from an async function
async def example():
    # Beanie uses PyMongo async client under the hood
    client = AsyncMongoClient("mongodb://user:pass@host:27017")

    # Initialize beanie with the Product document class
    await init_beanie(database=client.db_name, document_models=[Product])

    chocolate = Category(name="Chocolate", description="A preparation of roasted and ground cacao seeds.")
    # Beanie documents work just like pydantic models
    tonybar = Product(name="Tony's", price=5.95, category=chocolate)
    # And can be inserted into the database
    await tonybar.insert()

    # You can find documents with pythonic syntax
    product = await Product.find_one(Product.price < 10)
    # And update them
    await product.set({Product.name:"Gold bar"})


if __name__ == "__main__":
    asyncio.run(example())

```

---

## Utils

### Find by Id

```python title:repository.py
from beanie import PydanticObjectId

from v1.models import User


async def get_user_by_id(self, user_id: str) -> User | None:
    return await User.get(PydanticObjectId(user_id))

```


### Define models

```python title:repository.py
from datetime import datetime

from beanie import Document
from pymongo import ASCENDING, IndexModel


class Token(Document):
    user_id: str
    token: str
    platform: str
    expires_at: datetime

    class Settings:
        name = "tokens"

        indexes = [
            IndexModel([("expires_at", ASCENDING)], expireAfterSeconds=0),
            IndexModel([("token", ASCENDING)], unique=True),
        ]

```


### Tests alongside `PyTest`

```python title:conftest.py
"""Root conftest — mock MongoDB initialization for all tests."""

from __future__ import annotations

from unittest.mock import AsyncMock, patch

import pytest


@pytest.fixture(autouse=True)
def mock_mongodb_init():
    """Prevent real MongoDB connections during tests."""
    with (
        patch(
            "backend.v2.core.infrastructure.vde_mongo"
            ".VDEMongo.init_security_db",
            new_callable=AsyncMock,
        ),
        patch(
            "backend.v2.core.infrastructure.vde_mongo.VDEMongo.init_appdex_db",
            new_callable=AsyncMock,
        ),
        patch(
            "backend.v2.core.infrastructure.vde_mongo.VDEMongo.init_efiro_db",
            new_callable=AsyncMock,
        ),
    ):
        yield


# Tests execution command
# env/bin/python -m pytest backend/v2/jwt/tests/test_main.py -v --tb=short

```

```python title:repository.py
"""
JWT API Unit Tests
==================
Tests for JWT CRUD operations and DEX-CHANNEL header validations
with platform-specific database separation.
"""

from __future__ import annotations

import json
from datetime import UTC, datetime, timedelta
from unittest.mock import AsyncMock, MagicMock, patch

import pytest
from fastapi.testclient import TestClient

from backend.main import app
from backend.v2.core.config import PROJECT_V2_PATHS_PREFIX

BASE_URL = f"/{PROJECT_V2_PATHS_PREFIX}/auth"
VALID_PLATFORMS = ["Efiro", "DexmiManager", "DexmiPass"]
DEXMI_PLATFORMS = ["DexmiManager", "DexmiPass"]

# Patch targets (shortened for readability)
_SEC = "backend.v2.core.application.security.SecurityClient"
_REPO = "backend.v2.jwt.infrastructure.repository.RefreshTokenRepository"
_UC = "backend.v2.jwt.application.use_cases"


# ── Fixtures ────────────────────────────────────────────────────────
@pytest.fixture()
def client():
    """Create a TestClient with mocked lifespan."""
    with TestClient(app) as c:
        yield c


# ── Helpers ─────────────────────────────────────────────────────────
def _mock_user(platform: str = "Efiro") -> MagicMock:
    """Build a mock User matching the given platform schema."""
    user = MagicMock()
    user.id = "507f1f77bcf86cd799439011"
    user.password = "$2b$12$hashed"
    if platform.lower() == "efiro":
        user.email = "test@example.com"
    else:
        user.user = "@testuser"
        user.user_details = MagicMock()
        user.user_details.email = "test@example.com"
    return user


def _mock_refresh_token(
    platform: str = "Efiro",
    expired: bool = False,
) -> MagicMock:
    """Build a mock RefreshToken document."""
    tok = MagicMock()
    tok.user_id = "507f1f77bcf86cd799439011"
    tok.token = "mock.jwt.token"
    tok.platform = platform
    if expired:
        tok.expires_at = datetime(2020, 1, 1, tzinfo=UTC)
    else:
        tok.expires_at = datetime.now(UTC) + timedelta(days=5)
    return tok


async def _login_decrypt(string: str, key_name: str) -> str:
    """Side-effect for SecurityClient.decrypt during login flow."""
    if key_name == "FRONTEND2SECURITY_BODY_FERNET_KEY":
        return json.dumps({"email": "test@example.com", "password": "enc_pwd"})
    if key_name == "FRONTEND2SECURITY_PWD_FERNET_KEY":
        return "plain_password"
    return string


# ####################################################################
# 1. DEX-CHANNEL Header Validation
# ####################################################################
class TestDexChannelValidation:
    """All endpoints must reject missing / invalid DEX-CHANNEL.

    The check is case-sensitive: "Efiro", "DexmiManager", "DexmiPass"
    are the only accepted values.
    """

    # ── Login ───────────────────────────────────────────────────────
    def test_login_missing_header_returns_422(
        self, client: TestClient
    ) -> None:
        resp = client.post(f"{BASE_URL}/login", json={"data": "x"})
        assert resp.status_code == 422

    @pytest.mark.parametrize(
        "platform",
        [
            "InvalidPlatform",
            "efiro",           # wrong case
            "EFIRO",           # wrong case
            "dexmimanager",    # wrong case
            "DexMiManager",    # wrong case
            "dexmipass",       # wrong case
        ],
    )
    def test_login_invalid_header_returns_422(
        self, client: TestClient, platform: str
    ) -> None:
        resp = client.post(
            f"{BASE_URL}/login",
            json={"data": "x"},
            headers={"DEX-CHANNEL": platform},
        )
        assert resp.status_code == 422
        assert resp.json()["success"] is False

    # ── Refresh ─────────────────────────────────────────────────────
    def test_refresh_missing_header_returns_422(
        self, client: TestClient
    ) -> None:
        resp = client.post(
            f"{BASE_URL}/refresh",
            headers={"Authorization": "Bearer tok"},
        )
        assert resp.status_code == 422

    @pytest.mark.parametrize(
        "platform",
        [
            "InvalidPlatform",
            "efiro",
            "EFIRO",
            "dexmimanager",
            "DexMiManager",
            "dexmipass",
        ],
    )
    def test_refresh_invalid_header_returns_422(
        self, client: TestClient, platform: str
    ) -> None:
        resp = client.post(
            f"{BASE_URL}/refresh",
            headers={
                "DEX-CHANNEL": platform,
                "Authorization": "Bearer tok",
            },
        )
        assert resp.status_code == 422
        assert resp.json()["success"] is False

    # ── Logout ──────────────────────────────────────────────────────
    def test_logout_missing_header_returns_422(
        self, client: TestClient
    ) -> None:
        resp = client.delete(
            f"{BASE_URL}/logout",
            headers={"Authorization": "Bearer tok"},
        )
        assert resp.status_code == 422

    @pytest.mark.parametrize(
        "platform",
        [
            "InvalidPlatform",
            "efiro",
            "EFIRO",
            "dexmimanager",
            "DexMiManager",
            "dexmipass",
        ],
    )
    def test_logout_invalid_header_returns_422(
        self, client: TestClient, platform: str
    ) -> None:
        resp = client.delete(
            f"{BASE_URL}/logout",
            headers={
                "DEX-CHANNEL": platform,
                "Authorization": "Bearer tok",
            },
        )
        assert resp.status_code == 422
        assert resp.json()["success"] is False

    # ── Valid platforms accepted ─────────────────────────────────────
    @pytest.mark.parametrize("platform", VALID_PLATFORMS)
    def test_login_accepts_valid_platforms(
        self, client: TestClient, platform: str
    ) -> None:
        """Valid DEX-CHANNEL passes the header check (201)."""
        with (
            patch(
                f"{_UC}.LoginJWTUseCase.execute",
                new_callable=AsyncMock,
                return_value={
                    "access_token": "at",
                    "refresh_token": "rt",
                },
            ),
            patch(
                f"{_SEC}.decrypt",
                new_callable=AsyncMock,
                return_value=('{\"email\": \"t@t.com\", \"password\": \"p\"}'),
            ),
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "encrypted"},
                headers={"DEX-CHANNEL": platform},
            )
            assert resp.status_code == 201

    @pytest.mark.parametrize("platform", VALID_PLATFORMS)
    def test_refresh_accepts_valid_platforms(
        self, client: TestClient, platform: str
    ) -> None:
        with patch(
            f"{_UC}.RefreshJWTUseCase.execute",
            new_callable=AsyncMock,
            return_value={
                "access_token": "at",
                "refresh_token": "rt",
            },
        ):
            resp = client.post(
                f"{BASE_URL}/refresh",
                headers={
                    "DEX-CHANNEL": platform,
                    "Authorization": "Bearer tok",
                },
            )
            assert resp.status_code == 201

    @pytest.mark.parametrize("platform", VALID_PLATFORMS)
    def test_logout_accepts_valid_platforms(
        self, client: TestClient, platform: str
    ) -> None:
        with patch(
            f"{_UC}.LogoutJWTUseCase.execute",
            new_callable=AsyncMock,
        ):
            resp = client.delete(
                f"{BASE_URL}/logout",
                headers={
                    "DEX-CHANNEL": platform,
                    "Authorization": "Bearer tok",
                },
            )
            assert resp.status_code == 200


# ####################################################################
# 2. Login — Request Body Schema Validation
# ####################################################################
class TestLoginBodyValidation:
    """Verify FastAPI/Pydantic schema checks before business logic runs."""

    def test_missing_data_field_returns_422(
        self, client: TestClient
    ) -> None:
        """Omitting the required `data` field fails at schema level."""
        resp = client.post(
            f"{BASE_URL}/login",
            json={},
            headers={"DEX-CHANNEL": "Efiro"},
        )
        assert resp.status_code == 422
        assert resp.json()["success"] is False

    def test_empty_data_field_returns_422(
        self, client: TestClient
    ) -> None:
        """`data=""` violates min_length=1 on RequestJWT."""
        resp = client.post(
            f"{BASE_URL}/login",
            json={"data": ""},
            headers={"DEX-CHANNEL": "Efiro"},
        )
        assert resp.status_code == 422
        assert resp.json()["success"] is False

    @pytest.mark.parametrize("platform", DEXMI_PLATFORMS)
    def test_dexmi_invalid_email_format_returns_422(
        self, client: TestClient, platform: str
    ) -> None:
        """For Dexmi platforms the DTO uses EmailStr, so a non-email string
        triggers Pydantic validation error → 422."""

        async def _decrypt_bad_email(string: str, key_name: str) -> str:
            if key_name == "FRONTEND2SECURITY_BODY_FERNET_KEY":
                return json.dumps(
                    {"email": "not-a-valid-email", "password": "pwd"}
                )
            return string

        with patch(
            f"{_SEC}.decrypt",
            new_callable=AsyncMock,
            side_effect=_decrypt_bad_email,
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": platform},
            )
        assert resp.status_code == 422
        assert resp.json()["success"] is False

    @pytest.mark.parametrize("platform", DEXMI_PLATFORMS)
    def test_dexmi_empty_email_returns_422(
        self, client: TestClient, platform: str
    ) -> None:
        """Empty string also fails EmailStr validation on Dexmi platforms."""

        async def _decrypt_empty_email(string: str, key_name: str) -> str:
            if key_name == "FRONTEND2SECURITY_BODY_FERNET_KEY":
                return json.dumps({"email": "", "password": "pwd"})
            return string

        with patch(
            f"{_SEC}.decrypt",
            new_callable=AsyncMock,
            side_effect=_decrypt_empty_email,
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": platform},
            )
        assert resp.status_code == 422
        assert resp.json()["success"] is False


# ####################################################################
# 3. Login — Platform-specific Search
# ####################################################################
class TestLoginPlatformSearch:
    """Verify correct repository method is called per platform."""

    def _run_login(
        self,
        client: TestClient,
        platform: str,
    ) -> tuple:
        """Execute a login and return (response, email_mock, user_mock)."""
        mock_user = _mock_user(platform)
        mock_email = AsyncMock(return_value=mock_user)
        mock_username = AsyncMock(return_value=mock_user)

        with (
            patch(
                f"{_SEC}.decrypt",
                new_callable=AsyncMock,
                side_effect=_login_decrypt,
            ),
            patch(
                f"{_SEC}.encrypt",
                new_callable=AsyncMock,
                return_value="enc",
            ),
            patch(
                f"{_SEC}.verify",
                new_callable=AsyncMock,
                return_value=True,
            ),
            patch(
                f"{_REPO}.get_user_by_email",
                mock_email,
            ),
            patch(
                f"{_REPO}.get_user_by_username",
                mock_username,
            ),
            patch(
                f"{_REPO}.create",
                new_callable=AsyncMock,
            ),
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "encrypted_body"},
                headers={"DEX-CHANNEL": platform},
            )
        return resp, mock_email, mock_username

    def test_efiro_uses_get_user_by_email(self, client: TestClient) -> None:
        resp, mock_email, mock_username = self._run_login(client, "Efiro")
        assert resp.status_code == 201
        data = resp.json()["data"]
        assert "access_token" in data
        assert "refresh_token" in data
        mock_email.assert_called_once()
        mock_username.assert_not_called()

    def test_dexmi_manager_uses_get_user_by_username(
        self, client: TestClient
    ) -> None:
        resp, mock_email, mock_username = self._run_login(
            client, "DexmiManager"
        )
        assert resp.status_code == 201
        mock_username.assert_called_once()
        mock_email.assert_not_called()

    def test_dexmi_pass_uses_get_user_by_username(
        self, client: TestClient
    ) -> None:
        resp, mock_email, mock_username = self._run_login(client, "DexmiPass")
        assert resp.status_code == 201
        mock_username.assert_called_once()
        mock_email.assert_not_called()


# ####################################################################
# 4. Login — Error Cases
# ####################################################################
class TestLoginErrors:
    """Verify error responses for login edge cases."""

    def test_efiro_empty_email_returns_404(
        self, client: TestClient
    ) -> None:
        """Efiro uses email: str (no min_length), so an empty string passes
        DTO validation and reaches the repository.  Without a matching user
        the use case raises UserNotFoundError → 404.

        NOTE: CreateJWTRequestEfiroDTO.email has no length validator, unlike
        the Dexmi variant which uses EmailStr.  Empty-email → 422 only applies
        to DexmiManager / DexmiPass (see TestLoginBodyValidation).
        """

        async def _decrypt(string: str, key_name: str) -> str:
            if key_name == "FRONTEND2SECURITY_BODY_FERNET_KEY":
                return json.dumps({"email": "", "password": "pwd"})
            return string

        with (
            patch(
                f"{_SEC}.decrypt",
                new_callable=AsyncMock,
                side_effect=_decrypt,
            ),
            patch(
                f"{_REPO}.get_user_by_email",
                new_callable=AsyncMock,
                return_value=None,
            ),
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": "Efiro"},
            )
        assert resp.status_code == 404
        assert resp.json()["success"] is False

    def test_empty_password_returns_400(self, client: TestClient) -> None:
        async def _decrypt(string: str, key_name: str) -> str:
            if key_name == "FRONTEND2SECURITY_BODY_FERNET_KEY":
                return json.dumps({"email": "a@b.com", "password": ""})
            return string

        with patch(
            f"{_SEC}.decrypt",
            new_callable=AsyncMock,
            side_effect=_decrypt,
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": "Efiro"},
            )
        assert resp.status_code == 400
        assert resp.json()["success"] is False

    def test_efiro_user_not_found_returns_404(
        self, client: TestClient
    ) -> None:
        with (
            patch(
                f"{_SEC}.decrypt",
                new_callable=AsyncMock,
                side_effect=_login_decrypt,
            ),
            patch(
                f"{_REPO}.get_user_by_email",
                new_callable=AsyncMock,
                return_value=None,
            ),
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": "Efiro"},
            )
        assert resp.status_code == 404
        assert resp.json()["success"] is False

    @pytest.mark.parametrize("platform", DEXMI_PLATFORMS)
    def test_dexmi_user_not_found_returns_404(
        self, client: TestClient, platform: str
    ) -> None:
        with (
            patch(
                f"{_SEC}.decrypt",
                new_callable=AsyncMock,
                side_effect=_login_decrypt,
            ),
            patch(
                f"{_REPO}.get_user_by_username",
                new_callable=AsyncMock,
                return_value=None,
            ),
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": platform},
            )
        assert resp.status_code == 404
        assert resp.json()["success"] is False

    def test_efiro_wrong_password_returns_401(
        self, client: TestClient
    ) -> None:
        with (
            patch(
                f"{_SEC}.decrypt",
                new_callable=AsyncMock,
                side_effect=_login_decrypt,
            ),
            patch(
                f"{_SEC}.verify",
                new_callable=AsyncMock,
                return_value=False,
            ),
            patch(
                f"{_REPO}.get_user_by_email",
                new_callable=AsyncMock,
                return_value=_mock_user("Efiro"),
            ),
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": "Efiro"},
            )
        assert resp.status_code == 401
        assert resp.json()["success"] is False

    @pytest.mark.parametrize("platform", DEXMI_PLATFORMS)
    def test_dexmi_wrong_password_returns_401(
        self, client: TestClient, platform: str
    ) -> None:
        with (
            patch(
                f"{_SEC}.decrypt",
                new_callable=AsyncMock,
                side_effect=_login_decrypt,
            ),
            patch(
                f"{_SEC}.verify",
                new_callable=AsyncMock,
                return_value=False,
            ),
            patch(
                f"{_REPO}.get_user_by_username",
                new_callable=AsyncMock,
                return_value=_mock_user(platform),
            ),
        ):
            resp = client.post(
                f"{BASE_URL}/login",
                json={"data": "enc"},
                headers={"DEX-CHANNEL": platform},
            )
        assert resp.status_code == 401
        assert resp.json()["success"] is False


# ####################################################################
# 5. Refresh — Token Renewal
# ####################################################################
class TestRefresh:
    """Verify JWT refresh flow per platform."""

    @pytest.mark.parametrize("platform", VALID_PLATFORMS)
    def test_refresh_returns_new_tokens(
        self, client: TestClient, platform: str
    ) -> None:
        with patch(
            f"{_UC}.RefreshJWTUseCase.execute",
            new_callable=AsyncMock,
            return_value={
                "access_token": "new_at",
                "refresh_token": "new_rt",
            },
        ):
            resp = client.post(
                f"{BASE_URL}/refresh",
                headers={
                    "DEX-CHANNEL": platform,
                    "Authorization": "Bearer old_refresh_token",
                },
            )
        assert resp.status_code == 201
        data = resp.json()["data"]
        assert data["access_token"] == "new_at"
        assert data["refresh_token"] == "new_rt"

    def test_refresh_invalid_token_returns_401(
        self, client: TestClient
    ) -> None:
        with patch(
            f"{_UC}.RefreshJWTUseCase.execute",
            new_callable=AsyncMock,
            side_effect=__import__(
                "backend.v2.core.domain.exceptions",
                fromlist=["InvalidTokenError"],
            ).InvalidTokenError("Invalid refresh token"),
        ):
            resp = client.post(
                f"{BASE_URL}/refresh",
                headers={
                    "DEX-CHANNEL": "Efiro",
                    "Authorization": "Bearer bad_token",
                },
            )
        assert resp.status_code == 401
        assert resp.json()["success"] is False

    def test_refresh_jwt_error_returns_401(
        self, client: TestClient
    ) -> None:
        """A malformed / tampered JWT triggers JWTError → 401."""
        from jose.exceptions import JWTError

        with patch(
            f"{_UC}.RefreshJWTUseCase.execute",
            new_callable=AsyncMock,
            side_effect=JWTError("Signature verification failed"),
        ):
            resp = client.post(
                f"{BASE_URL}/refresh",
                headers={
                    "DEX-CHANNEL": "Efiro",
                    "Authorization": "Bearer malformed.token",
                },
            )
        assert resp.status_code == 401
        assert resp.json()["success"] is False

    def test_refresh_missing_bearer_returns_403(
        self, client: TestClient
    ) -> None:
        resp = client.post(
            f"{BASE_URL}/refresh",
            headers={"DEX-CHANNEL": "Efiro"},
        )
        assert resp.status_code == 403
        assert resp.json()["success"] is False


# ####################################################################
# 6. Logout
# ####################################################################
class TestLogout:
    """Verify JWT logout flow per platform."""

    @pytest.mark.parametrize("platform", VALID_PLATFORMS)
    def test_logout_success(self, client: TestClient, platform: str) -> None:
        with patch(
            f"{_UC}.LogoutJWTUseCase.execute",
            new_callable=AsyncMock,
        ):
            resp = client.delete(
                f"{BASE_URL}/logout",
                headers={
                    "DEX-CHANNEL": platform,
                    "Authorization": "Bearer refresh_token",
                },
            )
        assert resp.status_code == 200
        assert resp.json()["success"] is True

    def test_logout_invalid_token_returns_401(
        self, client: TestClient
    ) -> None:
        with patch(
            f"{_UC}.LogoutJWTUseCase.execute",
            new_callable=AsyncMock,
            side_effect=__import__(
                "backend.v2.core.domain.exceptions",
                fromlist=["InvalidTokenError"],
            ).InvalidTokenError("Invalid token"),
        ):
            resp = client.delete(
                f"{BASE_URL}/logout",
                headers={
                    "DEX-CHANNEL": "Efiro",
                    "Authorization": "Bearer bad_token",
                },
            )
        assert resp.status_code == 401
        assert resp.json()["success"] is False

    def test_logout_jwt_error_returns_401(
        self, client: TestClient
    ) -> None:
        """A malformed / tampered JWT triggers JWTError → 401."""
        from jose.exceptions import JWTError

        with patch(
            f"{_UC}.LogoutJWTUseCase.execute",
            new_callable=AsyncMock,
            side_effect=JWTError("Signature verification failed"),
        ):
            resp = client.delete(
                f"{BASE_URL}/logout",
                headers={
                    "DEX-CHANNEL": "Efiro",
                    "Authorization": "Bearer malformed.token",
                },
            )
        assert resp.status_code == 401
        assert resp.json()["success"] is False

    def test_logout_missing_bearer_returns_403(
        self, client: TestClient
    ) -> None:
        resp = client.delete(
            f"{BASE_URL}/logout",
            headers={"DEX-CHANNEL": "Efiro"},
        )
        assert resp.status_code == 403
        assert resp.json()["success"] is False


# ####################################################################
# 7. Use-Case Level — Platform-specific Repository Dispatch
# ####################################################################
class TestUseCasePlatformDispatch:
    """Directly test LoginJWTUseCase to verify repository dispatch."""

    @pytest.mark.parametrize(
        "platform,expected_method,dto_name",
        [
            ("Efiro", "get_user_by_email", "CreateJWTRequestEfiroDTO"),
            ("DexmiManager", "get_user_by_username", "CreateJWTRequestDexmiDTO"),
            ("DexmiPass", "get_user_by_username", "CreateJWTRequestDexmiDTO"),
        ],
    )
    @pytest.mark.asyncio()
    async def test_execute_calls_correct_repo_method(
        self, platform: str, expected_method: str, dto_name: str
    ) -> None:
        """Login use case must call the right search method and accept the
        correct DTO type for the given platform."""
        import importlib

        dtos_module = importlib.import_module("backend.v2.jwt.application.dtos")
        DTO = getattr(dtos_module, dto_name)

        mock_user = _mock_user(platform)
        mock_repo = MagicMock()
        mock_repo.get_user_by_email = AsyncMock(return_value=mock_user)
        mock_repo.get_user_by_username = AsyncMock(return_value=mock_user)
        mock_repo.create = AsyncMock()

        mock_request = MagicMock()
        mock_request.app.security_client = MagicMock()
        mock_request.app.security_client.decrypt = AsyncMock(
            return_value="plain_pwd"
        )
        mock_request.app.security_client.encrypt = AsyncMock(
            return_value="enc"
        )
        mock_request.app.security_client.verify = AsyncMock(return_value=True)

        with patch(
            f"{_REPO}.__init__",
            return_value=None,
        ):
            from backend.v2.jwt.application.use_cases import (
                LoginJWTUseCase,
            )

            uc = LoginJWTUseCase(mock_request, platform)
            uc._repository = mock_repo

            body = DTO(email="test@example.com", password="enc_pwd")
            result = await uc.execute(body)

        assert "access_token" in result
        assert "refresh_token" in result
        getattr(mock_repo, expected_method).assert_called_once()

        if expected_method == "get_user_by_email":
            mock_repo.get_user_by_username.assert_not_called()
        else:
            mock_repo.get_user_by_email.assert_not_called()

```

---