# S0-05 · Chất lượng mã (Code Quality: Spotless & Detekt)

## Vì sao cần
Trong kiến trúc multi-module, sự thiếu nhất quán về định dạng code và các "code smell" tiềm ẩn (như hàm quá phức tạp, biến thừa, vi phạm quy ước Compose) rất dễ phát sinh. Spotless và Detekt đóng vai trò như hai người gác cổng tự động: Spotless chuẩn hóa phong cách format bằng `ktlint`, còn Detekt phân tích tĩnh (static analysis) để chặn lỗi kiến trúc và cấu trúc mã ngay khi build.

## Khái niệm chính
- **Spotless (Code Formatter)**: Công cụ định dạng mã nguồn tự động, tích hợp bộ engine `ktlint`. Hoạt động qua 2 task: `spotlessCheck` (kiểm tra đúng chuẩn không) và `spotlessApply` (tự động viết lại code theo chuẩn).
- **Detekt (Static Code Analysis)**: Bộ phân tích cú pháp tĩnh dành riêng cho Kotlin. Detekt phân tích AST (Abstract Syntax Tree) để phát hiện độ phức tạp, đặt tên sai chuẩn, hoặc rò rỉ trạng thái thông qua file luật `detekt.yml`.
- **Bộ luật Compose cho Detekt**: Mở rộng các quy tắc dành riêng cho Jetpack Compose (ví dụ: hàm Composable phải viết hoa chữ đầu, tham số `Modifier` phải đứng đầu danh sách tùy chọn, không truyền trực tiếp `MutableState` qua tham số).
- **Gắn vào lifecycle `check`**: Cả Spotless và Detekt đều liên kết với task `./gradlew check` tiêu chuẩn của Gradle. Khi chạy `check`, Gradle sẽ tự động kích hoạt kiểm tra toàn diện.
- **Convention Plugin quản lý tập trung**: Thay vì cấu hình lặp lại ở từng module, ta cấu hình Spotless ở root project hoặc qua convention plugin trong `build-logic`, áp dụng đồng bộ cho toàn bộ dự án.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| Dùng phím tắt IDE (Cmd+Option+L) để format | Spotless tự động chuẩn hóa bằng ktlint | Đảm bảo 100% nhất quán giữa mọi thiết bị, không phụ thuộc vào setting của IDE |
| Code review bằng mắt để soi khoảng trắng, đặt tên | Detekt và Spotless tự động kiểm tra trên CI | Giảm tải thời gian review, tập trung hoàn toàn vào logic nghiệp vụ |
| Tự ghi nhớ các best practice của Jetpack Compose | Detekt Compose rules | Báo lỗi hoặc cảnh báo ngay lập tức nếu Composable vi phạm quy tắc recomposition |

## API và cú pháp cần biết
- `spotless { kotlin { ktlint() } }`: Kích hoạt engine ktlint cho mã nguồn Kotlin trong dự án.
- `detekt { config.setFrom(files("$rootDir/config/detekt/detekt.yml")) }`: Chỉ định file luật phân tích cho Detekt.
- `detektPlugins(libs.findLibrary("...").get())`: Nạp thêm các bộ luật mở rộng (như Compose rules).

Ví dụ cấu hình Spotless cơ bản trong Gradle Kotlin DSL:
```kotlin
configure<SpotlessExtension> {
    kotlin {
        target("**/*.kt")
        targetExclude("**/build/**")
        ktlint("1.5.0")
    }
}
```

## Lỗi hay gặp
- **Quên chạy `spotlessApply` trước khi commit**: Khiến CI báo lỗi chỉ vì thừa dấu cách hoặc xuống dòng sai quy cách.
- **File luật `detekt.yml` quá khắt khe ban đầu**: Thiết lập ngưỡng độ phức tạp (cyclomatic complexity) quá thấp khiến code thông thường cũng bị chặn. Cần tinh chỉnh phù hợp với nhu cầu thực tế của dự án.
- **Xung đột phiên bản giữa Detekt và Kotlin**: Detekt phụ thuộc vào trình phân tích cú pháp của Kotlin; cần đảm bảo phiên bản Detekt hỗ trợ phiên bản Kotlin đang dùng (Kotlin 2.2.x).

## Đọc thêm
- [Spotless Gradle Plugin Docs](https://github.com/diffplug/spotless/tree/main/plugin-gradle)
- [Detekt Official Documentation](https://detekt.dev/)
- [Compose Rules for Detekt](https://mrmans0n.github.io/compose-rules/detekt/)
