# Rà soát iS0-02 · Chia package

Ngày rà soát: 2026-11-14 · Nhánh `task/iS0-02-chia-package` · Trạng thái: **Xong** (đóng sau rà lại lần 3 của bản làm lại, 2026-11-15)

## Lệnh đã chạy
- `cd Core && xcodebuild -scheme Core-Package -destination 'platform=iOS Simulator,name=iPhone 17' test` → **TEST SUCCEEDED**, 5/5 test qua.
- `xcodebuild -project ChiTieu.xcodeproj -scheme ChiTieu … build` → **BUILD SUCCEEDED**. Chỉ còn cảnh báo AppIntents, không phải từ code.
- Đối chiếu tiêu chí xong: package `Core` đã đạt. Còn thiếu `Features`, app chưa liên kết module nào, và chưa ghi quy tắc phụ thuộc.

## Bắt buộc sửa (3)

### 1. Chưa có package `Features`
- **Vị trí:** gốc repo. Hiện chỉ có `Core/`.
- **Vì sao:** tiêu chí yêu cầu có `Features/{QuickAdd, Transactions, Budget, Stats, Settings, Onboarding}`, mỗi module có target test.
- **Hướng sửa:**
  - Tạo local package thứ hai ở `Features/`, phụ thuộc `Core` qua đường dẫn tương đối. Ví dụ trong file lý thuyết của bạn đã đúng hướng này.
  - Mỗi feature là một target, kèm một test target.
  - Mỗi feature chỉ khai báo phụ thuộc vào product của `Core`, không phụ thuộc feature nào khác.
  - Thêm package này vào `packages` trong `project.xcproj`.

### 2. Target App chưa lắp ghép module nào
- **Vị trí:** `ChiTieu.xcodeproj/project.xcproj:13-18`. Package `Core` đã được thêm vào project, nhưng target `ChiTieu` không liên kết product nào. Phase `frameworks` đang trống.
- **Vì sao:** tiêu chí là "Target App chỉ lắp ghép". Nếu app không liên kết module, chưa có gì chứng minh app build được cùng các module. Lỗi cấu hình (platform, isolation, access level) sẽ chỉ lộ ra ở iS1.
- **Hướng sửa:**
  - Cho target `ChiTieu` liên kết các product của `Features`, cùng các product `Core` mà app thật sự cần.
  - Trong app, `import` ít nhất một module để chứng minh liên kết đã thông.
  - Không liên kết `CoreTesting` vào app, vì module này chỉ dành cho test.

### 3. Chưa ghi quy tắc phụ thuộc
- **Vị trí:** `Core/Package.swift`. Không có README.
- **Vì sao:** tiêu chí yêu cầu quy tắc phụ thuộc "được ghi trong Package.swift hoặc README". Sau này người rà soát, CI hay agent khác cần một nơi để đối chiếu.
- **Hướng sửa:** viết vài dòng comment ở đầu mỗi `Package.swift`, hoặc một `README.md` chung, nêu rõ:
  - ai được phụ thuộc ai;
  - Parser và Model chỉ được dùng Foundation;
  - Feature không phụ thuộc Feature khác;
  - `CoreTesting` chỉ dành cho test target.

## Nên sửa (3)

### 1. Test target không import đúng module của nó
- **Vị trí:** `Core/Tests/*/…Tests.swift:9`. Cả 5 file đều `@testable import Model`, kể cả `ParserTests`, `DataTests`, `DesignSystemTests` và `CoreTestingTests`.
- **Vì sao:**
  - Test đang "qua" nhưng không chạm vào module mà nó đứng tên. `ParserTests` không hề kiểm tra `Parser` biên dịch và import được.
  - File biên dịch được vì `Model` là phụ thuộc bắc cầu. Đây đúng là kiểu import ngầm mà quy tắc ranh giới cần tránh.
- **Hướng sửa:**
  - Mỗi test import đúng module của nó.
  - Thay `#expect(true)` bằng một khẳng định nhỏ chạm vào type public của module đó.

