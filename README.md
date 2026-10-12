# VÕ – Nền tảng thương mại điện tử võ phục và phụ kiện võ thuật

> Đồ án môn Thương mại điện tử – Nhóm 2. Trạng thái: **đang phát triển**. Tài liệu mô tả kế hoạch và phạm vi dự kiến, không khẳng định các tính năng đã hoàn thành.

## 1. Giới thiệu

VÕ là dự án xây dựng hệ thống thương mại điện tử bán võ phục và phụ kiện võ thuật. Giai đoạn đầu tập trung vào **Vovinam, Taekwondo, Karate và võ cổ truyền**, cùng các sản phẩm như đai, đồ bảo hộ và túi đựng. Dự án kết hợp **B2C** (khách mua lẻ) và **B2B** (đơn đồng phục của võ đường/câu lạc bộ).

### Giá trị khác biệt dự kiến
- Danh mục và hướng dẫn chọn võ phục theo môn võ, kích thước và nhu cầu.
- Tư vấn size theo chiều cao/cân nặng; ghi nhận yêu cầu đổi size.
- Hỗ trợ đặt đồng phục số lượng lớn, thêu tên và in logo theo yêu cầu.
- Mô-đun đường dẫn đặt hàng riêng cho võ đường/CLB (giai đoạn mở rộng).

## 2. Phạm vi chức năng

| Phân hệ | Chức năng | Ưu tiên |
|---|---|---|
| Storefront | Trang chủ, danh mục, lọc, tìm kiếm, chi tiết sản phẩm, bảng size | MVP |
| Khách hàng | Đăng ký/đăng nhập, hồ sơ, địa chỉ, giỏ hàng, lịch sử đơn | MVP |
| Checkout | Đặt hàng, COD và chuyển khoản theo luồng xác nhận phù hợp | MVP |
| Đơn hàng | Trạng thái đơn, hủy đơn có điều kiện, ghi nhận đổi/trả | MVP |
| Admin | Quản lý sản phẩm, biến thể size, tồn kho, danh mục, khách hàng, đơn hàng | MVP |
| Vận chuyển | Phí vận chuyển, mã vận đơn, cập nhật và tra cứu trạng thái | MVP |
| Bảo mật | Xác thực, RBAC, kiểm tra dữ liệu đầu vào, bảo vệ dữ liệu cá nhân | MVP |
| B2B | Yêu cầu báo giá, đơn số lượng lớn, cấu hình logo/thêu tên | Giai đoạn 2 |
| Võ đường | Link đặt hàng riêng, tổng hợp đơn và size theo CLB | Giai đoạn 2 |
| Marketing | Mã giảm giá, bài viết SEO, báo cáo KPI | Giai đoạn 2 |

**Lưu ý:** Chuyển khoản và vận chuyển thật chỉ được đánh dấu đã tích hợp khi có kết nối/đối soát xác thực; dữ liệu demo phải ghi rõ là mô phỏng.

## 3. Yêu cầu chất lượng và vận hành

- Tối thiểu có luồng đặt đơn hoàn chỉnh từ giao diện đến cơ sở dữ liệu và giao diện quản trị.
- Quản lý tồn kho theo SKU/size; tránh bán vượt tồn kho khi nhiều người đặt cùng lúc.
- Không để lộ mật khẩu, API key, thông tin thanh toán hay dữ liệu cá nhân trên GitHub.
- Có HTTPS khi triển khai, phân quyền admin/khách hàng, log và xử lý lỗi.
- Có kiểm thử chức năng, API, phân quyền và tải; ghi lại kết quả.
- Có kế hoạch sao lưu, kiểm tra khôi phục, giám sát và đánh giá rủi ro.

## 4. Công nghệ

**Chưa chốt.** Đề xuất tham khảo: React + Vite + TypeScript, Node.js/Express hoặc NestJS, PostgreSQL, Prisma, Docker, GitHub Actions. Việc chọn stack sẽ được ghi trong `docs/architecture.md` sau khi nhóm thống nhất.

