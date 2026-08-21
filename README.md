# Spec Template — Hướng dẫn sử dụng

Bộ template này kết hợp triết lý **GitHub Spec Kit** (constitution + tách
business/technical) và **OpenSpec** (current truth `specs/` + delta thay đổi
`changes/`).

## Cấu trúc

```
spec-template/
├── CLAUDE.md                      # Nguyên tắc bất biến — sửa ngay khi setup dự án
├── DESIGN.md                       # Design system (màu/font/spacing/component) — xem mục riêng bên dưới
├── specs/                          # Trạng thái HIỆN TẠI (current truth)
│   ├── vision.md                   # Bài toán, đối tượng dùng, phạm vi
│   ├── architecture.md             # Kiến trúc tổng thể
│   ├── data-model.md               # CHỈ ER tổng quan (tên bảng + quan hệ) + quy ước chung
│   │                                #   (field-level chi tiết nằm ở specs/<module>.md — xem
│   │                                #    quy tắc "ownership entity" trong CLAUDE.md mục 4)
│   ├── cross-cutting/              # Chính sách áp dụng MỌI module (không thuộc riêng module nào)
│   │   ├── error-handling.md       #   Chiến lược xử lý lỗi + catalog error code
│   │   └── logging.md              #   Convention logging: level, format, retention
│   ├── example-module-auth.md      # Ví dụ spec 1 module — copy & đổi tên khi thêm module
│   │                                #   (thêm mục "## UI" nếu module đơn giản)
│   └── example-module-auth-ui.md   # Ví dụ UI feature spec riêng — dùng khi module có
│                                    #   nhiều màn hình (xem CLAUDE.md mục 5)
└── changes/
    ├── _template/                  # Copy thư mục này mỗi khi có ticket mới — KỂ CẢ ticket đầu tiên
    │   ├── proposal.md             # Chỉ cần cho size Medium/Large
    │   ├── plan.md                 # Chỉ cần cho size Medium/Large
    │   ├── delta-spec.md           # BẮT BUỘC — mọi size (xem "Các mục trong delta-spec.md" bên dưới)
    │   ├── ui-delta-spec.md        # OPTIONAL — chỉ khi UI đủ phức tạp để tách riêng
    │   └── tasks.md                # BẮT BUỘC — mọi size
    └── _archive/                   # Nơi chứa các change đã merge (lịch sử)
```

## Nguyên tắc: mọi thay đổi đều qua `changes/`, KHÔNG có ngoại lệ

Kể cả lúc khởi tạo dự án (khi `specs/` còn hoàn toàn trống), flow vẫn giống
1 ticket bình thường — chỉ khác là `delta-spec.md` lúc này toàn mục
`(MỚI)` (vì baseline = rỗng), và size chắc chắn là Large. Xem "Flow cho dự
án mới hoàn toàn" bên dưới.

## Cách xử lý 1 ticket mới — bao gồm cả ticket đầu tiên

1. Nhận ticket từ Backlog (vd `TICKET-1` cho lần khởi tạo, `TICKET-123`
   cho các lần sau).
2. Xác định độ lớn theo bảng trong `CLAUDE.md` mục 6 (Small/Medium/Large).
   Ticket khởi tạo dự án luôn là **Large**.
3. Copy `changes/_template/` → `changes/TICKET-123-mo-ta-ngan/`.
4. Viết theo ĐÚNG THỨ TỰ (mỗi file phụ thuộc quyết định của file trước —
   xem lý do trong `docs/design-decisions.md` mục 5):
   - `proposal.md` (Why) → xác nhận trước khi qua bước sau
   - `plan.md` (kiến trúc/kỹ thuật) → xác nhận trước khi qua bước sau
   - `delta-spec.md` (requirement cụ thể, EARS notation, đánh dấu
     `(MỚI)`/`(SỬA)`/`(XOÁ)` — xem "Các mục trong delta-spec.md" bên dưới)
     → xác nhận trước khi qua bước sau
   - `tasks.md` (breakdown để code)
   Nếu size Small: chỉ cần `delta-spec.md` + `tasks.md`, bỏ qua 2 file đầu
   và 2 checkpoint đầu.
   Nếu ticket là refactor thuần (không đổi behavior/contract nào):
   `delta-spec.md` vẫn BẮT BUỘC tồn tại, nhưng mục 1 chỉ cần ghi
   `**Không đổi spec** — lý do: <mô tả>` (xem `CLAUDE.md` mục 4). Vẫn phải
   điền mục 1d — refactor là nơi rủi ro vỡ ngầm cao nhất.
