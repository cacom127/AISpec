# Delta Spec — <Tên thay đổi ngắn>

> File BẮT BUỘC cho mọi thay đổi (Small/Medium/Large). Chỉ ghi PHẦN THAY ĐỔI
> so với spec hiện tại trong `specs/` — không viết lại toàn bộ spec module.
> Khi merge, nội dung này sẽ được gộp vào file tương ứng trong `specs/`
> (tạo file mới nếu chưa tồn tại — vd ticket khởi tạo dự án đầu tiên, khi
> `specs/` còn trống; lúc đó mọi mục ở đây đều là `(MỚI)`, không có ngoại
> lệ về flow — xem CLAUDE.md mục 4).

- **Ticket ID**: <TICKET-123>
- **Module bị ảnh hưởng**: `specs/<tên-module>.md`
- **Loại thay đổi**: ☐ Thêm mới &nbsp; ☐ Sửa &nbsp; ☐ Xoá &nbsp; ☐ Không đổi spec (refactor thuần)
- **Ticket là fix bug**: ☐ Có (BẮT BUỘC điền mục 0) &nbsp; ☐ Không

<!-- Nếu tick "Không đổi spec": xoá mục 1/1b/2 bên dưới, chỉ ghi:
**Không đổi spec** — lý do: <vd: đổi ORM, tối ưu query, không đổi
behavior/contract nào>. Xem CLAUDE.md mục 4. Nếu có bất kỳ thay đổi
acceptance criteria nào dù nhỏ, KHÔNG được tick mục này — phải viết đầy
đủ mục 1. -->

## 0. Phân tích bug (CHỈ khi ticket là fix bug)

> Bug analysis KHÔNG phải requirement — mục này trả lời "đang sai cái gì và
> vì sao", TRƯỚC khi quyết định có phải sửa spec hay không. Xoá cả mục này
> nếu ticket không phải fix bug.
>
> Phân loại ĐÚNG 1 trong 2 (quyết định luôn mục 1 viết gì):
> - **Loại A — code ≠ spec** (spec đúng, code làm sai): spec KHÔNG đổi →
>   mục 1 chỉ ghi `**Không đổi spec**`, sửa code là đủ.
> - **Loại B — spec sai/thiếu** (code làm đúng theo spec nhưng spec sai):
>   PHẢI sửa spec → mục 1 có mục (SỬA)/(MỚI) tương ứng.
>
> Mục này KHÔNG fold vào `specs/` (giống mục 1d) — nó thuộc về ticket.

- **Loại bug**: ☐ A — code ≠ spec &nbsp; ☐ B — spec sai/thiếu
- **Requirement liên quan**: <vd: AUTH-02 — hoặc "chưa có requirement nào phủ">

### 0a. Hành vi hiện tại (đang sai)

<!-- KHÔNG dùng "shall" ở đây — đây là mô tả thực tế, không phải cam kết -->
- When a user logs in successfully after 2 failed attempts, the system
  **does not reset the failed-attempt counter** — lần sai thứ 3 (dù cách
  đó vài giờ) vẫn làm account bị lock.

### 0b. Hành vi mong đợi (đúng)

- When a user submits valid credentials, the system shall reset the
  failed-attempt counter to 0.

<!-- Loại B: dòng trên PHẢI xuất hiện lại ở mục 1 dưới dạng (SỬA)/(MỚI).
     Loại A: dòng trên chỉ diễn giải requirement đã có, KHÔNG thêm ID mới. -->

### 0c. Nguyên nhân gốc (root cause)

- <vd: `AuthService.onLoginSuccess()` không gọi `resetFailedAttempts()`;
  counter chỉ bị xoá bởi cron dọn mỗi 15 phút.>

> Sau khi điền mục 0, BẮT BUỘC điền tiếp mục 1d — fix bug là loại thay đổi
> dễ làm vỡ chỗ khác nhất.

## 1. Yêu cầu thay đổi (EARS notation)

<!-- Nếu THÊM mới, đánh số tiếp theo ID cuối cùng trong specs/<module>.md -->
- **[AUTH-05] (MỚI)** When a user requests password reset, the system
  shall send a one-time link valid for 30 minutes.

<!-- Nếu SỬA, ghi rõ ID cũ và nội dung cũ/mới -->
- **[AUTH-02] (SỬA)**
  - Cũ: lock account sau 5 lần thử trong 1 phút.
  - Mới: lock account sau 3 lần thử trong 1 phút (theo yêu cầu security
    audit của khách hàng).

