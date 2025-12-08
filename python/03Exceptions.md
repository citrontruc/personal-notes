# Exceptions

## Exception groups

Introduced in python 3.11.

Python has a new exception group functionality where you can group exceptions together. This is used in cases where multiple exceptions should be raised (ex: async, multiple user callback errors, complex calculations, propagating errors).

In order to except elements from an exception group, use the except*
keyword. In exception where more than one error can happen, it can be useful to retrieve them all.

```python
from exceptiongroup import ExceptionGroup  # On Python <3.11
# On Python 3.11+, built-in: no import needed.

def check_positive(x):
    if x <= 0:
        raise ValueError(f"{x} is not positive")

def check_even(x):
    if x % 2 != 0:
        raise ValueError(f"{x} is not even")

def run_validations(x):
    errors = []

    for func in (check_positive, check_even):
        try:
            func(x)
        except Exception as e:
            errors.append(e)

    if errors:
        raise ExceptionGroup("Validation failed", errors)

    return True


try:
    run_validations(-3)
except* ValueError as group:
    print("Caught ValueErrors:")
    for err in group.exceptions:
        print(" -", err)
```
