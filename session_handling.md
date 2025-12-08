# Handling user sessions

## Solution 1) Sticky sessions

When the user connects for the first time to your app, route him to a server with load balancing. Store his IP address and the server number so you can reroute him back to the same server every time he logs back in. Have a database to store info.

However, it doesn't scale up and introduce a SPOF.

## Solution 2) JWT ==> GOOD

Base64 encoded.
It has three components: headers, payload and signature. From the header and payload, you can check the signature.

Standard procedure:
- User gives name + password.
- Server creates JWT using oauth info.
- Send back signed JWT to user to include in future request.

Risk of being stolen. In order to avoid this, set an expiry time on the JWT to limit the damage.