### 2. Parser phụ thuộc Model: phải giữ Model thuần
- **Vị trí:** `Core/Package.swift:23`.
- **Vì sao:** `Parser` chỉ được dùng Foundation, nên mọi thứ nó phụ thuộc cũng phải thuần như vậy. Nếu sau này `@Model` của SwiftData hoặc type SwiftUI lọt vào `Model`, `Parser` sẽ gián tiếp dính SwiftData.
- **Hướng sửa:**
  - Ghi vào quy tắc (Bắt buộc 3): `Model` chỉ chứa value type thuần, như `Money` và kết quả parser.
  - Entity SwiftData nằm trong `Data`.
  - Tự trả lời câu hỏi: bên Android, `core:parser` phụ thuộc gì?

### 3. `platforms` có `.macOS(.v14)`
- **Vị trí:** `Core/Package.swift:10`.
- **Vì sao:** giữ macOS để chạy `swift test` nhanh trên Mac là hợp lý, nhất là cho Parser. Nhưng mốc 14 lệch với iOS 26.0. Khi `DesignSystem` dùng API mới (như Liquid Glass), `swift build` trên macOS sẽ báo lỗi availability, dù build cho iOS vẫn qua.
- **Hướng sửa:**
  - Nâng mốc macOS cho khớp đời 26.
  - Hoặc bỏ macOS và chỉ test qua `xcodebuild`.
  - Chọn một cách và ghi lý do.

## Gợi ý thêm để học (4)
1. **Default isolation của package khác của app.**
   - App target đang đặt `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`. Target trong package thì mặc định **nonisolated**.
   - Với Parser, Model và Data, nonisolated là đúng.
   - Với các target Feature (view, `@Observable` view model), nên cân nhắc `swiftSettings` với `.defaultIsolation(MainActor.self)` (SE-0466, tools ≥ 6.2) cho cùng hành vi với app. Chốt điều này khi tạo `Features`.
2. **Type trùng tên module.** `Core/Sources/DesignSystem/DesignSystem.swift:10` khai báo `public struct DesignSystem`, trùng tên module. Swift sẽ hiểu `DesignSystem.X` là thành viên của struct chứ không phải của module, gây lỗi khó hiểu khi cần ghi rõ tên module. Các module khác đã dùng hậu tố `…Module`, nên đồng nhất cách này.
3. **`swift-tools-version: 6.4`** buộc mọi máy và runner CI phải có Swift 6.4 (Xcode 27.2). Hãy kiểm tra runner có phiên bản này khi làm iS0-04. Nếu không, hạ xuống mức thấp nhất đủ cho các tính năng bạn dùng.
4. **File lý thuyết `iS0-02-chia-package.md`**, vài chỗ chưa chính xác:
   - Dòng 11: "chỉ import Foundation để đảm bảo an toàn luồng". An toàn luồng không đến từ việc import. Nó đến từ value type, `Sendable` và isolation. Lý do thật của "chỉ Foundation" là: không dính UI hay framework lưu trữ, test được nhanh trên Mac, và giữ logic giống bản Android.
   - Dòng 13: test "không cần khởi động simulator" chỉ đúng khi chạy `swift test` trên macOS. `xcodebuild … test` với đích iOS vẫn dùng simulator.
   - Dòng 59: thiếu `platforms` không làm package "mặc định biên dịch theo macOS". Package sẽ dùng mốc tối thiểu mặc định của toolchain cho nền tảng đích. Khi dùng API iOS 26, kết quả là **lỗi** availability chứ không phải cảnh báo.
   - Còn thiếu: SwiftPM không tự cấm Feature → Feature hay import bắc cầu (xem Nên sửa 1). Quy tắc chỉ được giữ bằng cách khai báo `dependencies` cẩn thận và rà soát `Package.swift`.

## Điểm làm tốt
- Đặt tên `CoreTesting` thay cho `Testing` là quyết định đúng, vì tên `Testing` sẽ đụng module Swift Testing.
- Dùng local package với một target test cho mỗi module, tools-version 6.x nên Swift 6 language mode bật sẵn. Package được thêm vào project bằng `"kind": "local"`, gọn và ít xung đột.
- Có `Core/.gitignore` bỏ qua `.build` và `xcuserdata`. Trạng thái trong tiến độ ghi "Đang làm" chứ không tự ghi Xong.

## Kết luận
Chưa đóng được task. Cần sửa 3 mục Bắt buộc và các mục Nên sửa (hoặc ghi `hoãn <mục>`). Sửa xong thì báo lại, mentor sẽ rà phần thay đổi.

---

