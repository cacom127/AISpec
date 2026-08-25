# Tasks — <Tên thay đổi ngắn>

> File BẮT BUỘC cho mọi thay đổi. Chia nhỏ việc thành các task có thể làm
> và test độc lập. AI agent implement theo đúng thứ tự/nội dung ở đây.

- **Ticket ID**: <TICKET-123>
- **Dựa trên**: `delta-spec.md` (+ `plan.md` nếu có)

## Checklist

- [ ] **T1** — <Mô tả task, vd: "Thêm endpoint POST /auth/reset-password">
      - Liên quan: AUTH-05
      - File dự kiến: `src/auth/reset.ts`
- [ ] **T2** — <vd: "Sửa logic lock account trong AuthService">
      - Liên quan: AUTH-02
      - File dự kiến: `src/auth/service.ts`
- [ ] **T3** — <vd: "Viết test case cho T1, T2">
      - Liên quan: TC-AUTH-15, TC-AUTH-03
      - Gồm cả regression test cho MỌI dòng ở `delta-spec.md` mục 1d
      - Nếu là fix bug: viết test TÁI HIỆN bug (phải FAIL trước khi fix)
        trước khi sửa code
- [ ] **T4** — Review chéo + cập nhật `specs/<module>.md` khi merge
      - Nếu ticket thêm/xoá endpoint: thêm 1 dòng vào `specs/api-catalog.md`
- [ ] **T5** — Verify trước khi archive: tất cả checkbox trên đã tick, và
      mọi mục trong `delta-spec.md` đã fold vào đúng file `specs/` — TRỪ
      mục 1d (regression guard) không fold (xem CLAUDE.md mục 4) — chưa
      đủ 2 điều kiện thì KHÔNG archive
- [ ] **T6** — Di chuyển thư mục này vào `changes/_archive/` sau khi merge

## Trạng thái

| Trạng thái  | Ngày cập nhật | Ghi chú |
|-------------|----------------|----------|
| Đang làm    | YYYY-MM-DD     |          |
