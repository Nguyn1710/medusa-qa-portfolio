# 🔴 Bug Report — BUG-001: Broken Access Control on Order Retrieval Endpoint

> **Document Type:** Bug Report (Deep-Dive)
> **Author:** Nguyen Le — QA Engineer (Entry Level)
> **Date Discovered:** August 2026
> **Test Phase:** API Smoke Testing — Newman / Postman

---

## 1. Bug Summary

| Field | Detail |
|---|---|
| **Bug ID** | BUG-001 |
| **Title** | Unauthenticated `GET /store/orders/{id}` Returns HTTP 200 with Full Order Data |
| **Severity** | 🔴 Critical |
| **Priority** | 🔴 High |
| **OWASP Category** | **A01:2021 — Broken Access Control** |
| **Module** | Storefront API — Order Retrieval |
| **Feature** | `GET /store/orders/{id}` endpoint |
| **Status** | Open (as of August 2026 — staging environment) |
| **Reported By** | Nguyen Le |
| **Jira Reference** | See also: Newman test `06 - Negative & Edge Cases > Get Order - No Auth` |

---

## 2. Environment

| Parameter | Value |
|---|---|
| **Application** | Medusa V2 — Storefront API |
| **Environment** | Staging |
| **Backend Host** | `https://backend-***.up.railway.app` |
| **Test Tool** | Postman / Newman CLI (`newman run` with htmlextra reporter) |
| **Report Reference** | [`docs/reports/newman-storefront-e2e-report.html`](./reports/newman-storefront-e2e-report.html) |
| **Affected Test** | Newman Collection: `06 - Negative & Edge Cases > Get Order - No Auth` |

---

## 3. Preconditions

1. At least one order exists in the system with a known `order_id` (e.g., obtained from a prior authenticated checkout flow in the same Newman collection run).
2. The request is made **without** an `Authorization` header and **without** any session cookie.
3. The Medusa V2 backend is deployed and reachable at the staging URL.

---

## 4. Steps to Reproduce

```
Step 1: Obtain a valid order_id
  - Method: Complete a checkout flow via the Storefront API (as done in earlier
    Newman collection steps: register → login → browse → cart → checkout)
  - Record the order_id returned in the checkout response

Step 2: Send an unauthenticated GET request to the order endpoint
  - Method: GET
  - URL: {{base_url}}/store/orders/{{order_id}}
  - Headers:
      Content-Type: application/json
      x-publishable-api-key: {{pub_key}}
      [NO Authorization header]
      [NO session cookie]
  - Body: (empty)

Step 3: Observe the HTTP response status code and response body
```

**Postman / Newman equivalent:**
```http
GET https://backend-***.up.railway.app/store/orders/order_01XXXXXXXXXXXXXXXX
Content-Type: application/json
x-publishable-api-key: pk_***
```
*(No `Authorization` or `Cookie` header included)*

---

## 5. Expected Result vs Actual Result

| | Detail |
|---|---|
| **Expected** | `HTTP 401 Unauthorized` — The server should reject any request that does not present a valid authenticated session for the customer who owns the order. No order data should be returned. |
| **Actual** | `HTTP 200 OK` — The server returns the complete order payload, including: customer name, shipping address, ordered items, prices, and order status. |

**Actual response (redacted):**
```json
HTTP/1.1 200 OK

{
  "order": {
    "id": "order_01XXXXXXXXXXXXXXXX",
    "status": "pending",
    "customer": {
      "email": "[customer email]",
      ...
    },
    "shipping_address": {
      "first_name": "[name]",
      "last_name": "[name]",
      "address_1": "[street address]",
      ...
    },
    "items": [...],
    "total": ...,
    ...
  }
}
```

---

## 6. Evidence

| Evidence | Description |
|---|---|
| **Newman HTML Report** | [`docs/reports/newman-storefront-e2e-report.html`](./reports/newman-storefront-e2e-report.html) — Interactive report showing 2 assertion failures |
| **Newman Screenshot** | [`screenshots/newman-report.png`](../screenshots/newman-report.png) — Dashboard view of the Newman run |
| **Assertion Failures** | 2 out of 55 total assertions failed — both in test `06 - Negative & Edge Cases > Get Order - No Auth` |

**Newman assertion that failed:**
```javascript
// Assertion 1: Status code check
pm.test("Status code is 401", function () {
    pm.response.to.have.status(401);
});
// → FAILED: Received 200

// Assertion 2: Body should not contain order data  
pm.test("Response should not contain order data", function () {
    pm.expect(pm.response.json()).to.not.have.property("order");
});
// → FAILED: Response contained full order object
```

---

## 7. Root Cause Analysis

> **Note:** This analysis is based on observable HTTP behavior only. No server-side code access was available during testing. Developer confirmation is recommended.

