# S0-02 · Version Catalog

## Vì sao cần
Quản lý tập trung toàn bộ phiên bản thư viện và plugin trong một tệp `libs.versions.toml`. Điều này giúp tránh phân mảnh phiên bản giữa các module đa dự án, hỗ trợ auto-complete khi khai báo dependency và đảm bảo việc nâng cấp thư viện diễn ra nhất quán và an toàn.

## Khái niệm chính
- **Bốn khối cấu hình chính**:
  - `[versions]`: Định nghĩa các biến số phiên bản tái sử dụng.
  - `[libraries]`: Khai báo các thư viện cụ thể theo tọa độ `group:name` và tham chiếu tới `version.ref`.
  - `[bundles]`: Gom nhóm các thư viện thường đi liền với nhau thành một alias duy nhất.
  - `[plugins]`: Khai báo Gradle plugin ID và version để áp dụng trong các file build script.
- **Quy tắc đặt tên Alias**: Dấu gạch ngang `-` trong TOML được ánh xạ thành dấu chấm `.` trong build script Kotlin DSL (ví dụ: `androidx-core-ktx` -> `libs.androidx.core.ktx`).
- **Compose BOM (Bill of Materials)**: Quản lý phiên bản tương thích giữa tất cả các thư viện Compose con mà không cần chỉ định phiên bản riêng cho từng artifact.
- **Tách biệt khai báo và sử dụng**: Version Catalog chỉ khai báo danh mục khả dụng, chưa tự động kéo dependency về cho đến khi được module cụ thể gọi trong `dependencies { ... }`.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| `ext.versions` trong root `build.gradle` | File TOML chuẩn hóa (`gradle/libs.versions.toml`) | Chuẩn hóa toàn hệ sinh thái Gradle, có schema rõ ràng |
| Chuỗi hardcode `implementation("com.google...:1.2.0")` | Type-safe accessors `implementation(libs.my.lib)` | Tránh lỗi gõ phím, IDE hỗ trợ gợi ý và refactor chuẩn xác |
| Quản lý version rải rác từng module | Khai báo một chỗ duy nhất trong catalog | Dễ dàng nâng cấp đồng loạt cho toàn bộ multi-module |

## API và cú pháp cần biết
- `version.ref`: Tham chiếu tới key tương ứng trong bảng `[versions]`.
- `platform(...)`: Dùng để chỉ định BOM khi implementation thư viện.
- `alias(libs.plugins.xxx) apply false`: Khai báo plugin ở root build script để tải sẵn plugin cho các module con.

Cú pháp mẫu trong `libs.versions.toml`:
```toml
[versions]
timber = "5.0.1"

[libraries]
timber = { group = "com.jakewharton.timber", name = "timber", version.ref = "timber" }

[bundles]
logging = ["timber"]
```

## Lỗi hay gặp
- **Đặt tên alias chứa dấu gạch dưới `_` thay vì `-`**: Gradle khuyến nghị dùng kebab-case (`my-lib-name`) để chuyển đổi thành camelCase hoặc dot-notation chuẩn trong Kotlin DSL.
- **Ghi nhầm tên group hoặc artifact**: Dẫn đến lỗi `Could not resolve...` khi sync. Cách tránh: tra cứu tọa độ maven chính xác trên Google Maven hoặc Maven Central.
- **Quên khai báo plugin ở root trước khi apply trong module**: Khiến Gradle không tìm thấy class plugin ở subproject.

## Đọc thêm
- [Android Developer: Migrate your build to version catalogs](https://developer.android.com/build/migrate-to-catalogs)
- [Gradle User Manual: Sharing dependency versions between projects](https://docs.gradle.org/current/userguide/platforms.html)
- [Compose BOM Mapping](https://developer.android.com/develop/ui/compose/bom/bom-mapping)
