# Error Handling — Current Truth

> Cross-cutting concern — áp dụng cho MỌI module, không thuộc riêng module
> nào. Nguyên tắc tổng quát nằm trong `CLAUDE.md` mục 2; file này chứa
> chiến lược chi tiết + catalog error code dùng chung toàn hệ thống.

## 1. Chiến lược xử lý lỗi (EARS notation)

- **[ERR-01]** Khi có exception không được handle, hệ thống shall trả
  HTTP 500 với message chung ("Đã có lỗi xảy ra, vui lòng thử lại"), và
  log full stack trace ở nội bộ — KHÔNG trả stack trace ra client.
- **[ERR-02]** Khi validate input thất bại, hệ thống shall trả HTTP 400
  với error code từ catalog (mục 2) kèm message song ngữ.
- **[ERR-03]** Khi gọi external API (3rd-party) thất bại, hệ thống shall
  retry tối đa 3 lần với exponential backoff trước khi trả lỗi cho client.
- **[ERR-04]** Trong khi có vi phạm business rule đã biết (vd tồn kho
  không đủ), hệ thống shall trả HTTP 409 với error code cụ thể, KHÔNG
  dùng HTTP 500.

## 2. Catalog error code

> Quy ước đặt tên: `<MODULE>_<SỐ THỨ TỰ>`. Khi thêm error code mới, luôn
> đi qua `changes/<ticket-id>/delta-spec.md` như mọi thay đổi khác.

| Code       | HTTP | Message (VI)                  | Message (JP)                              |
|------------|------|--------------------------------|---------------------------------------------|
| AUTH_001   | 401  | Sai tài khoản hoặc mật khẩu    | メールアドレスまたはパスワードが正しくありません |
| AUTH_002   | 423  | Tài khoản đang bị khoá         | アカウントがロックされています               |
| INV_001    | 409  | Số lượng tồn kho không đủ      | 在庫数が不足しています                       |
| SYS_001    | 500  | Đã có lỗi xảy ra, vui lòng thử lại | エラーが発生しました。しばらくしてから再試行してください |

## 3. Trách nhiệm hiển thị lỗi (frontend)

- Lỗi field-level (400 validation) hiển thị ngay dưới field liên quan.
- Lỗi hệ thống (500) hiển thị dạng toast/snackbar, không chặn toàn màn hình.
- Xem thêm quy ước hiển thị theo từng UI cụ thể trong `specs/<module>-ui.md`.

## 4. Lịch sử thay đổi

| Ngày       | Ticket ID | Thay đổi                          |
|------------|-----------|--------------------------------------|
| YYYY-MM-DD | TICKET-1 | Khởi tạo chiến lược + catalog ban đầu |
