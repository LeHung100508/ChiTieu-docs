# ChiNhanh – tài liệu dự án

Thư mục này là bản chính của mọi tài liệu dự án ChiNhanh. Nó là repo `ChiTieu-docs`, được gắn vào repo Android (`ChiTieu`) và repo iOS (`ChiTieu-ios`) dưới dạng git submodule tại `docs/` (xem mục 13 của `luat-agent.md`). Mọi chỉnh sửa (đặc tả, wireframe, mockup) phải sửa trực tiếp trên các file ở đây, giữ cho link giữa các file luôn đúng.

- `dac-ta-android.html`: đặc tả yêu cầu phần mềm bản Android (36 FR, ảnh sơ đồ trong `images/`)
- `wireframe.html`: wireframe 11 màn hình, widget và bảng khác biệt Android/iOS
- `dac-ta-ios.html`: đặc tả bản iOS, chỉ ghi phần khác bản Android
- `ke-hoach-android.html`: kế hoạch phát triển Android, 9 sprint 2 tuần
- `ke-hoach-ios.html`: kế hoạch iOS gồm sprint làm quen iL và 9 sprint iS0–iS8, luân phiên với Android theo sprint; có bản đồ kiến thức iOS
- `task-android.html`: 107 task chia theo sprint, kèm phân tích đường găng và quyết định kỹ thuật
- `task-ios.html`: task iOS theo sprint; hiện mới chia sprint iL (12 task), iS0–iS8 chia trước khi bắt đầu iS0
- `luat-agent.md`: luật cho agent AI hướng dẫn người dùng tự code từng task (mentor, không viết code hộ); mục 13 dành cho bản iOS
- `tien-do.md`: trạng thái từng task Android, nợ kỹ thuật, ngoại lệ, nhật ký sprint; agent cập nhật theo `luat-agent.md`
- `tien-do-ios.md`: tiến độ bản iOS, cùng cấu trúc với `tien-do.md` (tạo ở task iL-01)
- `hoc/`: tài liệu lý thuyết agent soạn cho từng task, cả Android và iOS (mã task iOS bắt đầu bằng `i`)
- `shared/parser/`: bộ câu mẫu và từ điển từ khóa dùng chung; test của cả hai bản đọc trực tiếp từ `docs/shared/parser/`
- `prompt-agent.md`: prompt khởi động lần đầu, prompt cho các phiên sau, và cách chuẩn bị repo GitHub

Các đường dẫn trong `luat-agent.md` tính từ gốc repo ứng dụng đang mở. Cho tới khi tách thành repo riêng, thư mục này vẫn là `docs/` thường trong repo Android.
- `mockup/*.html`: 12 màn hình mockup; `mockup/_source/` là file `.dc.html` gốc của canvas, sửa cả hai nếu cần đồng bộ lên canvas
- Bản trực tuyến trên claude.ai không còn là bản chính; chỉ cập nhật khi được yêu cầu.
