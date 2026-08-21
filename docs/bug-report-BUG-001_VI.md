# 🔴 Báo Cáo Lỗi — BUG-001: Kiểm Soát Truy Cập Bị Phá Vỡ trên Endpoint Lấy Thông Tin Đơn Hàng

> **Loại tài liệu:** Báo cáo lỗi chi tiết (Bug Report Deep-Dive)
> **Tác giả:** Nguyen Le — Kỹ sư QA (Entry Level)
> **Ngày phát hiện:** Tháng 8 năm 2026
> **Giai đoạn kiểm thử:** API Smoke Testing — Newman / Postman

---

## 1. Tóm Tắt Lỗi

| Trường | Chi Tiết |
|---|---|
| **Bug ID** | BUG-001 |
| **Tiêu đề** | `GET /store/orders/{id}` không cần xác thực vẫn trả về HTTP 200 cùng toàn bộ dữ liệu đơn hàng |
| **Mức độ nghiêm trọng** | 🔴 Nghiêm trọng (Critical) |
| **Độ ưu tiên** | 🔴 Cao (High) |
| **Danh mục OWASP** | **A01:2021 — Broken Access Control (Kiểm soát truy cập bị phá vỡ)** |
| **Module** | Storefront API — Truy xuất đơn hàng |
| **Tính năng** | Endpoint `GET /store/orders/{id}` |
| **Trạng thái** | Mở (Open — tính đến tháng 8 năm 2026 — môi trường staging) |
| **Người báo cáo** | Nguyen Le |
| **Tham chiếu Jira** | Xem thêm: Newman test `06 - Negative & Edge Cases > Get Order - No Auth` |

---

## 2. Môi Trường

| Tham số | Giá trị |
|---|---|
| **Ứng dụng** | Medusa V2 — Storefront API |
| **Môi trường** | Staging |
| **Backend Host** | `https://backend-***.up.railway.app` |
| **Công cụ kiểm thử** | Postman / Newman CLI (`newman run` với htmlextra reporter) |
| **Tham chiếu báo cáo** | [`docs/reports/newman-storefront-e2e-report.html`](./reports/newman-storefront-e2e-report.html) |
| **Test bị ảnh hưởng** | Newman Collection: `06 - Negative & Edge Cases > Get Order - No Auth` |

---

## 3. Điều Kiện Tiên Quyết (Preconditions)

1. Tồn tại ít nhất một đơn hàng trong hệ thống với `order_id` đã biết (ví dụ: lấy từ luồng checkout đã xác thực trước đó trong cùng một lần chạy Newman collection).
2. Request được gửi đi **không có** header `Authorization` và **không có** session cookie nào.
3. Medusa V2 backend đã được triển khai và truy cập được tại URL staging.

---

## 4. Các Bước Tái Hiện (Steps to Reproduce)

```
Bước 1: Lấy một order_id hợp lệ
  - Phương pháp: Hoàn thành luồng checkout qua Storefront API (như đã thực hiện
    ở các bước trước của Newman collection: đăng ký → đăng nhập → duyệt sản phẩm → giỏ hàng → thanh toán)
  - Ghi lại order_id được trả về trong response checkout

Bước 2: Gửi GET request không xác thực đến endpoint đơn hàng
  - Method: GET
  - URL: {{base_url}}/store/orders/{{order_id}}
  - Headers:
      Content-Type: application/json
      x-publishable-api-key: {{pub_key}}
      [KHÔNG CÓ header Authorization]
      [KHÔNG CÓ session cookie]
  - Body: (trống)

Bước 3: Quan sát HTTP response status code và response body
```

**Tương đương Postman / Newman:**
```http
GET https://backend-***.up.railway.app/store/orders/order_01XXXXXXXXXXXXXXXX
Content-Type: application/json
x-publishable-api-key: pk_***
```
*(Không bao gồm header `Authorization` hay `Cookie`)*

---

## 5. Kết Quả Mong Đợi vs Kết Quả Thực Tế

| | Chi tiết |
|---|---|
| **Mong đợi** | `HTTP 401 Unauthorized` — Server phải từ chối mọi request không xuất trình phiên xác thực hợp lệ của khách hàng sở hữu đơn hàng. Không có dữ liệu đơn hàng nào được trả về. |
| **Thực tế** | `HTTP 200 OK` — Server trả về toàn bộ payload đơn hàng, bao gồm: tên khách hàng, địa chỉ giao hàng, sản phẩm đã đặt, giá cả và trạng thái đơn hàng. |

