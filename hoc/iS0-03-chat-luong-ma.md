# iS0-03 · Chất lượng mã (Code Quality: swift-format & SwiftLint)

> Khái niệm về phân tích tĩnh và linter đã có ở [S0-05-chat-luong-ma.md](S0-05-chat-luong-ma.md). File này tập trung vào bộ công cụ tương ứng của hệ sinh thái iOS/Swift.

## Vì sao cần
Trong kiến trúc multi-package (project Xcode kết hợp `Core` và `Features`), sự thiếu nhất quán về định dạng và các "code smell" (force unwrap `!`, closure giữ retain cycle, hàm quá dài, biến thừa) rất dễ phát sinh khi nhiều phần code phát triển song song.

Bộ đôi `swift-format` và `SwiftLint` đóng vai trò hai người gác cổng tự động:
- `swift-format`: Chuẩn hóa phong cách layout mã nguồn (thụt dòng, khoảng trắng, xuống dòng).
- `SwiftLint`: Phân tích cú pháp tĩnh (static analysis) để ép tuân thủ quy ước ngữ nghĩa và an toàn bộ nhớ.

## Khái niệm chính
- **swift-format (Apple Official Formatter)**: Công cụ định dạng mã nguồn chính thức được Apple tích hợp sẵn trong Swift toolchain (`swift format`). Công cụ đọc cú pháp cây SwiftSyntax để kiểm tra (`lint`) hoặc tự động sửa (`format`). Cấu hình qua `.swift-format` (dạng JSON).
- **SwiftLint (Static Code Analysis / Linter)**: Công cụ phân tích mã nguồn tĩnh phổ biến nhất cho Swift (từ Realm). SwiftLint chạy phần lớn rule dựa trên SwiftSyntax (và SourceKit khi cần phân tích sâu hơn hoặc `swiftlint analyze`) để bắt lỗi quy ước lập trình, cảnh báo rủi ro an toàn luồng, force unwrap và độ phức tạp mã. Cấu hình qua `.swiftlint.yml`.
- **Phân định ranh giới rõ ràng**:
  - `swift-format` phụ trách **hình thức / layout**: khoảng cách dòng, thụt lề (indent), độ dài dòng.
  - `SwiftLint` phụ trách **ngữ nghĩa / an toàn**: cấm force unwrap, kiểm tra đặt tên kiểu chữ lạc đà (camelCase), cấm print trong code production, rà soát capture list.
  - Tắt các rule format về layout trong SwiftLint để tránh xung đột giữa 2 công cụ.
- **Cố định phiên bản (Version Pinning)**: Cần cố định một phiên bản SwiftLint xác định để kết quả lint hoàn toàn nhất quán giữa máy lập trình cá nhân và môi trường CI (GitHub Actions ở iS0-04).
- **Quy tắc rà soát không sửa tự động (Mục 13.7 của Luật)**: Agent chỉ chạy lệnh lint để kiểm tra, tuyệt đối không tự ý chạy lệnh sửa đè file (`swift format --in-place` hay `swiftlint --fix`). Người dùng là người tự quyết định và chạy format.

## So với cách cũ bạn quen
| Cũ (Android / Kotlin) | Mới (iOS / Swift) | Khác biệt chính |
| --- | --- | --- |
| Spotless (engine `ktlint`) | `swift-format` (Apple SwiftSyntax) | Cả hai đều chuẩn hóa format, nhưng `swift-format` là công cụ chính chủ của Apple |
| Detekt (`detekt.yml`) | `SwiftLint` (`.swiftlint.yml`) | Cùng phân tích tĩnh AST; SwiftLint tích hợp sâu với SourceKit của Xcode |
| Gắn vào `./gradlew check` | Chạy lệnh CLI hoặc chạy qua Xcode Build Phase / CI Actions | iOS không có Gradle lifecycle chung; ta quản lý qua CLI và CI workflow |
| `spotlessCheck` / `spotlessApply` | `swift format lint` / `swift format --in-place` | Bạn tự chạy format trước khi commit |

## API và cú pháp cần biết
- `swift format lint --recursive --strict ChiTieu Core Features`: Kiểm tra định dạng đúng phạm vi với cờ `--strict` (báo lỗi trả exit code khác 0 nếu có vi phạm).
- `swift format format --in-place --recursive ChiTieu Core Features`: Tự động format lại code theo cấu hình `.swift-format`.
- `swiftlint lint --strict`: Quét và báo cáo tất cả vi phạm theo cấu hình `.swiftlint.yml`, trả lỗi nếu có bất kỳ warning hoặc lệch phiên bản `swiftlint_version`.
- Cấu hình loại trừ và cố định phiên bản trong `.swiftlint.yml`:
```yaml
swiftlint_version: 0.65.1

included:
  - ChiTieu
  - Core
  - Features
excluded:
  - "**/DerivedData"
  - "**/.build"
  - "docs"
  - "luyen-tap"
disabled_rules:
  - trailing_comma       # Giao cho swift-format quản lý
  - line_length          # Giao cho swift-format quản lý
  - trailing_whitespace  # Giao cho swift-format quản lý
opt_in_rules:
  - force_unwrapping     # Cấm force unwrap (!) để đảm bảo an toàn crash
```

## Lỗi hay gặp
- **Xung đột quy tắc giữa 2 công cụ**: Cả `swift-format` và `SwiftLint` đều có quy tắc về khoảng trắng hoặc độ dài dòng nhưng thiết lập thông số khác nhau, dẫn đến sửa công cụ này thì công cụ kia báo lỗi. Cách tránh: tắt các quy tắc layout bên `SwiftLint` và giao toàn quyền layout cho `swift-format`.
- **Quên loại trừ thư mục `.build` và `DerivedData`**: Khiến linter quét nhầm hàng nghìn file build tạm sinh ra, gây treo máy hoặc báo lỗi giả.
- **Cài đặt SwiftLint khác phiên bản**: Máy phát triển dùng bản mới, CI dùng bản cũ (hoặc ngược lại) khiến code ở máy sạch nhưng CI lại đỏ. Cách tránh: cố định bằng key `swiftlint_version: 0.65.1` trong `.swiftlint.yml` (chạy kèm `--strict` sẽ chặn lệch phiên bản) và trên CI tải đúng release binary `portable_swiftlint.zip` từ GitHub.
- **Bật quá nhiều opt-in rule khắt khe ngay từ đầu**: Làm việc code tính năng bị chậm vì phải sửa những lỗi tiểu tiết không quan trọng. Nên bắt đầu từ các rule cốt lõi nhất.

## Đọc thêm
- [Apple swift-format Repository](https://github.com/swiftlang/swift-format)
- [Realm SwiftLint Documentation](https://realm.github.io/SwiftLint/)
- [SwiftLint Rule Directory](https://realm.github.io/SwiftLint/rule-directory.html)
- [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/)
