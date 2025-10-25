# 🛍️ EProject Phase 1 – Hệ thống thương mại điện tử (Microservices)

## 1. 🎯 Hệ thống giải quyết vấn đề gì

Hệ thống được xây dựng nhằm **mô phỏng quy trình hoạt động của một nền tảng thương mại điện tử** (E-Commerce System) theo **kiến trúc microservices**.

Mỗi chức năng chính (người dùng, sản phẩm, đơn hàng, xác thực, cổng giao tiếp) được triển khai dưới dạng **một dịch vụ độc lập**, giúp hệ thống:
- Dễ dàng mở rộng, bảo trì và triển khai độc lập từng module.
- Giảm thiểu xung đột khi phát triển đồng thời.
- Ứng dụng các công nghệ phổ biến trong thực tế như **Docker**, **MongoDB**, **RabbitMQ**, **JWT**.

Người dùng có thể:
- Đăng ký / đăng nhập (qua dịch vụ Auth)
- Xem danh sách sản phẩm (qua Product)
- Tạo đơn hàng (qua Order)
- Giao tiếp giữa các dịch vụ thông qua **API Gateway** hoặc **RabbitMQ**.

---

## 2. 🧩 Hệ thống có bao nhiêu dịch vụ

Hệ thống gồm **6 dịch vụ chính**, được định nghĩa trong file `docker-compose.yml`:

| STT | Tên Service | Vai trò chính |
|-----|--------------|----------------|
| 1️⃣ | **mongo** | Cơ sở dữ liệu NoSQL – lưu dữ liệu người dùng, sản phẩm, đơn hàng |
| 2️⃣ | **rabbitmq** | Hàng đợi thông điệp – hỗ trợ giao tiếp bất đồng bộ giữa các service |
| 3️⃣ | **auth** | Xác thực người dùng, tạo JWT, quản lý tài khoản |
| 4️⃣ | **product** | Quản lý sản phẩm (thêm, sửa, xóa, xem) |
| 5️⃣ | **order** | Tạo và quản lý đơn hàng, nhận dữ liệu sản phẩm qua RabbitMQ |
| 6️⃣ | **gateway** | API Gateway – điểm vào duy nhất của toàn hệ thống |

---

## 3. ⚙️ Ý nghĩa chi tiết từng dịch vụ

### 🧱 3.1. Auth Service
- Chức năng: Quản lý tài khoản người dùng (đăng ký, đăng nhập).  
- Sinh token JWT để xác thực cho các yêu cầu từ người dùng.  
- Kết nối với MongoDB thông qua biến môi trường `MONGODB_AUTH_URI`.

### 📦 3.2. Product Service
- Chức năng: Cung cấp dữ liệu sản phẩm.  
- Có endpoint `/products` và `/products/:id` để hiển thị thông tin.  
- Sử dụng JWT để xác thực và RabbitMQ để gửi thông tin đến các service khác.  
- Database: `product` trong MongoDB.

### 🛒 3.3. Order Service
- Chức năng: Tạo đơn hàng mới, xem danh sách đơn hàng.  
- Khi người dùng đặt hàng, service này sẽ **gửi yêu cầu đến Product service** để kiểm tra thông tin sản phẩm.  
- Dùng RabbitMQ để nhận phản hồi bất đồng bộ.  
- Database: `order` trong MongoDB.

### 🌐 3.4. API Gateway
- Là điểm vào duy nhất của người dùng.  
- Tất cả request bên ngoài đều đi qua Gateway và được **chuyển tiếp (proxy)** đến các service phù hợp.  
- Có thể xử lý xác thực chung, logging, hoặc rate limit.

### 🐇 3.5. RabbitMQ
- Giúp các dịch vụ giao tiếp **bất đồng bộ** thông qua message queue.  
- Ví dụ: Khi Order service cần thông tin sản phẩm, thay vì gọi trực tiếp Product API, nó gửi message đến hàng đợi RabbitMQ → Product service nhận, xử lý và trả lại kết quả.

### 🗄️ 3.6. MongoDB
- Lưu trữ toàn bộ dữ liệu của hệ thống (Auth, Product, Order).  
- Mỗi service có **database riêng biệt**, tránh xung đột dữ liệu.

---

## 4. 🧠 Các mẫu thiết kế (Design Patterns) được sử dụng

| Mẫu thiết kế | Ý nghĩa | Ứng dụng trong dự án |
|---------------|----------|----------------------|
| **Microservices Architecture** | Tách hệ thống thành nhiều dịch vụ nhỏ, độc lập | Toàn bộ hệ thống gồm các service riêng: Auth, Product, Order, Gateway |
| **Repository Pattern** | Quản lý thao tác với cơ sở dữ liệu tách biệt khỏi logic nghiệp vụ | Trong các file `controllers` và `models` |
| **Observer / Publish-Subscribe** | Giao tiếp giữa các dịch vụ thông qua RabbitMQ | Order gửi thông báo → Product nhận và phản hồi |
| **API Gateway Pattern** | Cung cấp điểm truy cập duy nhất vào hệ thống | Service `gateway` đóng vai trò trung gian điều hướng request |
| **JWT Authentication Pattern** | Bảo mật và xác thực người dùng qua token | Dùng trong Auth và xác minh ở các service khác |

