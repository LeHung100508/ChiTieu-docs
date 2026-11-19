# Rà soát iS0-06 · Design system: component

Ngày rà soát: 2026-11-19 · Nhánh `task/iS0-06-ds-components` · Trạng thái: **Xong** (đóng 2026-11-19 sau khi rà lại lần 1)

> Bản rà soát tự viết trước đó (ghi "Xong") đã được thay bằng bản này. Theo luật, mentor rà soát và đổi trạng thái.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ✅ exit 0 |
| `swiftlint lint --strict` | ✅ exit 0 |
| `cd Core && swift test` | ✅ 3 test DesignSystem qua |
| `cd Core && xcodebuild -scheme Core-Package -destination 'platform=iOS Simulator,name=iPhone 18 Pro' test` | ✅ exit 0 |
| App: `xcodebuild … build` | ✅ exit 0 |
| Đối chiếu `docs/mockup/ios-home.html` | ❌ Thẻ tổng và màu số tiền lệch mockup (Bắt buộc 2, Nên sửa 3) |
| Tự tính tương phản chip chưa chọn (`Color.secondary` trên `secondarySystemFill`) | ❌ Sáng 3,07–3,19, dưới 4,5 |
| Chạy thử `Int64.formatted(.currency(code: "VND").locale(vi_VN))` | `-62.000 ₫`: dấu `-` là U+002D, khoảng trắng là U+00A0 |

## Bắt buộc sửa (2)

### 1. Vùng chạm của `CategoryChip` chưa đạt 44 pt
- **Vị trí:** `Core/Sources/DesignSystem/CategoryChip.swift:45-62`.
- **Vấn đề:**
  - `.frame(minHeight: 44)` và `.contentShape(Rectangle())` đặt **bên ngoài** `Button`. Hai modifier này chỉ làm khung bố cục bao quanh nút to ra. Vùng nhận chạm của `Button` vẫn bằng label của nó.
  - Label cao khoảng 22 pt (một dòng `body`) cộng 2 × 8 pt padding, tức khoảng 38 pt. Phần còn thiếu đến 44 pt là khoảng trống, bấm vào đó không có tác dụng.
- **Vì sao:** "Vùng chạm ≥ 44 pt" là tiêu chí của task và nằm trong checklist 13.8.
- **Hướng sửa:**
  - Đưa `frame(minHeight:)` và `contentShape` vào **trong** label của `Button`, sau phần nền capsule. Cách này giữ capsule nhỏ như cũ mà vùng chạm vẫn đủ 44 pt.
  - Tự kiểm tra: bật Accessibility Inspector, hoặc tạm thêm `.border(.red)` trong Preview để thấy khung bấm thật.
  - Sửa luôn mục 2 trong file lý thuyết (snippet và bảng so sánh), ghi rõ là với `Button` phải đặt trong label.

### 2. Chưa có "thẻ tổng" như mockup
- **Vị trí:** `Core/Sources/DesignSystem/SummaryCard.swift`.
- **Vấn đề:**
  - Trong `ios-home.html`, thẻ tổng là thẻ đầu màn Tổng quan, gồm:
    - nhãn "Đã chi tháng này";
    - số tiền cỡ hero `4.250.000 ₫`;
    - dòng phụ "Hôm nay 62.000 ₫";
    - thanh tiến độ kèm "68% ngân sách".
  - `SummaryCard` hiện tại lại là thẻ ngân sách theo **danh mục**: icon, tên, %, "trên hạn mức". Thẻ này gần với màn Ngân sách (iS3) hơn.
- **Vì sao:** task S0-08 và iS0-06 đều yêu cầu component "thẻ tổng". Màn Tổng quan ở iS2 sẽ dùng nó. Checklist mục 6: "giao diện khớp mockup".
- **Hướng sửa:**
  - Làm thẻ tổng theo mockup: nhận nhãn, số tiền chính, dòng phụ và tỉ lệ ngân sách (có thể không có), và dùng lại `AmountText(.hero)`.
  - Thẻ ngân sách theo danh mục giữ lại được nhưng nên đổi tên cho đúng nghĩa (ví dụ `BudgetProgressCard`). Cũng có thể bỏ, để làm ở iS3.

