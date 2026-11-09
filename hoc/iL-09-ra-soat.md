# Rà soát iL-09 · SwiftUI: view và state

Ngày rà soát: 2026-11-09 · Nhánh `task/iL-09-swiftui-view-state` · Trạng thái: **Xong** (đóng sau rà lại lần 3)

## Lệnh đã chạy
- `xcodebuild -project luyen-tap/SachDoc/SachDoc.xcodeproj -scheme SachDoc -destination 'platform=iOS Simulator,name=iPhone 17' build` → **BUILD SUCCEEDED**. Chỉ có một cảnh báo của công cụ AppIntents ("Metadata extraction skipped"), không đến từ code.

## Bắt buộc sửa (2)

### 1. Badge thí nghiệm nằm trong view thật
Vị trí: `luyen-tap/SachDoc/SachDoc/BookRowView.swift:27-41`.
- **Vấn đề:** có hai badge "Đã đọc/Chưa đọc" giống nhau, chỉ khác thứ tự `padding` và `background`.
- **Vì sao:** đây là code thí nghiệm. Để trong `BookRowView` thì mỗi dòng của danh sách đều hiện hai badge.
- **Hướng sửa:** `BookRowView` chỉ giữ một badge đúng. Đưa phần so sánh hai thứ tự ra chỗ riêng, ví dụ `#Preview("Thứ tự modifier")` hoặc một view nhỏ chỉ dùng trong preview.

### 2. Giải thích thứ tự `padding` và `background`
Đây là tiêu chí xong của task. Phần này trả lời trực tiếp với mentor trong chat.

## Nên sửa (4)

### 1. Preview dùng `.constant(...)`
Vị trí: `BookRowView.swift:55-73`.
- **Vấn đề:** bấm nút trong canvas không đổi gì.
- **Vì sao:** không thử được tương tác `@Binding`, mà đó là trọng tâm của task.
- **Hướng sửa:**
  - Dùng `@Previewable @State`, xem ví dụ ở `docs/hoc/iL-09-swiftui-view-state.md`.
  - Thêm preview "Tên rất dài" để kiểm tra bố cục khi tên dài.

### 2. Danh sách chưa tách thành view con
Vị trí: `ContentView.swift:16-37`.
- **Vấn đề:** danh sách nằm thẳng trong `ContentView`. `init(books:)` phải dùng `Optional` cùng `_books = State(initialValue:)`.
- **Vì sao:**
  - Task yêu cầu tách view con.
  - `State(initialValue:)` gán trong `init` chỉ có tác dụng lần đầu. Đây chính là bẫy "@State cho dữ liệu từ ngoài" trong tài liệu lý thuyết.
- **Hướng sửa:**
  - Tách phần `List` và trạng thái rỗng sang `BookListView`, nhận dữ liệu bằng `@Binding`.
  - `ContentView` giữ `@State`.
  - Khi đó preview "Rỗng" của `BookListView` không cần `init` tùy biến nữa.

### 3. Model và dữ liệu mẫu nằm lẫn trong file view
Vị trí: `BookItem` ở `BookRowView.swift:11`; dữ liệu mẫu ở `ContentView.swift:11-15`.
- **Hướng sửa:** tạo `Book.swift` chứa model, cùng dữ liệu mẫu tĩnh trong `extension`, để preview và app dùng chung. iL-10 sẽ cần đúng chỗ này.

### 4. Dọn code
- Bỏ comment kiểu hướng dẫn ở `ContentView.swift:22`.
- Chỉnh lại thụt lề ở các dòng 19–27.
- Thay `HStack(content: {` bằng trailing closure.
- Bỏ `import Foundation` thừa, vì SwiftUI đã kéo theo Foundation.
- Sửa tên tác giả bị gõ sai "Paulo Côelh" ở `BookRowView.swift:61`.

## Gợi ý thêm để học (3)
1. **Nút chỉ có icon:**
   - Thêm `.accessibilityLabel` để VoiceOver đọc "Đánh dấu đã đọc" thay vì tên symbol.
   - Vùng chạm của icon `.title2` đang nhỏ hơn 44 pt.
2. **Shared scheme:** `luyen-tap/SachDoc/SachDoc.xcodeproj/xcshareddata/` nên commit cùng task, vì CI sau này sẽ cần nó. `xcuserdata/` đã được `.gitignore` bỏ qua, đúng như mong muốn.
3. **Thí nghiệm identity:** thử `ForEach(books.indices)` thay cho `ForEach($books)`, rồi xóa một phần tử để thấy vì sao dùng chỉ số làm identity dễ gây lỗi.

## Điểm làm tốt
- `ForEach($books) { $book in … }` truyền binding cho từng dòng; `BookItem` conform `Identifiable` với id `UUID`.
- Một nguồn sự thật: `ContentView` giữ `@State`, `BookRowView` chỉ mượn qua `@Binding`.
- Có trạng thái rỗng bằng `ContentUnavailableView`. `.buttonStyle(.borderless)` giúp chạm nút không kích hoạt cả dòng.

