# Refactoring Code

## Table of content

- [Refactoring Code](#refactoring-code)
  - [Table of content](#table-of-content)
  - [How to proceed](#how-to-proceed)

## How to proceed

Audit the code by looking at how it works. Diagnose its most important flaws and treat them in this order.

- Start by identifying god objects or bloated functions. We have to deal with them first.
- Create logical entities to regroup values (record or classes). Rename variables to make things more explicit.
- Separate large pile of code in multiple smaller functions / classes.
- Use Interface Segregation & Dependency inversion. Identify the methods that look alike, create objects to put them inside and create unifying interfaces.

Start by fixing architecture before fixing the code.

Once the architecture is clearer, start with Open/Closed and Liskov substitution principle. Remove elements that are unnecessary and then elements that are not supposed to be here. You can then start adding tests.

Don't try to be too smart from the get go. Start by not changing to much and then fix elements one after another. When this is done, start fixing performances in the code.
