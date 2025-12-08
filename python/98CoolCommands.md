# Cool Commands

## Deferred evaluation of annotations

In python 3.14, annotations are no longer eagerly evaluated, but instead we have a deferred evaluation:

```python
class A:
  def some_func(self, *args) -> B | None:  # Just Works (TM)
    ...

class B:
  ...
```
