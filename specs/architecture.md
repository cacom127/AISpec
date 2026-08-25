# Architecture — Current Truth

> File này mô tả TRẠNG THÁI HIỆN TẠI đã chốt của kiến trúc hệ thống.
> Không ghi ở đây các đề xuất đang bàn — những cái đó thuộc về `changes/`.
> File này chỉ được cập nhật KHI một `changes/<ticket-id>/delta-spec.md`
> liên quan đến kiến trúc được merge.

## 1. Tổng quan hệ thống

<Sơ đồ / mô tả ngắn hệ thống đang có — tên các service chính, vai trò của
từng service>

```
[Mobile App (Flutter)] --> [API Gateway] --> [Backend Service] --> [DB]
                                       └----> [3rd-party integration]
```

## 2. Danh sách module/domain

| Module      | Vai trò                  | Spec chi tiết                  |
|-------------|--------------------------|--------------------------------|
| auth        | Xác thực, phân quyền     | `specs/example-module-auth.md` |
| inventory   | Quản lý tồn kho / RFID   | `specs/inventory.md`           |
| ...         | ...                      | ...                            |

## 3. Ràng buộc hạ tầng

- **[ARCH-01]** Hệ thống shall chạy trên AWS, region `<ap-northeast-1>`.
- **[ARCH-02]** Hệ thống shall lưu toàn bộ dữ liệu khách hàng trong
  `<Nhật Bản>` để tuân thủ APPI (data residency).
- **[ARCH-03]** Mọi pull request shall pass `<lint + unit test + build>`
  trước khi được merge.

> ID `ARCH-xx` dùng để `delta-spec.md` tham chiếu khi thay đổi ràng buộc
> kiến trúc (vd "Sửa ARCH-01: thêm region dự phòng"), và để trace ngược từ
> checklist hạ tầng — cùng quy ước với `AUTH-xx` trong `specs/<module>.md`
> (xem CLAUDE.md mục 7).

## 4. Lịch sử thay đổi kiến trúc lớn

| Ngày       | Ticket ID       | Thay đổi                        |
|------------|-----------------|-----------------------------------|
| YYYY-MM-DD | TICKET-XXX     | <mô tả ngắn thay đổi kiến trúc>  |

<!-- Mỗi dòng ở đây trỏ về changes/_archive/TICKET-XXX/ để xem đầy đủ lý do -->
