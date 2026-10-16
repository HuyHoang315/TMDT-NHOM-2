# Thiết kế Cơ sở dữ liệu MongoDB Atlas (docs/05-csdl.md)

> **Dự án**: Website Thương mại điện tử Võ phục & Dụng cụ Võ thuật "Đai Trắng"  
> **Cơ sở dữ liệu**: MongoDB Atlas (NoSQL Document Store)  

---

## 1. Sơ đồ Quan hệ Thực thể Logic (ERD)

```mermaid
erDiagram
    USERS ||--o{ STUDENT_PROFILES : "quản lý"
    USERS ||--o{ ORDERS : "đặt"
    USERS ||--o{ ACTIVITY_LOGS : "thao tác"
    CLUBS ||--o{ CLUB_CAMPAIGNS : "tổ chức"
    CLUB_CAMPAIGNS ||--o{ ORDERS : "gom đơn"
    MARTIAL_ARTS ||--o{ PRODUCTS : "phân loại"
    CATEGORIES ||--o{ PRODUCTS : "phân loại"
    PRODUCTS ||--o{ PRODUCT_VARIANTS : "chứa"
    ORDERS ||--o{ ORDER_ITEMS : "bao gồm"
    ORDERS ||--o{ ORDER_STATUS_HISTORY : "ghi nhận"
    ORDERS ||--o| RETURN_REQUESTS : "khiếu nại đổi"

    USERS {
        string id PK
        string phone UK "Chỉ mục duy nhất"
        string full_name
        string hashed_password
        string role "customer, coach, staff, admin"
        string email UK "Sparse unique"
        int failed_login_attempts
        datetime locked_until
        datetime created_at
    }

    STUDENT_PROFILES {
        string id PK
        string user_id FK
        string student_name
        string martial_art
        float height_cm
        float weight_kg
        string recommended_size
        datetime measured_date
    }

    PRODUCTS {
        string id PK
        string name
        string slug UK "Chỉ mục duy nhất"
        string martial_art_id FK
        string category_id FK
        int price "Số nguyên VND"
        int discount_percent
        string thumbnail
        string material
        boolean is_customizable "Cho phép thêu tên"
    }

    PRODUCT_VARIANTS {
        string sku PK
        string product_id FK
        string size
        string color
        int price "Số nguyên VND"
        int stock ">= 0 (Ràng buộc kho)"
        int low_stock_threshold
    }

    ORDERS {
        string id PK
        string order_code UK "Định dạng DT-YYYYMMDD-XXXX"
        string user_id FK
        string campaign_id FK "Nếu là đơn đợt võ đường"
        string channel "retail hoặc club"
        int subtotal "Số nguyên VND"
        int shipping_fee
        int discount_amount
        int total_amount "Số nguyên VND"
        string payment_method "COD hoặc MoMo"
        string status "pending_payment, confirmed, processing, shipping, delivered, completed, cancelled"
        datetime created_at
    }

    RETURN_REQUESTS {
        string id PK
        string order_code FK
        string old_size
        string new_size
        string reason
        string status "pending, approved, rejected, completed"
        datetime requested_at
    }
```

---

## 2. Từ điển Dữ liệu các Collections trên MongoDB Atlas

### 2.1. Collection `users`
Lưu trữ thông tin tài khoản khách hàng, HLV võ đường, nhân viên vận hành và quản trị viên.

| Trường | Kiểu dữ liệu | Ràng buộc | Ý nghĩa / Ghi chú |
|---|---|---|---|
| `_id` | ObjectId | Khóa chính MongoDB | Định danh bản ghi nội bộ |
| `id` | String | Unique Index | Mã định danh dạng UUID chuẩn |
| `phone` | String | Unique Index | Số điện thoại đăng nhập chính thức |
| `full_name` | String | Not null | Họ và tên người dùng |
| `hashed_password` | String | Not null | Mật khẩu băm an toàn bằng Bcrypt (Mục 9) |
| `role` | String | Index | `customer`, `coach`, `staff`, `admin` |
| `email` | String | Sparse Unique | Email cá nhân (tùy chọn) |
| `address` | String | Nullable | Địa chỉ giao hàng mặc định |
| `failed_login_attempts` | Integer | Default 0 | Số lần nhập sai mật khẩu liên tiếp |
| `locked_until` | DateTime | Nullable | Thời điểm mở khóa tài khoản (khóa 15 phút) |
| `reset_token` | String | Nullable | Mã OTP / Token khôi phục mật khẩu |
| `reset_token_expires_at`| DateTime | Nullable | Thời hạn của token quên mật khẩu |
| `is_active` | Boolean | Default True | Trạng thái kích hoạt tài khoản |
| `created_at` | DateTime | Default Now | Thời điểm tạo tài khoản |

### 2.2. Collection `products`
Lưu trữ danh mục sản phẩm võ phục, đai, dụng cụ bảo hộ và các biến thể kích thước / màu sắc.

| Trường | Kiểu dữ liệu | Ràng buộc | Ý nghĩa / Ghi chú |
|---|---|---|---|
| `id` | String | Unique Index | Mã định danh sản phẩm |
| `name` | String | Not null | Tên sản phẩm |
| `slug` | String | Unique Index | Đường dẫn URL thân thiện chuẩn SEO |
| `martial_art_id` | String | Index | Môn võ: `vovinam`, `taekwondo`, `karate`, `vo_co_truyen` |
| `category_id` | String | Index | Loại: `vo_phuc`, `dai`, `bao_ho`, `tui_dung`, `phu_kien` |
| `price` | Integer | >= 0 | Giá niêm yết (VND, số nguyên) |
| `original_price` | Integer | Nullable | Giá gốc trước giảm (VND) |
| `thumbnail` | String | URL | Ảnh đại diện sản phẩm |
| `material` | String | Not null | Chất liệu vải (ví dụ: Kaki Cotton 100%) |
| `is_customizable` | Boolean | Default False| Hỗ trợ dịch vụ thêu tên riêng |
| `variants` | Array[Object] | Biến thể | Danh sách các biến thể SKU, size, màu sắc, tồn kho |

---

## 3. Các Ràng buộc Toàn vẹn Dữ liệu trên MongoDB Atlas

1. **Ràng buộc Tồn kho không âm**:
   - Mọi thao tác trừ tồn kho khi tạo đơn đặt hàng sử dụng toán tử nguyên tử (Atomic Update) của MongoDB:
     ```python
     result = await db["products"].update_one(
         {"id": product_id, "variants.sku": sku, "variants.stock": {"$gte": quantity}},
         {"$inc": {"variants.$.stock": -quantity}}
     )
     ```
   - Nếu `result.modified_count == 0`, giao dịch bị từ chối do không đủ tồn kho, triệt tiêu 100% nguy cơ bán vượt tồn kho (Race condition).
2. **Ràng buộc Duy nhất (Unique Indexes)**:
   - `users.phone`: Không cho phép trùng số điện thoại.
   - `products.slug`: Mỗi sản phẩm có slug SEO duy nhất.
   - `orders.order_code`: Mã đơn dạng `DT-YYYYMMDD-XXXX` là duy nhất toàn hệ thống.
