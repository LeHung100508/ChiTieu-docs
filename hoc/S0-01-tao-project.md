# S0-01 · Tạo project

## Vì sao cần
Khởi tạo cấu trúc dự án Android hiện đại với AGP 9, Kotlin 2.x và Jetpack Compose ngay từ đầu. Điều này giúp dự án có nền tảng ổn định, tương thích với các công cụ build mới nhất và sẵn sàng cho các thư viện hiện đại mà không gặp rào cản di chuyển sau này.

## Khái niệm chính
- **AGP 9 & Kotlin 2.x**: Phiên bản Android Gradle Plugin mới nhất tối ưu hóa hiệu năng build và tận dụng trình biên dịch K2 của Kotlin.
- **Compose Compiler tích hợp**: Từ Kotlin 2.0, Compose Compiler được tích hợp trực tiếp qua JetBrains Kotlin Compose plugin thay vì thư viện compiler riêng lẻ.
- **Java 17 target**: Android Studio hiện đại và AGP yêu cầu tối thiểu JDK 17 cho môi trường biên dịch bytecode và chạy Gradle daemon.
- **targetSdk vs compileSdk**: `compileSdk` xác định bộ API SDK dùng khi build, `targetSdk` cam kết phiên bản hành vi hệ điều hành mà app đã được tối ưu và kiểm thử.
- **applicationId cố định**: Định danh duy nhất của ứng dụng trên Google Play và thiết bị Android, không thể thay đổi sau khi ứng dụng được phát hành.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| Groovy DSL (`build.gradle`) | Kotlin DSL (`build.gradle.kts`) | Type-safety, code completion và điều hướng trong IDE chính xác hơn |
| `androidx.compose.compiler` plugin riêng | `org.jetbrains.kotlin.plugin.compose` | Không cần canh khớp phiên bản Kotlin với phiên bản Compose Compiler |
| Java 8 / 11 | Java 17 | Tận dụng cú pháp Java mới và tương thích hoàn toàn với Gradle 8+/9+ |
| XML Layouts (`setContentView(R.layout...)`) | Jetpack Compose (`setContent { ... }`) | Khai báo UI hoàn toàn bằng Kotlin, không cần XML view binding |

## API và cú pháp cần biết
- `enableEdgeToEdge()`: Tự động cấu hình cửa sổ hiển thị tràn viền theo khuyến nghị Material 3.
- `setContent { ... }`: Thiết lập composable gốc gắn vào Activity.
- `compileOptions { ... }`: Cấu hình cấp độ tương thích mã nguồn Java cho Android module.

Ví dụ cấu hình Java 17 trong Gradle Kotlin DSL:
```kotlin
android {
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
}
```

## Lỗi hay gặp
- **Permission denied khi chạy gradlew**: Quên gán quyền thực thi cho script wrapper trên macOS/Linux. Cách tránh: chạy `chmod +x gradlew` và lưu thuộc tính file vào git.
- **Lệch phiên bản Java**: Đặt compileOptions là Java 11 trong khi môi trường Gradle chạy Java 17 hoặc ngược lại. Cách tránh: đồng bộ cấu hình Java 17 trên toàn bộ module.
- **Đổi applicationId sau khi code sâu**: Dẫn đến việc đổi đường dẫn package và gây khó khăn khi tích hợp dịch vụ bên thứ ba. Cách tránh: chốt ID cố định ngay từ bước đầu tiên.

## Đọc thêm
- [Android Gradle Plugin Release Notes](https://developer.android.com/build/releases/gradle-plugin)
- [Kotlin 2.0 and Compose Compiler Setup](https://developer.android.com/develop/ui/compose/compiler)
- [Meet Android 15 & API Levels](https://developer.android.com/about/versions/15)
