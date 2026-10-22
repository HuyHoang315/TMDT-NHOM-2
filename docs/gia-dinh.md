# Tài liệu các Giả định Hệ thống (docs/gia-dinh.md)

> **Dự án**: Website Thương mại điện tử Võ phục & Dụng cụ Võ thuật "Đai Trắng"  
> **Môn học**: Đồ án Thương mại điện tử  
> **Ngày cập nhật**: 16/10/2026  

---

## 1. Giả định về Giá bán và Tiền tệ
- Đơn vị tiền tệ chính thức: **Việt Nam Đồng (VND)**.
- Toàn bộ giá trị tiền tệ trong cơ sở dữ liệu được lưu bằng **số nguyên (Integer)**, không dùng số thực (`Float`/`Double`) để triệt tiêu hoàn toàn sai số làm tròn số học.
- Khung giá tham khảo:
  - Võ phục tiêu chuẩn: 220.000 VND – 550.000 VND / bộ tùy môn phái và kích thước.
  - Đai võ thuật: 45.000 VND – 120.000 VND / sợi.
  - Dụng cụ bảo hộ (giáp, găng, bảo hộ ống đồng): 150.000 VND – 480.000 VND / món.
  - Phụ kiện & túi đựng: 90.000 VND – 250.000 VND / món.

---

## 2. Giả định về Bậc giá sỉ cho Võ đường / CLB (B2B)
- Chiết khấu số lượng theo đợt đặt hàng chung:
  - Từ **10 đến 29 bộ**: Giảm **10%** trên tổng giá niêm yết.
  - Từ **30 đến 49 bộ**: Giảm **15%** trên tổng giá niêm yết.
  - Từ **50 đến 99 bộ**: Giảm **20%** trên tổng giá niêm yết.
  - Từ **100 bộ trở lên**: Giảm **25%** trên tổng giá niêm yết.
- Đặt cọc đợt hàng chung B2B: Võ đường đặt cọc trước **50%** giá trị đợt hàng qua MoMo khi chốt sổ; 50% còn lại thanh toán khi nhận hàng lô chung.

---

## 3. Giả định về Dịch vụ thêu tên và in logo
- Phí thêu tên võ sinh: **30.000 VND / sản phẩm** (cấu hình linh hoạt).
- Ràng buộc thêu tên:
  - Tối đa **20 ký tự**, hỗ trợ đầy đủ tiếng Việt có dấu.
  - Đơn hàng có thêu tên bắt buộc thanh toán trước (MoMo), **không hỗ trợ COD**.
  - Sản phẩm đã thêu tên riêng **không áp dụng chính sách đổi trả / hoàn tiền** (trừ lỗi rách vải do nhà sản xuất).

---

## 4. Giả định về Vận chuyển và Giao nhận
- Cơ chế vận chuyển nội bộ giả lập (`MockShippingProvider`):
  - Nội thành TP. Hồ Chí Minh: Phí cố định **25.000 VND**, thời gian giao 1–2 ngày.
  - Các tỉnh thành khác: Phí cố định **40.000 VND**, thời gian giao 3–5 ngày.
  - Miễn phí vận chuyển (Freeship): Đơn hàng có giá trị tạm tính từ **500.000 VND** trở lên.

---

## 5. Giả định về Đổi size và Trả hàng
- Thời hạn yêu cầu đổi size: Trong vòng **7 ngày** kể từ thời điểm đơn hàng chuyển trạng thái `delivered` (Giao hàng thành công).
- Quá 7 ngày hệ thống tự động khóa tính năng yêu cầu đổi trả.
- Sản phẩm được đổi size phải còn nguyên tem mác, không thêu tên riêng.

---

## 6. Giả định về Quản lý tồn kho
- Ngưỡng cảnh báo tồn kho thấp mặc định: **5 sản phẩm / biến thể (size, màu)**.
- Cơ chế giữ chỗ tồn kho cho đơn hàng MoMo: Giữ chỗ tối đa **15 phút**. Nếu sau 15 phút khách hàng chưa hoàn tất thanh toán, hệ thống tự động hoàn lại số lượng tồn kho và hủy phiên thanh toán.

---

## 7. Giả định về Phân quyền Quản trị (RBAC) & Nhật ký Hoạt động (F22, F23)
- **Vai trò Quản trị viên (`admin`)**: Có toàn quyền trên hệ thống, bao gồm truy cập báo cáo doanh thu, xuất dữ liệu tài chính dạng CSV, duyệt võ đường, hoàn tiền thủ công, phân quyền tài khoản và tra cứu toàn bộ nhật ký kiểm toán hệ thống.
- **Vai trò Nhân viên vận hành (`staff`)**: Chỉ được phép thao tác các nghiệp vụ đóng gói hàng, điều chỉnh kho kiểm kê, cập nhật trạng thái giao nhận và duyệt nhận hàng đổi size; **bắt buộc bị chặn (403 Forbidden) khi truy cập báo cáo doanh thu tài chính và nhật ký hoạt động hệ thống**.
- **Nhật ký kiểm toán (`activity_logs`)**: Tự động lưu trữ bất biến các hành động quan trọng gồm: `ADMIN_LOGIN`, `CHANGE_ORDER_STATUS`, `UPDATE_PRODUCT_PRICE`, `ADJUST_INVENTORY`, `REFUND_ORDER`, `APPROVE_RETURN`, `RECEIVE_RETURN`, `APPROVE_DOJO`, `CHANGE_USER_ROLE`.

---

## 8. Giả định về An toàn Thông tin & Tối ưu hóa Tìm kiếm (Mục 9 & F24)
- **Bảo vệ tiêu đề phản hồi (Security Headers OWASP)**: Toàn bộ phản hồi HTTP tự động được đính kèm các trường tiêu đề bảo mật: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `X-XSS-Protection: 1; mode=block`, `Strict-Transport-Security` (HSTS), `Content-Security-Policy` (CSP), `Referrer-Policy: strict-origin-when-cross-origin`.
- **Giới hạn tần suất yêu cầu (Rate Limiting)**: Áp dụng thuật toán trượt thời gian 60 giây (Sliding Window): tối đa 120 yêu cầu/phút cho các API công khai; tối đa 15 yêu cầu/phút cho các endpoint xác thực nhạy cảm (`/auth/login`, `/auth/register`, `/auth/forgot-password`). Khi vượt ngưỡng, hệ thống từ chối với mã HTTP 429 và thông báo thời gian chờ qua tiêu đề `Retry-After: 60`.
- **Làm sạch dữ liệu đầu vào (Input Sanitization)**: Khử thẻ script, thuộc tính sự kiện nguy hiểm (onerror, onload), liên kết pseudo-protocol (`javascript:`) để chống XSS; loại bỏ ký tự toán tử MongoDB (`$where`, `$gt`...) để chống NoSQL Injection.
- **Che giấu dữ liệu nhạy cảm (Data Masking)**: Che phủ tuyệt đối (`***MASKED***`) các trường `password`, `hashed_password`, `secret_key`, `token` trước khi in ra log hoặc truyền qua phản hồi API.
- **Cấu trúc dữ liệu SEO (Schema.org & OpenGraph)**: Cung cấp đầy đủ sitemap động chuẩn XML Schema tại `/sitemap.xml`, quy tắc thu thập thông tin tại `/robots.txt`, siêu dữ liệu mạng xã hội (OpenGraph, Twitter Card) và dữ liệu có cấu trúc JSON-LD (`Product`, `BreadcrumbList`, `LocalBusiness`).


