# iL-10 · SwiftUI: Observable và điều hướng

## Vì sao cần
Khi ứng dụng lớn dần, dữ liệu không thể chỉ nằm trong các biến `@State` rời rạc của từng màn hình mà cần một tầng quản lý dữ liệu tập trung (Store/ViewModel), đồng thời hỗ trợ chuyển trang (Navigation) và mở các biểu mẫu thêm/sửa (Modal Sheet). Task này trang bị cho bạn kiến trúc điều hướng và luồng dữ liệu chuẩn mực của iOS 17+.

## Khái niệm chính
- **Macro `@Observable` (Observation framework)**: Đánh dấu một `class` là nguồn dữ liệu có thể theo dõi. SwiftUI tự động nhận biết chính xác view nào đang đọc thuộc tính nào của object để chỉ re-render đúng view đó khi dữ liệu đổi, tối ưu hơn hẳn Combine (`ObservableObject`) cũ.
- **`@Bindable`**: Property wrapper dùng để tạo các liên kết hai chiều (`Binding`) tới các thuộc tính bên trong một đối tượng `@Observable`. Rất phổ biến khi truyền model vào Form hoặc Sheet để chỉnh sửa.
- **Type-safe Navigation (`navigationDestination(for:)`)**: Điều hướng dựa trên dữ liệu (data-driven navigation). Thay vì gán cứng View đích vào `NavigationLink`, ta gắn dữ liệu `NavigationLink(value: book)`. `NavigationStack` sẽ lắng nghe và gọi `.navigationDestination(for: Book.self) { book in ... }` để dựng View tương ứng khi người dùng chạm vào.
- **Yêu cầu `Hashable`**: Mọi kiểu dữ liệu dùng làm `value` trong `NavigationLink` bắt buộc phải tuân thủ protocol `Hashable` để `NavigationStack` phân biệt các màn hình trong ngăn xếp.
- **Modal Sheet (`.sheet`)**: Hiển thị màn hình dạng tấm trượt từ dưới lên, gồm 2 cách kích hoạt:
  - `.sheet(isPresented: $isShowing)`: Điều khiển bằng cờ Boolean (thường dùng khi Thêm mới).
  - `.sheet(item: $selectedBook)`: Điều khiển bằng một đối tượng `Identifiable` (thường dùng khi Chỉnh sửa một phần tử cụ thể).

## So với cách cũ bạn quen
| Cũ (Android / Compose) | Mới (iOS / SwiftUI 17+) | Khác biệt chính |
| --- | --- | --- |
| `ViewModel` kế thừa từ `androidx.lifecycle.ViewModel` | `class` gắn macro `@Observable` | Không cần kế thừa lớp cha, là class Swift thuần túy |
| `StateFlow<T>` + `collectAsStateWithLifecycle()` | Thuộc tính thường `var items: [T]` | SwiftUI tự động phát hiện truy cập thuộc tính mà không cần khai báo `Flow` |
| Navigation Compose: `NavHost` + chuỗi route String `"detail/{id}"` | `NavigationStack` + `.navigationDestination(for: T.self)` | Điều hướng an toàn kiểu (Type-safe), truyền thẳng đối tượng dữ liệu thay vì parse String |
| `ModalBottomSheet` trong Compose | Modifier `.sheet(...)` | Tự động có hiệu ứng trượt, vuốt xuống để đóng chuẩn iOS |

## API và cú pháp cần biết
- `@Observable class BookStore`: Khai báo store/model quản lý dữ liệu.
- `@Bindable var store: BookStore`: Tạo binding từ object `@Observable` (chỉ dùng với class `@Observable`, không dùng cho struct).
- `NavigationLink(value: item.id) { ... }`: Nút điều hướng đính kèm dữ liệu (nên ưu tiên truyền id bất biến).
- `.navigationDestination(for: UUID.self) { id in ... }`: Đăng ký view đích cho kiểu dữ liệu điều hướng.
- `.sheet(isPresented: $isPresented) { ... }`: Mở sheet theo trạng thái đóng/mở.
- `.sheet(item: $selectedItem) { item in ... }`: Mở sheet với dữ liệu của item được chọn.

Ví dụ minh họa cú pháp (không liên quan bài toán SachDoc):
```swift
@Observable class CounterStore {
    var count = 0
}

struct CounterEditor: View {
    @Bindable var store: CounterStore
    var body: some View {
        Stepper("Giá trị: \(store.count)", value: $store.count)
    }
}
```

## Lỗi hay gặp
- **Quên `Hashable` khi điều hướng bằng cả Model**: Khi dùng `NavigationLink(value: item)`, nếu kiểu dữ liệu chưa conform `Hashable`, Xcode sẽ báo lỗi biên dịch ngay tại `NavigationLink(value:)` và tại `.navigationDestination`. Tốt nhất nên điều hướng bằng `id` (`UUID` hoặc `String`) vì chúng đã mặc định là `Hashable`, giúp Model không bị ép phụ thuộc vào navigation.
- **Dùng nhầm `@Binding` thay vì `@Bindable` cho `@Observable class`**: `@Binding` dành cho value type (struct, boolean); với class `@Observable`, ta dùng `@Bindable` để sinh ra binding cho các thuộc tính bên trong nó.
- **Đặt `.navigationDestination` bên ngoài `NavigationStack` hoặc lồng trong nhánh `if/else`**: Modifier này bắt buộc phải nằm bên trong phạm vi của `NavigationStack` và nên đặt ở cấp ngoài cùng của màn hình để luôn được đăng ký cố định bất kể danh sách rỗng hay có dữ liệu.
- **Sheet không đóng được**: Quên gán `dismiss()` từ `@Environment(\.dismiss)` trong nút "Xong" hoặc "Hủy" của Sheet.

## Đọc thêm
- [Apple Developer – Managing model data in your app (Observation)](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app)
- [Apple Developer – NavigationStack](https://developer.apple.com/documentation/swiftui/navigationstack)
- [Apple Developer – Migrating from ObservableObject to Observable](https://developer.apple.com/documentation/swiftui/migrating-from-the-observable-object-protocol-to-the-observable-macro)
