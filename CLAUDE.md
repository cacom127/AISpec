# CLAUDE.md — Constitution của dự án

> File này là NGUYÊN TẮC BẤT BIẾN, luôn được AI agent (Claude Code, Cursor...) nạp
> vào context đầu tiên, bất kể đang làm task nào. Chỉ sửa file này khi có quyết
> định ảnh hưởng TOÀN dự án — không dùng để ghi chi tiết 1 feature.
>
> Quy ước: mỗi mục dùng EARS notation ("... hệ thống shall ...") để AI đọc không
> phải đoán, và có thể sinh test/checklist trực tiếp từ đó.

## 1. Bối cảnh dự án

- **Tên dự án**: <tên hệ thống>
- **Khách hàng**: <tên khách hàng>
- **Stack chính**: <Flutter / Web / AWS / ...>
- **Repo**: <đường dẫn repo>
- **Ngôn ngữ giao tiếp với khách hàng**: 日本語 (tài liệu 提案書/見積書/テスト仕様書 viết tiếng Nhật)
- **Ngôn ngữ code/comment**: <English/Tiếng Việt>
- **Ngôn ngữ viết spec/EARS**: Tiếng Việt, giữ nguyên 3 từ khoá tiếng Anh
  `shall` / `shall NOT` / `shall CONTINUE TO` (xem mục 7)

## 2. Nguyên tắc kỹ thuật bất biến

<!-- Ví dụ mẫu, thay bằng nguyên tắc thật của dự án -->

- Hệ thống shall dùng <ngôn ngữ/framework> phiên bản <X> hoặc mới hơn.
- Hệ thống shall NOT thêm runtime dependency đã không được cập nhật trong
  12 tháng gần nhất.
- Hệ thống shall enforce lint/format rule định nghĩa trong `<config file>`
  trước khi merge.
- Hệ thống shall NOT làm giảm test coverage hiện có ở bất kỳ pull request.
- Hệ thống shall log mọi external API call kèm request/response status để
  phục vụ truy vết (yêu cầu audit của khách hàng Nhật). Chi tiết
  format/level/retention xem `specs/cross-cutting/logging.md`; chiến lược
  xử lý lỗi và catalog error code xem `specs/cross-cutting/error-handling.md`.

## 3. Design system (UI)

- Visual identity của hệ thống (color, typography, spacing, component
  style) nằm trong `DESIGN.md` ở project root — theo format chuẩn
  `google-labs-code/design.md`.
- AI agent shall đọc `DESIGN.md` trước khi sinh hoặc sửa bất kỳ code UI
  nào, và shall NOT tự nghĩ ra giá trị màu/font/spacing ngoài những gì đã
  được định nghĩa ở đó.
- Nếu cần 1 giá trị design chưa có trong `DESIGN.md` (màu mới, component
  mới...), AI agent shall cập nhật `DESIGN.md` trước, không hardcode giá
  trị trực tiếp trong code UI.
- `DESIGN.md` chỉ mô tả token/style của component ở mức atomic (nút này
  trông thế nào). Layout/hành vi/state của TỪNG màn hình/chức năng cụ thể
  được mô tả riêng trong `specs/<module>-ui.md` (xem mục 5).
- Trước khi merge thay đổi UI, chạy `npx @google/design.md lint DESIGN.md`
  để kiểm tra token hợp lệ và tương phản WCAG.
- Hệ thống shall NOT chấp nhận một giá trị trong code trái với `DESIGN.md`.
  Nếu code và file lệch nhau, phải sửa 1 trong 2 để khớp lại trước khi
  merge — không được để cả hai cùng tồn tại khác nhau.

## 4. Quy ước tổ chức spec (áp dụng cho toàn dự án)

- Specification của hệ thống nằm ở `specs/` (current truth — trạng thái đã
  chốt, đã merge) và `changes/` (đề xuất/thay đổi đang thực hiện).
- `specs/` gồm: `vision.md`, `architecture.md` (nền tảng — ít đổi),
  `data-model.md` (**CHỈ** ER tổng quan + quy ước chung — ít đổi, KHÔNG
  chứa field-level chi tiết), `api-catalog.md` (**CHỈ** danh sách endpoint
  + quy ước chung — KHÔNG chứa schema chi tiết), `cross-cutting/` (chính
  sách áp dụng mọi module — vd error-handling, logging), và `<module>.md`
  cho từng domain nghiệp vụ.
