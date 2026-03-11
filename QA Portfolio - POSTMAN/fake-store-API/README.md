Fake Store API

**API Under Test:** [Fake Store API](https://fakestoreapi.com)  
**Environment:** FakeStore DEV  
**Date:** March 2026

---

Project Overview

This project is part of a structured QA automation portfolio built to demonstrate real-world API testing skills using Postman. It focuses on an e-commerce API, simulating the kind of testing relevant to retail and loyalty platforms (e.g. products, users, carts, authentication).

The goal was not just to make requests work, but to think like a QA engineer: validate business logic, handle edge cases, write meaningful assertions, and document findings professionally.

---

Collection Structure

```
Fake Store API
├── 📁 Products
│   ├── GET  get-all-products
│   ├── GET  get-single-product
│   ├── GET  get-products-by-category
│   └── GET  get-limited-products
├── 📁 Users
│   ├── GET  get-single-user
│   └── POST login-user
├── 📁 Carts
│   ├── GET  get-user-cart
│   └── POST add-new-cart
└── 📁 Negative Tests
    ├── GET  get-non-existent-product
    └── POST login-with-wrong-credentials
```

---

Test Run Summary

| Metric | Result |
|---|---|
| Total Tests | 30 |
| Passed | 30 |
| Failed | 0 |
| Avg. Response Time | 118ms |
| Environment | FakeStore DEV |

---

Test Coverage

Products
- Status code validation (200)
- Response time under 2000ms
- Response is a valid JSON array
- Array is not empty
- Product object contains required fields (`id`, `title`, `price`, `category`)
- Single product returns correct `id`
- Category filter returns only matching products
- Limit parameter returns correct number of results

Users & Authentication
- Login returns a token on valid credentials
- Token is a non-empty string
- POST requests correctly return `201 Created`

Carts
- Cart retrieval returns correct `userId`
- New cart creation returns `201 Created` with a cart `id`

Negative Tests
- Invalid product ID is handled gracefully (see Bug Report below)
- Login with wrong credentials returns `401 Unauthorized`
- Error response does not contain a token

---

🐛 Bug Report

 BUG-001 — Invalid Product ID Returns 200 with Empty Body

| Field | Detail |
|---|---|
| **Endpoint** | `GET /products/99999` |
| **Severity** | Medium |
| **Status** | Open (known API limitation) |

**Steps to Reproduce:**
1. Send `GET https://fakestoreapi.com/products/99999`
2. Observe response

**Expected Result:**  
`404 Not Found` with a descriptive JSON error message, e.g.:
```json
{
  "message": "Product not found"
}
```

**Actual Result:**  
`200 OK` with an empty body (`content-length: 0`)

**HTTP Response Headers (actual):**
```
HTTP/2.0 200 OK
content-type: application/json; charset=utf-8
content-length: 0
```

**Impact:**  
A client application consuming this API would receive a success status code with no data, leading to potential null reference errors or silent failures. This violates REST conventions where a missing resource should return `404`.

**Notes:**  
This is a known limitation of the Fake Store API mock server. In a production environment, this would be raised as a defect and prioritised for fix before release.

---

BUG-002 — Login Error Response Returns Plain Text Instead of JSON

| Field | Detail |
|---|---|
| **Endpoint** | `POST /auth/login` (invalid credentials) |
| **Severity** | Low |
| **Status** | Open (known API limitation) |

**Steps to Reproduce:**
1. Send `POST https://fakestoreapi.com/auth/login` with wrong username/password
2. Observe response format

**Expected Result:**  
`401 Unauthorized` with a structured JSON error:
```json
{
  "error": "username or password is incorrect"
}
```

**Actual Result:**  
`401 Unauthorized` with a plain text string:
```
username or password is incorrect
```

**Impact:**  
Inconsistent response format across the API. Clients expecting JSON will fail to parse the error response, potentially causing unhandled exceptions.

---

Key Learnings & QA Observations

- **POST requests return `201`, not `200`** — A common misconception. `201 Created` is the semantically correct HTTP status for resource creation. Tests should assert the right code, not just assume `200`.

- **Never assume error responses are JSON** — The login failure endpoint returns plain text. Always check the `Content-Type` header before calling `response.json()` in test scripts.

- **Debug with raw HTTP responses** — When a test behaves unexpectedly, inspecting the raw headers (e.g. `content-length: 0`) gives more reliable information than relying on the response body alone.

- **Document API defects, don't just skip them** — The non-existent product test was adjusted to match the API's actual (incorrect) behaviour, with the defect formally documented. This is standard practice in professional QA.

---

How to Run This Collection

 Postman Desktop (Manual)
1. Download and install [Postman](https://www.postman.com/downloads/)
2. Import `Fake Store API.postman_collection.json`
3. Import `FakeStore_DEV.postman_environment.json`
4. Select `FakeStore DEV` as the active environment
5. Click **Run Collection**

---

## 🌐 Environment Variables

| Variable | Value | Description |
|---|---|---|
| `baseUrl` | `https://fakestoreapi.com` | Base URL for all requests |

---

📁 Repository Structure

```
postman-qa-portfolio/
├── FakeStoreAPI.postman_collection.json
├── FakeStore_DEV.postman_environment.json
├── README.md
├──FakeStoreAPIResults.png
```

---

