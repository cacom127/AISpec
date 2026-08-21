# Spec Template — bản BROWNFIELD (dự án đang chạy)

Dùng bản này khi dự án **đã có code chạy nhưng chưa có spec** — cùng lắm là
có vài tài liệu requirement/設計書 cũ. Nếu dự án làm mới từ đầu, dùng bản ở
root repo thay vì bản này.

## Khác bản greenfield ở đâu

| | Greenfield (root) | Brownfield (here) |
|---|---|---|
| `specs/` lúc bắt đầu | Trống, rồi ticket đầu tiên (Large) viết toàn bộ | Trống, và **để trống có chủ đích** — đầy dần theo vùng code bị chạm |
| Định nghĩa "xong" | `specs/` phản ánh đủ hệ thống | **Mọi thay đổi đều có spec** — KHÔNG phải "spec phủ hết code" |
| Khi thiếu spec thì tin ai | Không có tình huống này | **Code** là sự thật quan sát được |
| Luật `DESIGN.md` / lint | Áp toàn repo | Chỉ áp cho màn hình/file **được sửa trong ticket** |
| File thêm | — | `specs/_coverage.md` + ô "Độ phủ" ở mỗi module spec |

## File dùng chung với root — đừng copy trùng

Bản brownfield chỉ override những file THỰC SỰ khác. Khi áp vào dự án thật:

| Lấy từ | File |
|---|---|
| `brownfield/` | `CLAUDE.md`, `specs/_coverage.md`, `specs/example-module-auth.md` |
| Root repo | `DESIGN.md`, toàn bộ `changes/_template/`, và `specs/` còn lại (`vision.md`, `architecture.md`, `data-model.md`, `cross-cutting/`, `example-module-auth-ui.md`) |

`changes/_template/` dùng chung nguyên vẹn — mục 0/1c/1d/4 hoạt động y như
nhau. Riêng mục **1d (regression guard)** còn quan trọng hơn ở brownfield,
vì phần lớn hành vi xung quanh chưa có spec để mà dựa vào.

Lý do tách kiểu này: luật chung chỉ sửa 1 lần, 2 bản không drift. Chạy
`diff CLAUDE.md brownfield/CLAUDE.md` sẽ thấy đúng phần khác nhau.

## Cách vào — TICKET-0

**Đừng reverse-engineer toàn bộ `specs/` trước khi bắt đầu.** Vài tuần
công, không ai review nổi, lỗi thời ngay khi viết xong, và không tạo ra giá
trị nào ở thời điểm viết. Đây là cách phổ biến nhất làm chết một đợt
adoption.

Chỉ làm những thứ RẺ mà có giá trị ngay:

| Làm ở TICKET-0 | Vì sao rẻ |
|---|---|
| `CLAUDE.md` điền thật (stack, ràng buộc, người chốt) | Viết tay 1 lần, không cần đọc code |
| `architecture.md` — bảng module + `ARCH-xx` ràng buộc hạ tầng | Suy ra từ repo/infra, không cần đọc business logic |
| `data-model.md` mục 1 (ER) + mục 2 (entity → module) | Suy ra từ schema/migration |
| `specs/_coverage.md` — điền trạng thái ban đầu | Là quyết định, không phải khảo cứu |
| `DESIGN.md` | Nhờ AI đọc screenshot UI hiện tại soạn nháp |
| **KHÔNG** làm `specs/<module>.md` | Phần đắt nhất — để spec-on-touch lo |

TICKET-0 nên là size **Small/Medium**, không phải Large như ticket genesis
của bản greenfield.

## Spec-on-touch — luật vận hành chính

> Ticket nào chạm vùng code chưa có spec thì `delta-spec.md` của ticket đó
> phải ghi nhận luôn hành vi HIỆN CÓ của phần bị chạm, rồi mới ghi thay đổi
> lên trên.

Ghi nhận dùng dạng `(MỚI — ghi nhận hành vi có sẵn)` ở mục 1. Đây không
phải thêm tính năng, mà là chép cái đã tồn tại vào văn bản.

**Chỉ ghi nhận trong bán kính ảnh hưởng, không phải cả module** — thường
2–5 dòng, cùng nguyên tắc với mục 1d.