- **Quy tắc ownership entity**: mỗi entity/bảng dữ liệu shall có ĐÚNG 1
  module sở hữu. Field/type/constraint chi tiết của entity đó được viết
  trong `specs/<module-sở-hữu>.md` (mục `## Data Model`), KHÔNG viết trong
  `specs/data-model.md`. `specs/data-model.md` chỉ ghi tên bảng, quan hệ,
  và bảng mapping entity → module sở hữu. Module khác nếu chỉ tham chiếu
  (FK) tới entity đó thì KHÔNG lặp lại field, chỉ ghi chú tham chiếu.
- **Quy tắc ownership endpoint**: mỗi API endpoint shall có ĐÚNG 1 module
  sở hữu. Request/response schema chi tiết được viết trong
  `specs/<module-sở-hữu>.md`, KHÔNG viết trong `specs/api-catalog.md` —
  file này chỉ giữ 1 dòng/endpoint (method, path, module sở hữu). Module
  khác nếu chỉ GỌI endpoint đó thì KHÔNG lặp lại schema, chỉ ghi chú tham
  chiếu.
- Khi 1 thay đổi chỉ thêm/sửa field của 1 entity đã tồn tại → chỉ động vào
  `specs/<module-sở-hữu>.md`, KHÔNG động vào `specs/data-model.md`. Chỉ
  khi THÊM/XOÁ hẳn 1 bảng, hoặc thay đổi quan hệ giữa các bảng, mới cần
  cập nhật thêm `specs/data-model.md`.
- Mọi thay đổi shall được track bằng ticket ID từ Backlog
  (vd `changes/TICKET-123-add-2fa/`) để trace ngược lại ticket gốc.
- Mọi thư mục `changes/<ticket-id>/` shall có tối thiểu `delta-spec.md` và
  `tasks.md`. `proposal.md` và `plan.md` là optional, chỉ bắt buộc với
  thay đổi Medium/Large (xem mục 6).
- Khi một thay đổi được merge, `delta-spec.md` của nó shall được fold vào
  file tương ứng dưới `specs/`, **tạo file mới nếu chưa tồn tại** (vd
  ticket khởi tạo dự án đầu tiên, khi toàn bộ `specs/` còn rỗng — không có
  exception riêng, `delta-spec.md` lúc này chỉ toàn mục `(MỚI)`). Sau đó
  `changes/<ticket-id>/` shall được move sang `changes/_archive/`.
- **Thay đổi thuần kỹ thuật, không đổi spec** (vd: refactor nội bộ, đổi
  ORM/thư viện, tối ưu performance mà không đổi behavior/contract nào):
  `delta-spec.md` vẫn BẮT BUỘC phải tồn tại (không được bỏ qua), nhưng chỉ
  cần ghi `**Không đổi spec** — lý do: <mô tả>` thay vì liệt kê mục
  MỚI/SỬA/XOÁ ở mục 1. Không tự ý suy diễn "refactor nên khỏi cần
  delta-spec" — nếu có bất kỳ thay đổi acceptance criteria nào dù nhỏ,
  phải quay lại viết đầy đủ mục 1.
- Trước khi move `changes/<ticket-id>/` sang `changes/_archive/`, AI agent
  shall verify: (1) mọi checkbox trong `tasks.md` đã tick xong, (2) mọi
  mục trong `delta-spec.md` đã được fold vào đúng file `specs/` tương ứng
  (kể cả trường hợp "Không đổi spec" ở trên). Nếu chưa đủ 1 trong 2 điều
  kiện, KHÔNG được archive.
- Mục regression guard (`delta-spec.md` mục 1d) shall NOT được fold vào
  `specs/` khi merge — hành vi được bảo vệ vốn đã tồn tại trong `specs/`
  dưới ID gốc, viết lại sẽ lặp dữ liệu. Mục này chỉ có hiệu lực trong phạm
  vi ticket và được archive cùng ticket. Cách viết xem mục 7.
