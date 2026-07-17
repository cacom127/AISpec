# Spec Template — Hướng dẫn sử dụng

Bộ template này kết hợp triết lý **GitHub Spec Kit** (constitution + tách
business/technical) và **OpenSpec** (current truth `specs/` + delta thay đổi
`changes/`). 

## Cấu trúc

```
spec-template/
├── CLAUDE.md                      # Nguyên tắc bất biến — sửa ngay khi setup dự án
├── specs/                          # Trạng thái HIỆN TẠI (current truth)
│   ├── architecture.md             # Kiến trúc tổng thể
│   └── example-module-auth.md      # Ví dụ spec 1 module — copy & đổi tên khi thêm module
└── changes/
    ├── _template/                  # Copy thư mục này mỗi khi có ticket mới
    │   ├── proposal.md             # Chỉ cần cho size Medium/Large
    │   ├── plan.md                 # Chỉ cần cho size Medium/Large
    │   ├── delta-spec.md           # BẮT BUỘC — mọi size
    │   └── tasks.md                # BẮT BUỘC — mọi size
    └── _archive/                   # Nơi chứa các change đã merge (lịch sử)
```

## Cách bắt đầu 1 dự án mới

1. Copy toàn bộ thư mục `spec-template/` vào root repo, đổi tên tuỳ ý (hoặc
   giữ nguyên cấu trúc con `specs/`, `changes/`, `CLAUDE.md`).
2. Điền `CLAUDE.md`: tên dự án, stack, nguyên tắc kỹ thuật thật của team.
3. Điền `specs/architecture.md` với kiến trúc thật (có thể sơ sài lúc đầu,
   sẽ đầy dần theo thời gian).
4. Xoá file `specs/example-module-auth.md` mẫu, hoặc giữ lại làm tài liệu
   tham khảo cách viết.

## Cách xử lý 1 ticket mới (workflow hàng ngày)

1. Nhận ticket từ Backlog, vd `TICKET-123`.
2. Xác định độ lớn theo bảng trong `CLAUDE.md` mục 4 (Small/Medium/Large).
3. Copy `changes/_template/` → `changes/TICKET-123-mo-ta-ngan/`.
4. Nếu Small: chỉ điền `delta-spec.md` + `tasks.md`, xoá `proposal.md` và
   `plan.md` không dùng.
   Nếu Medium/Large: điền đủ 4 file, theo thứ tự proposal → plan →
   delta-spec → tasks.
5. Đưa cả thư mục `changes/TICKET-123-.../` vào context khi làm việc với
   AI agent (Claude Code/Cursor) cùng với `CLAUDE.md`.
6. Sau khi code xong, test pass, review xong:
   - Gộp nội dung `delta-spec.md` vào file tương ứng trong `specs/`.
   - Cập nhật bảng "Lịch sử thay đổi" trong file `specs/` đó.
   - Di chuyển `changes/TICKET-123-.../` sang `changes/_archive/`.

## Gợi ý dùng với Claude Code / Cursor

- Đặt `CLAUDE.md` ở root — các tool này tự động đọc file này làm system
  context mỗi phiên làm việc.
- Khi bắt đầu 1 task, có thể prompt: *"Đọc changes/TICKET-123-xxx/ và
  specs/auth.md, implement theo delta-spec.md và tasks.md"*.
- Với Cursor, có thể thêm rule trong `.cursor/rules/spec-workflow.mdc` trỏ
  về quy tắc trong `CLAUDE.md` mục 6 (không tự sửa `specs/` trực tiếp).

## Lưu ý quan trọng

- Không sửa `specs/` trực tiếp — mọi thay đổi phải đi qua `changes/` trước.
- Việc quá nhỏ (fix typo, đổi text) không cần cả bộ này — chỉ áp dụng cho
  thay đổi có ảnh hưởng đến hành vi hệ thống hoặc cần AI agent code theo.
- Giữ acceptance criteria (EARS notation) đủ cụ thể để map sang test case
  — đây là cách chống "spec chỉ để đọc, không ai enforce".
