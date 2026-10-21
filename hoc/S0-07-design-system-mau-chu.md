# S0-07 · Design system: màu và chữ

## Vì sao cần
Một Design System chuẩn hóa trong Jetpack Compose giúp toàn bộ các màn hình chia sẻ chung bảng màu ngữ nghĩa, phông chữ và hình khối hình học. Điều này loại bỏ hoàn toàn việc hard-code màu sắc rải rác, tự động hóa chuyển đổi sáng/tối (Dark Mode) và hỗ trợ màu sắc cá nhân hóa (Dynamic Color trên Android 12+).

## Khái niệm chính
- **ColorScheme:** Tập hợp các vai trò màu sắc có ngữ nghĩa của Material 3 (primary, onPrimary, surface, onSurface, error, outline...) đảm bảo tỷ lệ tương phản chuẩn WCAG.
- **Màu hạt giống (Seed Color):** Màu gốc `#1B6B4A` (xanh ngọc lục bảo) qua thuật toán Material Color Utilities sinh ra 5 bảng màu tonal palettes và các cặp màu sáng/tối.
- **Typography:** Hệ thống thang phân cấp kiểu chữ gồm 15 kiểu (Display, Headline, Title, Body, Label ứng với Large, Medium, Small).
- **Phông chữ Roboto Flex:** Phông chữ dạng variable font, tối ưu hiển thị tiếng Việt có dấu và linh hoạt thay đổi độ dày (`FontWeight`).
- **Shapes:** Quy chuẩn bán kính bo góc các bề mặt (`RoundedCornerShape`) từ `extraSmall` (4.dp) đến `extraLarge` (28.dp).
- **Dynamic Color:** Trích xuất bảng màu hệ thống từ hình nền người dùng qua `dynamicLightColorScheme` / `dynamicDarkColorScheme` (từ Android 12 - API 31).

## So với cách cũ bạn quen

| Cũ (XML & View System) | Mới (Jetpack Compose Material 3) | Khác biệt chính |
| --- | --- | --- |
| `res/values/colors.xml` và `themes.xml` | `Color.kt` và `Theme.kt` (Kotlin code) | Kiểm tra kiểu tại thời điểm compile, không cần phân tích XML lúc runtime |
| `?attr/colorPrimary`, `?android:attr/textColorPrimary` | `MaterialTheme.colorScheme.primary` | Truy cập trực tiếp qua Compose Ambient / CompositionLocal |
| `res/values-night/themes.xml` | Khối điều kiện `if (darkTheme) DarkColorScheme else LightColorScheme` | Tự động recompose toàn bộ UI ngay khi trạng thái đổi |
| `Widget.Design.ShapeAppearance` | `MaterialTheme.shapes` | Định nghĩa bo góc bằng Kotlin `RoundedCornerShape(dp)` |

## API và cú pháp cần biết
- `lightColorScheme(...)`: Hàm tạo bộ màu giao diện sáng.
- `darkColorScheme(...)`: Hàm tạo bộ màu giao diện tối.
- `dynamicLightColorScheme(context)` / `dynamicDarkColorScheme(context)`: Tạo ColorScheme động theo theme Android 12+.
- `Typography(...)`: Định nghĩa kiểu chữ với các `TextStyle`.
- `Shapes(...)`: Cấu hình các cấp độ bo góc của MaterialTheme.

Đoạn code minh họa (Theme mẫu):
```kotlin
@Composable
fun DemoAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> DemoDarkColorScheme
        else -> DemoLightColorScheme
    }
    MaterialTheme(colorScheme = colorScheme, typography = DemoTypography, content = content)
}
```

## Lỗi hay gặp
1. **Hard-code màu sắc trong composable:** Dùng trực tiếp `Color(0xFF...)` thay vì `MaterialTheme.colorScheme.*`, làm vỡ giao diện khi người dùng chuyển sang Dark Mode.
2. **Không kiểm tra phiên bản Android khi gọi Dynamic Color:** Gọi `dynamicLightColorScheme` mà thiếu kiểm tra `Build.VERSION.SDK_INT >= Build.VERSION_CODES.S` gây crash trên Android 11 trở xuống.
3. **Màu chữ không tương phản với nền:** Không dùng đúng cặp `role` và `onRole` (ví dụ: vẽ chữ màu `primary` trên nền `surface` có thể bị chìm ở Dark Theme; luôn ưu tiên `onPrimary` trên nền `primary`, `onSurface` trên nền `surface`).
4. **Bỏ quên export namespace:** Không tổ chức gói rõ ràng trong `core:designsystem`, khiến các module feature khó tái sử dụng Theme.

## Đọc thêm
- [Material 3 Design Tokens & Theming](https://m3.material.io/styles/color/the-color-system/tokens)
- [Jetpack Compose Material 3 Theming Guide](https://developer.android.com/develop/ui/compose/designsystems/material3)
- [Material Theme Builder Tool](https://material-foundation.github.io/material-theme-builder/)
