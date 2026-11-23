# EV Station-based Rental System

Hệ thống thuê xe máy điện theo trạm (station-based). Khách hàng đặt xe online, thanh toán cọc qua PayOS, nhận xe và trả xe tại các trạm. Nhân viên trạm xử lý giao/nhận xe và xác minh giấy tờ. Quản trị viên quản lý toàn bộ hệ thống.

## Công nghệ sử dụng

| Thành phần | Công nghệ |
|---|---|
| Backend | ASP.NET Core Web API (.NET 8), Entity Framework Core, SQL Server |
| Xác thực | JWT Bearer |
| Thanh toán | PayOS (chuyển khoản/QR) và tiền mặt tại trạm |
| Lưu trữ ảnh | Appwrite Storage (ảnh giấy tờ, ảnh xe) |
| Frontend | React 18, Vite, React Router v6, Ant Design, Leaflet / React-Leaflet (bản đồ) |
| Tài liệu API | Swagger (Swashbuckle) |

## Cấu trúc dự án

```
EV_Station_based_Rental_System/
├── BE/EVRentalSystem/
│   ├── API/              # Controllers, Program.cs, cấu hình (appsettings.json)
│   ├── BusinessObject/   # Models, DTOs, Enum, cấu hình JWT/PayOS/Appwrite
│   ├── Repositories/     # Truy cập dữ liệu, DbContext, Migrations
│   └── Services/         # Nghiệp vụ (business logic)
└── FE/FE-EVRental/
    └── src/
        ├── pages/        # user-pages, staff-pages, Admin.jsx, Login, Register
        ├── components/   # NavBar, StationMap, BookingForm, Review*, ...
        ├── contexts/     # AuthContext, CartContext, ReviewContext
        ├── api/ hooks/ services/ utils/
```

## Vai trò người dùng

- **Renter (Khách hàng):** đăng ký, xác minh giấy tờ, tìm xe/trạm, đặt thuê, thanh toán, nhận/trả xe.
- **Station Staff (Nhân viên trạm):** xác minh khách, bàn giao và nhận xe, quản lý thanh toán và xe tại trạm.
- **Admin (Quản trị viên):** quản lý xe, trạm, khách hàng, nhân viên, báo cáo.

## Chức năng chính

### 1. Dành cho khách hàng (Renter)
- Đăng ký, đăng nhập (JWT), xem và cập nhật hồ sơ cá nhân.
- Tải lên và cập nhật giấy tờ tùy thân (CCCD/GPLX) để được xác minh.
- Xem danh sách xe, lọc theo hãng; xem chi tiết xe (`/vehicles`, `/vehicles/:id`).
- Tìm trạm trên bản đồ, xem trạm còn xe sẵn có (`/stations`, `/stations/:id`).
- Đặt thuê xe: chọn xe, trạm, thời gian; có giỏ hàng (`/cart`) và thanh toán (`/checkout`).
- Thanh toán cọc qua PayOS; xử lý các trang kết quả thanh toán thành công/thất bại (`/payment-success`, `/payment-failure`, `/payment-callback`).
- Nhận xe (`/pickup/...`) và trả xe (`/return/...`) tại trạm, kèm màn hình xác nhận.
- Có bộ đếm thời gian phiên thuê (SessionTimer).
- Xem lịch sử thuê xe (`/history`, `/user-history`).
- Đánh giá, nhận xét xe và trạm.
- Nhận thông báo, đánh dấu đã đọc.

### 2. Dành cho nhân viên trạm (`/staff`)
- **Bàn giao xe (Handover):** xem các lượt thuê chờ giao/chờ nhận, xác nhận bắt đầu thuê, hoàn tất trả xe, ghi nhận tình trạng xe.
- **Xác minh khách hàng:** duyệt hoặc từ chối giấy tờ đang chờ xử lý.
- **Quản lý thanh toán:** xem thanh toán PayOS và tiền mặt tại trạm, tạo phí phát sinh (phí trả xe, hư hỏng...).
- **Quản lý xe tại trạm:** xem và cập nhật tình trạng xe, báo cáo sự cố.
- Thống kê số lượt giao/nhận của từng nhân viên.
- Danh sách có phân trang.

### 3. Dành cho quản trị viên (`/admin`)
- **Dashboard:** thống kê tổng quan.
- **Quản lý trạm thuê xe:** thêm, sửa, xóa trạm; xem trạm hoạt động/ngưng hoạt động; xem tồn kho xe của trạm.
- **Lịch sử giao/nhận xe:** theo dõi các lượt thuê đang diễn ra và đã hoàn thành.
- **Quản lý khách hàng:** xem danh sách, khóa (suspend) hoặc kích hoạt tài khoản.
- **Quản lý nhân viên:** tạo tài khoản nhân viên, phân công hoặc gỡ khỏi trạm.
- **Quản lý loại xe và hãng xe:** thêm, sửa, xóa hãng; quản lý mẫu xe, kích hoạt/ngưng xe, thêm xe vào trạm, quản lý tồn kho theo biển số.
- **Báo cáo:** thống kê doanh thu và hoạt động.

