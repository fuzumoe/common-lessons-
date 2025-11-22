# 📘 Database Connectivity in Python

### MongoDB • MySQL • PostgreSQL • Redis  
### + ODM/ORM: Beanie (MongoDB), Tortoise ORM (MySQL/PostgreSQL)

------------------------------------------------------------------------

## 1. Introduction

Python interacts with databases using:

1. **Low-level drivers**  
2. **ORM/ODM frameworks**

This lesson covers connectivity + CRUD + ORM/ODM for:  
- **MongoDB** (pymongo + Beanie)  
- **MySQL** (mysql-connector + Tortoise ORM)  
- **PostgreSQL** (psycopg + Tortoise ORM)  
- **Redis** (redis-py)

------------------------------------------------------------------------

## 1.1 Connection & Client Concepts

Before we dive into CRUD, it’s important to understand what **“client”** and **“connection”** mean for each database and library.

### General Ideas

- A **client** is the main object you create in Python to talk to the database.
- A **connection** is an active session/channel between your app and the DB server.
- SQL DBs (MySQL/Postgres) also use a **cursor** to execute SQL and fetch results.
- ORMs/ODMs usually manage connections and pools internally.

### MongoDB

**pymongo**

- Uses `MongoClient`:
  ```python
  client = MongoClient("mongodb://user:pass@host:port/")
  ```
- Manages connection pooling, authentication, server selection.

**Beanie**

- Uses `AsyncIOMotorClient` and `init_beanie`.
- After initialization you work with `Document` models, and Beanie handles connections.

### MySQL

**mysql-connector**

- You create a connection and cursor explicitly:
  ```python
  client = mysql.connector.connect(...)
  cur = client.cursor()
  ```

**Tortoise ORM**

- You pass a DB URL to `Tortoise.init`, it creates and manages a pool of connections.

### PostgreSQL

**psycopg**

- You manage `conn` (connection) and `cur` (cursor) explicitly.
```python

client = psycopg.connect(...)
cur = client.cursor()
```

**Tortoise ORM**

- Similar to MySQL: connection details via URL, pooling handled internally.

### Redis

- `redis.Redis(...)` acts as both client and connection handler, exposing Redis commands directly.

------------------------------------------------------------------------

# 2. MongoDB

## 2.1 MongoDB CRUD using `pymongo`

```python
from pymongo import MongoClient
from dotenv import load_dotenv
import os

load_dotenv()

MONGO_USER = os.getenv("MONGO_ROOT_USERNAME")
MONGO_PASS = os.getenv("MONGO_ROOT_PASSWORD")
MONGO_DB   = os.getenv("MONGO_DATABASE")
MONGO_PORT = os.getenv("MONGO_PORT")

client = MongoClient(f"mongodb://{MONGO_USER}:{MONGO_PASS}@localhost:{MONGO_PORT}/")
db = client[MONGO_DB]
users = db["users"]

# CREATE
users.insert_one({"name": "Alice", "age": 30})

# READ
users.find_one({"name": "Alice"})

# UPDATE
users.update_one({"name": "Alice"}, {"$inc": {"age": 1}})

# DELETE
users.delete_one({"name": "Bob"})
```

------------------------------------------------------------------------

## 2.2 MongoDB CRUD using **Beanie ODM**

```python
import asyncio
from motor.motor_asyncio import AsyncIOMotorClient
from beanie import Document, init_beanie
from dotenv import load_dotenv
import os

load_dotenv()

MONGO_USER = os.getenv("MONGO_ROOT_USERNAME")
MONGO_PASS = os.getenv("MONGO_ROOT_PASSWORD")
MONGO_DB   = os.getenv("MONGO_DATABASE")
MONGO_PORT = os.getenv("MONGO_PORT")

class User(Document):
    name: str
    age: int
    active: bool = True

async def main():
    client = AsyncIOMotorClient(
        f"mongodb://{MONGO_USER}:{MONGO_PASS}@localhost:{MONGO_PORT}/"
    )
    await init_beanie(database=client[MONGO_DB], document_models=[User])

    await User.find_all().delete()

    # CREATE
    alice = await User(name="Alice", age=30).insert()
    bob = await User(name="Bob", age=25).insert()

    # READ
    print(await User.find_one(User.name == "Alice"))
    print(await User.find(User.age >= 30).to_list())

    # UPDATE
    alice.age = 31
    await alice.save()
    await User.find(User.name == "Bob").update({"$set": {"active": False}})

    # DELETE
    await alice.delete()
    await User.find(User.active == False).delete()

asyncio.run(main())
```