5. Đưa cả thư mục `changes/TICKET-123-.../` vào context khi làm việc với
   AI agent (Claude Code/Cursor) cùng với `CLAUDE.md`.
6. Sau khi code xong, test pass, review xong:
   - Gộp nội dung `delta-spec.md` vào file tương ứng trong `specs/`,
     **tạo file mới nếu chưa tồn tại** (đúng trường hợp ticket đầu tiên).
     Mục 1c fold vào `specs/<module>-ui.md`. **Mục 0 và 1d KHÔNG fold** —
     chúng chỉ có hiệu lực trong phạm vi ticket.
   - Cập nhật bảng "Lịch sử thay đổi" trong file `specs/` đó.
   - **Verify trước khi archive**: mọi checkbox trong `tasks.md` đã tick
     xong, VÀ mọi mục cần fold trong `delta-spec.md` đã fold vào đúng file
     `specs/`. Chưa đủ 2 điều kiện thì KHÔNG được archive (`CLAUDE.md`
     mục 4).
   - Di chuyển `changes/TICKET-123-.../` sang `changes/_archive/`.

## Các mục trong `delta-spec.md`

| Mục | Khi nào cần | Fold vào `specs/` khi merge? |
|-----|--------------|-------------------------------|
| **0. Phân tích bug** | Chỉ khi ticket là fix bug — hành vi hiện tại (đang sai), hành vi mong đợi, nguyên nhân gốc. Phân loại A (code ≠ spec → spec KHÔNG đổi) hoặc B (spec sai/thiếu → phải sửa spec) | ❌ Không — thuộc về ticket |
| **1. Yêu cầu thay đổi** | Luôn (refactor thuần → chỉ ghi "Không đổi spec") | ✅ `specs/<module>.md` |
| **1b. Data model** | Khi thêm/sửa field hoặc bảng | ✅ module sở hữu (+ `data-model.md` nếu thêm/xoá bảng hoặc đổi quan hệ) |
| **1c. Thay đổi UI** | UI đơn giản 1–2 màn hình; phức tạp hơn → tách `ui-delta-spec.md` | ✅ `specs/<module>-ui.md` |
| **1d. Regression guard** | Khi có (SỬA)/(XOÁ), fix bug, hoặc refactor thuần — liệt kê hành vi KHÔNG được làm vỡ | ❌ Không — hành vi đã có ID gốc trong `specs/` |
| **2. Test mapping** | Luôn — mỗi ID ↔ ít nhất 1 test case | ❌ Không (nội dung test case nằm ngoài spec) |
| **3. Ghi chú cho AI agent** | Khi ticket có ràng buộc riêng | ❌ Không |
| **4. Soát chất lượng** | BẮT BUỘC trước khi chốt | ❌ Không |

Hai quy ước dễ nhầm:

- `shall ...` = cam kết **làm mới** hành vi → map sang test tính năng.
  `shall CONTINUE TO ...` = cam kết **không làm vỡ** hành vi đang chạy đúng
  → map sang regression test.
- Mục 4 soát 5 loại lỗi (mơ hồ / xung đột / thiếu edge case / giả định
  không nói ra / phạm vi), và phải soát trên **cả requirement cũ** cùng
  module — xung đột thường nằm giữa dòng vừa thêm và dòng đã có từ trước.
  AI agent phải **báo cáo và hỏi lại**, không tự sửa requirement.

## Flow cho dự án mới hoàn toàn (lúc `specs/` còn trống)

Vẫn dùng đúng flow ở trên — không có bước riêng. Chỉ khác về nội dung:

```
Ticket đầu tiên: TICKET-1-project-setup (size Large)

changes/TICKET-1-project-setup/
├── proposal.md   → Bài toán kinh doanh, phạm vi dự án
├── plan.md        → Kiến trúc, data model, tech stack đã chọn
├── delta-spec.md   → TOÀN BỘ requirement ban đầu, tất cả đánh dấu (MỚI):
│                     [ARCH-01] (MỚI), [AUTH-01] (MỚI)...
│                     Field-level entity map vào specs/<module>.md (mục
│                     Data Model); quan hệ/tên bảng map vào data-model.md
└── tasks.md        → Breakdown build MVP

Merge → fold delta-spec.md vào specs/vision.md, specs/architecture.md,
specs/<module>.md (field-level, tạo mới từng file vì đang rỗng), và
specs/data-model.md (chỉ ER tổng quan)
→ archive changes/TICKET-1-project-setup/
```

