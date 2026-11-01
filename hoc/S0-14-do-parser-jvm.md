# S0-14 · Đo hiệu năng bộ phân tích trên JVM (NFR-02)

## Vì sao cần
NFR-02 quy định ứng dụng phải hoàn tất phân tích câu tự nhiên trong vòng ≤ 50 ms để không gây cảm giác khựng cho người dùng khi gõ phím. Bài đo trên JVM ở S0-14 giúp phát hiện sớm các điểm nghẽn hiệu năng (thuật toán tốn thời gian, Regex lặp lại, cấp phát bộ nhớ thừa) trước khi đưa pipeline xuống thiết bị thật ở S4-10.

## Khái niệm chính
- **Warmup JVM (Khởi động máy ảo):** Khi một hàm chạy lần đầu trên JVM, thời gian thực thi thường cao bất thường do nạp class (ClassLoading), biên dịch JIT (Just-In-Time) từ bytecode sang machine code, và nạp tài nguyên tĩnh (`category_keywords.json`). Giai đoạn warmup cho pipeline chạy lặp lại trước để đưa JIT về trạng thái tối ưu ổn định (HotSpot).
- **Phân vị 95 (p95 Percentile - Nearest-Rank):** Điểm giá trị mà tại đó 95% số câu được xử lý nhanh hơn hoặc bằng mức này, và chỉ có 5% số câu chậm hơn. Chỉ số p95 phản ánh độ trễ của các câu phức tạp (đa khoản, nhiều từ) tốt hơn nhiều so với giá trị trung bình (Average) vốn dễ che giấu các đột biến giật lag (spikes). Công thức Nearest-Rank chuẩn: `ceil(size * p) - 1`.
- **Tránh JIT Dead Code Elimination (DCE):** Trình biên dịch C2 của JVM rất thông minh; nếu mã bên trong vùng đo trả về kết quả nhưng kết quả đó không bao giờ được đọc hoặc sử dụng, JIT có thể tối ưu bằng cách bỏ qua toàn bộ khối mã đó. Ta cần gom kết quả vào một biến tích luỹ (Blackhole) và `assertTrue` sau vòng lặp để ép JIT phải tính toán thật.
- **Cold Start vs Warm JVM:** Khi chạy toàn bộ test suite, các test trước có thể đã nạp sẵn class và từ điển vào bộ nhớ. Cần có bài test đo riêng lần gọi đầu tiên và kiểm tra trên JVM mới độc lập để phát hiện thời gian khởi tạo thực tế (~117–154 ms khi nạp và giải mã JSON từ điển), từ đó lên kế hoạch nạp trước (preload) ngoài luồng chính.
- **Độ chính xác nano giây:** `System.currentTimeMillis()` phụ thuộc vào đồng hồ hệ thống và độ phân giải thấp (1–15 ms). Để đo vi thời gian (micro-benchmark), bắt buộc dùng `System.nanoTime()` hoặc hàm `measureNanoTime {}` của thư viện chuẩn Kotlin.
- **Loại trừ chi phí I/O:** Trong vòng lặp đo đạc, không in log (`println`, `Log.d`), không đọc ổ đĩa hay parse lại file JSON bộ mẫu. Dữ liệu đầu vào phải được nạp sẵn vào bộ nhớ trước khi bấm giờ.

## So với cách cũ bạn quen
| Cách cũ | Cách chuẩn (Micro-benchmark) | Khác biệt chính |
| :--- | :--- | :--- |
| `System.currentTimeMillis()` trước và sau | `measureNanoTime {}` chuyển sang milliseconds | Độ phân giải cao (nano giây), không phụ thuộc chỉnh giờ hệ thống |
| Tính trung bình cộng (Mean) | Tính phân vị p50, p90, p95, max | Bắt được trường hợp câu khó nhất bị chậm (outliers) |
| Chạy 1 lần rồi kết luận | Warmup 10–20 vòng, sau đó đo 50–100 vòng lặp | Loại bỏ nhiễu do classloading và bộ thu gom rác (GC) |
| Đặt log ngay trong vòng lặp | Tách riêng mảng ghi nhận kết quả, in tổng hợp sau | Log ra console tốn từ vài ms đến hàng chục ms, làm sai lệch kết quả |

## API và cú pháp cần biết
- `kotlin.system.measureNanoTime`: Đo thời gian thực thi của một lambda tính bằng nano giây.
- Công thức tính p95: Sau khi sắp xếp danh sách thời gian tăng dần, chỉ số phần tử p95 là `ceil(0.95 * totalCases) - 1`.

```kotlin
// Minh họa cách đo thời gian, dùng Blackhole và tính p95 chuẩn Nearest-Rank
val durationsMs = mutableListOf<Double>()
var blackholeSum = 0

for (task in sampleTasks) {
    val elapsedNanos = measureNanoTime {
        val result = task.execute()
        blackholeSum += result.hashCode() // Tránh Dead Code Elimination
    }
    durationsMs.add(elapsedNanos / 1_000_000.0)
}
check(blackholeSum != 0) // Tiêu thụ kết quả Blackhole

durationsMs.sort()
val p95Index = ceil(durationsMs.size * 0.95).toInt() - 1
val p95Value = durationsMs[p95Index]
```

## Lỗi hay gặp
1. **Quên warmup JVM:** Vòng chạy đầu tiên của JVM có thể mất từ 100–300 ms do JIT compilation. Nếu không warmup trước khi đo, p95 sẽ bị kéo vượt ngưỡng 50 ms một cách oan uổng.
2. **Khởi tạo tài nguyên bên trong vùng đo:** Khởi tạo `Clock`, nạp từ điển JSON hoặc tạo danh sách `mockWallets` bên trong block đo lường khiến ta đo chi phí đọc ổ đĩa/khởi tạo object thay vì đo thuật toán phân tích câu.
3. **Phân vị không chính xác:** Không sắp xếp mảng kết quả trước khi lấy chỉ số phần trăm dẫn đến việc lấy ngẫu nhiên một phần tử giữa chừng.
4. **Biên dịch Regex mỗi lần gọi:** Tạo Regex mới (`Regex(...)` hoặc `split("\\s+".toRegex())`) trong thân hàm phân tích câu thay vì định nghĩa tĩnh ở cấp file làm tăng thời gian xử lý và cấp phát GC liên tục.

## Đọc thêm
- [Kotlin measureNanoTime API Reference](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.system/measure-nano-time.html)
- [How to avoid JVM benchmarking pitfalls](https://wiki.openjdk.org/display/HotSpot/MicroBenchmarks)
