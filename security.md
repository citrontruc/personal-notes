# Security

- [Security](#security)
  - [CIA Triad](#cia-triad)
  - [Store data securely](#store-data-securely)
  - [JWT](#jwt)

## CIA Triad

The CIA triad: fundamental model in cybersecurity. Guiding principal for data and application security.

- Confidentiality (do encryption at rest, check user access)
- Integrity (avoid data tempering, avoid ransomware)
- Availability (avoid DDoS and avoid downtime)

## Store data securely

Hash + salt + do repeat hashes so that it takes time to find even one password ==> Complicated to do brute force because it takes very long. Doing multiple hashes is called streching.

## JWT

A JWT token consists of three different elements: header with gives metadata about the token (security protocol used, type of token or key id), Payload contains the actual data (user id, name, mail...) and finally the third part is the signature obtained by concatenating information and encoding + a key.

JWT allows stateless authentication (you give all the necessary information for the authentication, no need to store more). Supports CORS (you give all the information). Problem is that it has long messages, complicated to revoke & refresh tokens.
