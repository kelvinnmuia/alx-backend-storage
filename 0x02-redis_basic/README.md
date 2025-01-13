# 0x02. Redis basic
## The Domains/Concepts covered in this project: `Back-end` `Redis`


This project introduced me to Redis, a versatile in-memory data structure store that functions as a database, cache, and message broker. I learned how to interact with Redis using Python, perform operations on data structures like strings and hashes, and leverage its speed and real-time capabilities to enhance backend performance.

## Install Redis on Ubuntu 18.04

```
$ sudo apt-get -y install redis-server
$ pip3 install redis
$ sed -i "s/bind .*/bind 127.0.0.1/g" /etc/redis/redis.conf
```

## Use Redis in a container

Redis server is stopped by default - when you are starting a container, you should start it with: `service redis-server start`


## Tasks :page_with_curl:

**0. Writing strings to Redis**

Create a `Cache` class. In the `__init__` method, store an instance of the Redis client as a private variable named `_redis` (using `redis.Redis()`) and flush the instance using `flushdb`.

Create a `store` method that takes a `data` argument and returns a string. The method should generate a random key (e.g. using `uuid`), store the input data in Redis using the random key and return the key.

Type-annotate `store` correctly. Remember that `data` can be a `str`, `bytes`, `int` or `float`.

```
bob@dylan:~$ cat main.py
#!/usr/bin/env python3
"""
Main file
"""
import redis

Cache = __import__('exercise').Cache

cache = Cache()

data = b"hello"
key = cache.store(data)
print(key)

local_redis = redis.Redis()
print(local_redis.get(key))

bob@dylan:~$ python3 main.py 
3a3e8231-b2f6-450d-8b0e-0f38f16e8ca2
b'hello'
bob@dylan:~$ 
```

  * [exercise.py](./exercise.py)

**1. Reading from Redis and recovering original type**

Redis only allows to store string, bytes and numbers (and lists thereof). Whatever you store as single elements, it will be returned as a byte string. Hence if you store `"a"` as a UTF-8 string, it will be returned as `b"a"` when retrieved from the server.

In this exercise we will create a `get` method that take a `key` string argument and an optional `Callable` argument named `fn`. This callable will be used to convert the data back to the desired format.

Remember to conserve the original `Redis.get` behavior if the key does not exist.

Also, implement 2 new methods: `get_str` and `get_int` that will automatically parametrize `Cache.get` with the correct conversion function.

The following code should not raise:

```
cache = Cache()

TEST_CASES = {
    b"foo": None,
    123: int,
    "bar": lambda d: d.decode("utf-8")
}

for value, fn in TEST_CASES.items():
    key = cache.store(value)
    assert cache.get(key, fn=fn) == value
```

  * [exercise.py](./exercise.py)

**2. Incrementing values**

Familiarize yourself with the `INCR` command and its python equivalent.

In this task, we will implement a system to count how many times methods of the `Cache` class are called.

Above `Cache` define a `count_calls` decorator that takes a single `method` `Callable` argument and returns a `Callable`.

As a key, use the qualified name of `method` using the `__qualname__` dunder method.

Create and return function that increments the count for that key every time the method is called and returns the value returned by the original method.

Remember that the first argument of the wrapped function will be `self` which is the instance itself, which lets you access the Redis instance.

Protip: when defining a decorator it is useful to use `functool.wraps` to conserve the original function’s name, docstring, etc. Make sure you use it as described here.

Decorate `Cache.store` with `count_calls`.

```
bob@dylan:~$ cat main.py
#!/usr/bin/env python3
""" Main file """

Cache = __import__('exercise').Cache

cache = Cache()

cache.store(b"first")
print(cache.get(cache.store.__qualname__))

cache.store(b"second")
cache.store(b"third")
print(cache.get(cache.store.__qualname__))

bob@dylan:~$ ./main.py
b'1'
b'3'
bob@dylan:~$ 
```

  * [exercise.py](./exercise.py)
