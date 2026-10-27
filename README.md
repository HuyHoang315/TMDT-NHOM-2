# ĐAI TRẮNG – NỀN TẢNG THƯƠNG MẠI ĐIỆN TỬ VÕ PHỤC & DỤNG CỤ VÕ THUẬT (B2C & B2B)

> **Đồ án môn học**: Thương mại Điện tử (TMĐT)  
> **Nhóm thực hiện**: Nhóm 2  
> **Trạng thái**: Hoàn thành toàn diện 8 Giai đoạn phát triển (F01 – F24)  
> **Cơ sở dữ liệu chính thức**: **MongoDB Atlas Cloud Database**  
> **Bộ kiểm thử tự động**: 43/43 bài test đạt tỷ lệ **100% PASS**

---

## 1. Giới thiệu Dự án

**"Đai Trắng"** là hệ thống thương mại điện tử chuyên sâu cho thị trường võ thuật Việt Nam, phục vụ **4 môn phái tiêu biểu**:
1. **Vovinam - Việt Võ Đạo** (Võ phục lam truyền thống, đai các cấp).
2. **Taekwondo** (Võ phục quyền Poomsae và đối kháng Kyorugi, vải gân kim cương).
3. **Karate** (Võ phục Kata dày đứng form và Kumite siêu nhẹ).
4. **Võ Cổ Truyền Việt Nam** (Võ phục đen truyền thống, nút thắt tàu, đai ngũ sắc).

### Điểm khác biệt cốt lõi:
- **Mô hình kết hợp B2C & B2B2C**: Vừa bán lẻ cho võ sinh, vừa cung cấp giải pháp **gom đơn đặt hàng số lượng lớn theo đợt cho các Võ đường & Câu lạc bộ võ thuật** với chiết khấu sỉ tự động từ 10% đến 25%.
- **Tư vấn chọn size thông minh (F04)**: Tự động đối chiếu số đo nhân trắc học (chiều cao, cân nặng, môn phái) với cơ sở dữ liệu bảng size trên MongoDB Atlas; tự động chọn size lớn hơn khi số đo nằm giữa 2 size; cảnh báo nhắc đo lại size sau 6 tháng.
- **Dịch vụ Thêu tên riêng theo yêu cầu (F07)**: Cá nhân hóa tên võ sinh và võ đường; tự động áp dụng ràng buộc thanh toán trước (MoMo), chặn COD và khóa quyền đổi trả đối với hàng đã thêu.
- **Quản lý tồn kho nguyên tử (Atomic Update - F08)**: Trừ tồn kho đồng thời an toàn trên MongoDB Atlas, triệt tiêu hoàn toàn rủi ro bán vượt tồn kho (oversell) khi có hàng trăm võ sinh cùng đặt hàng.
- **Bảo mật & Tối ưu hóa SEO chuẩn công nghiệp (Mục 9 & F24)**: Tiêu đề bảo mật OWASP, Rate Limiting chống Brute-force/DDoS, chống XSS & NoSQL Injection, Dynamic Sitemap XML, Schema.org JSON-LD structured data.

---

## 2. Kiến trúc & Công nghệ Sử dụng