------------------------------------------------------------------------

# 3. MySQL

## 3.1 MySQL CRUD using `mysql-connector`

```python
import mysql.connector
from dotenv import load_dotenv
import os

load_dotenv()

MYSQL_USER = os.getenv("MYSQL_USER")
MYSQL_PASS = os.getenv("MYSQL_PASSWORD")
MYSQL_DB   = os.getenv("MYSQL_DATABASE")
MYSQL_PORT = os.getenv("MYSQL_PORT")

conn = mysql.connector.connect(
    host="localhost",
    port=MYSQL_PORT,
    user=MYSQL_USER,
    password=MYSQL_PASS,
    database=MYSQL_DB
)
cursor = conn.cursor()

# CREATE
cursor.execute("INSERT INTO users_mysql (name, age) VALUES (%s, %s)", ("Alice", 30))

# READ
cursor.execute("SELECT * FROM users_mysql")
cursor.fetchall()

# UPDATE
cursor.execute("UPDATE users_mysql SET age = age + 1 WHERE name=%s", ("Alice",))

# DELETE
cursor.execute("DELETE FROM users_mysql WHERE name=%s", ("Bob",))

conn.commit()
```

------------------------------------------------------------------------

## 3.2 MySQL CRUD using **Tortoise ORM**

```python
import asyncio
from tortoise import Tortoise, fields
from tortoise.models import Model
from dotenv import load_dotenv
import os

load_dotenv()

MYSQL_USER = os.getenv("MYSQL_USER")
MYSQL_PASS = os.getenv("MYSQL_PASSWORD")
MYSQL_DB   = os.getenv("MYSQL_DATABASE")
MYSQL_PORT = os.getenv("MYSQL_PORT")

DB_URL = f"mysql://{MYSQL_USER}:{MYSQL_PASS}@localhost:{MYSQL_PORT}/{MYSQL_DB}"

class User(Model):
    id = fields.IntField(pk=True)
    name = fields.CharField(max_length=100)
    age = fields.IntField()

    def __str__(self):
        return f"User(id={self.id}, name={self.name}, age={self.age})"

async def main():
    await Tortoise.init(db_url=DB_URL, modules={"models": ["__main__"]})
    await Tortoise.generate_schemas()

    await User.all().delete()

    # CREATE
    alice = await User.create(name="Alice", age=30)
    bob = await User.create(name="Bob", age=25)

    # READ
    print(await User.all())
    print(await User.filter(age__gte=26))
    alice_db = await User.get(id=alice.id)

    # UPDATE
    alice_db.age = 31
    await alice_db.save()
    await User.filter(name="Bob").update(age=27)

    # DELETE
    await alice_db.delete()
    await User.filter(age__lt=27).delete()

    print(await User.all())

asyncio.run(main())
```

------------------------------------------------------------------------

# 4. PostgreSQL

## 4.1 PostgreSQL CRUD using `psycopg`

```python
import psycopg
from dotenv import load_dotenv
import os

load_dotenv()

PG_USER = os.getenv("POSTGRES_USER")
PG_PASS = os.getenv("POSTGRES_PASSWORD")
PG_DB   = os.getenv("POSTGRES_DB")
PG_PORT = os.getenv("POSTGRES_PORT")

conn = psycopg.connect(
    f"host=localhost port={PG_PORT} dbname={PG_DB} user={PG_USER} password={PG_PASS}"
)
cur = conn.cursor()

# CREATE
cur.execute("INSERT INTO users_pg (name, age) VALUES (%s, %s)", ("Alice", 30))

# READ
cur.execute("SELECT * FROM users_pg")
cur.fetchall()

# UPDATE
cur.execute("UPDATE users_pg SET age = age + 1 WHERE name=%s", ("Alice",))

# DELETE
cur.execute("DELETE FROM users_pg WHERE name=%s", ("Bob",))

conn.commit()
cur.close()
conn.close()
```

