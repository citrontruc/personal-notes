# Redis use cases

Redis is a database.

## Caching

Put in between of web server and database. Responses are cached on the redis hash data structure. Stored with cache keys.

If we find the answer in the cache, we have a cache hit, if not we have a cache miss. Every data in cache needs to have a timeout value.

When we have a cache miss, we can put the request in a queue. Requests are picked up by a query worker which makes requests and stores them back in redis.

Careful with interacting with databases and queues. We need to put in place locks.

## Session store

How it works:
- Session data is stored in the Redis hash data structure
- An expiry time is set for each user's data
- The expiry time gets renewed whenever the user requests something

It let them:
- Scale stateless web servers easily
- Handle traffic spikes

## rate limiting

Redis holds counter for number of requests allowed for each endpoint. Counter gets updated with each request.