## Nên sửa (5)

### 1. Định dạng tiền gắn cứng `vi_VN`, lại bị chép ở 3 chỗ
- **Vị trí:** `AmountText.swift:43`, `SummaryCard.swift:56` và `:61`, `DesignSystemTests.swift:52`.
- **Vì sao:**
  - NFR-12 của đặc tả iOS ghi: "Tiền định dạng bằng FormatStyle **theo locale**". Gắn cứng `vi_VN` thì người dùng tiếng Anh vẫn thấy định dạng Việt.
  - Cùng một biểu thức xuất hiện 3 lần, nên mỗi lần đổi phải sửa 3 chỗ.
- **Hướng sửa:**
  - Gom phần định dạng vào **một** chỗ, ví dụ một hàm hoặc một `FormatStyle` tĩnh trong `DesignSystem`, nhận `Locale` làm tham số.
  - Trong view, lấy locale từ `@Environment(\.locale)`. Đơn vị tiền vẫn cố định là `VND`.
  - Sửa luôn "Lỗi hay gặp 1" và "Khái niệm chính" trong file lý thuyết.

### 2. Test không kiểm tra code của dự án
- **Vị trí:** `DesignSystemTests.swift:50-55`.
- **Vì sao:** test tự gọi lại `formatted(.currency…)`, tức là đang kiểm tra Foundation chứ không kiểm tra component. Component có định dạng sai thì test vẫn xanh. Checklist mục 6: "test đúng hành vi chứ không chỉ chạy qua".
- **Hướng sửa:**
  - Test hàm định dạng chung ở Nên sửa 1 với các ca: số âm, 0, số lớn, `vi_VN` và `en_US`.
  - Tách logic thuần của thẻ thành hàm `internal` (tỉ lệ, phần trăm, mức cảnh báo) để test được qua `@testable import`. Cần các ca biên: hạn mức 0, đúng 80%, đúng 100%, vượt 100%.

### 3. `AmountText` tự tô đỏ số âm, khác mockup
- **Vị trí:** `AmountText.swift:46-54`.
- **Vấn đề:**
  - Mockup Tổng quan hiển thị khoản chi `−62.000 ₫` bằng màu chữ thường, không phải màu đỏ.
  - Component đang suy nghĩa (chi hay thu) từ dấu của số. Trong khi đó, model sau này có trường loại giao dịch riêng.
- **Hướng sửa:**
  - Bỏ phần tự tô đỏ theo dấu. Màu mặc định là `textPrimary`, nơi gọi tự truyền màu khi cần, ví dụ thu nhập dùng `statusIncome`.
  - Mockup dùng dấu trừ "−" (U+2212), còn FormatStyle cho ra "-" (U+002D). Chỗ này có thể hoãn, nhưng nên ghi lại.

### 4. Chip chưa chọn thiếu tương phản ở chế độ sáng
- **Vị trí:** `CategoryChip.swift:36`.
- **Vì sao:** `Color.secondary` đặt trên `secondarySystemFill` chỉ đạt **3,19** trên nền trắng và **3,07** trên `#F2F2F7`, dưới ngưỡng 4,5 của WCAG AA cho chữ thường. Ở chế độ tối thì đạt (4,6 và 5,5).
- **Hướng sửa:** dùng `textPrimary` cho chữ của chip chưa chọn (tương phản trên 15). Nếu muốn chip nhạt hơn, chỉ làm nhạt phần icon.

### 5. Chuỗi hard-code trong component
- **Vị trí:**
  - `AmountText.swift:58-60`: "Âm … đồng".
  - `SummaryCard.swift:62`: "đã chi … trên hạn mức … phần trăm".
  - `SummaryCard.swift:101`: "trên …".