<!-- Nếu XOÁ, ghi rõ ID và lý do -->
- **[XXX-YY] (XOÁ)** — Lý do: <tính năng không còn dùng>

## 1b. Thay đổi Data Model (nếu có)

> Map đúng file — xem CLAUDE.md mục 4 (quy tắc ownership entity):
> - Chỉ thêm/sửa FIELD của entity đã tồn tại → ghi ở đây, fold vào
>   `specs/<module-sở-hữu>.md` mục "Data Model". KHÔNG động vào
>   `specs/data-model.md`.
> - THÊM/XOÁ hẳn 1 bảng, hoặc đổi quan hệ giữa các bảng → ghi thêm 1 dòng
>   cho `specs/data-model.md` (chỉ tên bảng + quan hệ, không field).

<!-- Ví dụ: thêm field vào bảng đã có (chỉ động vào module sở hữu) -->
- **[DM-AUTH-04] (MỚI — field)** Thêm field `User.phone` (string,
  nullable) — map vào `specs/example-module-auth.md` mục Data Model.

<!-- Ví dụ: thêm bảng mới (động vào CẢ module sở hữu LẪN data-model.md) -->
- **[DM-ORDERS-01] (MỚI — bảng)** Thêm bảng `Order`, `OrderItem` — map
  field-level vào `specs/orders.md` mục Data Model, VÀ thêm dòng quan hệ
  `USER ||--o{ ORDER : places` vào `specs/data-model.md` mục 1.

## 1c. Thay đổi UI (nếu có)

> Chỉ dùng cho thay đổi UI ĐƠN GIẢN (1–2 màn hình, vài dòng hành vi). Nếu
> cần state matrix đầy đủ hoặc đụng nhiều màn hình/luồng phức tạp → tách
> riêng `ui-delta-spec.md` và bỏ trống mục này (xem CLAUDE.md mục 5).
>
> - ID theo format `UI-<MODULE>-<số màn hình>-<số thứ tự>` (vd
>   `UI-AUTH-05-1`), đánh số tiếp theo ID cuối trong `specs/<module>-ui.md`.
> - CHỈ ghi TÊN component/token của `DESIGN.md`, KHÔNG ghi giá trị
>   màu/font/spacing cụ thể.
> - Khi merge, mục này fold vào `specs/<module>-ui.md` (tạo file mới nếu
>   chưa tồn tại) — KHÔNG fold vào `DESIGN.md`.

- **Màn hình bị ảnh hưởng**: <vd: Login (SỬA — thêm state "Account Locked")>

<!-- Hành vi tương tác, EARS, đánh dấu MỚI/SỬA/XOÁ giống mục 1 -->
- **[UI-AUTH-02-1] (MỚI)** While tài khoản đang ở trạng thái locked, the
  system shall hiện banner cảnh báo trên màn hình Login kèm thời gian còn
  lại, và disable nút "Đăng nhập".

- **Thay đổi design token trong `DESIGN.md`**: <"Không đổi" — hoặc liệt kê
  token thêm/sửa. BẮT BUỘC ghi nếu ticket sửa `DESIGN.md` và ảnh hưởng
  nhiều màn hình, xem CLAUDE.md mục 8>
- **Tham chiếu thiết kế**: <link Figma/ảnh mockup — chỉ tham khảo, KHÔNG
  phải nguồn chân lý>

## 1d. Hành vi KHÔNG được thay đổi (regression guard)

> BẮT BUỘC khi mục 1 có (SỬA)/(XOÁ), khi ticket là fix bug, hoặc khi ticket
> thuần refactor ("Không đổi spec") — refactor là nơi rủi ro vỡ ngầm cao nhất.
> Ghi lại những hành vi ĐANG CHẠY ĐÚNG mà ticket này KHÔNG được làm vỡ.
>
> Quy ước phân biệt 2 loại cam kết (xem CLAUDE.md mục 7):
> - `shall ...`             → phải LÀM MỚI cái này      → test tính năng
> - `shall CONTINUE TO ...` → phải KHÔNG LÀM VỠ cái này → regression test
>
> Cách viết:
> - CHỈ tham chiếu ID requirement đã có trong `specs/`, KHÔNG viết lại nội
>   dung requirement (tránh lặp dữ liệu).
> - CHỈ liệt kê requirement dùng chung code path / chung entity với phần
>   đang sửa — thường 2–5 dòng, KHÔNG liệt kê cả module.
> - Mục này KHÔNG fold vào `specs/` khi merge (xem CLAUDE.md mục 4) — hành
>   vi được bảo vệ vốn đã tồn tại trong `specs/` dưới ID gốc; regression
>   guard chỉ có hiệu lực trong phạm vi ticket và được archive cùng ticket.

