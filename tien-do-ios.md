# Tiến độ ChiTieu iOS

- **Repo GitHub:** https://github.com/LeHung100508/ChiTieu-ios.git
- **Task hiện tại:** iS0-02 (Chia package)
- **Lượt hiện tại:** iOS iS0 (lượt sau: Android S1; được làm song song S1). Ngày 2026-11-12 người dùng đổi thứ tự luân phiên thành iSn ngay sau Sn, để giữ nhịp iOS sau iL. iL đóng ngày 2026-11-12. S0 đóng ngày 2026-11-01, S0-15 dời sang trước S5. Thứ tự luân phiên xem mục 13.3 của `docs/luat-agent.md`
- **Cập nhật lần cuối:** 2026-11-14

Trạng thái: Chưa làm · Đang làm · Chờ rà soát · Cần sửa · Xong. Quy trình xem `docs/luat-agent.md`.

> File này chỉ là bản ghi phụ. Căn cứ chính để xác định tiến độ là source code và lịch sử git (commit `[iL-xx]`, `[iSx-yy]`), xem mục 4, 9 và 13.6 của luật.

## iL Tuần 3–4, sau Android S0 · Làm quen Swift, Xcode và SwiftUI

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| iL-01 | Tạo repo iOS | 1 | Xong | 2026-11-03 | Khởi tạo repo iOS, .gitignore Xcode/SwiftPM, gắn submodule docs/, CLAUDE.md & AGENTS.md chuẩn |
| iL-02 | Cài Xcode, chạy app đầu tiên | 1,5 | Xong | 2026-11-04 | Xcode 27.2, chạy SachDoc thành công trên Simulator và iPhone thật; Deployment Target 26.0 |
| iL-03 | Swift: optional và value type | 2 | Xong | 2026-11-05 | Package SwiftCoBan, 5 test Swift Testing xanh; có test biên parseAge("0") và test struct sao chép, class dùng chung |
| iL-04 | Swift: enum, protocol, generics | 2,5 | Xong | 2026-11-05 | 9 test xanh; trả lời đạt 3 câu về some/any, generic với mảng dị thể, so sánh với sealed class và interface của Kotlin |
| iL-05 | Swift: closure, lỗi, Codable | 2 | Xong | 2026-11-06 | 15 test xanh; đọc books.json qua Bundle.module, CodingKeys, trường optional; test lỗi phân biệt dataCorrupted và keyNotFound |
| iL-06 | ARC và retain cycle | 1,5 | Xong | 2026-11-07 | 18 test xanh; test giải phóng cho weak (2 object, closure) và unowned (Book–Bookmark); trả lời đạt 4 câu weak/unowned, Memory Graph, struct không cần weak self |
| iL-07 | Concurrency: async/await và Task | 1,5 | Xong | 2026-11-07 | 24 test xanh; async let, test nhánh lỗi, test hủy chứng minh được checkCancellation và isCancelled có tác dụng |
| iL-08 | Concurrency: actor và Swift 6 | 2,5 | Xong | 2026-11-08 | 31 test xanh, build sạch Swift 6 mode; ghi lại 3 lỗi thật (sending non-Sendable vào Task, biến toàn cục, @MainActor từ nonisolated) đã đối chiếu với compiler 6.4 |
| iL-09 | SwiftUI: view và state | 3 | Xong | 2026-11-09 | Book, BookRowView (@Binding), BookListView, ContentView (@State, một NavigationStack); preview tương tác bằng @Previewable, có trạng thái rỗng và tên dài; giải thích đúng thứ tự padding/background; build sạch |
| iL-10 | SwiftUI: Observable và điều hướng | 3 | Xong | 2026-11-09 | BookStore @Observable có add/update/book(for:); điều hướng bằng id với navigationDestination(for: UUID.self); thêm và sửa qua sheet trên bản nháp; @Bindable cho $store.books; đã chạy đúng trên iPhone thật; build sạch |
| iL-11 | Debug | 1 | Xong | 2026-11-11 | Nhật ký debug lỗi cài vào (books[0] = book gây trùng ID trong ForEach): breakpoint có điều kiện, po/p/expr, View Hierarchy đo nút tròn 26×26 pt (chưa đạt 44 pt); build sạch |
| iL-12 | Cổng 0 và tổng kết | 0,5 | Xong | 2026-11-12 | Cổng 0 đạt: SwiftCoBan 31 test xanh, app đã chạy trên iPhone thật, tự giải thích đạt 5 khái niệm (ARC, actor isolation, identity cần hỏi lại một lượt) |

