# iS0-02 · Chia package (local Swift package)

> Bản mentor soạn lại, thay bản người dùng viết ở lần làm thứ nhất. Khái niệm chia module đã có ở [S0-04-khung-module.md](S0-04-khung-module.md). File này chỉ ghi phần riêng của iOS.

## Vì sao cần
Ta muốn có ranh giới cứng giữa các phần của app:
- `Parser` không dính UI hay SwiftData.
- Feature này không gọi thẳng feature khác.

Trên iOS, đơn vị tạo ranh giới là **module**. Mỗi module là một target của local Swift package, và chỉ thấy được những gì nó khai báo phụ thuộc cùng những gì là `public`.

## Khái niệm chính
- **Package, target, product:**
  - Một `Package.swift` mô tả một package.
  - Mỗi `target` là một module, có thư mục `Sources/<Tên>`.
  - `product` (`.library`) là phần package cho bên ngoài dùng: app hoặc package khác.
  - Target trong cùng package dùng nhau bằng tên. Package khác phải đi qua `product`.
- **Local package:** package nằm ngay trong repo. Project Xcode tham chiếu nó bằng đường dẫn. Package này phụ thuộc package kia bằng `.package(path:)`. Nhờ vậy cấu trúc module nằm trong file Swift dễ đọc, không nằm trong `project.xcproj`.
- **App target chỉ lắp ghép:**
  - App liên kết product của các feature, rồi dựng `App`, `Scene` và điều hướng.
  - App không chứa logic nghiệp vụ.
  - Phải *link* product thì app mới `import` được module.
- **Access control là ranh giới:**
  - Chỉ thứ `public` mới dùng được từ module khác.
  - `package` (Swift 5.9) cho phép dùng chung giữa các target trong **cùng** package, nhưng ẩn với bên ngoài. Ví dụ: Core muốn chia sẻ nội bộ mà không lộ cho Features.
- **Default isolation theo target:**
  - App target của bạn đang đặt `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`.
  - Target trong package thì mặc định **nonisolated**. Cài đặt của app không lan sang package.
  - Mỗi target tự chọn isolation qua `swiftSettings`. Phần thuần dữ liệu và logic nên để nonisolated. Phần UI có thể chọn MainActor.
- **SwiftPM không tự giữ luật kiến trúc:**
  - Một target *import được* phụ thuộc bắc cầu dù không khai báo. Lần làm trước đã gặp chuyện này: `DataTests` import được `Model`.
  - SwiftPM không biết "Feature không được phụ thuộc Feature".
  - Luật chỉ được giữ khi `dependencies` khai báo đúng, được ghi thành văn bản, và được rà soát mỗi khi `Package.swift` đổi.

## So với cách cũ bạn quen
| Cũ (Gradle) | Mới (SwiftPM) | Khác biệt chính |
| --- | --- | --- |
| `include(":core:model")` trong `settings.gradle.kts` | `.target(name:)` trong `Package.swift` | Một package có thể chứa nhiều module |
| `implementation(project(":core:model"))` | `dependencies: ["Model"]` (cùng package) hoặc `.product(name:package:)` (khác package) | Không có cặp `api`/`implementation`; import bắc cầu vẫn lọt |
| Module `:app` phụ thuộc mọi feature | App target link product trong Xcode (Frameworks, Libraries…) | Khai báo trong project, không trong `Package.swift` |
| Plugin `chitieu.jvm.library` cho `core:parser` | Target chỉ `import Foundation`, có thể build cho macOS | Không có plugin; giữ bằng luật và rà soát |
| `internal` của Kotlin (theo module) | `internal` (module) và `package` (package) | Swift có thêm mức `package` |
| `src/test` cùng module | `.testTarget` riêng, `@testable import <Module>` | Test là một module riêng, phải import đúng module cần test |

## API và cú pháp cần biết
- `// swift-tools-version: X.Y`: dòng đầu file.
  - Quyết định API `PackageDescription` nào dùng được.
  - Từ 6.0 trở lên, Swift 6 language mode được bật mặc định.
  - Tools version càng cao thì máy và CI càng phải mới.
- `platforms:` khai báo mốc OS tối thiểu, ví dụ `.iOS("26.0")`.
  - Nếu thêm macOS thì `swift test` chạy được ngay trên Mac, không cần simulator. Khi đó mốc macOS phải đủ mới cho API mà code dùng.
  - Nếu thiếu `platforms`, package dùng mốc mặc định cũ của toolchain, và gọi API mới sẽ ra **lỗi** availability.
- `.library(name:targets:)`, `.target(name:dependencies:swiftSettings:)`, `.testTarget(...)`.
- `.package(path: "../Khác")` kết hợp `.product(name: "X", package: "Khác")` để phụ thuộc sang package khác.
- `swiftSettings: [.defaultIsolation(MainActor.self)]` đặt isolation mặc định cho một target (SE-0466, tools ≥ 6.2).
- Chạy test:
  - `xcodebuild -scheme <TênPackage>-Package -destination '…' test`, chạy trong thư mục package. Scheme `-Package` do Xcode tự sinh.
  - `swift test`, chỉ chạy trên macOS.

Minh họa một package khác bài toán:
```swift
let package = Package(
    name: "Weather",
    platforms: [.iOS("26.0")],
    products: [.library(name: "Forecast", targets: ["Forecast"])],
    dependencies: [.package(path: "../Geo")],
    targets: [
        .target(name: "Forecast", dependencies: [.product(name: "Coordinates", package: "Geo")]),
        .testTarget(name: "ForecastTests", dependencies: ["Forecast"]),
    ]
)
```

## Lỗi hay gặp
- **Đặt tên module `Testing`:** tên này đụng module Swift Testing, vì vậy phải dùng tên khác. Tương tự, **không đặt type trùng tên module**, ví dụ `struct DesignSystem` trong module `DesignSystem`. Khi đó `DesignSystem.X` bị hiểu là thành viên của struct, không phải của module.
- **Test import sai module:** test vẫn qua nhờ import bắc cầu nhưng không kiểm tra được gì. Mỗi test target phải `@testable import` đúng module của nó.
- **Quên link product vào app:** package hiện trong Xcode nhưng `import` báo "No such module".
- **Mặc định isolation lệch giữa app và package:** view model đặt trong Feature mà không có `@MainActor`. Kết quả là lỗi concurrency xuất hiện ở chỗ app gọi vào. Hãy chốt isolation cho từng target ngay khi tạo.
- **Module lõi "bẩn":** `Parser` chỉ dùng Foundation nhưng lại phụ thuộc một module có `import SwiftData`. Khi đó Parser gián tiếp dính SwiftData. Mọi thứ Parser phụ thuộc cũng phải thuần.

## Đọc thêm
- [Swift Package Manager – PackageDescription](https://docs.swift.org/package-manager/PackageDescription/PackageDescription.html)
- [Apple – Organizing your code with local packages](https://developer.apple.com/documentation/xcode/organizing-your-code-with-local-packages)
- [SE-0386 – `package` access modifier](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0386-package-access-modifier.md)
- [SE-0466 – Control default actor isolation inference](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0466-control-default-actor-isolation.md)