## 5. Cấu trúc repository dự kiến

```text
vo-ecommerce/
├── apps/
│   ├── web/                  # Giao diện khách hàng
│   ├── admin/                # Giao diện quản trị (hoặc tích hợp web)
│   └── api/                  # REST API và nghiệp vụ
├── packages/                 # Kiểu dữ liệu và logic dùng chung (nếu cần)
├── database/                 # Schema, migrations, seed dữ liệu mẫu
├── docs/
│   ├── requirements/         # SRS, quy trình nghiệp vụ, use cases
│   ├── diagrams/             # ERD, sequence, activity, architecture
│   ├── api/                  # Đặc tả endpoint
│   ├── testing/              # Test cases, kết quả, bằng chứng
│   ├── operations/           # Deployment, backup, restore, monitoring
│   └── references/           # Nguồn tham khảo và khảo sát
├── .github/workflows/        # CI khi được thiết lập
├── .env.example              # Biến môi trường mẫu, không chứa bí mật
├── .gitignore
├── README.md
└── LICENSE                   # Chỉ thêm nếu nhóm chọn giấy phép
```

## 6. Thiết lập môi trường

Chưa có lệnh chạy chính thức khi nhóm chưa chốt stack và đưa source code vào repository. Sau khi triển khai, cập nhật mục này bằng các lệnh cài đặt, migration, seed và chạy `web`, `admin`, `api`, kèm yêu cầu phiên bản Node.js và cơ sở dữ liệu.

## 7. Quy trình làm việc Git

- `main`: phiên bản ổn định/được duyệt; `develop`: tích hợp.
- Nhánh công việc: `feature/<ten-tinh-nang>`, `fix/<ten-loi>`, `docs/<noi-dung>`.
- Tạo issue có mô tả, người phụ trách và điều kiện nghiệm thu.
- Mở pull request về `develop`, qua review và kiểm thử trước khi merge.
- Không push `.env`, secrets, database thật hoặc dữ liệu khách hàng thật.

Lệnh ví dụ:

```bash
git clone <REPO_URL>
cd <REPO_DIRECTORY>
git checkout -b feature/product-catalog
git add .
git commit -m "feat: add product catalog"
git push -u origin feature/product-catalog
```

## 8. Tài liệu và minh chứng cần có

- Đặc tả yêu cầu, nghiệp vụ B2C/B2B, use case, kiến trúc và ERD.
- Mô tả database, API, giao diện thiết kế và ảnh màn hình.
- Kịch bản kiểm thử, kết quả test, báo cáo lỗi và đánh giá hiệu năng.
- Hướng dẫn deploy, tài khoản demo an toàn, backup/restore và monitoring.
- Báo cáo Word và slide thuyết trình.

## 9. Quy định nộp bài

Theo đề bài: nộp **source code + báo cáo `.docx` + trình chiếu**, đóng gói **một tệp `.zip`** với mẫu tên `TMDT_NHOM_2.zip`. Báo cáo cần có danh sách thành viên và tỷ lệ tham gia.

## 10. Tiến độ

- [ ] Chốt yêu cầu, khảo sát nhu cầu và xác thực nguồn cung
- [ ] Hoàn thành SRS, use case, ERD, Figma và API contract
- [ ] Xây dựng storefront và admin
- [ ] Kết nối database, thanh toán dự kiến và luồng vận chuyển
- [ ] Kiểm thử nghiệp vụ, an toàn, hiệu năng
- [ ] Triển khai online và kiểm tra backup/restore
- [ ] Hoàn thiện báo cáo, slide và gói nộp bài

## 11. Tài liệu cơ sở

- `Nhóm 2 -Elearning.docx` – kế hoạch kinh doanh của nhóm.
- `ĐỀ BÀI TẬP LỚN.pdf` – yêu cầu môn học.

*Tài liệu này phải được cập nhật theo tiến độ phát triển thực tế; không đánh dấu hoàn thành cho những chức năng chưa được kiểm chứng.*