<!-- Ví dụ: ticket giảm ngưỡng lock account từ 5 lần xuống 3 lần (AUTH-02) -->
- **[AUTH-02] (GIỮ NGUYÊN)** When an account is locked, the system shall
  CONTINUE TO keep it locked for exactly 15 minutes — ticket này CHỈ đổi
  ngưỡng số lần thử, KHÔNG đổi thời gian lock.
- **[AUTH-01] (GIỮ NGUYÊN)** When a user submits valid credentials, the
  system shall CONTINUE TO issue a token valid for 8 hours, và bộ đếm số
  lần sai shall CONTINUE TO reset về 0 — việc đổi cách đếm không được ảnh
  hưởng luồng login thành công.

<!-- Nếu ticket chỉ toàn mục (MỚI), không đụng code path nào đang chạy:
     ghi "Không có — ticket chỉ thêm mới, không sửa hành vi hiện có." -->

## 2. Acceptance criteria / Test mapping

| ID       | Test case tương ứng (file/tên)         |
|----------|-------------------------------------------|
| AUTH-05  | `TC-AUTH-15: Reset password qua email`   |
| AUTH-02  | `TC-AUTH-03: Lock account sau N lần sai`  |
| AUTH-02 (giữ nguyên) | `TC-AUTH-09 (regression): Thời gian lock vẫn 15 phút` |
| AUTH-01 (giữ nguyên) | `TC-AUTH-01 (regression): Login đúng vẫn phát token 8h` |
| UI-AUTH-02-1 | `TC-AUTH-UI-04: Banner account locked + disable nút` |

> Mỗi dòng ở mục 1d phải map tới ít nhất 1 regression test case trong bảng
> này (xem CLAUDE.md mục 7).

## 3. Ghi chú cho AI agent khi implement

<Nếu có ràng buộc riêng cho thay đổi này mà constitution không cover, ghi
ở đây — vd: "chỉ sửa file X, không đụng vào Y">

## 4. Soát chất lượng requirement (BẮT BUỘC trước khi chốt)

> Chạy checklist này trên mục 1 + 1b + 1c + 1d, VÀ trên các requirement CŨ
> cùng module trong `specs/<module>.md` — lỗi hay gặp nhất là requirement
> mới xung đột với requirement cũ đã tồn tại (xem CLAUDE.md mục 7).
>
> AI agent shall BÁO CÁO phát hiện và HỎI LẠI người chốt, KHÔNG tự sửa nội
> dung requirement.

- [ ] **Mơ hồ (ambiguity)** — không còn từ định tính không đo được
      ("nhanh", "nhiều", "file lớn", "thân thiện"); mọi con số có đơn vị.
- [ ] **Xung đột (conflict)** — không có 2 requirement (mới↔mới, hoặc
      mới↔cũ) mà riêng lẻ đều hợp lý nhưng cộng lại thì bất khả thi.
- [ ] **Thiếu edge case (completeness)** — đã trả lời: input rỗng/null?
      giá trị biên? gọi đồng thời? mất mạng giữa luồng? không đủ quyền?
- [ ] **Giả định không nói ra (unstated assumption)** — giả định về thứ
      tự, timezone, encoding, đơn vị tiền tệ... đã viết thành chữ.
- [ ] **Phạm vi (scenario in/out)** — đã ghi rõ trường hợp nào CỐ Ý không
      hỗ trợ; nếu có `proposal.md` thì khớp với mục Non-goals ở đó.
- [ ] **Trace được** — mỗi ID ở mục 1 / 1b / 1c / 1d có ít nhất 1 dòng ở mục 2.

**Phát hiện cần người chốt** (giữ lại để trace khi review):

| Loại | Nội dung phát hiện | Kết luận |
|------|---------------------|-----------|
| Xung đột | <vd: AUTH-02 lock 15 phút vs yêu cầu mới "user phải login lại được trong 1 phút"> | <giữ nguyên AUTH-02 / sửa thành ...> |
