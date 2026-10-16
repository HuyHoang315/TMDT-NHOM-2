# Tài liệu Đặc tả Yêu cầu Hệ thống (docs/01-yeu-cau.md)

> **Dự án**: Website Thương mại điện tử Võ phục & Dụng cụ Võ thuật "Đai Trắng" (Mô hình B2C & B2B)  
> **Cơ sở dữ liệu**: MongoDB Atlas  
> **Backend**: Python FastAPI  
> **Frontend**: Next.js & Bootstrap/Tailwind  

---

## 1. Danh sách Tác nhân Hệ thống (Actors)

| Tác nhân | Vai trò và Mô tả quyền hạn |
|---|---|
| **Khách vãng lai** | Người dùng chưa đăng nhập: Xem sản phẩm, tìm kiếm, lọc, sử dụng công cụ tư vấn size, thêm sản phẩm vào giỏ hàng phiên. |
| **Khách hàng** | Phụ huynh hoặc võ sinh có tài khoản: Đặt hàng, thanh toán COD/MoMo, lưu hồ sơ size võ sinh, theo dõi hành trình đơn hàng, gửi yêu cầu đổi size trong 7 ngày. |
| **Huấn luyện viên / Võ đường (B2B)** | Đại diện câu lạc bộ/võ đường: Đăng ký thông tin võ đường, tạo các đợt gom đơn chung cho học viên, theo dõi danh sách đặt hàng, chốt đợt và nhận chiết khấu sỉ theo bậc số lượng. |
| **Nhân viên vận hành (Staff)** | Tiếp nhận và xử lý đơn hàng, quản lý kho, nhập/điều chỉnh tồn kho, xử lý yêu cầu đổi trả hàng. Không xem được báo cáo doanh thu tài chính. |
| **Quản trị viên (Admin)** | Toàn quyền kiểm soát hệ thống: Quản lý sản phẩm, danh mục, bảng size, duyệt võ đường, phân quyền, cấu hình hệ thống, xem báo cáo doanh thu và nhật ký hoạt động. |
| **Hệ thống MoMo** | Cổng thanh toán trực tuyến bên ngoài xử lý giao dịch điện tử và gửi thông báo IPN xác nhận thanh toán. |
| **Đơn vị vận chuyển** | Hệ thống đối tác giao nhận bên ngoài (mô phỏng Mock Shipping Provider) tiếp nhận đơn, tạo mã vận đơn và cập nhật trạng thái giao hàng. |

---

## 2. Bảng Yêu cầu Chức năng (F01 – F24)

### Phân hệ Khách hàng & Bán lẻ (B2C)
- **F01. Quản lý tài khoản**: Đăng ký, đăng nhập (bảo vệ chống brute force khóa sau 5 lần sai trong 15 phút), đăng xuất, quên mật khẩu (môi trường dev in token/email ra console).
- **F02. Xem danh mục & tìm kiếm**: Xem danh mục theo môn võ (Vovinam, Taekwondo, Karate, Võ cổ truyền) và loại sản phẩm (võ phục, đai, bảo hộ, túi đựng, phụ kiện); bộ lọc đa tiêu chí; tìm kiếm tên; sắp xếp theo giá và độ mới.
- **F03. Chi tiết sản phẩm**: Xem thư viện hình ảnh, mô tả chi tiết, bảng size chuẩn, chọn biến thể (kích thước, màu sắc), hiển thị tình trạng tồn kho tức thời.
- **F04. Công cụ tư vấn size thông minh**: Nhập chiều cao (cm), cân nặng (kg), độ tuổi, môn võ; hệ thống tự động tính toán đối chiếu bảng size trong CSDL để gợi ý size phù hợp (kèm khuyến nghị size lớn hơn nếu số đo nằm giữa hai size).
- **F05. Quản lý hồ sơ size**: Khách hàng lưu nhiều hồ sơ cho các con/võ sinh; hệ thống gửi cảnh báo nhắc đo lại kích thước khi hồ sơ quá 6 tháng.
- **F06. Giỏ hàng**: Thêm, sửa số lượng, xóa; hỗ trợ giỏ hàng theo phiên cho khách chưa đăng nhập và tự động đồng bộ (merge) vào tài khoản sau khi đăng nhập; kiểm tra tồn kho tại thời điểm thêm và checkout.
- **F07. Dịch vụ thêu tên riêng**: Tùy chọn thêu tên võ sinh (tối đa 20 ký tự, có dấu tiếng Việt) cho từng dòng sản phẩm có tính phí; đơn hàng có thêu tên tự động khóa phương thức COD.
- **F08. Quy trình đặt hàng & thanh toán**: Nhập thông tin giao hàng, tính phí vận chuyển theo vùng; chọn thanh toán COD (trừ tồn kho ngay) hoặc MoMo (giữ chỗ tồn kho 15 phút, quá hạn tự động hoàn kho).
- **F09. Theo dõi đơn hàng**: Xem danh sách đơn, chi tiết đơn, mã vận đơn và lịch sử thay đổi trạng thái theo thời gian thực.
- **F10. Đổi size & Trả hàng**: Gửi yêu cầu đổi size trực tuyến trong vòng 7 ngày kể từ khi đơn hàng giao thành công; chọn size mới và đính kèm hình ảnh minh chứng.

