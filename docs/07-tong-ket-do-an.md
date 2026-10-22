# Báo Cáo Tổng Kết Đồ Án Thương Mại Điện Tử (docs/07-tong-ket-do-an.md)

---

## I. THÔNG TIN CHUNG DỰ ÁN

- **Tên dự án**: Nền tảng Thương mại Điện tử Võ phục & Phụ kiện Võ thuật **"ĐAI TRẮNG"**
- **Mô hình kinh doanh**: Kết hợp **B2C** (Bán lẻ cho võ sinh/phụ huynh) và **B2B / B2B2C** (Gom đơn giá sỉ theo đợt cho các Võ đường & Câu lạc bộ võ thuật).
- **Phạm vi 4 môn phái võ thuật**:
  1. **Vovinam - Việt Võ Đạo** (Võ phục xanh dương, đai lam/vàng/đỏ).
  2. **Taekwondo** (Võ phục cổ trắng/cổ đen, vải gân kim cương).
  3. **Karate** (Võ phục Kata dày đứng form, Kumite mỏng nhẹ).
  4. **Võ Cổ Truyền Việt Nam** (Võ phục đen truyền thống, nút thắt nút tàu).
- **Nền tảng công nghệ**:
  - **Cơ sở dữ liệu**: MongoDB Atlas Cloud (sử dụng driver `motor` async và `pymongo`, fallback tự động sang `mongomock_motor` khi chạy kiểm thử độc lập).
  - **Backend**: Python 3.9+ FastAPI, kiến trúc Modular Monolith hướng Domain-Driven Design (DDD).
  - **Frontend**: Next.js 14 App Router, React, Tailwind CSS, TypeScript.
  - **Thanh toán**: Tích hợp MoMo Payment Gateway v2 (ký số HMAC-SHA256, hỗ trợ chế độ MoMo Mock cho demo/test offline).
  - **Kiểm thử tự động**: Pytest + pytest-asyncio + HTTPX AsyncClient.

---

## II. MA TRẬN 24 TÍNH NĂNG NGHIỆP VỤ (F01 – F24)