---

## 5. 🔗 Các dịch vụ giao tiếp như thế nào

Hệ thống sử dụng **hai cơ chế giao tiếp chính**:

### 🧩 5.1. Giao tiếp đồng bộ (Synchronous)
- Qua **HTTP REST API**.
- Ví dụ: Gateway → Auth, Gateway → Product.
- Các endpoint sử dụng `axios` hoặc `fetch` để gọi giữa các service.

### 🕊️ 5.2. Giao tiếp bất đồng bộ (Asynchronous)
- Qua **RabbitMQ (Message Queue)**.
- Ví dụ: Order gửi yêu cầu kiểm tra sản phẩm đến hàng đợi RabbitMQ → Product nhận message, xử lý → phản hồi kết quả qua queue khác.

---

## 🚀 Cách chạy dự án

```bash
# 1. Clone repository
git clone https://github.com/yourusername/EProject-Phase-1.git
cd EProject-Phase-1

# 2. Chạy docker compose
docker-compose up --build

# 3. Kiểm tra các service
# - Auth Service: http://localhost:3000
# - Product Service: http://localhost:3001
# - Order Service: http://localhost:3002
# - API Gateway: http://localhost:3003
# - RabbitMQ Management UI: http://localhost:15672

---

## 🏗️ Kiến trúc hệ thống (System Architecture)

Sơ đồ dưới đây mô tả cách các dịch vụ trong hệ thống giao tiếp với nhau.

```mermaid
flowchart LR
    subgraph Client
        U[Người dùng / Postman]
    end

    subgraph Gateway[API Gateway]
    end

    subgraph Auth[Auth Service]
    end

    subgraph Product[Product Service]
    end

    subgraph Order[Order Service]
    end

    subgraph MQ[RabbitMQ]
    end

    subgraph DB[MongoDB]
        ADB[(auth DB)]
        PDB[(product DB)]
        ODB[(order DB)]
    end

    U -->|HTTP Request| Gateway
    Gateway -->|API call /auth| Auth
    Gateway -->|API call /products| Product
    Gateway -->|API call /orders| Order

    Auth --> ADB
    Product --> PDB
    Order --> ODB

    Order <-->|message queue| MQ
    Product <-->|message queue| MQ

    MQ -.-> Order
    MQ -.-> Product


Docker compose up --bulid
<img width="1303" height="456" alt="image" src="https://github.com/user-attachments/assets/3a3d4d18-d2fa-4682-a0b3-f6dd7574461b" />

Chạy docker: docker-compose up -d
<img width="946" height="206" alt="image" src="https://github.com/user-attachments/assets/dd4d33fd-1fce-4a8c-bbe4-017d191e8999" />

Test các chức năng trên POSTMAN
1/ Register(POST)

<img width="1411" height="724" alt="image" src="https://github.com/user-attachments/assets/f207e504-0850-48a0-9e4f-ae98794ee2c3" />

2/ Login (POST)

<img width="1392" height="676" alt="image" src="https://github.com/user-attachments/assets/ff01ba42-7809-4590-8428-aecdda62ccd5" />

token để xem/tạo/đặt hàng sản phẩm
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjY4ZjM4ODI1ZjE1NTNmNDk0ZWYyYTdlNiIsImlhdCI6MTc2MDc5MDYwMX0.cGCyTJqb9HGWThxlfEJKdoBjsnXR2Xrimz6Kye6D25M

3/ Create products(POST)
Nhập token tab Authorization
<img width="1403" height="447" alt="image" src="https://github.com/user-attachments/assets/30d0207d-d910-435d-b15d-65571421e29f" />

Nhập ở tab Body
<img width="1406" height="735" alt="image" src="https://github.com/user-attachments/assets/5111627c-a452-4b0e-a86b-0105e5748b75" />

4/Read all products(GET)
Nhập token tab Authorization
<img width="1409" height="764" alt="image" src="https://github.com/user-attachments/assets/fd2b247e-e828-4a3b-8630-60e297ab11c7" />


5/Order products(POST)
Nhập token tab Authorization
<img width="1384" height="414" alt="image" src="https://github.com/user-attachments/assets/42ba69d4-fb4a-4c7e-b1ec-09ece5c2e3ce" />

Tab Body
<img width="1400" height="745" alt="image" src="https://github.com/user-attachments/assets/30a3d286-466e-4b1c-ad25-9215252ac159" />

