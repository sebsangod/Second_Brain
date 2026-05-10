---
aliases:
  - Pymongo
tags:
  - learning
  - dev/backend
date: 2026-05-09
---
**Sources**: [Pymongo](https://www.mongodb.com/es/docs/languages/python/pymongo-driver/current/)

**Related:** [[MongoDB]], [[Python]], [[Collection]], [[Document]], [[Pydantic]]

---

## Description

_PyMongo_ is a ``Python`` package that you can use to **connect to and communicate with** ``MongoDB``.

---

## Details

### Installation

```bash title:bash
pip install pymongo
```


### Connection

```python title:main.py
from typing import Any
from asyncio import run
from pymongo import AsyncMongoClient

async def main():
    try:
        uri: str = "<connection string URI>"
        client: AsyncMongoClient[dict[str, Any]] = AsyncMongoClient(uri)

        # ...

        await client.close()

    except Exception as e:
        raise Exception("Exception: ", e)

run(main())

```

The connection ``URI`` is built by the following parts:

```plaintext
"mongodb://<user>:<pwd>@<host>:<port>/?authSource=<db_name>"
```


### Databases and collections

#### Access

```python title:main.py
from pymongo.database import Database
from pymongo.collection import Collection

database: Database = client["db_name"]
collection: Collection = database["collection_name"]

```

>[!TIP]
>If the provided ``collection`` name does not already exist in the database, ``MongoDB`` implicitly creates the ``collection`` when you first insert data into it.


#### Create

```python title:main.py
database = client["db_name"]
new_collection = await database.create_collection("name")

```

#### List

Call the _list_collections()_ method. The method returns a cursor containing all ``collections`` in the database **and their associated metadata**.

```python title:main.py
collections = await database.list_collections()
for c in collections:
    print(c)

```

To query for **only the names** of the collections in the database, call the _list_collection_name()_ method as follows:

```python title:main.py
collections = await database.list_collection_names()
for c in collections:
    print(c)

```


#### Delete

```python  title:main.py
await collection.drop()

```

>[!WARNING]
>Dropping a ``collection`` from a database permanently deletes all ``documents`` and all indexes within that ``collection``.


#### Type Hints

First, **define a class to represent a document from the database**. The class must inherit from the `TypedDict` class and must contain the same fields as the ``documents`` in the database. After you define your class, **include its name as the generic type for the** Database type hint**:

```python title:main.py
from typing import TypedDict, Any
from pymongo import AsyncMongoClient
from pymongo.asynchronous.database import Database

class Movie(TypedDict):
    name: str
    year: int

client: AsyncMongoClient[dict[str, Any]] = AsyncMongoClient()
database: Database[Movie] = client["db_name"]

```

``Pydantic`` could also be used for the data validation part:

```python title:main.py
from typing import Any
from pydantic import BaseModel
from pymongo import AsyncMongoClient
from pymongo.asynchronous.database import Database

class MovieDB(BaseModel):
    name: str
    year: int

class MovieCollection(BaseModel):
    name: str
    year: int

client: AsyncMongoClient[dict[str, Any]] = AsyncMongoClient()
database: Database[MovieDB] = client["db_name"]
collection: Collection[MovieCollection] = database["name"]

```

### CRUD Operations

#### Find One

```python title:main.py
results = await collection.find_one({"<field>": "<value>"})
print(result)

```


#### Find Multiple

```python title:main.py
results = await collection.find({"<field>": "<value>"})
async for document in results:
    print(document)

```


#### Insert One

```python title:main.py
result = await collection.insert_one({"<field>": "<value>"})
print(result.acknowledged)

```


#### Insert Multiple

```python title:main.py
document_list = [
   {"<field>": "<value>"},
   {"<field>": "<value>"}
]
result = await collection.insert_many(document_list)
print(result.acknowledged)

```


#### Update One

Updates **the first document** that matches the search criteria:

The following example uses the _update_one()_ method to update the _name_ value of a document named _"Bagels N Buns"_ in the _restaurants_ ``collection``:

```python title:main.py
restaurants = database["restaurants"]

query_filter = {"name": "Bagels N Buns"}
update_operation = {"$set": {"name": "2 Bagels 2 Buns"}}

result = await restaurants.update_one(query_filter, update_operation)
print(result.modified_count)

```


#### Update Multiple

Updates **all documents** that match the search criteria:

```python title:main.py
query_filter = {{"<field_to_match>": "<value_to_match>"}
update_operation = {"$set": {"<field>": "<value>"}}

result = await collection.update_many(query_filter, update_operation)
print(result.modified_count)

```


#### Replace One

```python title:main.py
query_filter = {"<field_to_match>": "<value_to_match>"}
replace_document = {"<new_field_name>": "<new_value>"}

result = await collection.replace_one(query_filter, replace_document)
print(result.modified_count)

```


#### Delete One

```python title:main.py
query_filter = {"<field_to_match>": "<value_to_match>"}
result = await collection.delete_one(query_filter)
print(result.deleted_count)

```


#### Delete Multiple

```python title:main.py
query_filter = {"<field_to_match>": "<value_to_match>"}
result = await collection.delete_many(query_filter)
print(result.deleted_count)

```


#### Count Documents in a Collection

```python title:main.py
count = await collection.count_documents({})
print(count)

```

```python title:main.py
count = await collection.count_documents({"<field>": "<value>"})
print(count)

```


### $ Operator

Operators with _$_ are the heart of ``MongoDB``. **They allow complex operations to be performed directly on the database in an efficient and atomic manner**.

#### 1. Query Operators

##### $eq - Equals to

```python title:main.py
# Finds documents where age is equal to 25
users.find({"age": {"$eq": 25}})
# Equals to:
users.find({"age": 25})

# Async version
async def find_users_by_age():
    async for user in users.find({"age": {"$eq": 25}}):
        print(user)

```

##### $ne - Not equal to

```python title:main.py
# Users who are not 25 years old
users.find({"age": {"$ne": 25}})

# Async version
async def find_users_not_age():
    cursor = users.find({"age": {"$ne": 25}})
    users_list = await cursor.to_list(length=100)
    return users_list

```

##### $gt, $gte, $lt, $lte - Greater than, greater than or equal, less than, less than or equal

```python title:main.py
# Users between 18 and 65 years old
users.find({
    "age": {
        "$gte": 18,
        "$lte": 65
    }
})

# Async version
async def find_adults():
    cursor = users.find({
        "age": {
            "$gte": 18,
            "$lte": 65
        }
    })
    return await cursor.to_list(length=None)

```

##### $in, $nin - In/Not in a list

```python title:main.py
# Users from certain cities
users.find({"city": {"$in": ["Madrid", "Barcelona", "Valencia"]}})

# Users not from these cities
users.find({"city": {"$nin": ["Madrid", "Barcelona"]}})

# Async version
async def find_users_by_cities():
    cities = ["Madrid", "Barcelona", "Valencia"]
    cursor = users.find({"city": {"$in": cities}})
    return await cursor.to_list(length=None)

```

#### 2. Logic Operators

##### $and - Logical AND

```python title:main.py
# Active users older than 18
users.find({
    "$and": [
        {"age": {"$gt": 18}},
        {"status": "active"}
    ]
})

# Async version
async def find_active_adults():
    cursor = users.find({
        "$and": [
            {"age": {"$gt": 18}},
            {"status": "active"}
        ]
    })
    return await cursor.to_list(length=None)

```


##### $or - Logical OR

```python title:main.py
# VIP users or users with more than 1000 points
users.find({
    "$or": [
        {"type": "VIP"},
        {"points": {"$gt": 1000}}
    ]
})

# Async version
async def find_vip_or_high_points():
    cursor = users.find({
        "$or": [
            {"type": "VIP"},
            {"points": {"$gt": 1000}}
        ]
    })
    return await cursor.to_list(length=None)

```


##### $not - Logical NOT

```python title:main.py
# Users who are NOT older than 65
users.find({"age": {"$not": {"$gt": 65}}})

# Async version
async def find_not_elderly():
    cursor = users.find({"age": {"$not": {"$gt": 65}}})
    return await cursor.to_list(length=None)

```

#### 3. Array Operators

##### $all - Contains all elements

```python title:main.py
# Documents that contain ALL these tags
posts.find({"tags": {"$all": ["mongodb", "database"]}})

# Async version
async def find_posts_with_all_tags():
    cursor = posts.find({"tags": {"$all": ["mongodb", "database"]}})
    return await cursor.to_list(length=None)

```


##### $elemMatch - Complex element match

```python title:main.py
# Users with at least one hobby that costs more than 100
users.find({
    "hobbies": {
        "$elemMatch": {
            "cost": {"$gt": 100}
        }
    }
})

# Async version
async def find_expensive_hobbies():
    cursor = users.find({
        "hobbies": {
            "$elemMatch": {
                "cost": {"$gt": 100}
            }
        }
    })
    return await cursor.to_list(length=None)

```


##### $size - Array size

```python title:main.py
# Users with exactly 3 hobbies
users.find({"hobbies": {"$size": 3}})

# Async version
async def find_users_three_hobbies():
    cursor = users.find({"hobbies": {"$size": 3}})
    return await cursor.to_list(length=None)

```

#### 4. Existence Operators

##### $exists - Field exists

```python title:main.py
# Documents that have the 'email' field
users.find({"email": {"$exists": True}})

# Documents that do NOT have the 'phone' field
users.find({"phone": {"$exists": False}})

# Async version
async def find_users_with_email():
    cursor = users.find({"email": {"$exists": True}})
    return await cursor.to_list(length=None)

```


##### $type - Data type

```python title:main.py
# Documents where 'age' is a number
users.find({"age": {"$type": "number"}})

# You can also use the numeric BSON code
users.find({"age": {"$type": 1}})  # 1 = double, 16 = int32, 18 = int64

# Async version
async def find_users_numeric_age():
    cursor = users.find({"age": {"$type": "number"}})
    return await cursor.to_list(length=None)

```


#### 5. Update Operators

##### Field Operators

###### $set - Set value

```python title:main.py
# Update a user's email
users.update_one(
    {"_id": ObjectId("...")},
    {"$set": {"email": "nuevo@email.com"}}
)

# Async version
async def update_user_email(user_id: str, new_email: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$set": {"email": new_email}}
    )
    return result.modified_count > 0

```


###### $unset - Delete field

```python title:main.py
# Delete the 'temporaryData' field
users.update_one(
    {"_id": ObjectId("...")},
    {"$unset": {"temporaryData": ""}}
)

# Async version
async def remove_temp_data(user_id: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$unset": {"temporaryData": ""}}
    )
    return result.modified_count > 0

```


###### $rename - Rename field

```python title:main.py
# Rename a field
users.update_many(
    {},
    {"$rename": {"oldFieldName": "newFieldName"}}
)

# Async version
async def rename_field():
    result = await users.update_many(
        {},
        {"$rename": {"oldFieldName": "newFieldName"}}
    )
    return result.modified_count

```


###### $inc - Increment numeric value

```python title:main.py
# Increment points by 10
users.update_one(
    {"_id": ObjectId("...")},
    {"$inc": {"points": 10}}
)

# Decrement (increment with negative number)
users.update_one(
    {"_id": ObjectId("...")},
    {"$inc": {"attempts": -1}}
)

# Async version
async def add_points(user_id: str, points: int):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$inc": {"points": points}}
    )
    return result.modified_count > 0

```


###### $mul - Multiply numeric value

```python title:main.py
# Duplicate points
users.update_one(
    {"_id": ObjectId("...")},
    {"$mul": {"points": 2}}
)

# Async version
async def multiply_points(user_id: str, multiplier: float):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$mul": {"points": multiplier}}
    )
    return result.modified_count > 0

```


###### $min, $max - Minimum / Maximum value

```python title:main.py
# Only update if the new value is smaller
users.update_one(
    {"_id": ObjectId("...")},
    {"$min": {"lowestScore": 85}}
)

# Only update if the new value is larger
users.update_one(
    {"_id": ObjectId("...")},
    {"$max": {"highestScore": 95}}
)

# Async version
async def update_best_score(user_id: str, score: int):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$max": {"highestScore": score}}
    )
    return result.modified_count > 0

```

##### Array Operators

###### $push - Add element to end

```python title:main.py
# Add a new hobby
users.update_one(
    {"_id": ObjectId("...")},
    {"$push": {"hobbies": "reading"}}
)

# Add multiple elements
users.update_one(
    {"_id": ObjectId("...")},
    {"$push": {"hobbies": {"$each": ["gaming", "cooking"]}}}
)

# Async version
async def add_hobby(user_id: str, hobby: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$push": {"hobbies": hobby}}
    )
    return result.modified_count > 0

async def add_multiple_hobbies(user_id: str, hobbies: list):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$push": {"hobbies": {"$each": hobbies}}}
    )
    return result.modified_count > 0

```


###### $addToSet - Add only if it doesn't exist

```python title:main.py
# Add hobby only if it doesn't exist yet
users.update_one(
    {"_id": ObjectId("...")},
    {"$addToSet": {"hobbies": "reading"}}
)

# Async version
async def add_unique_hobby(user_id: str, hobby: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$addToSet": {"hobbies": hobby}}
    )
    return result.modified_count > 0

```


###### $pop - Remove first or last element

```python  title:main.py
# Remove last element
users.update_one(
    {"_id": ObjectId("...")},
    {"$pop": {"hobbies": 1}}
)

# Remove first element
users.update_one(
    {"_id": ObjectId("...")},
    {"$pop": {"hobbies": -1}}
)

# Async version
async def remove_last_hobby(user_id: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$pop": {"hobbies": 1}}
    )
    return result.modified_count > 0

```


###### $pull - Remove matching elements

```python title:main.py
# Remove specific hobby
users.update_one(
    {"_id": ObjectId("...")},
    {"$pull": {"hobbies": "reading"}}
)

# Remove elements that meet a condition
users.update_one(
    {"_id": ObjectId("...")},
    {"$pull": {"scores": {"$lt": 50}}}
)

# Async version
async def remove_hobby(user_id: str, hobby: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$pull": {"hobbies": hobby}}
    )
    return result.modified_count > 0

```


###### $pullAll - Remove multiple specific values

```python title:main.py
# Remove multiple specific hobbies
users.update_one(
    {"_id": ObjectId("...")},
    {"$pullAll": {"hobbies": ["reading", "gaming"]}}
)

# Async version
async def remove_multiple_hobbies(user_id: str, hobbies_to_remove: list):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$pullAll": {"hobbies": hobbies_to_remove}}
    )
    return result.modified_count > 0

```

##### Positional Operator

###### $ - Element that matched in the query

```python title:main.py
# Update the first array element that matches
users.update_one(
    {"hobbies.name": "reading"},
    {"$set": {"hobbies.$.cost": 50}}
)

# Async version
async def update_matching_hobby(user_id: str, hobby_name: str, new_cost: int):
    result = await users.update_one(
        {"_id": ObjectId(user_id), "hobbies.name": hobby_name},
        {"$set": {"hobbies.$.cost": new_cost}}
    )
    return result.modified_count > 0

```


###### $[] - All array elements

```python title:main.py
# Update all elements
users.update_one(
    {"_id": ObjectId("...")},
    {"$set": {"hobbies.$[].active": True}}
)

# Async version
async def activate_all_hobbies(user_id: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$set": {"hobbies.$[].active": True}}
    )
    return result.modified_count > 0

```


###### $\[\<identifier\>\] - Elements that meet a condition

```python title:main.py
# Update only expensive hobbies
users.update_one(
    {"_id": ObjectId("...")},
    {"$set": {"hobbies.$[expensive].discounted": True}},
    array_filters=[{"expensive.cost": {"$gt": 100}}]
)

# Async version
async def discount_expensive_hobbies(user_id: str):
    result = await users.update_one(
        {"_id": ObjectId(user_id)},
        {"$set": {"hobbies.$[expensive].discounted": True}},
        array_filters=[{"expensive.cost": {"$gt": 100}}]
    )
    return result.modified_count > 0

```


#### 6. Aggregation Operators

##### $match - Filter documents

```python title:main.py
pipeline = [
    {"$match": {"age": {"$gte": 18}}}
]
result = list(users.aggregate(pipeline))

# Async version
async def get_adult_users():
    pipeline = [
        {"$match": {"age": {"$gte": 18}}}
    ]
    cursor = users.aggregate(pipeline)
    return await cursor.to_list(length=None)

```

##### $group - Group documents

```python title:main.py
pipeline = [
    {
        "$group": {
            "_id": "$city",
            "averageAge": {"$avg": "$age"},
            "count": {"$sum": 1}
        }
    }
]
result = list(users.aggregate(pipeline))

# Async version
async def get_city_stats():
    pipeline = [
        {
            "$group": {
                "_id": "$city",
                "averageAge": {"$avg": "$age"},
                "count": {"$sum": 1},
                "maxAge": {"$max": "$age"},
                "minAge": {"$min": "$age"}
            }
        }
    ]
    cursor = users.aggregate(pipeline)
    return await cursor.to_list(length=None)

```


##### $project - Select/transform fields

```python title:main.py
pipeline = [
    {
        "$project": {
            "name": 1,
            "email": 1,
            "age": 1,
            "isAdult": {"$gte": ["$age", 18]},
            "hobbyCount": {"$size": "$hobbies"}
        }
    }
]
result = list(users.aggregate(pipeline))

# Async version
async def get_user_summary():
    pipeline = [
        {
            "$project": {
                "name": 1,
                "email": 1,
                "age": 1,
                "isAdult": {"$gte": ["$age", 18]},
                "hobbyCount": {"$size": "$hobbies"}
            }
        }
    ]
    cursor = users.aggregate(pipeline)
    return await cursor.to_list(length=None)

```


##### $sort - Sort

```python title:main.py
pipeline = [
    {"$sort": {"age": -1, "name": 1}}
]
result = list(users.aggregate(pipeline))

# Async version
async def get_users_sorted():
    pipeline = [
        {"$sort": {"age": -1, "name": 1}}
    ]
    cursor = users.aggregate(pipeline)
    return await cursor.to_list(length=None)

```


##### $limit and $skip - Pagination

```python title:main.py
def get_users_page(page: int, page_size: int = 10):
    skip = (page - 1) * page_size
    pipeline = [
        {"$skip": skip},
        {"$limit": page_size}
    ]
    return list(users.aggregate(pipeline))

# Async version
async def get_users_page_async(page: int, page_size: int = 10):
    skip = (page - 1) * page_size
    pipeline = [
        {"$skip": skip},
        {"$limit": page_size}
    ]
    cursor = users.aggregate(pipeline)
    return await cursor.to_list(length=None)

```


##### $lookup - JOIN with another collection

```python
orders = db.orders

pipeline = [
    {
        "$lookup": {
            "from": "users",
            "localField": "userId",
            "foreignField": "_id",
            "as": "userInfo"
        }
    }
]
result = list(orders.aggregate(pipeline))

# Async version
async def get_orders_with_users():
    pipeline = [
        {
            "$lookup": {
                "from": "users",
                "localField": "userId",
                "foreignField": "_id",
                "as": "userInfo"
            }
        }
    ]
    cursor = orders.aggregate(pipeline)
    return await cursor.to_list(length=None)
```

___

## Practical Use Cases

### Blog System (Async)

```python title:main.py
from datetime import datetime
from uuid import uuid4

class BlogDAL:
    def __init__(self, posts_collection):
        self._posts = posts_collection

    async def create_post(self, title: str, content: str, author: str, tags: list):
        post = {
            "title": title,
            "content": content,
            "author": author,
            "tags": tags,
            "comments": [],
            "likes": 0,
            "published": True,
            "createdAt": datetime.now()
        }
        result = await self._posts.insert_one(post)
        return str(result.inserted_id)

    async def add_comment(self, post_id: str, author: str, text: str):
        comment = {
            "id": uuid4().hex,
            "author": author,
            "text": text,
            "likes": 0,
            "createdAt": datetime.now()
        }
        result = await self._posts.update_one(
            {"_id": ObjectId(post_id)},
            {"$push": {"comments": comment}}
        )
        return result.modified_count > 0

    async def like_comment(self, post_id: str, comment_id: str):
        result = await self._posts.update_one(
            {"_id": ObjectId(post_id), "comments.id": comment_id},
            {"$inc": {"comments.$.likes": 1}}
        )
        return result.modified_count > 0

    async def remove_unpopular_comments(self, post_id: str):
        result = await self._posts.update_one(
            {"_id": ObjectId(post_id)},
            {"$pull": {"comments": {"likes": 0}}}
        )
        return result.modified_count > 0

    async def get_posts_by_tags(self, tags: list):
        cursor = self._posts.find(
            {"tags": {"$all": tags}},
            {"title": 1, "author": 1, "likes": 1, "createdAt": 1}
        )
        return await cursor.to_list(length=None)

```


### E-commerce System (Async)

```python title:main.py
class EcommerceDAL:
    def __init__(self, users_collection):
        self._users = users_collection

    async def add_to_cart(self, user_id: str, product_id: str, quantity: int, price: float):
        cart_item = {
            "productId": ObjectId(product_id),
            "quantity": quantity,
            "price": price,
            "addedAt": datetime.now()
        }
        result = await self._users.update_one(
            {"_id": ObjectId(user_id)},
            {"$addToSet": {"cart": cart_item}}
        )
        return result.modified_count > 0

    async def increase_quantity(self, user_id: str, product_id: str, increment: int = 1):
        result = await self._users.update_one(
            {
                "_id": ObjectId(user_id),
                "cart.productId": ObjectId(product_id)
            },
            {"$inc": {"cart.$.quantity": increment}}
        )
        return result.modified_count > 0

    async def remove_from_cart(self, user_id: str, product_id: str):
        result = await self._users.update_one(
            {"_id": ObjectId(user_id)},
            {"$pull": {"cart": {"productId": ObjectId(product_id)}}}
        )
        return result.modified_count > 0

    async def get_cart_total(self, user_id: str):
        pipeline = [
            {"$match": {"_id": ObjectId(user_id)}},
            {
                "$project": {
                    "cartTotal": {
                        "$sum": {
                            "$map": {
                                "input": "$cart",
                                "as": "item",
                                "in": {"$multiply": ["$$item.quantity", "$$item.price"]}
                            }
                        }
                    },
                    "itemCount": {"$size": "$cart"}
                }
            }
        ]
        cursor = self._users.aggregate(pipeline)
        result = await cursor.to_list(length=1)
        return result[0] if result else None

    async def clear_cart(self, user_id: str):
        result = await self._users.update_one(
            {"_id": ObjectId(user_id)},
            {"$set": {"cart": []}}
        )
        return result.modified_count > 0

```


### To-Do List System

```python title:main.py
from motor.motor_asyncio import AsyncIOMotorCollection
from uuid import uuid4

class ToDoDAL:
    def __init__(self, todo_collection: AsyncIOMotorCollection):
        self._todo_collection = todo_collection

    async def list_todo_lists(self, session=None):
        # Using $size to count items
        cursor = self._todo_collection.find(
            {},
            projection={
                "name": 1,
                "item_count": {"$size": "$items"},
                "completed_count": {
                    "$size": {
                        "$filter": {
                            "input": "$items",
                            "cond": {"$eq": ["$$this.checked", True]}
                        }
                    }
                }
            },
            sort={"name": 1},
            session=session,
        )
        async for doc in cursor:
            yield doc

    async def create_todo_list(self, name: str, session=None) -> str:
        response = await self._todo_collection.insert_one(
            {"name": name, "items": []},
            session=session,
        )
        return str(response.inserted_id)

    async def get_todo_list(self, id: str | ObjectId, session=None):
        doc = await self._todo_collection.find_one(
            {"_id": ObjectId(id)},
            session=session,
        )
        return doc

    async def delete_todo_list(self, id: str | ObjectId, session=None) -> bool:
        response = await self._todo_collection.delete_one(
            {"_id": ObjectId(id)},
            session=session,
        )
        return response.deleted_count == 1

    async def create_item(self, id: str | ObjectId, label: str, session=None):
        # Using $push to add item
        result = await self._todo_collection.find_one_and_update(
            {"_id": ObjectId(id)},
            {
                "$push": {
                    "items": {
                        "id": uuid4().hex,
                        "label": label,
                        "checked": False,
                        "createdAt": datetime.now()
                    }
                }
            },
            session=session,
            return_document=True,  # ReturnDocument.AFTER in recent versions
        )
        return result

    async def set_checked_state(self, doc_id: str | ObjectId, item_id: str, checked_state: bool, session=None):
        # Using $ (positional operator) to update specific element
        result = await self._todo_collection.find_one_and_update(
            {"_id": ObjectId(doc_id), "items.id": item_id},
            {"$set": {"items.$.checked": checked_state}},
            session=session,
            return_document=True,
        )
        return result

    async def delete_item(self, doc_id: str | ObjectId, item_id: str, session=None):
        # Using $pull to delete specific item
        result = await self._todo_collection.find_one_and_update(
            {"_id": ObjectId(doc_id)},
            {"$pull": {"items": {"id": item_id}}},
            session=session,
            return_document=True,
        )
        return result

    async def mark_all_complete(self, doc_id: str | ObjectId, session=None):
        # Using $[] to update all elements of the array
        result = await self._todo_collection.find_one_and_update(
            {"_id": ObjectId(doc_id)},
            {"$set": {"items.$[].checked": True}},
            session=session,
            return_document=True,
        )
        return result

    async def delete_completed_items(self, doc_id: str | ObjectId, session=None):
        # Using $pull with condition to delete completed items
        result = await self._todo_collection.find_one_and_update(
            {"_id": ObjectId(doc_id)},
            {"$pull": {"items": {"checked": True}}},
            session=session,
            return_document=True,
        )
        return result

    async def get_todo_stats(self):
        # Complex aggregation pipeline
        pipeline = [
            {
                "$project": {
                    "name": 1,
                    "totalItems": {"$size": "$items"},
                    "completedItems": {
                        "$size": {
                            "$filter": {
                                "input": "$items",
                                "cond": {"$eq": ["$$this.checked", True]}
                            }
                        }
                    }
                }
            },
            {
                "$addFields": {
                    "completionPercentage": {
                        "$cond": {
                            "if": {"$eq": ["$totalItems", 0]},
                            "then": 0,
                            "else": {
                                "$multiply": [
                                    {"$divide": ["$completedItems", "$totalItems"]},
                                    100
                                ]
                            }
                        }
                    }
                }
            },
            {"$sort": {"completionPercentage": -1}}
        ]
        cursor = self._todo_collection.aggregate(pipeline)
        return await cursor.to_list(length=None)

```

---

## Claude Sessions
