# Sơ đồ Hoạt động & Sơ đồ Tuần tự Hệ thống (docs/03-hoat-dong-tuan-tu.md)

> **Dự án**: Website Thương mại điện tử Võ phục & Dụng cụ Võ thuật "Đai Trắng"  
> **Cơ sở dữ liệu**: MongoDB Atlas  

---

## 1. Sơ đồ Hoạt động: Đặt hàng & Trừ tồn kho (F06, F07, F08)

```mermaid
flowchart TD
    Start(["Khách hàng kiểm tra Giỏ hàng"]) --> CheckEmbroidery{"Đơn có sản phẩm thêu tên không? (F07)"}
    
    CheckEmbroidery -- Có --> SelectMoMoOnly["Bắt buộc chọn thanh toán trực tuyến (MoMo)\nKhóa phương thức COD"]
    CheckEmbroidery -- Không --> SelectMethod{"Khách hàng chọn phương thức thanh toán?"}
    
    SelectMethod -- COD --> CalcShip["Tính phí giao hàng qua ShippingProvider\n(Nội thành 25k, Tỉnh 40k, Freeship >= 500k)"]
    SelectMoMoOnly --> CalcShip
    SelectMethod -- MoMo --> CalcShip
    
    CalcShip --> CreateOrder["Hệ thống tạo mã đơn DT-YYYYMMDD-XXXX"]
    
    CreateOrder --> CheckPay{"Phương thức thanh toán?"}
    
    CheckPay -- COD --> AtomicDeduct["Trừ tồn kho nguyên tử (Atomic Update) trên MongoDB Atlas\n(products.variants.stock >= quantity)"]
    AtomicDeduct --> CheckStock{"Đủ tồn kho?"}
    CheckStock -- Không --> Rollback["Báo lỗi hết hàng & Rollback giỏ hàng"] --> EndFail(["Dừng quy trình"])
    CheckStock -- Có --> SaveCOD["Trạng thái đơn: confirmed\nGhi nhận StockMovement xuất kho\nSinh mã vận đơn tracking"] --> EndSuccess(["Đặt hàng COD thành công"])
    
    CheckPay -- MoMo --> HoldStock["Khóa giữ chỗ tồn kho 15 phút\nTrạng thái đơn: pending_payment"]
    HoldStock --> RedirectMoMo["Ký số HMAC-SHA256 & Chuyển sang MoMo"]
    RedirectMoMo --> WaitIPN{"Khách thanh toán thành công trong 15p?"}
    WaitIPN -- Có --> RecvIPN["Nhận IPN: Xác minh chữ ký\nChuyển đơn sang paid\nTrừ hẳn tồn kho"] --> EndSuccess
    WaitIPN -- Không / Quá hạn --> ReleaseStock["Hủy phiên thanh toán\nHoàn trả tồn kho giữ chỗ\nTrạng thái đơn: cancelled"] --> EndFail
```

---

## 2. Sơ đồ Tuần tự: Đăng nhập & Cơ chế Chống Dò Mật khẩu (F01, Mục 9)

```mermaid
sequenceDiagram
    autonumber
    actor User as Khách hàng
    participant Client as Frontend (Next.js)
    participant API as FastAPI Backend (/auth/login)
    participant DB as MongoDB Atlas (users collection)

    User->>Client: Nhập Số điện thoại và Mật khẩu
    Client->>API: POST /api/v1/auth/login
    API->>DB: find_one({phone: phone})
    
    alt Không tìm thấy tài khoản
        DB-->>API: null
        API-->>Client: 401 Unauthorized (Số điện thoại hoặc mật khẩu không chính xác)
    else Tìm thấy tài khoản
        DB-->>API: User Document
        alt Tài khoản đang bị khóa tạm thời (locked_until > now)
            API-->>Client: 400 Bad Request (Tài khoản bị tạm khóa 15 phút do nhập sai nhiều lần)
        else Tài khoản hợp lệ
            API->>API: verify_password(plain_pw, hashed_pw)
            alt Sai mật khẩu
                API->>API: attempts = failed_attempts + 1
                alt attempts >= 5
                    API->>DB: update_one({locked_until: now + 15m, failed_attempts: 0})
                    API-->>Client: 400 Bad Request (Nhập sai 5 lần. Tạm khóa 15 phút)
                else attempts < 5
                    API->>DB: update_one({failed_attempts: attempts})
                    API-->>Client: 401 Unauthorized (Sai mật khẩu. Còn N lần thử)
                end
            else Đúng mật khẩu
                API->>DB: update_one({failed_attempts: 0, locked_until: null})
                API->>API: create_access_token(sub=user_id, role=role)
                API-->>Client: 200 OK + JWT Bearer Token
                Client-->>User: Đăng nhập thành công, chuyển hướng trang chủ
            end
        end
    end
```

---

## 3. Sơ đồ Tuần tự: Đặt hàng COD & Trừ Tồn kho Tức thời (F08)

```mermaid
sequenceDiagram
    autonumber
    actor Customer as Khách hàng
    participant API as FastAPI Backend (/orders)
    participant Shipping as MockShippingProvider
    participant DB as MongoDB Atlas (products & orders)

    Customer->>API: POST /api/v1/orders (Địa chỉ, Items, COD)
    API->>API: Kiểm tra ràng buộc thêu tên (F07: Hàng thêu không được chọn COD)
    API->>Shipping: quote(address, weight, subtotal)
    Shipping-->>API: Phí vận chuyển (25k/40k hoặc 0k nếu >= 500k)
    
    API->>API: Sinh mã đơn DT-YYYYMMDD-XXXX
    
    loop Duyệt từng sản phẩm trong đơn
        API->>DB: update_one({id: prod_id, "variants.sku": sku, "variants.stock": {$gte: qty}}, {$inc: {"variants.$.stock": -qty}})
        alt Tồn kho không đủ
            DB-->>API: modified_count == 0
            API->>DB: Rollback hoàn lại tồn kho các item đã trừ
            API-->>Customer: 400 Bad Request (Sản phẩm không đủ tồn kho)
        else Trừ thành công
            DB-->>API: modified_count == 1
            API->>DB: insert_one(stock_movements: {quantity_change: -qty, reason: "Xuất đơn COD"})
        end
    end

    API->>Shipping: create_shipment(order)
    Shipping-->>API: tracking_code (DT-SHIP-XXXXXXXX)
    
    API->>DB: insert_one(orders: {order_code, status: "confirmed", items, tracking_code})
    API-->>Customer: 200 OK (Đặt hàng thành công, trạng thái confirmed)
```
