# API

## Table of content

- [API](#api)
  - [Table of content](#table-of-content)
  - [What does a rest API even mean?](#what-does-a-rest-api-even-mean)
  - [Patch vs put](#patch-vs-put)
  - [Idempotency](#idempotency)
  - [Status codes](#status-codes)
  - [Api Testing](#api-testing)

## What does a rest API even mean?

Client-Server with standardized request methods. Stateless, Uniform interfaces and paths, cacheable values (with additional intermediary needed), layered systems (load balancer, auth, db...). Maybe have an api gateway to handle this.

## Patch vs put

Put remplace une valeur par une autre. PATCH remplace une partie de la valeur (juste quelques champs).

## Idempotency

Risk: double billing if request failed when trying to reach server or when server failed to answer back. What to do: store idempotency key on the server side. If request is sent twice to the server, you won't get a double payment.

## Status codes

200 - Success
201 - created
202 - Accepted
204 - Not Content (happens mostly with patch when we changed a value)

300 - Multiple choices
301 - moved permanently. You get a new url
302 - temporary change
303 - Redirection

400 - bad request
401 - Unauthorized ( unauthenticated )
403 - Forbidden (we know you and it's a no)
404 - Not found
408 - Timeout
422 - Unprocessable entity (on échoue une validation business)
429 - too many requests

500 - Internal Server Error
501 - Not implemented
502 - Bad gateway
503 - Unavailable
504 - Gateway Timeout

## Api Testing

- Positive path testing: When doing something correct, you have a success code. Make sure your doocumentation specifies expected return codes. Verify payloads.
- Negative path testing: when doing something incorrect, we have a failure code. Test for values over thresholds (payload too large, malformed objects, use null and empty inputs...)
- Authentication & Authorization testing (test multiple users and expired tokens, test sensitive operations).
- Response time (simulate that for realistic operations, we have realistic execution time).
- Data validation (do it both for success and failure, test field level constraints (size & other)).
- Integration testing => Ordering a product updates stocks and launches the payment methods.
- Injection testing (sanitize requests).