## Kết luận
Chưa đóng được task. Cần:
- sửa 2 mục Bắt buộc;
- sửa các mục Nên sửa, hoặc ghi lại bằng `hoãn <mục>`;
- báo lại để rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-09)
Lệnh đã chạy: `xcodebuild … build` → **BUILD SUCCEEDED**, không có cảnh báo từ code.

### Đã đạt
- Bắt buộc 1: `BookRowView` chỉ còn một badge. Phần thí nghiệm nằm trong `#Preview("Thử nghiệm thứ tự modifier")`.
- Bắt buộc 2: đã trả lời đúng trong chat. `background` lấy kích thước của view mà nó bọc. Với chuỗi `.padding().background(.red).padding().background(.blue)`, màu đỏ ở trong và màu xanh thành viền bao ngoài.
- Nên sửa 1: các preview đã dùng `@Previewable @State`, có thêm preview "Tên sách rất dài".
- Nên sửa 3: đã tạo `Book.swift`, dữ liệu mẫu nằm trong `extension BookItem`.
- Nên sửa 2 (một phần): đã tách `BookListView` nhận `@Binding`.

### Bắt buộc sửa (1)
1. **`NavigationStack` lồng nhau.**
   - Vị trí: `BookListView.swift:12` tự bọc `NavigationStack`. Trong khi đó `ContentView.swift:13` và hai preview (`BookListView.swift:30`, `:37`) cũng bọc thêm một lớp nữa. `navigationTitle` bị đặt ở 3 nơi.
   - Vì sao:
     - Mỗi màn hình chỉ nên có một `NavigationStack`, đặt ở gốc.
     - Lồng stack sẽ làm `navigationDestination` và `path` ở iL-10 chạy sai: đẩy màn nhầm stack, hoặc thanh điều hướng hiện lạ.
     - Đặt title ở nhiều nơi thì không rõ nơi nào quyết định.
   - Hướng sửa:
     - Chọn **một** chỗ sở hữu `NavigationStack`, nên là view gốc `ContentView`.
     - `BookListView` chỉ mô tả nội dung và tự khai báo `navigationTitle` của chính nó.
     - Preview muốn thấy thanh tiêu đề thì tự bọc stack.

### Nên sửa (2)
1. **Code chết.** `ContentView.swift:19-23` (`init(books:)`) không còn ai gọi. Bỏ đi.
2. **Dọn code (mục Nên sửa 4 cũ, chưa làm):**
   - Comment kiểu hướng dẫn vẫn còn ở `BookListView.swift:17`.
   - Thụt lề lệch ở `BookListView.swift:14-22`.
   - Còn `HStack(content: {` và `VStack(…, content: {` ở `BookRowView.swift:14-15`.
   - Còn `import Foundation` thừa ở `BookRowView.swift:8`.
   - Dòng trắng thừa ở `BookRowView.swift:36` và `ContentView.swift:16`.

### Kết luận
Chưa đóng được task. Sửa mục Bắt buộc và 2 mục Nên sửa (hoặc `hoãn <mục>`), rồi báo lại.

---

## Rà lại lần 2 (2026-11-09)
Lệnh đã chạy: `xcodebuild … build` → **BUILD SUCCEEDED**, không có cảnh báo từ code.

### Đã đạt
- **Mục bắt buộc (NavigationStack lồng nhau):** đã sửa. Giờ chỉ `ContentView` sở hữu `NavigationStack`, còn preview tự bọc stack riêng.
- **Mục nên sửa 1:** đã bỏ `init(books:)`.
- **Mục nên sửa 2:** đã bỏ `import Foundation`, `HStack` đã dùng trailing closure.

### Nên sửa còn sót (1)
1. **Dọn code chưa xong:**
   - Comment kiểu hướng dẫn còn ở `BookListView.swift:9`.
   - Thụt lề lệch ở `BookListView.swift:7-14`. Gợi ý: chọn hết rồi bấm Ctrl+I.
   - Còn `VStack(…, content: {` ở `BookRowView.swift:14`.
   - Dòng trắng thừa ở `BookRowView.swift:35` và `ContentView.swift:16`.

### Gợi ý (không chặn)
- `navigationTitle` đang đặt ở `ContentView`, nên mỗi preview của `BookListView` phải lặp lại tiêu đề. Nếu chuyển tiêu đề vào trong `BookListView` thì preview tự có tiêu đề, và mỗi màn tự khai báo tiêu đề của chính nó. Sang iL-10, khi có thêm màn chi tiết, cách này sẽ gọn hơn.

---

## Rà lại lần 3 (2026-11-09)
`xcodebuild … build` → **BUILD SUCCEEDED**, không cảnh báo. Đã dọn xong comment, thụt lề và `VStack` trailing closure.

**Kết luận:** Đóng được task.
