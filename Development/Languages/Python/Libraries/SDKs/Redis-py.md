---
aliases:
  - Redis-py
tags:
  - learning
  - dev/backend
date: 2026-05-09
---
**Sources**: [Datacamp](https://www.datacamp.com/tutorial/python-redis-beginner-guide), [Redis-py](https://github.com/redis/redis-py)

**Related:** [[Redis]], [[Python]], [[Pub-Sub]], [[Hashed Table]], [[Set]], [[Queue]], [[API]]

---

## Description

_Redis-py_ **is the official Redis client library for** ``Python``. It provides a user-friendly interface for communicating with ``Redis`` from within your ``Python`` code.

---

## Key concepts

- Support for **all** ``Redis`` **data types**
- Pipeline and transaction support for efficient and **atomic operations**
- ``Pub/sub`` messaging support
- **Automatic encoding/decoding** of data to/from bytes, strings, and other Python types
- Cluster mode support for working with ``Redis`` **clusters**

---

## Details

### Installation

```bash title:bash
pip install redis
```


### Connection

```python title:main.py
from redis import Redis


r = Redis(
	host='localhost',
	port=6379,
	db=0, # The default Redis database index. Goes from 0 to 15
	decode_responses=True
)

```

---

## Examples

### Hashes

Hashes are similar to ``Python`` _dictionaries_ (`hashed table`) but stored in ``Redis``. They’re best for grouping related fields (e.g., user details).

```python title:main.py
# HSET: Store 'name' and 'email' fields for a user hash key
r.hset("user:1001", "name", "Alice")
r.hset("user:1001", "email", "alice@example.com")

# HGET: Retrieve a single field from the hash
email = r.hget("user:1001", "email")
print(email.decode('utf-8'))  # alice@example.com

# HDEL: Remove a field from the hash
r.hdel("user:1001", "email")

```


#### Storing and accessing structured user profiles

```python title:main.py
def create_user_profile(user_id, name, email):
    """
    Creates a user profile in Redis under the key 'user:{user_id}'.
    'name' and 'email' are stored as separate fields in the hash.
    """
    user_key = f"user:{user_id}"
    r.hset(user_key, mapping={"name": name, "email": email})

def get_user_profile(user_id):
    """
    Retrieves and returns all fields in the user profile hash
    as a Python dictionary. Keys and values are decoded from bytes.
    """
    user_key = f"user:{user_id}"
    profile_data = r.hgetall(user_key)
    return {k.decode('utf-8'): v.decode('utf-8') for k, v in profile_data.items()}

def delete_user_profile(user_id):
    """
    Deletes the entire user profile key from Redis.
    """
    user_key = f"user:{user_id}"
    r.delete(user_key)

# Usage demonstration
create_user_profile(1002, "Bob", "bob@example.com")
print(get_user_profile(1002))  # e.g. {'name': 'Bob', 'email': 'bob@example.com'}
delete_user_profile(1002)

```


### Sets and Sorted Sets

Data in ``Redis`` can be managed using _sets_ or _sorted sets_. 


#### Sets

A _set_ in ``Redis`` is similar to a mathematical _set_. It contains a **collection of unique elements, and no duplicate elements are allowed.** _Sets_ can be used to model relationships between different entities, such as users who have liked a post on social media.

``Redis`` _sets_ **ensure uniqueness of members**. Attempting to add a duplicate member will have no effect.

```python title:main.py
# SADD: Add multiple members to a set
r.sadd("tags:python", "redis", "windows", "backend")

# SMEMBERS: Retrieve all unique members in the set
tags = r.smembers("tags:python")
print(tags)  # {b'redis', b'windows', b'backend'}

```

### Sorted sets

_Sorted sets_ in ``Redis`` are similar to regular sets, but **each member also has a corresponding score**. This allows for **efficient sorting and ranking operations** on the _set_.

_Sorted sets_ are often used for **leaderboards**, where the score represents a player's rank or points.

_Sorted sets_ store members in a **specific order determined by their numeric scores**. _zrange("leaderboard", 0, -1, withscores=True)_ returns all members from rank 0 to the end, including their scores.

```python title:main.py
# ZADD: Add members with scores
r.zadd("leaderboard", {"player1": 10, "player2": 20})

# ZRANGE: Retrieve members in ascending order of score
leaders = r.zrange("leaderboard", 0, -1, withscores=True)
print(leaders)  # [(b'player1', 10.0), (b'player2', 20.0)]

```

#### Managing tags or leaderboards

```python title:main.py
def add_tag(post_id, tag):
    """
    Adds a 'tag' to the set of tags belonging to a specific post.
    Each post has its own set under 'post:{post_id}:tags'.
    """
    r.sadd(f"post:{post_id}:tags", tag)

def get_tags(post_id):
    """
    Retrieves all tags for a specific post, decoding the bytes into strings.
    """
    raw_tags = r.smembers(f"post:{post_id}:tags")
    return {tag.decode('utf-8') for tag in raw_tags}

def update_leaderboard(player, score):
    """
    Updates or inserts a player's score in the 'game:leaderboard' sorted set.
    A higher score indicates a better position if sorting descending.
    """
    r.zadd("game:leaderboard", {player: score})

def get_leaderboard():
    """
    Returns an ascending list of (player, score) tuples from the leaderboard.
    To invert the sorting (highest first), you'd use ZREVRANGE.
    """
    entries = r.zrange("game:leaderboard", 0, -1, withscores=True)
    return [(player.decode('utf-8'), score) for player, score in entries]

# Usage demonstration
add_tag(123, "python")
add_tag(123, "redis")
print(get_tags(123))

update_leaderboard("Alice", 300)
update_leaderboard("Bob", 450)
print(get_leaderboard())

```

___

## Features in ``Python`` Applications

``Redis`` has several features that make it a popular choice for data storage in ``Python`` applications: ``queueing``, ``locking``, and ``caching``. 


### ``Redis`` as a ``queue`` (with blocking)

Let’s explore using ``Redis`` as a queue with blocking using _BLPOP_.

_BLPOP_ is a blocking operation that **waits until an element is available in a list**.

_blocking_consumer_ uses _blpop_ in a loop. If the list is empty, _blpop_ will **wait until another item is pushed.**

**Once an item is received, it’s removed from the list and printed.**

This approach is ideal for producer-consumer patterns where workers consume tasks as they appear.

```python title:main.py
def blocking_consumer(queue_name):
    """
    Continuously listens to the specified queue (Redis list) using BLPOP,
    which blocks until new items are pushed. Once an item arrives,
    it is removed from the queue and processed.
    """
    print(f"Waiting on queue: {queue_name}")
    while True:
        result = r.blpop(queue_name)
        if result:
            list_name, task_bytes = result
            task = task_bytes.decode('utf-8')
            print(f"Received task: {task}")
        else:
            print("Queue is empty or an error occurred.")
            break

def enqueue_task(queue_name, task):
    """
    Pushes a task to the end of a Redis list (queue).
    """
    r.rpush(queue_name, task)

# Example usage:
enqueue_task("blocking_queue", "task_block_1")
enqueue_task("blocking_queue", "task_block_2")

# In a real application, the consumer might run in a separate thread or process
blocking_consumer("blocking_queue")

```


### Implementing ``locks`` in ``Redis``

Distributed _locks_ **prevent multiple clients or processes from simultaneously modifying the same resource.** ``Redis`` can help avoid race conditions in a distributed environment.

```python title:main.py
from time import sleep
from redis.exceptions import LockError


def process_critical_section():
    """
    Acquires a lock named 'resource_lock' with a timeout of 10 seconds.
    The lock automatically expires after 10 seconds to prevent deadlocks.
    """
    lock = r.lock("resource_lock", timeout=10)
    try:
        # Attempt to acquire the lock, wait for up to 5 seconds if another process holds it
        acquired = lock.acquire(blocking=True, blocking_timeout=5)
        if acquired:
            print("Lock acquired; performing critical operation...")
            sleep(3)  # Simulate some operation
        else:
            print("Failed to acquire lock within 5 seconds.")
    except LockError:
        print("A LockError occurred, possibly releasing already released lock.")
    finally:
        # Always release the lock in a finally block to ensure cleanup
        lock.release()
        print("Lock released.")

# Usage demonstration
process_critical_section()

```

We create a lock with _timeout=10_, meaning if the lock isn’t released manually, ``Redis`` will automatically remove it in 10 seconds to prevent indefinite blocking (deadlock). _lock.acquire(blocking=True, blocking_timeout=5)_ tries to acquire the lock for 5 seconds before giving up. After finishing, _lock.release()_ frees the resource for other processes.

#### Ready-to-use lock class

```python title:main.py
import redis
import uuid
import time

class RedisLock:
    def __init__(self, redis_client, lock_name, timeout=10):
        """
        Inicializa un lock distribuido en Redis
        
        Args:
            redis_client: Cliente de Redis
            lock_name: Nombre del lock
            timeout: Tiempo de expiración en segundos
        """
        self.redis = redis_client
        self.lock_name = f"lock:{lock_name}"
        self.timeout = timeout
        self.identifier = str(uuid.uuid4())  # ID único para este cliente
        
    def acquire(self, blocking=True, blocking_timeout=None):
        """
        Intenta adquirir el lock
        
        Args:
            blocking: Si True, espera hasta adquirir el lock
            blocking_timeout: Tiempo máximo de espera en segundos
        
        Returns:
            bool: True si se adquirió el lock, False si no
        """
        end_time = None
        if blocking_timeout is not None:
            end_time = time.time() + blocking_timeout
            
        while True:
            # Intenta establecer el lock con SET NX EX
            acquired = self.redis.set(
                self.lock_name,
                self.identifier,
                nx=True,  # Solo si no existe
                ex=self.timeout  # Tiempo de expiración
            )
            
            if acquired:
                return True
            
            # Si no es bloqueante, retorna inmediatamente
            if not blocking:
                return False
            
            # Si hay timeout de bloqueo, verifica si se alcanzó
            if end_time and time.time() > end_time:
                return False
            
            # Espera un poco antes de reintentar
            time.sleep(0.1)
    
    def release(self):
        """
        Libera el lock solo si este cliente lo posee
        
        Returns:
            bool: True si se liberó exitosamente
        """
        # Script Lua para liberar de forma atómica
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        
        result = self.redis.eval(lua_script, 1, self.lock_name, self.identifier)
        return bool(result)
    
    def extend(self, additional_time=None):
        """
        Extiende el tiempo de expiración del lock
        
        Args:
            additional_time: Tiempo adicional en segundos (usa timeout por defecto)
        
        Returns:
            bool: True si se extendió exitosamente
        """
        if additional_time is None:
            additional_time = self.timeout
            
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("expire", KEYS[1], ARGV[2])
        else
            return 0
        end
        """
        
        result = self.redis.eval(
            lua_script, 
            1, 
            self.lock_name, 
            self.identifier,
            additional_time
        )
        return bool(result)
    
    def __enter__(self):
        """Soporte para context manager (with statement)"""
        if not self.acquire():
            raise RuntimeError(f"No se pudo adquirir el lock: {self.lock_name}")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """Libera el lock al salir del context manager"""
        self.release()


# Ejemplo de uso
if __name__ == "__main__":
    # Conectar a Redis
    r = redis.Redis(host='localhost', port=6379, decode_responses=True)
    
    print("=== Ejemplo 1: Uso básico ===")
    lock = RedisLock(r, "mi_recurso", timeout=5)
    
    if lock.acquire():
        print("✓ Lock adquirido")
        try:
            # Simula trabajo con el recurso
            print("Trabajando con el recurso protegido...")
            time.sleep(2)
        finally:
            lock.release()
            print("✓ Lock liberado")
    else:
        print("✗ No se pudo adquirir el lock")
    
    print("\n=== Ejemplo 2: Uso con context manager ===")
    try:
        with RedisLock(r, "otro_recurso", timeout=5):
            print("✓ Lock adquirido automáticamente")
            print("Trabajando con el recurso...")
            time.sleep(1)
        print("✓ Lock liberado automáticamente")
    except RuntimeError as e:
        print(f"✗ Error: {e}")
    
    print("\n=== Ejemplo 3: Lock no bloqueante ===")
    lock1 = RedisLock(r, "recurso_compartido", timeout=5)
    lock2 = RedisLock(r, "recurso_compartido", timeout=5)
    
    lock1.acquire()
    print("✓ Primer cliente adquirió el lock")
    
    if lock2.acquire(blocking=False):
        print("✓ Segundo cliente adquirió el lock")
        lock2.release()
    else:
        print("✗ Segundo cliente no pudo adquirir el lock (esperado)")
    
    lock1.release()
    print("✓ Primer cliente liberó el lock")

```


### ``Caching``

``Caching`` is a common usage scenario: **store frequently accessed data in memory, reducing load on databases or external** ``APIs``.

```python title:main.py
import requests
import json

def get_user_data(user_id):
    """
    Retrieves user data from a hypothetical API endpoint.
    If the data is found in Redis (cache), use that. Otherwise, call the API,
    store the response in Redis with a 60-second expiration, and return it.
    """
    cache_key = f"user_data:{user_id}"
    cached_data = r.get(cache_key)
    if cached_data:
        print("Cache hit!")
        return json.loads(cached_data)

    print("Cache miss. Fetching from API...")
    response = requests.get(f"https://api.example.com/users/{user_id}")
    user_info = response.json()

    # Store in Redis for 60 seconds
    r.setex(cache_key, 60, json.dumps(user_info))
    return user_info

# Usage
user = get_user_data(42)  # First call => cache miss
user_again = get_user_data(42)  # Subsequent call => cache hit

```

We check if _user_data:{user_id}_ exists in ``Redis``. If it does, that’s a cache hit, and we skip the ``API`` call. If not, we fetch from the remote ``API``, then _setex_ (set + expiration) for 60 seconds. Subsequent calls within that timeframe will retrieve the cached data, reducing latency.

---

## Utils

### Custom `Python` connection class

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

## Claude Sessions
