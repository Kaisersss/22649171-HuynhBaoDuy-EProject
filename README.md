# EProject Phase 1 – Hệ thống thương mại điện tử (Microservices)

## 1. Hệ thống giải quyết vấn đề gì

Hệ thống được xây dựng nhằm **mô phỏng quy trình hoạt động của một nền tảng thương mại điện tử** (E-Commerce System) theo **kiến trúc microservices**.

Người dùng có thể:

- Đăng ký / đăng nhập (qua dịch vụ Auth)
- Xem danh sách sản phẩm (qua Product)
- Tạo đơn hàng (qua Order)
- Giao tiếp giữa các dịch vụ thông qua **API Gateway** hoặc **RabbitMQ**.

---

## 2. Hệ thống có bao nhiêu dịch vụ

Hệ thống gồm **6 dịch vụ chính**, được định nghĩa trong file `docker-compose.yml`:

| STT | Tên Service  | Vai trò chính                                                       |
| --- | ------------ | ------------------------------------------------------------------- |
| 1️⃣  | **mongo**    | Cơ sở dữ liệu NoSQL – lưu dữ liệu người dùng, sản phẩm, đơn hàng    |
| 2️⃣  | **rabbitmq** | Hàng đợi thông điệp – hỗ trợ giao tiếp bất đồng bộ giữa các service |
| 3️⃣  | **auth**     | Xác thực người dùng, tạo JWT, quản lý tài khoản                     |
| 4️⃣  | **product**  | Quản lý sản phẩm (thêm, sửa, xóa, xem)                              |
| 5️⃣  | **order**    | Tạo và quản lý đơn hàng, nhận dữ liệu sản phẩm qua RabbitMQ         |
| 6️⃣  | **gateway**  | API Gateway – điểm vào duy nhất của toàn hệ thống                   |

---

## 3. Ý nghĩa chi tiết từng dịch vụ

### 3.1. Auth Service

- Chức năng: Quản lý tài khoản người dùng (đăng ký, đăng nhập).
- Sinh token JWT để xác thực cho các yêu cầu từ người dùng.
- Kết nối với MongoDB thông qua biến môi trường `MONGODB_AUTH_URI`.

### 3.2. Product Service

- Chức năng: Cung cấp dữ liệu sản phẩm.
- Có endpoint `/products` và `/products/:id` để hiển thị thông tin.
- Sử dụng JWT để xác thực và RabbitMQ để gửi thông tin đến các service khác.
- Database: `product` trong MongoDB.

### 3.3. Order Service

- Chức năng: Tạo đơn hàng mới, xem danh sách đơn hàng.
- Khi người dùng đặt hàng, service này sẽ **gửi yêu cầu đến Product service** để kiểm tra thông tin sản phẩm.
- Dùng RabbitMQ để nhận phản hồi bất đồng bộ.
- Database: `order` trong MongoDB.

### 3.4. API Gateway

- Là điểm vào duy nhất của người dùng.
- Tất cả request bên ngoài đều đi qua Gateway và được **chuyển tiếp (proxy)** đến các service phù hợp.
- Có thể xử lý xác thực chung, logging, hoặc rate limit.

### 3.5. RabbitMQ

- Giúp các dịch vụ giao tiếp **bất đồng bộ** thông qua message queue.
- Ví dụ: Khi Order service cần thông tin sản phẩm, thay vì gọi trực tiếp Product API, nó gửi message đến hàng đợi RabbitMQ → Product service nhận, xử lý và trả lại kết quả.

### 3.6. MongoDB

- Lưu trữ toàn bộ dữ liệu của hệ thống (Auth, Product, Order).
- Mỗi service có **database riêng biệt**, tránh xung đột dữ liệu.

---

## 4. Các mẫu thiết kế (Design Patterns) được sử dụng

| Mẫu thiết kế                     | Ý nghĩa                                                           | Ứng dụng trong dự án                                                  |
| -------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------- |
| **Microservices Architecture**   | Tách hệ thống thành nhiều dịch vụ nhỏ, độc lập                    | Toàn bộ hệ thống gồm các service riêng: Auth, Product, Order, Gateway |
| **Repository Pattern**           | Quản lý thao tác với cơ sở dữ liệu tách biệt khỏi logic nghiệp vụ | Trong các file `controllers` và `models`                              |
| **Observer / Publish-Subscribe** | Giao tiếp giữa các dịch vụ thông qua RabbitMQ                     | Order gửi thông báo → Product nhận và phản hồi                        |
| **API Gateway Pattern**          | Cung cấp điểm truy cập duy nhất vào hệ thống                      | Service `gateway` đóng vai trò trung gian điều hướng request          |
| **JWT Authentication Pattern**   | Bảo mật và xác thực người dùng qua token                          | Dùng trong Auth và xác minh ở các service khác                        |

---

## 5. Các dịch vụ giao tiếp như thế nào

Hệ thống sử dụng **hai cơ chế giao tiếp chính**:

### 5.1. Giao tiếp đồng bộ (Synchronous)

- Qua **HTTP REST API**.
- Ví dụ: Gateway → Auth, Gateway → Product.
- Các endpoint sử dụng `axios` hoặc `fetch` để gọi giữa các service.

### 5.2. Giao tiếp bất đồng bộ (Asynchronous)

- Qua **RabbitMQ (Message Queue)**.
- Ví dụ: Order gửi yêu cầu kiểm tra sản phẩm đến hàng đợi RabbitMQ → Product nhận message, xử lý → phản hồi kết quả qua queue khác.

---

Download source code Run npm install Setup all microservices Test all business logic with POSTMAN

Docker compose up --bulid
`<img width="1303" height="456" alt="image" src="./public/img/1.png" />`

Chạy docker: docker-compose up -d

`<img width="1392" height="676" alt="image" src="public/img/2.png" />`

Test các chức năng trên POSTMAN
1/ Register(POST)

<img width="1411" height="724" alt="image" src="public/img/3.png" />

2/ Login (POST)

<img width="1392" height="676" alt="image" src="public/img/4.png" />

token để xem/tạo/đặt hàng sản phẩm
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjY4ZjM4ODI1ZjE1NTNmNDk0ZWYyYTdlNiIsImlhdCI6MTc2MDc5MDYwMX0.cGCyTJqb9HGWThxlfEJKdoBjsnXR2Xrimz6Kye6D25M

3/ Create products(POST)
Nhập token tab Authorization
`<img width="1403" height="447" alt="image" src="public/img/5.png" />`

Nhập ở tab Body
`<img width="1406" height="735" alt="image" src="public/img/6.png" />`

4/Read all products(GET)
Nhập token tab Authorization
`<img width="1409" height="764" alt="image" src="public/img/7.png" />`

5/Order products(POST)
Nhập token tab Authorization
`<img width="1384" height="414" alt="image" src="public/img/8.png" />`

Tab Body
`<img width="1400" height="745" alt="image" src="public/img/9.png" />`
