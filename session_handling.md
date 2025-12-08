# Handling user sessions

## Solution 1) Sticky sessions

When the user connects for the first time to your app, route him to a server with load balancing. Store his IP address and the server number so you can reroute him back to the same server every time he logs back in. Have a database to store info.

However, it doesn't scale up and introduce a SPOF.

## Solution 2) JWT ==> GOOD

Base64 encoded.
