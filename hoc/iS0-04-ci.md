# iS0-04 · CI Pipeline (Continuous Integration với GitHub Actions cho iOS)

> Khái niệm về CI và GitHub Actions đã có ở [S0-06-ci.md](S0-06-ci.md). File này tập trung vào thiết lập CI trên môi trường macOS cho dự án iOS.

## Vì sao cần
Một dự án iOS hiện đại với multi-package và bộ công cụ kiểm tra chất lượng (swift-format, SwiftLint, Swift Testing) cần một môi trường độc lập, khách quan để tự động xác thực code trước khi gộp vào `main`.

CI Pipeline trên GitHub Actions đóng vai trò người gác cổng tự động, đảm bảo mọi Pull Request và commit trên `main`:
- Định dạng và phân tích tĩnh sạch 100% (swift-format và SwiftLint với `--strict`).
- 100% Unit Tests của các package vượt qua.
- App target biên dịch thành công, không bị lỗi liên kết hay lỗi data race ở mức biên dịch (Swift 6 language mode).

## Khái niệm chính
- **Runner Xcode 27 (`runs-on: xcode-27`)**: Từ 2026, GitHub Actions cung cấp runner image gắn theo phiên bản Xcode chính (chạy trên macOS 27, Apple Silicon `arm64`, miễn phí cho repo public). Runner cũ `macos-15` chỉ có Xcode 16.4 (Swift 6.1), không đọc được định dạng `project.xcproj` của Xcode 27 và không có SDK iOS 26/27.
- **Checkout kèm Submodule (`submodules: true`)**: Repo tài liệu `ChiTieu-docs` được gắn làm submodule tại `docs/` (chứa từ điển và bộ 200 câu mẫu parser dùng chung giữa Android và iOS). Bước checkout dùng `actions/checkout@v6` (chạy trên Node.js 24) kèm `submodules: true`.
- **Cài đặt công cụ cố định phiên bản**: Tránh dùng `brew install swiftlint` trên CI vì Homebrew luôn kéo bản mới nhất. CI sẽ tải file binary giải nén trực tiếp `portable_swiftlint.zip` đúng phiên bản `0.65.1` từ GitHub Releases.
- **3 chặng kiểm tra tuần tự (Pipeline Stages)**:
  1. **Lint**: Chạy `swift format lint` và `swiftlint lint` với cờ `--strict` trên phạm vi `ChiTieu Core Features`.
  2. **Test**: Chạy `swift test` cho cả 2 local packages `Core` và `Features`.
  3. **Build**: Dùng `xcodebuild build` để build App target `ChiTieu` với destination simulator.
- **Không đưa bí mật vào workflow (NFR-08, NFR-09)**: Repo là public nên tuyệt đối không commit file cấu hình chứa chứng chỉ phát hành, provisioning profile, hay API key cá nhân lên workflow.

## So với cách cũ bạn quen
| Cũ (Android - S0-06) | Mới (iOS - iS0-04) | Khác biệt chính |
| --- | --- | --- |
| `runs-on: ubuntu-latest` (Linux) | `runs-on: xcode-27` (macOS 27 Apple Silicon) | Build iOS bắt buộc chạy trên macOS và đúng đời Xcode; repo public được miễn phí |
| `./gradlew check assembleDebug` | Các lệnh CLI riêng biệt (`swift format`, `swiftlint`, `swift test`, `xcodebuild`) | iOS không có lifecycle Gradle chung, ta định nghĩa từng step rõ ràng |
| `setup-java` & `setup-gradle` | Nhãn runner (`runs-on: xcode-27`) | Phiên bản Xcode do nhãn runner quyết định (chỉ cần `xcode-select` nếu muốn đổi sang bản Xcode khác có sẵn trên cùng runner) |
| Submodule ít khi dùng | `actions/checkout@v6` với `submodules: true` | Cần submodule `docs/` để test parser đọc bộ test case dùng chung |

## API và cú pháp cần biết
- `actions/checkout@v6` kèm `submodules: true`: Nạp đủ submodule trên runtime Node.js 24 (thay cho `v4` dùng Node.js 20 đã deprecated).
- `swift test --enable-code-coverage` và `swift test --show-codecov-path`: Bật thu thập độ phủ mã nguồn và trả về đường dẫn file JSON coverage chuẩn LLVM (`data[0].totals.lines`) để báo cáo lên log hoặc `$GITHUB_STEP_SUMMARY`.
- `destination "generic/platform=iOS Simulator"`: Tùy chọn của `xcodebuild` để biên dịch cho kiến trúc simulator chung mà không cần khởi động simulator cụ thể, tiết kiệm tài nguyên CI.
- `portable_swiftlint.zip`: Bản release standalone của SwiftLint, tải về giải nén dùng ngay mà không cần qua trình quản lý gói.

Minh họa cấu trúc một job CI trên GitHub Actions:
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  check:
    runs-on: xcode-27
    steps:
      - uses: actions/checkout@v6
        with:
          submodules: true
      - run: swift test --package-path Core --enable-code-coverage
```

## Lỗi hay gặp
- **Chọn nhầm runner đời cũ (`macos-15` thay vì `xcode-27`)**: `macos-15` chỉ có Xcode 16.4 (Swift 6.1), không đọc được file `project.xcproj` của Xcode 27, không có SDK iOS 26/27, và từ chối `swift-tools-version: 6.2`.
- **Quên `submodules: true`**: CI checkout repo thành công nhưng thư mục `docs/` bị rỗng. Khi chạy test đọc file mẫu JSON ở các task sau sẽ bị lỗi `fileNotFound`.
- **Không dùng cờ `--strict` khi lint**: `swiftlint` hoặc `swift-format` khi phát hiện cảnh báo vẫn trả exit code 0 nếu thiếu `--strict`, khiến CI vẫn báo xanh dù code vi phạm quy tắc.
- **Tải phiên bản SwiftLint sai**: Khiến `swiftlint_version: 0.65.1` trong file cấu hình kích hoạt lỗi exit 2 trên CI.
- **Chạy xcodebuild với simulator cụ thể mà simulator đó chưa được cài trên runner**: Gây lỗi `Unable to find a destination matching the provided destination specifier`. Dùng `generic/platform=iOS Simulator` là cách an toàn nhất cho bước build kiểm tra.

## Đọc thêm
- [GitHub Actions: Building and testing Swift](https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-swift)
- [actions/runner-images: macOS 15 Readme](https://github.com/actions/runner-images/blob/main/images/macos/macos-15-arm64-Readme.md)
- [Apple: Testing Your App on Continuous Integration](https://developer.apple.com/documentation/xcode/testing-your-app-in-continuous-integration)
