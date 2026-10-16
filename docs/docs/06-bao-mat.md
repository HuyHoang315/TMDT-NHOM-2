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
| **Gian lận Thanh toán** | Xác minh chữ ký số HMAC-SHA256 trên webhook IPN của MoMo, đối soát đúng mã đơn và số tiền thực thu, không lưu thông tin thẻ ngân hàng của khách. | `backend/app/modules/orders/` | Sẽ tích hợp GĐ 4 |
| **Bảo vệ Dữ liệu Trẻ em** | Chỉ thu thập các chỉ số tối thiểu phục vụ tư vấn chọn size võ phục (chiều cao, cân nặng, năm sinh); cung cấp tính năng cho phép xóa hồ sơ size bất cứ lúc nào. | `backend/app/modules/users/` | Sẽ tích hợp GĐ 2 |
| **Thao tác sai của Nhân viên** | Phân quyền vai trò nghiêm ngặt (Staff không xem được báo cáo doanh thu); ghi nhật ký hoạt động (`ActivityLog`) cho các thao tác đổi trạng thái đơn, chỉnh kho, hoàn tiền. | `backend/app/modules/admin/` | Sẽ tích hợp GĐ 6 |
