# Rà soát iS0-08 · Parser: chuẩn hóa và tách từ

Ngày rà soát: 2026-11-20 · Nhánh `task/iS0-08-chuan-hoa` · Trạng thái: **Xong** (đóng 2026-11-20 sau rà lại lần 2)

> Toàn bộ code của task (`Token.swift`, `TextNormalizer.swift`, `Tokenizer.swift` và test) được viết theo `ngoại lệ:` và đã ghi trong tiến độ. Mentor rà soát như mọi task khác.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ✅ exit 0 |
| `swiftlint lint --strict` | ✅ exit 0 |
| `cd Core && swift test` | ✅ 8 test của `TokenizerTests` đều qua |
| `cd Core && xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) | ✅ |
| Gọi `tokenize` và `normalize` với 8 câu biên trong một chương trình nháp | Xem các mục Nên sửa 1–2. Chuỗi dạng NFD, chữ hoa, `Đ`, `1tr2` và emoji đều xử lý đúng |
| Chạy thử snippet regex trong file lý thuyết | ❌ `lookbehind is not currently supported` |
| Đối chiếu `docs/shared/parser/README.md` | Xuống dòng là một trong các dấu tách giao dịch (dòng 198). Có 3 câu mẫu chứa `\n` (`tc_141`–`tc_143`) |

## Bắt buộc sửa (0)
Tiêu chí xong đã đạt. Có test cho chữ có dấu, chữ hoa, khoảng trắng thừa và cụm số dính đơn vị (`45k`, `1tr2`, `2củ`, `5lít`, `50nghìn`, `30ngàn`).

## Nên sửa (5)

### 1. Có hai pipeline chuẩn hóa, cho ra kết quả khác nhau
- **Vị trí:** `TextNormalizer.normalize` (`TextNormalizer.swift:44-50`) và `Tokenizer.tokenize` (`Tokenizer.swift:15-78`).
- **Vấn đề:**
  - `tokenize` tự tách số, chữ và dấu câu bằng regex của nó. Hàm này không gọi `separateNumberAndUnits` và cũng không gọi `normalize`.
  - Hai hàm cho kết quả khác nhau với cùng một câu. Ví dụ `"tiền điện: 450k;\r\n…"`:
    - `tokenize().text` cho ra `"tiền điện : 450 k ; tiền nước…"`;
    - `normalize().text` cho ra `"tiền điện: 450 k; tiền nước…"`, và mất luôn ký tự xuống dòng.
- **Vì sao:** từ iS0-09 trở đi, các bước sau sẽ dùng một trong hai kết quả. Nếu chỗ này dùng hàm này, chỗ kia dùng hàm kia, thì vị trí token và chuỗi so khớp không còn khớp nhau. Lỗi kiểu này rất khó dò.
- **Hướng sửa:**
  - Chọn **một** cửa vào duy nhất. Đề xuất dùng `Tokenizer.tokenize`, vì nó giữ được vị trí và loại token.
  - `text` và `unaccentedText` đều suy ra từ danh sách token.
  - Bỏ `normalize` và `separateNumberAndUnits` nếu không còn ai dùng. Nếu giữ lại, cho chúng gọi qua `tokenize`.
  - Test chuyển sang kiểm tra cửa vào đó.

### 2. Xuống dòng kiểu `\r\n` bị mất
- **Vị trí:** `Tokenizer.swift:22`, `(?<newline>\r?\n)`.
- **Vấn đề:**
  - Trong Swift, `"\r\n"` là **một** `Character` (grapheme cluster). Vì vậy `\r?\n` không khớp được nó.
  - Mentor đã thử với chuỗi `"a\r\nb\nc"`: `\r?\n` khớp 1 lần, còn `\R` khớp 2 lần.
  - Hệ quả: token xuống dòng biến mất, và hai giao dịch trên hai dòng bị dính vào nhau.
  - Ngoài ra `normalizeWhitespace` cũng gộp mọi ký tự xuống dòng thành dấu cách.
- **Vì sao:** README dùng chung với Android quy định xuống dòng là dấu tách giao dịch. Văn bản dán từ web, email hoặc từ Windows thường có `\r\n`.
- **Hướng sửa:**
  - Dùng `\R`, hoặc kiểm tra `Character.isNewline` để bắt mọi kiểu xuống dòng.
  - Thêm test có `"\r\n"`.

### 3. Test của `Tokenizer` còn mỏng
- **Vị trí:** `TokenizerTests.swift:55-123`.
- **Vấn đề:**
  - Phần lớn các ca khó (`1tr2`, `1,5tr`, `45.000`, `Đ`) chỉ được test qua `TextNormalizer`.
  - `Tokenizer`, tức là kết quả thật mà các bước sau sẽ dùng, chỉ có một câu được kiểm chi tiết.
  - Test dấu câu chỉ kiểm `types.contains(...)`, không kiểm thứ tự hay nội dung token.
- **Hướng sửa:** thêm test có tham số (`arguments:`) cho `tokenize`. Mỗi ca so nguyên danh sách `(text, type)`. Các ca cần có:
  - `1tr2`, `1,5tr`, `1.250.000`, `ĐỔ XĂNG 50K`;
  - `"sáng phở 45k, trưa cơm 35k"` (dấu phẩy tách giao dịch, không phải dấu thập phân);
  - `15/10`, có `\r\n`;
  - một chuỗi dạng NFD. Chuỗi này hiện xử lý đúng, nên khóa lại bằng test để không bị hỏng về sau.

### 4. File lý thuyết có snippet chạy không được
- **Vị trí:** `hoc/iS0-08-chuan-hoa-tach-tu.md`, mục "API và cú pháp" số 2 và dòng thứ ba của bảng so sánh.
- **Vấn đề:**
  - Regex `(?<=\d)(?=…)|(?<=…)(?=\d)` dùng lookbehind. Swift Regex hiện chưa hỗ trợ, và mentor đã chạy thử: lỗi `lookbehind is not currently supported`.
  - Code thật lại dùng cách khác: regex literal có capture đặt tên, kèm closure `replacing`.
  - File chưa mô tả `Tokenizer`, trong khi đây là phần chính của task: regex có nhiều nhánh, `match.output.<tên>`, loại token và cách tính vị trí.
- **Hướng sửa:**
  - Thay snippet bằng cách đã dùng trong code.
  - Thêm một mục ngắn về regex literal có capture đặt tên.
  - Thêm một mục về "`\r\n` là một `Character`".

### 5. Tự kiểm tra hiểu bài (vì task viết theo ngoại lệ)
Task này được chọn cho bạn tự viết, vì `String`, `Character`, Unicode và Regex là trọng tâm học của iS0. Phần này giờ do agent viết, nên hãy trả lời bằng lời của bạn trong chat:
1. Câu `"phở"` gõ ở dạng NFD (chữ `o` cộng hai dấu tổ hợp) vẫn `==` với `"phở"` dạng NFC, và test vẫn qua. Vì sao Swift so sánh được như vậy? `count` của hai chuỗi này có bằng nhau không?
2. Vì sao `\r?\n` không khớp `"\r\n"`, còn `\R` thì khớp? (liên quan Nên sửa 2)
3. Vì sao phải thay `đ` thành `d` bằng tay, trước khi gọi `folding(options: .diacriticInsensitive)`?
4. Trong `#/(?<num>\d)(?<unit>[^\d\s.,])/#`, `match.output.num` có kiểu gì? Vì sao gọi được nó bằng tên mà không cần ép kiểu?
5. `Token` lưu `startIndex` dạng `Int` (đếm theo `Character`). Mỗi lần đổi giữa `Int` và `String.Index` tốn chi phí gì? Nếu lưu luôn `Range<String.Index>` thì được gì, mất gì?

