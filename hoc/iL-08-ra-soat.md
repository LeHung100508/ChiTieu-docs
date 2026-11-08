# Rà soát iL-08 · Concurrency: actor và Swift 6

Ngày rà soát: 2026-11-07 · Nhánh `task/iL-08-actor-swift-6` · Trạng thái: **Xong** (đã rà lại sau khi sửa)

## Lệnh đã chạy
- `swift build --build-tests` (build lại toàn bộ) → sạch, không cảnh báo.
- `swift test` × 3 → 30 test trong 10 suite đều pass.
- Kiểm tra chéo: tạo package tạm có cùng cấu hình (tools 6.4, `ApproachableConcurrency`, Swift 6 mode), viết lại đoạn code gây lỗi theo ghi chép để xem compiler báo gì thật.

## Bắt buộc sửa (1)

### 1. Ghi chép 3 lỗi chưa khớp với compiler đang dùng
Vị trí: `docs/hoc/iL-08-actor-swift-6.md:47-62`.

| Lỗi | Đang ghi | Compiler 6.4 báo thật |
| --- | --- | --- |
| 1. Class không Sendable trong `Task { }` | `capture of 'record' with non-sendable type … in a @Sendable closure` | Không báo lỗi. Với `ApproachableConcurrency` và region-based isolation, `Task { }` chụp tham số không Sendable vẫn được chấp nhận |
| 2. Biến toàn cục | `global variable … is not concurrency-safe in Swift 6 mode` | `var '…' is not concurrency-safe because it is nonisolated global shared mutable state` |
| 3. Thuộc tính `@MainActor` | `…from a non-isolated context` | Khớp (compiler viết `nonisolated`) |

- **Vì sao:** tiêu chí là *gặp* lỗi do compiler báo rồi sửa. Hai câu đầu giống tài liệu viết cho Swift 5.10/6.0. Mục tiêu là quen cách compiler *đang dùng* báo lỗi, vì iS0–iS2 sẽ gặp đúng những lỗi này.
- **Hướng sửa:** trong package, viết tạm từng đoạn gây lỗi → `swift build` → chép nguyên văn thông báo → sửa → cập nhật ghi chép.
- **Riêng lỗi 1:** tìm tình huống compiler 6.4 thật sự chặn việc truyền object không Sendable. Lỗi kiểu mới thường có chữ `sending` và "risks causing data races". Thử so sánh `Task.detached` với `Task { }`, và trường hợp vẫn dùng object sau khi đã đưa nó vào task.

## Nên sửa (2)
1. **Comment đề bài còn sót** – `ActorPractice.swift` dòng 11, 17, 23, 40 ("Gán count cho bookId…", "Trả về stock[bookId] ?? 0"…). Chúng chỉ lặp lại dòng code bên dưới, thực chất là TODO. Xóa đi.
2. **Tên biến sai nghĩa** – `ActorPractice.swift:24`: `guard let book = stock[bookId]`. Giá trị là *số lượng*, không phải cuốn sách. Đặt lại tên cho đúng.

## Gợi ý thêm để học (3)
1. **Test check-then-act chặt hơn:** tồn kho 50, 100 task cùng mua → đúng 50 lần thành công, tồn kho về 0. Bắt được lỗi bán quá số lượng.
2. **Actor reentrancy:** `purchase` không có `await` bên trong nên kiểm tra và trừ kho liền một mạch. Nếu chèn một `await` (giả lập gọi cổng thanh toán) vào giữa `guard` và lệnh trừ thì chuyện gì có thể xảy ra?
3. `.swiftLanguageMode(.v6)` trong `Package.swift` là thừa vì tools-version 6.4 đã mặc định Swift 6. Giữ để ghi rõ ý định cũng được.

## Điểm làm tốt
- `BookInventory` giữ trạng thái `private` trong actor; test 100 task qua `withTaskGroup` đúng cách thử data race.
- Lỗi 3 có hai cách sửa (`async` hoặc `@MainActor`), có code minh họa cho cả hai.
- Không dùng `@unchecked Sendable` hay `nonisolated(unsafe)`, đúng checklist 13.8.

## Kết luận
Chưa đóng được task. Sửa mục Bắt buộc và 2 mục Nên sửa (hoặc `hoãn <mục>`), rồi báo lại để rà phần thay đổi.
