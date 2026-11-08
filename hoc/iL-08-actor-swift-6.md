# iL-08 · Concurrency: actor và Swift 6

## Vì sao cần
Trong lập trình đa luồng, Data Race (nhiều luồng cùng truy cập và sửa đổi một vùng nhớ) là nguyên nhân hàng đầu gây crash và lỗi dữ liệu khó tái hiện. Swift 6 mang đến bước đột phá lớn: chuyển toàn bộ việc phát hiện Data Race từ lúc chạy runtime thành lỗi biên dịch compile-time, sử dụng mô hình `actor`, giao thức `Sendable` và `@MainActor`.

## Khái niệm chính
- **`actor`**: Kiểu tham chiếu (reference type) tương tự `class`, nhưng tự động bảo vệ trạng thái nội tại bằng cơ chế Actor Isolation. Tại một thời điểm, chỉ duy nhất một tác vụ được quyền truy cập trạng thái của actor. Bên ngoài muốn truy cập trạng thái của actor bắt buộc phải dùng `await`.
- **Actor Reentrancy**: Khi một phương thức trong actor gặp điểm tạm dừng `await`, actor sẽ tạm thời giải phóng luồng để các tác vụ khác chen vào thực thi. Sau khi `await` xong và tiếp tục, trạng thái của actor có thể đã bị thay đổi bởi tác vụ chen ngang đó.
- **`@MainActor`**: Global actor đặc biệt chạy trên luồng chính (Main Thread). Bất kỳ class, hàm, hoặc view nào được đánh dấu `@MainActor` đều đảm bảo việc truy cập và cập nhật trạng thái UI luôn diễn ra an toàn trên main thread.
- **`Sendable` protocol**: Đánh dấu các kiểu dữ liệu an toàn khi truyền qua ranh giới giữa các luồng hoặc giữa các actor. Bao gồm: value type bất biến hoặc chỉ chứa thuộc tính `Sendable` (`struct`, `enum`), `actor`, hoặc `final class` bất biến có các thuộc tính `let` kiểu `Sendable`.
- **Swift 6 Language Mode**: Chế độ kiểm tra nghiêm ngặt (Complete Concurrency Checking). Trình biên dịch bắt buộc mọi dữ liệu truyền qua các tác vụ bất đồng bộ phải là `Sendable`, và mọi biến dùng chung phải được cô lập (isolated), loại bỏ hoàn toàn Data Race ngay từ lúc build.

## So với cách cũ bạn quen
| Cũ (Kotlin / Android) | Mới (Swift 6 Concurrency) | Khác biệt chính |
| --- | --- | --- |
| `synchronized`, `ReentrantLock`, `Mutex` | `actor` | Swift tích hợp actor ở cấp ngôn ngữ, tự quản lý hàng đợi thực thi, không cần lock/unlock thủ công, tránh deadlock |
| `@MainThread` / `Dispatchers.Main` | `@MainActor` | Swift kiểm tra compile-time; gọi hàm `@MainActor` từ background bắt buộc phải có `await` |
| Mọi object đều truyền được qua Coroutine (dễ data race) | Giao thức `Sendable` | Swift 6 cấm truyền reference type có thể thay đổi (`class var`) qua ranh giới concurrency |
| ThreadSanitizer phát hiện data race khi chạy | Swift 6 Compiler báo lỗi ngay khi build | Không còn data race lọt vào môi trường production |

## API và cú pháp cần biết
- `actor BankAccount { var balance: Int; func deposit(amount: Int) { balance += amount } }`: Khai báo actor bảo vệ số dư.
- `await account.deposit(amount: 100)`: Truy cập phương thức của actor từ bên ngoài phải có `await`.
- `@MainActor class ViewModel { ... }`: Cô lập toàn bộ ViewModel trên Main Thread.
- `struct Message: Sendable { let text: String }`: Khai báo struct an toàn khi gửi qua ranh giới concurrency.
- `nonisolated func getID() -> String`: Đánh dấu phương thức không truy cập trạng thái cô lập, gọi từ ngoài không cần `await`.

Ví dụ minh họa cú pháp (không liên quan bài toán SachDoc):
```swift
actor PrinterQueue {
    private var pendingJobs: [String] = []
    
    func addJob(_ job: String) {
        pendingJobs.append(job)
    }
    
    nonisolated var printerModel: String { "Canon-LBP2900" }
}
```

