# CLAUDE.md — Constitution của dự án

> File này là NGUYÊN TẮC BẤT BIẾN, luôn được AI agent (Claude Code, Cursor...) nạp
> vào context đầu tiên, bất kể đang làm task nào. Chỉ sửa file này khi có quyết
> định ảnh hưởng TOÀN dự án — không dùng để ghi chi tiết 1 feature.
>
> Quy ước: mỗi mục dùng EARS notation ("The system shall...") để AI đọc không
> phải đoán, và có thể sinh test/checklist trực tiếp từ đó.

## 1. Bối cảnh dự án

- **Tên dự án**: <tên hệ thống>
- **Khách hàng**: <tên khách hàng>
- **Stack chính**: <Flutter / Web / AWS / ...>
- **Repo**: <đường dẫn repo>
- **Ngôn ngữ giao tiếp với khách hàng**: 日本語 (tài liệu 提案書/見積書/テスト仕様書 viết tiếng Nhật)
- **Ngôn ngữ code/comment**: <English/Tiếng Việt>

## 2. Nguyên tắc kỹ thuật bất biến

<!-- Ví dụ mẫu, thay bằng nguyên tắc thật của dự án -->

- The system shall use <ngôn ngữ/framework> version <X> or higher.
- The system shall not introduce a runtime dependency that has not been
  updated in the last 12 months.
- The system shall enforce lint/format rules defined in `<config file>`
  before merge.
- The system shall not lower existing test coverage on any pull request.
- The system shall log all external API calls with request/response status
  for traceability (yêu cầu audit của khách hàng Nhật).

## 3. Quy ước tổ chức spec (áp dụng cho toàn dự án)

- The system's specification lives in `specs/` (current truth — trạng thái
  đã chốt, đã merge) and `changes/` (đề xuất/thay đổi đang thực hiện).
- Every change shall be tracked using the ticket ID from Backlog
  (vd: `changes/TICKET-123-add-2fa/`) để trace ngược lại ticket gốc.
- Every `changes/<ticket-id>/` folder shall contain at minimum
  `delta-spec.md` and `tasks.md`. `proposal.md` and `plan.md` are optional,
  required only for Medium/Large changes (xem mục 4).
- When a change is merged, its `delta-spec.md` shall be folded into the
  corresponding file under `specs/`, and the `changes/<ticket-id>/` folder
  shall be moved to `changes/_archive/`.

## 4. Phân loại độ lớn thay đổi (Change Sizing)

| Size  | Tiêu chí                                              | File bắt buộc                              |
|-------|--------------------------------------------------------|---------------------------------------------|
| Small | Fix bug, đổi 1 field/UI nhỏ, < nửa ngày công          | `delta-spec.md`, `tasks.md`                 |
| Medium| Thêm tính năng trong module có sẵn                    | + `proposal.md`                             |
| Large | Tính năng mới ảnh hưởng nhiều module / đổi kiến trúc  | + `proposal.md`, `plan.md` (cần duyệt trước khi vào `tasks.md`) |

## 5. Quy tắc viết acceptance criteria

- The system's acceptance criteria shall be written in EARS notation:
  `[Điều kiện/trigger] the system shall [hành vi mong đợi]`.
- Every acceptance criterion shall be traceable to at least one test case
  (unit/integration/manual test theo format Excel test case của dự án).
- Ambiguous verbs (vd: "nhanh", "thân thiện", "ổn định") shall be replaced
  with measurable criteria (vd: "phản hồi trong < 2s với 100 concurrent user").

## 6. Không được làm gì (Explicit non-goals / cấm)

- The AI agent shall NOT modify files under `specs/` directly without a
  corresponding entry in `changes/`.
- The AI agent shall NOT invent business requirements not present in
  `proposal.md` / `delta-spec.md` — nếu thiếu thông tin, phải hỏi lại.
- The AI agent shall NOT delete or rewrite historical folders under
  `changes/_archive/`.

## 7. Liên hệ / người chốt quyết định

- **Product/Business owner**: <tên/role>
- **Technical owner**: <tên/role>
- **Khách hàng phía Nhật (liên hệ nếu cần confirm requirement)**: <tên/role>