- **Vì sao:** định nghĩa Hoàn thành chung của kế hoạch iOS áp dụng từ iS0: "mọi chuỗi nằm trong String Catalog, có đủ tiếng Việt và tiếng Anh".
- **Hướng sửa:**
  - Có thể ghi `hoãn` sang iS1, vì String Catalog được học ở iS1. Lúc đó cần thêm `defaultLocalization` cho package và `.xcstrings` dùng `bundle: .module`.
  - Trước khi hoãn, cân nhắc bỏ nhãn tự chế cho `AmountText`. VoiceOver đã đọc chuỗi tiền theo locale. Hãy bật VoiceOver hoặc Accessibility Inspector để nghe thử rồi mới quyết định, vì "Lỗi hay gặp 3" trong file lý thuyết chưa được kiểm chứng.

## Gợi ý thêm để học (6)
1. **`ProgressView(value:)`** kèm `.tint(...)` có thể thay cho `GeometryReader` tự vẽ. VoiceOver tự đọc được giá trị phần trăm, và code ngắn hơn.
2. **Chữ trên nền nhấn bị lặp:** biểu thức `colorScheme == .dark ? .black : .white` có ở `CategoryChip.swift:34` và `DesignSystemPreview.swift:14`. Đây đúng là lúc nên đưa nó thành token (`AppColor.onFill`, hoặc một color asset có hai biến thể sáng/tối), như đã nhắc ở iS0-05.
3. **`abs(amount)`** sẽ crash với `Int64.min`. Dùng `amount.magnitude` thì an toàn.
4. **`CategoryIconView`:**
   - Trong mockup, ô icon có nền **đặc** theo màu từng danh mục và icon trắng (`#0062CC`, `#B25000`, …), không phải nền tint.
   - Cỡ cố định 40 pt và `.system(size:)` không co giãn theo Dynamic Type. Có thể dùng `@ScaledMetric`.
5. **Thiếu Preview AX5** cho thẻ và chip. Hàng icon, tên và % dễ bị cắt chữ ở cỡ chữ lớn nhất. `ViewThatFits` hoặc chuyển sang `VStack` khi `dynamicTypeSize.isAccessibilitySize` là hai cách xử lý thường gặp.
6. **Code thừa:**
   - `.isButton` trong `accessibilityAddTraits` là thừa, vì `Button` đã có sẵn trait này.
   - `DesignSystem.swift` vẫn còn `DesignSystemModule` (đã nhắc ở iS0-05).

## Điểm làm tốt
- Bốn component đều `public`, có `public init`, đặt mỗi cái một file và có `#Preview` sáng và tối. Lint `--strict` sạch.
- `AmountText` dùng lại `amountStyle` và token chữ của iS0-05. Chip có trait `.isSelected`. Thẻ dùng `.accessibilityElement(children: .combine)`: hướng tiếp cận VoiceOver đúng.
- Thanh tiến độ có giới hạn từ 0 đến 1 và có 3 ngưỡng màu, xử lý được trường hợp hạn mức bằng 0.

## Kết luận ban đầu
Chưa đóng được task. Cần sửa Bắt buộc 1–2 và Nên sửa 1–4. Nên sửa 5 có thể ghi `hoãn`. Sửa xong thì báo lại, mentor sẽ rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-19)

> Bản "Rà lại lần 1" tự viết (kết luận "Đóng được") đã được thay bằng bản này.

