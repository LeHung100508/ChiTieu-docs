# S0-03 · Convention plugins

## Vì sao cần
Trong kiến trúc multi-module, các module thường lặp lại cấu hình giống nhau (compileSdk, minSdk, Java 17, Compose, Hilt, Room). Convention plugins đóng gói các cấu hình chuẩn mực này thành các Gradle plugin nội bộ, giúp các module con chỉ cần khai báo 1 dòng plugin duy nhất, loại bỏ hoàn toàn việc sao chép cấu hình.

## Khái niệm chính
- **Composite build (`build-logic`)**: Một Gradle build độc lập nằm trong dự án, được kết nối với build chính qua `pluginManagement { includeBuild("build-logic") }`.
- **Chia sẻ Version Catalog**: `build-logic` có thể tái sử dụng tệp `gradle/libs.versions.toml` của dự án chính thông qua cấu hình `versionCatalogs.create("libs")`.
- **Plugin tái sử dụng (`Plugin<Project>`)**: Mỗi convention plugin là một class Kotlin cài đặt interface `Plugin<Project>` (hoặc Precompiled Script Plugin), áp dụng các plugin nền tảng và thiết lập thông số mặc định.
- **Tách biệt mối quan tâm (SoC)**:
  - `android.application`: Cấu hình cho module chạy app.
  - `android.library`: Cấu hình chung cho Android library (compileSdk, minSdk, Java 17).
  - `android.library.compose`: Bật Compose, gán compose compiler và runtime.
  - `android.hilt`: Bật KSP và Hilt dependency injection.
  - `android.room`: Cấu hình Room database compiler và đường dẫn schema.
  - `jvm.library`: Module Kotlin thuần (dành riêng cho `core:parser` không dính dáng tới Android framework).

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| `allprojects { ... }` hoặc `subprojects { ... }` | Convention plugins trong `build-logic` | Tránh coupling ngầm, tuân thủ Configuration Cache và isolated projects của Gradle hiện đại |
| File script dùng chung `apply from: "../common.gradle"` | Custom Gradle plugins có type-safe API | Được IDE hỗ trợ auto-complete, phát hiện lỗi biên dịch script ngay khi gõ |
| Copy-paste block `android { ... }` qua từng module | Chỉ cần `alias(libs.plugins.chinhanh.android.library)` | Tinh gọn, sửa một chỗ tự động cập nhật toàn bộ mọi module |

## API và cú pháp cần biết
- `includeBuild("build-logic")`: Khai báo liên kết build composite trong `settings.gradle.kts`.
- `plugins { `kotlin-dsl` }`: Bật tính năng viết Gradle plugin bằng Kotlin DSL trong `build-logic`.
- `extensions.configure<LibraryExtension> { ... }`: Cấu hình khối `android { ... }` của AGP từ bên trong plugin.

Ví dụ khung một convention plugin Kotlin thuần:
```kotlin
class SampleLibraryConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            with(pluginManager) {
                apply("org.jetbrains.kotlin.jvm")
            }
        }
    }
}
```

## Lỗi hay gặp
- **Quên `includeBuild("build-logic")` ở root `settings.gradle.kts`**: Dẫn đến Gradle không tìm thấy plugin ID khi module con áp dụng.
- **Không chia sẻ `libs.versions.toml` vào `build-logic`**: Khiến `build-logic` phải hardcode phiên bản AGP hoặc Kotlin thay vì dùng Version Catalog.
- **Xung đột phiên bản AGP / KSP với AGP 9**: Một số thư viện cũ chưa tương thích AGP 9; nếu gặp lỗi không tương thích, chuyển về AGP 8.x theo quyết định kỹ thuật của dự án.

## Đọc thêm
- [Android Developer: Best practices for Gradle builds in modular apps](https://developer.android.com/build/configure-modular-builds)
- [Now in Android (Google Architecture Sample - build-logic)](https://github.com/android/nowinandroid/tree/main/build-logic)
- [Gradle Docs: Composing builds](https://docs.gradle.org/current/userguide/composite_builds.html)
