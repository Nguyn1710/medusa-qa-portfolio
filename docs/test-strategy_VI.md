# 🗺️ Chiến Lược Kiểm Thử — Dự Án QA Medusa V2

> **Loại tài liệu:** Chiến lược kiểm thử (Test Strategy)
> **Phiên bản:** 1.0
> **Tác giả:** Nguyen Le — Kỹ sư QA (Entry Level)
> **Ngày:** Tháng 8 năm 2026
> **Hệ thống kiểm thử (SUT):** Medusa V2 — Nền tảng Thương mại điện tử Headless
> **Môi trường:** Staging — Backend trên Railway, Storefront trên Vercel

---

## 1. Tổng Quan & Mục Đích

Tài liệu này định nghĩa **chiến lược kiểm thử dựa trên rủi ro (risk-based testing)** được áp dụng trong dự án QA Medusa V2. Nó mô tả phạm vi kiểm thử, lý do ưu tiên, kỹ thuật kiểm thử, công cụ sử dụng, và tiêu chí vào/ra (entry/exit) cùng tiêu chí Pass/Fail.

Mục tiêu chưa bao giờ là viết các test "xanh" (pass) — mục tiêu là **tìm ra vấn đề thực sự trước khi chúng lọt ra sản phẩm thực tế**. Mọi quyết định trong tài liệu này đều phản ánh mục tiêu đó.

---

## 2. Hệ Thống Kiểm Thử (SUT)

**Medusa V2** là một nền tảng thương mại điện tử headless mã nguồn mở cấp độ sản xuất. Môi trường staging được triển khai bao gồm:

| Thành phần | Triển khai | Mẫu URL |
|---|---|---|
| **Backend API** | Railway | `https://backend-***.up.railway.app` |
| **Admin Dashboard** | Cùng host | `https://backend-***.up.railway.app/app` |
| **Customer Storefront** | Vercel | *(triển khai riêng)* |

**Hai bề mặt kiểm thử được bao phủ:**
- **Admin Dashboard** — Xác thực, Quản lý sản phẩm, Quản lý đơn hàng, Tồn kho
- **Customer Storefront** — Duyệt sản phẩm, Tìm kiếm, Giỏ hàng, Thanh toán E2E, Quản lý tài khoản

---

## 3. Phạm Vi

### 3.1 Trong Phạm Vi

| Module | Vùng bao phủ |
|---|---|
| **Auth** | Happy path đăng nhập, đăng nhập tiêu cực (negative), quản lý phiên, redirect logic, khả năng chống injection |
| **Đơn hàng** | Tạo đơn hàng, đơn nháp (draft orders), chuyển đổi trạng thái đơn, edge cases |
| **Sản phẩm** | UI tạo sản phẩm, danh sách sản phẩm, kiểm tra hiển thị, edge cases |
| **Tồn kho** | Đồng bộ tồn kho, hiển thị tồn kho, quản lý tồn kho |
| **Storefront E2E** | Luồng thanh toán end-to-end (duyệt → giỏ hàng → thanh toán → xác nhận đơn), API smoke test |

### 3.2 Ngoài Phạm Vi

- Performance load testing / stress testing (ngoài kiểm tra SLA cơ bản)
- Kiểm thử ứng dụng mobile native (không có trong phạm vi)
- Kiểm thử tích hợp payment gateway trên môi trường production
- Tích hợp bên thứ ba ngoài Medusa core API
- Penetration testing đầy đủ — kiểm thử bảo mật chỉ giới hạn ở OWASP Top 10 spot-checks
- Unit test nội bộ của framework tự động hóa

---

## 4. Ưu Tiên Dựa Trên Rủi Ro

Không phải module nào cũng mang mức rủi ro như nhau. Nỗ lực kiểm thử được phân bổ dựa trên công thức:

**`Mức độ rủi ro = Mức độ ảnh hưởng đến nghiệp vụ × Khả năng xảy ra lỗi`**

