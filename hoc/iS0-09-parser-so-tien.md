# iS0-09 · Parser: số tiền (FR-02)

> Task giải thuật theo mục 2 của luật: `AmountParser` và test của nó do agent viết. Người dùng viết phần nối vào `TransactionParser.parse` và khung test 200 câu.

## Vì sao cần
Người Việt viết số tiền theo rất nhiều kiểu: `45k`, `45.000`, `50000đ`, `1tr2`, `1,5tr`, `2 củ rưỡi`, `5 lít`, `năm chục`. `AmountParser` đọc luồng `Token` của iS0-08 và trả về từng khoản tiền (`Int64` đồng), kèm vị trí và độ tin cậy. Nhờ có vị trí, các bước sau biết phần nào của câu là số tiền, để không đưa nhầm phần đó vào ghi chú.

## Khái niệm chính
- **Hệ số và ký hiệu là hai thứ khác nhau.**
  - Hệ số **nhân** vào số: `k`/`nghìn`/`ngàn` ×1.000; `tr`/`triệu`/`củ` ×1.000.000; `lít`/`loét` ×100.000; `chục` ×10.000; `trăm` (nói tắt trăm nghìn) ×100.000.
  - Ký hiệu tiền tệ `đ`/`đồng`/`vnd` **không nhân** gì cả. Nó chỉ xác nhận rằng số đứng trước là tiền: `50000đ` = 50.000.
- **Khớp theo thứ tự ưu tiên.** Tại mỗi token, thử lần lượt các mẫu trong bảng dưới. Mẫu dài, cụ thể đứng trước mẫu ngắn, chung chung. Mẫu nào khớp thì tiêu thụ token của nó.

  | # | Mẫu | Ví dụ |
  | --- | --- | --- |
  | 1 | `<số>k<số>` viết liền | `85k5` = 85.500 |
  | 2 | `<số> tr/củ <số>` | `1tr2`, `1 củ 8`, `một triệu hai`, `2tr500k` |
  | 3 | `<chữ số> trăm [rưỡi]` | `hai trăm rưỡi` = 250.000 |
  | 4 | `<số> <đơn vị> rưỡi` | `1 củ rưỡi`, `1 lít rưỡi`, `70 nghìn rưỡi` |
  | 5 | `nửa củ/triệu` | 500.000 |
  | 6 | `<số> chục` | `năm chục` = 50.000 |
  | 7 | `<số> <đơn vị> [đ]` | `45k`, `1,5tr`, `5 lít`, `45 nghìn đồng` |
  | 8 | Số không đơn vị `[đ]` | `450.000`, `350000`, `50000đ` |

- **Phần lẻ tính theo số chữ số**, không theo giá trị. Sau `tr`: `2` → 200.000, `05` → 50.000, `250` → 250.000. Sau `k`: `5` → 500, `05` → 50.
- **Dấu chấm hay dấu phẩy là gì?** Nếu mọi nhóm sau dấu đều có 3 chữ số thì đó là phân cách hàng nghìn (`45.000`, `1,250,000`). Nếu chỉ có một dấu và không phải hàng nghìn thì đó là số thập phân (`1,5tr`, `45.5k`). Với đơn vị lớn thì ưu tiên hiểu là thập phân: `1.250tr` = 1.250.000.
- **Tránh nhận nhầm:**
  - `45k 2 ly`: chỉ ghép `k` với số đứng **liền** sau nó.
  - `3tr 2 người`: số theo sau là từ đếm, nên là số lượng.
  - `ngày 15`: số chỉ ngày.
  - Số trần dưới 1.000, hoặc bắt đầu bằng `0` (số điện thoại), không phải số tiền.
