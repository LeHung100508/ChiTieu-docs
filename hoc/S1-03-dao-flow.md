# S1-03 · DAO trả về Flow

## Vì sao cần
Ở S1-02 ta đã dựng xong bảng và khóa ngoại. Tầng DAO là cầu nối cung cấp dữ liệu cho Repository (S1-06) và UI. Bằng cách trả về Kotlin `Flow` và tính toán tổng hợp trực tiếp trong SQL (số dư ví, tổng theo danh mục/tháng), ứng dụng đạt kiến trúc phản ứng một chiều (UDF): cứ có thay đổi ở database là giao diện tự động cập nhật mà không cần truy vấn lại thủ công.

## Khái niệm chính
- **Truy vấn phản ứng bằng `Flow<T>`:** Hàm DAO trả về `Flow` **không** dùng từ khóa `suspend`; Room tự động chạy SQL trên background thread và phát lại giá trị mỗi khi bảng liên quan thay đổi. Các thao tác ghi (`@Insert`, `@Update`, `@Upsert`, `@Query` sửa/xóa) mới dùng `suspend`.
- **Tính toán trực tiếp bằng SQL (Single Source of Truth):**
  - **Số dư ví (FR-23):** `initial_balance + tổng INCOME - tổng EXPENSE + tổng TRANSFER đến (to_wallet_id) - tổng TRANSFER đi (wallet_id)`. Dùng `COALESCE(SUM(...), 0)` kết hợp `CASE WHEN` để tránh giá trị `NULL` khi ví chưa có giao dịch.
  - **Tổng theo danh mục và tháng (FR-27):** Tính `SUM(amount)` trong khoảng `[startMillis, endMillis)` (nửa mở để không trùng hoặc sót ranh giới ngày) nhóm theo `category_id`.
- **Tránh `OnConflictStrategy.REPLACE`:** Trong SQLite, `REPLACE` xóa dòng cũ rồi chèn lại, sẽ bị `@ForeignKey(RESTRICT)` chặn hoặc kích hoạt `CASCADE` làm mất oan ngân sách và từ điển cá nhân của danh mục. Luôn dùng **`@Upsert`** (cập nhật tại chỗ nếu trùng khóa) hoặc tách `@Insert` + `@Update`.
- **Bảo vệ vòng đời xóa mềm (`deleted_at`):** Mọi câu truy vấn đọc (kể cả tính số dư ví, thống kê) đều phải lọc `deleted_at IS NULL`. Đánh dấu xóa, hoàn tác, dọn rác và sửa giao dịch (`AND deleted_at IS NULL`) đều viết bằng `@Query` riêng — không dùng `@Update(TransactionEntity)` nhận `tx.asEntity()` để tránh vô tình khôi phục giao dịch đang trong cửa sổ 5 giây chờ xóa.
- **`@Transaction` bảo đảm toàn vẹn:** Dùng cho `upsertTotalBudget` (giữ đúng 1 dòng `category_id IS NULL` trong `budgets`) và cho các truy vấn đọc kèm `@Relation` (ví dụ cây danh mục cha–con).

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| `LiveData` hoặc gọi tay lại mỗi khi đổi DB | Kotlin `Flow<List<T>>` | Chuẩn Coroutines, dễ kết hợp toán tử `map`, `combine` ở Repository |
| Lấy hết list về Kotlin rồi lặp `sumOf` | SQL `COALESCE(SUM(CASE WHEN ...), 0)` | Tận dụng index ghép, tốn O(1) bộ nhớ dù có 10.000 giao dịch |
| `OnConflictStrategy.REPLACE` | `@Upsert` (Room 2.5+) | Chạy `UPDATE` tại chỗ khi xung đột, an toàn với khóa ngoại |

## API và cú pháp cần biết
- `@Dao`, `@Query`, `@Insert`, `@Update`, `@Upsert`, `@Transaction`, `@Relation`, `@Embedded`.
- `COALESCE(expr, 0)`: Trả về `0` nếu biểu thức tổng `SUM(...)` ra `NULL`.

