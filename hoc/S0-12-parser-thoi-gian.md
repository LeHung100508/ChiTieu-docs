# S0-12 · Parser: thời gian (FR-03)

## Vì sao cần
Trong chi tiêu thường nhật, người dùng thường ghi chép lại các khoản phát sinh từ trước như `hôm qua`, `sáng nay`, `tối qua`, `thứ 2 tuần trước`, hoặc ghi rõ ngày như `ngày 5`, `15/10`.
Thành phần `DateTimeParser` có nhiệm vụ quét luồng `Token` sau chuẩn hóa, nhận diện các biểu thức thời gian tương đối và tuyệt đối, quy đổi về mốc thời gian cụ thể (kiểu `LocalDateTime` hoặc `Instant` với múi giờ `Asia/Ho_Chi_Minh`), đồng thời tiêu thụ các token này để không bị sót lại vào tên món (`note`).

## Khái niệm chính
- **Mốc thời gian tương đối theo ngày:**
  - `hôm nay`: cùng ngày tham chiếu.
  - `hôm qua`: lùi 1 ngày so với mốc tham chiếu (`minusDays(1)`).
  - `hôm kia`: lùi 2 ngày so với mốc tham chiếu (`minusDays(2)`).
- **Khung giờ các buổi trong ngày (theo quy ước thiết kế bộ mẫu):**
  - `sáng` / `sáng nay`: mặc định 08:00.
  - `trưa` / `trưa nay`: mặc định 12:00.
  - `chiều` / `chiều nay`: mặc định 15:00.
  - `tối` / `tối nay`: mặc định 20:00.
  - `tối qua`: 20:00 của ngày hôm qua.
  - `đêm qua`: 22:00 của ngày hôm qua.
- **Thứ trong tuần và tuần trước:**
  - `thứ 2` đến `thứ 7`, `chủ nhật`: ánh xạ tương ứng `DayOfWeek.MONDAY` đến `DayOfWeek.SUNDAY`.
  - Cụm `tuần trước`: xác định ngày trong tuần thuộc về tuần liền trước mốc tham chiếu.
- **Thời gian tuyệt đối:**
  - `ngày X` (ví dụ: `ngày 5`): ngày thứ X trong tháng hiện tại của mốc tham chiếu.
  - `D/M` hoặc `ngày D tháng M` (ví dụ: `15/10`): ngày và tháng cụ thể trong năm hiện tại.
- **Phân biệt trạng từ thời gian và món ăn/mô tả:**
  - Các cụm từ kết hợp như `"ăn sáng"`, `"cơm trưa"`, `"ăn tối"`, `"cf sáng"` là mô tả khoản chi, không được bóc tách từ buổi thành trạng từ thời gian.
  - Từ chỉ buổi chỉ được nhận diện là thời gian khi đứng độc lập làm trạng từ hoặc có từ bổ trợ chỉ thời gian rõ ràng (`sáng nay`, `hôm qua`).
- **Kiểm thử tất định (Deterministic Testing) với `Clock`:**
  - Không bao giờ gọi `LocalDateTime.now()` trực tiếp. Mọi hàm xử lý thời gian đều nhận tham số `Clock` (hoặc `ZoneId`) để dễ dàng cố định thời gian trong unit test bằng `Clock.fixed()`.

## So với cách cũ bạn quen
| Cách cũ (`Calendar`, `Date`, `SimpleDateFormat`) | Cách mới (`java.time` trong Java 8+ / Android API 26+) | Khác biệt chính |
| :--- | :--- | :--- |
| `Date` và `Calendar` là mutable, dễ bị lỗi sửa đổi trạng thái ngầm định và không an toàn luồng (thread-unsafe). | `LocalDate`, `LocalTime`, `LocalDateTime`, `Instant` đều là immutable và an toàn luồng tuyệt đối. | Hạn chế tối đa bug đa luồng và tác dụng phụ khi truyền đối tượng giữa các hàm. |
| Chỉ số tháng đếm từ 0 (tháng 1 là 0, tháng 12 là 11), rất dễ gây nhầm lẫn. | Tháng đếm chuẩn từ 1 đến 12, hoặc dùng enum `Month` tường minh (`Month.OCTOBER`). | Tránh lỗi lệch tháng thường gặp ở code Java cũ. |
| Phụ thuộc vào `System.currentTimeMillis()` gây khó khăn khi viết unit test cho các ca lùi ngày. | Truyền `Clock` trừu tượng, dễ dàng mock hoặc dùng `Clock.fixed()` với mốc thời gian bất kỳ. | Unit test luôn chạy ổn định 100%, không bị ảnh hưởng bởi giờ của máy chạy test hay CI. |

## API và cú pháp cần biết
- `Clock.fixed(instant, zoneId)`: Tạo đối tượng đồng hồ cố định thời gian phục vụ kiểm thử.
- `LocalDate.now(clock)`: Lấy ngày hiện tại dựa trên một `Clock` cụ thể.
- `localDate.atTime(hour, minute)`: Ghép ngày và giờ thành `LocalDateTime`.
- `TemporalAdjusters`: Cung cấp các phép điều chỉnh lịch nâng cao như `previous(DayOfWeek)`.

Ví dụ minh họa điều chỉnh lịch sự kiện định kỳ (khác bài toán parser):
```kotlin
fun calculateNextMeeting(clock: Clock, targetDay: DayOfWeek, meetingHour: Int): LocalDateTime {
    val today = LocalDate.now(clock)
    val nextDate = today.with(TemporalAdjusters.next(targetDay))
    return nextDate.atTime(meetingHour, 0)
}
```

## Lỗi hay gặp
1. **Dùng giờ hệ thống trực tiếp:** Gọi `LocalDateTime.now()` thay vì `LocalDateTime.now(clock)` khiến unit test câu `hôm qua` bị sai lệch nếu chạy vào ngày khác với dataset test.
2. **Nuốt nhầm từ chỉ buổi của món ăn:** Xử lý từ `"sáng"` độc lập mà không kiểm tra token đứng liền trước (`"ăn"`, `"cơm"`, `"cf"`), làm mất ngữ cảnh của món ăn trong ghi chú.
3. **Lệch múi giờ:** Không chỉ định múi giờ `Asia/Ho_Chi_Minh` khi chuyển đổi giữa `Instant` và `LocalDateTime`, dẫn đến kết quả sai lệch 7 tiếng nếu môi trường máy hoặc CI chạy ở múi giờ UTC.
4. **Tính sai tuần trước:** Trừ 7 ngày đơn thuần cho một thứ trong tuần có thể sai nếu ngày hiện tại chưa qua thứ đó trong tuần. Cần lấy ngày đầu tuần hiện tại rồi lùi về thứ tương ứng của tuần trước.

## Đọc thêm
- [Package java.time - Android Developers](https://developer.android.com/reference/java/time/package-summary)
- [TemporalAdjusters in Java 8 - Baeldung](https://www.baeldung.com/java-temporal-adjuster)
- [Deterministic testing with Clock - Martin Fowler](https://martinfowler.com/articles/replaceThrowWithNotification.html)