## Làm lại từ đầu (2026-11-14)
Người dùng chọn bỏ lần làm thứ nhất (package `Core` chưa commit, thay đổi trong `project.xcproj`) để làm lại theo Bước 2–3. Các nhận xét ở trên vẫn dùng làm checklist cho lần làm mới.

---

## Rà lại lần 2 (2026-11-14)
Lệnh đã chạy:
- `cd Core && xcodebuild -scheme Core-Package -destination 'platform=iOS Simulator,name=iPhone 17' test` → **TEST SUCCEEDED** (5/5 tests passed).
- `cd Features && xcodebuild -scheme Features-Package -destination 'platform=iOS Simulator,name=iPhone 17' test` → **TEST SUCCEEDED** (6/6 tests passed).
- `xcodebuild -project ChiTieu.xcodeproj -scheme ChiTieu -destination 'generic/platform=iOS Simulator' build` → **BUILD SUCCEEDED** (19 targets linked và biên dịch sạch).
- `swift test --package-path Core` và `swift test --package-path Features` đều pass 100%.

### Bắt buộc sửa (0)
Không còn mục nào.
1. Đã có package `Features` với 6 module (`QuickAdd`, `Transactions`, `Budget`, `Stats`, `Settings`, `Onboarding`) và 6 test target tương ứng.
2. Target App `ChiTieu` đã liên kết đúng 6 feature products và 3 core products (`Data`, `DesignSystem`, `Model`).
3. Quy tắc phụ thuộc đã được ghi rõ ràng, chi tiết ở đầu cả hai file `Core/Package.swift` và `Features/Package.swift`.

### Nên sửa (0)
Không còn mục nào.
1. Mọi test target đều import đúng module tương ứng của nó.
2. `Model` giữ thuần Foundation value types.
3. `platforms` đồng nhất `.iOS("26.0")` và `.macOS(.v26)`.

### Gợi ý thêm để học (1)
- Ở task tiếp theo `iS0-03`, khi cài đặt `swift-format` và `SwiftLint`, chúng ta sẽ cấu hình để tự động kiểm tra định dạng và quy chuẩn code cho cả `ChiTieu`, `Core` và `Features`.

### Điểm làm tốt
- Chia 2 local packages `Core` và `Features` rất tường minh, ranh giới rõ ràng.
- Đặt tên `CoreTesting` tránh xung đột với Apple `Testing` framework.
- Cấu hình default actor isolation `@MainActor` chính xác cho các target UI trong `Features`.
- Comment luật phụ thuộc được ghi chi tiết, chặt chẽ (đặc biệt việc giới hạn `Parser` chỉ cho `QuickAdd` và `CoreTesting` chỉ cho test targets).

**Kết luận:** Đóng được task.

---

## Rà lại lần 1 · bản làm lại (2026-11-15)
**Lệnh đã chạy:**

| Lệnh | Kết quả |
| --- | --- |
| `Core`: `xcodebuild -scheme Core-Package … test` và `swift test` | ✅ |
| `Features`: `xcodebuild -scheme Features-Package … test` | ❌ **TEST FAILED** |
| `Features`: `swift test` | ❌ **Build failed** |
| App: `xcodebuild … build` | ✅ BUILD SUCCEEDED |

**Đã đạt so với lần 1:**
- Có đủ 2 package, 11 module, mỗi module có một test target.
- Luật phụ thuộc được ghi ở đầu cả hai `Package.swift`.
- `DesignSystem` và các Feature đặt `.defaultIsolation(MainActor.self)`.
- Test của Core import đúng module của nó.
- App link 6 feature và không link `CoreTesting`.
- Tools-version 6.2.

### Bắt buộc sửa (1)
1. `Features/Tests/*/…Tests.swift:13`: cả 6 test vẫn gọi tên cũ `…PlaceHolder()` hoặc `…Placeholder()`, trong khi type đã đổi thành `…View`. Test của package `Features` vì vậy không biên dịch được.
   - **Vì sao:** tiêu chí xong là "test của mọi module chạy được bằng xcodebuild". Lỗi này lẽ ra lộ ngay nếu chạy lại lệnh tự kiểm tra của bước 3 trước khi báo xong.
   - **Hướng sửa:** cập nhật test theo tên View mới. Sau đó chạy lại cả bốn lệnh test, gồm `xcodebuild` và `swift test` cho mỗi package.

