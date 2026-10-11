# Prompt dùng với agent – Dự án ChiNhanh

Các prompt dưới đây dùng cùng `docs/luat-agent.md`. Copy nguyên khối vào agent.

## Chuẩn bị một lần (bạn tự làm)

1. Tạo repo trên GitHub (nên để private), ví dụ `chinhanh-android`.
2. Clone về máy, ví dụ `C:\dev\chinhanh-android`.
3. Copy toàn bộ thư mục `C:\dev\docs\ChiNhanh` vào repo thành thư mục `docs/`. Từ lúc này `docs/` trong repo là bản chính của tài liệu.
4. Mở agent ngay trong thư mục repo rồi dán prompt "Lần đầu" bên dưới.

## Lần đầu: khởi động dự án

```text
Đây là repo GitHub của dự án ChiNhanh, app Android ghi chi tiêu bằng câu tiếng Việt. Toàn bộ tài liệu nằm trong docs/. Repo hiện mới có tài liệu, chưa có code.

Hãy làm lần lượt:
1. Đọc docs/luat-agent.md và làm đúng vai trò trong đó: bạn là mentor, không viết code hộ. Bạn hướng dẫn tôi tự code từng task, cung cấp lý thuyết trước khi làm, rà soát khi tôi báo xong, và chỉ chuyển task khi mọi thứ đã ổn.
2. Kiểm tra cài đặt theo mục 12 của luật. Nếu gốc repo chưa có CLAUDE.md (chứa dòng @docs/luat-agent.md) hoặc AGENTS.md thì tạo hai file đó. Không tạo file nào khác.
3. Đồng bộ và rà tiến độ theo source code đúng như mục 4 của luật. Ghi URL repo GitHub vào docs/tien-do.md (hỏi tôi nếu không lấy được từ git remote).
4. Báo kết quả, đưa lệnh để tôi commit các file vừa tạo, rồi bắt đầu Bước 1 của task hiện tại (dự kiến là S0-01). Lưu ý S0-01 sẽ tạo project Android ngay ở gốc repo này.
```

## Mỗi lần sau (trên bất kỳ máy nào)

Claude Code tự nạp luật qua `CLAUDE.md`, nên chỉ cần:

```text
Tiếp tục dự án ChiNhanh. Hãy đồng bộ và rà tiến độ theo source code hiện tại (mục 4 của docs/luat-agent.md), đối chiếu với docs/tien-do.md, rồi báo task hiện tại và bước tiếp theo. Chưa làm gì thêm cho đến khi tôi trả lời.
```

Nếu muốn agent biết bối cảnh chuyển máy, thêm một dòng:

```text
Lần trước tôi làm trên <tên máy khác>, đã commit [wip] và push. Hôm nay tôi làm trên <tên máy này>.
```

Với công cụ không tự đọc `CLAUDE.md` hay `AGENTS.md`, thêm câu này ở đầu prompt: `Đọc và tuân theo docs/luat-agent.md trước khi làm bất cứ việc gì.`

## Trong lúc làm

Dùng các câu lệnh ngắn ở mục 10 của luật: `bắt đầu`, `lý thuyết`, `bước tiếp`, `gợi ý`, `xong task`, `hoãn <mục>`, `tạm dừng`, `đồng bộ`, `ngoại lệ: <phần>`.

## Cuối phiên

```text
tạm dừng
```

Agent sẽ cập nhật `docs/tien-do.md`, tóm tắt chỗ đang dừng và đưa lệnh commit `[wip]`. Bạn chạy lệnh đó rồi `git push` để máy khác làm tiếp được.
