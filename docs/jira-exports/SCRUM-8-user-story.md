# SCRUM-8 — User Story Export

> 🔗 **View on Jira Live:** [SCRUM-8 ↗](https://nguyen1710.atlassian.net/browse/SCRUM-8)
> 📌 **Exported:** August 2026 | **Project:** SCRUM — Medusa V2 QA

---

## Ticket Information

| Field | Value |
|---|---|
| **Ticket ID** | SCRUM-8 |
| **Type** | `Story` |
| **Summary** | Admin Session Persistence & Auto-Redirect |
| **Priority** | 🔴 High |
| **Status** | ✅ Done |
| **Reporter** | Nguyen Le |
| **Assignee** | Nguyen Le |
| **Sprint** | Sprint 1 — Auth & Session Module |
| **Story Points** | 3 |
| **Labels** | `auth`, `session`, `regression` |
| **Linked Test Cases** | [`MED_AUTH_TC_004`] — *Redirect tự động đến /app/orders khi đã có session hợp lệ* |
| **Bug Found** | See [BUG-002](../bug-report-BUG-001.md#bug-002) — Session redirect not working |

---

## User Story

> **As an** authenticated admin user,
> **I want** the system to automatically redirect me to `/app/orders` when I navigate to `/app/login` with a valid session,
> **So that** I do not need to re-authenticate unnecessarily, and the system correctly respects my existing login state.

---

## Description

When an admin user is already authenticated (a valid session token is present in cookies or local storage) and manually navigates to `/app/login`, the system should recognize the active session and redirect the user to the main application view (`/app/orders`) without rendering the login form.

**Current observed behavior (Bug):**
The login page renders as if no session exists, even when a valid session token is present. The frontend does not read or act on the existing session state at route level.

**Expected behavior:**
- If session is valid → redirect to `/app/orders` before rendering the login component
- If session is expired or absent → render the login form normally

**Why this matters:**
1. Unnecessary re-authentication degrades user experience for admins
2. Inconsistent session state handling raises concerns about other session-dependent flows in the application

---

## Acceptance Criteria

- [ ] **AC-1:** Given an authenticated user (valid session in browser storage), when they navigate to `/app/login`, then they are automatically redirected to `/app/orders` without seeing the login form.
- [ ] **AC-2:** Given a user with no session or an expired session, when they navigate to `/app/login`, then the login form is displayed normally.
- [ ] **AC-3:** Given a user with an invalid/corrupted token, when they navigate to `/app/login`, then the invalid token is cleared and the login form is displayed.
- [ ] **AC-4:** The redirect must happen before the login form is rendered (no flash of login page before redirect).

---

## Test Coverage

| Test Case ID | Title | Result |
|---|---|---|
| `MED_AUTH_TC_004` | Redirect tự động đến /app/orders khi đã có session hợp lệ | 🔴 **Fail** — Bug confirmed |

**Bug confirmed during test execution:** Video evidence attached to this Jira ticket (MP4, 15MB). The login page renders visibly before any redirect occurs — and in this case, no redirect occurs at all.

---

## Notes

- This story was logged after manual test execution revealed the session guard is missing at the frontend route level.
- Proposed fix: Add a session guard component or route-level hook that checks for a valid auth token before mounting the login page component.
- Backend authentication itself is functional — the issue is purely a frontend routing concern.
