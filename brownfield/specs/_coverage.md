# Độ phủ spec — dự án brownfield

> File này CHỈ có ở bản brownfield. Nó trả lời: phần nào của hệ thống đã có
> spec, phần nào chưa, và phần nào **cố ý** không có.
>
> Vì sao cần: `specs/` ở dự án đang chạy luôn lỗ chỗ (xem CLAUDE.md mục 4).
> Nếu không ghi lại thì "chưa ai kịp viết" và "đã quyết định không viết"
> trông y như nhau — người mới join hoặc AI agent không phân biệt được.

- **Ngày cập nhật gần nhất**: <YYYY-MM-DD>

## 1. Độ phủ theo module

> Mỗi module có ĐÚNG 1 trạng thái. Trạng thái `Cố ý không spec` BẮT BUỘC
> kèm lý do — không được để trống.

| Module | Trạng thái | Ghi chú / lý do |
|---|---|---|
| auth | Một phần | Đã spec: login, lock account, RBAC (`AUTH-01..04`). Chưa: SSO, đổi mật khẩu |
| inventory | Chưa spec | Chưa có ticket nào chạm tới kể từ <YYYY-MM-DD> |
| billing | Cố ý không spec | Sắp thay bằng SaaS ngoài trong <Qx/YYYY> — spec viết ra sẽ bỏ đi |
| legacy-report | Cố ý không spec | Read-only, không còn yêu cầu mới. Nếu buộc phải sửa thì spec TRƯỚC (xem mục 2) |

## 2. Vùng "không ai chạm" — phân biệt 2 loại

> Code không bị chạm có 2 lý do rất khác nhau, mà từ ngoài nhìn giống nhau
> (đều 0 commit). Phải phân biệt, vì loại thứ hai là chỗ rủi ro cao nhất
> của hệ thống.

| Vùng | Vì sao không bị chạm | Cần spec chủ động? |
|---|---|---|
| <vd: module report> | **Ổn định thật** — đúng, đủ, không có yêu cầu mới | Không |
| <vd: engine tính giá> | **Không ai dám** — không ai hiểu, chạm là vỡ | **Có — ưu tiên cao** |

Dấu hiệu nhận biết loại "không ai dám": từng có bug ở đó; có người từng nói
"đừng sửa file đó"; không có test; tác giả đã rời dự án.

Với loại này KHÔNG chờ spec-on-touch — mở ticket riêng để đọc code rồi viết
spec. Vì đến lúc buộc phải chạm thì thường là lúc khẩn cấp, không còn thời
gian viết spec.

## 3. Vùng BẮT BUỘC spec dù không ai chạm

> Ở đây spec không phục vụ việc code, mà phục vụ **trả lời câu hỏi từ bên
> ngoài** (khách Nhật, auditor). Không đợi bị chạm mới viết.

- [ ] Xử lý PII / dữ liệu cá nhân (APPI)
- [ ] Retention và xoá log
- [ ] Tính tiền / thuế / hoá đơn
- [ ] Phân quyền và audit trail
- [ ] <ràng buộc hợp đồng khác>

## 4. Lịch sử

| Ngày | Ticket ID | Thay đổi độ phủ |
|---|---|---|
| YYYY-MM-DD | TICKET-0 | Khởi tạo bảng độ phủ |
