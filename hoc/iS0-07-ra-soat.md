# Rà soát iS0-07 · Model parser và đọc bộ câu mẫu

Ngày rà soát: 2026-11-20 · Nhánh `task/iS0-07-model-parser` · Trạng thái: **Xong**

> Bản rà soát tự viết trước đó (ghi "Xong") đã được thay bằng bản này. Theo luật, mentor rà soát và đổi trạng thái.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ❌ **exit 1**, 4 lỗi `[Spacing] add 1 space` ở `ParserTestSuiteModel.swift:60-63` |
| `swiftlint lint --strict` | ✅ exit 0 |
| `cd Core && swift test` | ✅ 200 known issue, không có lỗi |
| `cd Core && xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) | ✅ TEST SUCCEEDED. Đọc file JSON qua `#filePath` chạy được cả trên simulator |
| Thống kê `test_cases.json` | 229 giao dịch kỳ vọng: 202 EXPENSE, 19 INCOME, **8 TRANSFER có `sourceWallet`/`targetWallet`**. 25 câu cho ra nhiều giao dịch, 2 câu cho ra rỗng |
| Thử Swift Testing trong một package nháp | `@Test(arguments: try load())`: khi `load()` ném lỗi, test **đỏ** với "Caught error". `withKnownIssue` mà không có issue nào xảy ra thì test **đỏ** với "Known issue was not recorded" |

## Bắt buộc sửa (2)

### 1. swift-format không qua, CI sẽ đỏ
- **Vị trí:** `Core/Tests/ParserTests/ParserTestSuiteModel.swift:60-63`.
- **Vấn đề:** các comment cuối dòng (`// Tests/ParserTests`…) chỉ cách code một khoảng trắng. swift-format yêu cầu hai khoảng trắng.
- **Vì sao:** bước lint trên CI chạy đúng lệnh này với `--strict`, nên PR sẽ bị chặn. Bản tự rà soát ghi "exit 0", nhưng khi chạy lại thì không đúng.
- **Hướng sửa:** sửa khoảng trắng bằng tay, hoặc chạy `swift format format -i` **chỉ cho file này**, rồi chạy lại lệnh lint chuẩn.

### 2. Model chưa đủ những gì task yêu cầu
- **Vị trí:** `Core/Sources/Model/ParsedTransaction.swift:18-25`.
- **Vấn đề:**
  - **Độ tin cậy:** task ghi "độ tin cậy 0–1 **cho từng trường**". Đây cũng là quyết định đã chốt ở mục 3 của luật: "mỗi trường có độ tin cậy từ 0 đến 1". Model hiện chỉ có một `confidence` chung cho cả giao dịch. Ngay file lý thuyết của bạn cũng ghi "cho từng trường", nên code và tài liệu đang lệch nhau.
  - **Chuyển khoản:** có `TransactionType.transfer`, nhưng không có ví nguồn và ví đích. Như vậy 8 giao dịch TRANSFER trong bộ mẫu (ví dụ `tc_117`: "chuyển 2tr từ VCB sang ví tiền mặt") không biểu diễn được bằng model này.
- **Vì sao:**
  - Từ iS0-08 đến iS0-11, từng bước của parser sẽ gán giá trị và độ tin cậy cho từng trường. Màn xác nhận sau này cũng dựa vào độ tin cậy của từng trường để tô sáng trường cần kiểm tra lại.
  - Sửa model khi đã có nhiều code dùng nó sẽ tốn công hơn nhiều so với sửa bây giờ.
- **Hướng sửa:**
  - Mỗi trường mang theo giá trị và độ tin cậy của nó. Có thể làm một kiểu generic nhỏ dạng "giá trị kèm độ tin cậy", rồi dùng cho số tiền, danh mục, thời gian, ví và ghi chú.
  - Thêm cách biểu diễn ví nguồn và ví đích cho chuyển khoản.
  - Cân nhắc kiểm tra độ tin cậy nằm trong đoạn từ 0 đến 1 ngay ở `init`.
  - Sửa xong thì cập nhật test `Codable` trong `ModelTests`.

## Nên sửa (4)

### 1. Khung test chưa so với kết quả kỳ vọng, và sẽ tự đỏ khi parser bắt đầu chạy
- **Vị trí:** `Core/Tests/ParserTests/ParserTests.swift:52-60`.
- **Vấn đề:**
  - Test chỉ kiểm `!results.isEmpty`, không so với `expected`.
  - `parse` được gọi với `referenceTime` mặc định là `Date()`, không phải `referenceTime` và `timezone` của bộ mẫu. Trong khi đó mọi `datetime` kỳ vọng đều tính từ mốc 2026-10-20 10:00 +07:00.
  - `withKnownIssue` sẽ báo đỏ ("Known issue was not recorded", mentor đã thử) ngay khi parser trả về bất kỳ kết quả nào, kể cả kết quả sai. Nghĩa là sang iS0-08 hoặc iS0-09, test sẽ đỏ hàng loạt vì lý do không liên quan.
  - 2 câu có `expected` rỗng thì kiểm `!isEmpty` là sai về logic.
