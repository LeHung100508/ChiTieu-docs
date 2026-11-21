# iS0-12 · Đo hiệu năng parser (NFR-02)

## Vì sao cần
Yêu cầu phi chức năng NFR-02 quy định parser phải hoàn thành phân tích câu trong vòng $\le 50$ ms để đảm bảo trải nghiệm nhập liệu tức thì (khi người dùng gõ câu trên bàn phím, giao diện phân tích mượt mà không bị khựng). Bài đo ở iS0 là phép đo sớm trên máy phát triển để phát hiện trước các điểm nghẽn thuật toán, Regex hay cấp phát bộ nhớ trước khi đo chính thức trên thiết bị thật ở iS4.

## Khái niệm chính
- **Swift Testing chưa có API đo hiệu năng:** Framework Swift Testing mới (`import Testing`) hiện tại chưa có API benchmark tương đương `measure` của XCTest. Do đó, ta có hai cách tiếp cận:
  1. Dùng `XCTest` với `measure { ... }` hoặc `measure(metrics: ...)` trong target test.
  2. Dùng `ContinuousClock` trong Swift để tự đo vi thời gian (nanoseconds/milliseconds) và tính toán phân vị chi tiết (p50, p95, max).
- **Đo vi thời gian với `ContinuousClock` (Swift 5.7+ / Swift 6):**
  - Đồng hồ đo thời gian đơn điệu (monotonic clock), không bị nhảy giờ hay ảnh hưởng khi người dùng/hệ thống chỉnh giờ.
  - Độ chính xác cấp nano giây.
- **Phân vị (Percentiles) - Nearest-Rank Method:**
  - **p50 (Median):** Mức thời gian mà 50% số câu chạy nhanh hơn.
  - **p95:** Mức thời gian mà 95% số câu chạy nhanh hơn. Đây là chỉ số quan trọng nhất của NFR-02 vì nó phản ánh độ trễ của các câu phức tạp (nhiều khoản, nhiều từ) mà không bị che giấu bởi giá trị trung bình (Average).
  - **Max:** Thời gian của câu chạy chậm nhất.
  - Công thức Nearest-Rank chuẩn: sau khi sắp xếp mảng thời gian tăng dần, chỉ số p95 là `Int(ceil(Double(count) * 0.95)) - 1`.
- **Khởi động (Warmup):**
  - Lần chạy đầu tiên thường tốn thêm thời gian nạp trang bộ nhớ (page fault) và khởi tạo cache nội bộ.
  - Cần chạy lặp thử 1–2 lượt trước khi bắt đầu bấm giờ đo chính thức.
- **Tránh tối ưu hóa Dead Code Elimination (DCE):**
  - Trình biên dịch Swift (`swiftc -O`) có thể nhận biết kết quả trả về không được sử dụng và tối ưu bằng cách bỏ qua toàn bộ khối mã. Cần cộng dồn kết quả (như tổng số giao dịch parse được) và kiểm tra bằng `#expect` hoặc `XCTAssert` sau vòng lặp.
- **Loại trừ chi phí I/O:**
  - Nạp sẵn toàn bộ 200 câu mẫu và `ParserConfiguration` vào bộ nhớ trước khi đo.
  - Tuyệt đối không đặt `print()`, ghi log hay đọc file đĩa bên trong vòng lặp đo.

## So với cách cũ bạn quen (Android S0-14)
| Đặc điểm | Android S0-14 (JVM) | iOS iS0-12 (Swift) |
| --- | --- | --- |
| Cơ chế thực thi | Bytecode chạy trên JVM, cần JIT compile | Mã máy biên dịch sẵn AOT (Ahead-of-Time) |
| Công cụ đo | `measureNanoTime {}` trong Kotlin | `ContinuousClock` trong Swift hoặc XCTest `measure` |
| Hiện tượng Warmup | Nặng (100–300 ms) do JIT compilation | Nhẹ hơn, chủ yếu là nạp trang bộ nhớ và cache Regex |
| Mục tiêu NFR-02 | $\le 50$ ms mỗi câu | $\le 50$ ms mỗi câu |

## API và cú pháp cần biết
- `ContinuousClock`:
  ```swift
  let clock = ContinuousClock()
  let duration = clock.measure {
      _ = parser.parse(input, referenceTime: now)
  }
  let milliseconds = Double(duration.components.attoseconds) / 1e15 + Double(duration.components.seconds) * 1_000.0
  ```
- Công thức tính p95:
  ```swift
  let sorted = durations.sorted()
  let p95Index = Int(ceil(Double(sorted.count) * 0.95)) - 1
  let p95 = sorted[p95Index]
  ```

## Lỗi hay gặp
1. **Khởi tạo parser hoặc nạp JSON từ điển trong vòng lặp đo:** Khiến ta đo thời gian đọc ổ đĩa và parse JSON thay vì đo tốc độ phân tích câu.
2. **Đặt `print()` trong vòng lặp đo:** I/O ra terminal làm tăng thời gian mỗi câu thêm từ vài ms đến hàng chục ms.
3. **Quên `sorted()` trước khi lấy p95:** Lấy nhầm phần tử chưa sắp xếp khiến giá trị phân vị sai lệch hoàn toàn.

## Đọc thêm
- [Apple: ContinuousClock](https://developer.apple.com/documentation/swift/continuousclock)
- [Đặc tả ChiTieu iOS: NFR-02](../dac-ta-ios.html)
- [Android S0-14 · Đo parser trên JVM](S0-14-do-parser-jvm.md)
