# iL-09 · SwiftUI: view, modifier, @State và @Binding

## Vì sao cần
Mọi màn hình của ChiNhanh bản iOS đều viết bằng SwiftUI. Task này giúp bạn quen ba thứ nền tảng: view là gì, modifier hoạt động ra sao, và ai *sở hữu* state. Bạn đã biết Compose, nên phần lớn ý tưởng sẽ quen thuộc. Chỗ cần để ý là những điểm SwiftUI làm khác.

## Khái niệm chính
- **View là `struct` nhẹ, tạo đi tạo lại liên tục.** `body` chỉ *mô tả* giao diện. SwiftUI gọi `body` mỗi khi state thay đổi rồi so sánh với lần trước để cập nhật màn hình. Đừng giữ dữ liệu cần sống lâu trong thuộc tính thường của view.
- **Modifier bọc view thành view mới.** `Text("A").padding()` không sửa `Text`. Nó tạo ra một view mới bọc ngoài `Text`. Modifier viết sau sẽ tác động lên *kết quả* của modifier viết trước, nên thứ tự modifier quyết định kết quả.
- **`@State`: state do chính view sở hữu.** SwiftUI cất giá trị ở bên ngoài struct, nên giá trị vẫn còn khi struct được tạo lại. State sống chừng nào view còn giữ nguyên *identity* trong cây view. Luôn khai báo `private`.
- **`@Binding`: tham chiếu hai chiều tới state của view khác.** View con đọc và ghi được, nhưng không sở hữu giá trị. Truyền xuống bằng `$tenBien`, gọi là projected value.
- **Một nguồn sự thật.** Mỗi dữ liệu chỉ có *một* chủ sở hữu. View nào cần sửa dữ liệu đó thì nhận `@Binding`, không tự giữ một bản sao.
- **Identity trong `List`/`ForEach`.** Phần tử phải có id ổn định: dùng kiểu conform `Identifiable` hoặc truyền `id:`. Nếu id sai, state của dòng bị gắn nhầm sang dòng khác.

## So với cách cũ bạn quen
| Cũ (Compose) | Mới (SwiftUI) | Khác biệt chính |
| --- | --- | --- |
| Hàm `@Composable` | `struct` conform `View`, có thuộc tính `body` | View là giá trị. Tham số của view là thuộc tính của struct |
| `remember { mutableStateOf(x) }` | `@State private var x` | Không cần `remember`. SwiftUI tự giữ giá trị theo identity của view |
| State hoisting: `value` + `onValueChange` | `@Binding var value` | Gộp cả đọc lẫn ghi vào một thuộc tính |
| Chuỗi `Modifier.a().b()` | Chuỗi `.a().b()` gắn sau view | Thứ tự đều quan trọng, nhưng **cách đọc ngược nhau**. Hãy tự kiểm chứng bằng Preview, đừng mang trực giác từ Compose sang |
| `LazyColumn` + `key` | `List`/`ForEach` + `Identifiable` | Id là bắt buộc, không phải tùy chọn |
| `@Preview` | `#Preview { … }` | Trong một file có thể có nhiều `#Preview`, mỗi cái đặt tên riêng |

## API và cú pháp cần biết
- `VStack`, `HStack`, `ZStack`, `Spacer`: xếp view theo chiều dọc, chiều ngang, chồng lên nhau, và chèn khoảng trống.
- `List`, `ForEach`: danh sách. Hai kiểu này cần phần tử `Identifiable` hoặc tham số `id:`.
- `Toggle`, `Button`, `Image(systemName:)`: các control cơ bản và biểu tượng SF Symbols.
- `#Preview("Tên") { … }`: một preview có tên. `@Previewable @State` dùng để tạo state ngay trong preview, cho view nào cần `@Binding`.

Ví dụ (bài toán khác, công tắc đèn):
```swift
struct Room: View {
    @State private var isOn = false              // Room sở hữu state
    var body: some View { LightSwitch(isOn: $isOn) }
}
struct LightSwitch: View {
    @Binding var isOn: Bool                      // chỉ mượn, không sở hữu
    var body: some View { Toggle("Đèn", isOn: $isOn) }
}
#Preview("Đang tắt") { @Previewable @State var on = false; LightSwitch(isOn: $on) }
```

## Lỗi hay gặp
- **Dùng `@State` cho dữ liệu cha truyền xuống.** `@State` chỉ lấy giá trị khởi tạo ở lần đầu, nên khi cha đổi giá trị thì view con không cập nhật. Dữ liệu từ cha thì dùng `let` (chỉ đọc) hoặc `@Binding` (cần sửa).
- **Dùng chỉ số mảng làm id**, ví dụ `ForEach(0..<books.count)`. Khi thêm hoặc xóa phần tử, state bị gắn nhầm dòng. Hãy dùng id thật của dữ liệu.
- **Có tác dụng phụ trong `body`**, như in log hay gán state. `body` có thể được gọi rất nhiều lần, nên việc bất đồng bộ phải đặt trong `.task`.
- **Quên dấu `$`**: truyền `isOn` thay vì `$isOn` thì compiler báo lệch kiểu giữa `Bool` và `Binding<Bool>`.
- **Gom hết vào một `body` thật dài.** Tách view con giúp đọc dễ hơn, mỗi view có preview riêng, và SwiftUI chỉ cập nhật lại phần nào có state thay đổi.

## Đọc thêm
- [Managing user interface state](https://developer.apple.com/documentation/swiftui/managing-user-interface-state)
- [State](https://developer.apple.com/documentation/swiftui/state) và [Binding](https://developer.apple.com/documentation/swiftui/binding)
- [Configuring views (modifier)](https://developer.apple.com/documentation/swiftui/configuring-views)
- [Previews in Xcode](https://developer.apple.com/documentation/swiftui/previews-in-xcode)
