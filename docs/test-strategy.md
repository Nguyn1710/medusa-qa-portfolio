# 🗺️ Test Strategy — Medusa V2 QA Project

> **Document Type:** Test Strategy
> **Version:** 1.0
> **Author:** Nguyen Le — QA Engineer (Entry Level)
> **Date:** August 2026
> **System Under Test:** Medusa V2 — Headless E-Commerce Platform
> **Environment:** Staging — Backend on Railway, Storefront on Vercel

---

## 1. Overview & Purpose

This document defines the **risk-based test strategy** applied to the Medusa V2 QA project. It describes the scope of testing, rationale for prioritization, testing techniques used, tools employed, and the criteria for entry, exit, and pass/fail evaluation.

The goal of this strategy was never to write passing tests — it was to **find real issues before they reach production**. Every decision in this document reflects that objective.

---

## 2. System Under Test (SUT)

**Medusa V2** is a production-grade open-source headless e-commerce platform. The deployed staging environment consisted of:

| Component | Deployment | URL Pattern |
|---|---|---|
| **Backend API** | Railway | `https://backend-***.up.railway.app` |
| **Admin Dashboard** | Same host | `https://backend-***.up.railway.app/app` |
| **Customer Storefront** | Vercel | *(separate deployment)* |

**Two testing surfaces were covered:**
- **Admin Dashboard** — Authentication, Product Management, Order Management, Inventory
- **Customer Storefront** — Browse, Search, Cart, End-to-End Checkout, Account Management

---

## 3. Scope

### 3.1 In Scope

| Module | Coverage Areas |
|---|---|
| **Auth** | Login happy path, negative login, session management, redirect logic, injection resistance |
| **Orders** | Order creation, draft orders, order status transitions, edge cases |
| **Products** | Product creation UI, product listing, display validation, edge cases |
| **Inventory** | Stock sync, inventory display, stock management |
| **Storefront E2E** | End-to-end checkout flow (browse → cart → checkout → order confirmation), API smoke test |

### 3.2 Out of Scope

- Performance load testing / stress testing (beyond basic SLA validation)
- Mobile native app testing (no mobile app in scope)
- Payment gateway integration testing in production environment
- Third-party integrations beyond the Medusa core API
- Penetration testing (full pen test scope) — security testing was limited to OWASP Top 10 spot-checks on input fields and authentication endpoints
- Automation framework internal unit tests

---

## 4. Risk-Based Prioritization

Not all modules carry equal risk. Test effort was distributed based on the formula:

**`Risk Level = Business Impact × Likelihood of Failure`**

| Module | Risk Level | Business Impact | Likelihood | Rationale | TC Count |
|---|:---:|:---:|:---:|---|:---:|
| **Auth** | 🔴 High | Critical | High | Entry gate to entire system; session bugs cascade to all flows; security vulnerabilities concentrate here | 40 |
| **Orders** | 🔴 High | Critical | Medium | Core revenue flow; errors mean lost sales or data corruption; draft order logic is complex | 58 |
| **Products** | 🟡 Medium | Medium | Medium | Frequent data changes; display bugs damage customer trust; complex UI drawer interactions | 59 |
| **Inventory** | 🟡 Medium | Medium | Low | Stock sync errors cause overselling; lower complexity than order flows | 53 |
| **Storefront E2E** | 🟡 Medium | High | Low | Customer-facing UX; checkout abandonment risk; API layer adds attack surface | 50 |

> **Result:** Auth and Orders received the deepest test coverage, including negative testing, boundary analysis, and security spot-checks. This was validated by outcomes: Auth had the highest defect density (17 bugs), Orders second (8 bugs).

---

## 5. Test Types Applied

| Type | Purpose | Application in This Project |
|---|---|---|
| ✅ **Happy Path** | Verify core user journeys execute successfully end-to-end | Login with valid credentials → redirect to `/app/orders`; Full checkout flow completion |
| ❌ **Negative Testing** | Verify the system rejects invalid input gracefully and informatively | Login with wrong password, empty fields, invalid email formats → expect error message, no redirect |
| 🔲 **Boundary Value Analysis** | Stress-test behavior at edge values | Cart quantity = 0, maximum character limits on form fields, empty vs single-character inputs |
| 🔒 **Security Testing** | Verify auth guards, data isolation, and access control | Unauthenticated requests to protected endpoints → must return 401; session persistence validation |
| ⚡ **Performance SLA** | Validate API response times stay within acceptable thresholds | All API responses must complete in < 5000ms (validated via Newman assertions) |
| 💉 **Injection Testing** | Check resistance to XSS and SQL injection in user-controlled input fields | `<script>alert(1)</script>` into email field; `' OR '1'='1` into password field |

---

## 6. Testing Approaches & Techniques

### 6.1 Manual Functional Testing

**Scope:** 260 test cases across 91 scenarios, organized in Excel (`Testcases.xlsx`).

**Technique — Equivalence Partitioning:**
Each input field was tested across three equivalence classes:
- Valid inputs (expected to succeed)
- Invalid inputs — wrong type/format (expected to be rejected with error)
- Boundary inputs — at or near the limits of acceptable values

**Technique — Decision Table Testing:**
Applied for multi-condition login flows (email format × password validity × session state).

**Technique — State Transition Testing:**
Applied for session management: unauthenticated → authenticated → session expired → reauthenticated.

### 6.2 API Smoke Testing (Newman/Postman)

**Scope:** 22 requests, 55 assertions across the full Storefront E2E Checkout Flow.

**Approach:**
- Happy path: register → login → browse → add to cart → checkout → confirm order
- Negative/edge cases: unauthenticated access to protected endpoints, invalid request payloads