| Mã | Tên tính năng nghiệp vụ | Vị trí cài đặt (Backend API) | Ràng buộc nghiệp vụ đã xử lý | Trạng thái |
|:---:|---|---|---|:---:|
| **F01** | Đăng ký, Đăng nhập, Tài khoản & Bảo mật | `app/modules/auth/router.py` | Băm bcrypt, khóa tài khoản 15 phút sau 5 lần sai liên tiếp, OTP console dev. | ✅ Hoàn thành |
| **F02** | Danh mục & Bộ lọc sản phẩm đa môn võ | `app/modules/products/router.py` | Lọc theo 4 môn võ, loại sản phẩm, khoảng giá, kích cỡ, màu sắc, sắp xếp. | ✅ Hoàn thành |
| **F03** | Chi tiết sản phẩm & Bảng size tích hợp | `app/modules/products/router.py` | Biến thể tồn kho (in_stock/low_stock), tự động kèm bảng size chuẩn theo môn võ. | ✅ Hoàn thành |
| **F04** | Tư vấn chọn size thông minh | `app/modules/size_advisor/router.py` | Đối chiếu nhân trắc học (chiều cao, cân nặng, môn phái), chọn size lớn hơn khi nằm giữa 2 size, xử lý ngoài bảng size. | ✅ Hoàn thành |
| **F05** | Hồ sơ kích cỡ võ sinh | `app/modules/size_advisor/router.py` | Lưu hồ sơ người thân/võ sinh nhí, cảnh báo nhắc đo lại kích cỡ khi hồ sơ quá 6 tháng (> 180 ngày). | ✅ Hoàn thành |
| **F06** | Giỏ hàng vãng lai & Thành viên | `app/modules/cart/router.py` | Giỏ hàng vãng lai theo session_id, tự động gộp giỏ hàng (`/cart/merge`) khi đăng nhập, kiểm tra tồn kho tức thời. | ✅ Hoàn thành |
| **F07** | Dịch vụ Thêu tên riêng lên võ phục | `app/modules/orders/router.py` | Tối đa 20 ký tự tiếng Việt, phí 30.000 VND/bộ; **bắt buộc thanh toán trước (MoMo), chặn COD, không áp dụng đổi trả**. | ✅ Hoàn thành |
| **F08** | Đặt hàng & Trừ tồn kho nguyên tử | `app/modules/orders/router.py` | Mã đơn `DT-YYYYMMDD-XXXX`, atomic update trừ kho chống oversell, MockShippingProvider (HCM 25k, Tỉnh 40k, Freeship >= 500k). | ✅ Hoàn thành |
| **F09** | Theo dõi đơn hàng & Chống IDOR | `app/modules/orders/router.py` | Tra cứu theo mã đơn, khách đã đăng nhập chỉ xem đơn của mình, khách vãng lai xác thực số điện thoại. | ✅ Hoàn thành |
| **F10** | Yêu cầu Đổi size sản phẩm | `app/modules/admin/router.py` | Áp dụng trong vòng **7 ngày** từ khi đơn `delivered`; **tuyệt đối từ chối đổi trả nếu sản phẩm có thêu tên riêng**. | ✅ Hoàn thành |
| **F11** | Đăng ký & Duyệt CLB / Võ đường | `app/modules/dojos/router.py` | Đăng ký thông tin võ đường, Admin phê duyệt kích hoạt tài khoản HLV. | ✅ Hoàn thành |
| **F12** | Tạo đợt gom đơn võ đường (Campaign) | `app/modules/dojos/router.py` | Tự động tính chiết khấu sỉ 10–25% theo số lượng dự kiến, sinh link chia sẻ và mã QR cho phụ huynh. | ✅ Hoàn thành |
| **F13** | Phụ huynh đặt hàng theo đợt gom | `app/modules/dojos/router.py` | Hưởng giá sỉ theo đợt, tùy chọn MoMo cá nhân hoặc CLB thanh toán gộp (`CLUB_CONSOLIDATED`). | ✅ Hoàn thành |
| **F14** | HLV theo dõi đợt & Xuất file CSV | `app/modules/dojos/router.py` | Xuất danh sách đo size & thêu tên dạng CSV chuẩn **UTF-8 BOM (`\ufeff`)**, đóng đợt tạo lô hàng chung (`consignment_batch`). | ✅ Hoàn thành |
| **F15** | Đặt cọc đợt hàng chung B2B | `app/modules/dojos/router.py` | Võ đường đặt cọc 50% qua MoMo khi đóng đợt; ghi nhận 50% còn lại khi giao hàng tận nơi. | ✅ Hoàn thành |
| **F16** | Quản lý sản phẩm & Bảng size (Staff/Admin) | `app/modules/products/router.py` | Thêm, sửa, xóa sản phẩm và bảng size chuẩn; chặn Customer 403 Forbidden. | ✅ Hoàn thành |
| **F17** | Quản lý kho & Biến động tồn kho | `app/modules/orders/router.py` | Điều chỉnh tồn kho thủ công, cảnh báo tồn kho thấp (<= 5), lưu lịch sử biến động `StockMovement`. | ✅ Hoàn thành |
| **F18** | In Phiếu đóng gói kiện hàng (Packing Slip) | `app/modules/admin/router.py` | Thông tin người nhận, SKU, size, đơn vị vận chuyển, tự động cảnh báo: "ĐƠN CÓ HÀNG THÊU TÊN - KIỂM TRA KỸ". | ✅ Hoàn thành |
| **F19** | Quản trị Thanh toán & Đối soát MoMo | `app/modules/payments/router.py` | Danh sách giao dịch MoMo, đối soát mã đơn và số tiền, đánh dấu hoàn tiền thủ công cho Admin. | ✅ Hoàn thành |
| **F20** | Quản lý Đổi trả & Hoàn kho | `app/modules/admin/router.py` | Phê duyệt/từ chối đổi size, nhận hàng trả về kho -> **tự động cộng hoàn tồn kho (+1)** cho SKU cũ. | ✅ Hoàn thành |
| **F21** | Đánh giá & Nhận xét sản phẩm | `app/modules/reviews/router.py` | Đánh giá 1-5 sao kèm ảnh chụp thực tế, kiểm tra điều kiện đã từng mua hàng thành công. | ✅ Hoàn thành |
| **F22** | Báo cáo Doanh thu & Thống kê Kinh doanh | `app/modules/admin/router.py` | Doanh thu theo kênh (Bán lẻ vs Võ đường), theo ngày/tháng, top sản phẩm, xuất CSV UTF-8 BOM; **Chặn Staff 403**. | ✅ Hoàn thành |
| **F23** | Phân quyền RBAC & Audit Logging | `app/modules/admin/router.py`, `app/core/activity.py` | Phân quyền chặt chẽ Admin/Staff; lưu vết bất biến `activity_logs`: đăng nhập, đổi trạng thái đơn, sửa giá, chỉnh kho, hoàn tiền. | ✅ Hoàn thành |
| **F24** | Tối ưu hóa Tìm kiếm (SEO & Schema.org) | `app/modules/seo/router.py` | Dynamic `sitemap.xml`, `robots.txt`, siêu dữ liệu OpenGraph/Twitter Card, dữ liệu có cấu trúc JSON-LD `Product`, `BreadcrumbList`, `LocalBusiness`. | ✅ Hoàn thành |

