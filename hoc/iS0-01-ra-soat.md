# Rà soát iS0-01 · Tạo project

Ngày rà soát: 2026-11-14 · Nhánh `task/iS0-01-tao-project` · Trạng thái: **Xong** (đóng sau rà lại lần 1)

> Bản rà soát tự viết trước đó (ghi "Xong") đã được thay bằng bản này. Theo luật, mentor rà soát và đổi trạng thái.

## Lệnh đã chạy
- `xcodebuild -project ChiTieu.xcodeproj -scheme ChiTieu -destination 'platform=iOS Simulator,name=iPhone 17' build` → **BUILD SUCCEEDED**.
  - Chỉ có một cảnh báo, đến từ công cụ AppIntents ("Metadata extraction skipped"), không đến từ code.
- `xcodebuild -list` → có một target `ChiTieu` và scheme `ChiTieu`. Scheme đã nằm trong `xcshareddata/`.
- Phần mentor chưa kiểm chứng được: app chạy trên iPhone thật. Ngoài ra, toàn bộ project chưa được commit (`ChiTieu/` và `ChiTieu.xcodeproj/` đang ở trạng thái untracked).

## Bắt buộc sửa (1)

### 1. Xác nhận tiêu chí "chạy trên iPhone thật"
- **Vì sao:** đây là một phần của tiêu chí xong. Mentor không chạy được trên thiết bị của bạn.
- **Hướng sửa:**
  - Chạy app trên iPhone thật.
  - Báo lại kết quả. Nếu signing tự động với Bundle ID mới có lỗi gì, ghi lại.

## Nên sửa (1)

### 1. Deployment target ở hai cấp đang lệch nhau
- **Vị trí:** `ChiTieu.xcodeproj/project.xcproj:116` là cấp project, đang để `27.2`. Còn `project.xcproj:46` là cấp target, đang để `26.0`.
- **Vấn đề:** app hiện build cho iOS 26.0 vì giá trị ở target ghi đè giá trị ở project. Nhưng từ iS0-02 trở đi, mọi target mới đều thừa kế giá trị `27.2` của project. Đó có thể là target test, và về sau là widget hay extension. Khi đó các target này sẽ chỉ chạy được trên iOS 27.2, trái với quyết định "target iOS 26.0" (luật 13.5, NFR-13). Lỗi loại này thường chỉ lộ ra khi cài lên máy chạy iOS 26.
- **Hướng sửa:**
  - Đặt `26.0` ở cấp project.
  - Bỏ phần ghi đè ở cấp target, để cả project chỉ còn một nơi khai báo.
  - Sau đó tự kiểm tra bằng `xcodebuild -showBuildSettings`.

## Gợi ý thêm để học (2)
1. **Ngôn ngữ phát triển.**
   - Hiện tại: `"development": "en"`, và `supported` chỉ có `Base`.
   - NFR-12 yêu cầu tiếng Việt và tiếng Anh, làm bằng String Catalog ở iS1.
   - Nên nghĩ trước: ngôn ngữ gốc của chuỗi là tiếng Việt hay tiếng Anh? Chọn cho giống quy ước bên Android. Lựa chọn này quyết định key và bản dịch nằm ở đâu trong String Catalog.
2. **Code mẫu của template.** `ContentView` vẫn là "Hello, world!" với icon `globe`. Để vậy cho app trống thì được, nhưng nên dọn đi khi lắp màn hình đầu tiên. Dòng comment header do Xcode tự sinh ("Created by…") có thể bỏ khi cấu hình SwiftLint/swift-format ở iS0-03.

## Điểm làm tốt
- Thiết lập đúng ngay từ đầu:
  - Swift 6 (`SWIFT_VERSION = 6.0`);
  - `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`;
  - `SWIFT_APPROACHABLE_CONCURRENCY`;
  - iPhone và iPad (`TARGETED_DEVICE_FAMILY = 1,2`), đúng NFR-13.
- Dùng thư mục đồng bộ (`"kind": "folder"`). Kết hợp với định dạng project mới, thêm file sẽ không làm sinh ra ID mới trong project, nên ít xung đột merge khi làm trên nhiều máy.
- Đã có shared scheme và `.gitignore` bỏ qua `xcuserdata/` và `.DS_Store`. Bundle ID và App Group ID đã được chốt và ghi vào tiến độ. Chưa bật capability App Group, đúng như tiêu chí.

## Kết luận
Chưa đóng được task. Cần:
- xác nhận đã chạy trên iPhone thật;
- sửa deployment target (hoặc ghi `hoãn <mục>`);
- báo lại để mentor rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-14)
`xcodebuild … build` → **BUILD SUCCEEDED**, code không có cảnh báo. `-showBuildSettings` cho `IPHONEOS_DEPLOYMENT_TARGET = 26.0`.
- Bắt buộc 1: người dùng xác nhận app đã build và chạy được trên iPhone thật.
- Nên sửa 1: cấp project đã đổi thành `26.0`.
- Gợi ý, không chặn: cấp target vẫn còn ghi đè `26.0` (`project.xcproj:46`). Hiện hai giá trị trùng nhau nên không sao. Nếu xóa dòng ghi đè này thì sau này chỉ cần sửa ở một chỗ.

**Kết luận:** Đóng được task.
