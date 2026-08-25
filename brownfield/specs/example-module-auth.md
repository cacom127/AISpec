# Module: Auth — Current Truth

> Ví dụ mẫu cho 1 file spec trong `specs/`. Copy file này và đổi tên khi
> tạo spec cho module khác (vd `specs/inventory.md`, `specs/billing.md`).
>
> **BẢN BROWNFIELD** — ô "Độ phủ" ngay dưới đây là BẮT BUỘC (CLAUDE.md mục 4).

- **Độ phủ**: MỘT PHẦN — chỉ gồm requirement đã đi qua `changes/`.
  **Hành vi không xuất hiện trong file này KHÔNG có nghĩa là không tồn
  tại** — phải đối chiếu code. Tổng quan: `specs/_coverage.md`.
- **Đã spec**: login, lock account, RBAC (`AUTH-01..04`)
- **Chưa spec**: SSO, đổi mật khẩu, quên mật khẩu

## 1. Mục đích module

Quản lý xác thực người dùng và phân quyền truy cập cho toàn hệ thống.

## 2. Yêu cầu hiện tại (Requirements — EARS notation)

- **[AUTH-01]** Khi user submit credentials hợp lệ, hệ thống shall phát
  session token có hạn 8 tiếng.
- **[AUTH-02]** Khi user đăng nhập sai 5 lần trong 1 phút, hệ thống shall
  khoá tài khoản trong 15 phút.
- **[AUTH-03]** Trong khi session token đã hết hạn, hệ thống shall từ chối
  mọi API request đã xác thực với HTTP 401.
- **[AUTH-04]** Hệ thống shall hỗ trợ phân quyền theo role, gồm `admin`,
  `store_staff`, `viewer`.

> Mỗi ID (AUTH-01, AUTH-02...) dùng để trace từ test case ngược lại yêu cầu,
> và để `delta-spec.md` tham chiếu khi có thay đổi (vd "Sửa AUTH-02: giảm
> xuống 3 lần thử").
>
> **Brownfield**: requirement được ghi nhận từ code (không do ai thiết kế)
> thì nên đánh dấu nguồn — vd `[AUTH-03] (ghi nhận từ code, TICKET-12)`.
> Lý do: nó phản ánh "code đang làm gì", CHƯA được khách xác nhận là "đúng
> ý muốn". Khi nào confirm được với khách thì bỏ dấu này đi.

## 3. Ràng buộc kỹ thuật đã chốt

- Token: JWT, ký bằng <thuật toán>, secret rotate mỗi <X> ngày.
- Không lưu password dạng plaintext hoặc hash yếu (MD5/SHA1).

## 4. Data Model (field-level — module này SỞ HỮU entity User, Session)

> Xem `specs/data-model.md` mục 2 để biết entity nào thuộc module nào.
> Đây là nơi DUY NHẤT định nghĩa field chi tiết của User/Session — không
> lặp lại ở `specs/data-model.md`.

### User
| Field    | Type   | Constraint                     |
|----------|--------|----------------------------------|
| id       | uuid   | PK                                |
| email    | string | unique, not null                 |
| password | string | hashed (bcrypt), not null         |
| role     | enum   | admin / store_staff / viewer      |

### Session
| Field       | Type     | Constraint                        |
|-------------|----------|--------------------------------------|
| id          | uuid     | PK                                    |
| user_id     | uuid     | FK → User.id, cascade delete          |
| expires_at  | datetime | not null                              |

### Ràng buộc dữ liệu (EARS notation)

- **[DM-AUTH-01]** Field `User.email` shall là duy nhất trên toàn hệ thống.
- **[DM-AUTH-02]** Khi một `User` bị xoá, hệ thống shall cascade-delete
  mọi bản ghi `Session` liên quan.
- **[DM-AUTH-03]** Hệ thống shall NOT cho phép `Session.expires_at` được
  set vào thời điểm trong quá khứ lúc tạo bản ghi.

## 5. UI (tuỳ chọn — nếu module đơn giản, không cần tách file *-ui.md riêng)

- Xem `DESIGN.md` cho token màu/font/component dùng chung.
- Layout, state, hành vi tương tác chi tiết của từng màn hình auth: xem
  file riêng `specs/auth-ui.md` nếu module có nhiều màn hình.

## 6. Lịch sử thay đổi module này

| Ngày       | Ticket ID    | Thay đổi                                    |
|------------|--------------|-----------------------------------------------|
| YYYY-MM-DD | TICKET-1    | Khởi tạo: entity User/Session, AUTH-01..03    |
| YYYY-MM-DD | TICKET-XXX  | Thêm AUTH-04 (role-based access control)      |

<!-- Trỏ về changes/_archive/TICKET-XXX/ để xem đầy đủ proposal/plan gốc -->