- **Vì sao:** iS0-09 cần đo "đúng số tiền ở ≥ 95% câu mẫu" (Cổng 1). Khung test phải so sánh được từng trường thì mới đo được.
- **Hướng sửa:**
  - Truyền `referenceTime` và `TimeZone` của bộ mẫu vào `parse`.
  - So kết quả với `expected`: trước hết là số lượng giao dịch và từng trường.
  - Thiết kế cách đánh dấu "chưa hỗ trợ" sao cho khi một câu bắt đầu đúng thì test không đỏ. Ví dụ dùng tùy chọn `isIntermittent` của `withKnownIssue`, hoặc giữ một danh sách tag và trường đã hỗ trợ, mở rộng dần qua từng task.
  - Cân nhắc thêm một test tính tỉ lệ đúng của trường số tiền, để iS0-09 chỉ cần đổi ngưỡng.

### 2. `try?` nuốt lỗi khi nạp bộ mẫu
- **Vị trí:** `ParserTests.swift:52`, `arguments: (try? …) ?? []`.
- **Vì sao:** nếu nạp file lỗi (submodule chưa checkout, JSON đổi cấu trúc), test có tham số sẽ chạy 0 ca và vẫn xanh. Mentor đã thử: Swift Testing nhận `arguments: try …` và báo lỗi rõ ràng khi biểu thức ném lỗi.
- **Hướng sửa:** dùng `try` thay cho `try? … ?? []`.

### 3. Giải mã còn để ở dạng `String`
- **Vị trí:** `ParserTestSuiteModel.swift:19-28` và `:44-52`.
- **Vấn đề:**
  - `referenceTime`, `timezone`, `datetime` và `type` đều đang được giải mã thành `String`.
  - Riêng `referenceTime` được chuyển sang `Date` trong một test riêng. 229 giá trị `datetime` và mọi giá trị `type` thì chưa hề được kiểm.
- **Vì sao:** tiêu chí của task là "`referenceTime` và múi giờ được giải mã đúng". Nếu giải mã thẳng thành `Date`, `TimeZone` và `TransactionType`, thì một giá trị sai trong file JSON sẽ làm lỗi ngay khi nạp, chứ không phải khi so sánh. Test so sánh sau này cũng gọn hơn.
- **Hướng sửa:**
  - Đặt `dateDecodingStrategy` của `JSONDecoder` là `.iso8601`. Với múi giờ, dùng một `init(from:)` nhỏ, hoặc một property tính từ chuỗi đã giải mã.
  - Dùng lại `TransactionType` của `Model` cho trường `type`.

### 4. File lý thuyết có chỗ chưa đúng
- **Vị trí:** `hoc/iS0-07-parser-model-testcases.md`.
- **Vấn đề:**
  - Dòng "Khái niệm chính" ghi độ tin cậy cho từng trường, nhưng code chỉ có một giá trị chung (xem Bắt buộc 2).
  - Snippet 2: `formatter` có `.withFractionalSeconds` nhưng không được dùng, vì closure lại tạo `ISO8601DateFormatter()` mới. Nếu dùng `formatter` đó thì chuỗi `2026-10-20T10:00:00+07:00` (không có phần lẻ của giây) sẽ **không** parse được. Cách đơn giản hơn là `decoder.dateDecodingStrategy = .iso8601`.
  - Bảng so sánh ghi `assumeTrue` tương đương `withKnownIssue`. Thật ra hai cái khác nhau: `assumeTrue` **bỏ qua** test, còn `withKnownIssue` **chạy** test, ghi nhận lỗi đã biết, và báo đỏ khi lỗi đó không còn xảy ra. Cần ghi đúng điểm này, vì nó liên quan trực tiếp tới Nên sửa 1.

## Gợi ý thêm để học (4)
1. **`Money` được mã hóa thành `{"amount": 45000}`**, do `Codable` tự sinh. Nếu muốn mã hóa thành số trần `45000` giống trong JSON mẫu và file sao lưu sau này, hãy tự viết `encode`/`init(from:)` dùng `singleValueContainer`. Cũng nên cân nhắc `ExpressibleByIntegerLiteral` để viết `let m: Money = 45_000`.
2. **Phép `+` và `-` của `Money`** sẽ crash khi tràn số `Int64`. Với số tiền thật thì khó xảy ra, nhưng nên biết đó là hành vi mặc định của Swift (khác với Kotlin, vốn âm thầm quay vòng giá trị).
3. **`referenceTime: Date = Date()`** làm mặc định trong một API thuần sẽ khiến kết quả phụ thuộc vào thời điểm chạy. Nên bắt buộc truyền vào, hoặc để ViewModel truyền. Như vậy test luôn xác định, và lỗi quên truyền như ở Nên sửa 1 không xảy ra được.
4. **Dọn nhỏ:**
   - Header của `Parser.swift`, `ModelTests.swift` và `ParserTests.swift` vẫn ghi `File.swift`.
   - Comment trong `Package.swift` vẫn ghi màu thương hiệu `#1B6B4A`. iOS đã chốt `#1E7A4E`.

