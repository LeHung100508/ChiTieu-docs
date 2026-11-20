# iS0-08 · Parser: chuẩn hóa và tách từ (Normalization & Tokenization)

## Vì sao cần
Bộ phân tích ngôn ngữ tự nhiên (Parser) nhận văn bản nhập tự do từ người dùng (gõ phím hoặc giọng nói) với vô số biến thể: chữ hoa/thường tùy tiện, khoảng trắng thừa/tab, tiếng Việt có dấu hoặc không dấu, và đặc biệt là số viết dính liền với đơn vị tiền tệ (`45k`, `1tr2`, `2củ`). Chuẩn hóa (Normalization) và Tách từ (Tokenization) là tầng đầu tiên trong pipeline phân tích, đưa câu nhập về dạng chuẩn mực, tạo bản không dấu để so khớp từ điển, đồng thời bảo toàn chuỗi gốc để trích xuất ghi chú (`note`).

## Khái niệm chính
- **Chuẩn hóa chữ thường và khoảng trắng:** Đưa về chữ thường với `Locale(identifier: "vi_VN")`, thu gọn nhiều khoảng trắng liên tiếp hoặc ký tự tab thành một dấu cách đơn, loại bỏ khoảng trắng ở hai đầu chuỗi (`trimmingCharacters`).
- **Bỏ dấu tiếng Việt (Diacritics Removal) và bẫy `đ`/`Đ`:** Ký tự `đ` và `Đ` trong Unicode là phụ âm độc lập ("LATIN SMALL LETTER D WITH STROKE"), không phải `d` kết hợp combining diacritic mark. Các hàm strip diacritics chuẩn (`folding` hay `CFStringTransform`) đều giữ nguyên `đ`/`Đ`. Do đó, cần thay thế thủ công `đ` $\rightarrow$ `d`, `Đ` $\rightarrow$ `D` trước khi strip dấu.
- **Tách cụm số và đơn vị tiền tệ:** Chèn khoảng trắng giữa số và đơn vị (`45k` $\rightarrow$ `45 k`, `1tr2` $\rightarrow$ `1 tr 2`, `2củ` $\rightarrow$ `2 củ`, `500nghìn` $\rightarrow$ `500 nghìn`). Ngoại lệ quan trọng: giữ nguyên dấu chấm/phẩy phân cách thập phân hoặc hàng nghìn giữa hai chữ số (`1.5tr` $\rightarrow$ `1.5 tr`, `45.000` $\rightarrow$ `45.000`).
- **Ký tự xuống dòng `\R` và Grapheme Cluster:** Trong Swift, chuỗi `"\r\n"` (CRLF) được xem là **một `Character` duy nhất** (extended grapheme cluster). Regex dạng `\r?\n` sẽ không khớp được `"\r\n"`. Bắt buộc phải dùng `\R` để nhận diện chính xác mọi ký tự ngắt dòng.
- **Mô hình Token và NormalizedSentence:** Tầng này phân rã câu thành mảng `tokens`, lưu trữ `text`, `unaccented`, loại token và chỉ số vị trí `startIndex..<endIndex` đối chiếu trên chuỗi gốc `raw`.

## So với cách cũ bạn quen
| Android (Kotlin) | iOS (Swift 6) | Khác biệt chính |
| :--- | :--- | :--- |
| `lowercase(Locale.forLanguageTag("vi-VN"))` | `lowercased(with: Locale(identifier: "vi_VN"))` | Swift chỉ định locale trực tiếp qua identifier. |
| `Normalizer.normalize(..., Normalizer.Form.NFD)` | `folding(options: .diacriticInsensitive, locale: ...)` | Swift có sẵn API `folding` tiện dụng trong Foundation, không cần regex xóa combining marks. |
| `Regex("...")` runtime matching | Swift Regex literal `#/pattern/#` | Swift Regex được kiểm tra cú pháp và sinh kiểu `match.output` strongly-typed lúc compile. |
| `data class Token(...)` | `public struct Token: Sendable, Hashable` | Swift dùng `struct` thuần giá trị, thread-safe tuyệt đối giữa các background task. |

## API và cú pháp cần biết

1. **Bỏ dấu tiếng Việt chuẩn xác (xử lý `đ`/`Đ`):**
```swift
let lower = text.lowercased(with: Locale(identifier: "vi_VN"))
let replaced = lower.replacingOccurrences(of: "đ", with: "d")
let unaccented = replaced.folding(
    options: .diacriticInsensitive,
    locale: Locale(identifier: "vi_VN")
)
```

2. **Tách cụm số và chữ dính liền bằng Named Capture Groups:**
```swift
let numFollowedByUnit = #/(?<num>\d)(?<unit>[^\d\s.,])/#
let unitFollowedByNum = #/(?<unit>[^\d\s.,])(?<num>\d)/#

var result = text.replacing(numFollowedByUnit) { "\($0.output.num) \($0.output.unit)" }
result = result.replacing(unitFollowedByNum) { "\($0.output.unit) \($0.output.num)" }
```

3. **Phân rã Token với Regex literal đa dòng và `\R`:**
```swift
let tokenPattern =
    #/
    (?<newline>\R)
    | (?<number>\d+(?:[.,]\d+)*)
    | (?<word>\p{L}+)
    | (?<punct>[^\s\p{L}\d])
    /#

for match in raw.matches(of: tokenPattern) {
    let start = raw.distance(from: raw.startIndex, to: match.range.lowerBound)
    let end = raw.distance(from: raw.startIndex, to: match.range.upperBound)
    // match.output.newline, match.output.number, match.output.word...
}
```

## Lỗi hay gặp
1. **Dùng Lookbehind trong Swift Regex:** Cú pháp `(?<=...)` hiện chưa được Swift Regex hỗ trợ (`lookbehind is not currently supported`). Cần dùng capture groups thay thế.
2. **Quên xử lý chữ `đ`/`Đ`:** Dùng `folding(options: .diacriticInsensitive, ...)` mà không thay thế `đ`/`Đ` sẽ khiến `"đổ xăng"` biến thành `"đo xang"` thay vì `"do xang"`, làm trượt khớp từ điển danh mục ở iS0-11.
3. **Dùng `\r?\n` thay vì `\R`:** Vì `"\r\n"` là một grapheme cluster đơn lẻ, pattern `\r?\n` sẽ bỏ qua ký tự xuống dòng CRLF, làm dính các giao dịch tách dòng.
4. **Làm vỡ số thập phân hoặc số phân cách hàng nghìn:** Viết regex tách mọi ký tự không phải số sau số sẽ làm `"1.5tr"` bị chẻ thành `"1 . 5 tr"` hoặc `"45.000"` thành `"45 . 000"`. Phải loại trừ dấu `.` và `,` nằm giữa các chữ số.
5. **Gắn `@MainActor` vào các hàm Parser:** Parser là tác vụ tính toán thuần túy (CPU-bound) chạy trên luồng nền, các hàm và struct trong module này phải giữ `nonisolated`.

## Đọc thêm
- [Apple Developer: String.folding(options:locale:)](https://developer.apple.com/documentation/swift/string/folding(options:locale:))
- [Apple Developer: Swift Regex Overview](https://developer.apple.com/documentation/swift/regex)
- [Unicode Standard Annex #29: Unicode Text Segmentation (Grapheme Clusters)](https://unicode.org/reports/tr29/)
