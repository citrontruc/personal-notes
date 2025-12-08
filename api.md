# API

## What does a reste API even mean?

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