## Điểm làm tốt
- Đọc file ngoài package bằng `#filePath`: không chép file, chạy được trên macOS, trên simulator iOS và trên CI (đã checkout submodule). File lý thuyết có ghi lý do chọn cách này.
- `Parser` không đặt `defaultIsolation`, nên `TransactionParser.parse` là nonisolated và chạy được ngoài luồng chính. `parse` trả về mảng, đúng với 25 câu có nhiều giao dịch.
- `TestCase` có `CustomTestStringConvertible`, nên báo cáo hiển thị dạng `tc_001: phở bò 45k`, rất dễ dò từng câu. Test kiểm tra `referenceTime` theo lịch có múi giờ `Asia/Ho_Chi_Minh`, nên không phụ thuộc múi giờ của máy chạy.

## Kết luận ban đầu
Chưa đóng được task. Cần sửa Bắt buộc 1–2 và Nên sửa 1–4, hoặc ghi `hoãn <mục>`.

---

## Rà lại lần 1 (2026-11-20)

> Bản "Rà lại lần 1" tự viết (kết luận "Đóng được") đã được thay bằng bản này.

**Lệnh đã chạy (mentor tự chạy):**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `cd Core && swift test` → ✅, 198 known issue, 2 ca qua (là 2 câu có `expected` rỗng).
- `xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) → ✅.
- App build → ✅.

**Đã đạt:**
- **Bắt buộc 1:** ✅ Lint sạch.
- **Bắt buộc 2:** ✅
  - `ParsedField<Value>` mang giá trị kèm độ tin cậy, được kẹp trong đoạn 0–1, và có test cho việc kẹp.
  - Có `sourceWallet` và `targetWallet`.
  - Test `Codable` dùng một giao dịch chuyển khoản.
- **Nên sửa 2:** ✅ Đã dùng `arguments: try …`.
- **Nên sửa 3:** ✅ `referenceTime` và `datetime` giải mã thành `Date` (`.iso8601`), `type` giải mã thành `TransactionType`.
- **Nên sửa 4:** ✅ File lý thuyết đã sửa phần độ tin cậy, `.iso8601` và sự khác nhau giữa `assumeTrue` và `withKnownIssue`.
- **Gợi ý 1:** ✅ `Money` mã hóa thành số trần, có `ExpressibleByIntegerLiteral`.

**Còn lại:**
- **Nên sửa 1 mới đạt một phần.**
  - Đã truyền `referenceTime` và `TimeZone`, đã dùng `isIntermittent`. Nhưng test chỉ so **số lượng** giao dịch, chưa so từng trường.
  - Vì toàn bộ phép so sánh nằm trong `withKnownIssue(isIntermittent: true)`, test này **không bao giờ đỏ được**. Parser có làm hỏng một câu đang đúng thì test cũng không báo.
  - Mentor chấp nhận để phần này sang iS0-09, task mà tiêu chí chính là "đúng số tiền ở ≥ 95% câu mẫu". Điều kiện là bạn ghi `hoãn`, để mục này nằm trong Nợ kỹ thuật và được làm trước iS0-09. Phần cần làm lúc đó: so từng trường, tính tỉ lệ đúng, và đưa các câu đã đúng ra khỏi `withKnownIssue` để chặn hồi quy.

**Gợi ý nhỏ (không chặn):**
- `type` chưa có độ tin cậy. Khi câu không có từ nào chỉ loại, parser sẽ mặc định là chi tiêu, nên có độ tin cậy thấp hơn. Cân nhắc đổi thành `ParsedField<TransactionType>`.
- Test có tham số gọi `loadTestSuite()` 200 lần, tức là đọc và giải mã file khoảng 100 KB 200 lần. Có thể nạp một lần vào một hằng `static let`.
- `parsedTimeZone` quay về `.current` khi identifier sai, nên lỗi bị che. Trong test, dùng `#require(TimeZone(identifier:))` sẽ an toàn hơn.
- Còn sót code giữ chỗ và header cũ:
  - `Model.swift` vẫn còn `ModelModule` từ iS0-02.
  - Header `File.swift` trong `Parser.swift`, `ModelTests.swift`, `ParserTests.swift`.
  - Comment màu `#1B6B4A` trong `Package.swift`.

**Kết luận:** đóng được task khi người dùng ghi `hoãn` cho phần còn lại của Nên sửa 1, hoặc sửa nó luôn.

## Đóng task (2026-11-20)
Người dùng ghi `hoãn` phần còn lại của Nên sửa 1 (so sánh từng trường, tỉ lệ đúng, chặn hồi quy). Phần này đã được thêm vào Nợ kỹ thuật, phải xử lý trước iS0-09. Không còn mục Bắt buộc, các tiêu chí xong đều đạt.

**Kết luận:** Đóng được task.
