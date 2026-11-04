---
aliases:
  - JWT
tags:
  - learning
  - dev/security
date: 2026-09-27
---
**Sources**: [JWT](https://www.jwt.io/)

**Related:** [[Security]]

---

## Description

_JSON Web Token_, an open standard (RFC 7519) used to securely transmit information between two parties as a compact, URL-safe JSON object.

---

## Key concepts

A JWT consists of three parts separated by dots (`.`):

- **Header:** Contains metadata about the token, including the token type (JWT) and the cryptographic signing algorithm used (like HMAC SHA-256 or RSA).

- **Payload:** Contains the "claims" or statements about an entity (typically user data, permissions, expiration time, and subject identifier).

- **Signature:** A cryptographic hash created by combining the encoded header, encoded payload, and a secret key, ensuring the data has not been altered in transit.

---

## Details

### Use Cases

- **Authentication:** When a user logs in, the server generates and signs a _JWT_ and sends it back to the client. Subsequent requests from the client include this token, verifying the user's identity without requiring the server to query a database for every request.

- **Authorization:** Once logged in, the token allows the user to access specific routes, services, and resources permitted by the claims encoded within the token.

- **Information Exchange:** Safely passing identity or business-process claims between different systems or microservices because the digital signature lets the receiver verify the sender's authenticity.

---

## Example

#### JWT
<p><span style="color: orange;">eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9</span>.<span style="color: purple;">eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0</span>.<span style="color: green;">KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30</span></p>

```json title:"Decoded Header"
{
  "alg": "HS256",
  "typ": "JWT"
}
```

```json title:"Decoded Payload"
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "iat": 1516239022
}
```

```plaintext title:"Signature Verification Secret"
a-string-secret-at-least-256-bits-long
```

---

## Claude Sessions
