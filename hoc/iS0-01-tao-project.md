# iS0-01 · Tạo project Xcode cho app sản phẩm

## Vì sao cần
Khởi tạo project Xcode 27 chính thức cho app ChiTieu nằm độc lập ở gốc repo (tách khỏi `luyen-tap/`). Cấu hình đúng chuẩn Swift 6, thư mục đồng bộ và Shared Scheme ngay từ đầu giúp tránh xung đột git, bảo đảm mã nguồn an toàn luồng và sẵn sàng cho CI/máy khác cùng làm việc.

## Khái niệm chính
- **Project sản phẩm và cấu trúc repo**: App nằm ở thư mục gốc repo (ví dụ `ChiTieu/` và `ChiTieu.xcodeproj`), tách rời hoàn toàn khỏi sandbox `luyen-tap/`.
- **Synchronized Groups (Thư mục đồng bộ)**: Tính năng của Xcode giúp đồng bộ trực tiếp project navigator với hệ thống file thực tế trên đĩa. Thay vì ghi nhận từng file với ID riêng trong `.pbxproj` gây xung đột merge git triền miên, Xcode quản lý theo thư mục tương tự như Android Studio với Gradle.
- **Swift 6 Mode & Default Actor Isolation**: Bật Swift 6 language mode và đặt `Default Actor Isolation` thành `MainActor` (`SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`). Mọi type/func trong target app mặc định cô lập an toàn trên MainActor trừ khi được đánh dấu `nonisolated`.
- **Bundle ID & App Group ID**: Chốt Bundle ID (ví dụ `hung.lct.chitieu` hoặc `com.hungle.ChiTieu`) và App Group ID (`group.hung.lct.chitieu`). Chưa bật capability App Group trong Signing & Capabilities vì cần tài khoản Apple Developer Program trả phí, nhưng ID cần chốt trước để thiết kế schema và container ở iS1.
- **Shared Scheme**: Scheme mặc định được Xcode lưu trong `xcuserdata/` (bị `.gitignore` bỏ qua). Phải chuyển sang **Shared Scheme** để lưu vào thư mục `xcshareddata/xcschemes/` và commit lên git, giúp CI và máy khác build được.
- **Ôn lại từ Cổng 0**:
  - *Điều kiện để `struct` là `Sendable`*: Tất cả các stored property của struct phải có kiểu đạt `Sendable`. Nếu struct là public, nó không tự suy luận `Sendable` mà phải khai báo tường minh `: Sendable`.
  - *Optional chaining làm phẳng kiểu*: Cú pháp `a?.b?.c` luôn chỉ trả về một tầng `Optional` (kiểu `C?`), không bao giờ lồng thành `C??` dù đi qua nhiều tầng optional liên tiếp.

## So với cách cũ bạn quen
| Cũ (Android) | Mới (iOS) | Khác biệt chính |
| --- | --- | --- |
| `settings.gradle.kts` + thư mục nguồn | Xcode Project + Synchronized Group | Xcode đồng bộ thư mục đĩa vào project mà không cần khai báo từng file rời rạc |
| `applicationId` trong `build.gradle.kts` | Bundle Identifier trong Build Settings | Định danh duy nhất toàn cầu của app trên Apple ecosystem |
| Gradle build variant (`assembleDebug`) | Xcode Scheme (`Debug` / `Release`) | Phải cấu hình Shared Scheme thì lệnh dòng lệnh mới tìm thấy scheme để build |
| Coroutine Dispatchers (`Dispatchers.Main`) | Swift Concurrency + `MainActor` isolation | Compiler kiểm tra data race tĩnh tại thời điểm biên dịch thay vì runtime |

## API và cú pháp cần biết
- `xcodebuild -list -project <Path.xcodeproj>`: Liệt kê các target, build configuration và scheme chia sẻ của project.
- `xcodebuild -project <Path.xcodeproj> -scheme <Scheme> -destination '<Destination>' build`: Lệnh build dòng lệnh dùng để kiểm chứng app trên simulator.
- `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`: Cờ build setting thiết lập cách ly mặc định cho target app.

Ví dụ kiểm tra project bằng lệnh dòng lệnh:
```bash
# Kiểm tra danh sách scheme đã được chia sẻ công khai hay chưa
xcodebuild -list -project ChiTieu.xcodeproj

# Build thử target app trên simulator iPhone 17
xcodebuild -project ChiTieu.xcodeproj \
  -scheme ChiTieu \
  -destination 'platform=iOS Simulator,name=iPhone 17' \
  build
```

## Lỗi hay gặp
- **Quên đánh dấu Shared Scheme**: Scheme vẫn nằm ở User data riêng của máy bạn. Khi đẩy lên GitHub hoặc kéo sang máy khác, `xcodebuild` báo không tìm thấy scheme. Cách tránh: vào `Product` › `Scheme` › `Manage Schemes...`, tích chọn ô `Shared` cho scheme `ChiTieu`.
- **Tạo group truyền thống thay vì Synchronized Group**: Khi thêm/xóa file, file `.pbxproj` liên tục sinh diff lớn và xung đột merge git. Cách tránh: khi tạo group, chọn Synchronized Root Group trỏ vào thư mục nguồn trên đĩa.
- **Bật App Group capability quá sớm**: Bật capability App Group khi chỉ có Personal Team (Apple ID miễn phí) sẽ gây lỗi Signing (yêu cầu paid Apple Developer account). Cách tránh: chỉ ghi nhận App Group ID vào tài liệu/tiến độ, chưa thêm entitlement vào target.
- **Đặt Deployment Target cao hơn 26.0**: Vô tình để target theo phiên bản mới nhất của simulator dẫn tới không đúng yêu cầu dự án. Cách tránh: luôn kiểm tra Deployment Target đặt đúng `iOS 26.0`.

## Đọc thêm
- [Xcode 16 / 26 Release Notes - Synchronized Groups](https://developer.apple.com/documentation/xcode-release-notes)
- [Migrating to Swift 6 and Actor Isolation](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/)
- [Managing Schemes in Xcode](https://developer.apple.com/documentation/xcode/customizing-the-build-schemes-for-a-project)