**Response thực tế (đã ẩn thông tin nhạy cảm):**
```json
HTTP/1.1 200 OK

{
  "order": {
    "id": "order_01XXXXXXXXXXXXXXXX",
    "status": "pending",
    "customer": {
      "email": "[email khách hàng]",
      ...
    },
    "shipping_address": {
      "first_name": "[tên]",
      "last_name": "[tên]",
      "address_1": "[địa chỉ]",
      ...
    },
    "items": [...],
    "total": ...,
    ...
  }
}
```

---

## 6. Bằng Chứng (Evidence)

| Bằng chứng | Mô tả |
|---|---|
| **Newman HTML Report** | [`docs/reports/newman-storefront-e2e-report.html`](./reports/newman-storefront-e2e-report.html) — Báo cáo tương tác hiển thị 2 assertion thất bại |
| **Newman Screenshot** | [`screenshots/newman-report.png`](../screenshots/newman-report.png) — Giao diện dashboard của lần chạy Newman |
| **Assertion thất bại** | 2 trong tổng số 55 assertion thất bại — cả hai ở test `06 - Negative & Edge Cases > Get Order - No Auth` |

**Assertion Newman đã thất bại:**
```javascript
// Assertion 1: Kiểm tra status code
pm.test("Status code is 401", function () {
    pm.response.to.have.status(401);
});
// → THẤT BẠI: Nhận được 200

// Assertion 2: Body không được chứa dữ liệu đơn hàng
pm.test("Response should not contain order data", function () {
    pm.expect(pm.response.json()).to.not.have.property("order");
});
// → THẤT BẠI: Response chứa đối tượng order đầy đủ
```

---

## 7. Phân Tích Nguyên Nhân Gốc Rễ

> **Lưu ý:** Phân tích này dựa trên hành vi HTTP quan sát được mà không có quyền truy cập server-side logs hay source code. **Cần dev xác nhận.**

**Nguyên nhân gốc rễ có khả năng cao nhất:**
Route handler của `GET /store/orders/{id}` **không thực thi xác thực (authentication)** trước khi trả về dữ liệu đơn hàng. Trong kiến trúc Medusa V2, các guard xác thực cấp route phải được áp dụng rõ ràng cho từng endpoint. Nguyên nhân có thể là một trong các trường hợp sau:

| Giả thuyết | Khả năng |
|---|---|
| Route middleware hoàn toàn thiếu guard `authenticate` | 🔴 Cao |
| Route sử dụng "optional authentication" — nếu không có token, tiến hành mà không kiểm tra auth | 🟡 Trung bình |
| Guard auth có mặt nhưng bị bỏ qua do cấu hình sai | 🟡 Trung bình |

**Những gì KHÔNG phải nguyên nhân gốc rễ:**
Vấn đề không nằm ở thiết lập Postman collection — test cố tình bỏ qua mọi header auth và response xác nhận dữ liệu được trả về. Lỗi này có thể tái hiện được.

**[CẦN DEV XÁC NHẬN]:** Vui lòng xác minh middleware chain nào được áp dụng cho `GET /store/orders/:id` trong định nghĩa route và xác nhận liệu middleware `authenticate` có mặt và đang hoạt động không.

---

## 8. Phân Tích Tác Động (Impact Analysis)

| Chiều Tác Động | Đánh Giá |
|---|---|
| **Bảo mật thông tin (Confidentiality)** | 🔴 **Nghiêm trọng** — Thông tin nhận dạng cá nhân (PII) của khách hàng (tên, địa chỉ, sản phẩm đã mua) bị lộ cho các bên không xác thực |
| **Tách biệt dữ liệu** | 🔴 **Nghiêm trọng** — Bất kỳ người dùng nào biết hoặc đoán được order ID đều có thể lấy đơn hàng của khách hàng khác |
| **Vector tấn công** | Order ID trong Medusa tuân theo mẫu prefix có thể dự đoán được (`order_01...`). Việc dò quét order ID theo thứ tự (sequential enumeration) là khả thi |
| **Quy định pháp luật** | Vi phạm tiềm năng GDPR/bảo vệ dữ liệu nếu dữ liệu khách hàng có thể truy cập mà không cần xác thực |
| **Nghiệp vụ** | Rủi ro uy tín; nguy cơ trách nhiệm pháp lý do lộ dữ liệu |
| **Khả năng khai thác** | 🔴 **Cực kỳ dễ khai thác** — Không cần công cụ đặc biệt; một GET request không xác thực đơn giản là có thể tái hiện lỗi |

**Kịch bản tấn công (lý thuyết, chỉ để minh họa):**
```
1. Kẻ tấn công thực hiện checkout thử để lấy định dạng order_id hợp lệ
2. Kẻ tấn công viết vòng lặp để dò quét order ID theo thứ tự
3. Mỗi vòng lặp lấy được đơn hàng của một khách hàng khác kèm đầy đủ PII
4. Kết quả: trích xuất hàng loạt dữ liệu đơn hàng của khách hàng mà không cần xác thực
```

