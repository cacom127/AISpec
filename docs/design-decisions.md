# Design Decisions — Lý do đằng sau cấu trúc spec template này

> File này đúc kết lại QUÁ TRÌNH LẬP LUẬN dẫn đến cấu trúc hiện tại của
> repo (không chỉ kết luận). Dùng khi cần hiểu "tại sao lại làm thế này"
> — vd khi Claude Code hỏi lại, hoặc khi có người mới join dự án.
> Không phải spec — không fold vào `specs/`, không đi qua `changes/`.

## 1. Vì sao dùng mô hình lai Spec Kit + OpenSpec

- **GitHub Spec Kit**: workflow tuyến tính Specify → Plan → Tasks →
  Implement, có `constitution.md` là nguyên tắc bất biến. Mạnh ở việc
  tách rõ business intent (spec) và quyết định kỹ thuật (plan), nhưng
  yếu ở việc mỗi lần sửa phải viết lại spec khá nhiều.
- **OpenSpec**: tách `specs/` (current truth, đã chốt) và `changes/`
  (delta, đang làm) — như Git (main vs feature branch) áp cho tài liệu.
  Mạnh ở việc chỉ cần viết PHẦN THAY ĐỔI, không viết lại toàn bộ. Yếu ở
  việc thiếu tầng "nguyên tắc toàn dự án" như Spec Kit.
- **Kết hợp**: lấy `CLAUDE.md` (constitution) + tách business/technical
  (proposal/plan) từ Spec Kit; lấy cơ chế delta + `specs/` vs `changes/`
  từ OpenSpec. Điều này giải quyết đúng 2 khuyết điểm của nhau.
