# Rà soát iL-10 · SwiftUI: Observable và điều hướng

Ngày rà soát: 2026-11-09 · Nhánh `task/iL-10-observable-navigation` · Trạng thái: **Xong** (đóng sau rà lại lần 2)

## Lệnh đã chạy
- `xcodebuild -project luyen-tap/SachDoc/SachDoc.xcodeproj -scheme SachDoc -destination 'platform=iOS Simulator,name=iPhone 17' build` → **BUILD SUCCEEDED**, không có cảnh báo từ code.
- Chưa chạy thử trên simulator: mentor chưa được cấp quyền dùng thiết bị.

## Bắt buộc sửa (1)
1. **Xác nhận tiêu chí "chạy đúng trên iPhone thật".** Mentor không kiểm chứng được phần này. Cần chạy app trên iPhone thật và báo lại kết quả của 4 luồng:
   - Thêm một sách mới. Thêm xong, sách xuất hiện ở cuối danh sách.
   - Mở chi tiết, bấm Sửa, đổi tên rồi Lưu. Màn chi tiết và danh sách đều cập nhật.
   - Bấm Sửa rồi Hủy. Dữ liệu giữ nguyên.
   - Chạm vào nút tròn ở một dòng. Trạng thái đọc đổi, nhưng app **không** mở màn chi tiết.

## Nên sửa (4)

### 1. Điều hướng bằng cả struct `BookItem`
Vị trí: `BookListView.swift:13,17` và `BookDetailView.swift:4-9`.
- **Vấn đề:** giá trị đưa vào navigation path chỉ là một *bản sao* của sách tại lúc chạm. `Hashable` được tự sinh trên mọi thuộc tính, nên sau khi sửa, bản sao đó không còn khớp với dữ liệu thật. Vì vậy `BookDetailView` phải tự tìm lại sách trong store qua `currentBook`. Khi không tìm thấy, `?? book` sẽ âm thầm hiện dữ liệu cũ.
- **Vì sao:** cách này tạo ra hai nguồn dữ liệu cho cùng một cuốn sách, trái với nguyên tắc "một nguồn sự thật" của iL-09.
- **Hướng sửa:**
  - Điều hướng bằng thứ không đổi, tức là id của sách.
  - Màn chi tiết nhận id rồi lấy sách từ store.
  - Khi đó `BookItem` không cần `Hashable` nữa.
  - Tự trả lời: nếu sách bị xóa trong lúc đang mở chi tiết, màn chi tiết nên hiển thị gì?

### 2. `navigationDestination` đặt trong nhánh `else`
Vị trí: `BookListView.swift:17`.
- **Vấn đề:** đích điều hướng chỉ được đăng ký khi danh sách có dữ liệu.
- **Vì sao:** đích điều hướng nên được đăng ký cố định, không phụ thuộc trạng thái hiển thị. Hiện chưa gây lỗi vì app chưa có chức năng xóa. Nhưng khi có xóa, hoặc khi chuyển sang dùng `path`, lỗi sẽ khó lần ra.
- **Hướng sửa:** gắn modifier này ở phía ngoài phần điều kiện, cùng cấp với `.toolbar` và `.sheet`.

### 3. Logic sửa dữ liệu nằm rải rác trong view
Vị trí: `BookListView.swift:31` và `BookDetailView.swift:59-61`.
- **Vấn đề:** view tự `append` và tự tìm `firstIndex` để thay phần tử.
- **Vì sao:** `BookStore` đóng vai trò như ViewModel nhưng lại không có hành vi nào. Logic thêm và sửa nên nằm ở một chỗ, để còn test được và để view chỉ phát ra ý định, theo đúng UDF.
- **Hướng sửa:** thêm các hàm như `add(_:)` và `update(_:)` vào `BookStore`, view chỉ gọi các hàm đó. Đồng thời sửa khoảng trắng thừa trong `firstIndex (where:` ở dòng 59.

### 4. Tài liệu `docs/hoc/iL-10-observable-dieu-huong.md`
- **Dòng 25** ghi `@Bindable var book: Book` để bind vào `TextField`. `@Bindable` chỉ dùng được với class `@Observable`, mà trong SachDoc thì `BookItem` là struct. Cần ghi rõ là dùng cho object `@Observable`, ví dụ `@Bindable var store: BookStore`.
- **Dòng 46:** nếu thiếu `Hashable`, compiler báo lỗi ngay tại `NavigationLink(value:)`, không chỉ tại `.navigationDestination`. Nên tự thử bằng cách bỏ `Hashable` để xem compiler báo gì thật.

