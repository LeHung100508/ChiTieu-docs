# iS0-06 · Design system: component

## Vì sao cần
Các màn hình của app (Tổng quan, Sổ giao dịch, Ngân sách) đều lặp lại những thành phần UI cơ bản: hiển thị số tiền có dấu và màu sắc theo loại, chip lọc danh mục, thẻ tổng hợp tiến độ và icon danh mục. Việc đóng gói thành các component tái sử dụng trong `DesignSystem` giúp giao diện nhất quán, tập trung logic định dạng số tiền VND, bảo đảm vùng chạm tối thiểu 44 pt và hỗ trợ đầy đủ VoiceOver cho người khiếm thị.

## Khái niệm chính
- **FormatStyle cho tiền tệ:** Sử dụng `formatted(.currency(code: "VND").locale(locale))` với locale linh hoạt lấy từ `@Environment(\.locale)` thay vì `NumberFormatter` kiểu cũ hay gắn cứng `vi_VN`, giúp hiển thị tiền tệ đúng ngôn ngữ người dùng đặt trong hệ thống.
- **Vùng chạm tối thiểu (Touch Target ≥ 44 pt):** Quy chuẩn Apple HIG yêu cầu mọi thành phần có thể tương tác phải có kích thước chạm tối thiểu 44×44 pt. Với `Button`, các modifier `.frame(minHeight: 44)` và `.contentShape(Rectangle())` **phải đặt bên trong label** của `Button` (sau phần background thị giác) để toàn bộ vùng 44 pt đều nhận sự kiện chạm.
- **Trợ năng VoiceOver:**
  - Nhãn đọc (`.accessibilityLabel`): Diễn đạt rõ nghĩa và an toàn số lớn (dùng `amount.magnitude` thay cho `abs(amount)` để chống tràn số với `Int64.min`).
  - Trạng thái chọn (`.accessibilityAddTraits(.isSelected)`): Báo cho người dùng biết chip đang được chọn hay không (không cần thêm `.isButton` vì `Button` đã có sẵn).
  - Gom nhóm (`.accessibilityElement(children: .combine)`): Gom các nhãn rời rạc trong thẻ tổng hợp thành một khối thông tin mạch lạc.
- **Tập trung hóa logic giao diện:** Các ngưỡng tính toán tỷ lệ, phần trăm và màu sắc cảnh báo của tiến độ ngân sách cần được định nghĩa ở một hàm tập trung duy nhất để cả view và unit test cùng sử dụng.

## So với cách cũ bạn quen
| Android (Compose / XML) | iOS (SwiftUI) | Khác biệt chính |
| --- | --- | --- |
| `Modifier.sizeIn(minWidth = 48.dp, minHeight = 48.dp)` | Đặt `.frame(minHeight: 44).contentShape(...)` trong label của Button | Chuẩn vùng chạm iOS là 44 pt (Android là 48 dp). Với Button, phải đặt trong label. |
| `NumberFormat.getCurrencyInstance(Locale.getDefault())` | `amount.formatted(.currency(code: "VND").locale(locale))` | `FormatStyle` trong Swift là struct kiểu tĩnh, nhận `Locale` từ `@Environment(\.locale)`. |
| `Modifier.semantics { selected = true }` | `.accessibilityAddTraits(isSelected ? .isSelected : [])` | SwiftUI dùng modifier `accessibilityAddTraits` để thêm ngữ nghĩa. |
| `Modifier.semantics(mergeDescendants = true)` | `.accessibilityElement(children: .combine)` | Gom các thành phần con để VoiceOver đọc liền mạch một lần thay vì từng nhãn lẻ. |

## API và cú pháp cần biết

1. **Định dạng tiền tệ tập trung với FormatStyle và Locale linh hoạt:**
```swift
extension Int64 {
    public func formattedVND(locale: Locale = .current) -> String {
        self.formatted(.currency(code: "VND").locale(locale))
    }
}
```

2. **Đảm bảo vùng chạm 44 pt cho Button mà không làm phình kích thước nhìn thấy:**
```swift
Button(action: action) {
    HStack { ... }
        .padding(.horizontal, 12)
        .padding(.vertical, 6)
        .background(Color.blue, in: Capsule())
        // Đặt bên trong label của Button để cả vùng 44 pt đều nhận tương tác:
        .frame(minHeight: 44)
        .contentShape(Rectangle())
}
.buttonStyle(.plain)
```

3. **Gắn trait và nhãn cho VoiceOver:**
```swift
Button(action: toggle) {
    Label("Ăn uống", systemImage: "fork.knife")
}
.accessibilityLabel("Ăn uống")
.accessibilityAddTraits(isSelected ? .isSelected : [])
```

4. **Gộp thông tin thẻ thành một khối dễ nghe:**
```swift
VStack {
    Text("Hạn mức")
    Text("80%")
}
.accessibilityElement(children: .combine)
.accessibilityLabel("Hạn mức: đã chi 80 phần trăm")
```

## Lỗi hay gặp
1. **Gắn cứng locale `vi_VN` và chép lặp lại ở nhiều nơi:** Khiến app không hiển thị đúng theo cài đặt ngôn ngữ/vùng của máy (ví dụ người dùng cài tiếng Anh) và khó bảo trì khi đổi định dạng. Nên tạo extension/hàm tập trung nhận `Locale`.
2. **Đặt vùng chạm 44 pt ở ngoài `Button`:** `.frame(minHeight: 44)` đặt ngoài `Button` chỉ làm nở khung view bên ngoài, còn nút bấm vẫn chỉ nhận sự kiện trong phần chữ nhỏ của label.
3. **Dùng `abs(amount)` khi xử lý số âm:** Sẽ gây crash ứng dụng do tràn số nếu gặp giá trị `Int64.min`. Luôn dùng `amount.magnitude` trong Swift.
4. **Viết test cho bản sao mà view không dùng:** Tách hàm static riêng cho test nhưng view lại tự tính nhẩm bằng logic nội bộ. Khi thay đổi ngưỡng trong view thì test vẫn xanh dù view đã sai.
5. **Độ tương phản của chữ trên nền xám nhạt dưới 4.5:1:** Dùng `Color.secondary` trên nền `surfaceSubtle` ở chế độ sáng không đạt chuẩn WCAG AA cho chữ thường.

## Đọc thêm
- [Apple HIG: Layout and touch targets](https://developer.apple.com/design/human-interface-guidelines/layout#Touch-targets)
- [Apple HIG: Accessibility on iOS](https://developer.apple.com/design/human-interface-guidelines/accessibility)
- [Apple Documentation: FormatStyle](https://developer.apple.com/documentation/foundation/formatstyle)