- **Bổ sung sau (Kiro — AWS)**: mục 11–14 dưới đây lấy từ Kiro, chủ yếu là
  3 thứ OpenSpec/Spec Kit không có: phân hoá spec theo LOẠI công việc
  (feature / bug / refactor), khai báo hành vi phải giữ nguyên (regression
  guard), và một bước soát chất lượng requirement có checklist cụ thể.
  Ngược lại, Kiro KHÔNG tách `specs/` vs `changes/` và không có cơ chế nào
  chống spec drift sau khi code (issue #2768 của họ bị đóng "not planned",
  #9435 vẫn mở) — nên phần lõi vẫn giữ theo OpenSpec.

## 2. Nguyên tắc lõi: KHÔNG CÓ EXCEPTION trong flow `changes/` → `specs/`

- Ban đầu có ý tưởng làm 1 luồng riêng cho "lúc khởi tạo dự án" (vì
  `specs/` còn trống, tưởng như không có gì để "diff"). Đã BỎ ý tưởng
  này — quyết định cuối: coi baseline rỗng = mọi nội dung genesis đều là
  `(MỚI)` trong `delta-spec.md`, đúng loại đã có sẵn. Không cần cấu trúc
  riêng, không cần xử lý ngoại lệ.
- Lý do chọn "không exception" dù phải viết dài hơn cho ticket đầu tiên:
  giảm số lượng luật phải nhớ/dạy AI agent, tránh rủi ro AI/dev quên áp
  dụng đúng exception khi gặp trường hợp biên.
- Rule cụ thể: khi merge, `delta-spec.md` fold vào `specs/`, **tạo file
  mới nếu chưa tồn tại** — 1 dòng duy nhất đủ để cover mọi trường hợp,
  kể cả ticket đầu tiên.

## 3. Nguyên tắc phân lớp: tách theo TỐC ĐỘ THAY ĐỔI, không theo "cùng chủ đề"

Đây là nguyên tắc dùng lặp lại nhiều lần để quyết định 1 nội dung nên đặt
ở đâu — không phải theo "cùng nhắc tới UI" hay "cùng nhắc tới data" mà
theo tốc độ đổi + đối tượng review:

| Vấn đề gặp phải | Cách giải quyết |
|---|---|
| `data-model.md` mô tả là "ít đổi" nhưng field nghiệp vụ đổi liên tục | Tách: `data-model.md` CHỈ giữ ER tổng quan (tên bảng + quan hệ) + quy ước chung, ít đổi thật. Field-level chi tiết dời xuống `specs/<module>.md` — đúng module sở hữu, đổi cùng nhịp business logic. |
| Error handling / logging không thuộc riêng module nào | Tạo `specs/cross-cutting/` — phân biệt với `architecture.md` bằng câu hỏi: đây là "cấu trúc" (cái gì tồn tại) hay "chính sách hành vi" (làm gì nhất quán)? |
| `DESIGN.md` (token/component atomic) khác `<module>-ui.md` (layout/behavior từng màn hình) | `DESIGN.md` = "nút trông thế nào", `<module>-ui.md` = "màn hình nào dùng nút đó, khi nào, bấm vào thì làm gì". 2 file bổ sung, không thay thế nhau. |
| API endpoint, integrations, permissions... đều có pattern tương tự | Tổng quát hoá thành: **Catalog/Index** (1 dòng/mục, trỏ module sở hữu) + **Cross-cutting convention** (chính sách chung) + **Module detail** (chi tiết ở module sở hữu). |

**Câu hỏi thử dùng để tự quyết định vị trí 1 nội dung mới:**
> "Nếu liệt kê tất cả các X trong hệ thống ra 1 bảng, mỗi dòng có thuộc
> về đúng 1 module không? Có cần 1 rule chung áp dụng cho mọi X không?"
> Nếu cả 2 "có" → áp pattern 3 lớp (catalog / cross-cutting / detail).

## 4. Quy tắc ownership: "1 entity/endpoint = 1 module sở hữu"

- Mỗi entity hoặc API endpoint chỉ có ĐÚNG 1 module sở hữu — field/schema
  chi tiết đặt tại module đó.
- Module khác nếu chỉ THAM CHIẾU (FK, hoặc gọi API) tới entity/endpoint
  đó thì KHÔNG lặp lại chi tiết, chỉ ghi chú tham chiếu ngược lại module
  sở hữu.
- Mục đích: 1 ticket sửa module nào chỉ đụng vào file của module đó, giữ
  tính cô lập, tránh conflict giữa các ticket không liên quan.

## 5. Thứ tự viết file trong `changes/<ticket-id>/` — vì sao không thể đảo

```
proposal.md (Why) → plan.md (giải pháp kỹ thuật) → delta-spec.md
  (cam kết CHỐT, nằm trong giới hạn đã chọn ở plan) → tasks.md (breakdown)
```

- Mỗi file **thu hẹp dần "không gian có thể sai"** — proposal rộng nhất
  (chưa biết cách làm), plan thu hẹp theo giới hạn kỹ thuật thật,
  delta-spec chốt cam kết NẰM TRONG giới hạn đó, tasks chỉ còn chia việc.
- Nếu viết `delta-spec.md` TRƯỚC `plan.md`: rủi ro cam kết những con số
  (vd "OTP tự xoá ngay khi hết hạn") mà giải pháp kỹ thuật thực tế không
  làm được (vd không có Redis, chỉ có cron dọn mỗi 15 phút) → phải sửa
  lại delta-spec đã duyệt, phá vỡ tác dụng của checkpoint.
- Mỗi bước nên có checkpoint (người xác nhận) trước khi qua bước sau —
  đúng người duyệt đúng phần (PM duyệt proposal, Tech lead duyệt plan).

## 6. Vì sao KHÔNG dùng ảnh/Figma làm nguồn chân lý cho UI spec

- Ảnh chỉ thể hiện được 1 trạng thái tĩnh — không thể hiện hành vi (khi
  bấm nút thì gọi API nào, lỗi thì hiện gì, khi nào disable...).
- Ảnh không "diff" được — phá vỡ cơ chế `delta-spec` chỉ ghi phần thay
  đổi; reviewer không so sánh được ảnh cũ/mới bằng text diff.
- Ảnh không tra cứu được theo ID (`UI-AUTH-05-1`) như tra text — AI agent
  không map được ảnh vào acceptance criteria/test case.
- Quyết định: ảnh/Figma chỉ là TÀI LIỆU THAM KHẢO (mục "Tham chiếu thiết
  kế" ở cuối file), nguồn chân lý luôn là text (EARS + state matrix).

## 7. Vì sao EARS notation viết được đa ngôn ngữ, không cần tiếng Anh

- EARS là 1 PATTERN cấu trúc câu (trigger → chủ thể → hành vi bắt buộc),
  không phải cú pháp riêng của tiếng Anh — áp dụng được cho tiếng Việt,
  tiếng Nhật với cùng độ chính xác.
- Với team song ngữ VI/JP, có thể viết 2 ngôn ngữ trong cùng 1 dòng để
  gửi khách hàng Nhật xác nhận trực tiếp trên spec.

## 8. Vì sao test case KHÔNG nằm trong spec

- Spec (behavior) và test case (bước test cụ thể, dữ liệu test) là 2
  artifact khác định dạng, khác người dùng, khác tốc độ đổi — gộp chung
  gây duplicate, dễ lệch pha khi sửa 1 chỗ quên sửa chỗ kia.
- `delta-spec.md` chỉ giữ BẢNG MAPPING (ID yêu cầu ↔ tên test case), nội
  dung test case thật nằm ở nơi khác (file Excel giao khách hàng, test
  code, test management tool).

## 9. Vì sao tách `CLAUDE.md` (root) và có thể có thêm `src/CLAUDE.md`

- Claude Code hỗ trợ CLAUDE.md nhiều cấp: file ở thư mục cha nạp ngay
  lúc khởi động (luôn có), file ở thư mục con chỉ nạp khi Claude thực sự
  làm việc trong thư mục đó — cơ chế "load-khi-cần" giảm nhiễu context.
- Root `CLAUDE.md`: nguyên tắc toàn dự án (business + kỹ thuật cấp cao +
  quy ước tổ chức spec) — luôn cần, ít đổi.
- `src/CLAUDE.md` (tuỳ chọn, chưa tạo trong template): convention code
  chi tiết (naming, pattern, test convention) — chỉ cần khi AI đụng code.
- `README.md` khác `CLAUDE.md` về bản chất: README dành cho NGƯỜI (không
  tự nạp làm AI context), CLAUDE.md dành cho AI + người.

## 10. Vì sao DESIGN.md không tự sinh được bằng lệnh cài đặt

- CLI `@google/design.md` chỉ có lệnh `lint`/`diff`/`export`/`spec` —
  không có `init`/`generate`. Phải tự viết tay, nhờ AI phân tích ảnh
  mockup để soạn draft, hoặc dùng Google Stitch (nơi format này bắt
  nguồn) để generate từ mô tả brand hoặc ảnh chụp UI hiện có.
- Format gồm YAML front matter (token — máy đọc) + markdown body theo
  8 section thứ tự chuẩn (Overview → Colors → Typography → Layout →
  Elevation → Shapes → Components → Do's and Don'ts).
## 11. Vì sao phân loại thay đổi theo LOẠI, không chỉ theo SIZE

Bảng Change Sizing (`CLAUDE.md` mục 6) phân theo ĐỘ LỚN (Small/Medium/
Large) — trả lời "cần bao nhiêu file". Nó KHÔNG trả lời "nội dung
`delta-spec.md` phải có hình dạng gì", trong khi 3 loại việc dưới đây có
thể cùng size mà cần nội dung khác nhau hẳn:

| Loại việc | Vấn đề nếu dùng chung 1 khuôn |
|---|---|
| Feature mới | (khuôn gốc — không có vấn đề) |
| Refactor thuần | Không có mục MỚI/SỬA/XOÁ nào để ghi → dev/AI kết luận "khỏi cần delta-spec", mất luôn dấu vết thay đổi |
| Fix bug | Bug analysis (đang sai gì, vì sao) KHÔNG phải requirement; nhồi vào mục 1 làm lẫn giữa "mô tả thực tế" và "cam kết" |

- **Refactor thuần**: giải bằng cách vẫn BẮT `delta-spec.md` phải tồn tại,
  nhưng cho phép mục 1 chỉ ghi `**Không đổi spec** — lý do: ...`. Chọn
  cách này thay vì miễn hẳn file, vì miễn hẳn sẽ sinh ra 1 exception —
  đúng thứ mục 2 ở trên đã quyết định tránh.
- **Fix bug**: giải bằng mục 0 riêng (hành vi hiện tại / mong đợi /
  nguyên nhân gốc), và buộc phân loại ĐÚNG 1 trong 2:
  - **Loại A — code ≠ spec**: spec đang đúng, code làm sai → KHÔNG sửa
    spec. Nếu không phân biệt, dev hay "sửa spec cho khớp code" — tức là
    hợp thức hoá cái bug thành yêu cầu.
  - **Loại B — spec sai/thiếu**: code làm đúng theo spec nhưng spec sai →
    PHẢI sửa spec; chỉ sửa code thì spec và code lệch nhau ngay.

Nguồn: Kiro có 4 workflow riêng (Spec / Plan / Bug Fix / Quick Spec) và
dùng `bugfix.md` thay cho `requirements.md` ở ticket bug — cùng lập luận
"một khuôn không phục vụ được các điểm khởi đầu khác nhau".

## 12. Vì sao regression guard (mục 1d) KHÔNG fold vào `specs/`

Mục 1d liệt kê hành vi mà ticket KHÔNG được làm vỡ, viết dạng
`[ID] (GIỮ NGUYÊN) ... shall CONTINUE TO ...`.

Câu hỏi khi thiết kế: lúc merge thì fold nó vào `specs/` như mục 1, hay
không? Quyết định: **KHÔNG**, vì:

- Những hành vi đó ĐÃ nằm trong `specs/` rồi, dưới ID gốc (`AUTH-01`...).
  Fold vào sẽ tạo bản sao thứ hai của cùng một yêu cầu → đúng loại lặp dữ
  liệu mà quy tắc ownership (mục 4) tồn tại để tránh.
- Nó không phải mô tả hệ thống, mà là **ràng buộc phạm vi của 1 ticket**:
  "lần này đừng chạm vào mấy chỗ này". Hết ticket là hết hiệu lực.
- Áp đúng nguyên tắc mục 3 (tách theo tốc độ thay đổi): requirement sống
  vĩnh viễn ở `specs/`; regression guard chết theo ticket, nên nằm trong
  `changes/<ticket-id>/` và được archive cùng ticket.

Hệ quả kéo theo: cổng verify trước khi archive phải ghi rõ "trừ mục 0 và
1d" — nếu không, người verify thấy 2 mục chưa fold sẽ tưởng chưa xong.

Vì sao dùng `shall CONTINUE TO` thay vì `shall`: để phân biệt 2 loại cam
kết mà người đọc / AI / tester cần tách bạch — `shall` = phải LÀM MỚI
(→ test tính năng), `shall CONTINUE TO` = phải KHÔNG LÀM VỠ (→ regression
test). Viết `shall` trần thì dev dễ tưởng đây là việc mới cần code thêm.

Nguồn: mục "Unchanged Behavior (Regression Prevention)" trong `bugfix.md`
của Kiro.

## 13. Vì sao bước soát chất lượng (mục 4) phải quét cả requirement CŨ

`CLAUDE.md` mục 7 từ đầu đã có luật "thay động từ mơ hồ bằng tiêu chí đo
được". Nhưng đó là LUẬT, không phải QUY TRÌNH — không có bước nào bắt ai
chạy nó, nên trên thực tế nó chỉ là lời khuyên.

Mục 4 biến luật đó thành checklist 5 loại lỗi phải soát: mơ hồ, xung đột,
thiếu edge case, giả định không nói ra, phạm vi cố ý loại trừ.

Điểm quan trọng nhất là **phải soát trên cả requirement CŨ cùng module**,
không chỉ dòng vừa thêm. Lý do: loại lỗi **xung đột** về bản chất không
nằm trong một dòng nào cả — nó nằm ở QUAN HỆ giữa dòng mới và dòng đã tồn
tại. Đọc riêng dòng mới thì thấy hoàn toàn hợp lý.

Ví dụ: `specs/example-module-auth.md` đã có `[AUTH-02]` lock account 15
phút. Ticket mới thêm "user phải login lại được trong vòng 1 phút". Từng
dòng đều đúng; đứng cạnh nhau thì bất khả thi. Không soát dòng cũ thì phát
hiện ra ở giai đoạn test — hoặc tệ hơn, khách Nhật phát hiện.

Vì sao AI agent phải BÁO CÁO chứ không tự sửa: quyết định giữ hay đổi một
requirement là quyết định nghiệp vụ, thuộc người chốt (`CLAUDE.md` mục 9),
không thuộc AI — đúng tinh thần mục 8 "không tự suy diễn yêu cầu". Mỗi
phát hiện phải kết thúc bằng đúng 1 trong 2 kết luận (giữ nguyên / sửa
thành ...) và ghi vào bảng ở mục 4 để trace lại khi review.

Nguồn: bước "Analyze Requirements" của Kiro, kèm cơ chế mỗi phát hiện chỉ
có 2 lựa chọn trả lời (A = giữ nguyên, B = sửa).

## 14. Vì sao có cổng verify trước khi archive

Archive là hành động MỘT CHIỀU: `changes/<ticket-id>/` rời khỏi vùng đang
làm, và `CLAUDE.md` mục 8 cấm sửa lại `changes/_archive/`. Nếu archive khi
chưa fold hết `delta-spec.md` vào `specs/`, hậu quả là `specs/` thiếu
requirement mà không còn chỗ nào nhắc rằng nó thiếu — spec âm thầm sai, và
đây đúng kiểu lỗi không ai phát hiện được bằng cách đọc code.

Nên trước khi move phải verify đủ 2 điều kiện: (1) mọi checkbox trong
`tasks.md` đã tick, (2) mọi mục CẦN fold trong `delta-spec.md` đã fold vào
đúng file `specs/` — trừ mục 0 và 1d (xem mục 12).

Chọn làm bằng checklist thủ công thay vì script/hook vì repo này là
template, không ràng buộc vào stack hay CI cụ thể nào. Khi áp vào dự án
thật thì có thể tự động hoá bằng hook.

Nguồn: `openspec validate --archived` (OpenSpec v1.9.0) — cùng mục đích
"đảm bảo mọi thứ đã archive là hoàn chỉnh".
