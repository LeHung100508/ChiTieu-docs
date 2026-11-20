# iS0-07 · Model parser và đọc bộ câu mẫu

## Vì sao cần
Bộ phân tích ngôn ngữ tự nhiên (Parser) là tính năng cốt lõi của ChiTieu, giúp người dùng ghi chép chi tiêu bằng một câu nói hay câu gõ tự nhiên (như *"phở bò 45k"*, *"tiền điện 850k hôm qua"*). Để phát triển bộ parser theo phương pháp TDD (Test-Driven Development) và đồng bộ 100% logic với phiên bản Android, ta cần dựng các kiểu dữ liệu nền tảng (`Money`, `ParsedTransaction`), đọc trọn vẹn bộ 200 câu mẫu dùng chung từ `docs/shared/parser/test_cases.json`, và thiết lập khung test có tham số (Parameterized Testing) trong Swift Testing.

## Khái niệm chính
- **Struct `Money`:** Đóng gói giá trị số tiền nguyên `Int64` (đơn vị đồng), tuân thủ `Sendable`, `Hashable`, `Codable`, `Comparable`, `ExpressibleByIntegerLiteral`, dùng custom `singleValueContainer` mã hóa ra số nguyên trần (VD: `45000`), ngăn chặn hoàn toàn việc nhầm lẫn đơn vị tiền hoặc dùng số thực trôi nổi (`Double`) gây sai số làm tròn.
- **`ParsedField<Value>` và `ParsedTransaction`:** Model chứa kết quả phân tích: loại giao dịch (`TransactionType`: chi tiêu, thu nhập, chuyển khoản), số tiền, danh mục (`categoryKey`), thời gian, ví, ví nguồn (`sourceWallet`), ví đích (`targetWallet`), ghi chú. Mỗi trường được bọc trong `ParsedField<Value>` mang giá trị kèm độ tin cậy (`confidence`) được kẹp chặt trong đoạn `0.0...1.0`.
- **Bộ câu mẫu dùng chung (Single Source of Truth):** File `docs/shared/parser/test_cases.json` chứa 200 câu mẫu thực tế kèm kết quả kỳ vọng, được chia sẻ giữa cả hai repo Android và iOS qua submodule `docs/`.
- **Đọc file ngoài Package trong Test bằng `#filePath`:** SwiftPM giới hạn các resource nội bộ trong thư mục target. Để test đọc được file tài liệu nằm ngoài package ở cả máy cá nhân lẫn môi trường CI, ta dùng `#filePath` để truy xuất đường dẫn tương đối ngược về gốc repo.
- **Parameterized Testing với `arguments: try ...` trong Swift Testing:** Cho phép chạy một hàm `@Test` lặp qua toàn bộ 200 câu mẫu một cách độc lập. Dùng `try` thay vì `try?` để fail sớm nếu file JSON hỏng hoặc thiếu.
- **Xử lý Known Issues trong TDD với `isIntermittent: true`:** Khác với `assumeTrue` trong JUnit (bỏ qua hoàn toàn), `withKnownIssue` vẫn thực thi test. Nếu thêm `isIntermittent: true`, Swift Testing cho phép một số ca test pass dần mà không báo lỗi "Known issue was not recorded", cực kỳ phù hợp cho quá trình TDD qua từng sprint.

## So với cách cũ bạn quen
| Android (Kotlin / JUnit 5) | iOS (Swift / Swift Testing) | Khác biệt chính |
| --- | --- | --- |
| `@JvmInline value class Money(val amount: Long)` | `public struct Money: Sendable, Hashable, Comparable` | Swift dùng `struct` gọn nhẹ; có thể định nghĩa phép toán `+`, `-`. |
| `@ParameterizedTest @MethodSource("testCases")` | `@Test(arguments: testSuite.testCases)` | Swift Testing tích hợp tham số kiểm thử trực tiếp vào macro `@Test`. |
| `java.time.Instant` / `ZonedDateTime` | `Date` kèm `ISO8601DateFormatter` | Swift nạp chuỗi ISO 8601 chuẩn qua `decoder.dateDecodingStrategy = .iso8601`. |
| `org.junit.jupiter.api.Assumptions.assumeTrue(...)` | `withKnownIssue(isIntermittent: true) { ... }` | `assumeTrue` **bỏ qua** test; `withKnownIssue` **chạy** test, ghi nhận lỗi đã biết và cần `isIntermittent: true` khi test pass dần trong TDD. |

## API và cú pháp cần biết

1. **Xác định đường dẫn file ngoài package bằng `#filePath`:**
```swift
let testFilePath = URL(fileURLWithPath: #filePath)
let repoRoot = testFilePath
    .deletingLastPathComponent() // Tests/ParserTests
    .deletingLastPathComponent() // Tests
    .deletingLastPathComponent() // Core
    .deletingLastPathComponent() // ChiTieu-ios (root)
let jsonURL = repoRoot.appendingPathComponent("docs/shared/parser/test_cases.json")
```

2. **Giải mã ISO 8601 chuẩn:**
```swift
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
let suite = try decoder.decode(TestSuite.self, from: data)
```

3. **Khung test có tham số và `withKnownIssue(isIntermittent: true)`:**
```swift
@Test("Phân tích 200 câu mẫu", arguments: try TestSuiteLoader.loadTestSuite().testCases)
func testParseAllCases(testCase: TestCase) async throws {
    let suite = try TestSuiteLoader.loadTestSuite()
    let results = TransactionParser.parse(
        testCase.input,
        referenceTime: suite.referenceTime,
        timeZone: suite.timeZone
    )
    withKnownIssue("Giải thuật parser đang được phát triển theo TDD (iS0-08 đến iS0-11)", isIntermittent: true) {
        #expect(results.count == testCase.expected.count)
    }
}
```

## Lỗi hay gặp
1. **Lệch múi giờ khi parse thời gian:** File JSON chứa mốc thời gian dạng `+07:00` (`Asia/Ho_Chi_Minh`). Nếu dùng `DateFormatter` mặc định của hệ thống mà máy runner CI ở múi giờ UTC, test so sánh ngày giờ sẽ bị lệch 7 tiếng.
2. **Crash đường dẫn file trên CI:** Dùng đường dẫn tuyệt đối dạng `/Users/username/...` thay vì tính tương đối từ `#filePath`. Trên máy ảo CI GitHub Actions đường dẫn người dùng hoàn toàn khác.
3. **Quên `Sendable` cho model test case:** Swift Testing chạy các ca test song song trên các thread khác nhau, vì vậy mảng truyền vào `arguments:` và struct `TestCase` bắt buộc phải là `Sendable`.
4. **Để test fail làm đỏ CI:** Ở iS0-07, chúng ta mới xây dựng khung model và test nạp dữ liệu, chưa viết giải thuật nhận diện. Nếu assert trực tiếp mà không bọc trong `withKnownIssue`, CI sẽ fail ngay lập tức.
5. **Gắn `@MainActor` vào Parser:** Parser là tác vụ tính toán thuần túy (CPU-bound) sẽ được gọi trong luồng phụ khi người dùng nhập văn bản. Không được gắn `@MainActor` vào các hàm của Parser.

## Đọc thêm
- [Swift Testing: Parameterized Testing](https://developer.apple.com/documentation/testing/parameterizedtesting)
- [Swift Testing: Known Issues](https://developer.apple.com/documentation/testing/definingexpectations#Known-issues)
- [Apple Documentation: ISO8601DateFormatter](https://developer.apple.com/documentation/foundation/iso8601dateformatter)
