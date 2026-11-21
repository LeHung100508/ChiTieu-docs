# Rà soát iS0-11 · Parser: danh mục và tách khoản (FR-04, FR-07)

Ngày rà soát: 2026-11-21 · Nhánh `task/iS0-11-danh-muc` · Trạng thái: **Xong** (đóng 2026-11-21 sau rà lại lần 3)

> Task giải thuật theo mục 2 của luật. Agent viết `ClauseSplitter`, `CategoryDictionary`, `WalletMatcher`, `ClauseAnalyzer` và test của chúng. Người dùng làm ba việc: đưa từ điển vào app, nối vào `TransactionParser`, và kiểm các trường trong test 200 câu.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ✅ exit 0 |
| `swiftlint lint --strict` | ✅ exit 0 |
| `cd Core && swift test` | ❌ **Lỗi biên dịch**: `invalid redeclaration` của `testParameterizedAllCases` |
| App: `xcodebuild … build` | ✅. Đã kiểm tra trong `ChiTieu.app` có `category_keywords.json` |
| Tìm `CategoryDictionary`, `ParserConfiguration`, `Bundle.main` trong `ChiTieu/` và `Features/` | ❌ Không có kết quả nào |

## Bắt buộc sửa (2)

### 1. Test không biên dịch được
- **Vị trí:** `Core/Tests/ParserTests/ParserTests.swift`. Hàm `testParameterizedAllCases` bản cũ vẫn còn, và có thêm một hàm mới trùng tên.
- **Vì sao:** cả target `ParserTests` không build được, nên CI sẽ đỏ ở bước `Run Core Tests`. Rất có thể bạn đã báo xong mà chưa chạy lại `swift test`.
- **Hướng sửa:** xóa bản cũ, chỉ giữ bản mới. Sau đó chạy `swift test` trong `Core/`.

### 2. App chưa đọc từ điển khi chạy
- **Vấn đề:** file JSON đã nằm trong bundle (bước A đúng), nhưng không có dòng code nào trong app đọc nó ra `CategoryDictionary`.
- **Vì sao:** tiêu chí xong của task ghi "app đọc được từ điển khi chạy". Mới có file trong bundle thì chưa chứng minh được điều đó.
- **Hướng sửa:**
  - Ở chỗ lắp ráp của app (`ChiTieuApp` hoặc một hàm tạo phụ thuộc), đọc file qua `Bundle.main.url(forResource:withExtension:)`, rồi tạo `CategoryDictionary(data:)` và `ParserConfiguration`.
  - Xử lý lỗi rõ ràng, không dùng `try!`.
  - Để tự kiểm tra, cho hiện `dictionary.version` ở một chỗ (ví dụ màn Cài đặt), hoặc in ra log bản debug. Mentor sẽ chạy app trên simulator để xem.

## Nên sửa (2)

### 1. `configuration` là Optional, có nhánh dự phòng
- **Vị trí:** `Parser.swift:13-15` và `:60-73`.
- **Vì sao:**
  - Nếu app quên truyền `configuration`, parser vẫn chạy nhưng **âm thầm** bỏ trống loại, danh mục, ví và ghi chú. Lỗi kiểu này khó phát hiện.
  - Đoạn tạo `ParsedTransaction` bị viết lặp hai lần.
- **Hướng sửa:** đổi thành `init(configuration: ParserConfiguration)`, không Optional, và bỏ nhánh `else`. Thiếu cấu hình thì compiler báo lỗi ngay.

### 2. Lịch đến từ hai nguồn
- **Vị trí:** `Parser.swift:31-38`.
- **Vấn đề:** `DateParser` dùng một `Calendar` tạo từ tham số `timeZone`, còn `ClauseAnalyzer` dùng `configuration.calendar`. Nếu hai múi giờ khác nhau, thời gian và việc nhận diện cụm thời gian (để bỏ khỏi ghi chú) sẽ lệch nhau.
- **Hướng sửa:** dùng `configuration.calendar` cho cả hai, và bỏ tham số `timeZone` khỏi `parse`. Chỉ giữ tham số thời điểm hiện tại.

## Câu hỏi kiểm tra hiểu (mục 2 của luật)
Năm câu trả lời của bạn đều **đúng nội dung**, nhưng một phần không khớp với năm câu đã hỏi:

| Câu đã hỏi | Đánh giá |
| --- | --- |
| 1. Vì sao `thu tiền phòng trọ` ra thu nhập chứ không phải nhà ở | ⏳ Chưa trả lời |
| 2. Vì sao `mua rau và thịt` không tách, còn `mua bánh mì 20k và cà phê 25k` thì tách; vì sao `và` bị bỏ khỏi ghi chú ở câu sau | ⚠️ Mới đạt nửa đầu (câu trả lời số 1 của bạn). Chưa giải thích vì sao `và` bị bỏ |
| 3. `quẹt thẻ tín dụng` | ✅ Đạt |
| 4. `ăn tối 300k chuyển khoản vcb` khác `chuyển 2tr từ VCB sang ví tiền mặt` ở đâu | ⚠️ Câu trả lời số 4 của bạn giải thích đúng luật "phải có đủ ví nguồn và ví đích" qua câu `rút tiền`, nhưng chưa so sánh hai câu đã hỏi |
| 5. Vì sao `CategoryDictionary` nhận `Data` thay vì tự đọc bằng `Bundle` | ⏳ Chưa trả lời |
| (thêm) Từ đơn ngắn phải có dấu; ghi chú cắt từ `raw` | ✅ Đúng, có ví dụ `4g` và `Grab` |