```text
TMDT-NHOM-2/
├── backend/                  # Python FastAPI Backend
│   ├── app/
│   │   ├── core/             # Database (MongoDB Atlas), Config, Security, Middleware, Activity Log
│   │   ├── modules/          # 9 Domain Modules (Auth, Products, Size Advisor, Cart, Orders,
│   │   │                     #                    Payments, Dojos, Shipping, Admin, SEO)
│   │   └── seed_mongo.py     # Script khởi tạo dữ liệu mẫu phong phú lên MongoDB Atlas
│   └── tests/                # 7 bộ kiểm thử tự động tương ứng với các giai đoạn (43 tests)
├── frontend/                 # Next.js 14 Web Storefront & Back-office UI
│   ├── src/
│   │   ├── app/              # Next.js App Router (Products, Size Advisor, Dojos, Cart, Admin)
│   │   ├── components/       # Reusable UI components
│   │   └── lib/              # API client, Context, Utilities
└── docs/                     # Toàn bộ hồ sơ tài liệu thiết kế hệ thống
    ├── 01-yeu-cau.md         # Đặc tả yêu cầu nghiệp vụ B2C & B2B
    ├── 02-use-case.md        # Biểu đồ Use Case tổng quát & đặc tả chi tiết
    ├── 03-hoat-dong-tuan-tu.md # Biểu đồ hoạt động và tuần tự (Activity & Sequence)
    ├── 04-kien-truc.md       # Kiến trúc hệ thống tổng thể 3-Tier / Modular Monolith
    ├── 05-csdl.md            # Thiết kế cơ sở dữ liệu MongoDB Atlas (Collection & Index)
    ├── 06-bao-mat.md         # Ma trận an ninh bảo mật Mục 9
    ├── 07-tong-ket-do-an.md  # Báo cáo nghiệm thu 24 tính năng & ma trận kiểm thử
    └── gia-dinh.md           # Bảng tổng hợp các giả định nghiệp vụ cốt lõi
```

