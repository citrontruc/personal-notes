# OOP

## Interfaces

In order to implement an interface in python, create an abstract class with only abstract methods.

```python
"""
An interface to handle elements that can be controllable.
"""
from abc import ABC
from abc import abstractmethod

from pygame import Surface


class IControllable(ABC):
    @abstractmethod
    def get_position(self) -> list:
        pass

    @abstractmethod
    def handle_input(self, delta_time: float, action_dict: dict) -> None:
        pass

    @abstractmethod
    def update(self, delta_time: float, event_list: list) -> None:
        pass

    @abstractmethod
    def draw(self, surface: Surface) -> None:
        pass
```
