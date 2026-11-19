# Rà soát iS0-05 · Design system: màu và chữ

Ngày rà soát: 2026-11-19 · Nhánh `task/iS0-05-ds-mau-chu` · Trạng thái: **Xong** (đóng 2026-11-19 sau khi người dùng xác nhận Preview)

> Bản rà soát tự viết trước đó (ghi "Xong") đã được thay bằng bản này. Theo luật, mentor rà soát và đổi trạng thái.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ✅ exit 0 |
| `swiftlint lint --strict` | ✅ exit 0 |
| `cd Core && swift test` (macOS, giống bước test trên CI) | ❌ **lỗi biên dịch** |
| `cd Core && xcodebuild -scheme Core-Package … test` (iOS) | ❌ **lỗi biên dịch** |
| App: `xcodebuild … build` | ✅ BUILD SUCCEEDED |
| Tự tính độ tương phản WCAG cho 4 màu trạng thái và màu thương hiệu, trên nền sáng (`#FFFFFF`, `#F2F2F7`) và nền tối (`#000000`, `#1C1C1E`) | ✅ Thấp nhất 4,73 (StatusWarning trên `#F2F2F7`). Mọi cặp đều ≥ 4,5, đạt AA cho chữ thường |

## Bắt buộc sửa (1)

### 1. Test của `DesignSystem` không biên dịch được
- **Vị trí:** `Core/Tests/DesignSystemTests/DesignSystemTests.swift:36-48`.
- **Lỗi:**
  ```
  main actor-isolated static property 'compact' can not be referenced from a nonisolated autoclosure
  ```
- **Vì sao:**
  - Target `DesignSystem` đặt `.defaultIsolation(MainActor.self)`, nên `AppSpacing.*` và `AppRadius.*` đều bị cô lập trên MainActor.
  - Target test thì mặc định nonisolated. Hàm `@Test` không có `@MainActor` nên không được đọc các giá trị đó một cách đồng bộ.
  - Đây đúng là điều đã ghi ở mục "Lỗi hay gặp" của iS0-02.
  - Tiến độ ghi "unit test nạp asset màu và kiểm tra token". Nếu push, CI sẽ đỏ ở bước `Run Core Tests`.
- **Hướng sửa:** chọn một trong hai cách và ghi lý do.
  - Gắn `@MainActor` cho hàm test, giống cách các test của `Features` đang làm.
  - Hoặc đánh dấu các token là `nonisolated`. Hằng số `CGFloat` là `Sendable` và không cần MainActor, nên cách này cho phép đọc token ở mọi context, kể cả từ code nonisolated sau này. Cách này có lợi hơn về lâu dài.
  - Sửa xong, chạy cả `swift test` lẫn `xcodebuild … test` trong `Core/`.

## Nên sửa (1)

### 1. Preview đang tự làm sai tương phản và chưa cho thấy đúng các kiểu chữ
- **Vị trí:** `Core/Sources/DesignSystem/DesignSystemPreview.swift:26-48`.
- **Vấn đề:**
  - Các thẻ màu đặt chữ `textSecondary` (xám) lên nền màu đậm. Ví dụ xám trên `BrandPrimary` ở chế độ sáng chỉ đạt khoảng **1,9:1**.
  - Phần typography thì áp `amountStyle` (có `monospacedDigit`) cho cả 10 style, và mọi dòng đều là cùng một chuỗi số tiền. Vì vậy không nhìn ra `body`, `caption`, `screenTitle` trông thế nào với chữ thường, và cũng không biết dòng nào là style nào.
- **Vì sao:** tiêu chí của task là "Preview hiện đúng ở sáng, tối và cỡ chữ lớn nhất". Preview là công cụ để chính bạn kiểm tra bảng màu và Dynamic Type. Khi Preview tự tạo ra lỗi tương phản thì không còn dùng nó để kiểm tra được nữa.
- **Hướng sửa:**
  - Thẻ màu: chọn màu chữ tương phản với nền, hoặc hiển thị ô màu kèm nhãn đặt bên ngoài ô.
  - Typography: mỗi dòng ghi tên style kèm một câu chữ thường. Chỉ áp `amountStyle` cho các style số tiền.

