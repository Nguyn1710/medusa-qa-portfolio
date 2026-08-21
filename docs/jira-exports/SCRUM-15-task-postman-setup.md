# SCRUM-15 — Task Export

> 🔗 **View on Jira Live:** [SCRUM-15 ↗](https://nguyen1710.atlassian.net/browse/SCRUM-15)
> 📌 **Exported:** August 2026 | **Project:** SCRUM — Medusa V2 QA

---

## Ticket Information

| Field | Value |
|---|---|
| **Ticket ID** | SCRUM-15 |
| **Type** | `Task` |
| **Summary** | Setup Postman automated tests for Auth APIs |
| **Priority** | 🟢 Low |
| **Status** | ✅ Done |
| **Reporter** | Nguyen Le |
| **Assignee** | Nguyen Le |
| **Sprint** | Sprint 1 — Auth & Session Module |
| **Story Points** | 2 |
| **Labels** | `api-testing`, `postman`, `newman`, `automation`, `smoke-test` |

---

## Description

Set up a Postman collection for API smoke testing of the Medusa V2 Storefront. The collection must cover the complete E2E checkout flow and include negative/edge case tests — particularly for unauthenticated access to protected endpoints.

The primary goal is to have a runnable, shareable Postman collection that can also be executed via Newman CLI for CI integration.

**Scope of the Postman collection:**
1. Register a new customer account
2. Login and capture the auth token
3. Browse products (storefront catalog)
4. Add item to cart / create cart
5. Update cart (shipping address, shipping option)
6. Complete checkout / place order
7. **Negative & Edge Cases** — unauthenticated requests to protected endpoints

---

## Acceptance Criteria

- [ ] **AC-1:** Postman collection created and importable (`.json` export format)
- [ ] **AC-2:** Collection covers all 22 planned API requests across the Storefront E2E checkout flow
- [ ] **AC-3:** At least 55 assertions are defined across all requests (status codes, response body validation)
- [ ] **AC-4:** Collection uses Postman Environment variables for `base_url`, `pub_key`, `auth_token`, `order_id`, etc.
- [ ] **AC-5:** Collection is runnable via Newman CLI: `newman run collection.json --environment env.json`
- [ ] **AC-6:** Newman htmlextra reporter is configured and generates an HTML report on completion
- [ ] **AC-7:** Negative test included: `GET /store/orders/{id}` without Authorization header — expected 401
- [ ] **AC-8:** Collection is executed at least once successfully before task is marked Done

---

## Implementation Notes

### Folder Structure in Postman Collection

```
📁 Medusa V2 — Storefront E2E
  ├── 01 - Customer Registration
  │     └── POST /store/customers
  ├── 02 - Auth Login
  │     └── POST /auth/token (capture token to env variable)
  ├── 03 - Browse Products
  │     └── GET /store/products
  ├── 04 - Cart Operations
  │     ├── POST /store/carts (create cart)
  │     ├── POST /store/carts/:id/line-items (add item)
  │     └── POST /store/carts/:id/shipping-methods
  ├── 05 - Checkout
  │     └── POST /store/carts/:id/complete
  ├── 06 - Negative & Edge Cases
  │     ├── GET /store/orders/:id [NO AUTH] ← BUG-001 discovered here
  │     └── [other negative cases]
  └── 07 - Cleanup (optional)
```

### Newman Run Command

```bash
newman run "Medusa-Storefront-E2E.postman_collection.json" \
  --environment "Medusa-Staging.postman_environment.json" \
  --reporters htmlextra \
  --reporter-htmlextra-export "docs/reports/newman-storefront-e2e-report.html"
```

### Environment Variables Used

| Variable | Description | Source |
|---|---|---|
| `base_url` | Backend API base URL | Static — Railway staging URL |
| `pub_key` | Publishable API key | Static — from Medusa admin settings |
| `auth_token` | Customer JWT from login | Dynamic — captured in test script |
| `cart_id` | Cart ID from create cart | Dynamic — captured in test script |
| `order_id` | Order ID from checkout | Dynamic — captured in test script |

---

## Outcome

| Metric | Value |
|---|---|
| Total Requests | 22 |
| Total Assertions | 55 |
| Assertions Passed | 53 |
| Assertions Failed | **2** — BUG-001 (unauthenticated order access) |
| Newman Report | [`docs/reports/newman-storefront-e2e-report.html`](../reports/newman-storefront-e2e-report.html) |

**Key finding from this task:** The negative test in folder `06 - Negative & Edge Cases` directly uncovered **BUG-001** — a Critical security vulnerability where `GET /store/orders/{id}` returns HTTP 200 without authentication.

This demonstrates the value of including **negative assertions** in API smoke tests, not just happy-path validation.

---

## Related Tickets

- **SCRUM-8** — Session persistence story (same sprint, auth domain)
- **SCRUM-12** — Session timeout bug (same sprint, auth domain)
- **BUG-001** — Security bug discovered through this task: [`docs/bug-report-BUG-001.md`](../bug-report-BUG-001.md)
