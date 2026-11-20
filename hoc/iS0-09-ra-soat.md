# Rà soát iS0-09 · Parser: số tiền (FR-02)

Ngày rà soát: 2026-11-20 · Nhánh `task/iS0-09-so-tien` · Trạng thái: **Xong**

> Đây là task giải thuật theo mục 2 của luật. `AmountParser` và test của nó thuộc phần agent viết. Phần nối vào `TransactionParser.parse` và khung test 200 câu thuộc phần người dùng viết.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ✅ exit 0 |
| `swiftlint lint --strict` | ✅ exit 0 |
| `cd Core && swift test` | ✅ 21 test qua. Đúng số tiền ở **200/200** câu mẫu |
| `cd Core && xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) | ✅ |
| Thử `AmountParser` với 23 câu **ngoài** bộ mẫu, trong một chương trình nháp | ❌ Xem bảng bên dưới. Có 3 lỗi nặng: sai số tiền 1.000 lần, gộp nhầm số lượng vào số tiền, và crash |
| Tìm trong `test_cases.json` | Không có câu nào dạng `<số không có dấu chấm> đ`, `<số>k <số>` có khoảng trắng, hay `<số>m` |

Đúng 100% trên bộ mẫu nhưng sai ở những cách viết phổ biến mà bộ mẫu không có. Đó là dấu hiệu giải thuật đang **khớp theo bộ mẫu** (overfitting), chưa thật sự tổng quát.

| Câu thử | Kết quả | Đúng phải là |
| --- | --- | --- |
| `cafe 50000đ` | 50.000.000 | 50.000 |
| `ăn trưa 35000 đồng` | 35.000.000 | 35.000 |
| `nạp thẻ 200000 vnd` | 200.000.000 | 200.000 |
| `trà sữa 45k 2 ly` | 45.200 | 45.000 |
| `9999999999999999k` | **crash** (exit 133, trap do tràn số) | không có số tiền hoặc bị loại |
| `mua 5m vải 200k` | [5.000.000, 200.000] | [200.000] |
| `tiền nhà 1tr05` | 1.500.000 | 1.050.000 |
| `xăng 70 nghìn rưỡi` | 70.000 | 70.500 |
| `hai mươi nghìn`, `năm mươi nghìn`, `mười lăm nghìn` | [] | 20.000; 50.000; 15.000 |
| `45.000`, `1.250.000đ`, `2tr500k`, `một triệu hai`, `ngày 15 mua sách 120k`, `mua 3 cái áo 300k` | ✅ đúng | |

## Bắt buộc sửa (3)

### 1. `đ`, `đồng`, `vnd` bị nhân thêm 1.000
- **Vị trí:** `AmountParser+Matching.swift:192`. Danh sách nhân 1.000 có chứa `"d", "vnd", "dong"`.
- **Vì sao:**
  - `đ`, `đồng` và `vnd` là **ký hiệu tiền tệ**, không phải hệ số. Vì vậy `50000đ` bị đọc thành 50 triệu.
  - Câu mẫu `150.000 đ` vẫn đúng chỉ là **trùng hợp**: `parseDecimalAmount` coi `150.000` là số thập phân 150,000, rồi nhân 1.000.
  - File lý thuyết cũng ghi sai luật này ("`đ`, `vnd` ×1.000").
- **Hướng sửa:**
  - Bỏ `d`, `vnd`, `dong` khỏi nhóm nhân 1.000. Chúng chỉ là hậu tố đi sau số, như `makeAmountWithOptionalCurrency` đã xử lý.
  - Với nhóm có hệ số, dấu chấm trong `45.000` phải được hiểu là phân cách hàng nghìn, không phải phần thập phân. Cần có luật rõ ràng để phân biệt `1.5tr` với `45.000`, ví dụ dựa vào số chữ số sau dấu chấm.

### 2. `45k 2 ly` bị đọc thành 45.200
- **Vị trí:** `matchCompoundK` (`:23-51`).
- **Vì sao:** hàm này không kiểm tra số thứ hai có **dính liền** với `k` hay không. Kiểu viết "giá + số lượng" (`45k 2 ly`, `100k 3 cái`) rất phổ biến, và hệ quả là sai tiền mà không có cảnh báo nào.
- **Hướng sửa:**
  - Với `k`, chỉ ghép khi số phụ nằm liền ngay sau đơn vị, tức `unit.endIndex == second.startIndex`, như trong `85k5`.
  - Với `củ`, `tr`, bộ mẫu có `1 củ 2` (có khoảng trắng) nên không thể yêu cầu dính liền. Thay vào đó, không ghép khi token ngay sau số phụ là một danh từ đơn vị đếm (`ly`, `cái`, `cốc`…), hoặc khi số phụ có từ 3 chữ số trở lên mà vẫn bị tách rời.

### 3. Tràn số làm crash app
- **Vị trí:** mọi phép `*` và `+` trên `Int64`, ví dụ `:42`, `:75`, `:271`, `:279`.
- **Vì sao:** đầu vào là chữ người dùng gõ. Swift **trap** khi tràn số, nên một chuỗi số dài sẽ làm crash app. Parser không được crash vì dữ liệu nhập.
- **Hướng sửa:**
  - Dùng `multipliedReportingOverflow(by:)` và `addingReportingOverflow(_:)`. Khi tràn thì bỏ qua cụm đó (trả `nil`).
  - Có thể đặt thêm trần hợp lý cho một giao dịch.
  - Thêm test cho trường hợp tràn số.

## Nên sửa (4)

### 1. `m` được coi là triệu
- **Vị trí:** `:59` và `:194`.
- **Vì sao:** bộ mẫu không có câu nào dùng `m`. Trong khi đó, đời thường `5m` thường là 5 mét (`mua 5m vải`), nên câu này bị đọc thành 5 triệu và còn bị tách thành 2 giao dịch.
- **Hướng sửa:** bỏ `m`. Nếu muốn hỗ trợ thì cần quyết định có chủ đích, và bổ sung câu mẫu vào bộ dùng chung, đồng bộ với Android.

### 2. `1tr05` thành 1.500.000
- **Vị trí:** `:66-73`.
- **Vì sao:** hệ số của số phụ đang được tính theo **giá trị** của nó (`5 < 10`), thay vì theo **số chữ số** của nó (`"05"` có 2 chữ số). Lỗi tương tự với `85k05`.
- **Hướng sửa:** tính hệ số từ độ dài chuỗi `second.text`. Một chữ số nhân 100.000, hai chữ số nhân 10.000, ba chữ số nhân 1.000.

### 3. Độ tin cậy luôn là 1.0
- **Vị trí:** `ParsedAmount.confidence` luôn mang giá trị mặc định. `Parser.swift` còn gán `datetime` bằng `referenceTime` với độ tin cậy 1.0, dù chưa hề nhận diện thời gian.
- **Vì sao:** độ tin cậy cho từng trường là quyết định đã chốt từ iS0-07. Màn xác nhận sau này dựa vào nó để tô sáng trường cần người dùng kiểm tra lại.
- **Hướng sửa:**
  - Số có đơn vị rõ ràng (`45k`, `450.000 đ`) có độ tin cậy cao.
  - Số trần không có đơn vị (`350000`) và tiếng lóng dễ nhầm (`lít`, `chục`) có độ tin cậy thấp hơn.
  - Với `datetime`: để `nil` cho tới iS0-10, hoặc gán độ tin cậy thấp và ghi rõ đây là giá trị mặc định.

### 4. File lý thuyết `hoc/iS0-09-parser-so-tien.md`
- **Vấn đề:**
  - Ghi sai luật "`đ`, `vnd` ×1.000" (xem Bắt buộc 1).
  - Snippet 2 nhân bằng `Double`, trái ngược với chính "Lỗi hay gặp 1" của file.
  - Chưa mô tả **thứ tự ưu tiên** của các hàm `match…`, trong khi đây là cốt lõi của giải thuật. Ví dụ: vì sao `matchCompoundK` phải chạy trước `matchNumberWithUnit`.
  - Link "Đọc thêm" là đường dẫn `file:///Users/…` trên máy cá nhân. Repo là public, nên link này vừa không mở được trên máy khác, vừa lộ tên tài khoản máy.
