# API Documentation (Swagger UI)

Serves the interactive Swagger UI and OpenAPI 3.1 specification for the `auth-service`.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/docs` or `/docs/`
- **Authentication**: None (Public)

---

## Request

### Headers
None required.

### Body Parameters
None.

---

## Response

### 200 OK
Returns the interactive Swagger UI web interface (`text/html`), allowing developers to browse endpoints, inspect schemas, and execute test requests directly in the browser.

---

## Curl Example

```bash
curl -i http://localhost:4001/docs/
```
