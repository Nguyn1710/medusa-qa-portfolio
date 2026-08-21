<p align="center">
  <img src="https://img.shields.io/badge/Manual%20Testing-260%20Test%20Cases-blue?style=for-the-badge" alt="Manual Testing"/>
  <img src="https://img.shields.io/badge/Bugs%20Found-39%20Confirmed-red?style=for-the-badge" alt="Bugs Found"/>
  <img src="https://img.shields.io/badge/Security%20Bug-1%20Critical-darkred?style=for-the-badge" alt="Security Bug"/>
  <img src="https://img.shields.io/badge/Automation-163%20Tests-43B02A?style=for-the-badge" alt="Automation"/>
  <img src="https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white" alt="CI/CD"/>
</p>

# 🧪 Medusa V2 — Hồ Sơ QA & Nghiên Cứu Tình Huống Kiểm Thử

> **Đây là tài liệu hồ sơ chính** cho một dự án QA trọn vòng đời trên [Medusa V2](https://medusajs.com/), một nền tảng thương mại điện tử headless mã nguồn mở.
>
> Dự án bao trùm **toàn bộ vòng đời QA** — từ chiến lược kiểm thử và thiết kế test case thủ công, đến báo cáo lỗi (bug), kiểm thử API với Postman/Newman, và một framework tự động hóa đầy đủ với Selenium + REST Assured cùng CI/CD.

> Mục tiêu chưa bao giờ là viết ra các test "xanh" (pass). Mục tiêu là **tìm ra những vấn đề thực sự** trước khi chúng lọt ra sản phẩm thực tế.

---

## 📁 Cấu Trúc Repository

Dự án này được chia thành hai repository — có chủ đích, nhằm mô phỏng cách tài liệu QA và mã tự động hóa được quản lý riêng biệt trong các nhóm thực tế.

| Repository | Nội Dung Bên Trong |
|---|---|
| 📋 **[`medusa-qa-portfolio`](https://github.com/Nguyn1710/medusa-qa-portfolio)**| Chiến lược kiểm thử, Test case thủ công (Excel), Báo cáo lỗi (Jira), Số liệu kiểm thử & Dashboard |
| 🤖 **[`medusa-v2-automation`](https://github.com/Nguyn1710/medusa-v2-automation)** | Framework tự động hóa Java — Selenium UI + REST Assured API + TestNG + Allure + CI/CD |

---

## 🎯 Phạm Vi & Mục Tiêu Dự Án

**Hệ thống kiểm thử (SUT):** Medusa V2 — một nền tảng thương mại điện tử headless cấp độ sản xuất, được triển khai trên Railway (backend) và Vercel (storefront).

**Kiểm thử bao phủ hai bề mặt:**

- **Admin Dashboard** — xác thực, quản lý sản phẩm, quản lý đơn hàng, tồn kho
- **Customer Storefront** — duyệt sản phẩm, tìm kiếm, giỏ hàng, thanh toán end-to-end, quản lý tài khoản

**Những câu hỏi mà dự án này đặt ra để trả lời:**
1. Hệ thống có hoạt động đúng với tất cả các yêu cầu chức năng đã tài liệu hóa không?
2. Có những trường hợp biên (edge case) hay luồng tiêu cực (negative path) nào phá vỡ hành vi mong đợi không?
3. Có lỗ hổng bảo mật nào có thể làm lộ dữ liệu khách hàng hoặc bỏ qua kiểm soát truy cập không?
4. Chúng ta có thể xây dựng một tầng tự động hóa dễ bảo trì để tự động bắt lỗi hồi quy (regression) không?

---

## 📊 Tổng Quan Kiểm Thử

| Tầng | Phương Pháp | Khối Lượng | Kết Quả |
|---|---|---|---|
| 📋 Thiết kế Test | Thủ công — dựa trên Excel | 91 Kịch bản → 260 Test Case | 221 Pass / **39 Fail** |
| 🐛 Theo dõi lỗi | Jira (bảng SCRUM) | 39 lỗi đã xác nhận | 1 Bảo mật, 38 Chức năng |
| 📮 API Smoke | Postman + Newman | 22 request / 55 assertion | 2 lỗi được phát hiện |
| 🤖 UI Automation | Selenium + TestNG | 97 test case | Bộ test hồi quy đầy đủ |
| 🔌 API Automation | REST Assured | 66 test case | Thực thi song song |
| ⚙️ CI/CD | GitHub Actions | Mỗi lần push lên `main` | Tự động triển khai Allure Report |

**Tỷ lệ pass tổng thể: 85%** — với 39 lỗi đã được xác nhận, ghi nhận, phân loại và theo dõi đến khi xử lý xong.

---

## 🗺️ Chiến Lược Kiểm Thử

### Ưu Tiên Dựa Trên Rủi Ro

Không phải module nào cũng mang mức độ rủi ro như nhau. Nỗ lực kiểm thử được phân bổ dựa trên **mức độ ảnh hưởng đến nghiệp vụ × khả năng xảy ra lỗi**:

| Module | Mức Rủi Ro | Lý Do | Số Lượng TC |
|---|:---:|---|:---:|
| Auth (Đăng nhập / Phiên) | 🔴 Cao | Cổng vào toàn hệ thống; lỗi phiên ảnh hưởng đến mọi luồng | 40 |
| Đơn hàng | 🔴 Cao | Luồng doanh thu cốt lõi; lỗi ở đây đồng nghĩa mất doanh số hoặc hỏng dữ liệu | 58 |
| Sản phẩm | 🟡 Trung bình | Dữ liệu thay đổi thường xuyên; lỗi hiển thị ảnh hưởng đến niềm tin khách hàng | 59 |
| Tồn kho | 🟡 Trung bình | Lỗi đồng bộ tồn kho gây bán vượt số lượng | 53 |
| Storefront E2E | 🟡 Trung bình | Trải nghiệm hướng đến khách hàng; rủi ro bỏ giỏ hàng khi checkout | 50 |

### Các Loại Kiểm Thử Được Áp Dụng

| Loại | Mục Đích | Ví Dụ |
|---|---|---|
| ✅ Happy Path | Xác minh các luồng người dùng cốt lõi hoạt động đúng | Đăng nhập với thông tin hợp lệ → chuyển hướng đến `/app/orders` |
| ❌ Kiểm thử tiêu cực | Xác minh hệ thống từ chối input không hợp lệ một cách hợp lý | Đăng nhập sai mật khẩu → hiển thị lỗi, KHÔNG chuyển hướng |
| 🔲 Kiểm thử biên | Gây áp lực lên các giá trị biên | Số lượng trong giỏ = 0, giới hạn ký tự tối đa trên các trường form |
| 🔒 Kiểm thử bảo mật | Xác minh cơ chế bảo vệ xác thực và tách biệt dữ liệu | Request chưa xác thực đến endpoint được bảo vệ → phải trả về 401 |
| ⚡ SLA Hiệu năng | Xác thực thời gian phản hồi API nằm trong ngưỡng chấp nhận được | Mọi phản hồi API < 5000ms |
| 💉 Kiểm thử Injection | Kiểm tra khả năng chống chịu XSS và SQL injection trong các trường input | Chèn `<script>alert(1)</script>` vào trường email đăng nhập |

---

## 🐛 Các Lỗi Chính Được Phát Hiện

> Đây không phải là các lỗi giả lập — chúng được phát hiện trong quá trình thực thi kiểm thử thực tế trên môi trường staging đang chạy thật.

### 🔴 BUG-001 — Lỗ hổng bảo mật: Truy cập đơn hàng không cần xác thực vẫn trả về HTTP 200

| Trường | Chi Tiết |
|---|---|
| **ID** | BUG-001 (Newman: `06 - Negative & Edge Cases > Get Order - No Auth`) |
| **Mức độ nghiêm trọng** | 🔴 Nghiêm trọng (Critical) |
| **Độ ưu tiên** | 🔴 Cao |
| **Module** | Storefront API — Truy xuất đơn hàng |
| **Môi trường** | `https://backend-***.up.railway.app` (staging) |

**Điều gì đã xảy ra:**
Khi một request `GET /store/orders/{id}` được gửi đi **mà không có bất kỳ token xác thực nào**, API trả về `HTTP 200 OK` cùng toàn bộ dữ liệu đơn hàng — thay vì `HTTP 401 Unauthorized` như mong đợi.

**Tại sao điều này quan trọng:**
Đây là lỗi kiểm soát truy cập bị phá vỡ (broken access control). Trong môi trường sản xuất thực tế, kẻ tấn công có thể dò quét (enumerate) các order ID và lấy được thông tin đơn hàng nhạy cảm (tên khách hàng, địa chỉ giao hàng, sản phẩm đã mua) thuộc về người dùng khác — mà không cần đăng nhập. Điều này ánh xạ trực tiếp đến **OWASP Top 10: A01:2021 — Broken Access Control**.

**Các bước tái hiện (Postman):**
```
GET {{base_url}}/store/orders/{{order_id}}
Headers: (không có header Authorization, không có session cookie)

Mong đợi: 401 Unauthorized
Thực tế:  200 OK  +  đầy đủ dữ liệu đơn hàng
```

**Đề xuất khắc phục:**
Thêm middleware phía server để xác minh rằng `customer_id` của phiên đang request khớp với `customer_id` của đơn hàng trước khi trả dữ liệu về. Việc xác thực phải được thực thi ở cấp độ route, chứ không chỉ giả định từ phía client.

---

### 🟡 BUG-002 — Lỗi chức năng: Chuyển hướng phiên không hoạt động trên `/app/login`

| Trường | Chi Tiết |
|---|---|
| **ID** | [SCRUM-8](https://nguyen1710.atlassian.net/browse/SCRUM-8) — `MED_AUTH_TC_004` |
| **Mức độ nghiêm trọng** | 🟡 Trung bình |
| **Độ ưu tiên** | 🟡 Trung bình |
| **Module** | Admin — Xác thực / Quản lý phiên |
| **Môi trường** | `https://backend-***.up.railway.app/app/login` |

**Điều gì đã xảy ra:**
Khi người dùng đã xác thực (có phiên hợp lệ) và tự tay điều hướng đến `/app/login`, hệ thống lẽ ra phải tự động chuyển hướng họ đến `/app/orders`. Thay vào đó, trang đăng nhập vẫn được hiển thị như thể người dùng chưa xác thực — phiên đăng nhập bị bỏ qua.

**Tại sao điều này quan trọng:**
Đây là lỗi trải nghiệm người dùng (UX) có liên quan đến bảo mật. Người dùng có phiên hợp lệ bị buộc phải xác thực lại một cách không cần thiết. Quan trọng hơn, điều này cho thấy phía frontend không đọc hoặc xử lý trạng thái phiên một cách nhất quán — điều này đặt ra nghi vấn về các luồng phụ thuộc vào phiên khác.

**Bằng chứng:** Video ghi lại được đính kèm trong ticket Jira SCRUM-8 (MP4, 15MB).

**Đề xuất khắc phục:**
Thêm cơ chế bảo vệ phiên (session guard) tại cấp độ route `/app/login`: nếu tồn tại token xác thực hợp lệ trong cookie/local storage, chuyển hướng đến `/app/orders` trước khi render component đăng nhập.

---

### 📋 Phân Bố Lỗi Theo Module

| Module | Tổng Số Lỗi | Điểm Nổi Bật |
|---|:---:|---|
| Auth | 17 | Mật độ lỗi cao nhất — các bất nhất trong xử lý phiên |
| Đơn hàng | 8 | Các trường hợp biên khi tạo đơn hàng nháp (draft order) |
| Sản phẩm | 7 | Lỗi hiển thị UI trên drawer tạo sản phẩm |
| Tồn kho | 3 | Độ trễ hiển thị đồng bộ tồn kho |
| Storefront | 4 | Luồng thanh toán, bao gồm lỗi bảo mật nêu trên |
| **Tổng** | **39** | |

---

## 🤖 Framework Tự Động Hóa

Tầng tự động hóa nằm trong một repository riêng để giữ cho các mối quan tâm (concerns) được tách bạch:

🔗 **[`medusa-v2-automation`](https://github.com/Nguyn1710/medusa-v2-automation)** — README đầy đủ với kiến trúc, cài đặt và chi tiết CI.

### Vì Sao Tự Động Hóa Được Xây Dựng Sau Kiểm Thử Thủ Công

Kiểm thử thủ công được thực hiện trước — có chủ đích. Tự động hóa được xây dựng *sau khi* hành vi của hệ thống đã được hiểu rõ và các lỗi lớn đã được ghi nhận đầy đủ. Điều này phản ánh đúng trình tự chuyên nghiệp: bạn không tự động hóa một hệ thống chưa được biết rõ; bạn tự động hóa một hệ thống đã được biết rõ và tương đối ổn định.

39 lỗi được tìm thấy trong quá trình kiểm thử thủ công đã giúp xác định *luồng nào* đủ ổn định để tự động hóa và luồng nào cần được chú ý thủ công kỹ hơn.

### Tóm Tắt Phạm Vi Bao Phủ

```
┌──────────────────────────────────────────────────────┐
│            🤖 TỰ ĐỘNG HÓA: 163 Test Case              │
├──────────────────────────────────────────────────────┤
│  Test API (REST Assured)            66 test          │
│  ├─ Admin API (Auth, CRUD)          36 test          │
│  └─ Storefront API (Auth/Cart/Order)30 test          │
│                                                      │
│  Test UI — Admin (Selenium)         42 test          │
│  Test UI — Storefront (Selenium)    55 test          │
└──────────────────────────────────────────────────────┘
```

### Test Thủ Công Được Ánh Xạ Sang Tự Động Hóa Như Thế Nào

260 test case thủ công được dùng làm đặc tả (specification) cho tự động hóa. 163 test case tự động hóa bao phủ các **luồng ổn định, tần suất cao** — chủ yếu là các luồng smoke test và regression. Các test case còn lại cần thực thi thủ công do độ phức tạp (kiểm tra trực quan, các luồng phụ thuộc phiên, kiểm thử khám phá — exploratory testing).

| Test Case Thủ Công | Đã Tự Động Hóa | Chỉ Thủ Công | Tỷ Lệ Tự Động Hóa |
|:---:|:---:|:---:|:---:|
| 260 | 163 | 97 | ~63% |

### Ngăn Xếp Công Nghệ (Tech Stack)

| Công Cụ | Vai Trò |
|---|---|
| Java 11 + Maven | Ngôn ngữ cốt lõi & build |
| Selenium WebDriver 4.23 | Tự động hóa trình duyệt |
| REST Assured 5.4 | Client kiểm thử API |
| TestNG 7.10 | Thực thi & phân nhóm test |
| Allure 2.28 | Báo cáo HTML |
| Newman (Postman) | Trình chạy API smoke test |
| GitHub Actions | Pipeline CI/CD |

---

## 📮 Kiểm Thử API Smoke — Báo Cáo Newman

Bộ sưu tập Postman được thực thi qua Newman CLI để thực hiện kiểm thử smoke tự động trên toàn bộ luồng Storefront E2E Checkout.

| Chỉ số | Giá trị |
|---|---|
| Tổng số Request | 22 |
| Tổng số Assertion | 55 |
| Assertion Passed | 53 |
| Assertion Failed | **2** (lỗi đã xác nhận — xem BUG-001) |
| Công cụ chạy | Newman CLI + htmlextra reporter |

- 📊 **Báo cáo trực tiếp (Live):** [Xem Báo Cáo Newman HTML ↗](https://nguyn1710.github.io/medusa-qa-portfolio/reports/newman-storefront-e2e-report.html)
- 📁 **File gốc:** [docs/reports/newman-storefront-e2e-report.html](./docs/reports/newman-storefront-e2e-report.html)

![Báo Cáo Newman API Smoke — Storefront E2E Checkout Flow](./screenshots/newman-report.png)

> **Phát hiện chính:** 2 assertion thất bại liên quan trực tiếp đến **BUG-001** — endpoint truy xuất đơn hàng không xác thực trả về `HTTP 200` thay vì `401 Unauthorized`. Đây là lỗ hổng bảo mật đã được mô tả trong phần Bug ở trên.

---

## 📂 Nội Dung Trong Repository Này

```
medusa-qa-portfolio/
│
├── 📊 Testcases.xlsx              # Test case thủ công: 260 TC, 91 Kịch bản, Bug Log, Dashboard
│
├── 📋 docs/
│   ├── test-strategy.md           # Phương pháp kiểm thử dựa trên rủi ro, phạm vi, tiêu chí vào/ra
│   ├── bug-report-BUG-001.md      # Lỗi bảo mật — bài viết đầy đủ
│   ├── jira-exports/              # Bản xuất ticket Jira SCRUM-8 và các ticket quan trọng khác
│   └── reports/
│       └── newman-storefront-e2e-report.html  # Báo cáo Newman HTML tương tác
│
├── 📸 screenshots/
│   └── newman-report.png          # Ảnh chụp dashboard Newman
│
└── README.md                      # ← Chính là tài liệu này
```

---

## 💡 Những Điều Rút Ra & Bài Học

Dự án này được xây dựng trọn vẹn end-to-end như một hồ sơ QA. Những điều rút ra một cách thẳng thắn:

**Những gì đã làm tốt:**
- Ưu tiên kiểm thử dựa trên rủi ro giúp tập trung nỗ lực vào những chỗ quan trọng — Auth và Đơn hàng có mật độ lỗi cao nhất, đúng như đánh giá rủi ro ban đầu.
- Việc phát hiện lỗ hổng bảo mật (BUG-001) thông qua kiểm thử API tiêu cực đã cho thấy giá trị của việc kiểm thử những gì hệ thống *nên từ chối*, không chỉ những gì nó nên chấp nhận.
- Xây dựng tự động hóa *sau* kiểm thử thủ công đã ngăn việc tự động hóa dựa trên hành vi không ổn định hoặc sai lệch.

**Những gì sẽ làm khác đi trong môi trường nhóm:**
- Thiết lập khả năng truy xuất nguồn gốc (traceability) giữa test case thủ công và test tự động hóa ngay từ ngày đầu (hiện tại thủ công và tự động hóa là hai sản phẩm song song, không liên kết bằng ID).
- Viết báo cáo lỗi ngay tại thời điểm phát hiện, không phải sau đó — Bug Log trong Testcases.xlsx được điền lại hồi cứu, khiến một số chi tiết bị mất.
- Đưa tự động hóa vào sớm hơn trong chu kỳ sprint như một lưới an toàn hồi quy (regression net), thay vì xây dựng tất cả vào cuối.

---

## 👤 Giới Thiệu

**Nguyen Le** — Kỹ sư QA (Entry Level)

Có nền tảng phát triển phần mềm (~1 năm), đang chuyển hướng hoàn toàn sang QA/QC. Dự án này được xây dựng để chứng minh kỹ năng kiểm thử thực chiến xuyên suốt toàn bộ vòng đời QA: thiết kế test, thực thi thủ công, báo cáo lỗi, kiểm thử API, và tự động hóa.

- 🔗 Repo Tự động hóa: [github.com/Nguyn1710/medusa-v2-automation](https://github.com/Nguyn1710/medusa-v2-automation)
- 🐛 Bug Tracker: [nguyen1710.atlassian.net](https://nguyen1710.atlassian.net/browse/SCRUM-8)

---

<p align="center">
  <i>Được xây dựng để tìm ra vấn đề thực sự — không chỉ để viết test "xanh".</i>
  <b>Được xây dựng bằng ❤️ để học hỏi QA</b><br/>
</p>