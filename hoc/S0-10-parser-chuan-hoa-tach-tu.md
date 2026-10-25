# S0-10 · Parser: chuẩn hóa và tách từ (Normalization & Tokenization)

## Vì sao cần
Bộ phân tích ngôn ngữ tự nhiên (Parser) trong ChiTieu nhận đầu vào là các câu gõ hoặc giọng nói tự do của người dùng. Trong thực tế, dữ liệu này có vô vàn biến thể:
- Viết hoa tùy tiện: `"Phở Bò 45K"`, `"ĂN TRƯA 50k"`.
- Dính số và đơn vị: `"45k"`, `"1tr2"`, `"2củ"`, `"5lít"`, `"50nghìn"`.
- Khoảng trắng thừa, ký tự đặc biệt, dấu câu đặt ngẫu nhiên: `"bún bò  ,  45k ."`.
- Tiếng Việt có dấu hoặc gõ không dấu: `"bún bò"` vs `"bun bo"`.

Nếu để các bước phân tích sau (bóc tách số tiền S0-11, thời gian S0-12, danh mục S0-13) tự xử lý chuỗi thô, mã nguồn sẽ bị trùng lặp, cồng kềnh và dễ phát sinh lỗi biên. **Chuẩn hóa (Normalization) và Tách từ (Tokenization)** là tầng đầu tiên (Tầng 1) trong Pipeline: nó đưa câu nhập về dạng chuẩn mực, tạo bản không dấu để so khớp từ khóa, đồng thời giữ nguyên bản gốc để trích xuất trường ghi chú (`note`).

---

## Khái niệm chính

### 1. Chuẩn hóa chuỗi (Normalization)
Một câu đầu vào cần được biến đổi qua các bước:
1. **Chuyển chữ thường (Lowercase):** Dùng `lowercase(Locale.forLanguageTag("vi-VN"))` để không phụ thuộc cấu hình máy.
2. **Chuẩn hóa khoảng trắng & dấu câu:** Gộp nhiều dấu cách liên tiếp thành 1 dấu cách, chuẩn hóa dấu ngắt khoản (dấu phẩy `,`, dấu chấm phẩy `;`, ký tự xuống dòng `\n`).
3. **Bản bỏ dấu tiếng Việt (Unaccented / Diacritics Removal):**
   - Dùng để so khớp linh hoạt với từ điển từ khóa (ví dụ: người dùng gõ `"do xang"` hay `"đổ xăng"` đều nhận diện được danh mục `transportation`).
   - **Lưu ý bẫy kinh điển:** Hàm `java.text.Normalizer.normalize(..., Normalizer.Form.NFD)` trong Java chỉ tách combining marks của các chữ như `á`, `ề`, `ô`... Ký tự `đ` và `Đ` trong bảng mã Unicode là ký tự phụ âm độc lập, **không bị ảnh hưởng bởi Normalizer**. Vì vậy, hàm bỏ dấu tiếng Việt bắt buộc phải có bước thay thế riêng:
     ```kotlin
     val withoutD = text.replace('đ', 'd').replace('Đ', 'D')
     ```

### 2. Tách cụm số và đơn vị (Number-Unit Splitting)
Người Việt rất hay gõ dính số và đơn vị tiền tệ: `45k`, `1tr2`, `2củ`, `5lít`, `100nghìn`.
Tầng chuẩn hóa cần chèn khoảng trắng hợp lý giữa số và chữ để tầng phân tích số tiền (S0-11) dễ dàng nhận diện token:
- `45k` $\rightarrow$ `45 k`
- `1tr2` $\rightarrow$ `1 tr 2`
- `2củ` $\rightarrow$ `2 củ`
- `5lít` $\rightarrow$ `5 lít`
- `50nghìn` $\rightarrow$ `50 nghìn`
- **Ngoại lệ quan trọng:** Không được ngắt số thập phân hoặc phân cách hàng nghìn:
  - `1.5tr` $\rightarrow$ `1.5 tr` (dấu chấm ở giữa hai chữ số phải giữ nguyên).
  - `1,5tr` $\rightarrow$ `1,5 tr` (dấu phẩy ở giữa hai chữ số phải giữ nguyên).
  - `45.000` $\rightarrow$ giữ nguyên `45.000`.

### 3. Mô hình dữ liệu Token & NormalizedSentence
Thay vì chỉ trả về một chuỗi `String`, tầng này trả về cấu trúc dữ liệu giàu thông tin:
```kotlin
enum class TokenType {
    WORD,
    NUMBER,
    PUNCTUATION,
    NEWLINE,
}

data class Token(
    val text: String,
    val unaccented: String,
    val startIndex: Int, // Vị trí bắt đầu trong raw (NFC)
    val endIndex: Int, // Vị trí kết thúc trong raw (NFC)
    val type: TokenType = TokenType.WORD,
)

data class NormalizedSentence(
    val raw: String, // Câu chuẩn hóa Unicode NFC
    val text: String, // Các token chữ thường nối bằng khoảng trắng
    val unaccentedText: String, // Các token không dấu nối bằng khoảng trắng
    val tokens: List<Token>, // Danh sách token kèm vị trí trên raw
)
```

---

## So với cách cũ bạn quen

| Cách cũ (String Manipulation thô) | Cách mới (Pipeline Normalization) | Khác biệt chính |
| :--- | :--- | :--- |
| Dùng `split(" ")` và `replace()` rải rác ở khắp ViewModel / Helper | Đóng gói thành một Component duy nhất (`Normalizer`) có đầu vào và đầu ra rõ ràng | Dễ viết Unit Test cô lập, tái sử dụng trong toàn bộ pipeline |
| So khớp chuỗi trực tiếp bằng `contains()` có dấu | Sinh đồng thời 2 bản: bản có dấu để hiển thị/ghi chú, bản không dấu để so khớp | Hỗ trợ gõ tiếng Việt không dấu tự nhiên mà không cần viết 2 bộ regex |
| Xử lý dính số `45k` bằng if/else lồng nhau | Dùng Regex tiền xử lý chuẩn hóa ranh giới từ (word boundaries) | Xử lý được các ca phức tạp như `1tr2`, `1,5tr`, `2củ` chỉ trong một lượt quét |

---

## Các trường hợp biên cần lưu ý (Edge Cases)
1. **Chuỗi rỗng hoặc chỉ có khoảng trắng:** Trả về danh sách token rỗng, không ném ngoại lệ (`IndexOutOfBoundsException`).
2. **Ký tự `đ` và `Đ`:** Đảm bảo `"đổ xăng"` $\rightarrow$ `"do xang"`, `"đi chợ"` $\rightarrow$ `"di cho"`.
3. **Số nhà và địa chỉ:** `"123 Lê Lợi"` không được biến thành `"123 L ê Lợi"`.
4. **Dấu gạch ngang và dấu phẩy:** Dấu phẩy ngăn cách hàng nghìn (`1,250,000`) khác với dấu phẩy ngăn cách hai khoản (`phở 45k, cafe 25k`).
