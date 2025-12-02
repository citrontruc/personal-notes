# API

## Patch vs put

Put remplace une valeur par une autre. PATCH remplace une partie de la valeur (juste quelques champs).

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
