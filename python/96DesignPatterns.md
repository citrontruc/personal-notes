# Design patterns

## Table of Content

- [Design patterns](#design-patterns)
  - [Table of Content](#table-of-content)
  - [Option data pattern](#option-data-pattern)
  - [Decorator](#decorator)
  - [Feature flags](#feature-flags)

## Option data pattern

Have a class to store your options. Store defaults + overrides. Immutable, validated configuration.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class ConnectionOptions:
    host: str
    port: int
    user: str
    password: str
    timeout: int = 30
    use_ssl: bool = True

class ConnectionOptionsBuilder:
    def __init__(self):
        self._opts = {
            "timeout": 30,
            "use_ssl": True,
        }

    def host(self, value): self._opts["host"] = value; return self
    def port(self, value): self._opts["port"] = value; return self
    def user(self, value): self._opts["user"] = value; return self
    def password(self, value): self._opts["password"] = value; return self
    def timeout(self, value): self._opts["timeout"] = value; return self
    def use_ssl(self, value): self._opts["use_ssl"] = value; return self

    def build(self) -> ConnectionOptions:
        return ConnectionOptions(**self._opts)

opts = (
    ConnectionOptionsBuilder()
    .host("db.local")
    .port(5432)
    .user("app")
    .password("secret")
    .timeout(10)
    .build()
)
```

## Decorator

Implement a decorator in python:

```python
def decorator(func):
    def wrapper():
        print("Before calling the function.")
        func()
        print("After calling the function.")
    return wrapper

@decorator # Applying the decorator to a function
def greet():
    print("Hello, World!")
greet()
```

You can have decorators that take arguments:

```python
def decorator_name(func):
    def wrapper(*args, **kwargs):
        print("Before execution")
        result = func(*args, **kwargs)
        print("After execution")
        return result
    return wrapper

@decorator_name
def add(a, b):
    return a + b

print(add(5, 3))
```

You can also have functions be arguments of a decorator. You can have decorator for object methods:

```python
def method_decorator(func):
    def wrapper(self, *args, **kwargs):
        print("Before method execution")
        res = func(self, *args, **kwargs)
        print("After method execution")
        return res
    return wrapper

class MyClass:
    @method_decorator
    def say_hello(self):
        print("Hello!")
obj = MyClass()
obj.say_hello()
```

## Feature flags

A pattern to let you roll out new features gradually. Easiest way is to have a configuration flag and a wrapper to check if features are enabled according to the source.

```python
class FeatureFlags:
    def __init__(self, source):
        self.source = source

    def enabled(self, name, default=False):
        return self.source.get(name, default)

flags = FeatureFlags({"new_feature": True})

if flags.enabled("new_feature"):
    run_new_logic()
```
