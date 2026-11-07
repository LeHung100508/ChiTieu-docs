# iL-06 · ARC và Retain Cycle trong Swift

## Vì sao cần
Khác với Android dùng bộ dọn rác Garbage Collector (GC) định kỳ quét tìm các đối tượng rác trên heap, iOS dùng cơ chế đếm tham chiếu tự động **ARC (Automatic Reference Counting)**. ARC giải phóng bộ nhớ ngay lập tức khi reference count về 0, nhưng hoàn toàn bất lực trước **Retain Cycle** (vòng lặp tham chiếu mạnh giữ nhau). Nắm vững ARC, `weak`, `unowned` và capture list trong closure là điều kiện tiên quyết để ứng dụng iOS chạy ổn định và không bị tràn bộ nhớ (OOM crash).

## Khái niệm chính
- **Cơ chế ARC**: Mỗi khi một biến tham chiếu mạnh (`strong reference`) trỏ vào một instance của `class`, reference count tăng 1; khi biến ra khỏi scope hoặc được gán bằng `nil`, reference count giảm 1. Khi count = 0, hàm hủy `deinit` được gọi và bộ nhớ heap được thu hồi ngay lập tức.
- **Retain Cycle (Vòng tham chiếu mạnh)**: Xảy ra khi đối tượng A giữ tham chiếu mạnh tới đối tượng B, và đối tượng B cũng giữ tham chiếu mạnh ngược lại tới A (hoặc một closure thuộc về A giữ mạnh `self`). Khi đó reference count của cả hai không bao giờ về 0, gây rò rỉ bộ nhớ vĩnh viễn dù không còn biến nào bên ngoài trỏ tới chúng.
- **Tham chiếu yếu (`weak`)**: Không làm tăng reference count. Khi đối tượng mà nó trỏ tới bị giải phóng, biến `weak` **tự động chuyển thành `nil`** (zeroing weak reference). Do đó, `weak` bắt buộc phải là biến (`var`) và có kiểu Optional (`T?`).
- **Tham chiếu không sở hữu (`unowned`)**: Cũng không làm tăng reference count, nhưng giả định đối tượng được trỏ tới **luôn luôn còn sống** chừng nào biến này còn tồn tại. Nếu đối tượng bị giải phóng trước mà ta vẫn truy cập qua `unowned`, ứng dụng sẽ crash ngay lập tức (`fatalError`).
- **Capture List trong Closure**: Khi closure truy cập thuộc tính hoặc hàm của `self`, closure sẽ mặc định giữ tham chiếu mạnh tới `self`. Nếu closure đó lại được lưu làm thuộc tính của `self`, vòng lặp tham chiếu sẽ hình thành. Để phá vỡ, ta dùng capture list: `{ [weak self] in guard let self else { return } ... }`.
- **Memory Graph Debugger**: Công cụ đồ họa tích hợp sẵn trong Xcode (biểu tượng 3 hình tròn nối nhau trên thanh debug bar lúc app đang chạy), hiển thị toàn bộ quan hệ tham chiếu trên heap và đánh dấu cảnh báo màu tím cho các Retain Cycle.

## So với cách cũ bạn quen
| Cũ (Kotlin / Android) | Mới (Swift / iOS) | Khác biệt chính |
| --- | --- | --- |
| Garbage Collector (GC) | Automatic Reference Counting (ARC) | ARC giải phóng ngay khi count = 0, không có hiện tượng giật lag để dọn rác (stop-the-world) |
| Vòng tham chiếu tròn vẫn được GC dọn nếu tách rời khỏi GC Root | Vòng tham chiếu mạnh (Retain Cycle) bị rò rỉ vĩnh viễn | Android ít khi lo ngại 2 object giữ nhau; iOS bắt buộc lập trình viên phải chủ động bẻ gãy bằng `weak` |
| Rò rỉ context qua Inner Class / Handler / Callback | Rò rỉ `self` qua Closure / Delegate | Cơ chế gây lỗi tương đồng; Swift giải quyết triệt để bằng `[weak self]` trong capture list |
| `WeakReference<T>` | Từ khóa cú pháp `weak var x: T?` | Swift tích hợp `weak` ở cấp độ ngôn ngữ, tự động gán `nil` mà không cần gọi `.get()` |
| Không có tương đương trực tiếp | Từ khóa cú pháp `unowned let x: T` | Dùng khi hai đối tượng có vòng đời gắn liền hoặc đối tượng được trỏ sống lâu hơn |

## API và cú pháp cần biết
- `deinit`: Hàm hủy tự động được gọi khi reference count = 0 (tương tự `finalize()` nhưng được đảm bảo thực thi ngay lập tức).
- `weak var delegate: SomeDelegate?`: Phá retain cycle trong mô hình ủy quyền (Delegate pattern).
- `{ [weak self] in ... }`: Capture list trong closure tránh giữ mạnh `self`.

Ví dụ minh họa cú pháp (không liên quan bài toán SachDoc):
```swift
class Order {
    var customer: Customer?
    deinit { print("Order đã giải phóng") }
}

class Customer {
    weak var order: Order? // Dùng weak để phá retain cycle
    deinit { print("Customer đã giải phóng") }
}
```

## Lỗi hay gặp
- **Lạm dụng `unowned` thay vì `weak`**: Dùng `unowned` khi đối tượng có thể bị giải phóng trước sẽ gây crash runtime: `Fatal error: Attempted to read an unowned reference but the object was already deallocated`. Nguyên tắc: Nếu không chắc chắn 100% về vòng đời, luôn ưu tiên `weak`.
- **Quên `[weak self]` trong closure thoát scope (`@escaping`)**: Nếu closure được gán vào thuộc tính hoặc truyền vào các tác vụ chạy ngầm bất đồng bộ mà giữ mạnh `self`, màn hình đã đóng nhưng ViewModel/Controller vẫn sống dai dẳng.
- **Nhầm tưởng `struct` bị retain cycle**: Struct là Value Type, lưu trực tiếp giá trị chứ không lưu con trỏ tham chiếu nên không bao giờ xảy ra retain cycle. Retain cycle chỉ xuất hiện trên `class`.

## Đọc thêm
- [The Swift Programming Language – Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)
- [Apple Developer – Viewing the Memory Graph](https://developer.apple.com/documentation/xcode/viewing-the-memory-graph-of-a-process)