- **Hướng sửa:** sửa cùng lúc với phần giải thuật. Thêm bảng thứ tự ưu tiên và các giới hạn đã biết.

## Gợi ý thêm để học (4)
1. **Số đọc bằng chữ có `mươi` và `nghìn`** (`hai mươi nghìn`, `mười lăm nghìn`) chưa được hỗ trợ. Tương tự với `70 nghìn rưỡi`. Đây là phạm vi FR-02 nhưng bộ mẫu chưa có câu nào kiểu này. Có thể bổ sung câu vào bộ dùng chung, cần đồng bộ với Android, và để làm ở S6 hoặc iS6.
2. **`lít` mơ hồ:** `đổ 5 lít xăng 100k` hiện cho ra [500.000, 100.000]. Đây là giới hạn đã biết của tiếng lóng. Nên ghi vào file lý thuyết và hạ độ tin cậy cho trường hợp này.
3. **Các câu thử ở bảng trên** nên được đưa vào `AmountParserTests` sau khi sửa, để chặn hồi quy. Nếu muốn Android dùng chung các câu này, cần thêm vào `docs/shared/parser/test_cases.json`. Việc đó cần người dùng đồng ý, vì làm đổi `totalCases` và ảnh hưởng tới cả hai bản.
4. `testAccuracyOn200CasesThreshold` gọi thẳng `AmountParser`, còn test có tham số gọi `TransactionParser`. Cả hai đều có ích. Nếu chỉ giữ một, nên đo qua `TransactionParser`, vì đó là đường đi thật của dữ liệu.

