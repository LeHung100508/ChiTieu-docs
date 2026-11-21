# iS0-11 · Parser: danh mục và tách khoản (FR-04, FR-06, FR-07)

> Task giải thuật theo mục 2 của luật.
> - **Agent viết:** `ClauseSplitter`, `CategoryDictionary`, `WalletMatcher`, `ClauseAnalyzer`, `ParserConfiguration` và test của chúng.
> - **Người dùng làm:** đưa từ điển vào app, nối các phần vào `TransactionParser`, kiểm chặt các trường trong test 200 câu, và trả lời câu hỏi kiểm tra.

## Vì sao cần
Có số tiền và thời gian rồi, một giao dịch vẫn còn thiếu: loại (chi, thu, chuyển khoản), danh mục (`seedKey`), ví và ghi chú. Ngoài ra, một câu có thể chứa nhiều khoản, ví dụ `sáng phở 45k, trưa cơm 35k`. Phần này lấp đủ các trường còn lại theo quy ước dùng chung với Android trong `docs/shared/parser/README.md`.

## Khái niệm chính
- **Tách khoản (`ClauseSplitter`).** Câu có N khoản tiền thì có N khoản. Ranh giới là dấu phân cách đầu tiên (`,` `;`, xuống dòng, `và`, `với`) nằm **giữa hai khoản tiền**. Nếu không có dấu phân cách thì ranh giới nằm ngay sau khoản tiền trước. Vì vậy `mua rau và thịt 80k` (một khoản tiền) không bị tách, và chữ `và` vẫn ở lại trong ghi chú.
- **Token "thuộc về" ai.** Trong mỗi khoản, các token đã dùng cho trường khác đều bị loại:
  - số tiền;
  - cụm thời gian;
  - bí danh ví và từ dẫn ví đứng ngay trước nó (`bằng`, `qua`, `chuyển khoản`, `quẹt`…);
  - dấu câu, và `và`/`với` khi chúng làm ranh giới.

  Phần còn lại vừa là **ghi chú**, vừa là chỗ tìm **từ khóa danh mục**.
- **Danh mục (`CategoryDictionary`).** Tìm cụm từ khóa khớp tốt nhất. Thứ tự ưu tiên khi so:
  1. nhiều token hơn;
  2. bản có dấu (độ tin cậy 1.0) hơn bản không dấu (0.8);
  3. cụm đứng trước.

  Ví dụ `thu tiền phòng trọ` (4 token, danh mục thu) thắng `tiền phòng trọ` (3 token, nhà ở). Không khớp gì thì là `other_expense`, độ tin cậy 0.3.
- **Từ đơn ngắn phải có dấu.** Khi bỏ dấu, `chợ` thành `cho`, `cá` thành `ca`, `táo` thành `tao`, đều trùng với từ khác. Vì vậy từ một token mà bản không dấu ngắn hơn 4 ký tự chỉ được khớp bản có dấu. Cụm từ hai token trở lên đã có ngữ cảnh, nên khớp được cả bản không dấu (`di cho` = `đi chợ`).
- **Loại giao dịch:**
  - **Chuyển khoản:** có cả ví nguồn (đi sau `từ`) và ví đích (đi sau `sang` hoặc `vào`). Với `rút tiền`, ví đích là ví còn lại, ví dụ `tiền mặt`.
  - **Thu:** danh mục là `income_*` hoặc `other_income`.
  - **Chi:** các trường hợp còn lại. Vì vậy `ăn tối 300k chuyển khoản vcb` là khoản chi trả bằng vcb, không phải chuyển khoản.
- **Ví (`WalletMatcher`).** So bí danh trên bản không dấu. Khi hai bí danh chồng lên nhau, bí danh dài hơn thắng: `thẻ tín dụng` thắng `quẹt thẻ`, `thẻ vcb` thắng `vcb`. Danh sách ví do app truyền vào, vì người dùng tự đặt tên ví (FR-06).
- **Ghi chú:**
  - Lấy các đoạn token liền nhau còn lại, cắt **nguyên văn** từ `raw`, nên giữ được `Grab` viết hoa và `4g` viết liền.
  - Các đoạn nối với nhau bằng một dấu cách: `đổ 50k xăng` cho ra `đổ xăng`.
  - Hư từ đứng ngay trước số tiền (`tiêu`, `hết`, `mất`, `được`) bị bỏ, nếu ngoài nó còn nội dung khác.
  - Chuyển khoản thuần túy không có ghi chú.
- **`ParserConfiguration`** gom những gì parser cần ngoài câu nhập: từ điển, danh sách ví, và `Calendar`. App tạo một lần rồi truyền vào parser.