### Nên sửa (2)
1. **Điều hướng lệch wireframe** (`ChiTieu/ContentView.swift:20-52`). Wireframe màn hình 2 quy định 4 tab là **Tổng quan** (Trang chủ, thuộc `QuickAdd`), Giao dịch, Ngân sách, Thống kê. **Cài đặt** không phải tab mà mở từ nút ⚙.
   - Bản hiện tại: Cài đặt là tab, không có tab Tổng quan, còn `QuickAdd` là sheet mở từ nút + trên tab Giao dịch.
   - **Hướng sửa:** đưa `QuickAddView` thành tab đầu. Chuyển Cài đặt sang nút ⚙ do app gắn vào toolbar của tab Tổng quan. Nếu muốn giữ sheet Nhập nhanh, cứ để; theo wireframe màn hình 3, nó được mở từ widget, Control và phím tắt.
2. **App link thừa product Core** (`project.xcproj`, `package-product-members`). App đang link `Data`, `DesignSystem` và `Model` nhưng không `import` module nào trong số đó. Hãy bỏ, khi nào app thật sự dùng thì thêm lại. Ví dụ ở iS1, app tạo `ModelContainer` thì mới cần `Data`.

### Gợi ý thêm để học (3)
1. Onboarding đang là sheet mở từ nút "Hướng dẫn". Wireframe dùng `fullScreenCover` ở lần mở đầu tiên. Để như hiện tại thì được, sửa khi làm Onboarding thật.
2. `ContentView.swift:18`: biến `showingQuicAdd` viết sai chính tả.
3. Test của Core vẫn là `#expect(true)`. Đây là smoke test nên chấp nhận được. Test thật sẽ có ở iS0-07.

**Kết luận:** chưa đóng được task. Còn 1 mục Bắt buộc và 2 mục Nên sửa (hoặc ghi `hoãn <mục>`).

## Rà lại lần 2 (2026-11-15)
**Lệnh đã chạy:**

| Lệnh | Kết quả |
| --- | --- |
| `Core`: xcodebuild test và swift test | ✅ |
| App build | ✅ |
| `Features`: xcodebuild test và swift test | ❌ **vẫn lỗi** |

- **Nên sửa 1 (điều hướng): đạt.**
  - Tab đầu là Tổng quan (`QuickAddView`), có đủ 4 tab đúng wireframe.
  - Nút ⚙ do app gắn và mở sheet Cài đặt.
  - Onboarding dùng `fullScreenCover`.
- **Nên sửa 2 (link thừa): đạt.** App chỉ còn link 6 product feature.
- **Bắt buộc 1: chưa đạt.** 5/6 test đã đúng tên, nhưng `Features/Tests/QuickAddTests/QuickAddTests.swift:13` gọi `OnboardingView()` thay vì `QuickAddView()`. Kết quả là `error: cannot find 'OnboardingView' in scope`.
  - Lỗi này đồng thời cho thấy ranh giới module đang hoạt động: `QuickAddTests` chỉ phụ thuộc `QuickAdd`, nên không thấy được `Onboarding`.

**Gợi ý thêm:** `OnboardingView` mở bằng `fullScreenCover` nhưng chưa có nút đóng. Mở ra rồi thì không thoát được, chỉ còn cách tắt app. Nên thêm nút đóng bằng `@Environment(\.dismiss)`, đặt ở app hoặc trong View.

**Kết luận:** chưa đóng được task. Còn 1 lỗi một dòng. Sửa xong thì chạy `swift test` trong `Features/` để thấy xanh trước khi báo.

## Rà lại lần 3 (2026-11-15)
**Lệnh đã chạy:** `xcodebuild … test` và `swift test` cho cả `Core` lẫn `Features` đều ✅. App build ✅, code không có cảnh báo.
- **Bắt buộc 1: đạt.** `QuickAddTests` đã gọi đúng `QuickAddView()`.
- **Gợi ý:** `OnboardingView` đã có nút "Bỏ qua" dùng `@Environment(\.dismiss)`, đạt.
- Hoãn sang các task sau (không chặn):
  - Test hiện vẫn là smoke test `#expect(true)`. Test thật bắt đầu từ iS0-07.
  - Chuỗi đang viết cứng. String Catalog làm ở iS1.
  - Dòng header "Created by…" sẽ dọn ở iS0-03.

**Kết luận:** Đóng được task.