## Điểm làm tốt
- Bước A đúng như đề xuất: file JSON được thêm vào target app dạng tham chiếu, không chép ra chỗ khác, và đã có trong bundle sau khi build.
- `zip(amounts, zip(dates, details))` không phụ thuộc vào chỉ số, đúng như gợi ý ở iS0-10.
- Test mới kiểm chặt đủ 6 trường, có so ghi chú theo README mục 5.7.

## Kết luận
Chưa đóng được task. Cần sửa Bắt buộc 1–2 và Nên sửa 1–2 (hoặc ghi `hoãn <mục>`), và trả lời câu 1, câu 5, nửa sau câu 2, và phần so sánh ở câu 4.

---

## Rà lại lần 1 (2026-11-21)
**Lệnh đã chạy:**
- swift-format `--strict` → ❌ `ParserTests.swift:47`: còn khoảng trắng thừa ở cuối dòng.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅ 34 test qua.
- `xcodebuild … test` (iPhone 18 Pro) → ✅.
- App build cho simulator → ✅.

**Đã đạt:**
- **Bắt buộc 1:** ✅ Chỉ còn một hàm `testParameterizedAllCases`, test biên dịch được và chạy qua.
- **Bắt buộc 2:** ✅ Về code. `loadParserConfiguration()` đọc file qua `Bundle.main`, tạo `CategoryDictionary` và `ParserConfiguration`, không dùng `try!`, và hiện "Từ điển v1" trên thanh công cụ. Mentor chưa xem trên simulator được vì chưa được cấp quyền dùng thiết bị.
- **Câu hỏi kiểm tra:** ✅ Cả 4 ý còn thiếu đều đạt:
  - câu 1: cụm 4 token thắng cụm 3 token;
  - câu 2: token ranh giới bị đánh dấu `claimed`;
  - câu 4: thiếu cặp ví nguồn và ví đích, `chuyển khoản` chỉ là từ dẫn ví;
  - câu 5: tách logic khỏi môi trường; `Bundle.main` của test runner không có tài nguyên của app.

**Còn lại:**
1. **Bắt buộc:** sửa khoảng trắng thừa ở `ParserTests.swift:47`. Nếu không, bước lint trên CI sẽ đỏ.
2. **Nên sửa 1 và 2 của lần trước** (`configuration` Optional; lịch đến từ hai nguồn) chưa được sửa. Sửa, hoặc ghi `hoãn`.
3. **Nên sửa mới:** `loadParserConfiguration()` gặp lỗi thì trả `nil` mà không ghi lại gì. Nếu file bị hỏng, nhãn phiên bản chỉ lặng lẽ biến mất. Nên ghi lỗi bằng `Logger` (`os`) để nhìn thấy được lỗi.
4. **Gợi ý:**
   - Nhãn "Từ điển v1" là giao diện để kiểm tra, nên để trong `#if DEBUG` hoặc chuyển vào màn Cài đặt.
   - Việc đọc file nên đặt ở chỗ lắp ráp của app (`ChiTieuApp`), rồi truyền xuống. Đặt trong giá trị khởi tạo của `@State` thì biểu thức đó chạy lại mỗi lần SwiftUI tạo lại `ContentView`.

## Rà lại lần 2 (2026-11-21)
**Lệnh đã chạy:**
- swift-format `--strict` → ❌ 12 lỗi:
  - `ChiTieuApp.swift:10` `[OrderedImports]`;
  - `ContentView.swift:39-49` `[Indentation]`, nằm trong khối `#if DEBUG`.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅ 34 test.
- `xcodebuild … test` (iPhone 18 Pro) → ✅.
- App build → ✅.
- **Chạy app trên simulator iPhone 18 Pro** → ✅ Thanh công cụ hiện **"Từ điển v1"**. App đọc được từ điển khi chạy, đạt tiêu chí xong.

**Đã đạt:**
- Khoảng trắng thừa ở `ParserTests.swift:47` đã sửa.
- **Nên sửa 1:** ✅ `TransactionParser(configuration:)` không còn Optional, đã bỏ nhánh dự phòng.
- **Nên sửa 2:** ✅ Đã bỏ tham số `timeZone`. `DateParser` và `ClauseAnalyzer` cùng dùng `configuration.calendar`.
- **Nên sửa mới:** ✅ Việc đọc file đã chuyển lên `ChiTieuApp`, ghi lỗi bằng `Logger` ở cả hai nhánh lỗi.
- **Gợi ý:** ✅ Nhãn phiên bản nằm trong `#if DEBUG`.

**Còn lại (Bắt buộc):** swift-format còn đỏ, nên CI sẽ chặn PR.
- **Import:** swift-format so thứ tự có phân biệt hoa thường, chữ hoa đứng trước. Vì vậy `import os` phải nằm **sau** `import SwiftUI`.
- **Thụt lề:** phần bên trong `#if DEBUG … #endif` cần thụt thêm 4 dấu cách so với dòng `#if`.

Lần thứ hai liên tiếp báo xong mà swift-format vẫn đỏ. Trước khi báo xong, hãy chạy lệnh lint chuẩn.

**Gợi ý nhỏ:** `subsystem` của `Logger` nên lấy từ `Bundle.main.bundleIdentifier`, thay vì ghi cứng một chuỗi.

**Kết luận:** đóng được task ngay khi swift-format sạch.

## Rà lại lần 3 (2026-11-21)
**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅ 34 test.
- App build → ✅.

Thứ tự import và phần thụt lề trong `#if DEBUG` đã sửa xong. Các tiêu chí xong đều đạt:
- test danh mục và tách khoản trên bộ mẫu xanh (200/200, đủ 6 trường);
- app đọc được từ điển khi chạy (đã xem "Từ điển v1" trên simulator ở lần rà thứ 2).

**Kết luận:** Đóng được task.