| Module | Mức Rủi Ro | Ảnh Hưởng Nghiệp Vụ | Khả Năng Xảy Ra | Lý Do | Số Lượng TC |
|---|:---:|:---:|:---:|---|:---:|
| **Auth** | 🔴 Cao | Nghiêm trọng | Cao | Cổng vào toàn hệ thống; lỗi phiên ảnh hưởng đến mọi luồng; lỗ hổng bảo mật tập trung tại đây | 40 |
| **Đơn hàng** | 🔴 Cao | Nghiêm trọng | Trung bình | Luồng doanh thu cốt lõi; lỗi đồng nghĩa mất doanh số hoặc hỏng dữ liệu; logic đơn nháp phức tạp | 58 |
| **Sản phẩm** | 🟡 Trung bình | Trung bình | Trung bình | Dữ liệu thay đổi thường xuyên; lỗi hiển thị ảnh hưởng niềm tin khách hàng; UI drawer phức tạp | 59 |
| **Tồn kho** | 🟡 Trung bình | Trung bình | Thấp | Lỗi đồng bộ tồn kho gây bán vượt số lượng; độ phức tạp thấp hơn luồng đơn hàng | 53 |
| **Storefront E2E** | 🟡 Trung bình | Cao | Thấp | UX hướng đến khách hàng; rủi ro bỏ thanh toán; tầng API tạo thêm bề mặt tấn công | 50 |

> **Kết quả:** Auth và Đơn hàng nhận được phạm vi kiểm thử sâu nhất, bao gồm negative testing, boundary analysis, và security spot-checks. Điều này được xác nhận qua kết quả: Auth có mật độ lỗi cao nhất (17 bugs), Đơn hàng thứ hai (8 bugs).

---

## 5. Các Loại Kiểm Thử Được Áp Dụng

| Loại | Mục Đích | Áp Dụng Trong Dự Án Này |
|---|---|---|
| ✅ **Happy Path** | Xác minh các luồng người dùng cốt lõi thực thi thành công end-to-end | Đăng nhập với thông tin hợp lệ → redirect đến `/app/orders`; Luồng thanh toán hoàn chỉnh |
| ❌ **Kiểm thử tiêu cực** | Xác minh hệ thống từ chối input không hợp lệ một cách hợp lý và có thông báo rõ ràng | Đăng nhập sai mật khẩu, bỏ trống trường, định dạng email sai → hiển thị lỗi, không redirect |
| 🔲 **Kiểm thử biên (BVA)** | Gây áp lực lên hành vi tại các giá trị biên | Số lượng giỏ hàng = 0, giới hạn ký tự tối đa trên form, input rỗng vs một ký tự |
| 🔒 **Kiểm thử bảo mật** | Xác minh cơ chế bảo vệ xác thực, tách biệt dữ liệu, và kiểm soát truy cập | Request không xác thực đến endpoint được bảo vệ → phải trả về 401; xác thực tính liên tục của phiên |
| ⚡ **SLA Hiệu năng** | Xác thực thời gian phản hồi API nằm trong ngưỡng chấp nhận được | Mọi phản hồi API phải hoàn thành trong < 5000ms (xác thực qua Newman assertions) |
| 💉 **Kiểm thử Injection** | Kiểm tra khả năng chống chịu XSS và SQL injection trong các trường input do người dùng kiểm soát | `<script>alert(1)</script>` vào trường email; `' OR '1'='1` vào trường mật khẩu |

---

## 6. Phương Pháp & Kỹ Thuật Kiểm Thử

### 6.1 Kiểm Thử Chức Năng Thủ Công

**Phạm vi:** 260 test case thuộc 91 kịch bản, được tổ chức trong Excel (`Testcases.xlsx`).

**Kỹ thuật — Phân vùng tương đương (Equivalence Partitioning):**
Mỗi trường input được kiểm thử trên ba lớp tương đương:
- Input hợp lệ (dự kiến thành công)
- Input không hợp lệ — sai kiểu/định dạng (dự kiến bị từ chối với thông báo lỗi)
- Input biên — ở hoặc gần giới hạn của giá trị chấp nhận được

**Kỹ thuật — Kiểm thử bảng quyết định (Decision Table Testing):**
Áp dụng cho luồng đăng nhập nhiều điều kiện (định dạng email × tính hợp lệ mật khẩu × trạng thái phiên).

**Kỹ thuật — Kiểm thử chuyển đổi trạng thái (State Transition Testing):**
Áp dụng cho quản lý phiên: chưa xác thực → đã xác thực → phiên hết hạn → xác thực lại.

### 6.2 Kiểm Thử API Smoke (Newman/Postman)