- **Ticket fix bug**: `delta-spec.md` shall có mục 0 (phân tích bug) —
  hành vi hiện tại (đang sai), hành vi mong đợi, và nguyên nhân gốc. Bug
  analysis KHÔNG phải requirement. Mỗi bug shall được phân loại ĐÚNG 1
  trong 2: (A) code ≠ spec → spec KHÔNG đổi, mục 1 ghi "Không đổi spec";
  (B) spec sai/thiếu → mục 1 phải có mục (SỬA)/(MỚI) tương ứng. Mục 0
  KHÔNG fold vào `specs/` (giống mục 1d) — nó thuộc về ticket, không phải
  current truth.

## 5. Quy ước tổ chức UI feature spec

- Layout, trạng thái màn hình (loading/error/empty...), và hành vi tương
  tác của TỪNG chức năng cụ thể được viết trong `specs/<module>-ui.md`
  (mẫu: `specs/example-module-auth-ui.md`), hoặc mục `## UI` trong
  `specs/<module>.md` nếu module chỉ có 1 màn hình đơn giản.
- UI feature spec tham chiếu ngược lại token trong `DESIGN.md`, không lặp
  lại giá trị màu/font cụ thể — chỉ ghi tên component/token.
- Mọi ĐIỀU HƯỚNG giữa các màn hình shall được viết thành 1 dòng EARS trong
  `specs/<module>-ui.md`, KHÔNG được chỉ nằm trong ASCII layout hoặc ảnh
  mockup — layout/ảnh không phải nguồn chân lý (xem `docs/design-decisions.md`
  mục 6). Dòng EARS shall ghi rõ MÀN HÌNH ĐÍCH.
- Nếu màn hình đích thuộc module khác, dòng EARS shall ghi thêm tên module
  đó. **Module chứa màn hình NGUỒN sở hữu cạnh điều hướng** — module đích
  KHÔNG lặp lại. Nhờ 2 luật này, 画面遷移図 suy ra được trực tiếp từ
  `specs/<module>-ui.md` (node = bảng danh sách màn hình, edge = các dòng
  điều hướng), không cần file sơ đồ riêng.
- Trong `changes/<ticket-id>/`: thay đổi UI đơn giản ghi thẳng vào mục
  "## 1c" của `delta-spec.md`; thay đổi UI phức tạp (nhiều màn hình) tách
  riêng `ui-delta-spec.md` (optional, xem mẫu trong
  `changes/_template/ui-delta-spec.md`).
- Khi merge, mục "## 1c" của `delta-spec.md` (và `ui-delta-spec.md` nếu có)
  shall được fold vào `specs/<module>-ui.md`, tạo file mới nếu chưa tồn
  tại — KHÔNG fold vào `DESIGN.md` (file đó chỉ chứa token/component
  atomic, không chứa layout/behavior riêng từng ticket).
- Ảnh/mockup/Figma chỉ là tài liệu tham khảo (mục "Tham chiếu thiết kế"),
  KHÔNG phải nguồn chân lý — nguồn chân lý luôn là nội dung text (EARS +
  state matrix), vì ảnh không diff/trace/test-map được.

## 6. Phân loại độ lớn thay đổi (Change Sizing)

| Size  | Tiêu chí                                              | File bắt buộc                              |
|-------|--------------------------------------------------------|---------------------------------------------|
| Small | Fix bug, đổi 1 field/UI nhỏ, < nửa ngày công          | `delta-spec.md`, `tasks.md`                 |
| Medium| Thêm tính năng trong module có sẵn                    | + `proposal.md`                             |
| Large | Tính năng mới ảnh hưởng nhiều module / đổi kiến trúc  | + `proposal.md`, `plan.md` (cần duyệt trước khi vào `tasks.md`) |

## 7. Quy tắc viết acceptance criteria

- Acceptance criteria của hệ thống shall được viết theo EARS notation:
  `[Điều kiện/trigger] hệ thống shall [hành vi mong đợi]`.
