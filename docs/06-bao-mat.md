# Kế hoạch & Ma trận An ninh Bảo mật (docs/06-bao-mat.md)

> **Dự án**: Website Thương mại điện tử Võ phục & Dụng cụ Võ thuật "Đai Trắng"  
> **Căn cứ**: Mục 9 – Đặc tả an ninh bảo mật bắt buộc trong đồ án  

---

## 1. Ma trận Nguy cơ & Biện pháp Bảo vệ

| Nguy cơ An ninh | Biện pháp Bắt buộc | Vị trí Cài đặt trong Mã nguồn | Trạng thái |
|---|---|---|:---:|
| **Lộ mật khẩu** | Băm mật khẩu bằng thuật toán Bcrypt (hoặc PBKDF2), độ dài tối thiểu 8 ký tự. Không lưu plaintext. | `backend/app/core/security.py`, `backend/app/modules/auth/schemas.py` | ✅ Đã kiểm thử |
| **Dò mật khẩu (Brute Force)** | Khóa tạm thời tài khoản sau 5 lần đăng nhập sai liên tiếp trong 15 phút. | `backend/app/modules/auth/router.py` (hàm `login`) | ✅ Đã kiểm thử |
| **Tấn công Injection (NoSQL / SQL)** | Không ghép chuỗi truy vấn trực tiếp; sử dụng ODM và Pydantic validation với truy vấn tham số hóa (Parameterized queries). | `backend/app/modules/` | ✅ Đã áp dụng |
| **Xem dữ liệu trái phép (IDOR)** | Khách hàng chỉ xem đơn hàng và hồ sơ size của chính mình; HLV chỉ xem đợt hàng của võ đường mình phụ trách; Kiểm tra quyền ở mọi API view. | `backend/app/core/dependencies.py` (`get_current_user`, `require_role`) | ✅ Đã thiết lập |
| **Tấn công Cross-Site Scripting (XSS)** | Escape dữ liệu mặc định trên giao diện, không sử dụng `dangerouslySetInnerHTML` với dữ liệu do người dùng nhập. | `frontend/`, Pydantic input models | ✅ Đã thiết lập |
| **Tấn công CSRF** | Sử dụng JWT Bearer Token trong header Authorization kết hợp cấu hình CORS nghiêm ngặt. | `backend/app/main.py`, `backend/app/core/config.py` | ✅ Đã thiết lập |
| **Lộ cấu hình & Secrets** | Mọi biến môi trường bí mật (SECRET_KEY, MONGODB_URI, MOMO keys) tách biệt trong file `.env`, có `.env.example`, chặn commit secrets lên Git qua `.gitignore`. | `.gitignore`, `.env.example` | ✅ Đã thiết lập |
| **Gian lận Thanh toán** | Xác minh chữ ký số HMAC-SHA256 trên webhook IPN của MoMo, đối soát đúng mã đơn và số tiền thực thu, xử lý lặp Idempotency, không lưu thông tin thẻ ngân hàng của khách. | `backend/app/modules/payments/router.py`, `backend/app/modules/payments/service.py` | ✅ Đã hoàn thành (GĐ 4) |
| **Bảo vệ Dữ liệu Trẻ em** | Chỉ thu thập các chỉ số tối thiểu phục vụ tư vấn chọn size võ phục (chiều cao, cân nặng, năm sinh); cung cấp tính năng xóa hồ sơ size bất cứ lúc nào, cảnh báo nhắc đo lại sau 6 tháng. | `backend/app/modules/size_advisor/router.py` | ✅ Đã hoàn thành (GĐ 2) |
| **Thao tác sai của Nhân viên** | Phân quyền vai trò nghiêm ngặt (Staff bị chặn 403 khi xem báo cáo doanh thu); ghi nhật ký hoạt động bất biến (`activity_logs`) cho mọi thao tác nhạy cảm. | `backend/app/modules/admin/router.py`, `backend/app/core/activity.py` | ✅ Đã hoàn thành (GĐ 6) |
| **Tấn công Từ chối Dịch vụ (DDoS / Brute-force)** | Giới hạn tần suất yêu cầu (Rate Limiting) trượt 60s: 120 req/phút chung, 15 req/phút cho endpoint xác thực nhạy cảm; trả về 429 và Retry-After. | `backend/app/core/middleware.py` (`RateLimitMiddleware`) | ✅ Đã hoàn thành (GĐ 7) |
| **Tấn công Web Khách (Clickjacking, Sniffing)** | Đính kèm đầy đủ tiêu đề bảo mật OWASP: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, HSTS, CSP, Referrer-Policy. | `backend/app/core/middleware.py` (`SecurityHeadersMiddleware`) | ✅ Đã hoàn thành (GĐ 7) |
| **Lọc Dữ liệu Độc hại (XSS & NoSQL Injection)** | Khử thẻ script, iframe, thuộc tính sự kiện nguy hiểm; escape ký tự toán tử MongoDB (`$where`, `$gt`...); che dữ liệu nhạy cảm (`***MASKED***`). | `backend/app/core/security.py` (`sanitize_html`, `sanitize_nosql_input`, `mask_sensitive_data`) | ✅ Đã hoàn thành (GĐ 7) |