## Gợi ý thêm để học (4)
1. **Màu thương hiệu: đã chốt `#1E7A4E`** theo mockup iOS (người dùng quyết định ngày 2026-11-19). Bản Android giữ `#1B6B4A`. Đã sửa `task-ios.html` và `ke-hoach-ios.html` cho khớp. Khi đổi `BrandPrimary` ở chế độ sáng, cần chú ý:
   - Tương phản trên nền `#FFFFFF` là 5,32, trên `#F2F2F7` là 4,77, cả hai đều đạt AA.
   - Trên `BrandTintBackground` hiện tại (`#D8F0E2`), tương phản chỉ còn **4,43**, dưới ngưỡng 4,5 cho chữ thường. Cần làm nền tint sáng hơn, hoặc chỉ dùng cặp này cho chữ lớn (ngưỡng 3:1).
   - Biến thể tối hiện tại (`#42C88C`) vẫn cần kiểm tra lại cho hợp với màu sáng mới.
2. **Nhánh `#if canImport(AppKit)` trong `Colors.swift`** chỉ có để `swift test` chạy được trên macOS trong CI. Đây chính là quyết định iS0-04 đã nhắc trước. Giữ cách này thì mỗi màu hệ thống mới đều phải viết hai nhánh. Còn một cách khác: chạy test `Core` bằng `xcodebuild … test` trên simulator. Nên chốt trước iS0-06, khi số component tăng lên.
3. **`DesignSystem.swift`** vẫn còn `public struct DesignSystemModule {}` từ iS0-02. Giờ module đã có nội dung thật, file giữ chỗ này là code chết.
4. **Ngoại lệ.** Tiến độ ghi hai dòng `ngoại lệ:` cho bảng màu (gồm cả `Colors.xcassets`, `Colors.swift` và phần `resources` trong `Package.swift`) và cho `Typography.swift` cùng `LayoutTokens.swift`. Như vậy phần lớn nội dung của task do agent viết. Đây là quyền của bạn và đã được ghi đúng luật. Nhưng mục tiêu số 1 là học công nghệ. Để chắc đã nắm, bạn tự trả lời ba câu:
   - `Color("BrandPrimary", bundle: .module)` lấy `.module` từ đâu ra?
   - Vì sao `.system(.title2, design: .rounded)` tự co giãn theo Dynamic Type?
   - Vì sao `resources: [.process("Resources")]` là bắt buộc?

## Điểm làm tốt
- Màu thương hiệu và màu trạng thái đặt trong Asset Catalog, có biến thể sáng/tối. Mọi cặp chữ trên nền đều đạt WCAG AA. Đã tự kiểm chứng.
- Kiểu chữ dựa hoàn toàn trên text style của hệ thống, nên tự co giãn theo Dynamic Type. Số tiền dùng `.rounded` kết hợp `monospacedDigit()`, đúng tinh thần iOS. Màu nền và màu chữ ưu tiên màu hệ thống, nên tương thích Liquid Glass.
- Có 3 `#Preview` (Sáng, Tối, AX5) và có minh họa `.glass` và `.glassProminent`. Test kiểm tra asset bằng `bundle: .module` là test có ý nghĩa.

## Kết luận
Chưa đóng được task. Cần sửa Bắt buộc 1 và Nên sửa 1 (hoặc ghi `hoãn <mục>`), và đổi `BrandPrimary` sang `#1E7A4E` theo Gợi ý 1. Sửa xong thì báo lại, mentor sẽ rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-19)
**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `cd Core && swift test` → ✅.
- `cd Core && xcodebuild -scheme Core-Package … test` → ✅ TEST SUCCEEDED.

