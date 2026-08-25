# API Catalog — Current Truth

> File này CHỈ giữ DANH SÁCH endpoint + quy ước chung — đúng nghĩa "index,
> ít đổi". KHÔNG chứa request/response schema chi tiết: phần đó nằm trong
> `specs/<module>.md` của module SỞ HỮU endpoint (xem CLAUDE.md mục 4, quy
> tắc ownership endpoint).
>
> File này chỉ cập nhật khi THÊM/XOÁ endpoint, hoặc đổi method/path —
> KHÔNG cập nhật khi chỉ đổi schema. Cùng cơ chế với `specs/data-model.md`.
>
> Khi thêm/xoá endpoint, vẫn đi qua `changes/<ticket-id>/delta-spec.md` như
> mọi thay đổi khác — không có exception riêng.

## 1. Danh sách endpoint

| Method | Path | Module sở hữu | Auth | Ai gọi | Schema chi tiết |
|---|---|---|---|---|---|
| POST | `/auth/login` | auth | Không | UI (Login) | `specs/example-module-auth.md` |
| POST | `/auth/reset-password` | auth | Không | UI (Reset Password) | `specs/example-module-auth.md` |
| GET | `/auth/me` | auth | Bearer | UI (mọi màn hình sau login) | `specs/example-module-auth.md` |
| POST | `/internal/auth/revoke-sessions` | auth | mTLS nội bộ | **Service khác** | `specs/example-module-auth.md` |
| POST | `/webhooks/idp/user-deleted` | auth | HMAC signature | **Bên thứ ba (IdP)** | `specs/example-module-auth.md` |
| — | (batch) `purge-expired-sessions` | auth | — | **Cron 15 phút/lần** | `specs/example-module-auth.md` |

> Cột **"Ai gọi"** là cột chống bỏ sót quan trọng nhất — 3 dòng cuối bảng
> trên KHÔNG do UI gọi, nên chúng không xuất hiện trong bất kỳ
> `specs/<module>-ui.md` nào. Xem mục 3.

## 2. Quy ước chung (áp dụng mọi endpoint)

- **[API-G01]** Every endpoint path shall dùng kebab-case và danh từ số
  nhiều cho collection (vd `/orders`, KHÔNG `/getOrder`).
- **[API-G02]** Every authenticated endpoint shall nhận token qua header
  `Authorization: Bearer <token>`.
- **[API-G03]** When a request fails, the system shall trả error code theo
  catalog trong `specs/cross-cutting/error-handling.md` — KHÔNG tự định
  nghĩa format lỗi riêng cho từng endpoint.
- **[API-G04]** Every list endpoint shall hỗ trợ phân trang qua
  `?page=&per_page=`, mặc định `per_page=<N>`, tối đa `<M>`.
- **[API-G05]** Breaking change trên endpoint đã release shall được phát
  hành dưới path version mới (`/v2/...`), KHÔNG sửa tại chỗ.

## 3. Checklist chống bỏ sót (BẮT BUỘC khi giao 一覧 cho khách)

> `一覧` nghĩa là *liệt kê đầy đủ*. Một danh sách thiếu âm thầm mà vẫn
> trình khách như là đủ thì tệ hơn không trình. Nguồn bỏ sót phổ biến nhất
> là endpoint KHÔNG do UI gọi.

- [ ] API do UI gọi — trích được từ các dòng EARS `gọi API ...` trong `*-ui.md`
- [ ] API service-to-service / nội bộ
- [ ] Webhook nhận từ bên thứ ba
- [ ] Job batch/cron (không phải HTTP, nhưng khách thường yêu cầu liệt kê)
- [ ] API chỉ dành cho admin/vận hành
- [ ] API còn sống nhưng đã deprecated — phải ghi rõ, không im lặng bỏ

## 4. Lịch sử thay đổi (chỉ log khi THÊM/XOÁ endpoint hoặc đổi method/path)

| Ngày       | Ticket ID | Thay đổi                                       |
|------------|-----------|------------------------------------------------|
| YYYY-MM-DD | TICKET-1  | Khởi tạo: `/auth/login`, `/auth/reset-password` |

<!-- Đổi schema của endpoint đã có KHÔNG log ở đây — xem lịch sử trong
     specs/<module>.md tương ứng. -->
