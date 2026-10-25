# S0-11 · Parser: số tiền (FR-02)

## Vì sao cần
Số tiền là trường dữ liệu cốt lõi nhất của một giao dịch tài chính. Trong ngôn ngữ tự nhiên hàng ngày của người Việt, số tiền được thể hiện bằng vô số cách nói tắt, tiếng lóng và quy ước dân gian: `45k`, `1tr2`, `1,5tr`, `2 củ`, `5 lít`, `năm chục`, `nửa củ`, `hai trăm rưỡi`.
Module `AmountParser` có nhiệm vụ quét danh sách `Token` sinh ra từ tầng chuẩn hóa S0-10, nhận diện chính xác giá trị số tiền (kiểu `Long`, đơn vị đồng VND), đồng thời xác định phạm vi token tiêu thụ để các bước sau không gom nhầm số tiền vào ghi chú (`note`).

## Khái niệm chính
- **Đơn vị chuẩn:**
  - Hàng nghìn: `k`, `nghìn`, `ngàn`, `đ`, `vnd` ($\times 1.000$).
  - Hàng triệu: `tr`, `trieu`, `triệu`, `m` ($\times 1.000.000$).
- **Tiếng lóng người Việt:**
  - `củ`: tương đương triệu ($\times 1.000.000$). Ví dụ: `2 củ` = 2.000.000 đ, `nửa củ` = 500.000 đ.
  - `lít`, `loét`: tương đương trăm nghìn ($\times 100.000$). Ví dụ: `5 lít` = 500.000 đ, `3 loét` = 300.000 đ.
  - `chục`: đi sau từ chỉ số, tương đương chục nghìn ($\times 10.000$). Ví dụ: `năm chục` = 50.000 đ, `ba chục` = 30.000 đ.
- **Cấu trúc số xen kẽ (Compound Amount):**
  - `<số> <đơn vị triệu/củ> <số>`: `1tr2` = 1.200.000 đ, `1 củ 2` = 1.200.000 đ (số phụ sau triệu có trọng số $100.000$).
  - `<số> k <số>`: `85k5` = 85.500 đ, `25k5` = 25.500 đ (số phụ sau k có trọng số $100$).
  - `<số/cụm> rưỡi`: `1 củ rưỡi` = 1.500.000 đ, `1 lít rưỡi` = 150.000 đ, `hai trăm rưỡi` = 250.000 đ.
- **Số định dạng chuẩn:**
  - Phân cách hàng nghìn: `45.000`, `1.250.000`, `350000`.
  - Số thập phân kèm đơn vị: `1.5tr`, `1,5tr` = 1.500.000 đ.
- **Từ chỉ số bằng chữ tiếng Việt:**
  - `nửa` (0.5), `một` (1), `hai` (2), `ba` (3), `bốn` (4), `năm` (5), `sáu` (6), `bảy` (7), `tám` (8), `chín` (9), `mười` (10), `trăm` (100).
- **Kết quả phân tích số tiền (`ParsedAmount`):**
  - `value: Long`: giá trị tiền quy đổi ra đồng.
  - `startIndex: Int`, `endIndex: Int`: vị trí ký tự trên chuỗi `raw`.
  - `tokenStartIndex: Int`, `tokenEndIndex: Int`: chỉ số token tương ứng trong danh sách token để đánh dấu đã tiêu thụ.

## So với cách cũ bạn quen
| Cách cũ (Regex thô trên toàn chuỗi) | Cách mới (Quét luồng Token có cấu trúc) | Khác biệt chính |
| :--- | :--- | :--- |
| Dùng Regex tìm trực tiếp trên chuỗi thô, dễ nuốt nhầm số nhà (`123 Lê Lợi`) hoặc số lượng (`2 quả`) | Duyệt qua danh sách `Token` đã phân loại `NUMBER`, `WORD` từ S0-10 | Phân biệt được ngữ cảnh và đơn vị tiền tệ đi kèm rõ ràng |
| Khó gom các cụm từ cách nhau bởi dấu cách (`1 củ 2`, `ba trăm rưỡi`) | Khớp mẫu (Pattern Matching) trên cửa sổ token liên tiếp | Nhận diện được cấu trúc đa từ tự nhiên |
| Tính toán qua kiểu `Double`, dễ sai số do biểu diễn dấu phẩy động | Tính toán trực tiếp bằng `Long` hoặc `BigDecimal` nguyên vẹn | Bảo toàn 100% tính chính xác của dữ liệu tài chính (NFR-06) |

## API và cú pháp cần biết
- `tokens.getOrNull(index)`: Truy cập an toàn phần tử theo chỉ số mà không lo `IndexOutOfBoundsException`.
- Xây dựng hàm parser trả về đối tượng kết quả hoặc `null`:
```kotlin
fun tryParseNumberWithUnit(tokens: List<Token>, index: Int): MatchResult? {
    val numToken = tokens.getOrNull(index) ?: return null
    val unitToken = tokens.getOrNull(index + 1) ?: return null
    if (numToken.type == TokenType.NUMBER && unitToken.type == TokenType.WORD) {
        val multiplier = when (unitToken.text) {
            "k", "nghin", "ngan" -> 1_000L
            "tr", "trieu", "cu" -> 1_000_000L
            "lit", "loet" -> 100_000L
            else -> return null
        }
        val amount = numToken.text.toLongOrNull()?.times(multiplier) ?: return null
        return MatchResult(amount, consumedCount = 2)
    }
    return null
}
```

## Lỗi hay gặp
1. **Sai số dấu phẩy động:** Dùng `1.5 * 1_000_000` với kiểu `Double` có thể gây sai lệch phần bù. Nên chuẩn hóa `1.5` thành `15` rồi nhân chia theo số nguyên.
2. **Quy tắc số phụ sau triệu và nghìn:** Trong `1tr2`, số `2` là 200 nghìn ($2 \times 100.000$). Nhưng trong `85k5`, số `5` là 5 trăm ($5 \times 100$).
3. **Tiêu thụ thiếu token:** Trong cụm `1 củ 2` hoặc `1 lít rưỡi`, nếu chỉ tiêu thụ token đầu thì token sau sẽ bị rớt lại thành ghi chú rác.
4. **Bẫy đơn vị trùng lặp:** Cần cẩn thận với câu có từ chỉ số lượng hoặc danh từ như `3 lít dầu` (ở đây `lít` có thể là lít dầu chứ không phải 100k, nhưng trong ngữ cảnh tiền bạc thông thường thì `lít` đứng sau số là 100k).
5. **Số đứng trần có dấu chấm:** `45.000` phải được nhận diện là 45 nghìn, không phải 45 đồng.

## Đọc thêm
- [Kotlin Standard Library: Long](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-long/)
- [Regex in Kotlin](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-regex/)
- [ChiTieu Parser Test Cases Specification](file:///Users/lehung/Developer/android-projects/ChiTieu/docs/shared/parser/README.md)
