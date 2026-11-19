# iS0-05 · Design system: màu và chữ (SwiftUI & iOS 26)

> Khái niệm chung về Design System và vai trò màu ngữ nghĩa đã có ở [S0-07-design-system-mau-chu.md](S0-07-design-system-mau-chu.md). File này tập trung vào cách triển khai trên SwiftUI (iOS 26, Liquid Glass, Dynamic Type) trong local package `Core/DesignSystem`.

## Vì sao cần
Trên iOS, người dùng kỳ vọng ứng dụng tự nhiên như app hệ thống: tự đổi màu sáng/tối, chữ phóng to mượt theo cài đặt **Dynamic Type**, và giữ hiệu ứng **Liquid Glass** mà không bị chắp vá. Gom màu sắc, kiểu chữ, khoảng cách và bo góc vào `Core/DesignSystem` giúp mọi màn hình trong `Features` thống nhất nhận diện thương hiệu (`#1B6B4A`), dễ bảo trì và đạt NFR-11, NFR-13.

## Khái niệm chính
- **Asset Catalog màu thích ứng (`Colors.xcassets` trong SwiftPM):** Mỗi Color Set định nghĩa sẵn cặp màu cho **Any Appearance (Sáng)** và **Dark Appearance (Tối)** (và có thể thêm High Contrast). SwiftUI tự đổi màu theo môi trường mà không cần `if colorScheme == .dark`.
- **Màu hệ thống & màu thương hiệu:** Khác với Material 3 phủ màu hạt giống lên toàn bộ bề mặt, iOS HIG ưu tiên **màu nền hệ thống** (`systemGroupedBackground`, `secondarySystemGroupedBackground`, `Color.primary`, `Color.secondary`) kết hợp **màu nhấn thương hiệu** (`#1B6B4A` ở nền sáng; bản sáng hơn ở nền tối để đạt tương phản WCAG ≥ 4,5:1) làm `tint`.
- **Dynamic Type & thang chữ hệ thống:** Dùng các `Font.TextStyle` chuẩn (`.largeTitle`, `.title2`, `.headline`, `.body`, `.subheadline`, `.footnote`, `.caption`) để chữ tự co giãn từ `xSmall` đến `accessibility5`. Với số tiền, kết hợp `design: .rounded` (`SF Pro Rounded`) và `.monospacedDigit()` để các chữ số thẳng cột.
- **Token khoảng cách và bo góc (`CGFloat`):** Tách thành enum namespace bất biến (`Spacing`, `Radius`) thay vì viết số rải rác. Tránh khóa cứng chiều cao (`frame(height:)`) để bố cục không bị cắt chữ ở `accessibility5`.
- **Ưu tiên style hệ thống cho Liquid Glass:** Không tự vẽ nền kính giả bằng `opacity` + `blur`. Dùng màu nền hệ thống, `.tint(...)`, và style chuẩn của SwiftUI (như `.glassEffect()`, `.buttonStyle(.glass)`) để hệ điều hành tự xử lý **Reduce Transparency** và **Reduce Motion** (NFR-11).
- **Tương thích `swift test` trên macOS:** Vì `Core/Package.swift` khai báo cả `.macOS(.v26)` để chạy `swift test` nhanh trên CI, mọi màu lấy từ `UIColor` cần bọc `#if canImport(UIKit)` (có nhánh fallback macOS) hoặc đặt trong `Colors.xcassets` và đọc qua `Bundle.module`.

## So với cách cũ bạn quen