## iS0 Tuần 5–6, sau iL (đối chiếu Android S0) · Khung dự án và bộ phân tích câu

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| iS0-01 | Tạo project | 2 | Xong | 2026-11-14 | Project `ChiTieu` ở gốc repo, iOS 26.0, Swift 6, mặc định MainActor, thư mục đồng bộ, shared scheme; chốt Bundle ID `hung.lct.chitieu`, App Group ID `group.hung.lct.chitieu` (chưa bật capability); đã chạy trên iPhone thật |
| iS0-02 | Chia package | 2,5 | Chưa làm | |  |
| iS0-03 | Chất lượng mã | 1,5 | Chưa làm | |  |
| iS0-04 | CI | 1,5 | Chưa làm | |  |
| iS0-05 | Design system: màu và chữ | 1,5 | Chưa làm | |  |
| iS0-06 | Design system: component | 1 | Chưa làm | |  |
| iS0-07 | Model parser và đọc bộ câu mẫu | 2,5 | Chưa làm | |  |
| iS0-08 | Parser: chuẩn hóa và tách từ | 1,5 | Chưa làm | |  |
| iS0-09 | Parser: số tiền (FR-02) | 2 | Chưa làm | | Giải thuật do agent viết theo mục 2 |
| iS0-10 | Parser: thời gian (FR-03) | 1,5 | Chưa làm | | Giải thuật do agent viết theo mục 2 |
| iS0-11 | Parser: danh mục và tách khoản (FR-04, FR-07) | 2 | Chưa làm | | Giải thuật do agent viết theo mục 2 |
| iS0-12 | Đo hiệu năng parser (NFR-02) | 1 | Chưa làm | |  |
| iS0-13 | Đối chiếu với Android | 1 | Chưa làm | |  |

## iS1 Tuần 9–10, sau Android S1 · Dữ liệu, ví, danh mục, onboarding

*(Chưa chia task chi tiết)*

## iS2 Tuần 13–14, sau Android S2 · Nhập bằng câu từ đầu đến cuối

*(Chưa chia task chi tiết)*

## iS3 Tuần 17–18, sau Android S3 · Giao dịch, ngân sách, widget

*(Chưa chia task chi tiết)*

## iS4 Tuần 21–22, sau Android S4 · Thống kê và hoàn thiện MVP

*(Chưa chia task chi tiết)*

## iS5 Tuần 25–26 (+27), sau Android S5 · TestFlight và phát hành App Store

*(Chưa chia task chi tiết)*

## iS6 Tuần 29–30, sau Android S6 · AI trên máy và học từ người dùng

*(Chưa chia task chi tiết)*

## iS7 Tuần 33–34, sau Android S7 · Nhập từ ảnh và lối tắt

*(Chưa chia task chi tiết)*

## iS8 Tuần 37–38, sau Android S8 · Tự động hóa, iCloud, sao lưu, phát hành v1.1

*(Chưa chia task chi tiết)*

## Nợ kỹ thuật

| Ngày | Task | Mục hoãn | Xử lý trước task |
| --- | --- | --- | --- |

## Ngoại lệ (agent viết hộ)

| Ngày | Task | Phần được viết hộ | Lý do |
| --- | --- | --- | --- |

## Nhật ký sprint

(Cuối mỗi sprint: kết quả so với "Xong khi", việc tốt, việc chậm, thay đổi cho sprint sau.)

### iL · 2026-11-02 → 2026-11-12 (khoảng 1,5 tuần, kế hoạch 2 tuần)
- **Kết quả so với "Xong khi" (Cổng 0):** đạt. `luyen-tap/SwiftCoBan` có 31 test, 10 suite, tất cả xanh. SachDoc đã chạy trên iPhone thật. Người dùng tự giải thích được 5 khái niệm. Với ARC, actor isolation và identity, mentor phải hỏi lại một lượt. 12/12 task xong, không có nợ kỹ thuật.
- **Việc tốt:** theo người dùng, mọi thứ đều ổn.
- **Việc chậm:** người dùng không thấy việc nào chậm.
- **Thay đổi cho sprint sau:** người dùng chưa muốn đổi gì.
- **Quan sát của mentor:** cần nhắc từ S1 và iS0.
  - Để mentor đổi cột trạng thái trong tiến độ. Người dùng đã nhiều lần tự ghi Xong trước khi rà soát.
  - Build và chạy test trước khi báo xong. Ở iL-11 có lần báo xong khi code còn BUILD FAILED.
  - Khi tự viết tài liệu `hoc/`, đối chiếu với tài liệu chính thức. Tài liệu của iL-10 và iL-11 có vài chỗ sai về API.
  - Ở Cổng 0, trả lời bằng lời của mình, không dán.
  - Chủ đề cần ôn lại ở iS0: điều kiện để một `struct` là `Sendable`; optional chaining làm phẳng kiểu kết quả.
