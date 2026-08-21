# 🟡 Bug Report — BUG-002: Session Timeout Does Not Clear Client Token / Form Data Loss

> **Document Type:** Bug Report (Deep-Dive)
> **Author:** Nguyen Le — QA Engineer (Entry Level)
> **Date Discovered:** August 2026
> **Test Phase:** Manual Functional Testing — Auth & Product Modules
> **Jira Reference:** [SCRUM-12](https://nguyen1710.atlassian.net/browse/SCRUM-12)

---

## 1. Bug Summary

| Field | Detail |
|---|---|
| **Bug ID** | BUG-002 |
| **Title** | Session timeout does not clear token on client; unsaved product form data lost without warning |
| **Severity** | 🟡 Medium |
| **Priority** | 🟡 Medium |
| **Module** | Admin — Authentication / Session Management; Admin — Product Creation Form |
| **Feature** | Session expiry handling; client-side token cleanup; form state persistence |
| **Status** | In Review (as of August 2026 — staging environment) |
| **Reported By** | Nguyen Le |
| **Jira Ticket** | [SCRUM-12](https://nguyen1710.atlassian.net/browse/SCRUM-12) |
| **Linked Test Cases** | `MED_PROD_TC_064` (Session timeout during product form), `MED_ORD_TC_059` (Session timeout during Draft Order form) |

---

## 2. Environment

| Parameter | Value |
|---|---|
| **Application** | Medusa V2 — Admin Dashboard |
| **Environment** | Staging |
| **URL** | `https://backend-***.up.railway.app/app` |
| **Test Tool** | Manual browser testing (incognito mode) |
| **Browsers Tested** | Chrome (latest) |

---

## 3. Preconditions

1. User is authenticated (valid session token present in browser local storage / cookies).
2. User is actively filling out a form with unsaved data (e.g., product creation form or draft order form).
3. The server-side session TTL is configured to expire after a fixed period of inactivity.

---

## 4. Steps to Reproduce

```
Step 1: Log in to Admin Dashboard
  - Navigate to: https://backend-***.up.railway.app/app/login
  - Enter valid credentials → confirm redirect to /app/orders

Step 2: Navigate to the Product Creation form
  - Go to Products → Click "New Product"
  - Begin filling out the form: enter Title, Description, add Variants, etc.
  - Do NOT publish or save at this point

Step 3: Trigger session expiry (without page refresh)
  - Method: Wait for the server-configured session TTL to elapse
    (or manually invalidate the session via server-side action)

Step 4: Continue interacting with the form or attempt to navigate
  - Try navigating to another admin page (e.g., /app/orders)
  - OR simply wait and observe the UI

Step 5: Observe what happens to:
  (a) The auth token in browser storage
  (b) The form data that was entered but not saved
  (c) Any user-facing message or redirect behavior
```

---

## 5. Expected Result vs Actual Result

### Expected Result

| # | Expected Behavior |
|---|---|
| 1 | When session expires, the next API call from the frontend receives `HTTP 401 Unauthorized` |
| 2 | The frontend intercepts the 401 and **clears** the stale auth token from local storage / cookies |
| 3 | The user is **redirected to `/app/login`** with a clear message: *"Your session has expired. Please log in again."* |
| 4 | Before redirecting (or at least before session expires), the system should **warn the user** about unsaved changes in the form |
| 5 | The form data entered by the user should either be preserved via auto-save or the user should be explicitly warned that data will be lost |

### Actual Result

| # | Actual Behavior |
|---|---|
| 1 | Server correctly returns `HTTP 401` on expired session ✅ |
| 2 | The frontend does **not** intercept the 401 to clear the stale token ❌ |
| 3 | The stale token **remains** in local storage after session expiry ❌ |
| 4 | When session expires during product form filling, the system **redirects to login** — but with **no prior warning** about unsaved data ❌ |
| 5 | All form data (title, description, variants, pricing entered so far) is **lost permanently** upon redirect ❌ |
| 6 | No "session expired" message is displayed — the session failure mode is **silent** ❌ |

**Summary of confirmed defects in this ticket:**
- **Defect A:** Stale token not cleared on 401 response
- **Defect B:** No session expiry warning to user
- **Defect C:** Unsaved form data destroyed without user consent or backup

---

## 6. Evidence

| Evidence | Description |
|---|---|
| **Jira Ticket** | [SCRUM-12 ↗](https://nguyen1710.atlassian.net/browse/SCRUM-12) — Full details and internal discussion |
| **Linked Test Cases** | `MED_PROD_TC_064` (Fail), `MED_ORD_TC_059` (Fail) — both confirmed this defect pattern |
| **Test Case Result** | `MED_PROD_TC_064`: Actual Result — "When session expires while filling the product creation form, the system redirects to login without saving a draft; all entered form data is lost with no prior warning displayed." |

---

## 7. Root Cause Analysis

> **Note:** Analysis based on observable behavior only. No server-side code access was available. Developer confirmation is recommended for the exact root cause.

**Defect A — Stale token not cleared:**
The frontend HTTP client (likely Axios or fetch) does not have a global response interceptor for `401 Unauthorized`. When the server rejects a request with 401, the rejection propagates as a JavaScript error but no cleanup logic executes. The token lingers in storage until manual logout or browser cache clear.

**Defect B — No session expiry warning:**
The frontend does not implement a session activity timer (e.g., a `setTimeout` based on the server-configured TTL). Without proactive monitoring, the only signal of session expiry is a failed API request — by which point data loss has already occurred.

**Defect C — Form data loss:**
The product creation form does not implement:
- Auto-save to a draft state on a periodic interval
- Unsaved changes detection (standard `beforeunload` event pattern)
- LocalStorage fallback for in-progress form state

**[DEV CONFIRMATION NEEDED]:**
1. What is the configured session TTL on the server?
2. Is there a global 401 interceptor in the frontend HTTP client? If yes, is it currently active for admin routes?
3. Does the product form have any draft auto-save mechanism in its current implementation?

---

## 8. Impact Analysis

| Impact Dimension | Assessment |
|---|---|
| **User Experience** | 🔴 High — A user can lose substantial work (complex product with many variants, pricing, images) with no warning. This is particularly damaging for large catalog operations. |
| **Security** | 🟡 Medium — Stale token in storage is a secondary security concern. On its own it is less severe; combined with XSS vulnerability it would be more serious. |
| **Data Integrity** | 🟡 Medium — No data corruption (no incorrect data written), but user work product is destroyed unexpectedly. |
| **Trust** | 🔴 High — Unexpected data loss without warning significantly erodes trust in the admin tool reliability. |
| **Frequency** | 🟡 Medium — Affects all admin users in long editing sessions (product catalog management is inherently time-consuming). |

---

## 9. Fix Recommendation

### Fix for Defect A — Clear stale token on 401

Add a global response interceptor in the frontend HTTP client:

```javascript
// Example: Axios interceptor
axios.interceptors.response.use(
  response => response,
  error => {
    if (error.response && error.response.status === 401) {
      // 1. Clear auth token
      localStorage.removeItem('_medusa_jwt');
      document.cookie = 'connect.sid=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;';
      // 2. Redirect with reason
      window.location.href = '/app/login?reason=session_expired';
    }
    return Promise.reject(error);
  }
);
```

### Fix for Defect B — Session expiry warning

Implement a proactive session timer that warns the user before expiry:

```javascript
// Warn user 2 minutes before session expires
const SESSION_TTL_MS = /* get from config */;
const WARN_BEFORE_MS = 2 * 60 * 1000; // 2 minutes

setTimeout(() => {
  showDialog("Your session will expire in 2 minutes. Please save your work.");
}, SESSION_TTL_MS - WARN_BEFORE_MS);
```

### Fix for Defect C — Unsaved form data protection

1. **Short-term:** Add a `beforeunload` event listener on the product form that warns users about unsaved changes:
   ```javascript
   window.addEventListener('beforeunload', (e) => {
     if (hasUnsavedChanges()) {
       e.preventDefault();
       e.returnValue = '';
     }
   });
   ```

2. **Long-term:** Implement periodic auto-save to draft (e.g., every 60 seconds), storing form state in local storage or server draft endpoint.

---

## 10. Lessons Learned — QA Perspective

*This section reflects personal learning from discovering and documenting this bug.*

**Forms are the most vulnerable surface for session-related data loss:**

During risk assessment, the Auth module was correctly identified as High risk. But the compounding effect of session management bugs on *other* features (like the product form) was underestimated. When a session bug causes data loss in a completely different module (Products), it demonstrates that session handling is a cross-cutting concern — its defects do not stay contained to the Auth module.

**The difference between server behavior and client behavior:**

The server is doing its job correctly: it expires sessions and returns 401. The bug is entirely in how the *client* responds to that signal. This distinction matters for bug reporting: the backend team may read "session timeout bug" and think their code is wrong. A precise report — "server-side behavior is correct; client-side 401 handler is missing" — prevents misdirected effort and accelerates resolution.

**Testing long-running sessions requires deliberate setup:**

Session timeout bugs are invisible in typical functional test runs, where sessions are fresh and short-lived. Discovering this defect required deliberately triggering an expired session — a step that must be explicitly designed into the test plan. This reinforces the value of including session state transitions as explicit test scenarios, not just "login works / logout works."

**Data loss bugs carry disproportionate user impact:**

A medium-severity bug by strict definition (no security vulnerability, no data corruption, workaround available) can feel Critical to the user who just spent 30 minutes building a complex product listing and lost it all. Severity classification should always be cross-checked against the realistic user impact scenario, not just the technical category.

---

*Bug report authored as part of the Medusa V2 QA Portfolio project by Nguyen Le.*
*See also: [BUG-001 (Security)](./bug-report-BUG-001.md) | [Test Strategy](./test-strategy.md) | [SCRUM-12 Jira Export](./jira-exports/SCRUM-12-bug-session-timeout.md)*