- **Không bao giờ crash.** Mọi phép tính dùng `multipliedReportingOverflow` / `addingReportingOverflow`. Khoản lớn hơn `maxAmount` (dưới 1.000 tỷ) bị bỏ qua.
- **Độ tin cậy** theo mức rõ ràng của cách viết:

  | Mức | Cách viết | Ví dụ |
  | --- | --- | --- |
  | 0,95 | Đơn vị chuẩn hoặc ký hiệu tiền tệ | `45k`, `50000đ` |
  | 0,9 | Số có phân cách hàng nghìn | `450.000` |
  | 0,8 | Tiếng lóng | `2 củ`, `năm chục` |
  | 0,6 | Số trần | `350000` |
  | 0,4 | Tiếng lóng dễ nhầm nghĩa đen | `5 lít xăng` |

## So với cách cũ bạn quen
| Cũ (Kotlin/Android) | Mới (Swift) | Khác biệt chính |
| --- | --- | --- |
| `Long` tràn số âm thầm quay vòng | `Int64` tràn thì **trap** (crash) | Với đầu vào từ người dùng phải dùng `…ReportingOverflow` |
| `when (unit) { in setOf(...) -> }` | `Set<String>.contains` + chuỗi `??` | `a ?? b ?? c` thử lần lượt các mẫu, mẫu đầu tiên khác `nil` thắng |
| `Pair<ParsedAmount, Int>` | Tuple có nhãn `(amount:, consumed:)` | Đọc rõ nghĩa hơn `.first`, `.second` |
| `BigDecimal("1.5") * 1_000_000` | `mantissa / 10^scale`, toàn số nguyên | Không dùng `Double`, nên không có sai số làm tròn |

## API và cú pháp cần biết
- `multipliedReportingOverflow(by:)` trả về `(partialValue, overflow)`. Dùng để kiểm tra tràn số mà không crash.
- Chuỗi `??` với các hàm trả về Optional: thử lần lượt từng mẫu.
- `guard … let …, let … else { return nil }`: gom nhiều điều kiện và nhiều bước đổi kiểu vào một chỗ.

```swift
// Ví dụ khác bài toán: cộng điểm thưởng an toàn
func addPoints(_ a: Int64, _ b: Int64) -> Int64? {
    let (sum, overflow) = a.addingReportingOverflow(b)
    return overflow ? nil : sum
}
let bonus = parseCode(text) ?? parseQR(text) ?? defaultBonus
```

## Lỗi hay gặp
1. **Coi `đ`/`vnd` là hệ số ×1.000.** `50000đ` sẽ thành 50 triệu. Câu `150.000 đ` vẫn "đúng" do trùng hợp, nên bộ mẫu không phát hiện được lỗi này.
2. **Ghép số lượng vào số tiền.** `45k 2 ly` thành 45.200. Cần kiểm tra số phụ có viết liền với đơn vị không.
3. **Tính phần lẻ theo giá trị.** `1tr05` thành 1.500.000. Phải tính theo số chữ số của phần lẻ.
4. **Phép nhân `Int64` không kiểm tra tràn.** Một chuỗi số dài làm crash app.
5. **Đúng 100% bộ mẫu chưa chắc đã đúng.** Phải thử thêm câu ngoài bộ mẫu. Các câu đó nằm trong `AmountParserTests.outOfSampleCases`.

## Giới hạn đã biết
- Chưa đọc số bằng chữ có `mươi` (`hai mươi nghìn`, `mười lăm nghìn`). Bộ mẫu chưa có câu nào kiểu này.
- `lít` luôn được hiểu là 100.000 đ, kể cả `5 lít xăng`. Khi đó chỉ hạ độ tin cậy xuống 0,4 để màn xác nhận tô sáng.
- Năm (`2025`) sẽ bị đọc như số trần, với độ tin cậy 0,6.

## Đọc thêm
- [Swift: Integers, overflow operators](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/advancedoperators/#Overflow-Operators)
- [`FixedWidthInteger.multipliedReportingOverflow(by:)`](https://developer.apple.com/documentation/swift/fixedwidthinteger/multipliedreportingoverflow(by:))
- Bộ câu mẫu và quy ước dùng chung: `docs/shared/parser/README.md`
