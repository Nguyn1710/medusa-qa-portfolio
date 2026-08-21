# 🟡 Báo Cáo Lỗi — BUG-002: Session Timeout Không Xóa Token Phía Client / Mất Dữ Liệu Form Chưa Lưu

> **Loại tài liệu:** Báo cáo lỗi chi tiết (Bug Report Deep-Dive)
> **Tác giả:** Nguyen Le — Kỹ sư QA (Entry Level)
> **Ngày phát hiện:** Tháng 8 năm 2026
> **Giai đoạn kiểm thử:** Kiểm thử chức năng thủ công — Module Auth & Sản phẩm
> **Tham chiếu Jira:** [SCRUM-12](https://nguyen1710.atlassian.net/browse/SCRUM-12)

---

## 1. Tóm Tắt Lỗi

| Trường | Chi Tiết |
|---|---|
| **Bug ID** | BUG-002 |
| **Tiêu đề** | Session timeout không xóa token phía client; dữ liệu form sản phẩm chưa lưu bị mất trắng không có cảnh báo |
| **Mức độ nghiêm trọng** | 🟡 Trung bình (Medium) |
| **Độ ưu tiên** | 🟡 Trung bình (Medium) |
| **Module** | Admin — Xác thực / Quản lý phiên; Admin — Form tạo sản phẩm |
| **Tính năng** | Xử lý hết hạn phiên; dọn dẹp token phía client; bảo toàn trạng thái form |
| **Trạng thái** | Đang xem xét (In Review — tính đến tháng 8 năm 2026 — môi trường staging) |
| **Người báo cáo** | Nguyen Le |
| **Ticket Jira** | [SCRUM-12](https://nguyen1710.atlassian.net/browse/SCRUM-12) |
| **Test case liên quan** | `MED_PROD_TC_064` (Session timeout trong form sản phẩm), `MED_ORD_TC_059` (Session timeout trong form Draft Order) |

---

## 2. Môi Trường

| Tham số | Giá trị |
|---|---|
| **Ứng dụng** | Medusa V2 — Admin Dashboard |
| **Môi trường** | Staging |
| **URL** | `https://backend-***.up.railway.app/app` |
| **Công cụ kiểm thử** | Kiểm thử thủ công trên trình duyệt (incognito mode) |
| **Trình duyệt kiểm thử** | Chrome (phiên bản mới nhất) |

---

## 3. Điều Kiện Tiên Quyết (Preconditions)

1. Người dùng đã xác thực (token phiên hợp lệ có trong local storage / cookies của trình duyệt).
2. Người dùng đang điền một form với dữ liệu chưa lưu (ví dụ: form tạo sản phẩm hoặc form draft order).
3. TTL phiên phía server được cấu hình để hết hạn sau một khoảng thời gian không hoạt động cố định.

---

## 4. Các Bước Tái Hiện (Steps to Reproduce)

```
Bước 1: Đăng nhập vào Admin Dashboard
  - Truy cập: https://backend-***.up.railway.app/app/login
  - Nhập thông tin đăng nhập hợp lệ → xác nhận redirect đến /app/orders

Bước 2: Điều hướng đến form tạo sản phẩm
  - Vào Products → Click "New Product"
  - Bắt đầu điền form: nhập Title, Description, thêm Variants, v.v.
  - KHÔNG publish hoặc save tại bước này

Bước 3: Kích hoạt hết hạn phiên (không refresh trang)
  - Phương pháp: Chờ TTL phiên phía server hết hạn
    (hoặc vô hiệu hóa phiên thủ công từ phía server)

Bước 4: Tiếp tục tương tác với form hoặc cố gắng điều hướng
  - Thử điều hướng sang trang admin khác (ví dụ: /app/orders)
  - HOẶC đơn giản là chờ và quan sát UI

Bước 5: Quan sát điều gì xảy ra với:
  (a) Auth token trong browser storage
  (b) Dữ liệu form đã nhập nhưng chưa lưu
  (c) Bất kỳ thông báo hay redirect nào từ phía user
```

---

## 5. Kết Quả Mong Đợi vs Kết Quả Thực Tế

### Kết Quả Mong Đợi

| # | Hành vi mong đợi |
|---|---|
| 1 | Khi phiên hết hạn, lần gọi API tiếp theo từ frontend nhận được `HTTP 401 Unauthorized` |
| 2 | Frontend chặn (intercept) 401 và **xóa** token xác thực cũ khỏi local storage / cookies |
| 3 | Người dùng được **redirect đến `/app/login`** với thông báo rõ ràng: *"Phiên làm việc của bạn đã hết hạn. Vui lòng đăng nhập lại."* |
| 4 | Trước khi redirect (hoặc ít nhất trước khi phiên hết hạn), hệ thống **cảnh báo người dùng** về các thay đổi chưa lưu trong form |
| 5 | Dữ liệu form đã nhập phải được bảo toàn qua auto-save hoặc người dùng được cảnh báo rõ ràng rằng dữ liệu sẽ bị mất |

### Kết Quả Thực Tế

| # | Hành vi thực tế |
|---|---|
| 1 | Server trả về `HTTP 401` đúng khi phiên hết hạn ✅ |
| 2 | Frontend **không** chặn 401 để xóa token cũ ❌ |
| 3 | Token cũ **vẫn còn** trong local storage sau khi phiên hết hạn ❌ |
| 4 | Khi phiên hết hạn trong lúc điền form sản phẩm, hệ thống **redirect về login** — nhưng **không có cảnh báo trước** về dữ liệu chưa lưu ❌ |
| 5 | Toàn bộ dữ liệu form (tiêu đề, mô tả, variant, giá đã nhập) **bị mất vĩnh viễn** sau redirect ❌ |
| 6 | Không hiển thị thông báo "phiên hết hạn" — lỗi phiên **xảy ra âm thầm** ❌ |

**Tóm tắt các lỗi được xác nhận trong ticket này:**
- **Lỗi A:** Token cũ không được xóa khi nhận 401
- **Lỗi B:** Không có cảnh báo hết hạn phiên cho người dùng
- **Lỗi C:** Dữ liệu form chưa lưu bị hủy mà không có sự đồng ý của người dùng

---

## 6. Bằng Chứng (Evidence)

| Bằng chứng | Mô tả |
|---|---|
| **Ticket Jira** | [SCRUM-12 ↗](https://nguyen1710.atlassian.net/browse/SCRUM-12) — Chi tiết đầy đủ và trao đổi nội bộ |
| **Test case liên quan** | `MED_PROD_TC_064` (Fail), `MED_ORD_TC_059` (Fail) — cả hai xác nhận kiểu lỗi này |
| **Kết quả test case** | `MED_PROD_TC_064`: Actual Result — "Khi phiên làm việc hết hạn trong lúc tạo sản phẩm, hệ thống tự động redirect về trang login mà không lưu nháp; toàn bộ dữ liệu form đã nhập bị mất trắng, không hiển thị bất kỳ cảnh báo nào trước khi hết hạn." |

---

## 7. Phân Tích Nguyên Nhân Gốc Rễ

> **Lưu ý:** Phân tích dựa trên hành vi quan sát được, không có quyền truy cập mã nguồn server. **Cần dev xác nhận.**

**Lỗi A — Token cũ không được xóa:**
Frontend HTTP client (có thể là Axios hoặc fetch) không có global response interceptor cho `401 Unauthorized`. Khi server từ chối request bằng 401, lỗi truyền lên như một JavaScript error nhưng không có logic dọn dẹp nào được thực thi. Token vẫn tồn tại trong storage cho đến khi đăng xuất thủ công hoặc xóa cache trình duyệt.

**Lỗi B — Không có cảnh báo hết hạn phiên:**
Frontend không triển khai bộ hẹn giờ hoạt động phiên (ví dụ: `setTimeout` dựa trên TTL cấu hình từ server). Không có giám sát chủ động, tín hiệu duy nhất về hết hạn phiên là một API request thất bại — đến lúc đó dữ liệu đã bị mất.

**Lỗi C — Mất dữ liệu form:**
Form tạo sản phẩm không triển khai:
- Auto-save định kỳ sang trạng thái nháp
- Phát hiện thay đổi chưa lưu (pattern sự kiện `beforeunload` chuẩn)
- Fallback LocalStorage cho trạng thái form đang nhập dở

**[CẦN DEV XÁC NHẬN]:**
1. TTL phiên được cấu hình phía server là bao nhiêu?
2. Hiện có global 401 interceptor nào trong frontend HTTP client không? Nếu có, nó có đang hoạt động cho admin routes không?
3. Form sản phẩm có bất kỳ cơ chế auto-save nháp nào trong triển khai hiện tại không?

---

## 8. Phân Tích Tác Động (Impact Analysis)

| Chiều Tác Động | Đánh Giá |
|---|---|
| **Trải nghiệm người dùng** | 🔴 Cao — Người dùng có thể mất khối lượng công việc đáng kể (sản phẩm phức tạp với nhiều variant, giá cả, hình ảnh) mà không có cảnh báo. Đặc biệt gây hại cho các hoạt động quản lý catalog lớn. |
| **Bảo mật** | 🟡 Trung bình — Token cũ trong storage là mối lo ngại bảo mật thứ cấp. Tự thân nó ít nghiêm trọng hơn; kết hợp với lỗ hổng XSS sẽ nghiêm trọng hơn. |
| **Toàn vẹn dữ liệu** | 🟡 Trung bình — Không có dữ liệu bị ghi sai, nhưng sản phẩm công việc của người dùng bị hủy bất ngờ. |
| **Niềm tin** | 🔴 Cao — Mất dữ liệu bất ngờ không có cảnh báo làm giảm đáng kể niềm tin vào độ tin cậy của công cụ admin. |
| **Tần suất** | 🟡 Trung bình — Ảnh hưởng đến tất cả admin trong các phiên chỉnh sửa dài (quản lý catalog sản phẩm vốn tốn thời gian). |

---

## 9. Đề Xuất Khắc Phục

### Khắc phục Lỗi A — Xóa token cũ khi nhận 401

Thêm global response interceptor trong frontend HTTP client:

```javascript
// Ví dụ: Axios interceptor
axios.interceptors.response.use(
  response => response,
  error => {
    if (error.response && error.response.status === 401) {
      // 1. Xóa auth token
      localStorage.removeItem('_medusa_jwt');
      document.cookie = 'connect.sid=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;';
      // 2. Redirect kèm lý do
      window.location.href = '/app/login?reason=session_expired';
    }
    return Promise.reject(error);
  }
);
```

### Khắc phục Lỗi B — Cảnh báo hết hạn phiên

Triển khai bộ hẹn giờ phiên chủ động cảnh báo người dùng trước khi hết hạn:

```javascript
// Cảnh báo người dùng 2 phút trước khi phiên hết hạn
const SESSION_TTL_MS = /* lấy từ config */;
const WARN_BEFORE_MS = 2 * 60 * 1000; // 2 phút

setTimeout(() => {
  showDialog("Phiên làm việc sẽ hết hạn sau 2 phút. Vui lòng lưu công việc của bạn.");
}, SESSION_TTL_MS - WARN_BEFORE_MS);
```

### Khắc phục Lỗi C — Bảo vệ dữ liệu form chưa lưu

1. **Ngắn hạn:** Thêm listener sự kiện `beforeunload` trên form sản phẩm để cảnh báo người dùng về thay đổi chưa lưu:
   ```javascript
   window.addEventListener('beforeunload', (e) => {
     if (hasUnsavedChanges()) {
       e.preventDefault();
       e.returnValue = '';
     }
   });
   ```

2. **Dài hạn:** Triển khai auto-save định kỳ sang nháp (ví dụ: mỗi 60 giây), lưu trạng thái form vào local storage hoặc endpoint nháp phía server.

---

## 10. Bài Học Rút Ra — Góc Nhìn QA

*Phần này phản ánh học hỏi cá nhân từ việc phát hiện và ghi nhận lỗi này.*

**Form là bề mặt dễ bị tổn thương nhất với mất dữ liệu liên quan đến phiên:**

Trong đánh giá rủi ro, module Auth được xác định đúng là High risk. Nhưng hiệu ứng kết hợp của lỗi quản lý phiên lên *các tính năng khác* (như form sản phẩm) đã bị đánh giá thấp. Khi lỗi phiên gây mất dữ liệu trong một module hoàn toàn khác (Products), nó chứng minh rằng quản lý phiên là mối quan tâm xuyên suốt (cross-cutting concern) — lỗi của nó không ở lại trong module Auth.

**Sự khác biệt giữa hành vi server và hành vi client:**

Server đang làm đúng: nó hết hạn phiên và trả về 401. Lỗi hoàn toàn nằm ở cách *client* phản hồi với tín hiệu đó. Sự phân biệt này quan trọng cho báo cáo lỗi: nhóm backend có thể đọc "lỗi session timeout" và nghĩ mã của họ sai. Một báo cáo chính xác — "hành vi phía server đúng; handler 401 phía client bị thiếu" — ngăn nỗ lực bị hướng sai và đẩy nhanh giải quyết.

**Kiểm thử phiên dài hạn đòi hỏi thiết lập có chủ đích:**

Lỗi session timeout vô hình trong các lần chạy kiểm thử chức năng thông thường, nơi các phiên vừa tươi vừa ngắn. Phát hiện lỗi này đòi hỏi cố ý kích hoạt phiên hết hạn — một bước phải được thiết kế rõ ràng vào kế hoạch kiểm thử. Điều này củng cố giá trị của việc bao gồm chuyển đổi trạng thái phiên như các kịch bản kiểm thử rõ ràng, không chỉ "đăng nhập hoạt động / đăng xuất hoạt động".

**Lỗi mất dữ liệu mang tác động người dùng không cân xứng:**

Một lỗi Medium theo định nghĩa nghiêm ngặt (không có lỗ hổng bảo mật, không hỏng dữ liệu, có workaround) có thể cảm giác Critical với người dùng vừa mất 30 phút xây dựng danh sách sản phẩm phức tạp. Phân loại mức độ nghiêm trọng luôn phải được kiểm tra chéo với kịch bản tác động người dùng thực tế, không chỉ theo danh mục kỹ thuật.

---

*Báo cáo lỗi được soạn thảo như một phần của dự án Medusa V2 QA Portfolio bởi Nguyen Le.*
*Xem thêm: [BUG-001 (Bảo mật)](./bug-report-BUG-001_VI.md) | [Chiến lược kiểm thử](./test-strategy_VI.md) | [SCRUM-12 Jira Export](./jira-exports/SCRUM-12-bug-session-timeout.md)*
