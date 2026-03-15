# Cache

## Table of content

- [Cache](#cache)
  - [Table of content](#table-of-content)
  - [Things to take care of](#things-to-take-care-of)
    - [Thunder Hurd problem](#thunder-hurd-problem)
    - [Cache penetration](#cache-penetration)
    - [Cache breakdown](#cache-breakdown)
    - [Cache crash](#cache-crash)

## Things to take care of

Always have cache expiry.

### Thunder Hurd problem

When a lot of cache keys expire at the same time, you have to do an enormous amount of calls on your database at the same time.

### Cache penetration

If you ask for a non existent key, the cache won't find it and keep asking the database for the key without ever getting an answer.

### Cache breakdown

Some element with a hot key that is used everytime expires. Your cache sends a lot of calls to get the item back.

### Cache crash

Cache is down. You have to use the database.