```kotlin
// Minh họa DAO (ví dụ quản lý kho hàng)
@Dao
interface InventoryDao {
    @Query("SELECT * FROM items WHERE deleted_at IS NULL ORDER BY name ASC")
    fun observeActive(): Flow<List<ItemEntity>>

    @Upsert
    suspend fun upsert(item: ItemEntity): Long

    @Query("UPDATE items SET qty = qty + :delta WHERE id = :id AND deleted_at IS NULL")
    suspend fun adjustQty(id: Long, delta: Int)
}
```

## Lỗi hay gặp
- **Gắn `suspend` cho hàm trả về `Flow`:** Room sẽ báo lỗi biên dịch ngay. Chỉ hàm thao tác một lần (one-shot insert/update/delete/select không phải Flow) mới dùng `suspend`.
- **Quên `COALESCE` với `SUM`:** Nếu một ví hoặc danh mục chưa có giao dịch nào, `SUM(...)` trả về `NULL` (SQLite), gây crash nếu cột map vào kiểu `Long` không null.
- **Quên lọc `deleted_at IS NULL` trong truy vấn tổng hợp:** Làm số dư ví hoặc thống kê tháng vẫn tính cả giao dịch vừa bấm xóa mềm.
- **Dùng `REPLACE` trên `WalletEntity`/`CategoryEntity`:** Gây `SQLiteConstraintException` do `RESTRICT` hoặc làm mất dữ liệu `BudgetEntity`/`KeywordRuleEntity` do `CASCADE`.
- **Dùng `Transaction.asEntity()` cho thao tác sửa giao dịch:** Ghi đè `deleted_at = null`, vô tình hủy trạng thái xóa mềm nếu có tranh chấp trong 5 giây.

## Đọc thêm
- Truy vấn bất đồng bộ với Kotlin Flow: https://developer.android.com/training/data-storage/room/async-queries#kotlin-flow
- Annotation `@Upsert` trong Room: https://developer.android.com/reference/androidx/room/Upsert
- Quan hệ dữ liệu với `@Relation` và `@Transaction`: https://developer.android.com/training/data-storage/room/relationships

## Bổ sung (rà lại trước khi bắt đầu)
- **Kiểu kết quả cho truy vấn tổng hợp:** Room map cột trả về vào một `data class` bất kỳ theo tên cột (dùng alias `AS`). Ví kèm số dư có thể là `@Embedded val wallet: WalletEntity` cộng thêm `val balance: Long`. Kiểu này không phải `@Entity`, nên không cần khai báo trong `@Database`.
- **Chuyển khoản đi và đến nằm ở hai cột khác nhau** (`wallet_id`, `to_wallet_id`). Vì vậy không thể chỉ `GROUP BY wallet_id` là ra số dư. Hãy nghĩ cách cho một dòng chuyển khoản được cộng vào ví đích và trừ ở ví nguồn: subquery tương quan theo từng ví, hoặc `JOIN` với điều kiện `OR` kèm `CASE`. Sau đó dùng `EXPLAIN QUERY PLAN` để kiểm tra cách nào dùng được index.
- **Upsert ngân sách tổng không tự chạy được bằng `@Upsert`:** unique index coi các giá trị `NULL` là khác nhau, nên `@Upsert` một dòng `category_id = NULL` sẽ không gặp xung đột nào và luôn chèn thêm dòng mới. Vì thế cần `@Transaction`: tìm dòng tổng hiện có, rồi cập nhật dòng đó hoặc chèn mới.
- **Ranh giới tháng do tầng trên tính:** FR-35 cho chọn ngày bắt đầu tháng, và tháng được xác định theo múi giờ của máy. DAO chỉ nhận `[start, end)` dạng `Instant` hoặc millis. Việc đổi "tháng 11, bắt đầu ngày 25" thành khoảng thời gian thuộc repository (S1-06), để DAO không phải biết gì về lịch.
- **Flow của Room phát lại mỗi khi bảng liên quan thay đổi**, kể cả khi kết quả không đổi. Tầng trên có thể dùng `distinctUntilChanged()`.
- **Phạm vi:** FR-18 yêu cầu "nạp theo trang", nhưng `PagingSource` (`room-paging` đã có trong catalog) để dành cho S3. Ở S1-03 chỉ cần `Flow` theo khoảng ngày. Test DAO thuộc S1-04, nên ở task này kiểm tra bằng việc build: Room kiểm tra SQL ngay lúc biên dịch.