Nếu công ty bạn chưa tạo Backlog project lúc bắt đầu code (chưa có ticket
ID thật), tạm dùng `000-init` làm tên thư mục, rồi **rename lại** thành
ticket ID thật khi archive — chỉ đổi tên, không đổi nội dung/flow.

## Cách dùng DESIGN.md (design system)

`DESIGN.md` theo đúng format chuẩn mở của Google (`google-labs-code/design.md`)
— gồm YAML token (màu, font, spacing, component) + phần markdown giải thích lý
do. File mẫu trong template đã có sẵn ví dụ đầy đủ, chỉnh lại giá trị cho
đúng brand thật của dự án.

1. **Không có lệnh tự sinh file** — bạn phải tự viết/chỉnh tay, hoặc nhờ AI
   phân tích ảnh chụp màn hình app hiện tại để soạn bản nháp, hoặc dùng
   Google Stitch để generate từ mô tả brand.
2. **Validate trước khi merge thay đổi UI**:
   ```bash
   npx @google/design.md lint DESIGN.md
   ```
   Bắt lỗi token tham chiếu sai, thiếu màu primary, tương phản không đạt
   WCAG AA...
3. **So sánh khi đổi design system** (vd đổi bảng màu):
   ```bash
   npx @google/design.md diff DESIGN.md DESIGN-v2.md
   ```
4. **Export sang Tailwind/DTCG nếu cần** (web admin dùng Tailwind, hoặc cầu
   nối sang thư viện Flutter hỗ trợ chuẩn DTCG):
   ```bash
   npx @google/design.md export --format css-tailwind DESIGN.md > theme.css
   ```
5. **Khi prompt AI agent**, có thể nhắc thẳng: *"Build màn hình X theo đúng
   token trong DESIGN.md"* — không cần giải thích lại màu/font mỗi lần.

**Quan trọng — chống drift**: nếu code dùng 1 giá trị màu/font khác với
`DESIGN.md`, phải sửa lại 1 trong 2 để khớp nhau trước khi merge (xem
CLAUDE.md mục 3). Không để cả hai cùng tồn tại khác nhau — nếu không,
`DESIGN.md` sẽ mất tác dụng làm nguồn chân lý.

**Phân biệt với UI feature spec**: `DESIGN.md` chỉ mô tả token/component ở
mức atomic ("nút này trông thế nào"). Layout, trạng thái màn hình
(loading/error/empty), và hành vi tương tác của từng chức năng cụ thể vẫn
viết riêng trong `specs/<module>-ui.md` (xem CLAUDE.md mục 5) — hai file
này bổ sung cho nhau, không thay thế nhau.

## Cross-cutting concern (error handling, logging...)

Những spec dùng chung cho MỌI module (không thuộc riêng module nào) nằm
trong `specs/cross-cutting/` — phân biệt với `vision.md`/`architecture.md`/
`data-model.md` (tả **cấu trúc** hệ thống) bằng tiêu chí: cross-cutting tả
**chính sách hành vi** lặp lại ở mọi nơi (log ra sao, lỗi trả về ra sao).
Khi thêm error code hoặc đổi convention log, vẫn đi qua `changes/` như mọi
thay đổi khác — không có exception riêng.

## Gợi ý dùng với Claude Code / Cursor

- Đặt `CLAUDE.md` và `DESIGN.md` ở root — các tool này tự động đọc 2 file
  này làm system context mỗi phiên làm việc (đặc biệt khi task có đụng UI).
- Khi bắt đầu 1 task, có thể prompt: *"Đọc changes/TICKET-123-xxx/ và
  specs/auth.md, implement theo delta-spec.md và tasks.md"*.
- Với Cursor, có thể thêm rule trong `.cursor/rules/spec-workflow.mdc` trỏ
  về quy tắc trong `CLAUDE.md` mục 8 (không tự sửa `specs/` trực tiếp).

## Lưu ý quan trọng

- Không sửa `specs/` trực tiếp — mọi thay đổi phải đi qua `changes/`
  trước, không có ngoại lệ, kể cả lúc `specs/` còn trống.
- Việc quá nhỏ (fix typo, đổi text) không cần cả bộ này — chỉ áp dụng cho
  thay đổi có ảnh hưởng đến hành vi hệ thống hoặc cần AI agent code theo.
- Giữ acceptance criteria (EARS notation) đủ cụ thể để map sang test case
  — đây là cách chống "spec chỉ để đọc, không ai enforce".
- Không để `DESIGN.md` trôi lệch khỏi code thật — coi nó như 1 nguồn chân
  lý, không phải tài liệu tham khảo (xem mục "Cách dùng DESIGN.md" ở trên).
