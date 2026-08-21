# 📌 Jira Tickets & Requirements Export

> 🔗 **Jira Live Board:** [Medusa Scrum Board & Backlog ↗](https://nguyen1710.atlassian.net/jira/software/projects/SCRUM/boards/1/backlog)

## 📑 Danh Sách Tickets Tiêu Biểu (Selected Sample)

| Key | Type | Summary | Priority | Status | Links |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SCRUM-8** | `Story` | Admin Session Persistence & Auto-Redirect | `High` | Done | [Local Markdown](./SCRUM-8-user-story.md) \| [Jira Live ↗](https://nguyen1710.atlassian.net/browse/SCRUM-8) |
| **SCRUM-12** | `Bug` | Session timeout does not clear token on client | `Medium` | In Review | [Local Markdown](./SCRUM-12-bug-session-timeout.md) \| [Jira Live ↗](https://nguyen1710.atlassian.net/browse/SCRUM-12) |
| **SCRUM-15** | `Task` | Setup Postman automated tests for Auth APIs | `Low` | Done | [Local Markdown](./SCRUM-15-task-postman-setup.md) \| [Jira Live ↗](https://nguyen1710.atlassian.net/browse/SCRUM-15) |

---

## 🔗 Ma Trận Đối Chiếu (Traceability Matrix)

- **`SCRUM-8`** $\rightarrow$ Được cover bởi test case `[MED_AUTH_TC_004]`. Lỗi xác nhận: thiếu session redirect tại `/app/login`.
- **`SCRUM-12`** $\rightarrow$ Tham chiếu chi tiết lỗi tại [docs/bug-report-BUG-002-session-timeout.md](../bug-report-BUG-002-session-timeout.md).
- **`SCRUM-15`** $\rightarrow$ Kết quả chạy Newman: [docs/reports/newman-storefront-e2e-report.html](../reports/newman-storefront-e2e-report.html). Phát hiện chính: **BUG-001** được tìm thấy qua task này — xem [docs/bug-report-BUG-001.md](../bug-report-BUG-001.md).

---

> **Lưu ý truy cập:** 
> - Các liên kết `Local Markdown` có thể xem trực tiếp trên GitHub repository.
> - Các liên kết `Jira Live` yêu cầu tài khoản được cấp quyền trong tổ chức Atlassian.