---

## 9. Đề Xuất Khắc Phục

**Ngay lập tức (Bắt buộc trước khi triển khai production):**

Thêm middleware xác thực phía server vào route lấy thông tin đơn hàng. Middleware phải:
1. Xác minh rằng có token phiên hoặc JWT hợp lệ trong request
2. Xác minh rằng `customer_id` liên kết với phiên khớp với `customer_id` trên đơn hàng được yêu cầu
3. Trả về `HTTP 401 Unauthorized` nếu không có auth hợp lệ
4. Trả về `HTTP 403 Forbidden` nếu có auth nhưng khách hàng không sở hữu đơn hàng được yêu cầu

**Ví dụ phương pháp tiếp cận (pseudocode):**
```javascript
// Route: GET /store/orders/:id
router.get('/store/orders/:id', 
  authenticate(),         // ← BẮT BUỘC: xác thực phiên
  async (req, res) => {
    const order = await getOrder(req.params.id);
    
    // ← BẮT BUỘC: xác thực quyền sở hữu
    if (order.customer_id !== req.session.customer_id) {
      return res.status(403).json({ message: 'Forbidden' });
    }
    
    return res.json({ order });
  }
);
```

**Bổ sung (Khuyến nghị):**
- Thêm integration test cụ thể xác minh rằng `GET /store/orders/{id}` trả về 401 khi gọi mà không có auth — đây nên là regression test vĩnh viễn.
- Rà soát tất cả các endpoint liên quan đến đơn hàng khác (`PATCH`, `POST` sub-resources) để tìm kiểu thiếu auth guard tương tự.

---

## 10. Bài Học Rút Ra — Góc Nhìn QA

*Phần này phản ánh học hỏi cá nhân từ việc phát hiện và ghi nhận lỗi này.*

**Tại sao kiểm thử API tiêu cực tìm ra lỗi này, trong khi kiểm thử chức năng thông thường có thể không:**

Trong quá trình kiểm thử chức năng thủ công trên Admin Dashboard, xác thực luôn hiện diện — người kiểm thử luôn đăng nhập. Lỗi này chỉ xuất hiện khi bạn cố ý kiểm thử những gì hệ thống nên **từ chối**, không chỉ những gì nó nên chấp nhận. Hầu hết người thiết kế test tập trung vào happy path và các lỗi hiển nhiên (sai mật khẩu, email không hợp lệ). Kiểm thử *sự vắng mặt của xác thực trên một data endpoint* đòi hỏi sự thay đổi tư duy có chủ đích, mang tính đối nghịch (adversarial mindset).

**Giá trị của các assertion bảo mật rõ ràng trong test tự động hóa:**

Newman collection đã bao gồm một test rõ ràng: *"Status code is 401"* cho request đơn hàng không xác thực. Nếu không có assertion đó, test sẽ pass (request hoàn thành thành công) dù hệ thống hoàn toàn không bảo vệ. Viết assertion bảo mật cụ thể — không chỉ "request hoàn thành thành công" mà còn "request bị từ chối đúng như mong đợi" — là một kỷ luật quan trọng trong kiểm thử API.

**OWASP như một checklist thiết kế test:**

Sử dụng OWASP Top 10 như một checklist có cấu trúc trong quá trình thiết kế test đã dẫn trực tiếp đến phát hiện này. A01 (Broken Access Control) cụ thể đề cập đến "bỏ qua kiểm tra kiểm soát truy cập bằng cách sửa đổi URL" — đây chính xác là những gì test này đã làm. Coi các danh mục OWASP như các danh mục kịch bản test, không chỉ là danh sách đọc, biến nhận thức bảo mật thành phạm vi test có thể thực hiện được.

**Kỷ luật ghi chép:**

Lỗi được tìm thấy thông qua thực thi Newman tự động, nhưng phân tích đầy đủ (giả thuyết nguyên nhân gốc rễ, phạm vi tác động, đề xuất khắc phục) đòi hỏi tư duy có cấu trúc *sau đó*. Viết một bug report đầy đủ như thế này — thay vì chỉ tóm tắt Jira ticket — buộc phải rõ ràng: Chính xác điều gì đã xảy ra? Kẻ tấn công có thể làm gì với điều này? Cần thay đổi gì để khắc phục? Kỷ luật này tạo ra các bàn giao cho developer tốt hơn và các sản phẩm portfolio tốt hơn.

---

*Báo cáo lỗi được soạn thảo như một phần của dự án Medusa V2 QA Portfolio bởi Nguyen Le.*
*Xem thêm: [Chiến lược kiểm thử](./test-strategy_VI.md) | [Báo cáo Newman](./reports/newman-storefront-e2e-report.html)*
