---
aliases:
  - Redis
tags:
  - learning
---
**Date**: 24/mar/2026
**Update**: 24/mar/2026
**Sources**: [Source]()

**Tags:** #database #nosql

**Related:** [[Python]]

---

## Description

Write here...

---

## Key concepts

Write here...

---

## Details

Write here...

---

## Snippets

Write here...

---

## Utils

### `Python` connection class

```bash title:.env
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=1
```

```python title:main.py
from json import dumps, loads
from logging import Logger, getLogger
from os import getenv
from typing import Any

from dotenv import load_dotenv
from redis import Redis

from backend.config import DOTENV_ABSPATH, LOGGING_PROJECT_NAME

load_dotenv(DOTENV_ABSPATH)
logger: Logger = getLogger(f"{LOGGING_PROJECT_NAME}.{__name__.split('.')[-1]}")


class RedisConnection:
    def __init__(self, env: str = "dev") -> None:
        if env.lower() == "prod":
            from backend.config import ProdQueues as queues
        elif env.lower() in ["dev", "test"]:
            from backend.config import DevQueues as queues

        self.HOST: str = str(getenv("REDIS_HOST"))
        self.PORT: int = int(getenv("REDIS_PORT"))

        self.QUEUE_NEW: str = queues.QUEUE_NEW
        self.QUEUE_EXPIRED: str = queues.QUEUE_EXPIRED
        self.QUEUE_VALIDATED: str = queues.QUEUE_VALIDATED
        self.QUEUE_ERROR: str = queues.QUEUE_ERROR

        self.connection = Redis(
            host=self.HOST,
            port=self.PORT,
            db=int(getenv("REDIS_DB")),
            decode_responses=True,
        )

        self.item = None

    def __str__(self) -> str:
        return (
            f"Redis(server={self.HOST}, "
            f"port={self.PORT}, "
            f"connection={self.connection}"
        )

    def __repr__(self) -> str:
        return (
            f"Redis(server={self.HOST}, "
            f"port={self.PORT}, "
            f"connection={self.connection}"
        )

    # PUSH functions
    # These functions were written to add new items to the queue
    def push_item(self, queue_name: str, item: dict) -> bool:
        """This function will push a new item to the queue"""
        try:
            item = dumps(item)
        except Exception as e:
            logger.exception(f"[{type(e).__name__}] => {str(e)}")
            return False

        try:
            self.connection.rpush(queue_name, item)
        except Exception as e:
            logger.exception(f"[{type(e).__name__}] => {str(e)}")
            return False

        logger.info("Item pushed successfully")
        return True

    # GET functions
    # These functions were writen to read the entire queue (get_XXXs)
    # These functions werew writen to read the first element of the queue
    # 	(get_XXX)
    def get_all(self, queue_name: str) -> list[dict]:
        new_items: list = []
        for item in self.connection.lrange(queue_name, 0, -1):
            new_items.append(loads(item))
        return new_items

    def get_first_item(self, queue_name: str) -> dict:
        new_item = None
        for item in self.connection.lrange(queue_name, 0, 1):
            new_item = loads(item)
        self.item = new_item
        return new_item

    def get_item(
        self, queue_name: str, key: str, value: Any
    ) -> dict[str, Any] | None:
        for item in self.connection.lrange(queue_name, 0, -1):
            new_item: dict[str, Any] = loads(item)
            if (
                isinstance(new_item, str)
                and value in new_item
                or (
                    isinstance(new_item, dict)
                    and key in new_item
                    and new_item[key] == value
                )
            ):
                self.item = new_item
                return self.item
        return None

    # ATOMIC functions
    # These functions were written to esure that the called process completes
    # 	completely.
    # if the process fails, the queues will not be modified at all
    # these types of functions help to prevent data loss mid-process
    def move_update_atomy(
        self,
        source: str,
        destination: str,
        key: dict[str, Any],
        new_item: dict,
    ) -> bool:

        script = """
        local src = KEYS[1]
        local dst = KEYS[2]
        local key_json = ARGV[1]
        local new_val = ARGV[2]

        local key_dict = cjson.decode(key_json)
        local len = redis.call('LLEN', src)
        local search_key, search_value

        for k, v in pairs(key_dict) do
            search_key = k
            search_value = v
            break  -- tomar sólo el primer par key:value
        end

        for i = 0, len - 1 do
            local item = redis.call('LINDEX', src, i)
            if item then
                local ok, data = pcall(cjson.decode, item)
                if ok and data[search_key] == search_value then
                    -- marcar temporalmente
                    redis.call('LSET', src, i, "__TO_DELETE__")
                    -- eliminar real
                    redis.call('LREM', src, 1, "__TO_DELETE__")
                    -- insertar en nueva cola
                    redis.call('RPUSH', dst, new_val)
                    return 1
                end
            end
        end
        return 0
        """
        try:
            new_item = dumps(new_item)
        except Exception as e:
            logger.exception(f"{type(e).__name__} => {str(e)}")
            return None
        try:
            result = self.connection.eval(
                script, 2, source, destination, dumps(key), new_item
            )
        except Exception as e:
            logger.exception(f"{type(e).__name__} => {str(e)}")
            return None
        if result <= 0:
            logger.warning(f"Element could not be processed => {key}")
            return False
        else:
            logger.info(f"Element processed successfully: => {key}")
            self.item = new_item
            return True


redis_conn = RedisConnection()

```

---