- EARS shall được viết bằng TIẾNG VIỆT, giữ nguyên ĐÚNG 3 từ khoá tiếng
  Anh: `shall`, `shall NOT`, `shall CONTINUE TO`. Giữ 3 từ này vì chúng là
  phần chịu lực của notation — grep ra được toàn bộ requirement, mọi điều
  cấm, và mọi regression guard; dịch thành "phải"/"không được" sẽ mất khả
  năng đó. Trigger dùng "Khi ..." / "Trong khi ..." / "Nếu ...". Thuật ngữ
  kỹ thuật (token, endpoint, HTTP 401, timestamp...) giữ tiếng Anh.
- Mọi acceptance criterion shall trace được tới ít nhất 1 test case
  (unit/integration/manual test theo format Excel test case của dự án).
- Động từ mơ hồ (vd "nhanh", "thân thiện", "ổn định") shall được thay bằng
  tiêu chí đo được (vd "phản hồi trong < 2s với 100 concurrent user").
- Khi một thay đổi sửa hoặc xoá hành vi đang có (có mục (SỬA)/(XOÁ) ở
  `delta-spec.md`, ticket là fix bug, hoặc ticket thuần refactor "Không
  đổi spec"), `delta-spec.md` shall khai báo regression guard (mục 1d)
  liệt kê những hành vi hiện có KHÔNG được thay đổi, viết dạng
  `**[ID] (GIỮ NGUYÊN)** ... hệ thống shall CONTINUE TO ...`.
- Quy ước phân biệt 2 loại cam kết — để AI agent, dev và tester đọc là
  biết ngay dòng đó thuộc loại nào:
  - `shall ...` = cam kết LÀM MỚI hành vi này → map sang test tính năng.
  - `shall CONTINUE TO ...` = cam kết KHÔNG LÀM VỠ một hành vi đang chạy
    đúng → map sang regression test.
- Mọi dòng `shall CONTINUE TO` shall trace được tới ít nhất 1 regression
  test case (ghi ở mục 2 của `delta-spec.md`).
- Regression guard shall CHỈ tham chiếu ID requirement đã tồn tại trong
  `specs/`, KHÔNG lặp lại nội dung requirement, và CHỈ liệt kê requirement
  dùng chung code path / chung entity với phần đang sửa (thường 2–5 dòng,
  không liệt kê cả module).
- Trước khi chốt `delta-spec.md`, AI agent shall chạy một requirement
  quality review (mục 4 của `delta-spec.md`), soát 5 loại lỗi: mơ hồ
  (ambiguity), xung đột (conflict), thiếu edge case (completeness), giả
  định không nói ra (unstated assumption), và phạm vi cố ý loại trừ
  (scenario in/out).
- Bước soát trên shall chạy trên CẢ requirement mới LẪN requirement cũ
  cùng module trong `specs/<module>.md` — xung đột thường nằm giữa dòng
  vừa thêm và dòng đã tồn tại từ trước, không nằm trong dòng mới.
- Khi bước soát phát hiện vấn đề, AI agent shall BÁO CÁO và HỎI LẠI
  người chốt, và shall NOT tự ý sửa nội dung requirement. Mỗi phát hiện
  shall kết thúc bằng đúng 1 trong 2 kết luận — giữ nguyên, hoặc sửa
  thành <nội dung mới> — và được ghi vào bảng ở mục 4 để trace khi review.

## 8. Không được làm gì (Explicit non-goals / cấm)

- AI agent shall NOT sửa file dưới `specs/` trực tiếp mà không có entry
  tương ứng trong `changes/` — không có ngoại lệ, kể cả ticket khởi tạo
  dự án đầu tiên (xem mục 4).
- AI agent shall NOT tự nghĩ ra business requirement không có trong
  `proposal.md` / `delta-spec.md` — nếu thiếu thông tin, phải hỏi lại.
- AI agent shall NOT xoá hoặc viết lại các thư mục lịch sử dưới
  `changes/_archive/`.
- AI agent shall NOT sửa `DESIGN.md` một cách tuỳ tiện trong lúc implement
  feature — thay đổi design token là quyết định riêng, cần được ghi trong
  `changes/<ticket-id>/delta-spec.md` như mọi thay đổi khác nếu ảnh hưởng
  nhiều màn hình.

## 9. Liên hệ / người chốt quyết định

- **Product/Business owner**: <tên/role>
- **Technical owner**: <tên/role>
- **Khách hàng phía Nhật (liên hệ nếu cần confirm requirement)**: <tên/role>
