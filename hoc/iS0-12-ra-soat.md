# Rà soát iS0-12 · Đo hiệu năng parser (NFR-02)

Ngày rà soát: 2026-11-21 · Nhánh `task/iS0-12-hieu-nang` · Trạng thái: **Xong** (đóng 2026-11-21 sau rà lại lần 1)

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ❌ 10 lỗi ở `ParserBenchmarkTests.swift:63-72` (chuỗi nhiều dòng trong `print`) |
| `swiftlint lint --strict` | ❌ `identifier_name`: tham số `p` ở `:85` ngắn hơn 3 ký tự |
| `swift test --filter ParserBenchmarkTests` (Debug, chạy 2 lần) | ✅ mean 2,10 và 2,13 ms; p50 2,10 và 2,13 ms; p95 2,36 ms; max 2,95 và 2,94 ms |
| `swift test -c release -Xswiftc -enable-testing --filter ParserBenchmarkTests` | ✅ mean 1,92 ms; p50 1,92 ms; p95 2,08 ms; max 2,28 ms |

Máy đo: máy Mac phát triển (macOS, Apple Silicon), không phải iPhone. Câu chậm nhất vẫn nhanh hơn ngưỡng 50 ms khoảng **17 lần**.

## Bắt buộc sửa (1)

### 1. Lint không qua
- **Vị trí:** `ParserBenchmarkTests.swift:63-72` và `:85`.
- **Vì sao:** CI chạy đúng hai lệnh lint này với `--strict`, nên PR sẽ bị chặn. Đây là lần thứ ba liên tiếp báo xong khi lint chưa sạch.
- **Hướng sửa:**
  - Chuỗi `"""` nhiều dòng: nội dung và dấu `"""` đóng phải thụt lề vào trong, ngang hàng với dòng mở. Swift sẽ bỏ phần thụt lề chung đó khỏi chuỗi.
  - Đổi tên tham số `p`, ví dụ thành `fraction`.
  - Chạy lại cả hai lệnh lint chuẩn trước khi báo xong.

## Nên sửa (2)

### 1. Task yêu cầu đo bằng XCTest `measure`
- **Vấn đề:** chi tiết task ghi "Đo bằng XCTest (`measure`)", và phần cần học có `XCTestCase` cùng `measure(metrics:)`. Bản hiện tại dùng `ContinuousClock` trong Swift Testing.
- **Đánh giá:** cách dùng `ContinuousClock` **đúng và còn tốt hơn** cho mục tiêu "mỗi câu ≤ 50 ms", vì nó đo được từng câu và tính được p95, max. `measure` của XCTest chỉ cho tổng thời gian của cả khối.
- **Hướng sửa:** chọn một trong hai cách.
  - Thêm một `XCTestCase` ngắn dùng `measure(metrics: [XCTClockMetric()])` cho cả 200 câu, để học API này (Swift Package chạy được cả XCTest lẫn Swift Testing).
  - Hoặc giữ nguyên, và ghi lý do chọn `ContinuousClock` vào tiến độ như một quyết định của bạn.

### 2. Kiểm tra p95 trong khi yêu cầu là "mỗi câu"
- **Vị trí:** `:75`.
- **Vì sao:** NFR-02 ghi "xử lý **một câu** ≤ 50 ms". Kiểm tra p95 thì 5% số câu (10 câu) có thể vượt ngưỡng mà test vẫn xanh.
- **Hướng sửa:** kiểm tra `maxDuration`, có thể giữ thêm p95 để tham khảo.
- **Lưu ý thêm:** test đo thời gian chạy trên runner dùng chung của CI có thể dao động. Hiện tại cách ngưỡng rất xa nên rủi ro thấp. Nếu sau này có lúc đỏ ngẫu nhiên, nên tách test đo hiệu năng khỏi CI, hoặc chỉ in số mà không kiểm tra.

## Gợi ý thêm để học (3)
1. **Bản Release chỉ nhanh hơn Debug khoảng 10%.** Điều đó cho thấy phần lớn thời gian là chi phí cố định mỗi lần gọi `parse`, chưa phải logic chính:
   - `WalletMatcher` tách token cho mọi bí danh **ở mỗi lần gọi**;
   - `DateParser.parse` chạy hai lần, một lần trong `resolve` và một lần trong `ClauseAnalyzer`.

   Đây là phần agent viết. Hiện chưa cần tối ưu (khoảng 2 ms), nhưng nên ghi lại để xem ở iS4, khi đo trên thiết bị thật.
2. **Số đo trên Mac thường nhanh hơn iPhone** vài lần. Phép đo chính thức ở iS4 phải chạy trên thiết bị, hoặc ít nhất trên simulator.
3. **File lý thuyết:** link NFR-02 đang trỏ tới `dac-ta-android.html`. Bản iOS có NFR-02 riêng trong `dac-ta-ios.html`, với cách kiểm tra là "test đo thời gian trên bộ 200 câu mẫu".

## Điểm làm tốt
- Đặt mọi việc chuẩn bị ngoài vòng đo: nạp bộ mẫu, từ điển, tạo `ParserConfiguration` và parser.
- Có hai lượt chạy khởi động trước khi đo, và có biến cộng dồn kết quả để compiler không bỏ qua phần code được đo.
- Đo từng câu và báo p50, p90, p95, max, mean. Công thức nearest-rank đúng. Đổi `Duration` sang mili giây đúng (`attoseconds / 1e15`).

## Kết luận
Chưa đóng được task. Cần sửa Bắt buộc 1. Nên sửa 1 và 2 thì sửa, hoặc ghi `hoãn`. Khi đóng task, mentor sẽ ghi số đo vào tiến độ.

---

## Rà lại lần 1 (2026-11-21)
**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅ 35 test Swift Testing và 1 test XCTest.
- `xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) → ✅.

**Đã đạt:**
- **Bắt buộc 1:** ✅ Chuỗi `"""` đã thụt lề đúng, tham số đổi tên thành `fraction`.
- **Nên sửa 1:** ✅ Thêm `ParserXCTestBenchmarks` dùng `measure(metrics: [XCTClockMetric()])`. Kết quả: trung bình **0,412 s cho 200 câu**, tức khoảng **2,06 ms/câu**, độ lệch chuẩn tương đối 0,54%. Số này khớp với phép đo `ContinuousClock` (mean khoảng 2,1–2,2 ms). Hai cách đo xác nhận lẫn nhau.
- **Nên sửa 2:** ✅ Có kiểm tra `maxDuration <= 50`, đúng với yêu cầu "mỗi câu" của NFR-02. Vẫn giữ p95 để tham khảo.

**Số đo đã ghi vào tiến độ** (máy Mac phát triển, 200 câu mẫu):

| Cách đo | mean | p95 | max |
| --- | --- | --- | --- |
| `ContinuousClock`, Debug | ≈ 2,1–2,2 ms | 2,4 ms | 2,9–3,0 ms |
| `ContinuousClock`, Release | 1,9 ms | 2,1 ms | 2,3 ms |
| XCTest `measure` | 2,06 ms/câu (0,412 s cho 200 câu) | | |

Cả ba cách đều đạt yêu cầu ≤ 50 ms mỗi câu. Phép đo chính thức trên thiết bị thật sẽ làm ở iS4.

**Kết luận:** Đóng được task.
