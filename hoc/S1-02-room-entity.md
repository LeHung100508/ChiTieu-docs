# S1-02 · Room: entity và schema

## Vì sao cần
S1-01 đã có domain model. Giờ cần một **hình dạng lưu trữ** cho 5 bảng của MVP (ví, danh mục, giao dịch, ngân sách, từ điển) cùng bảng FTS4 cho tìm kiếm (FR-20). Quyết định "Schema MVP" muốn schema đủ ngay ở S1, để không phải viết migration trước khi phát hành. Vì vậy task này là lúc nghĩ kỹ về khóa ngoại, index và ràng buộc unique.

## Khái niệm chính
- **Entity là kiểu của tầng lưu trữ, không phải domain.** Dùng kiểu nguyên thủy (`Long` cho tiền và thời gian, `String` cho enum), kèm annotation của Room. Việc chuyển đổi sang domain ở S1-06 (hoặc dạng `asExternalModel()` ngay trong `core:database`, bạn chọn ở bước 1).
- **Khóa ngoại (`@ForeignKey`) thể hiện quan hệ trong sơ đồ.** `onDelete` quyết định chuyện gì xảy ra khi xóa bản ghi cha. FR-24 nói danh mục "xóa được khi không còn giao dịch", nên hãy nghĩ: `RESTRICT`, `SET_NULL` hay `CASCADE` hợp với yêu cầu đó? Ví thì chỉ lưu trữ, không xóa.
- **Index** cho các cột hay lọc hoặc sắp xếp: ngày (`occurredAt`), ví, cặp ví và ngày. Room cảnh báo khi một cột khóa ngoại không có index. **Unique index** chặn dữ liệu trùng mà `require` ở domain không chặn được: `seedKey`, `keyword`, ngân sách theo cặp danh mục và kỳ.
- **FTS4 với external content.** Bảng ảo `@Fts4(contentEntity = …)` không chép dữ liệu, chỉ đánh chỉ mục trên `note` và `rawInput` của bảng giao dịch. Room tự sinh trigger để giữ hai bảng khớp nhau. Tokenizer `unicode61` hiểu chữ Unicode và mặc định bỏ dấu khi so khớp.
- **`exportSchema = true`.** Room ghi schema ra file JSON trong `core/database/schemas/` (convention plugin `chitieu.android.room` đã cấu hình `schemaDirectory`). File này được commit. Đây là "ảnh chụp" phiên bản 1, để S1-04 viết test migration về sau.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| `SQLiteOpenHelper`, viết `CREATE TABLE` tay | `@Entity`, `@Database` | Room sinh SQL và kiểm tra truy vấn lúc biên dịch |
| kapt | KSP | Nhanh hơn, plugin đã cấu hình sẵn |
| Một class dùng cho cả DB và UI | Entity riêng, domain riêng | Đổi schema không lan lên UI |
| `LIKE '%từ%'` để tìm | Bảng FTS4 với `MATCH` | Dùng chỉ mục, nhanh trên dữ liệu lớn, có tokenizer |
| Không lưu lịch sử schema | JSON schema được commit | Có căn cứ để test migration |

## API và cú pháp cần biết
- `@Entity(tableName, foreignKeys, indices)`, `@PrimaryKey(autoGenerate = true)`, `@ColumnInfo(name)`.
- `@ForeignKey(entity, parentColumns, childColumns, onDelete)`, `Index(value = [...], unique = true)`.
- `@Fts4(contentEntity = …, tokenizer = FtsOptions.TOKENIZER_UNICODE61)`. Bảng FTS có `rowid` ẩn, nối với `id` của bảng nội dung.
- `@TypeConverter` và `@TypeConverters`: chuyển kiểu mà SQLite không có (ví dụ `List<String>` cho bí danh ví). Room có sẵn cách lưu enum theo **tên** dạng TEXT.
- `@Database(entities = [...], version = 1, exportSchema = true)` trên một `abstract class` kế thừa `RoomDatabase`.

```kotlin
@Entity(
    tableName = "books",
    foreignKeys = [ForeignKey(Author::class, ["id"], ["author_id"], onDelete = ForeignKey.RESTRICT)],
    indices = [Index("author_id"), Index(value = ["isbn"], unique = true)],
)
data class BookEntity(
    @PrimaryKey(autoGenerate = true) val id: Long = 0,
    @ColumnInfo(name = "author_id") val authorId: Long, val isbn: String,
)
```

## Lỗi hay gặp
- **Unique index trên cột nullable.** SQLite coi các giá trị `NULL` là khác nhau, nên `UNIQUE(category_id, period)` vẫn cho phép **nhiều** ngân sách tổng (`category_id = NULL`). Cần một cách khác cho trường hợp này.
- **Chuyển khoản có hai khóa ngoại tới cùng bảng ví** (`walletId`, `toWalletId`). Cả hai đều cần index và cần cân nhắc `onDelete`.
- **Tokenizer và chữ "đ".** `unicode61` bỏ dấu kết hợp (sắc, huyền, mũ…), nhưng "đ" là một ký tự riêng, không phải "d" cộng dấu. Vì vậy tìm "dien" có thể không ra "điện". Chưa cần giải ở task này: ghi lại, rồi kiểm chứng bằng test khi làm tìm kiếm.
- **Quên `exportSchema` hoặc không commit file JSON:** S1-04 sẽ không có schema gốc để test migration.
- **Lưu `Money` hay `Instant` thẳng vào entity:** cần TypeConverter hoặc hỗ trợ value class của Room. Tôi không chắc phiên bản Room 2.8 xử lý value class ra sao. Dùng `Long` trong entity và chuyển đổi ở mapper là cách chắc chắn và rõ ràng nhất.

## Đọc thêm
- Define data using Room entities: https://developer.android.com/training/data-storage/room/defining-data
- Define relationships (foreign keys): https://developer.android.com/training/data-storage/room/relationships
- Full-text search với Room (`@Fts4`): https://developer.android.com/reference/androidx/room/Fts4
- SQLite FTS3/FTS4, tokenizer `unicode61`: https://www.sqlite.org/fts3.html#tokenizer
