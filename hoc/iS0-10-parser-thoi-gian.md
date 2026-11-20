# iS0-10 · Parser: thời gian (FR-03)

> Task giải thuật theo mục 2 của luật: `DateParser` và test của nó do agent viết. Người dùng nối nó vào `TransactionParser.parse`, chuyển phần kiểm `datetime` ra khỏi `withKnownIssue`, và trả lời câu hỏi kiểm tra.

## Vì sao cần
Người dùng ghi lại khoản chi đã xảy ra, và thường nói kèm thời gian: `hôm qua`, `tối qua`, `sáng nay`, `thứ 2 tuần trước`, `ngày 5`, `15/10`. Parser cần đổi các cụm này thành một `Date` chính xác theo múi giờ `Asia/Ho_Chi_Minh`. Kết quả phải ổn định khi chạy test, kể cả khi thời điểm hiện tại rơi vào lúc qua nửa đêm hay đầu tháng.

## Khái niệm chính
- **Inject `now` và `Calendar`.** `DateParser` không tự gọi `Date()`. Thời điểm hiện tại và lịch (đã đặt múi giờ) được truyền từ ngoài vào. Nhờ vậy test có thể đặt "bây giờ" là 00:30 ngày 1/11 để kiểm tra các trường hợp biên.
- **Hai loại cụm thời gian:**
  - `.day`: có ngày rõ ràng. Ví dụ: `hôm qua`, `hôm kia`, `ngày 5`, `15/10`, `thứ 2 tuần trước`, `sáng nay`, `tối qua`, `sáng hôm qua`.
  - `.period`: chỉ có buổi, đứng một mình **ở đầu khoản**. Ví dụ: `sáng phở 45k`. Nếu buổi nằm giữa khoản (`ăn sáng 30k`, `cơm trưa 40k`) thì đó là tên món, không phải thời gian.
- **Giờ của từng buổi:** sáng 08:00, trưa 12:00, chiều 15:00, tối 20:00, đêm 22:00. Cụm chỉ có ngày thì giữ nguyên giờ phút của `now`.
- **Chống ngày tương lai:**
  - `ngày 25` khi hôm nay là ngày 20 → tháng trước.
  - `15/12` khi đang ở tháng 1 → năm trước.
  - Có năm mà vẫn rơi vào tương lai → bỏ qua cụm đó.
  - Ngày không tồn tại (`30/2`, `ngày 31` của tháng có 30 ngày) → bỏ qua, không ném lỗi.
- **Lan ngữ cảnh trong câu nhiều khoản.** Câu được chia thành từng khoản theo khoản tiền. Ranh giới là dấu `,` `;`, xuống dòng, `và`, `với` nằm giữa hai khoản tiền. Sau đó:
  1. Khoản có cụm thời gian riêng thì dùng cụm đó.
  2. Cụm `.day` lan sang các khoản sau chưa có thời gian riêng.
  3. Khoản chỉ có buổi mà trước đó đã có ngày được lan thì ghép buổi vào ngày đó. Ví dụ: `hôm qua ăn sáng 35k, tối xem phim 100k` cho khoản thứ hai là tối hôm qua.
  4. Không có gì thì lấy `now`, với độ tin cậy 0,5.
- **So khớp có dấu hay không dấu.** Từ chỉ buổi phải so trên bản **có dấu**, vì khi bỏ dấu thì chúng trùng với từ khác: `sang` (chuyển sang), `toi` (tôi), `dem` (đếm). Các từ như `hôm`, `ngày`, `tuần`, `trước` thì không trùng, nên so trên bản không dấu để nhận cả cách gõ không dấu.

## So với cách cũ bạn quen
| Cũ (Android) | Mới (Swift) | Khác biệt chính |
| --- | --- | --- |
| `java.time.Clock` inject vào | Truyền `now: Date` và `Calendar` vào | Swift không có `Clock` cho ngày giờ trong Foundation. Truyền tham số là cách đơn giản nhất |
| `ZonedDateTime.minusDays(1)` | `calendar.date(byAdding: .day, value: -1, to: now)` | `Date` không có múi giờ. Mọi phép tính ngày phải đi qua `Calendar` |
| `LocalDate.of(y, m, d)` ném lỗi khi ngày sai | `DateComponents.isValidDate` | `calendar.date(from:)` tự **dồn** ngày sai: 31/9 thành 1/10. Phải kiểm tra trước |
| `DayOfWeek.MONDAY` | `calendar.component(.weekday, …)`: 1 là Chủ nhật | Phải tự đổi sang chỉ số mà thứ Hai là 0 |

## API và cú pháp cần biết
- `Calendar.date(byAdding:value:to:)`: cộng hoặc trừ theo đơn vị lịch, đúng cả khi qua tháng hay qua năm.
- `Calendar.date(bySettingHour:minute:second:of:)`: đặt giờ trên cùng một ngày.
- `DateComponents(calendar:timeZone:year:…)` và `.isValidDate`: kiểm tra ngày có tồn tại hay không.
- So sánh tuple: `(month, day) > (todayMonth, todayDay)` so lần lượt từng phần tử, như so từ điển.

```swift
// Ví dụ khác bài toán: hạn trả sách sau 14 ngày, lúc 17:00
let due = calendar.date(byAdding: .day, value: 14, to: borrowedAt)
let dueAt5pm = due.flatMap { calendar.date(bySettingHour: 17, minute: 0, second: 0, of: $0) }
let valid = DateComponents(calendar: calendar, year: 2026, month: 2, day: 30).isValidDate  // false
```

## Lỗi hay gặp
1. **Gọi `Date()` trong parser.** Test sẽ cho kết quả khác nhau tùy giờ chạy, và không kiểm tra được trường hợp nửa đêm.
2. **Tin `calendar.date(from:)` với ngày sai.** Hàm này trả về 1/10 cho ngày 31/9 thay vì trả `nil`.
3. **So `sang`, `toi` trên bản không dấu.** Khi đó `chuyển 2tr từ VCB sang ví` bị đọc thành "buổi sáng".
4. **Đọc năm thành số tiền.** Trong `15/10/2027`, năm 2027 bị `AmountParser` đọc thành 2.027 đồng. Đã sửa: số đứng cạnh dấu `/` là một phần của ngày.
5. **Coi `ăn sáng` là thời gian.** Buổi chỉ là thời gian khi nó đứng ở đầu khoản, hoặc đi kèm `nay`/`qua`/một ngày cụ thể.

## Giới hạn đã biết
- `1/2 kg thịt` sẽ bị đọc thành ngày 1/2 (độ tin cậy 0,8). Bộ mẫu chưa có câu dùng phân số.
- Chưa hiểu giờ cụ thể (`lúc 15h30`), `tuần trước` đứng một mình, `tháng trước`.
- Ngày tương lai có chủ đích (`mua vé xem phim ngày 30/10` khi hôm nay là 20/10) sẽ bị lùi về năm trước, theo luật chống ngày tương lai trong README.

## Đọc thêm
- [Apple: Calendar](https://developer.apple.com/documentation/foundation/calendar)
- [Apple: DateComponents.isValidDate](https://developer.apple.com/documentation/foundation/datecomponents/isvaliddate)
- Quy tắc thời gian dùng chung: `docs/shared/parser/README.md`, mục 4
