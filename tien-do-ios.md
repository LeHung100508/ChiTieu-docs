# Tiến độ ChiTieu iOS

- **Repo GitHub:** https://github.com/LeHung100508/ChiTieu-ios.git
- **Task hiện tại:** iL-08
- **Lượt hiện tại:** iOS iL (lượt sau: Android S1). S0 đóng ngày 2026-11-01, S0-15 dời sang trước S5. Thứ tự luân phiên xem mục 13.3 của `docs/luat-agent.md`
- **Cập nhật lần cuối:** 2026-11-07

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
| iL-08 | Concurrency: actor và Swift 6 | 2,5 | Chưa làm | | |
| iL-09 | SwiftUI: view và state | 3 | Chưa làm | | |
| iL-10 | SwiftUI: Observable và điều hướng | 3 | Chưa làm | | |
| iL-11 | Debug | 1 | Chưa làm | | |
| iL-12 | Cổng 0 và tổng kết | 0,5 | Chưa làm | | |

## iS0 Tuần 7–8, sau Android S1 · Khung dự án và bộ phân tích câu

*(Chưa chia task chi tiết; xem `docs/task-ios.html` và `docs/ke-hoach-ios.html`)*

## iS1 Tuần 11–12, sau Android S2 · Dữ liệu, ví, danh mục, onboarding

*(Chưa chia task chi tiết)*

## iS2 Tuần 15–16, sau Android S3 · Nhập bằng câu từ đầu đến cuối

*(Chưa chia task chi tiết)*

## iS3 Tuần 19–20, sau Android S4 · Giao dịch, ngân sách, widget

*(Chưa chia task chi tiết)*

## iS4 Tuần 23–24, sau Android S5 · Thống kê và hoàn thiện MVP

*(Chưa chia task chi tiết)*

## iS5 Tuần 27–28 (+29) · TestFlight và phát hành App Store

*(Chưa chia task chi tiết)*

## iS6 Tuần 31–32 · AI trên máy và học từ người dùng

*(Chưa chia task chi tiết)*

## iS7 Tuần 35–36 · Nhập từ ảnh và lối tắt

*(Chưa chia task chi tiết)*

## iS8 Tuần 37–38 · Tự động hóa, iCloud, sao lưu, phát hành v1.1

*(Chưa chia task chi tiết)*

## Nợ kỹ thuật

| Ngày | Task | Mục hoãn | Xử lý trước task |
| --- | --- | --- | --- |

## Ngoại lệ (agent viết hộ)

| Ngày | Task | Phần được viết hộ | Lý do |
| --- | --- | --- | --- |

## Nhật ký sprint

(Cuối mỗi sprint: kết quả so với "Xong khi", việc tốt, việc chậm, thay đổi cho sprint sau.)