| Cũ (Android · Compose M3 - S0-07) | Mới (iOS · SwiftUI iOS 26) | Khác biệt chính |
| --- | --- | --- |
| `lightColorScheme` / `darkColorScheme` trong Kotlin | `Colors.xcassets` (`Color("...", bundle: .module)`) + màu hệ thống | Hệ thống tự chọn nhánh Sáng/Tối/Tương phản cao theo `Environment` |
| `MaterialTheme.colorScheme` bọc ngoài root | Namespace `enum` + `.tint(...)` / `Environment` | SwiftUI không cần bọc 1 Theme composable khổng lồ cho mọi màu |
| `Roboto Flex` + 15 kiểu `Typography` | `SF Pro` / `SF Pro Rounded` + `Font.TextStyle` (Dynamic Type) | Chữ tự co giãn tới `accessibility5` khi dùng `TextStyle` chuẩn |
| `MaterialTheme.shapes` (`RoundedCornerShape`) | Token `CGFloat` + `RoundedRectangle(cornerRadius:style: .continuous)` | `.continuous` tạo đường cong squircle đặc trưng của Apple |
| Dynamic Color từ hình nền (Android 12+) | Màu `tint` thương hiệu trên nền hệ thống + Liquid Glass | iOS giữ màu thương hiệu làm điểm nhấn, nhường bề mặt cho hệ thống |

## API và cú pháp cần biết
- `Color("TênAsset", bundle: .module)`: Đọc màu từ `Colors.xcassets` nằm trong Swift Package.
- `Font.system(.largeTitle, design: .rounded, weight: .bold)`: Kiểu chữ hệ thống có co giãn Dynamic Type kết hợp thiết kế chữ tròn.
- `.monospacedDigit()`: Modifier giữ các chữ số có cùng độ rộng ngang (tránh giật bố cục khi số tiền thay đổi).
- `@ScaledMetric(relativeTo: .body)`: Co giãn một giá trị `CGFloat` theo Dynamic Type khi thực sự cần kích thước tỷ lệ theo chữ.
- `#Preview` kết hợp `.preferredColorScheme(.dark)` và `.environment(\.dynamicTypeSize, .accessibility5)`: Kiểm tra trực quan ngay trong Xcode Canvas.

Minh họa ngắn (cho ứng dụng thời tiết `Weather`):
```swift
public enum WeatherTypography {
    public static let temperature: Font = .system(.largeTitle, design: .rounded, weight: .bold)
    public static let condition: Font = .headline
}

#Preview("Tối · AX5") {
    Text("28°C").font(WeatherTypography.temperature).monospacedDigit()
        .preferredColorScheme(.dark)
        .environment(\.dynamicTypeSize, .accessibility5)
}
```

## Lỗi hay gặp
1. **Dùng `.font(.system(size: 38))` cố định:** Bỏ qua hoàn toàn Dynamic Type — khi người dùng kéo cỡ chữ lên `accessibility5`, chữ vẫn đứng yên (vi phạm NFR-11). Luôn dùng `Font.TextStyle` hoặc `@ScaledMetric`.
2. **Dùng nguyên `#1B6B4A` cho chữ trên nền tối:** Xanh đậm `#1B6B4A` rất rõ trên nền trắng nhưng bị chìm trên nền đen `#000000` ở Dark Mode. Cần pha bản sáng hơn trong biến thể Dark của Color Set để đạt tương phản ≥ 4,5:1.
3. **Quên `bundle: .module` trong Swift Package:** Viết `Color("BrandPrimary")` sẽ tìm trong bundle của app chính thay vì package `DesignSystem`, dẫn đến màu trong suốt mà không báo lỗi biên dịch.
4. **Import trực tiếp `UIKit` không có `#if canImport(UIKit)`:** Làm `swift test --package-path Core` trên macOS (và CI) bị lỗi biên dịch.
5. **Khóa cứng `frame(height:)` trên dòng chữ:** Ở cỡ chữ thường trông đẹp nhưng lên `accessibility5` sẽ bị cắt mất chữ. Hãy dùng `minHeight:` hoặc để view tự co giãn theo nội dung.

## Đọc thêm
- [Apple HIG – Color](https://developer.apple.com/design/human-interface-guidelines/color)
- [Apple HIG – Typography & Dynamic Type](https://developer.apple.com/design/human-interface-guidelines/typography)
- [Apple Documentation – Bundling resources with a Swift package](https://developer.apple.com/documentation/xcode/bundling-resources-with-a-swift-package)
- [Apple HIG – Materials & Liquid Glass](https://developer.apple.com/design/human-interface-guidelines/materials)