## Điểm làm tốt
- Đạt và vượt Cổng 1 trên bộ mẫu (200/200).
- Khung test 200 câu đã đổi sang **kiểm chặt** số lượng và số tiền, ngoài khối `withKnownIssue`. Như vậy câu nào bị hỏng sẽ báo đỏ ngay. Nợ kỹ thuật từ iS0-07 đã xử lý xong phần số tiền.
- Số thập phân xử lý bằng số nguyên, không dùng `Double`.
- Mỗi kết quả có vị trí ký tự và vị trí token, giúp việc trích ghi chú sau này.
- Bỏ qua số đứng sau `ngày` và `mùng`.

## Kết luận ban đầu
Chưa đóng được task. Cần sửa Bắt buộc 1–3 và Nên sửa 1–4. Theo mục 2 của luật, phần giải thuật và file lý thuyết là phần agent viết. Sau khi sửa xong, người dùng trả lời 3–5 câu hỏi kiểm tra hiểu thì mới đóng được task.

---

## Agent sửa phần giải thuật (2026-11-20)
Người dùng đồng ý ("Sửa luôn"). Theo mục 2, agent sửa phần giải thuật, test của giải thuật và file lý thuyết. Không đụng `Parser.swift` và `ParserTests.swift`, vì đó là phần của người dùng.

**Thay đổi:**
- `AmountParser.swift`:
  - Thêm các tập đơn vị tách riêng. `đ`, `đồng`, `vnd` là ký hiệu tiền tệ, không còn là hệ số.
  - Bỏ `m`.
  - Thêm từ đếm, từ chỉ chất lỏng và `maxAmount`.
  - Thêm 5 mức `Confidence`.
- `AmountParser+Number.swift` (file mới):
  - `parseNumber` phân biệt phân cách hàng nghìn với số thập phân.
  - `fractionValue` tính phần lẻ theo số chữ số.
  - `multiply` và `add` kiểm tra tràn số.
- `AmountParser+Matching.swift`:
  - `85k5` chỉ ghép khi viết liền.
  - `3tr 2 người` và `1tr 2tr` không bị ghép.
  - Hỗ trợ thêm `2tr500k`, `70 nghìn rưỡi`, `1,250,000`.
  - Bỏ qua số bắt đầu bằng `0`.
  - Mỗi mẫu gán độ tin cậy riêng.
