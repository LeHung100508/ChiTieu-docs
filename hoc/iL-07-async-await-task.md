# iL-07 · Swift Concurrency: async/await và Task

## Vì sao cần
Trước Swift 5.5, lập trình bất đồng bộ trên iOS dựa chủ yếu vào Grand Central Dispatch (GCD) và callback closure lồng nhau ("callback hell"), dễ dẫn đến rò rỉ bộ nhớ hoặc quên gọi completion handler. Swift Concurrency mang đến cú pháp `async`/`await` và mô hình **Bất đồng bộ có cấu trúc (Structured Concurrency)**, giúp mã bất đồng bộ đọc tuần tự, sáng sủa, đồng thời kiểm soát tài nguyên và luồng dữ liệu an toàn ngay từ lúc biên dịch.

## Khái niệm chính
- **`async` và `await`**: Hàm đánh dấu `async` là hàm có thể tạm dừng (suspend). Từ khóa `await` đánh dấu điểm tạm dừng tiềm năng (suspension point), nơi luồng hiện tại được giải phóng để làm việc khác trong khi chờ kết quả mà không hề làm nghẽn luồng (non-blocking).
- **Thực thi tuần tự vs Song song (`async let`)**:
  - Tuần tự: `let a = try await fetchA(); let b = try await fetchB()` (chờ xong A rồi mới bắt đầu gọi B).
  - Song song: `async let a = fetchA(); async let b = fetchB(); let (resA, resB) = try await (a, b)` (cả A và B cùng chạy đồng thời, chỉ chờ khi gom kết quả).
- **`Task`**: Đơn vị công việc bất đồng bộ cơ bản, tương tự `Coroutine` trong Kotlin. `Task { ... }` tạo một tác vụ mới kế thừa ngữ cảnh (priority, actor context) của nơi gọi.
- **`Task.sleep`**: Tạm dừng task trong một khoảng thời gian (nanoseconds hoặc `Duration`), không chặn luồng và hỗ trợ hủy (tương đương `delay()` trong Kotlin).
- **Cooperative Cancellation (Hủy hợp tác)**: Trong Swift, việc hủy task không ép buộc luồng dừng đột ngột mà chỉ đặt cờ hủy (`isCancelled = true`). Tác vụ đang chạy phải chủ động kiểm tra cờ này:
  - `Task.isCancelled`: Kiểm tra trả về giá trị `Bool`.
  - `try Task.checkCancellation()`: Kiểm tra và tự động ném `CancellationError` nếu task đã nhận tín hiệu hủy.
- **Structured Concurrency (Bất đồng bộ có cấu trúc)**: Các tác vụ con sinh ra bởi `async let` có vòng đời gắn chặt với tác vụ cha: nếu tác vụ cha bị hủy hoặc ném lỗi, tất cả các tác vụ con đang chạy ngầm sẽ tự động nhận tín hiệu hủy.

## So với cách cũ bạn quen
| Cũ (Kotlin / Coroutines) | Mới (Swift Concurrency) | Khác biệt chính |
| --- | --- | --- |
| `suspend fun` | `func ... async` | Cả hai đều đánh dấu hàm có thể tạm dừng; Swift yêu cầu từ khóa `await` tại nơi gọi |
| `delay(1000)` | `try await Task.sleep(for: .seconds(1))` | Cả hai đều giải phóng luồng, tự động ném lỗi khi bị hủy |
| `async { ... }.await()` | `async let x = ...; await x` | Swift hỗ trợ cú pháp `async let` tích hợp rất ngắn gọn cho các tác vụ con song song |
| `launch { ... }` | `Task { ... }` | Tạo một tác vụ mới chạy độc lập |
| `ensureActive()` / `isActive` | `try Task.checkCancellation()` / `Task.isCancelled` | Cơ chế hủy hợp tác (Cooperative Cancellation) hoàn toàn tương đồng |
| `job.cancel()` | `task.cancel()` | Gửi tín hiệu yêu cầu hủy tác vụ |

## API và cú pháp cần biết
- `func loadData() async throws -> String`: Khai báo hàm bất đồng bộ có ném lỗi.
- `async let first = taskA()`: Chạy tác vụ con đồng thời.
- `try await Task.sleep(for: .milliseconds(100))`: Tạm dừng task 100 ms.
- `Task.isCancelled`: Thuộc tính kiểm tra trạng thái hủy.
- `try Task.checkCancellation()`: Ném `CancellationError` nếu đã bị hủy.

Ví dụ minh họa cú pháp (không liên quan bài toán SachDoc):
```swift
func downloadAll() async throws -> (Data, Data) {
    async let image = downloadImage()
    async let text = downloadText()
    return try await (image, text) // Chạy song song cả hai tác vụ
}
```

## Lỗi hay gặp
- **Quên `try` khi gọi `Task.sleep`**: `Task.sleep` sẽ ném `CancellationError` nếu task bị hủy trong lúc đang ngủ, do đó bắt buộc phải gọi bằng `try await Task.sleep(...)`.
- **Nghĩ rằng `task.cancel()` sẽ dừng ngay một vòng lặp nặng**: Hủy trong Swift là hủy hợp tác. Nếu vòng lặp CPU nặng không chủ động kiểm tra `Task.checkCancellation()`, nó vẫn sẽ tiếp tục chạy cho đến hết.
- **Lạm dụng `Task.detached`**: `Task.detached` tách rời hoàn toàn khỏi ngữ cảnh cha (mất actor context, mất quyền ưu tiên). Trừ khi cần chạy việc nền đặc thù, luôn ưu tiên `Task { }` tiêu chuẩn hoặc `async let`.

## Đọc thêm
- [The Swift Programming Language – Concurrency](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/concurrency/)
- [Apple Developer – Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/)
