# iL-03 · Optional, Value Type và Swift Testing

## Vì sao cần
Swift xây dựng tính an toàn bộ nhớ và hạn chế lỗi runtime dựa trên hai trụ cột: **Optional** (loại bỏ hoàn toàn biệt đội lỗi NullPointerException) và **Value Type** (ngăn chặn tình trạng thay đổi dữ liệu ngầm ngoài ý muốn). Nắm vững hai khái niệm này cùng framework **Swift Testing** là bước khởi đầu bắt buộc trước khi bước vào viết code sản phẩm.

## Khái niệm chính
- **Optional (`T?`)**: Bản chất là một enum có 2 trạng thái: `.none` (tương đương `nil`) và `.some(Wrapped)`. Muốn truy cập giá trị bên trong, bắt buộc phải "mở hộp" (unwrap) an toàn.
- **Unwrap với `if let` và `guard let`**: `if let` tạo biến đã mở hộp chỉ sống trong phạm vi `{ ... }`. Ngược lại, `guard let` kiểm tra điều kiện sớm (early exit), bắt buộc phải thoát (`return`, `throw`) nếu gặp `nil`, và biến đã mở hộp sẽ tiếp tục được sử dụng ở phạm vi bên ngoài.
- **Value Type (`struct`) vs Reference Type (`class`)**: 
  - Khi gán hoặc truyền `struct` vào hàm, một bản sao độc lập hoàn chỉnh được tạo ra (copy-on-assignment). Thay đổi bản sao không làm thay đổi bản gốc.
  - Khi gán `class`, cả hai biến cùng trỏ vào một đối tượng duy nhất trên bộ nhớ heap. Thay đổi qua một biến sẽ ảnh hưởng đến biến còn lại.
- **`mutating func`**: Thuộc tính của `struct` mặc định không thể bị sửa đổi bởi các phương thức nội bộ. Nếu một phương thức cần thay đổi thuộc tính của struct, nó phải được khai báo với từ khóa `mutating`, và biến gọi phương thức đó phải là `var`.
- **`let` và `var` với struct**: Nếu một instance của `struct` được khai báo bằng `let`, toàn bộ instance đó là bất biến (immutable), bạn không thể thay đổi bất kỳ trường nào dù trường đó được khai báo bằng `var`.
- **Swift Testing**: Framework kiểm thử hiện đại của Apple đi kèm Swift 6. Dùng macro `@Test` để đánh dấu hàm test, `#expect(...)` để kiểm tra điều kiện (không dừng test nếu fail), và `#require(...)` để unwrap hoặc kiểm tra điều kiện bắt buộc (dừng ngay test nếu fail).

## So với cách cũ bạn quen
| Cũ (Kotlin / Android) | Mới (Swift / iOS) | Khác biệt chính |
| --- | --- | --- |
| `String?`, `?.`, `?:`, `!!` | `String?`, `?.`, `??`, `!` | Cú pháp tương đồng; toán tử elvis `?:` của Kotlin đổi thành `??` trong Swift |
| Smart cast: `if (x != null) x.length` | `if let x { x.count }` hoặc `guard let x else { return }` | Swift không tự smart cast biến optional mà đòi hỏi unwrap rõ ràng |
| `data class` (reference type, muốn sao chép dùng `.copy()`) | `struct` (value type thuần, tự động sao chép khi gán) | `struct` trong Swift lưu trên stack/inline, gán biến là tạo bản copy độc lập, không dùng chung tham chiếu |
| `class` (kế thừa, reference type) | `class` (kế thừa, reference type) | Hoàn toàn tương tự: dùng chung vùng nhớ trên heap |
| JUnit 5: `@Test`, `assertEquals(a, b)` | Swift Testing: `@Test`, `#expect(a == b)` | Macro `@Test` và `#expect` kiểm tra biểu thức tự nhiên, thông báo lỗi trực quan |

## API và cú pháp cần biết
- `??` (Nil-Coalescing Operator): Trả về giá trị mặc định nếu optional là `nil`.
- `?.` (Optional Chaining): Gọi thuộc tính/phương thức an toàn, dừng lại và trả về `nil` nếu gặp chuỗi `nil`.
- `mutating`: Cho phép hàm của struct sửa đổi thuộc tính nội tại.
- `#expect(a == b)`: Kiểm tra điều kiện đúng trong Swift Testing.

Ví dụ minh họa cú pháp (không liên quan bài toán SachDoc):
```swift
struct Point {
    var x: Int
    var y: Int
    mutating func moveRight(by step: Int) {
        x += step
    }
}

func parseScore(text: String?) -> Int {
    guard let text, let score = Int(text) else {
        return 0
    }
    return score
}
```

## Lỗi hay gặp
- **Dùng force unwrap (`!`)**: Gặp `nil` sẽ gây crash ứng dụng ngay lập tức (`fatalError`). Chỉ dùng khi 100% chứng minh được giá trị không thể `nil`.
- **Nhầm tưởng struct chia sẻ biến**: Sửa giá trị trên biến copy rồi thắc mắc vì sao struct ban đầu không đổi.
- **Quên từ khóa `mutating`**: Trình biên dịch sẽ báo lỗi `Cannot use mutating member on immutable value`.
- **Dùng `let` cho struct rồi cố gọi hàm mutating**: Bị báo lỗi biên dịch vì instance khai báo bằng `let` không cho phép gọi hàm `mutating`.
- **Nhầm `#expect` với `#require`**: `#expect` không dừng test khi thất bại; nếu muốn unwrap một giá trị để test tiếp các dòng sau, phải dùng `let unwrapped = try #require(optionalVal)`.

## Đọc thêm
- [The Swift Programming Language – Optionals](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/#Optionals)
- [The Swift Programming Language – Structures and Classes](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures/)
- [Apple Developer – Meet Swift Testing](https://developer.apple.com/documentation/testing)