## Gợi ý thêm để học (4)
1. Trạng thái chưa đọc đang gọi hai tên khác nhau: dòng danh sách ghi "Chưa đọc", màn chi tiết ghi "Đang đọc". Nên thống nhất một tên.
2. `Spacer()` ở `BookDetailView.swift:44` nằm trong `ScrollView`, nên không có tác dụng.
3. Nút `plus` chỉ có icon. Thêm `.accessibilityLabel("Thêm sách")` để VoiceOver đọc đúng.
4. Màn sửa có thể dùng `.sheet(item:)` thay cho cờ `Bool`. Điều kiện kiểm tra tên rỗng nên dùng `.whitespacesAndNewlines`.

## Điểm làm tốt
- `BookStore` được giữ bằng `@State` ở view gốc. `BookListView` dùng `@Bindable` để lấy `$store.books` cho từng dòng. `BookDetailView` chỉ cần `let store` mà vẫn tự cập nhật nhờ cơ chế Observation.
- `BookFormView` sửa trên một bản nháp `@State`, chỉ gọi `onSave` khi bấm Lưu. Nhờ vậy bấm Hủy không làm hỏng dữ liệu, và một form dùng được cho cả thêm lẫn sửa.
- Nút Lưu bị tắt khi tên sách rỗng. Mỗi view đều có `#Preview`, kể cả trạng thái rỗng.

## Kết luận
Chưa đóng được task. Cần làm 3 việc:
1. Xác nhận 4 luồng chạy đúng trên iPhone thật.
2. Sửa các mục Nên sửa, hoặc ghi `hoãn <mục>`.
3. Báo lại để rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-09)
Lệnh đã chạy: `xcodebuild … build` → **BUILD SUCCEEDED**, không có cảnh báo từ code. Người dùng xác nhận đã chạy đúng trên iPhone thật.

### Đã đạt
- Đã chạy trên iPhone thật: người dùng xác nhận.
- **Nên sửa 1:** điều hướng bằng `book.id`. `BookDetailView` nhận `bookId` và lấy sách qua `store.book(for:)`. Khi không tìm thấy sách thì hiện `ContentUnavailableView`. `BookItem` đã bỏ `Hashable`.
- **Nên sửa 2:** `navigationDestination` đã chuyển ra ngoài nhánh điều kiện.
- **Nên sửa 3:** `BookStore` có `addBook`, `updateBook`, `book(for:)`; view chỉ còn gọi các hàm này.
- **Nên sửa 4:** đã sửa dòng `@Bindable` và mục lỗi `Hashable` trong tài liệu.
- **Gợi ý đã làm:**
  - nút `plus` đã có `accessibilityLabel`;
  - thống nhất chữ "Chưa đọc" giữa danh sách và chi tiết;
  - bỏ `Spacer()` thừa trong `ScrollView`.

### Bắt buộc sửa (1)
1. **Tên SF Symbol có khoảng trắng ở đầu.**
   - Vị trí: `BookDetailView.swift:74`, đang viết `systemImage: " books.vertical"`.
   - Vì sao: SwiftUI không tìm thấy symbol nào có tên bắt đầu bằng dấu cách. Màn "Sách không tồn tại" vì vậy sẽ thiếu icon, và Xcode console in cảnh báo không tìm thấy symbol.
   - Hướng sửa: bỏ khoảng trắng ở đầu chuỗi. Sau đó xem lại bằng một preview truyền vào một `UUID()` mới, tức id không có trong store.

### Gợi ý (không chặn)
- `docs/hoc/iL-10-observable-dieu-huong.md`, dòng 20 của bảng vẫn ghi "truyền thẳng đối tượng dữ liệu". Điều này mâu thuẫn với khuyến nghị điều hướng bằng id ở các phần bên dưới.
- Thêm một `#Preview("Sách không tồn tại")` cho `BookDetailView`. Đó cũng là cách nhanh nhất để thấy lỗi ở mục bắt buộc sửa.
- Dọn khoảng trắng: `BookDetailView.swift` có 2 dòng trống ở dòng 2–3, và các dòng 23, 36, 38 chỉ chứa khoảng trắng.

---

## Rà lại lần 2 (2026-11-09)
`xcodebuild … build` → **BUILD SUCCEEDED**, không cảnh báo. Đã sửa tên symbol thành `"books.vertical"`. Hai gợi ý (dòng 20 của bảng trong tài liệu, preview "Sách không tồn tại") chưa làm, không chặn việc đóng task.

**Kết luận:** Đóng được task.
