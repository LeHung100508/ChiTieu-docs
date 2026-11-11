# iL-11 · Debug: Breakpoint, LLDB và View Hierarchy

## Vì sao cần
Khi viết code, lỗi hiển thị hoặc sai lệch logic là điều không thể tránh khỏi. Thay vì dùng `print()` thủ công và phải re-build tốn thời gian, việc sử dụng thành thạo bộ công cụ gỡ lỗi của Xcode (Breakpoint thông minh, dòng lệnh LLDB và trình soi giao diện 3D) giúp bạn chẩn đoán và khoanh vùng nguyên nhân gốc rễ chỉ trong vài giây.

## Khái niệm chính
- **Breakpoint có điều kiện (Conditional Breakpoint)**: Breakpoint chỉ kích hoạt tạm dừng luồng thực thi khi một biểu thức điều kiện trả về `true` (ví dụ: `book.title == "Nhà Giả Kim"` hoặc `books.count > 3`). Có thể gán thêm hành động (Action) như tự động in log hoặc phát âm thanh mà không cần dừng app.
- **LLDB (Low Level Debugger)**: Trình thông dịch dòng lệnh gỡ lỗi tích hợp trong Xcode Console khi app dừng tại breakpoint:
  - `po` (*print object*): In mô tả đối tượng cho người đọc, ưu tiên `debugDescription` (`CustomDebugStringConvertible`), sau đó mới tới `description` (`CustomStringConvertible`).
  - `p` (*print*): Từ Xcode 15, `p` là alias của `dwim-print` (*Do What I Mean*). Nó đọc trực tiếp giá trị biến giống `v` khi có thể và chỉ biên dịch JIT thành biểu thức khi cần thiết.
  - `v` (*frame variable*): Đọc trực tiếp biến từ stack frame mà không qua bộ biên dịch biểu thức, tốc độ cực nhanh và không gây side-effect.
  - `expression` (viết tắt `expr`): Thực thi một đoạn mã Swift lúc app đang chạy, giúp kiểm thử giả thuyết hoặc thay đổi trực tiếp giá trị biến có thể biến đổi (`var`) trên bộ nhớ.
- **Debug View Hierarchy**: Công cụ chụp và phân tích cây phân cấp giao diện 3D lúc runtime. Giúp nhìn xuyên qua các lớp View, kiểm tra kích thước frame thực tế, khoảng đệm padding, và phát hiện các view bị ẩn, bị đè hoặc lồng sai cấu trúc container.

## So với cách cũ bạn quen
| Cũ (Android Studio) | Mới (Xcode / iOS) | Khác biệt chính |
| --- | --- | --- |
| Click phải breakpoint > nhập Condition | Click phải breakpoint > Edit Breakpoint > Condition | Cú pháp điều kiện bằng Swift; hỗ trợ gán Action tự động chạy lệnh LLDB |
| Evaluate Expression (`Alt + F8`) | LLDB console: `p`, `po`, `expr` | Dùng dòng lệnh tương tác trực tiếp trong console bên dưới |
| `println(object)` trong Debugger | `po object` | In ra chuỗi mô tả của object |
| Layout Inspector | Debug View Hierarchy (icon 3 lớp layer) | Cho phép xoay 3D trực quan các lớp View lồng nhau của SwiftUI |

## API và cú pháp cần biết
- Condition trong Breakpoint: `book.title == "Nhà Giả Kim"` hoặc `book.isRead == false`.
- `po <tên_biến>`: In nội dung của biến (ví dụ `po store.books.count`).
- `p <tên_biến>`: In giá trị và kiểu của biến (từ Xcode 15 là alias của `dwim-print`).
- `expr <biến> = <giá_trị>`: Thay đổi giá trị biến có thể ghi (`var`) ngay tại thời điểm dừng (ví dụ `expr books[0].title = "Tên Mới"`; lưu ý không sửa được tham số `let` như `book.title` trong hàm `updateBook(_:)`).
- Nút **Debug View Hierarchy**: Icon 3 hình chữ nhật xếp chồng trên thanh công cụ Debug Bar (cạnh nút Pause/Resume).

Ví dụ thao tác dòng lệnh LLDB trong Xcode Console:
```lldb
(lldb) po store.books.count
3

(lldb) po store.books.map { $0.title }
["Nhà Giả Kim", "Đắc Nhân Tâm", "Tuổi Trẻ Đáng Giá Bao Nhiêu"]

(lldb) expr store.books[0].title = "Nhà Giả Kim (Bản dịch mới)"
(lldb) continue
```

