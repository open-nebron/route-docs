# JSON Web Key Set (JWKS)

Exposes the public RSA cryptographic keys in standard JWKS (RFC 7517) format. Sibling microservices (`core-api`, `chat-service`, `voice-service`) cache and consume this endpoint to verify asymmetric RS256 access tokens without synchronous coupling to `auth-service`.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/.well-known/jwks.json`
- **Authentication**: None (Public)
- **Caching**: `Cache-Control: public, max-age=3600` (1 hour)

---

## Request

### Headers
None required.

### Body Parameters
None.

---

## Response

### 200 OK
Returns an array of public RSA keys, including the active signing key and any historical rotated public keys.

#### Response Headers
| Header | Value | Description |
|---|---|---|
| `Content-Type` | `application/json` | JSON format |
| `Cache-Control` | `public, max-age=3600` | Caches keys for 1 hour to prevent redundant HTTP requests |

#### Example Response Body
```json
{
  "keys": [
    {
      "kty": "RSA",
      "n": "u1wE7xQ9p...",
      "e": "AQAB",
      "kid": "dev-1",
      "use": "sig",
      "alg": "RS256"
    }
  ]
}
```

### JWK Property Definitions
- `kty`: Key type (`RSA`)
- `kid`: Key ID matching the `kid` header inside JWT tokens
- `use`: Intended key usage (`sig` for digital signatures)
- `alg`: Algorithm intended for use with the key (`RS256`)
- `n`: RSA modulus (Base64URL encoded)
- `e`: RSA exponent (Base64URL encoded, e.g. `AQAB`)

---

## Token Verification in Sibling Repositories

Sibling microservices verify access tokens locally using standard JWT libraries and this JWKS endpoint:

```typescript
import * as jose from 'jose';

// Automatically fetches and caches the JWKS from auth-service
const JWKS = jose.createRemoteJWKSet(
  new URL('http://auth-service:4001/.well-known/jwks.json')
);

export async function verifyAccessToken(token: string) {
  const { payload } = await jose.jwtVerify(token, JWKS, {
    issuer: 'zenuxs-auth',
    audience: 'core', // or 'chat', 'voice'
    algorithms: ['RS256'],
  });

  return {
    userId: payload.sub as string,
    email: payload.email as string,
    sessionId: payload.sid as string,
  };
}
```

---

## Curl Example

```bash
curl -i http://localhost:4001/.well-known/jwks.json
```
