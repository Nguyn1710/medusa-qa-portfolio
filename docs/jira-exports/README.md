# 📌 Jira Tickets & Requirements Export

> 🔗 **Jira Live Board:** [Medusa Scrum Board & Backlog ↗](https://nguyen1710.atlassian.net/jira/software/projects/SCRUM/boards/1/backlog)

## 📑 Selected Ticket Samples

| Key | Type | Summary | Priority | Status | Links |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SCRUM-8** | `Story` | Admin Session Persistence & Auto-Redirect | `High` | Done | [Local Markdown](./SCRUM-8-user-story.md) \| [Jira Live ↗](https://nguyen1710.atlassian.net/browse/SCRUM-8) |
| **SCRUM-12** | `Bug` | Session timeout does not clear token on client | `Medium` | In Review | [Local Markdown](./SCRUM-12-bug-session-timeout.md) \| [Jira Live ↗](https://nguyen1710.atlassian.net/browse/SCRUM-12) |
| **SCRUM-15** | `Task` | Setup Postman automated tests for Auth APIs | `Low` | Done | [Local Markdown](./SCRUM-15-task-postman-setup.md) \| [Jira Live ↗](https://nguyen1710.atlassian.net/browse/SCRUM-15) |

---

## 🔗 Traceability Matrix

- **`SCRUM-8`** $\rightarrow$ Covered by test case `[MED_AUTH_TC_004]`. Bug confirmed: session redirect missing at `/app/login`.
- **`SCRUM-12`** $\rightarrow$ Detailed bug write-up available at [docs/bug-report-BUG-002-session-timeout.md](../bug-report-BUG-002-session-timeout.md).
- **`SCRUM-15`** $\rightarrow$ Newman run output: [docs/reports/newman-storefront-e2e-report.html](../reports/newman-storefront-e2e-report.html). Key finding: **BUG-001** discovered via this task — see [docs/bug-report-BUG-001.md](../bug-report-BUG-001.md).

---

> **Access Note:** 
> - `Local Markdown` links can be viewed directly within this repository.
> - `Jira Live` links require authorized access to the organization's Atlassian workspace.