**Phạm vi:** 22 request, 55 assertion trên toàn bộ luồng Storefront E2E Checkout.

**Phương pháp:**
- Happy path: đăng ký → đăng nhập → duyệt sản phẩm → thêm vào giỏ → thanh toán → xác nhận đơn
- Negative/edge cases: truy cập không xác thực đến endpoint được bảo vệ, payload request không hợp lệ

**Phát hiện chính:** 2 trong 55 assertion thất bại — cả hai đều liên quan đến **BUG-001** (unauthenticated order retrieval trả về HTTP 200).

Xem: [docs/bug-report-BUG-001.md](./bug-report-BUG-001.md)

### 6.3 Kiểm Thử Bảo Mật

**Tài liệu tham chiếu:** [OWASP Top 10 (2021)](https://owasp.org/Top10/)

| Danh mục OWASP | Kiểm thử Áp Dụng | Phát Hiện |
|---|---|---|
| **A01 — Broken Access Control** | GET `/store/orders/{id}` không có token xác thực | 🔴 **BUG-001 ĐÃ XÁC NHẬN** — Trả về 200 OK thay vì 401 |
| **A03 — Injection** | XSS payload vào trường email đăng nhập | ⚠️ Hệ thống đã sanitize input; không thực thi — nhưng không hiển thị thông báo lỗi validation |
| **A03 — Injection** | SQL injection vào trường mật khẩu | ⚠️ Hệ thống từ chối injection; không thực thi — nhưng không hiển thị thông báo lỗi validation |
| **A07 — Identification & Auth Failures** | Kiên tục phiên sau đăng xuất, truy cập với token cũ | 🟡 **BUG-002** — Session redirect không được thực thi tại `/app/login` |

---

## 7. Tiêu Chí Vào/Ra (Entry & Exit Criteria)

### 7.1 Tiêu Chí Vào (Testing bắt đầu khi:)

- [ ] Môi trường kiểm thử đã được triển khai và truy cập được (Railway backend + Admin dashboard)
- [ ] Tồn tại ít nhất một tài khoản admin hợp lệ và đang hoạt động
- [ ] Testcases.xlsx đã được hoàn thiện với các kịch bản và test case đã review
- [ ] Postman collection đã được import và biến môi trường đã cấu hình
- [ ] Newman CLI đã được cài đặt và có thể chạy được

### 7.2 Tiêu Chí Ra (Testing hoàn thành khi:)

- [ ] Tất cả 260 test case đã được thực thi (Pass hoặc Fail)
- [ ] Tất cả kết quả Fail đều có mục tương ứng trong Bug Log
- [ ] Tất cả bug mức Critical/High đã được báo cáo trên Jira
- [ ] Newman smoke test đã được thực thi và báo cáo đã được tạo
- [ ] Số liệu kiểm thử đã được tính toán và ghi vào sheet Dashboard

---

## 8. Tiêu Chí Pass / Fail

### 8.1 Cấp Độ Test Case

| Kết quả | Tiêu chí |
|---|---|
| **PASS** | Actual Result khớp chính xác với Expected Result (hoặc trong ngưỡng sai lệch chấp nhận được đã ghi nhận) |
| **FAIL** | Actual Result sai lệch so với Expected Result theo cách biểu hiện hành vi sai của hệ thống |
| **BLOCKED** | Test case không thể thực thi do phụ thuộc upstream hoặc vấn đề môi trường |

### 8.2 Phân Loại Mức Độ Nghiêm Trọng của Lỗi

| Mức Độ | Định Nghĩa | Ví Dụ Từ Dự Án Này |
|---|---|---|
| 🔴 **Critical** | Lỗ hổng bảo mật hoặc tính năng bị hỏng hoàn toàn không có workaround | BUG-001 — unauthenticated order access |
| 🟠 **High** | Tính năng cốt lõi bị hỏng; ảnh hưởng lớn đến người dùng | Luồng đăng nhập bị hỏng; lỗi tạo đơn hàng |
| 🟡 **Medium** | Tính năng bị hỏng một phần; tồn tại workaround | Bug session redirect (BUG-002); bất nhất hiển thị UI |
| 🟢 **Low** | Vấn đề thẩm mỹ hoặc UX nhỏ; không ảnh hưởng chức năng | Text bị cắt, text placeholder sai |

---

## 9. Công Cụ & Trách Nhiệm

| Công Cụ | Vai Trò | Phiên Bản Sử Dụng |
|---|---|---|
| **Microsoft Excel** | Quản lý test case — Testcases.xlsx | — |
| **Postman** | Soạn thảo API collection và kiểm thử API thủ công | — |
| **Newman CLI** | Trình chạy API smoke test tự động | — |
| **Newman htmlextra** | Tạo báo cáo HTML cho Newman | — |
| **Jira (Atlassian)** | Theo dõi lỗi và quản lý sprint (bảng SCRUM) | Cloud |
| **Java + Selenium WebDriver** | Framework tự động hóa UI | Java 11 / Selenium 4.23 |
| **REST Assured** | Framework tự động hóa API | 5.4 |
| **TestNG** | Trình chạy và phân nhóm test | 7.10 |
| **Allure** | Báo cáo HTML cho tự động hóa | 2.28 |
| **GitHub Actions** | Pipeline CI/CD cho bộ tự động hóa | — |

---

## 10. Giả Định & Ràng Buộc

| Loại | Mô Tả |
|---|---|
| **Giả định** | Môi trường staging phản ánh hành vi production cho các luồng được kiểm thử |
| **Giả định** | Thông tin đăng nhập admin ổn định trong suốt thời gian thực thi kiểm thử thủ công |
| **Giả định** | Mọi lỗi phát hiện trên staging đều tồn tại trên production trừ khi đã deploy fix |
| **Ràng buộc** | Không truy cập được server-side logs khi kiểm thử thủ công — phân tích nguyên nhân gốc rễ bị giới hạn ở HTTP responses quan sát được |
| **Ràng buộc** | Kiểm thử hiệu năng chỉ giới hạn ở xác thực SLA (thời gian phản hồi < 5000ms) — không phải full load testing |
| **Ràng buộc** | Kiểm thử bảo mật bị giới hạn ở OWASP spot-checks trên các bề mặt có thể truy cập — không phải full penetration test |

---

## 11. Rủi Ro & Biện Pháp Giảm Thiểu

| Rủi Ro | Khả Năng | Ảnh Hưởng | Biện Pháp Giảm Thiểu |
|---|---|---|---|
| Môi trường staging không ổn định | Trung bình | Cao | Thực thi lại các case thất bại để phân biệt flakiness môi trường với lỗi thực |
| Dữ liệu test bị nhiễm giữa các lần chạy | Trung bình | Trung bình | Dùng dữ liệu test duy nhất mỗi lần chạy; ghi nhận test data trong preconditions |
| Phạm vi bảo mật quá hẹp | Cao | Cao | Thừa nhận giới hạn rõ ràng; đề xuất review bảo mật chuyên dụng cho production |
| Thay đổi module Auth làm vô hiệu test case | Thấp | Cao | Duy trì test case trong Excel có version; re-validate sau bất kỳ thay đổi auth nào |

---

## 12. Báo Cáo & Theo Dõi Lỗi

Tất cả lỗi phát hiện trong quá trình kiểm thử thủ công đã được:
1. Ghi nhận vào sheet **Bug Log** của `Testcases.xlsx`
2. Báo cáo dưới dạng ticket trên [Jira SCRUM board](https://nguyen1710.atlassian.net/jira/software/projects/SCRUM/boards/1/backlog)
3. Các Jira export chính có sẵn tại [`docs/jira-exports/`](./jira-exports/)

**Tóm Tắt Lỗi (Cuối Cùng):**

| Module | Tổng Lỗi | Điểm Nổi Bật |
|---|:---:|---|
| Auth | 17 | Mật độ lỗi cao nhất |
| Đơn hàng | 8 | Edge cases đơn nháp |
| Sản phẩm | 7 | Lỗi hiển thị UI |
| Tồn kho | 3 | Độ trễ hiển thị đồng bộ tồn kho |
| Storefront | 4 | Bao gồm 1 lỗi bảo mật Critical |
| **Tổng** | **39** | 1 Critical, 38 Chức năng |

---

*Tài liệu chiến lược kiểm thử này được soạn thảo như một phần của dự án Medusa V2 QA Portfolio bởi Nguyen Le.*
*Chi tiết tự động hóa: xem [github.com/Nguyn1710/medusa-v2-automation](https://github.com/Nguyn1710/medusa-v2-automation)*
