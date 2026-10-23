# S0-08 · Design system: component

## Vì sao cần
Thay vì tự viết lại giao diện số tiền, chip hay thẻ ở từng màn hình dẫn đến lệch kích thước và màu sắc, các component cốt lõi được chuẩn hóa trong `core:designsystem` giúp toàn bộ ứng dụng nhất quán, dễ tái sử dụng và kiểm thử độc lập qua Compose Preview.

## Khái niệm chính
- **AmountText:** Composable hiển thị số tiền định dạng chuẩn tiếng Việt (VND: dấu chấm phân tách hàng nghìn, hậu tố `₫` hoặc `đ`), tự đổi màu theo loại giao dịch (Thu: màu xanh chính, Chi: màu thường hoặc nhấn đỏ khi âm).
- **Assist / Filter / Input Chip:** Chip Material 3 đại diện cho các trường thông tin (Danh mục, Ví, Ngày). Hỗ trợ trạng thái nổi bật (highlighted/warning) khi bộ phân tích câu chưa chắc chắn theo FR-08.
- **Thẻ tổng (Summary Card):** Container hiển thị thông số tài chính tổng quan (tổng chi, số dư), kế thừa bảng màu `surfaceContainer` và bo góc chuẩn của hệ thống theme.
- **Icon danh mục (CategoryIcon):** Icon nằm trong container hình tròn với màu nền dịu nhẹ (tonal container), đảm bảo nhận diện thị giác tức thì cho các danh mục chi tiêu.
- **Preview-Driven Development:** Mỗi component độc lập luôn đi kèm `@Preview` sáng và tối, giúp kiểm tra hiển thị mà không cần chạy toàn bộ ứng dụng.

## So với cách cũ bạn quen

| Cũ (XML & View System) | Mới (Jetpack Compose Component) | Khác biệt chính |
| --- | --- | --- |
| Tạo `custom_chip.xml`, custom `TextView` | Composable hàm thuần túy nhận `Modifier` và state | Không cần inflate layout XML, tái sử dụng trực tiếp |
| Dùng `DecimalFormat` thủ công trong Adapter | Hàm tiện ích format tiền tệ thuần túy hoặc tích hợp trong composable | Đảm bảo tính toán và hiển thị nhất quán một nơi |
| Đổi màu động qua code Java/Kotlin bằng `ContextCompat.getColor()` | Khai báo màu qua `MaterialTheme.colorScheme` | Tự động thích ứng khi chuyển đổi Dark Theme |

## API và cú pháp cần biết
- `NumberFormat.getCurrencyInstance(Locale("vi", "VN"))`: Bộ định dạng số tiền tệ theo chuẩn Việt Nam.
- `AssistChip`, `FilterChip`, `SuggestionChip`: Các component chip Material 3.
- `Card`, `ElevatedCard`: Khung thẻ hiển thị thông tin với hình khối và độ nổi Material 3.
- `Box(modifier = Modifier.clip(CircleShape))`: Tạo khung tròn cho icon danh mục.

Đoạn code minh họa (Component mẫu):
```kotlin
@Composable
fun StatusBadge(
    label: String,
    isActive: Boolean,
    modifier: Modifier = Modifier,
) {
    Surface(
        shape = MaterialTheme.shapes.small,
        color = if (isActive) MaterialTheme.colorScheme.primaryContainer else MaterialTheme.colorScheme.surfaceVariant,
        modifier = modifier,
    ) {
        Text(
            text = label,
            style = MaterialTheme.typography.labelSmall,
            color = if (isActive) MaterialTheme.colorScheme.onPrimaryContainer else MaterialTheme.colorScheme.onSurfaceVariant,
            modifier = Modifier.padding(horizontal = 8.dp, vertical = 4.dp),
        )
    }
}
```

## Lỗi hay gặp
1. **Hard-code định dạng chuỗi số tiền:** Tự viết ghép chuỗi thủ công `$amount đ` thay vì dùng format chuẩn có dấu phân cách hàng nghìn (ví dụ: `45.000 ₫`), gây khó đọc khi số tiền lớn.
2. **Thiếu tham số `modifier: Modifier = Modifier`:** Quên truyền `modifier` ngoài cùng làm composable mất tính linh hoạt khi tái sử dụng ở các màn hình khác.
3. **Không xử lý màu tương phản:** Đặt màu chữ tĩnh khiến chữ bị chìm hoặc biến mất khi chuyển sang Dark Theme (luôn dùng cặp `colorScheme.surfaceContainer` và `colorScheme.onSurface`).
4. **Không tách biệt định dạng tiền tệ:** Gộp logic format số tiền sâu vào ViewModel thay vì tách hàm tiện ích hoặc composable hiển thị chuyên trách.

## Đọc thêm
- [Material 3 Chips Component](https://developer.android.com/develop/ui/compose/components/chip)
- [Material 3 Cards Component](https://developer.android.com/develop/ui/compose/components/card)
- [Jetpack Compose Component Design Best Practices](https://developer.android.com/develop/ui/compose/mental-model)
