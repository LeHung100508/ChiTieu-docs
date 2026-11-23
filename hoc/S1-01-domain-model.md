# S1-01 · Domain model

## Vì sao cần
Mọi tầng phía trên (Room ở S1-02, repository ở S1-06, ViewModel, UI) cùng nói một "ngôn ngữ" là các kiểu trong `core:model`. Domain model là kiểu Kotlin thuần, **không biết Room, Compose hay Android**. Nhờ vậy entity của database đổi thế nào thì UI cũng không phải sửa theo. Sơ đồ nguồn nằm ở mục "Mô hình dữ liệu" trong `dac-ta-android.html` (ảnh `images/c6228671-dd7f.png`).

## Khái niệm chính
- **Phạm vi S1-01:** `Money`, `TransactionType`, `Transaction`, `Wallet`, `Category`, `Budget`, `KeywordRule`. Chưa làm `Tag`, `Attachment`, `RecurringRule` (thuộc các sprint sau, YAGNI).
- **Domain model khác entity.** Entity (S1-02) là hình dạng của bảng: `Long` cho thời gian, khóa ngoại, annotation `@Entity`. Domain là hình dạng mà app dùng. Việc chuyển đổi giữa hai bên nằm ở `core:data` hoặc `core:database`, không nằm trong `core:model`.
- **Value class cho đơn vị đo.** `Money(Long)` theo quyết định đã chốt: khi chạy nó gần như là `Long` trần, nhưng trình biên dịch chặn được lỗi truyền nhầm id hay timestamp vào chỗ số tiền. Phép toán (`+`, `-`, so sánh) khai báo bằng `operator fun` hoặc implement `Comparable`.
- **Bất biến:** `data class` toàn `val`, danh sách dùng `List` (chỉ đọc). Muốn sửa thì dùng `copy()`. Đây là nền cho UDF và cho Compose so sánh state.
- **Biểu diễn ràng buộc bằng kiểu.** Ví dụ: chỉ giao dịch chuyển khoản mới có ví đích; danh mục chỉ thuộc Chi hoặc Thu; `Budget.categoryId == null` nghĩa là ngân sách tổng. Bạn chọn giữa hai cách: kiểu phẳng có trường nullable kèm `init { require(...) }`, hoặc `sealed interface` cho từng loại. Mỗi cách có trade-off (bảng dưới).
- **Tên ổn định cho enum.** File sao lưu (FR-31) và bản iOS dùng chung định dạng. Tên hằng enum có thể thành giá trị trong file JSON, nên nghĩ trước khi đặt tên, và không dựa vào `ordinal`.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| Một class vừa là `@Entity` vừa là model cho UI | Tách entity (database) và domain (`core:model`) | Đổi schema không lan lên UI; `core:model` không phụ thuộc Room |
| `amount: Double` hoặc `Long` trần | `Money` dạng `@JvmInline value class` | Đúng kiểu ngay lúc biên dịch, gần như không tốn chi phí khi chạy |
| `Date`, `Calendar` | `java.time.Instant`, `LocalDate` | Bất biến, an toàn luồng, có sẵn từ API 26 |
| Hằng `Int` cho loại | `enum class` hoặc `sealed` | `when` vét cạn, compiler báo khi thiếu nhánh |

**Trade-off khi mô tả "chuyển khoản có ví đích":**
| Cách | Lợi | Hại |
| --- | --- | --- |
| Phẳng + `require` | Ánh xạ sang Room dễ, `copy()` đơn giản | Lỗi chỉ lộ ra khi chạy |
| `sealed` theo loại | Trạng thái sai không viết ra được | Mapper và form sửa phức tạp hơn |

## API và cú pháp cần biết
- `@JvmInline value class`: bọc đúng một thuộc tính. Bị boxing khi dùng kiểu nullable (`Money?`), generic hoặc qua interface.
- `operator fun plus/minus/unaryMinus`, `Comparable<T>`: cho kiểu tự định nghĩa dùng được `+`, `-`, `<`.
- `init { require(cond) { "thông báo" } }`: kiểm tra bất biến khi tạo đối tượng, sai thì ném `IllegalArgumentException`.
- `companion object { val ZERO = ... }`: hằng hay dùng.

```kotlin
@JvmInline
value class Meters(val value: Double) : Comparable<Meters> {
    operator fun plus(other: Meters) = Meters(value + other.value)
    override fun compareTo(other: Meters) = value.compareTo(other.value)
}
data class Run(val id: Long, val distance: Meters) {
    init { require(distance.value >= 0) { "distance âm" } }
}
```

## Lỗi hay gặp
- **Lưu thứ tính được.** Số dư ví không phải là trường (FR-23: tính từ giao dịch). Domain có thể có kiểu kết quả riêng, như "ví kèm số dư", nhưng không thêm `balance` vào `Wallet`.
- **Để tên hiển thị của danh mục mặc định trong model.** Danh mục mặc định lưu `seedKey`, tên lấy từ string resource; danh mục do người dùng tạo mới lưu tên. Model phải biểu diễn được cả hai trường hợp mà không chứa chuỗi tiếng Việt cứng.
- **Kéo phụ thuộc Android vào `core:model`.** Không `import android.*`, không `@Entity`, không `@Immutable` của Compose. Module này nên là JVM library như `core:parser`.
- **Dùng `ordinal` hay `toString()` của enum để lưu.** Đổi thứ tự hằng số là hỏng dữ liệu cũ.
- **Test cho có.** Chỉ cần test những chỗ có logic: phép toán của `Money`, các `require`. Không viết test cho getter.

## Đọc thêm
- Inline value classes: https://kotlinlang.org/docs/inline-classes.html
- Sealed classes và interfaces: https://kotlinlang.org/docs/sealed-classes.html
- Guide to app architecture, Data layer: https://developer.android.com/topic/architecture/data-layer
- Now in Android, module `core:model`: https://github.com/android/nowinandroid/tree/main/core/model
