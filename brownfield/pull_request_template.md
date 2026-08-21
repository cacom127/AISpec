<!--
MẪU PR TEMPLATE — GATE MỨC 1 cho luật spec-on-touch.

Cách dùng: copy file này vào dự án thật thành
  - GitHub : .github/pull_request_template.md
  - GitLab : .gitlab/merge_request_templates/default.md
rồi xoá khối comment này đi.

Đây là gate RẺ NHẤT (~10 phút) và YẾU NHẤT — vẫn tick bừa được. Mục đích của
nó là biến "quên" thành "cố tình" và để lại dấu vết. Khi dự án đã có bảng map
path -> module thì thay bằng gate mức 3 (CI check) — xem brownfield/README.md
mục "Dựng gate cho spec-on-touch".
-->

## Ticket

- **Ticket ID**: <TICKET-123>
- **Thư mục change**: `changes/<ticket-id>/`

## Spec

- [ ] `delta-spec.md` tồn tại (BẮT BUỘC mọi size — kể cả refactor thuần)
- [ ] Ticket này có chạm code **chưa có spec** không? Chọn 1:

      - [ ] Không — module bị chạm đã có spec đầy đủ
      - [ ] Có → đã ghi nhận hành vi hiện có vào `delta-spec.md` mục 1 dạng
            `(MỚI — ghi nhận hành vi có sẵn)`, chỉ trong bán kính ảnh hưởng
      - [ ] Có, nhưng CỐ Ý không ghi nhận → lý do: <bắt buộc ghi rõ>
- [ ] Đã cập nhật `specs/_coverage.md` nếu độ phủ của module thay đổi
- [ ] Mục **1d (regression guard)** đã điền — nếu ticket có (SỬA)/(XOÁ), là
      fix bug, hoặc là refactor thuần
- [ ] Mục **4 (soát chất lượng)** đã chạy, gồm soát cả requirement CŨ cùng
      module (không chỉ dòng vừa thêm)

## Test

- [ ] Mọi ID ở mục 1 / 1b / 1c / 1d có ít nhất 1 test case ở mục 2
- [ ] Có regression test cho mọi dòng `shall CONTINUE TO` ở mục 1d
- [ ] Nếu là fix bug: có test tái hiện bug, FAIL trước khi fix

## Reviewer

- [ ] Đã mở `specs/<module>.md` và `specs/_coverage.md` để đối chiếu —
      KHÔNG chỉ đọc diff của PR
