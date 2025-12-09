# Data Structures

## Typed Dicts

Lets you define an interface for a dictionnary:

```python
class Band(TypedDict):
    name: str
    members: ReadOnly[list[str]]

blur: Band = {"name": "blur", "members": []} # Band has to respect TypedDict
blur["name"] = "Blur"  # OK: "name" is not read-only
blur["members"] = ["Damon Albarn"]  # Type check error: "members" is read-only
blur["members"].append("Damon Albarn")  # OK: list is mutable
```

## Enums in python

```python
from enum import Enum

class Color(Enum):
    RED = 1
    BLUE = 2
```