**Key Finding:** 2 of 55 assertions failed — both mapping to **BUG-001** (unauthenticated order retrieval returning HTTP 200).

See: [docs/bug-report-BUG-001.md](./bug-report-BUG-001.md)

### 6.3 Security Testing

**Framework Reference:** [OWASP Top 10 (2021)](https://owasp.org/Top10/)

| OWASP Category | Test Applied | Finding |
|---|---|---|
| **A01 — Broken Access Control** | GET `/store/orders/{id}` without auth token | 🔴 **BUG-001 CONFIRMED** — 200 OK returned instead of 401 |
| **A03 — Injection** | XSS payload in login email field | ⚠️ System sanitized input; no execution — but no validation error message shown |
| **A03 — Injection** | SQL injection in password field | ⚠️ System rejected injection; no execution — but no validation error message shown |
| **A07 — Identification & Auth Failures** | Session persistence after logout, access with stale token | 🟡 **BUG-002** — session redirect not enforced at `/app/login` |

---

## 7. Entry & Exit Criteria

### 7.1 Entry Criteria (Testing begins when:)

- [ ] Test environment is deployed and accessible (Railway backend + Admin dashboard)
- [ ] At least one valid admin account exists and is confirmed active
- [ ] Testcases.xlsx is finalized with scenarios and test cases reviewed
- [ ] Postman collection is imported and environment variables are configured
- [ ] Newman CLI is installed and executable

### 7.2 Exit Criteria (Testing is complete when:)

- [ ] All 260 planned test cases have been executed (Pass or Fail)
- [ ] All Fail results have a corresponding Bug Log entry
- [ ] All Critical/High severity bugs have been reported in Jira
- [ ] Newman smoke test has been executed and report is generated
- [ ] Test metrics have been calculated and recorded in the Dashboard sheet

---

## 8. Pass / Fail Criteria

### 8.1 Test Case Level

| Verdict | Criteria |
|---|---|
| **PASS** | Actual Result matches Expected Result exactly (or acceptably within documented tolerance) |
| **FAIL** | Actual Result deviates from Expected Result in a way that represents incorrect system behavior |
| **BLOCKED** | Test case cannot be executed due to an upstream dependency or environment issue |

### 8.2 Severity Classification for Defects

| Severity | Definition | Examples from This Project |
|---|---|---|
| 🔴 **Critical** | Security vulnerability or complete feature failure with no workaround | BUG-001 — unauthenticated order access |
| 🟠 **High** | Core feature broken; significant user impact | Login flow broken; order creation failure |
| 🟡 **Medium** | Feature partially broken; workaround exists | Session redirect bug (BUG-002); UI display inconsistency |
| 🟢 **Low** | Minor cosmetic or UX issue; no functional impact | Text truncation, incorrect placeholder text |

---

## 9. Tools & Responsibilities

| Tool | Role | Version Used |
|---|---|---|
| **Microsoft Excel** | Test case management — Testcases.xlsx | — |
| **Postman** | API collection authoring and manual API testing | — |
| **Newman CLI** | Automated API smoke test runner | — |
| **Newman htmlextra** | HTML report generation for Newman runs | — |
| **Jira (Atlassian)** | Bug tracking and sprint management (SCRUM board) | Cloud |
| **Java + Selenium WebDriver** | UI automation framework | Java 11 / Selenium 4.23 |
| **REST Assured** | API automation framework | 5.4 |
| **TestNG** | Test runner and grouping | 7.10 |
| **Allure** | Automation HTML reporting | 2.28 |
| **GitHub Actions** | CI/CD pipeline for automation suite | — |

---

## 10. Assumptions & Constraints

| Type | Description |
|---|---|
| **Assumption** | The staging environment is representative of production behavior for the flows under test |
| **Assumption** | Admin credentials remain stable throughout the manual test execution period |
| **Assumption** | All defects found in staging are also present in production unless a fix is deployed |
| **Constraint** | No access to server-side logs during manual testing — root cause analysis was limited to observable HTTP responses |
| **Constraint** | Performance testing was scoped to SLA validation only (response time < 5000ms) — not full load testing |
| **Constraint** | Security testing was limited to OWASP spot-checks on accessible surfaces — not a full penetration test |

---

## 11. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Staging environment instability | Medium | High | Re-execute failed cases to distinguish environment flakiness from real defects |
| Test data contamination between runs | Medium | Medium | Use unique test data per run; document test data in preconditions |
| Security scope too narrow | High | High | Acknowledge limitation explicitly; recommend dedicated security review for production |
| Auth module changes invalidating test cases | Low | High | Maintain test cases in versioned Excel; re-validate after any auth changes |

---

## 12. Defect Reporting & Tracking

All defects discovered during manual testing were:
1. Logged in the **Bug Log** sheet of `Testcases.xlsx`
2. Reported as tickets on the [Jira SCRUM board](https://nguyen1710.atlassian.net/jira/software/projects/SCRUM/boards/1/backlog)
3. Key Jira exports available in [`docs/jira-exports/`](./jira-exports/)

**Bug Summary (Final):**

| Module | Total Bugs | Severity Highlight |
|---|:---:|---|
| Auth | 17 | Highest defect density |
| Orders | 8 | Draft order edge cases |
| Products | 7 | UI display bugs |
| Inventory | 3 | Stock sync display lag |
| Storefront | 4 | Includes 1 Critical security bug |
| **Total** | **39** | 1 Critical, 38 Functional |

---

*This test strategy was authored as part of the Medusa V2 QA Portfolio project by Nguyen Le.*
*Automation details: see [github.com/Nguyn1710/medusa-v2-automation](https://github.com/Nguyn1710/medusa-v2-automation)*
