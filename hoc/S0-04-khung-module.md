# S0-04 · Khung module (Modularization)

## Vì sao cần
Chia nhỏ ứng dụng monolith thành nhiều module độc lập theo tính năng (`feature`) và tầng lõi (`core`). Kiến trúc này giúp tăng tốc độ build nhờ Gradle biên dịch song song, cô lập phạm vi trách nhiệm của từng phần code và ngăn chặn triệt để việc phụ thuộc chéo giữa các màn hình.

## Khái niệm chính
- **Cấu trúc 2 tầng chính**:
  - `core:*`: Các khối xây dựng dùng chung cho toàn bộ app (data, database, model, parser, designsystem, ui, testing).
  - `feature:*`: Mỗi luồng nghiệp vụ hoặc màn hình chính là một module độc lập (onboarding, quickadd, transactions, budget, stats, settings).
- **Quy tắc ranh giới vàng**: Các module `feature` **không bao giờ phụ thuộc trực tiếp vào nhau** (`feature` $\rightarrow$ `feature` bị cấm hoàn toàn). Mọi điều hướng hoặc chia sẻ dữ liệu giữa các feature được điều phối bởi module `:app` thông qua Navigation 3 hoặc qua tầng `core`.
- **Phân tách `core:parser` thuần JVM**: Module `core:parser` áp dụng plugin `chitieu.jvm.library`, hoàn toàn không dính dáng tới Android SDK để chạy unit test siêu tốc trên JVM.
- **Module tích hợp `:app`**: Đóng vai trò orchestrator, phụ thuộc vào tất cả các feature để ghép nối đồ thị Navigation và dựng `MainActivity`.
- **Module `:benchmarks`**: Dành riêng cho kiểm thử hiệu năng khởi động (Macrobenchmark) và sinh Baseline Profile.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| Monolith 1 module (`:app` chứa tất cả) | Multi-module theo `core` và `feature` | Tận dụng build cache và biên dịch song song của Gradle |
| Phân chia package theo tầng (`com.app.ui`, `com.app.data`) | Module hóa theo tính năng kết hợp layer ở `core` | Tránh vô tình gọi class của màn hình khác, ép buộc kiến trúc UDF |
| Build lại toàn bộ khi sửa 1 dòng code | Chỉ biên dịch lại module bị thay đổi | Rút ngắn chu kỳ feedback khi lập trình và chạy test |

## API và cú pháp cần biết
- `include(":core:model")`: Khai báo module con trong `settings.gradle.kts`.
- `implementation(projects.core.model)` hoặc `implementation(project(":core:model"))`: Liên kết dependency giữa các module trong dự án.

Ví dụ khai báo dependency nội bộ trong Kotlin DSL:
```kotlin
dependencies {
    implementation(project(":core:model"))
    implementation(project(":core:designsystem"))
}
```

## Lỗi hay gặp
- **Circular Dependency (Phụ thuộc vòng tròn)**: Module A gọi B và B lại gọi A khiến Gradle báo lỗi đồ thị chu trình. Cách tránh: chuyển dữ liệu dùng chung xuống `core:model`.
- **Feature gọi Feature**: Màn hình `transactions` gọi trực tiếp ViewModel của `settings`. Cách tránh: đẩy event lên tầng Navigation ở `:app`.
- **Áp dụng sai plugin**: Đặt `chitieu.android.library` cho `core:parser` khiến nó bị gắn chặt vào Android SDK thay vì pure JVM.

## Đọc thêm
- [Android Developer: Guide to Android app modularization](https://developer.android.com/topic/modularization)
- [Now in Android Modularization Architecture](https://github.com/android/nowinandroid/blob/main/docs/ModularizationArchitecture.md)
- [Dependency Injection in Multi-module Projects](https://developer.android.com/training/dependency-injection/hilt-multi-module)