Câu nào chưa đạt, mentor sẽ giải thích lại rồi hỏi lại. Nếu muốn, bạn có thể ghi `hoãn` mục này để ôn trong iS0-09.

## Gợi ý thêm để học (3)
1. `vietnameseLocale` được khai báo lặp ở hai file. Có thể gom về một chỗ.
2. Emoji (`☕️`) đang bị xếp loại `.punctuation`. Không sai về chức năng, nhưng nếu thêm loại `.symbol` thì ý nghĩa rõ ràng hơn, hoặc có thể bỏ qua emoji.
3. Vị trí token hiện đếm theo `Character`, còn Android (Kotlin) đếm theo UTF-16. Hiện chưa sao, vì vị trí không đi ra ngoài thiết bị. Nhưng nếu sau này lưu vị trí vào file dùng chung giữa hai nền tảng, cần ghi rõ cách đếm.

## Điểm làm tốt
- Regex literal có capture đặt tên. Kiểu `match.output` được compiler kiểm tra lúc biên dịch, không phải ép kiểu.
- Xử lý đúng các ca khó:
  - `Đ` hoa và `đ` thường;
  - chuỗi dạng NFD (nhờ Swift so sánh theo tương đương chuẩn);
  - số thập phân và số có phân cách hàng nghìn không bị tách.