---

## III. MA TRẬN KIỂM THỬ TỰ ĐỘNG (AUTOMATED TEST MATRIX)

Toàn bộ **43 bài kiểm thử** tích hợp và đơn vị đều được viết bằng `pytest` và chạy tự động với kết quả **100% PASS**:

```text
============================= test session starts =============================
platform win32 -- Python 3.9.13, pytest-8.4.2, pluggy-1.6.0
rootdir: TMDT-NHOM-2\backend
plugins: anyio-4.12.1, asyncio-1.2.0

tests/test_auth_f01.py::test_register_success PASSED                     [  2%]
tests/test_auth_f01.py::test_register_duplicate_phone PASSED             [  4%]
tests/test_auth_f01.py::test_register_short_password PASSED              [  6%]
tests/test_auth_f01.py::test_login_success_and_me PASSED                 [  9%]
tests/test_auth_f01.py::test_login_failed_lockout_after_5_attempts PASSED [ 11%]
tests/test_auth_f01.py::test_forgot_and_reset_password_flow PASSED       [ 13%]
tests/test_auth_f01.py::test_logout PASSED                               [ 16%]
tests/test_stage2_catalog_advisor.py::test_f02_list_and_filter_products PASSED [ 18%]
tests/test_stage2_catalog_advisor.py::test_f03_product_detail_and_size_chart PASSED [ 20%]
tests/test_stage2_catalog_advisor.py::test_f04_size_advisor_logic PASSED [ 23%]
tests/test_stage2_catalog_advisor.py::test_f05_student_profiles_and_6_months_alert PASSED [ 25%]
tests/test_stage2_catalog_advisor.py::test_f16_admin_product_management_and_rbac PASSED [ 27%]
tests/test_stage3_cart_orders_cod.py::test_f06_cart_crud_and_merge PASSED [ 30%]
tests/test_stage3_cart_orders_cod.py::test_f07_embroidery_validation_and_cod_rejection PASSED [ 32%]
tests/test_stage3_cart_orders_cod.py::test_f08_cod_checkout_and_atomic_stock_deduction PASSED [ 34%]
tests/test_stage3_cart_orders_cod.py::test_section6_order_status_transitions_and_cancel_restore_stock PASSED [ 37%]
tests/test_stage3_cart_orders_cod.py::test_f09_order_tracking_and_idor_protection PASSED [ 39%]
tests/test_stage3_cart_orders_cod.py::test_f17_inventory_adjust_and_low_stock_alert PASSED [ 41%]
tests/test_stage4_momo_payments.py::test_momo_create_payment_request PASSED [ 44%]
tests/test_stage4_momo_payments.py::test_momo_ipn_invalid_signature_rejected PASSED [ 46%]
tests/test_stage4_momo_payments.py::test_momo_ipn_amount_mismatch_rejected PASSED [ 48%]
tests/test_stage4_momo_payments.py::test_momo_ipn_success_deducts_stock_and_records_payment PASSED [ 51%]
tests/test_stage4_momo_payments.py::test_momo_ipn_idempotency_safe_replay PASSED [ 53%]
tests/test_stage4_momo_payments.py::test_momo_ipn_failed_cancels_order PASSED [ 55%]
tests/test_stage4_momo_payments.py::test_momo_auto_cancel_15_minutes_expired PASSED [ 58%]
tests/test_stage4_momo_payments.py::test_f19_payment_transactions_and_manual_refund PASSED [ 60%]
tests/test_stage5_dojos_b2b.py::test_f11_club_registration_and_approval PASSED [ 62%]
tests/test_stage5_dojos_b2b.py::test_f12_create_campaign_and_discount_tier PASSED [ 65%]
tests/test_stage5_dojos_b2b.py::test_f13_parent_order_in_campaign PASSED [ 67%]
tests/test_stage5_dojos_b2b.py::test_f14_campaign_orders_and_export_csv PASSED [ 69%]
tests/test_stage5_dojos_b2b.py::test_f14_and_f15_close_campaign_consignment_and_deposit_flow PASSED [ 72%]
tests/test_stage6_admin_returns_reports.py::test_f10_return_request_success_and_restrictions PASSED [ 74%]
tests/test_stage6_admin_returns_reports.py::test_f20_admin_review_and_receive_return PASSED [ 76%]
tests/test_stage6_admin_returns_reports.py::test_f18_packing_slip_generation PASSED [ 79%]
tests/test_stage6_admin_returns_reports.py::test_f22_f23_revenue_report_and_rbac_staff_blocking PASSED [ 81%]
tests/test_stage6_admin_returns_reports.py::test_f23_activity_logs_and_user_role_management PASSED [ 83%]
tests/test_stage7_security_and_seo.py::test_security_headers_owasp_standards PASSED [ 86%]
tests/test_stage7_security_and_seo.py::test_rate_limiting_auth_endpoint PASSED [ 88%]
tests/test_stage7_security_and_seo.py::test_input_sanitization_xss_nosql_and_masking PASSED [ 90%]
tests/test_stage7_security_and_seo.py::test_f24_seo_sitemap_xml PASSED   [ 93%]
tests/test_stage7_security_and_seo.py::test_f24_seo_robots_txt PASSED    [ 95%]
tests/test_stage7_security_and_seo.py::test_f24_seo_product_metadata_and_structured_data PASSED [ 97%]
tests/test_stage7_security_and_seo.py::test_f24_seo_organization_meta PASSED [100%]

============================= 43 passed in 6.05s ==============================
```

