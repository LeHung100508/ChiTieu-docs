# iL-05 · Closure, Xử lý lỗi và Codable trong Swift

## Vì sao cần
Closure là nền tảng của các hàm thao tác dữ liệu và xây dựng giao diện SwiftUI (`ViewBuilder`). Cơ chế xử lý lỗi tường minh (`throws`, Typed Throws, `Result`) cùng giao thức `Codable` là công cụ chuẩn mực của Swift để tuần tự hóa và giải mã dữ liệu (JSON parsing), chuẩn bị trực tiếp cho việc đọc bộ câu mẫu và dữ liệu từ điển ở các sprint tiếp theo.

## Khái niệm chính
- **Closure và cú pháp rút gọn**: Closure là một khối mã có thể lưu trữ và truyền đi (tương đương Lambda trong Kotlin). Swift hỗ trợ cú pháp rút gọn linh hoạt: tự suy luận kiểu tham số, trailing closure (đưa khối mã ra ngoài ngoặc tròn nếu là tham số cuối), và dùng `$0`, `$1` thay cho tên tham số tường minh.
- **Hàm bậc cao (`map`, `filter`, `reduce`)**: Thao tác trên tập hợp tương tự Kotlin. `map` biến đổi từng phần tử, `filter` lọc theo điều kiện, `reduce` tích lũy toàn bộ phần tử về một giá trị duy nhất.
- **Xử lý lỗi với `throws` và `try`**: Swift yêu cầu mọi hàm có khả năng ném lỗi phải đánh dấu `throws`. Khi gọi hàm, bắt buộc phải dùng từ khóa `try`:
  - `try`: Thường đi kèm với khối `do { ... } catch { ... }` để xử lý lỗi tường minh.
  - `try?`: Chuyển kết quả hàm thành Optional (`T?`), nếu có lỗi ném ra thì tự động trả về `nil`.
  - `try!`: Ép buộc hàm phải chạy thành công, nếu có lỗi sẽ gây crash ứng dụng (tránh dùng).
- **Typed Throws (Tính năng mới của Swift 6)**: Cho phép chỉ định chính xác kiểu lỗi mà hàm có thể ném ra (ví dụ: `throws(BookError)`). Nhờ đó khối `catch` bắt được đúng kiểu lỗi cụ thể mà không cần ép kiểu từ `any Error`.
- **`Result<Success, Failure>`**: Kiểu enum chuẩn đại diện cho kết quả thành công `.success(value)` hoặc thất bại `.failure(error)`. Rất quen thuộc với Android developer.
- **`Codable` (`Encodable` & `Decodable`)**: Protocol chuẩn của Swift để parse dữ liệu. Khi tên khóa trong JSON khác với tên thuộc tính trong Swift, ta dùng `enum CodingKeys: String, CodingKey` để ánh xạ tùy biến.

## So với cách cũ bạn quen
| Cũ (Kotlin / Android) | Mới (Swift / iOS) | Khác biệt chính |
| --- | --- | --- |
| Lambda `{ it.name }` | Closure `{ $0.name }` | Kotlin dùng `it`, Swift dùng `$0`, `$1`... |
| Trailing lambda `list.map { ... }` | Trailing closure `list.map { ... }` | Cú pháp và tư duy hoàn toàn tương đương |
| Unchecked Exception (`throw Exception()`) | Checked `throws` (`throw MyError.notFound`) | Swift bắt buộc phải xử lý hoặc ném tiếp lỗi có `throws`, không có ngoại lệ ngầm |
| Class `Result<T>` | Enum `Result<Success, Failure>` | Swift tích hợp sẵn `Result` trong thư viện chuẩn dưới dạng enum |
| Gson / Moshi / kotlinx.serialization | `Codable` (`JSONDecoder`, `JSONEncoder`) | `Codable` được tích hợp sâu vào compiler, không cần thư viện ngoài |
| `@SerializedName("book_id")` | `enum CodingKeys: String, CodingKey { case id = "book_id" }` | Swift dùng enum `CodingKeys` an toàn kiểu compile-time thay vì annotation reflection |

## API và cú pháp cần biết
- `func fetch() throws(NetworkError) -> String`: Typed Throws chỉ định rõ kiểu lỗi.
- `JSONDecoder().decode(T.self, from: data)`: Giải mã JSON thành đối tượng `T`.
- `enum CodingKeys: String, CodingKey`: Ánh xạ tên thuộc tính Swift sang khóa JSON.

Ví dụ minh họa cú pháp (không liên quan bài toán SachDoc):
```swift
struct City: Decodable {
    let name: String
    let population: Int?
    
    enum CodingKeys: String, CodingKey {
        case name = "city_name"
        case population
    }
}
```

## Lỗi hay gặp
- **Quên liệt kê đủ case trong `CodingKeys`**: Khi đã tự khai báo `CodingKeys`, bạn bắt buộc phải liệt kê **toàn bộ** các thuộc tính cần decode, kể cả các thuộc tính có tên trùng với JSON. Trường nào bị quên sẽ không được decode.
- **Không khai báo trường Optional**: Nếu một trường trong JSON có thể bị thiếu (`missing`) hoặc mang giá trị `null`, thuộc tính tương ứng bắt buộc phải là Optional (`T?`). Nếu để non-optional, `JSONDecoder` sẽ ném lỗi `keyNotFound` hoặc `valueNotFound`.
- **Dùng `try!`**: Khi decode JSON từ mạng hoặc file, không bao giờ dùng `try!` vì chỉ cần JSON sai định dạng là app sẽ crash ngay lập tức.
- **Truyền trực tiếp chuỗi vào `JSONDecoder`**: `JSONDecoder.decode` nhận vào đối tượng kiểu `Data`, không nhận `String`. Cần chuyển chuỗi sang `Data` bằng `Data(jsonString.utf8)`.

## Đọc thêm
- [The Swift Programming Language – Closures](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/)
- [The Swift Programming Language – Error Handling](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling/)
- [Apple Developer – Encoding and Decoding Custom Types](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)