## Lỗi hay gặp
- **Truy cập thuộc tính của actor mà quên `await`**: Thuộc tính của actor bị cô lập, từ bên ngoài không thể đọc trực tiếp `let x = account.balance` mà phải `let x = await account.balance`.
- **Truyền mutable class qua Task mà không phải `Sendable`**: Trình biên dịch Swift 6 sẽ báo lỗi "passing argument of non-sendable type". Cần đổi thành `struct` hoặc `final class: Sendable` với các thuộc tính `let`.
- **Giả định trạng thái actor không đổi qua điểm `await` (Actor Reentrancy)**: Giữa 2 câu lệnh có `await`, một tác vụ khác có thể đã vào actor và thay đổi dữ liệu.
- **Biến toàn cục (Global variable) có thể thay đổi**: Trong Swift 6, `var globalCounter = 0` sẽ bị báo lỗi compiler vì không an toàn luồng. Cần chuyển vào actor hoặc `@MainActor`.

## 3 lỗi Data Race thực tế đã gặp và cách sửa (iL-08)

> **Lưu ý về Swift 6.4 & Region-Based Isolation (SE-0414):**
> Trong Swift 6.4, tính năng phân tích vùng cô lập (Region-Based Isolation) cho phép chuyển giao an toàn một object không Sendable nếu sau đó luồng hiện tại không còn dùng tới nó (không phát sinh race condition). Lỗi chỉ xảy ra khi object được dùng đồng thời ở cả hai nơi hoặc vượt qua ranh giới cô lập không thể chuyển giao.

1. **Lỗi 1: Truyền non-Sendable mutable class qua Task và tiếp tục truy cập ở luồng hiện tại**
   - *Tình huống gây lỗi:*
     ```swift
     let record = UnsafeBookRecord(title: "Swift 6")
     Task {
         print(record.title)
     }
     print(record.title) // Luồng hiện tại vẫn truy cập sau khi đã gửi vào Task!
     ```
   - *Thông báo lỗi từ Compiler 6.4:*
     `sending value of non-Sendable type '@concurrent () async -> ()' risks causing data races`
     (kèm ghi chú: `Passing value of non-Sendable type '@concurrent () async -> ()' as a 'sending' argument to initializer 'init(name:priority:operation:)' risks causing races in between local and caller code` và `access can happen concurrently`).
     Compiler gọi tên *closure* chứ không gọi tên `record`: closure chụp một object không Sendable nên chính closure cũng thành không Sendable.
   - *Nguyên nhân:* `class` là reference type có trạng thái biến đổi. Việc vừa gửi tham chiếu vào một `Task` nền vừa tiếp tục đọc/ghi ở luồng hiện tại sẽ tạo ra hai luồng cùng truy cập một vùng nhớ.
   - *Cách sửa:* Chuyển sang `struct` tuân thủ `Sendable` (value type được sao chép độc lập khi truyền qua Task) hoặc dùng `final class: Sendable` chỉ gồm thuộc tính `let`.

2. **Lỗi 2: Biến toàn cục có thể thay đổi (`var`)**
   - *Tình huống gây lỗi:*
     ```swift
     public var globalActiveReaderCount = 0
     ```
   - *Thông báo lỗi từ Compiler 6.4:*
     `var 'globalActiveReaderCount' is not concurrency-safe because it is nonisolated global shared mutable state`
   - *Nguyên nhân:* Biến toàn cục `var` không thuộc về bất kỳ actor nào, có thể bị đọc/ghi đồng thời từ mọi thread mà không có cơ chế đồng bộ.
   - *Cách sửa:* Đưa biến vào trong một `actor` (như `ReaderStatistics`) hoặc cô lập trên `@MainActor` (nếu là trạng thái UI).

3. **Lỗi 3: Truy cập thuộc tính `@MainActor` từ context đồng bộ không cô lập**
   - *Tình huống gây lỗi:*
     ```swift
     public func getTitleCount(viewModel: BookViewModel) -> Int {
         return viewModel.titles.count
     }
     ```
   - *Thông báo lỗi từ Compiler 6.4:*
     `main actor-isolated property 'titles' can not be referenced from a nonisolated context`
   - *Nguyên nhân:* `titles` được cô lập trên `@MainActor` (chỉ chạy trên main thread). Một hàm đồng bộ thông thường chạy ngoài main thread không thể tạm dừng để chuyển sang main thread nếu không có từ khóa `await`.
   - *Cách sửa:* Chuyển hàm thành `async` và dùng `await viewModel.titles.count`, hoặc đánh dấu chính hàm đó là `@MainActor`.

## Đọc thêm
- [The Swift Programming Language – Concurrency: Actors](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/#Actors)
- [Swift.org – Migrating to Swift 6](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/)
