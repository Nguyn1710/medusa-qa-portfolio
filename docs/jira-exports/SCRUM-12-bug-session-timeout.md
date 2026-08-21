# SCRUM-12 — Bug Report Export

> 🔗 **View on Jira Live:** [SCRUM-12 ↗](https://nguyen1710.atlassian.net/browse/SCRUM-12)
> 📌 **Exported:** August 2026 | **Project:** SCRUM — Medusa V2 QA

---

## Ticket Information

| Field | Value |
|---|---|
| **Ticket ID** | SCRUM-12 |
| **Type** | `Bug` |
| **Summary** | Session timeout does not clear token on client |
| **Priority** | 🟡 Medium |
| **Status** | 🔄 In Review |
| **Reporter** | Nguyen Le |
| **Assignee** | [TODO: bổ sung khi assign] |
| **Sprint** | Sprint 1 — Auth & Session Module |
| **Labels** | `auth`, `session`, `security`, `token-management` |
| **Linked Test Cases** | Auth module — session expiry test cases |
| **Severity** | Medium |

---

## Bug Description

When a user's session expires on the server side (due to server-side token invalidation or TTL expiry), the client-side auth token stored in the browser (local storage or cookies) is **not cleared**. 

This means:
- The user's browser still holds a stale/invalid token
- On the next page navigation or app reload, the frontend may still attempt to use the expired token for requests
- The user may see authenticated-looking UI state (e.g., username displayed, sidebar visible) before being eventually rejected by the server

**This creates a confusing and potentially misleading UX where the user appears logged in but cannot perform any authenticated actions.**

---

## Steps to Reproduce

```
Step 1: Log in to the Admin Dashboard with valid credentials
  - Navigate to: https://backend-***.up.railway.app/app/login
  - Enter valid email and password
  - Confirm redirect to /app/orders

Step 2: Trigger server-side session expiry
  - Method A: Wait for the server-configured session TTL to elapse
  - Method B (quicker): Manually invalidate the session server-side
    (e.g., restart the auth service or clear the session store)

Step 3: Without refreshing the browser, attempt any authenticated action
  - Example: Navigate to /app/products or attempt to load an order

Step 4: Observe the client-side behavior
  - Check browser DevTools → Application → Local Storage / Cookies
  - Observe whether the auth token is still present
  - Observe what UI is rendered and what network response is returned
```

---

## Expected Result

When the server-side session expires:
1. The next authenticated request should receive `HTTP 401 Unauthorized` from the server
2. Upon receiving 401, the frontend should:
   - **Clear** the stale auth token from local storage / cookies
   - **Redirect** the user to `/app/login`
   - **Display** an appropriate message (e.g., "Your session has expired. Please log in again.")
3. The user should land on a clean login state with no residual authenticated UI

---

## Actual Result

1. The server correctly returns `HTTP 401` when the session is expired ✅
2. The frontend **does not** intercept the 401 response to clear the token ❌
3. The stale token remains in local storage / cookies ❌
4. The user may see a brief flash of authenticated UI before the error surfaces ❌
5. No "session expired" message is displayed — the failure mode is silent ❌

---

## Impact Analysis

| Dimension | Assessment |
|---|---|
| **User Experience** | Confusing — user sees authenticated UI but cannot perform actions |
| **Security** | Medium — stale tokens in storage could be extracted and reused if XSS were also present; on its own, less severe than BUG-001 |
| **Data Integrity** | Low — expired token requests are rejected server-side; no data leakage |
| **Frequency** | Medium — affects all users after session TTL elapses |

---

## Fix Recommendation

1. **Add a global 401 interceptor** in the frontend HTTP client (e.g., Axios interceptor or fetch wrapper):
   ```javascript
   // Example: Axios interceptor
   axios.interceptors.response.use(
     response => response,
     error => {
       if (error.response && error.response.status === 401) {
         // Clear token
         localStorage.removeItem('auth_token');
         document.cookie = 'session=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;';
         // Redirect to login
         window.location.href = '/app/login?reason=session_expired';
       }
       return Promise.reject(error);
     }
   );
   ```

2. **Display user-facing message:** On redirect, show "Your session has expired. Please log in again." rather than silently redirecting.

3. **Regression test:** Add an automated test that validates the 401 → clear token → redirect flow.

---

## Related Tickets

- **SCRUM-8** — Session redirect not working at `/app/login` (related session handling issue)
- **BUG-001** — See [`docs/bug-report-BUG-001.md`](../bug-report-BUG-001.md) for the Critical security bug in the same auth domain