## So với cách cũ bạn quen
| Cũ (Android) | Mới (Swift) | Khác biệt chính |
| --- | --- | --- |
| Đọc từ điển bằng `context.assets.open(...)` bên trong parser | `CategoryDictionary(data:)`, app tự đọc `Data` từ bundle | Parser không phụ thuộc `Bundle`, nên test chỉ cần truyền `Data` |
| `Map<String, List<String>>` giữ thứ tự chèn | `[String: [String]]` không có thứ tự | Phải sắp xếp khóa (`keys.sorted()`) để kết quả ổn định giữa các lần chạy |
| `data class` + `copy()` | `struct` với `var claimed` truyền `inout` | Value type: mỗi khoản có bản `Clause` riêng, nên không lo bị dùng chung |

## API và cú pháp cần biết
- `zip(a, b)`: ghép hai dãy theo cặp, dừng ở dãy ngắn hơn.
- `Set.formUnion(_:)`, `Range.overlaps(_:)`, `Set.isDisjoint(with:)`: thao tác trên tập token.
- `for … where …`: lọc ngay trong vòng lặp.
- `inout`: hàm sửa trực tiếp biến của nơi gọi (ở đây là `Clause`).

```swift
// Ví dụ khác bài toán: chọn khung giờ họp không chồng nhau, khung dài ưu tiên trước
var booked: [Range<Int>] = []
for slot in slots.sorted(by: { $0.count > $1.count })
where !booked.contains(where: { $0.overlaps(slot) }) {
    booked.append(slot)
}
```

## Lỗi hay gặp
1. **Tách khoản theo mọi chữ `và`.** Làm vậy thì `mua rau và thịt 80k` bị chẻ đôi. Chỉ tách ở phân cách nằm giữa hai khoản tiền.
2. **Khớp từ khóa ngắn nhất hoặc đầu tiên.** Khi đó `thu tiền phòng trọ` bị xếp vào nhà ở. Phải ưu tiên cụm dài nhất.
3. **Coi mọi câu có chữ `chuyển khoản` là chuyển khoản.** `ăn tối 300k chuyển khoản vcb` là khoản chi. Chuyển khoản phải có đủ cả ví nguồn lẫn ví đích.
4. **Ghép lại ghi chú từ token đã chuẩn hóa.** Làm vậy sẽ mất chữ hoa (`grab`) và tách sai `4g` thành `4 g`. Phải cắt từ `raw` theo vị trí token.
5. **Đọc file bằng `#filePath` lúc app chạy.** `#filePath` là đường dẫn trên máy build. Khi app chạy, phải đọc từ bundle.

## Giới hạn đã biết
- Loại "thu" chỉ nhận ra được qua từ khóa danh mục. `nhận 2tr từ mẹ` hay `bán xe 10tr` sẽ thành khoản chi `other_expense`.
- Danh mục chỉ dựa vào từ khóa, chưa học từ lựa chọn của người dùng. Việc đó để S6 làm (luật cá nhân).
- `chuyển 500k cho vợ` là khoản chi với ghi chú `chuyển cho vợ`. README chỉ yêu cầu ghi chú có `cho vợ`, nên việc giữ chữ `chuyển` là có chủ đích.

## Giải thích code chi tiết

### Các bước của pipeline
```
raw ──Tokenizer──▶ tokens ──AmountParser──▶ amounts
                     ├──ClauseSplitter → khoảng token của từng khoản
                     ├──DateParser     → token thời gian (và mốc của từng khoản)
                     ├──WalletMatcher  → token ví
                     └──ClauseAnalyzer → mỗi khoản: loại, danh mục, ví, ghi chú
```
Mỗi token chỉ thuộc về một trường. Số tiền, thời gian, ví và dấu câu lấy token trước. Phần còn lại vừa là ghi chú, vừa là chỗ tìm từ khóa danh mục.

### `ClauseSplitter`
Ví dụ câu `mua bánh mì 20k và cà phê 25k`. Các token đánh số từ 0: `mua · bánh · mì · 20 · k · và · cà · phê · 25 · k`. Số tiền nằm ở `3..<5` và `8..<10`.
- `boundaries`: tìm token phân cách đầu tiên trong khoảng **giữa** hai khoản tiền (`5..<8`). Ở đây là `và` tại index 5. Nếu không có phân cách thì ranh giới là `tokenEndIndex` của khoản trước.
- `ranges`: ghép `[0] + cuts` với `cuts + [count]`, được `[0..<5, 5..<10]`.
- Câu chỉ có một khoản tiền thì không có ranh giới nào (`guard amounts.count > 1`). Vì vậy `mua rau và thịt 80k` không bị tách.
- `clauseIndex`: đếm số ranh giới nằm trước hoặc trùng một token, để biết token đó thuộc khoản nào. `DateParser.resolve` dùng hàm này.

### `ClauseAnalyzer.analyze`
1. Lấy khoảng token của từng khoản, cụm thời gian (đã bỏ các cụm chồng lên số tiền) và các ví tìm được.
2. Tạo `claimed: Set<Int>`, gồm token của số tiền, của thời gian, mọi token không phải chữ hay số, và token phân cách **đúng tại ranh giới** (`và` ở index 5 bị đánh dấu, `và` trong `mua rau và thịt` thì không).
3. `zip(ranges, amounts)`: mỗi khoản có một `Clause` riêng. `Clause` là `struct` có `var claimed`, nên mỗi khoản giữ **bản sao riêng**. Đánh dấu thêm ở khoản này không ảnh hưởng khoản khác. Thuộc tính `content` là các token trong khoản chưa bị đánh dấu.

