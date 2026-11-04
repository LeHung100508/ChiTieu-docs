# iL-02 · Xcode, simulator và chạy trên iPhone thật

## Vì sao cần
Mọi task iOS sau này đều cần một vòng "sửa → build → chạy" ổn định trên cả simulator lẫn máy thật. Task này dựng vòng đó và giúp bạn quen các khái niệm của Xcode tương ứng với Android Studio và Gradle mà bạn đã biết.

## Khái niệm chính
- **Project, target, scheme**: `.xcodeproj` chứa một hay nhiều *target*. Mỗi target là một sản phẩm build ra: app, test, extension. *Scheme* chọn target nào được build, chạy, test và với *configuration* nào (Debug hay Release).
- **Bundle Identifier**: định danh duy nhất của app, giống `applicationId`. Với Apple ID miễn phí, ID này phải chưa ai đăng ký, nên hãy đặt theo kiểu `com.<tên-bạn>.SachDoc`.
- **Deployment Target**: phiên bản iOS thấp nhất app chạy được, giống `minSdk`. SDK đi kèm Xcode thì giống `compileSdk`.
- **Simulator runtime**: Xcode không cài sẵn runtime. Bạn phải tải riêng trong Xcode › Settings › Components hoặc bằng lệnh. Simulator chạy code đã build cho Mac (Apple silicon), không giả lập phần cứng như AVD.
- **Signing**: iOS chỉ chạy app có chữ ký kèm *provisioning profile*, tức tấm vé cho biết app nào được cài lên thiết bị nào. "Automatically manage signing" cùng một *Personal Team* (Apple ID miễn phí) sẽ tự tạo vé đó.
- **Developer Mode**: công tắc trên iPhone, giống Developer options cộng USB debugging. Công tắc chỉ hiện ra sau khi iPhone đã kết nối với Xcode, và máy phải khởi động lại khi bật.

## So với cách cũ bạn quen
| Cũ (Android) | Mới (iOS) | Khác biệt chính |
| --- | --- | --- |
| Android Studio + Gradle | Xcode + `xcodebuild` | Cấu hình build nằm trong `.xcodeproj` (Build Settings), không phải file script |
| Module Gradle, build variant | Target, scheme, configuration | Scheme do người dùng chọn khi chạy, không sinh task kiểu `assembleDebug` |
| `applicationId`, `minSdk` | Bundle Identifier, Deployment Target | Bundle ID phải là duy nhất trên toàn hệ thống của Apple |
| AVD (emulator) | Simulator | Nhanh hơn nhưng không giống máy thật hoàn toàn (camera, hiệu năng, quyền) |
| Debug keystore tự sinh | Signing tự động + provisioning profile | Với tài khoản miễn phí, profile chỉ có hạn 7 ngày; hết hạn thì chạy lại từ Xcode |
| `adb` | `xcrun simctl`, `xcrun devicectl` | Một công cụ cho simulator, một công cụ cho máy thật |

## API và cú pháp cần biết
- `xcodebuild -version`: xem phiên bản Xcode, cũng là tiêu chí xong của task.
- `xcodebuild -downloadPlatform iOS`: tải runtime simulator iOS bằng dòng lệnh.
- `xcrun simctl list devices available`: liệt kê simulator dùng được.
- `xcrun devicectl list devices`: liệt kê iPhone đã ghép đôi với Mac.
- `@main` cùng giao thức `App`: điểm vào của app SwiftUI, thay cho `Application` + `MainActivity` khai báo trong manifest. Xcode sinh sẵn khi tạo project.

Ví dụ thao tác với simulator (không liên quan tới SachDoc):
```bash
xcrun simctl boot "iPhone 17"          # bật một simulator theo tên
xcrun simctl openurl booted https://apple.com
xcrun simctl shutdown all
```

## Lỗi hay gặp
- **Tạo git repo lồng nhau**: hộp thoại tạo project có ô "Create Git repository on my Mac". Repo `ChiTieu-ios` đã có sẵn, nên phải bỏ chọn ô này, nếu không sẽ có `.git` thứ hai trong `luyen-tap/SachDoc`.
- **Không có simulator để chọn**: chưa tải runtime iOS. Hãy tải trong Settings › Components rồi mở lại danh sách đích chạy.
- **"Failed to register bundle identifier"**: Bundle ID đã bị người khác dùng. Đổi phần tên cho riêng của bạn.
- **"Untrusted Developer" khi mở app trên iPhone**: lần đầu phải tin chứng chỉ trong Cài đặt › Cài đặt chung › Quản lý VPN & thiết bị.
- **Không thấy Developer Mode**: iPhone chưa từng kết nối với Xcode. Cắm cáp, chọn "Tin cậy", mở Xcode › Window › Devices and Simulators, rồi mới vào Cài đặt › Quyền riêng tư & Bảo mật.

## Đọc thêm
- [Creating an Xcode project for an app](https://developer.apple.com/documentation/xcode/creating-an-xcode-project-for-an-app)
- [Downloading and installing additional Xcode components](https://developer.apple.com/documentation/xcode/downloading-and-installing-additional-xcode-components)
- [Running your app in Simulator or on a device](https://developer.apple.com/documentation/xcode/running-your-app-in-simulator-or-on-a-device)
- [Enabling Developer Mode on a device](https://developer.apple.com/documentation/xcode/enabling-developer-mode-on-a-device)