## Lỗi hay gặp
- **Viết biểu thức điều kiện sai cú pháp Swift**: Trong ô Condition của Breakpoint, gõ sai tên biến hoặc sai kiểu dữ liệu sẽ khiến Breakpoint bị bỏ qua hoặc in cảnh báo lỗi cú pháp trong console.
- **Dùng `po` với struct phức tạp bị crash LLDB**: Đôi khi `po` cố gọi reflection trên các kiểu dữ liệu nội bộ của SwiftUI và bị timeout. Trong trường hợp đó, hãy dùng lệnh `v` (`frame variable`) thay thế.
- **Quên resume (`continue`) sau khi debug xong**: App sẽ ở trạng thái đóng băng mãi mãi nếu bạn không bấm nút `Continue` (phím tắt `Ctrl + Command + Y`) hoặc gõ lệnh `c` trong LLDB.
- **Soi View Hierarchy khi view chưa render xong**: Cần đợi màn hình ổn định rồi mới bấm nút Debug View Hierarchy để có bản chụp chính xác nhất.

## Nhật ký một lần debug thực tế (iL-11)

- **Triệu chứng:** Khi chỉnh sửa bất kỳ cuốn sách nào trong `SachDoc`, nội dung bị ghi đè nhầm lên cuốn sách đầu tiên trong danh sách, đồng thời Xcode Console bắn cảnh báo đỏ: `ForEach<...>: the ID ... occurs multiple times within the collection, this will give undefined results!`.
- **Quá trình điều tra bằng công cụ:**
  - Đặt **Conditional Breakpoint** tại hàm `BookStore.updateBook(_:)` với điều kiện `book.title != "Nhà Giả Kim"` (chỉ dừng khi sửa cuốn sách khác cuốn đầu tiên).
  - Khi breakpoint dừng tại thao tác sửa `"Đắc Nhân Tâm"`, dùng **LLDB Console** kiểm tra trực tiếp trạng thái bộ nhớ:
    - `po book.title` → `"Đắc Nhân Tâm"`
    - `p index` → `([SachDoc.BookItem].Index) 1`
    - `po books[0].title` → `"Nhà Giả Kim"`
    - `expr books[index] = book` → thử cập nhật đúng phần tử trong RAM. Tuy nhiên ngay sau khi gõ `c` (Continue), dòng code lỗi `books[0] = book` tại breakpoint vẫn chạy, khiến `books[0]` và `books[1]` cùng mang một UUID và SwiftUI vẫn bắn cảnh báo đỏ trùng ID! Điều này chứng tỏ `expr` chỉ giúp *thử giả thuyết* tại chỗ, không thay thế được việc sửa code gốc.
  - **Kiểm tra thực tế bằng View Hierarchy Debugger:**
    - *Câu hỏi:* Vùng chạm của nút tròn đánh dấu đọc trong `BookRowView` thực tế có kích thước bao nhiêu `pt`? Có đạt chuẩn `≥ 44 pt` theo checklist 13.8 không?
    - *Chọn view trong cây:* Soi cây phân cấp `UIHostingController` → cell của `List` (`BookRowView`) → chọn lớp nút/icon tròn checkmark trên mô hình 3D.
    - *Size Inspector:* Cho kích thước `Width = 26 pt`, `Height = 26 pt` (đúng kích thước nội tại của SF Symbol `.title2`).
    - *Kết luận:* Vùng chạm `26 × 26 pt` **chưa đạt chuẩn tối thiểu `44 × 44 pt`** của checklist 13.8. Trong các màn hình sản phẩm `ChiTieu` sau này, cần bổ sung `.frame(minWidth: 44, minHeight: 44)` và `.contentShape(Rectangle())` cho các nút icon để đảm bảo trải nghiệm chạm thoải mái.
- **Nguyên nhân gốc rễ:** Lệnh gán `books[0] = book` ghi đè nhầm lên vị trí `0` thay vì vị trí `index`. Việc lấy đối tượng `book` (mang UUID của phần tử số 1) gán đè vào vị trí `0` làm mảng `books` chứa 2 phần tử trùng lặp `UUID`, dẫn đến cảnh báo lỗi Identity của `ForEach`.
- **Cách sửa:** Khôi phục lại câu lệnh đúng `books[index] = book` và xóa breakpoint tạm.

## Đọc thêm
- [Apple Developer – Debugging tools in Xcode](https://developer.apple.com/documentation/xcode/debugging)
- [Apple Developer – Inspecting the View Hierarchy](https://developer.apple.com/documentation/xcode/inspecting-the-view-hierarchy)
- [Apple Developer – Stepping through code and inspecting variables with LLDB](https://developer.apple.com/documentation/xcode/stepping-through-code-and-inspecting-variables)