### Phân hệ Võ đường & Bán sỉ (B2B)
- **F11. Đăng ký tài khoản Võ đường**: Nhập thông tin CLB, môn võ, địa chỉ, người đại diện, logo; quản trị viên duyệt trước khi kích hoạt.
- **F12. Tạo đợt đặt hàng chung (Campaign)**: HLV chọn danh mục sản phẩm áp dụng, số lượng dự kiến, thời hạn chốt; hệ thống xác định bậc giá sỉ và sinh đường dẫn riêng kèm mã QR.
- **F13. Phụ huynh đặt hàng theo đợt**: Truy cập link/QR đợt hàng, chọn sản phẩm, chọn size (có tư vấn), nhập tên võ sinh; thanh toán MoMo hoặc chọn hình thức võ đường thanh toán gộp.
- **F14. Quản lý & Theo dõi đợt gom đơn**: HLV theo dõi danh sách đơn học viên theo thời gian thực, xuất file danh sách CSV; khi đóng đợt, hệ thống tự động gom thành 1 lô hàng giao về địa chỉ võ đường.
- **F15. Thanh toán gộp đơn B2B**: Võ đường thanh toán đặt cọc 50% qua MoMo khi đóng đợt; phần còn lại ghi nhận khi giao hàng tận nơi.

### Phân hệ Quản trị & Vận hành (Staff & Admin)
- **F16. Quản lý danh mục & sản phẩm**: Quản lý môn võ, loại sản phẩm, sản phẩm, các biến thể SKU, giá bán, bảng size.
- **F17. Quản lý kho hàng & tồn kho**: Nhập kho, điều chỉnh tồn kho thủ công, ghi nhận lịch sử biến động kho (`StockMovement`), cảnh báo tồn kho thấp theo ngưỡng cài đặt.
- **F18. Xử lý & Quản lý đơn hàng**: Lọc đơn theo trạng thái, chuyển trạng thái theo luồng hợp lệ nghiêm ngặt, in phiếu đóng gói, tạo vận đơn giao hàng.
- **F19. Đối soát & Quản lý thanh toán**: Theo dõi giao dịch thanh toán MoMo, đối soát mã đơn hàng, đánh dấu hoàn tiền thủ công khi hủy đơn.
- **F20. Xử lý yêu cầu đổi trả**: Duyệt/từ chối yêu cầu đổi size từ khách hàng; tự động tạo đơn đổi mới và hoàn nhập kho khi nhận lại hàng trả.
- **F21. Quản lý Võ đường & Đợt đặt hàng**: Duyệt hồ sơ đăng ký võ đường, giám sát và đóng đợt gom đơn sỉ.
- **F22. Báo cáo & Thống kê kinh doanh**: Báo cáo doanh thu theo ngày/tháng, doanh thu theo kênh (B2C bán lẻ vs B2B võ đường), top sản phẩm bán chạy, tỷ lệ đổi trả hàng, xuất dữ liệu CSV.
- **F23. Phân quyền (RBAC) & Nhật ký hoạt động (Audit Log)**: Quản trị viên toàn quyền; Nhân viên chỉ thao tác đơn, kho, đổi trả, bị chặn truy cập báo cáo doanh thu; Ghi nhật ký mọi hành động quan trọng (đăng nhập quản trị, đổi giá, chuyển trạng thái đơn, điều chỉnh kho).
- **F24. Tối ưu SEO kỹ thuật**: URL thân thiện slug tiếng Việt không dấu, thẻ title/meta description động, sinh tự động `sitemap.xml`, `robots.txt`, dữ liệu cấu trúc Schema.org Product (JSON-LD), hình ảnh chuẩn thuộc tính alt.

---

## 3. Yêu cầu Phi chức năng

1. **Hiệu năng & Tốc độ đáp ứng**:
   - Thời gian phản hồi API trung bình dưới 200ms cho các truy vấn xem danh mục và chi tiết sản phẩm.
   - Hỗ trợ cơ chế khóa bản ghi / atomic update trên MongoDB Atlas để ngăn chặn tình trạng Race Condition (bán vượt tồn kho khi có nhiều người đặt hàng đồng thời).
2. **Bảo mật dữ liệu**:
   - Mật khẩu mã hóa bằng thuật toán bcrypt an toàn.
   - Token JWT ký bằng khóa bí mật chuẩn HS256, thời hạn 24 giờ.
   - Cơ chế bảo vệ Brute Force: Khóa tài khoản tạm thời 15 phút sau 5 lần nhập sai liên tiếp.
   - Phòng chống các lỗ hổng OWASP Top 10: NoSQL Injection, XSS, CSRF, IDOR.
3. **Độ tin cậy & Tính toàn vẹn**:
   - Cơ chế Idempotent xử lý IPN MoMo: Nếu MoMo gửi lặp lại webhook IPN nhiều lần, hệ thống chỉ ghi nhận và xử lý cộng trừ trạng thái đơn hàng đúng một lần duy nhất.
4. **Khả năng mở rộng**:
   - Kiến trúc module hóa rõ ràng theo chuẩn Domain-Driven Design (DDD), dễ dàng tích hợp thêm cổng thanh toán mới (VNPay, ZaloPay) hoặc đơn vị vận chuyển thật (GHN, GHTK).