- `Token` giữ vị trí trên chuỗi gốc, nên các bước sau cắt được ghi chú từ `raw`. Có test cắt lại chuỗi để kiểm chứng điều này. Cả module vẫn nonisolated và `Sendable`.

## Kết luận
Chưa đóng được task. Không có mục Bắt buộc. Cần sửa Nên sửa 1–4, và trả lời các câu hỏi ở Nên sửa 5 (hoặc ghi `hoãn <mục>`). Sửa xong thì báo lại, mentor sẽ rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-20)
**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅ 12 test trong `TokenizerTests` đều qua, gồm 8 ca có tham số.
- `xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) → ✅.

**Đã đạt:**
- **Nên sửa 1:** ✅ `normalize` gọi `Tokenizer.tokenize`, nên chỉ còn một nguồn kết quả.
- **Nên sửa 2:** ✅ Đã dùng `(?<newline>\R)`, có ca test `"phở 45k\r\ncơm 35k"`.
- **Nên sửa 3:** ✅ Có test tham số so nguyên danh sách `(text, type)` cho 8 ca biên (có NFD và `\r\n`), và kiểm cắt chuỗi `raw` theo vị trí token.
- **Nên sửa 4:** ✅ Snippet lookbehind đã được thay bằng capture có tên. File có thêm mục về `\R`, grapheme cluster và regex literal của `Tokenizer`.

**Nên sửa 5 (câu trả lời của người dùng):**

| Câu | Đánh giá |
| --- | --- |
| 1. NFD `==` NFC | ✅ Đạt: Swift so sánh theo tương đương chuẩn (canonical equivalence), không so từng byte. Bổ sung phần `count`: cả hai dạng đều có `count == 3`, nhưng `unicodeScalars.count` khác nhau (3 và 5) |
| 2. `\r?\n` và `\R` | ✅ Đạt |
| 3. Chữ `đ` | ✅ Đạt: `đ` (U+0111) là một chữ cái riêng, không tách được thành `d` cộng dấu |
| 4. Kiểu của `match.output.num` | ⚠️ Mới đạt một phần: đúng là tuple có tên, được sinh lúc biên dịch, nhưng chưa nêu kiểu cụ thể |
| 5. `Int` và `Range<String.Index>` | ❌ Chưa đạt: "tối ưu hiệu năng, giảm linh hoạt" còn quá chung |

**Gợi ý nhỏ:** `separateNumberAndUnits` và `normalizeWhitespace` không còn nằm trong pipeline, chỉ test gọi tới. Hai hàm này là API `public` có thể cho kết quả khác `tokenize`. Nên bỏ chúng, hoặc chuyển sang `internal`.

**Kết luận:** chưa đóng được task. Chỉ còn câu 4 và câu 5 của Nên sửa 5.

## Rà lại lần 2 (2026-11-20)
Người dùng trả lời lại câu 4 và câu 5:

| Câu | Đánh giá |
| --- | --- |
| 4. Vì sao là `Substring`, và rủi ro khi giữ lâu | ✅ Đạt. `Substring` dùng chung bộ nhớ với chuỗi gốc nên cắt chuỗi tốn O(1), không phải cấp phát mới. Nếu giữ lâu, `Substring` giữ lại toàn bộ chuỗi gốc trong bộ nhớ. Muốn lưu lâu dài thì đổi sang `String(...)`, như `Tokenizer` đang làm |
| 5. Dùng `Index` của `raw` để cắt `text`; vì sao `Int` dễ lưu, dễ so sánh | ✅ Đạt, kèm 2 điều chỉnh nhỏ, xem bên dưới |

**Điều chỉnh nhỏ cho câu 5:**
- Dùng index của chuỗi khác thì Swift sẽ **trap** (crash) nếu index vượt ra ngoài chuỗi. Nếu index còn nằm trong chuỗi, Swift làm tròn về ranh giới ký tự gần nhất, nên app không crash nhưng **cắt ra sai nội dung**. Không có trường hợp "đọc lủng" giữa một ký tự UTF-8.
- `String.Index` cũng `Comparable`. Điểm yếu của nó là muốn tính khoảng cách hay dịch vị trí thì luôn phải có chuỗi gốc đi kèm, và nó không `Codable`.

**Kết luận:** Đóng được task.