Ví dụ, ticket đổi thời gian lock account ở module auth chưa có spec:

```
## 1. Yêu cầu thay đổi

- **[AUTH-02] (MỚI — ghi nhận hành vi có sẵn)** When a user fails login 5
  times within 1 minute, the system shall lock the account for 15 minutes.
- **[AUTH-02] (SỬA)**
  - Cũ: lock 15 phút → Mới: lock 30 phút (yêu cầu security audit).

## 1d. Hành vi KHÔNG được thay đổi (regression guard)

- **[AUTH-01] (GIỮ NGUYÊN)** When a user submits valid credentials, the
  system shall CONTINUE TO issue a token valid for 8 hours — AUTH-01 CHƯA
  có trong `specs/`, hành vi này quan sát được trong code (`AuthService`),
  và ticket này không được làm vỡ.
```

Lưu ý dòng cuối: ở brownfield, mục 1d được phép tham chiếu hành vi **chưa
có ID trong `specs/`** — lúc đó ghi rõ nguồn là code.

## Vậy `specs/` mãi không đầy đủ — có sao không?

Không sao, và đó là chủ đích. Giá trị của một trang spec đo bằng **số lần
nó được đọc để ra quyết định**. Code không ai chạm thì không ai cần quyết
định gì về nó → spec cho nó giá trị gần 0, nhưng vẫn tốn chi phí viết, chi
phí bảo trì, và rủi ro lệch pha âm thầm. Spec không ai đọc thì không ai
phát hiện nó đã sai so với code — lúc đó nó thành nguồn thông tin SAI, tệ
hơn là không có gì.

Sau 6–12 tháng, `specs/` sẽ đầy theo đúng vùng code hay đổi nhất — cũng
chính là vùng spec có giá trị nhất.

Ba ngoại lệ (vùng không ai chạm mà VẪN phải spec: ràng buộc pháp lý/audit,
vùng "không ai dám chạm", và cách phân biệt nó với vùng "ổn định thật") —
xem `specs/_coverage.md` mục 2 và 3.

## Tài liệu requirement / 設計書 có sẵn

Đừng migrate nguyên khối vào `specs/`. Tài liệu của dự án đang chạy gần như
luôn lỗi thời — đổ vào `specs/` là đầu độc nguồn chân lý ngay từ đầu.

- Dùng làm **nguyên liệu** khi viết spec module lần đầu (lúc spec-on-touch)
- Để link ở mục "Tài liệu gốc" của module spec, không copy nội dung
- Khi code và tài liệu cũ nói khác nhau: **code là sự thật quan sát được,
  tài liệu cũ là ý định**. Mâu thuẫn phải do người chốt quyết (`CLAUDE.md`
  mục 9) và ghi lại kết luận — đúng việc mà mục 4 của `delta-spec.md`
  (loại lỗi "xung đột") dùng để làm.

Với khách Nhật, chốt sớm một điều: 設計書 giao khách là **sản phẩm dẫn
xuất**, `specs/` là chân lý kỹ thuật nội bộ. Nếu để cả hai cùng mang tính
hợp đồng thì chắc chắn phân kỳ.

## Rủi ro thật và cách chặn

| Rủi ro | Cách chặn |
|---|---|
| Team không thực sự ghi nhận hành vi khi chạm lần đầu → `specs/` mãi trống, cả bộ này thành thủ tục hình thức | Gate ở PR (checklist hoặc hook) — không trông vào tự giác |
| AI agent đọc `specs/` lỗ chỗ rồi tưởng đó là toàn bộ, xoá hành vi "dư thừa" | Ô "Độ phủ" bắt buộc + `CLAUDE.md` mục 8 (cấm xoá hành vi chỉ vì nó không có trong spec) |
| Ticket toàn bug nhỏ rải rác nhiều module → overhead cảm giác nặng | Chỉ ghi nhận trong bán kính ảnh hưởng, 2–5 dòng |
| Có người kết luận "template này không dùng được" vì `specs/` lỗ chỗ | Truyền đạt trước: 6–12 tháng đầu lỗ chỗ là ĐÚNG thiết kế, không phải thất bại |