**Đã đạt:**
- **Bắt buộc 1:** `AppSpacing` và `AppRadius` đã chuyển sang `nonisolated`. Test biên dịch được và chạy qua trên cả macOS lẫn iOS.
- **Nên sửa 1, phần typography:** mỗi dòng có tên token kèm câu mẫu. Chỉ các style số tiền mới dùng `amountStyle`.

**Còn lại:**
1. **`BrandPrimary` vẫn là `#1B6B4A`**, chưa đổi sang `#1E7A4E` như đã chốt. Đề xuất giá trị (mentor đã tính tương phản):

   | Asset | Sáng | Tối | Tương phản |
   | --- | --- | --- | --- |
   | `BrandPrimary` | `#1E7A4E` | `#42C88C` (giữ nguyên) | Sáng trên `#FFFFFF`: 5,32. Tối trên `#1C1C1E`: 8,0 |
   | `BrandTintBackground` | `#E6F5EC` (thay `#D8F0E2`) | `#143D2B` (giữ nguyên) | Chữ brand trên tint: sáng 4,72, tối 5,7 |

2. **Preview ở chế độ tối:** các thẻ đặt chữ `.white` lên nền màu. Biến thể tối của các màu này đều sáng, nên tương phản chỉ còn **1,7 đến 2,8** (`#42C88C` 2,13; `#4ADE97` 1,72; `#FF7A45` 2,59; `#FF6B6B` 2,78). Nếu dùng chữ đen trên các nền này thì đạt 7,6 đến 12,2.
   - **Hướng sửa:** thêm một màu "chữ trên nền nhấn", trắng ở chế độ sáng và đen ở chế độ tối, rồi dùng nó cho các thẻ. Màu này cũng sẽ cần cho chip có nền đặc ở iS0-06.
3. Thẻ "Separator card" đang dùng `surfaceCard` chứ không phải `separator`. Ngoài ra `cardPrimary(text:)` (`DesignSystemPreview.swift:107`) không còn được gọi ở đâu, là code chết.

**Kết luận:** chưa đóng được task.

## Ngoại lệ đã viết hộ (2026-11-19)
Người dùng xác nhận `ngoại lệ: colorset BrandPrimary, BrandTintBackground, sửa DesignSystemPreview`. Mentor chỉ sửa đúng các phần đó:
- `BrandPrimary.colorset`: sáng đổi từ `#1B6B4A` thành `#1E7A4E`. Bản tối giữ `#42C88C`.
- `BrandTintBackground.colorset`: sáng đổi từ `#D8F0E2` thành `#E6F5EC`. Bản tối giữ `#143D2B`.
- `DesignSystemPreview.swift`:
  - Chữ trên nền nhấn dùng `onFill`: trắng ở chế độ sáng, đen ở chế độ tối, đọc từ `@Environment(\.colorScheme)`.
  - Nhãn thẻ ghi tên token.
  - Thẻ `separator` dùng đúng `AppColor.separator`.
  - Thẻ màu dàn đủ chiều ngang.
  - Bỏ `cardPrimary` (code chết).

Không tạo asset "chữ trên nền nhấn" vì phần này nằm ngoài ngoại lệ. `onFill` chỉ là biến riêng của Preview. Khi làm iS0-06, người dùng tự quyết định có đưa nó thành token hay không.

**Lệnh đã chạy sau khi sửa:** lint `--strict` (cả hai công cụ) ✅, `swift test` ✅, `xcodebuild … test` ✅, app build ✅. Mentor chưa xem trực quan được Preview. Người dùng mở Canvas trong Xcode và xem đủ ba bản Sáng, Tối, AX5.

**Kết luận:** đóng được task khi người dùng xác nhận ba Preview hiển thị đúng.

## Rà lại lần 2 (2026-11-19)
Người dùng xác nhận trong Xcode Canvas rằng cả ba Preview (Sáng, Tối, AX5) đều hiển thị đúng: chữ trên mọi thẻ đọc rõ, cỡ chữ AX5 không làm vỡ bố cục. Toàn bộ tiêu chí xong của task đã đạt.

**Kết luận:** Đóng được task.