---

## IV. HƯỚNG DẪN CHẤM BÀI & TÀI KHOẢN DEMO

### 1. Tài khoản Demo có sẵn trong hệ thống (`app/seed_mongo.py`)

| Vai trò | Số điện thoại | Mật khẩu | Quyền hạn chính |
|---|---|---|---|
| **Quản trị viên (Admin)** | `0988000001` | `AdminPass123` | Toàn quyền, xem báo cáo doanh thu tài chính, xuất CSV, phân quyền tài khoản, xem Audit Log. |
| **Nhân viên (Staff)** | `0988000002` | `StaffPass123` | Xử lý đơn hàng, in phiếu đóng gói, điều chỉnh kho, duyệt đổi trả; **Bị chặn 403 khi xem doanh thu**. |
| **Huấn luyện viên (Coach)** | `0988000003` | `CoachPass123` | Tạo đợt gom đơn cho CLB Vovinam Lam Sơn, theo dõi phụ huynh đặt hàng, xuất danh sách size CSV, đặt cọc 50%. |
| **Khách hàng (Customer 1)** | `0988000004` | `UserPass123` | Mua hàng bán lẻ, tư vấn chọn size thông minh, lưu hồ sơ size con cái, đặt thêu tên riêng. |
| **Khách hàng (Customer 2)** | `0988000005` | `UserPass123` | Phụ huynh tham gia đợt gom đơn của võ đường để nhận chiết khấu sỉ 20%. |

### 2. Kịch bản Demo 5 phút dành cho Giảng viên

1. **Khách hàng B2C mua võ phục & Tư vấn size**:
   - Truy cập trang sản phẩm `/products/vo-phuc-vovinam-tieu-chuan`.
   - Mở công cụ Tư vấn Size: nhập Chiều cao 163cm, Cân nặng 58kg -> Hệ thống tự động gợi ý **Size 3 (160-168cm)**.
   - Chọn dịch vụ **Thêu tên**: nhập "NGUYEN ANH KHOA" (+30.000 VND).
   - Hệ thống tự động phát hiện hàng thêu: **Ẩn tùy chọn COD, bắt buộc chọn MoMo**.
2. **Khách hàng đặt hàng & Thanh toán MoMo**:
   - Nhấn Đặt hàng -> Sinh đơn hàng `DT-YYYYMMDD-XXXX`.
   - Chuyển hướng thanh toán MoMo (chế độ Mock mô phỏng thành công).
   - Webhook IPN nhận kết quả -> Trừ tồn kho nguyên tử, chuyển đơn sang `paid`.
3. **Huấn luyện viên B2B tạo đợt gom đơn**:
   - Đăng nhập tài khoản HLV `0988000003`.
   - Tạo đợt gom cho CLB Lam Sơn: số lượng dự kiến 50 bộ -> Tự động kích hoạt **Bậc giảm giá 20%**.
   - Xuất file CSV danh sách size và thêu tên (mở trực tiếp trên Microsoft Excel chuẩn dấu tiếng Việt nhờ UTF-8 BOM).
   - HLV đóng đợt và đặt cọc 50% qua MoMo.
4. **Nhân viên & Quản trị viên Back-office**:
   - Đăng nhập Staff `0988000002`: Xem đơn, in Phiếu đóng gói (Packing Slip), duyệt yêu cầu đổi size.
   - Thử truy cập `/admin/reports/revenue` bằng Staff -> **Bị chặn 403 Forbidden**.
   - Đăng nhập Admin `0988000001`: Xem toàn bộ biểu đồ doanh thu, top sản phẩm bán chạy, xuất báo cáo CSV và tra cứu nhật ký kiểm toán `activity_logs`.