- **Backend Framework**: Python 3.9+, [FastAPI](https://fastapi.tiangolo.com/), Uvicorn.
- **Cơ sở dữ liệu**: [MongoDB Atlas](https://www.mongodb.com/atlas) Cloud NoSQL via `motor` (Async driver) và `pymongo`. Tự động fallback sang `mongomock_motor` khi chạy kiểm thử độc lập.
- **Frontend Framework**: [Next.js 14](https://nextjs.org/) (App Router), React 18, TypeScript, Tailwind CSS.
- **Cổng thanh toán**: Tích hợp MoMo API v2 (HMAC-SHA256 Signature Verification, Mock IPN Runner).
- **Kiểm thử tự động**: Pytest, pytest-asyncio, HTTPX.

---

## 3. Hướng dẫn Cài đặt & Khởi chạy Nhanh

### Bước 1: Cài đặt và Chạy Backend (Python FastAPI)

```bash
# 1. Di chuyển vào thư mục backend
cd TMDT-NHOM-2/backend

# 2. Tạo môi trường ảo Python và kích hoạt
python -m venv .venv
# Trên Windows PowerShell:
.\.venv\Scripts\Activate.ps1
# Trên Linux/macOS:
# source .venv/bin/activate

# 3. Cài đặt các thư viện phụ thuộc
pip install -r requirements.txt

# 4. Cấu hình biến môi trường
# File .env đã được tạo sẵn mẫu kết nối MongoDB Atlas:
# MONGODB_URI=mongodb+srv://...
# MOMO_MOCK=True

# 5. Nạp dữ liệu mẫu phong phú lên MongoDB Atlas
python app/seed_mongo.py

# 6. Khởi động Backend Server
uvicorn app.main:app --reload --port 8000
```
- **Swagger API Docs tương tác**: [http://localhost:8000/docs](http://localhost:8000/docs)
- **ReDoc Documentation**: [http://localhost:8000/redoc](http://localhost:8000/redoc)
- **Dynamic Sitemap XML (F24)**: [http://localhost:8000/sitemap.xml](http://localhost:8000/sitemap.xml)
- **Robots.txt (F24)**: [http://localhost:8000/robots.txt](http://localhost:8000/robots.txt)

---

### Bước 2: Chạy Bộ Kiểm thử Tự động (43/43 Tests PASS)

```bash
cd TMDT-NHOM-2/backend
.\.venv\Scripts\python.exe -m pytest tests/ -v
```
Kết quả chạy kiểm thử tự động xác nhận toàn bộ 43 ca kiểm thử từ Giai đoạn 1 đến Giai đoạn 7 đều **PASSED 100%**.

---

### Bước 3: Cài đặt và Chạy Frontend (Next.js)

```bash
# 1. Di chuyển vào thư mục frontend
cd TMDT-NHOM-2/frontend

# 2. Cài đặt các gói phụ thuộc
npm install

# 3. Khởi động Web Server chế độ phát triển
npm run dev
```
- Truy cập giao diện ứng dụng: [http://localhost:3000](http://localhost:3000)

---

## 4. Tài khoản Demo & Kịch bản Chấm Điểm 5 Phút

### Danh sách tài khoản mẫu (đã được tạo sẵn trong `app/seed_mongo.py`):

| Vai trò | Số điện thoại | Mật khẩu | Quyền hạn chính |
|---|---|---|---|
| **Quản trị viên (Admin)** | `0988000001` | `AdminPass123` | Toàn quyền, báo cáo doanh thu tài chính, xuất CSV, phân quyền, xem Audit Log. |
| **Nhân viên (Staff)** | `0988000002` | `StaffPass123` | Đóng gói hàng, điều chỉnh kho, duyệt đổi trả; **Bị chặn 403 khi xem doanh thu**. |
| **Huấn luyện viên (Coach)** | `0988000003` | `CoachPass123` | Quản lý võ đường Lam Sơn, tạo đợt gom đơn giá sỉ, xuất danh sách size CSV, cọc 50%. |
| **Khách hàng (Customer 1)** | `0988000004` | `UserPass123` | Mua hàng bán lẻ B2C, tư vấn size thông minh, lưu hồ sơ size, đặt thêu tên. |
| **Phụ huynh (Customer 2)** | `0988000005` | `UserPass123` | Mua theo đợt gom đơn của võ đường để hưởng chiết khấu sỉ 20%. |

### Kịch bản Demo 5 bước dành cho Giảng viên:
1. **Mua lẻ B2C & Tư vấn Size thông minh**: Khách hàng chọn võ phục Vovinam -> Nhập chiều cao 163cm, cân nặng 58kg -> Hệ thống gợi ý Size 3 -> Nhập thêu tên riêng: "NGUYEN ANH KHOA" (+30k) -> Hệ thống tự động **chặn COD, ép buộc MoMo**.
2. **Thanh toán MoMo & Trừ kho nguyên tử**: Đặt hàng -> Sinh mã đơn `DT-YYYYMMDD-XXXX` -> Thanh toán MoMo -> Webhook IPN đối soát HMAC-SHA256 -> Tồn kho trừ nguyên tử, chuyển đơn sang `paid`.
3. **Gom đơn B2B Võ đường**: HLV đăng nhập tài khoản `0988000003` -> Mở đợt gom đơn 50 bộ -> Tự động giảm giá sỉ 20% -> Xuất danh sách CSV chuẩn UTF-8 BOM -> Đóng đợt và đặt cọc 50% qua MoMo.
4. **Đổi size & Hoàn kho**: Khách gửi yêu cầu đổi size cho đơn đã giao (trong 7 ngày, chặn hàng thêu) -> Staff duyệt đổi -> Xác nhận nhận hàng trả về kho -> Tồn kho size cũ tự động cộng lại (+1).
5. **Bảo mật RBAC & Báo cáo Doanh thu**: Đăng nhập Staff cố truy cập báo cáo doanh thu -> **Bị chặn 403 Forbidden** -> Đăng nhập Admin xem toàn bộ doanh thu, top sản phẩm và tra cứu nhật ký kiểm toán `activity_logs`.

---

## 5. Danh mục 24 Tính năng Nghiệp vụ Đã Hoàn Thành (F01 – F24)

- **F01**: Đăng ký, đăng nhập JWT, bcrypt, tạm khóa sau 5 lần sai, OTP dev console.
- **F02**: Bộ lọc danh mục 4 môn võ, loại sản phẩm, giá, kích cỡ, màu sắc.
- **F03**: Chi tiết sản phẩm, biến thể tồn kho, tự động kèm bảng size theo môn võ.
- **F04**: Tư vấn chọn size nhân trắc học đối chiếu MongoDB Atlas, chuẩn mốc, nằm giữa 2 size, ngoài bảng size.
- **F05**: Quản lý hồ sơ size võ sinh, tự động cảnh báo nhắc đo lại sau 6 tháng (> 180 ngày).
- **F06**: Giỏ hàng vãng lai `session_id`, giỏ hàng thành viên, gộp giỏ hàng khi đăng nhập (`/cart/merge`).
- **F07**: Dịch vụ thêu tên riêng (tối đa 20 ký tự, 30k/bộ), bắt buộc thanh toán MoMo, chặn COD, không đổi trả.
- **F08**: Đặt hàng COD & MoMo, mã đơn `DT-YYYYMMDD-XXXX`, trừ kho nguyên tử chống oversell, `MockShippingProvider`.
- **F09**: Theo dõi đơn hàng theo mã đơn, bảo vệ chống lỗ hổng IDOR.
- **F10**: Yêu cầu đổi size trong 7 ngày từ lúc `delivered`, chặn đổi trả hàng thêu tên.
- **F11**: Đăng ký CLB/Võ đường, Admin duyệt kích hoạt tài khoản HLV.
- **F12**: Tạo đợt gom đơn (Campaign), chiết khấu sỉ 10–25% theo số lượng, sinh link & QR code chia sẻ.
- **F13**: Phụ huynh đặt hàng theo đợt, áp dụng giá sỉ, tùy chọn MoMo hoặc CLB thanh toán gộp.
- **F14**: HLV theo dõi đợt, xuất CSV UTF-8 BOM (`\ufeff`), đóng đợt và tạo lô giao chung về võ đường.
- **F15**: Võ đường đặt cọc 50% qua MoMo khi đóng đợt; ghi nhận 50% còn lại khi giao hàng.
- **F16**: CRUD sản phẩm & bảng size chuẩn cho Staff/Admin (chặn Customer 403).
- **F17**: Quản lý kho, điều chỉnh tồn kho, log `StockMovement`, cảnh báo tồn kho thấp.
- **F18**: In phiếu đóng gói (Packing Slip), tự động cảnh báo đơn có thêu tên riêng.
- **F19**: Quản lý giao dịch MoMo, đối soát mã đơn/số tiền, đánh dấu hoàn tiền thủ công.
- **F20**: Quản lý đổi trả, phê duyệt, nhận hàng trả về kho tự động hoàn tồn kho (+1).
- **F21** *(hoãn triển khai)*: Đánh giá & nhận xét sản phẩm — module sẽ bổ sung ở phiên bản sau.
- **F22**: Báo cáo doanh thu bán lẻ vs võ đường, theo ngày/tháng, top sản phẩm, xuất CSV UTF-8 BOM (chặn Staff 403).
- **F23**: Phân quyền RBAC (Admin toàn quyền, Staff giới hạn), nhật ký hoạt động bất biến `activity_logs`.
- **F24**: Tối ưu hóa SEO (Dynamic `sitemap.xml`, `robots.txt`, Schema.org JSON-LD `Product`, `BreadcrumbList`, `LocalBusiness`).

---

## 6. Liên kết Tài liệu Thiết kế Hệ thống

Xem chi tiết hồ sơ tài liệu kỹ thuật tại thư mục [`docs/`](./docs/):
- [`docs/01-yeu-cau.md`](./docs/01-yeu-cau.md): Mô tả yêu cầu nghiệp vụ B2C & B2B2C.
- [`docs/02-use-case.md`](./docs/02-use-case.md): Use case tổng quát và 5 ca sử dụng chính.
- [`docs/03-hoat-dong-tuan-tu.md`](./docs/03-hoat-dong-tuan-tu.md): Biểu đồ hoạt động và tuần tự nghiệp vụ.
- [`docs/04-kien-truc.md`](./docs/04-kien-truc.md): Kiến trúc hệ thống tổng thể và phân tầng logic.
- [`docs/05-csdl.md`](./docs/05-csdl.md): Thiết kế cơ sở dữ liệu MongoDB Atlas (Collection, Index, Embed vs Ref).
- [`docs/06-bao-mat.md`](./docs/06-bao-mat.md): Ma trận an ninh bảo mật Mục 9 theo tiêu chuẩn OWASP.
- [`docs/07-tong-ket-do-an.md`](./docs/07-tong-ket-do-an.md): Báo cáo tổng kết đồ án và ma trận kiểm thử 43 ca test.
- [`docs/gia-dinh.md`](./docs/gia-dinh.md): Toàn bộ giả định hệ thống về giá, chiết khấu, vận chuyển, đổi trả, bảo mật.