**Lệnh đã chạy (mentor tự chạy):**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `cd Core && swift test` → ✅ 4 test DesignSystem qua.
- `xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) → ✅.
- App build → ✅.

**Đã đạt:**
- **Bắt buộc 1:** ✅ `frame(minHeight: 44)` và `contentShape` đã nằm trong label (`CategoryChip.swift:58-59`). Capsule vẫn nhỏ, vùng chạm đủ 44 pt.
- **Bắt buộc 2:** ✅ `SummaryCard` mới khớp `ios-home.html`: nhãn, số tiền hero, dòng phụ, thanh tiến độ và nhãn ngân sách. Thẻ ngân sách theo danh mục đã đổi tên thành `BudgetProgressCard`.
- **Nên sửa 3:** ✅ Màu mặc định là `textPrimary`, dùng `amount.magnitude`.
- **Nên sửa 4:** ✅ Chip chưa chọn dùng `textPrimary`.
- **Nên sửa 5:** ✅ Đã ghi hoãn vào Nợ kỹ thuật.

**Còn lại (Nên sửa):**
1. **Test vẫn chưa chạm vào code mà view đang dùng.**
   - Vị trí: `BudgetProgressCard.swift:34-53` và `:157-175`.
   - Ba hàm `static` (`calculateRatio`, `calculatePercentage`, `determineProgressColor`) chỉ có test gọi tới. Còn view lại dùng bản sao riêng của nó (`ratio`, `percentage`, `progressColor`).
   - Nếu ai đó đổi ngưỡng trong view, test vẫn xanh. Đây vẫn đúng là vấn đề của Nên sửa 2.
   - `SummaryCard.swift:33-42` chép thêm một bản ngưỡng 0,8 và 1,0 thứ ba.
   - Hướng sửa: giữ **một** nơi định nghĩa ngưỡng và cho cả hai thẻ cùng gọi. Có thể là một hàm `static`, hoặc một `enum` mức ngân sách (an toàn, sắp vượt, vượt) có thuộc tính màu. Sau đó bỏ các bản sao.
2. **Còn một chỗ gắn cứng `vi_VN`.** `BudgetProgressCard.swift:62` vẫn tự gọi `.locale(Locale(identifier: "vi_VN"))`, không dùng `formattedVND(locale:)`.
3. **Test định dạng còn lỏng.** `DesignSystemTests.swift`:
   - Ca số âm chỉ kiểm `contains("62.000")`. Nếu mất dấu trừ, test vẫn qua.
   - Ca `en_US` chỉ kiểm `!isEmpty`. Hàm trả về chuỗi gì test cũng qua.
   - Ca số 0 kiểm `contains("0")`. Chuỗi "1.000 ₫" cũng qua.
   - Hướng sửa: kiểm có dấu trừ. Kiểm dấu phân cách theo locale: `vi` dùng `.`, `en` dùng `,`, ví dụ "45,000". Nếu so sánh nguyên chuỗi thì lưu ý khoảng trắng trước "₫" là U+00A0.
4. **File lý thuyết chưa sửa** (`hoc/iS0-06-design-system-component.md`):
   - Dòng 7, 19 và 63 vẫn khuyên dùng locale `vi_VN`, trái với NFR-12.
   - Snippet 2 (dòng 34-41) chưa ghi rõ rằng với `Button`, phải đặt `frame` và `contentShape` trong label. Đó chính là lỗi của Bắt buộc 1.

**Gợi ý nhỏ:**
- Còn vài comment cũ:
  - `AmountText.swift:46` ghi "số âm dùng statusDanger".
  - `CategoryChip.swift:31` ghi "textSecondary".
  - Header của `BudgetProgressCard.swift` vẫn ghi `SummaryCard.swift`.
- `SummaryCard` không có Preview trường hợp `budgetRatio == nil`, tức là chưa đặt ngân sách.

**Kết luận:** chưa đóng được task. Không còn mục Bắt buộc. Cần sửa 4 mục Nên sửa ở trên, hoặc ghi `hoãn <mục>`.

## Rà lại lần 2 (2026-11-19)
**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅ 4 test.
- `xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) → ✅.
- App build → ✅.

**Kết quả từng mục:**
1. ✅ `BudgetProgressCard` gọi các hàm `static` mà test đang kiểm tra. `SummaryCard` dùng lại `determineProgressColor`. Như vậy chỉ còn một nơi định nghĩa ngưỡng.
2. ✅ Không còn `vi_VN` trong code của component. Nhãn VoiceOver cũng dùng `formattedVND(locale:)`.
3. ✅ Test định dạng đã kiểm tra dấu trừ, so sánh nguyên chuỗi với số 0 và kiểm dấu phân cách `,` cho `en_US`.
4. ✅ File lý thuyết bỏ lời khuyên gắn cứng `vi_VN`, và ghi rõ với `Button` phải đặt `frame` và `contentShape` trong label.

**Còn lại (chỉ là gợi ý, không chặn việc đóng task):**
- Comment cũ ở `AmountText.swift:46` và `CategoryChip.swift:31`.
- Header của `BudgetProgressCard.swift` vẫn ghi tên `SummaryCard.swift`.
- `SummaryCard` chưa có Preview cho trường hợp chưa đặt ngân sách.

**Kết luận:** Đóng được task.