------------------------------------------------------------------------

## 4.2 PostgreSQL CRUD using **Tortoise ORM**

```python
import asyncio
from tortoise import Tortoise, fields
from tortoise.models import Model
from dotenv import load_dotenv
import os

load_dotenv()

PG_USER = os.getenv("POSTGRES_USER")
PG_PASS = os.getenv("POSTGRES_PASSWORD")
PG_DB   = os.getenv("POSTGRES_DB")
PG_PORT = os.getenv("POSTGRES_PORT")

DB_URL = f"postgres://{PG_USER}:{PG_PASS}@localhost:{PG_PORT}/{PG_DB}"

class UserPG(Model):
    id = fields.IntField(pk=True)
    name = fields.CharField(max_length=100)
    age = fields.IntField()

    def __str__(self):
        return f"UserPG(id={self.id}, name={self.name}, age={self.age})"

async def main():
    await Tortoise.init(db_url=DB_URL, modules={"models": ["__main__"]})
    await Tortoise.generate_schemas()

    await UserPG.all().delete()

    # CREATE
    alice = await UserPG.create(name="Alice", age=30)
    bob = await UserPG.create(name="Bob", age=25)

    # READ
    print(await UserPG.all())
    print(await UserPG.filter(age__gte=26))
    alice_db = await UserPG.get(id=alice.id)

    # UPDATE
    alice_db.age = 31
    await alice_db.save()
    await UserPG.filter(name="Bob").update(age=27)

    # DELETE
    await alice_db.delete()
    await UserPG.filter(age__lt=27).delete()

    print(await UserPG.all())

asyncio.run(main())
```

------------------------------------------------------------------------

# 5. Redis CRUD

```python
import redis
from dotenv import load_dotenv
import os

load_dotenv()

REDIS_PASS = os.getenv("REDIS_PASSWORD")
REDIS_PORT = os.getenv("REDIS_PORT")

r = redis.Redis(
    host="localhost",
    port=REDIS_PORT,
    password=REDIS_PASS,
    db=0
)

# CREATE
r.hset("user:1", mapping={"name": "Alice", "age": "30"})

# READ
print(r.hgetall("user:1"))

# UPDATE
r.hset("user:1", "age", "31")

# DELETE
r.delete("user:1")
```

### alternative from url
```python
import redis
from dotenv import load_dotenv
import os

load_dotenv()

REDIS_PASS = os.getenv("REDIS_PASSWORD")
REDIS_PORT = os.getenv("REDIS_PORT")

# Option 1: Using URL (recommended when you say "also from url")
REDIS_URL = f"redis://:{REDIS_PASS}@localhost:{REDIS_PORT}/0"
r = redis.Redis.from_url(REDIS_URL)

# Option 2: (previous style) Using host/port/password directly
# r = redis.Redis(
#     host="localhost",
#     port=REDIS_PORT,
#     password=REDIS_PASS,
#     db=0
# )

# CREATE
r.hset("user:1", mapping={"name": "Alice", "age": "30"})

# READ
print(r.hgetall("user:1"))

# UPDATE
r.hset("user:1", "age", "31")

# DELETE
r.delete("user:1")

```
------------------------------------------------------------------------

# 6. Summary Table

| Database   | Raw Driver        | ORM/ODM       | Notes            |
|------------|-------------------|----------------|------------------|
| MongoDB    | pymongo           | Beanie         | Document DB      |
| MySQL      | mysql-connector   | Tortoise ORM   | SQL              |
| PostgreSQL | psycopg           | Tortoise ORM   | SQL              |
| Redis      | redis-py          | None           | Key-value store  |
