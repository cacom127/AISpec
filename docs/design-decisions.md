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