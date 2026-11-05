# iL-04 · Enum, Protocol và Generics trong Swift

## Vì sao cần
Hệ thống kiểu của Swift dựa rất mạnh vào lập trình hướng giao thức (Protocol-Oriented Programming) và `enum` nâng cao để mô hình hóa dữ liệu chính xác, an toàn mà không cần lạm dụng kế thừa phân cấp (`class inheritance`). Nắm vững enum có associated value, protocol extension, generics cùng hai từ khóa `some` và `any` là chìa khóa để làm chủ SwiftUI và viết parser sau này.

## Khái niệm chính
- **Enum với Associated Value**: Enum trong Swift không chỉ chứa hằng số đơn thuần mà mỗi case có thể gắn kèm dữ liệu thuộc các kiểu khác nhau (tương đương `sealed class` trong Kotlin). Bản thân enum là value type.
- **Exhaustive `switch`**: Biểu thức `switch` trên enum bắt buộc phải xét hết mọi trường hợp (vét cạn). Khi thêm một case mới vào enum, trình biên dịch sẽ cảnh báo ở tất cả các vị trí `switch` chưa xử lý case đó.
- **Protocol & Protocol Extension**: Tương tự `interface` trong Kotlin, nhưng khi kết hợp với `extension`, protocol có thể cung cấp **cài đặt mặc định** (default implementation) cho phương thức mà không cần tạo lớp cha trừu tượng (abstract class).
- **Generics & Type Constraints**: Viết hàm và cấu trúc dữ liệu tái sử dụng được cho nhiều kiểu, có thể ràng buộc điều kiện bằng `<T: SomeProtocol>` hoặc mệnh đề `where`.
- **`some` (Opaque Type)**: Trả về một kiểu cụ thể duy nhất tuân thủ protocol, nhưng ẩn chi tiết kiểu cụ thể đó đối với bên ngoài (ví dụ: `var body: some View` trong SwiftUI). Kiểu được chốt lúc biên dịch (static dispatch), tối ưu hiệu năng.
- **`any` (Existential Type)**: Đại diện cho một "chiếc hộp" (box) có thể chứa **bất kỳ** đối tượng nào tuân thủ protocol tại thời điểm runtime (dynamic dispatch). Tốn thêm chi phí bộ nhớ (existential container) và tra bảng con trỏ hàm (witness table).

## So với cách cũ bạn quen
| Cũ (Kotlin / Android) | Mới (Swift / iOS) | Khác biệt chính |
| --- | --- | --- |
| `sealed class` / `sealed interface` | `enum` với associated value | Swift dùng cú pháp `enum` súc tích hơn, là value type thuần (không cấp phát heap) |
| `when` vét cạn | `switch` vét cạn | Cả hai đều ép xử lý hết mọi case; Swift dùng `case .tên(let data):` để bóc tách dữ liệu |
| `interface` có phương thức mặc định | `protocol` + `extension` | Swift tách rời phần đặc tả giao diện (trong protocol) và phần cài đặt mặc định (trong extension) |
| Generics `<T : Comparable>` | Generics `<T: Comparable>` hoặc `where T: Comparable` | Tương đương về mặt tư duy |
| Đa hình runtime qua Interface (`List<Shape>`) | `[any Shape]` | Nên viết rõ `any` để thấy chi phí dynamic dispatch; Swift 6 chưa bắt buộc, chỉ bắt buộc khi bật upcoming feature `ExistentialAny` |
| Không có tương đương trực tiếp | `some Shape` (Opaque type) | Cố định một kiểu cụ thể lúc compile-time, dùng xuyên suốt trong SwiftUI |

## API và cú pháp cần biết
- `case .tênCase(let value)`: Trích xuất associated value trong nhánh `switch`.
- `extension MyProtocol { func helper() { ... } }`: Định nghĩa logic mặc định cho protocol.
- `func render() -> some Shape`: Opaque return type (caller chỉ biết nó là Shape).
- `[any MyProtocol]`: Mảng chứa các đối tượng có kiểu khác nhau cùng tuân thủ protocol.

Ví dụ minh họa cú pháp (không liên quan bài toán SachDoc):
```swift
enum LoadingState {
    case idle
    case loading(progress: Double)
    case success(message: String)
    case failure(reason: String)
}

protocol PrintableItem {
    func printInfo() -> String
}

extension PrintableItem {
    func printInfo() -> String { "Thông tin mặc định" }
}
```

## Lỗi hay gặp
- **Lạm dụng `any` khi chỉ cần `some`**: `any` có chi phí boxing và tra cứu bảng hàm runtime; trong Swift 6 nên ưu tiên `some` trừ khi cần một danh sách dị thể (heterogeneous array).
- **Quên xét hết case trong `switch`**: Trình biên dịch báo lỗi `Switch must be exhaustive`. Hạn chế lạm dụng `default:` vì sẽ đánh mất khả năng compiler nhắc nhở khi thêm case mới.
- **So sánh `==` trên enum có associated value**: Enum có associated value muốn dùng toán tử `==` thì enum đó phải khai báo tuân thủ `Equatable` (và các kiểu dữ liệu kèm theo cũng phải là `Equatable`).
- **Ghi đè hàm trong extension protocol**: Nếu hàm được khai báo trong protocol, struct cài đặt sẽ được dispatch động; nếu hàm CHỈ có trong extension mà không khai báo ở protocol header, khi ép kiểu về protocol nó sẽ luôn gọi hàm của extension (static dispatch).

## Đọc thêm
- [The Swift Programming Language – Enumerations](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations/)
- [The Swift Programming Language – Protocols](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/)
- [The Swift Programming Language – Generics](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/)