- `AmountParserTests.swift`: thêm 19 câu ngoài bộ mẫu (test có tham số), test độ tin cậy và test `parseNumber`.
- `hoc/iS0-09-parser-so-tien.md`: viết lại. Có bảng thứ tự ưu tiên, luật dấu chấm và dấu phẩy, độ tin cậy, giới hạn đã biết, và link công khai.

**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅. Vẫn đúng **200/200** câu mẫu, 19/19 câu ngoài bộ mẫu.
- `xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) → ✅.

**Kết quả:** Bắt buộc 1–3 ✅. Nên sửa 1, 2, 4 ✅. Nên sửa 3 xong phần số tiền.

---

## Hoàn thành và đóng task (2026-11-20)
1. **Phần nối `Parser.swift:32`:** Đã sửa gán `datetime: nil` với ghi chú rõ ràng chờ `iS0-10` (nhận diện thời gian).
2. **Kiểm tra hiểu:** Đã giải đáp và làm rõ 5 câu hỏi kỹ thuật về giải thuật (sai số 1000 lần do ký hiệu tiền tệ, luật tính phần lẻ và từ đếm, thứ tự ưu tiên tham lam trong `matchAny`, mở rộng đọc số bằng chữ, an toàn tràn số và sự khác biệt giữa Swift trap vs Kotlin overflow wrap-around).

**Kết luận chung:** Toàn bộ tiêu chí nghiệm thu và Cổng 1 (200/200 câu mẫu đạt 100%, 19/19 ca biên ngoài bộ mẫu đạt 100%, không crash tràn số, lint và test sạch) đều đã thỏa mãn. **Đóng được task iS0-09.** Chuyển sang `iS0-10`.

## Rà lại lần 2 (2026-11-20)
**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅, 200/200 câu mẫu.
- `xcodebuild … test` (iPhone 18 Pro) → ✅.
- App build → ✅.

**Phần của người dùng:** ✅ `Parser.swift` để `datetime: nil` cho tới iS0-10, có ghi chú lý do.

**Câu hỏi kiểm tra hiểu (mục 2 của luật):**

| Câu | Đánh giá |
| --- | --- |
| 1. Vì sao `50000đ` bị đọc thành 50 triệu, còn `150.000 đ` vẫn đúng | ✅ Đạt: hai lỗi tự triệt tiêu nhau. `150.000` bị hiểu là số thập phân 150, rồi bị nhân 1.000 |
| 2. `1tr05` và `45k 2 ly` | ✅ Đạt, kèm một điều chỉnh. Số `2` trong `45k 2 ly` bị bỏ vì nó là số trần nhỏ hơn 1.000 và không có đơn vị (`matchRawNumber`). Danh sách từ đếm chỉ được dùng cho mẫu `<số> tr/củ <số>` có khoảng trắng, ví dụ `3tr 2 người` |
| 3. Thứ tự ưu tiên trong `matchAny` | ✅ Đạt: khớp tham lam, mẫu dài trước. Nếu đảo thứ tự, `1tr2` chỉ còn 1.000.000 |
| 4. Hỗ trợ `hai mươi nghìn` | ✅ Đạt. Cụ thể: thêm `mươi`, `lăm`, `mốt`, `tư` vào `vnNumberWords`, viết hàm ghép cụm chữ số thành một số, rồi dùng hàm đó trong `integerOrWord`, để mọi mẫu có đơn vị đều nhận được. Muốn sửa `test_cases.json` thì phải được người dùng đồng ý và đồng bộ với Android |
| 5. `multipliedReportingOverflow`; Swift và Kotlin khi tràn số | ✅ Đạt |

**Nợ kỹ thuật từ iS0-07** (so sánh từng trường, tỉ lệ đúng, chặn hồi quy): đã xử lý xong phần số tiền. Test có tham số kiểm chặt số lượng giao dịch và số tiền, còn `testAccuracyOn200CasesThreshold` kiểm tỉ lệ đúng. Các trường khác sẽ được đưa ra khỏi `withKnownIssue` ở task tương ứng của chúng.

**Kết luận:** Đóng được task.
