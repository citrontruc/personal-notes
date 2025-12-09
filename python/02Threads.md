# Threading

## Table of content

- [Threading](#threading)
  - [Table of content](#table-of-content)
  - [free-threading](#free-threading)

## free-threading

Python had a mechanism to control the number of threads used to execute commands. This constraint disappeared in python 3.13.

Note: You have to disable gil in configuration file: <https://dev.to/mechcloud_academy/unlocking-true-parallelism-a-developers-guide-to-free-threaded-python-314-175i>

```bash
./configure --disable-gil
```

Example:

```python
# cpu_bound_task.py
import time
import threading

def heavy_calculation(n):
    """A function that simulates a CPU-bound task."""
    total = 0
    for i in range(n):
        total += i * i

def main():
    num_threads = 4
    calculations_per_thread = 20_000_000
    threads = []

    start_time = time.time()

    for _ in range(num_threads):
        thread = threading.Thread(target=heavy_calculation, args=(calculations_per_thread,))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

    end_time = time.time()
    print(f"Execution time: {end_time - start_time:.2f} seconds")

if __name__ == "__main__":
    main()
```