### 4. Tổng hợp API (Backend)

Tất cả endpoint có tiền tố `/api/{Controller}`; hầu hết yêu cầu JWT.

| Controller | Chức năng chính |
|---|---|
| `Account` | Login, Register, danh sách/chi tiết tài khoản, cập nhật, tạo nhân viên trạm, khóa/kích hoạt tài khoản |
| `Renter` | CRUD thông tin khách thuê, tìm theo tài khoản/giấy tờ |
| `IDDocument` | Danh sách giấy tờ, danh sách chờ duyệt, khách cập nhật giấy tờ, xác minh (duyệt/từ chối) |
| `Brand` | CRUD hãng xe, tìm kiếm, đếm số xe theo hãng |
| `EVBike` | Danh sách xe, xe còn sẵn, xe theo trạm/hãng, thêm/sửa xe, kích hoạt/ngưng, thêm xe vào trạm |
| `EVBike_Stocks` | Quản lý từng xe vật lý (biển số): CRUD, tra theo biển số, tồn kho tại trạm của nhân viên |
| `Station` | CRUD trạm, trạm hoạt động/ngưng, trạm còn xe, tồn kho theo mẫu xe, ảnh trạm |
| `StationStaff` | CRUD nhân viên, phân công/gỡ khỏi trạm, lượt thuê được giao, bàn giao/nhận xe, thống kê |
| `Rental` | Tạo, cập nhật, xóa lượt thuê; tra theo trạm/tài khoản; bàn giao (`HandOverBike`) và trả xe (`ReturnBike`) |
| `Payment` | Tạo thanh toán cọc/phí, cập nhật trạng thái thành công/thất bại, danh sách PayOS/tiền mặt theo trạm |
| `Notification` | Tạo, xem, đếm chưa đọc, đánh dấu đã đọc |

### 5. Quy trình thuê xe

1. Khách đăng ký và tải giấy tờ, nhân viên xác minh (`Pending` → `Approved`/`Rejected`).
2. Khách chọn xe và trạm, tạo lượt thuê (`Pending`).
3. Thanh toán cọc qua PayOS → lượt thuê chuyển sang `Reserved`.
4. Nhân viên bàn giao xe tại trạm → `OnGoing`.
5. Khách trả xe, nhân viên nhận xe, tính phí phát sinh nếu có → `Completed` (hoặc `Cancelled`).

**Các trạng thái chính:**
- Tài khoản: `Pending`, `Active`, `Suspended`, `Deleted`.
- Lượt thuê: `Pending`, `Reserved`, `OnGoing`, `Cancelled`, `Completed`.
- Thanh toán: `Pending`, `Completed`, `Failed`. Phương thức: `PayOS`, `Cash`. Loại: `Deposit`, `Fee`, `Refund`.
- Xe: `Unavailable`, `Available`, `InMaintenance`.

## Hướng dẫn chạy dự án

### Yêu cầu
- .NET 8 SDK
- SQL Server
- Node.js 18 trở lên

### Backend
```bash
cd BE/EVRentalSystem
# Cập nhật ConnectionStrings:DefaultConnection trong API/appsettings.json
dotnet ef database update --project Repositories --startup-project API
dotnet run --project API
```
API chạy tại `http://localhost:5168`, Swagger có sẵn ở môi trường Development.

### Frontend
```bash
cd FE/FE-EVRental
npm install
npm run dev
```
Ứng dụng chạy tại `http://localhost:5173`. Có thể đổi địa chỉ API bằng biến môi trường `VITE_API_BASE_URL` (mặc định `http://localhost:5168`).

## Cấu hình

Các mục cần thiết lập trong `BE/EVRentalSystem/API/appsettings.json`:

- `ConnectionStrings:DefaultConnection`: chuỗi kết nối SQL Server.
- `JwtConfig`: khóa ký JWT, Issuer, Audience.
- `Appwrite`: `Endpoint`, `ProjectId`, `BucketId`.
- `PayOS`: `PAYOS_CLIENT_ID`, `PAYOS_API_KEY`, `PAYOS_CHECKSUM_KEY`.
- `Server` và `Client`: URL backend và frontend (dùng cho CORS và redirect thanh toán).

> **Cảnh báo bảo mật:** không commit khóa bí mật thật (PayOS, JWT, mật khẩu DB) lên GitHub. Hãy dùng `appsettings.Development.json` đã được gitignore, User Secrets hoặc biến môi trường.

## Tài liệu bổ sung

Thư mục `BE/EVRentalSystem` và thư mục gốc có các tài liệu chi tiết, ví dụ `Rental_API_Documentation.md`, `Payment_API_Documentation.md`, `Station_API_Documentation.md`, `NOTIFICATION_CRUD_DOCUMENTATION.md`, và các file `PAYOS_*.md` về tích hợp thanh toán PayOS.