### `analyzeClause`: thứ tự xử lý
1. **Ví:**
   - Đánh dấu token của bí danh.
   - `claimMarkers` đi **ngược** từ trước bí danh, đánh dấu các từ dẫn liên tiếp (`bằng`, `qua`, `quẹt`, `chuyển khoản`, `từ`, `sang`, `vào`, `atm`…). Hàm trả về tập từ dẫn để xác định vai trò của ví.
2. **`claimFiller`:**
   - Bỏ `tiêu`/`hết`/`mất`/`được` nếu nó đứng ngay trước số tiền.
   - Nếu bỏ xong mà ghi chú rỗng thì trả lại từ đó.
3. **Chuyển khoản (`TransferRoles`):**
   - Ví nguồn có `từ` đứng trước; ví đích có `sang` hoặc `vào`.
   - Nếu không có ví đích mà khoản có chữ `rút`, ví đích là ví còn lại.
   - Có đủ cả hai thì loại là `.transfer`, danh mục là `nil`. Bỏ thêm `chuyển`/`khoản`/`rút`/`nạp`/`tiền` khỏi ghi chú.
4. **Danh mục:**
   - `bestMatch` chạy trên các token **còn lại**, nên `đổ 50k xăng` vẫn khớp `đổ xăng`.
   - `income_*` thì loại là `.income`.
   - Không khớp thì là `other_expense`, độ tin cậy 0.3.
5. **Ghi chú:**
   - Gom các token còn lại thành những đoạn liên tiếp.
   - Cắt nguyên văn từ `raw` theo `startIndex`/`endIndex`, nên giữ được `Grab` và `4g`.
   - Nối các đoạn bằng một dấu cách, rồi thu gọn khoảng trắng.

### `WalletMatcher`
- Tách token cho mọi bí danh, rồi so trên bản không dấu. Khớp cả dấu cho độ tin cậy 0.95, chỉ khớp bản không dấu cho 0.85.
- Chọn các ví không chồng nhau, ví dài trước. Sắp xếp theo tuple `(độ dài, -vị trí)`, rồi bỏ ứng viên nào `overlaps` với ứng viên đã chọn.

### `CategoryDictionary`
- `init(data:)` giải mã qua `struct File`, rồi gọi `compile` cho từng cụm: tách token sẵn, lưu dãy có dấu và không dấu. Việc này chỉ làm một lần.
- `keys.sorted()`: vì `Dictionary` không giữ thứ tự, phải sắp xếp khóa thì kết quả mới ổn định.
- `allowsPlain`: chỉ cho khớp không dấu khi cụm có từ 2 token trở lên, hoặc từ đơn có bản không dấu dài từ 4 ký tự.
- `bestMatch` thử mọi vị trí với mọi cụm. `isBetter` chọn theo thứ tự: nhiều token hơn, có dấu hơn, đứng trước, rồi theo khóa.

### Lần theo hai câu khó
**`siêu thị 650k quẹt thẻ tín dụng`**
- Token: `siêu · thị · 650 · k · quẹt · thẻ · tín · dụng`.
- Ví: `thẻ tín dụng` (`5..<8`) thắng `quẹt thẻ` (`4..<6`), vì dài hơn và hai cụm chồng nhau.
- `claimMarkers` đánh dấu `quẹt`, rồi dừng ở số tiền.
- Không có `từ`, nên không phải chuyển khoản.
- Kết quả: `groceries`, ví `credit_card`, ghi chú `siêu thị`.

**`rút tiền mặt 1tr từ atm vcb`**
- Ví: `tiền mặt` (từ dẫn rỗng, vì `rút` không phải từ dẫn) và `vcb` (từ dẫn `{atm, từ}`).
- Ví nguồn là vcb. Không có `sang`/`vào` nhưng có `rút`, nên ví đích là cash.
- Kết quả là chuyển khoản. `rút` bị bỏ, ghi chú là `nil`.

### Cú pháp Swift đáng chú ý
- `inout Clause` và `&clause`: hàm sửa trực tiếp biến của nơi gọi.
- `for … where …`: lọc ngay trong vòng lặp.
- `category.map { … } ?? fallback`: có giá trị thì bọc lại, không có thì dùng mặc định.
- So sánh tuple `(a1, b1) > (a2, b2)`: so lần lượt từng thành phần.
- `zip`: ghép khoản tiền với khoảng token, ghép ví với từ dẫn.

## Đọc thêm
- Quy ước dùng chung: `docs/shared/parser/README.md`, mục 2 (ví), mục 3 (danh mục), mục 5 (ghi chú), mục 6 (tách khoản), mục 7 (từ điển).
- [Apple: Bundle](https://developer.apple.com/documentation/foundation/bundle)
- [Swift: In-Out Parameters](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions/#In-Out-Parameters)
