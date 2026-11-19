# API Documentation (Swagger UI) (`GET /docs`)

Serves the interactive Swagger UI and OpenAPI 3.1 specification for the `chat-service`.

---

## Route Overview

- **Method**: `GET`
- **Path**: `/docs` or `/docs/json`
- **Authentication**: None (Public)
- **Rate Limit**: None
- **Service**: `chat-service` (Port: `4003`)

---

## Request

### Headers
None required.

### Query Parameters
None.

### Body Parameters
None.

---

## Endpoints

### 1. Interactive Swagger UI (`GET /docs`)
Returns the interactive Swagger UI interface (`text/html`), allowing developers to inspect endpoints, review request/response schemas, and execute test requests directly from the browser.

### 2. OpenAPI 3.1 Specification (`GET /docs/json`)
Returns the complete OpenAPI 3.1 static schema document in JSON (`application/json`).

---

## Curl Example

```bash
# Browse UI
curl -i http://localhost:4003/docs/

# Fetch raw OpenAPI 3.1 schema
curl -i http://localhost:4003/docs/json
```