**Most likely root cause:**
The `GET /store/orders/{id}` route handler does **not enforce authentication** before returning order data. In Medusa V2's architecture, route-level authentication guards must be explicitly applied per endpoint. The most probable cause is one of the following:

| Hypothesis | Likelihood |
|---|---|
| Route middleware missing `authenticate` guard entirely | 🔴 High |
| Route uses `optional authentication` — if no token is present, it proceeds without auth check | 🟡 Medium |
| Auth guard is present but a misconfiguration causes it to be bypassed | 🟡 Medium |

**What is NOT the root cause:**
The issue is not in the Postman collection setup — the test explicitly omits all auth headers and the response confirms data is returned. This is reproducible.

**[DEV CONFIRMATION NEEDED]:** Please verify which middleware chain is applied to `GET /store/orders/:id` in the route definition and confirm whether the `authenticate` middleware is present and active.

---

## 8. Impact Analysis

| Impact Dimension | Assessment |
|---|---|
| **Confidentiality** | 🔴 **Critical** — Customer PII (name, address, items purchased) is exposed to unauthenticated parties |
| **Data Isolation** | 🔴 **Critical** — Any user knowing or guessing an order ID can retrieve another customer's order |
| **Attack Vector** | Order IDs in Medusa follow a predictable prefix pattern (`order_01...`). Sequential ID enumeration is feasible |
| **Regulatory** | Potential GDPR/data protection violation if customer data is accessible without authentication |
| **Business** | Reputational risk; potential legal liability for data exposure |
| **Exploitability** | 🔴 **Trivially exploitable** — No special tools needed; a simple unauthenticated GET request reproduces the issue |

**Attack scenario (theoretical, for illustration only):**
```
1. Attacker makes a test checkout to obtain one valid order_id format
2. Attacker writes a loop to enumerate order IDs sequentially
3. Each iteration retrieves a different customer's order with full PII
4. Result: mass data extraction of customer orders without any authentication
```

---

## 9. Fix Recommendation

**Immediate (Required before production deployment):**

Add server-side authentication middleware to the order retrieval route. The middleware must:
1. Verify that a valid session token or JWT is present in the request
2. Verify that the `customer_id` associated with the session matches the `customer_id` on the requested order
3. Return `HTTP 401 Unauthorized` if no valid auth is present
4. Return `HTTP 403 Forbidden` if auth is present but the customer does not own the requested order

**Example approach (pseudocode):**
```javascript
// Route: GET /store/orders/:id
router.get('/store/orders/:id', 
  authenticate(),         // ← MUST be present: validates session
  async (req, res) => {
    const order = await getOrder(req.params.id);
    
    // ← MUST be present: validates ownership
    if (order.customer_id !== req.session.customer_id) {
      return res.status(403).json({ message: 'Forbidden' });
    }
    
    return res.json({ order });
  }
);
```

**Secondary (Recommended):**
- Add an integration test that specifically verifies `GET /store/orders/{id}` returns 401 when called without auth — this should be a permanent regression test.
- Review all other order-related endpoints (`PATCH`, `POST` sub-resources) for the same missing auth guard pattern.

---

## 10. Lessons Learned — QA Perspective

*This section reflects personal learning from discovering and documenting this bug.*

**Why negative API testing found this, but functional testing might not have:**

During manual functional testing of the Admin Dashboard, authentication was always present — the tester was always logged in. This bug only surfaces when you specifically test what the system should **reject**, not what it should accept. Most test designers focus on the happy path and obvious failure modes (wrong password, invalid email). Testing *the absence of authentication on a data endpoint* is a deliberate, adversarial mindset shift.

**The value of explicit security assertions in automated tests:**

The Newman collection included an explicit test: *"Status code is 401"* for the unauthenticated order request. Without that assertion, the test would have passed (the request completed) even though the system was wide open. Writing security-specific assertions — not just "request completed successfully" but "request was rejected as expected" — is a critical discipline in API testing.

**OWASP as a test design checklist:**

Using OWASP Top 10 as a structured checklist during test design led directly to this discovery. A01 (Broken Access Control) specifically calls out "bypassing access control checks by modifying the URL" — which is exactly what this test did. Treating OWASP categories as test scenario categories, not just a reading list, turns security awareness into actionable test coverage.

**Documentation discipline:**

The bug was found through automated Newman execution, but the complete analysis (root cause hypotheses, impact scope, recommended fix) required structured thinking *after* the fact. Writing a full bug report like this one — rather than just a Jira ticket summary — forces clarity: What exactly happened? What could an attacker do with this? What needs to change to fix it? This discipline produces better developer handoffs and better portfolio artifacts.

---

*Bug report authored as part of the Medusa V2 QA Portfolio project by Nguyen Le.*
*See also: [Test Strategy](./test-strategy.md) | [Newman Report](./reports/newman-storefront-e2e-report.html)*
