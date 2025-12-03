# Memory

## How does it work

In C#, when you create an object of a reference type, the CLR (Common Language Runtime) allocates memory for the object on the managed heap. The variable that holds the reference to this object is stored on the stack.

When you pass a reference type to a method, you are actually passing the reference by default. This can lead to unintentional side effects if not managed properly.

## Garbage Collection

The fundamental operation of garbage collection in C# involves several key steps:

- **Marking**: The GC identifies which objects are still accessible from the root references. Root references include static variables, local variables, and CPU registers. During this phase, the GC traverses the object graph starting from these roots, marking all reachable objects.
- **Sweeping**: After marking, the garbage collector sweeps through the heap to identify unmarked objects, which are considered unreachable. These objects are eligible for collection.
- **Compacting**: To optimize memory usage, the GC may compact the heap by moving objects together, eliminating gaps left by collected objects. This compaction process helps in reducing fragmentation and allows for faster allocation of new objects.

## Dictionary

Interesting article: https://dotnetos.org/blog/2022-03-28-dictionary-implementation/

Actual implementation of a dictionary in dotnet: https://medium.com/@vosarat1995/how-c-dictionary-actually-works-47f3a156055b

### Data Structure

Dictionary has two main internal structures:
1) **Buckets**: an array of integers, each pointing to the index of the first entry in that bucket. Target bucket is found with (hashCode % buckets.Length) <== **THIS IS SHARDING**
2) **Entries**: an array of structs that actually hold the key, value, hash code, and a pointer to the next entry in case of a collision.

A single bucket can contain multiple entries if different keys have the same hash modulo array size. This is called chaining.

Average lookup is O(1), worst-case O(n) if all keys collide in the same bucket.

### Adding new elements

1) Compute the hash code of the key.
2) Find the target bucket (hashCode % buckets.Length).
3) Check existing entries in that bucket for equality (Equals() method).
4) If the key exists, update the value; if not, create a new entry and link it to the bucket chain.

GetHashCode() and Equals() are critical: poor hash distribution hurts performance

### Resizing

When the number of items exceeds a certain load factor (~0.75 of the array size), the dictionary resizes:
- Allocates a larger array for buckets and entries.
- Recalculates bucket indices for all entries because hashCode % newArray.Length changes.

This ensures lookups stay O(1) on average.

### Conclusion

- A Dictionary stores a set of buckets and entries.
- Index of a bucket — i - is calculated based on the hash code of the key.
- The value of the bucket[i] contains the index of a record in entries
- The matching entry either matches the key or has another entry in the links chain that does (or the key doesn’t exist in a dictionary)
- To find the actual value, Dictionary cycles by the links until it finds a matching key or stops at the dead-end (0 or -1 in next).
