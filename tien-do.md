# Tiến độ ChiTieu Android

- **Repo GitHub:** https://github.com/LeHung100508/ChiTieu.git
- **Task hiện tại:** S0-05
- **Cập nhật lần cuối:** 2026-10-15

Trạng thái: Chưa làm · Đang làm · Chờ rà soát · Cần sửa · Xong. Quy trình xem `docs/luat-agent.md`.

> File này chỉ là bản ghi phụ. Căn cứ chính để xác định tiến độ là source code và lịch sử git (commit `[Sx-yy]`), xem mục 4 và 9 của luật.

## S0 Tuần 1–2 · Khung dự án và bộ phân tích câu

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S0-01 | Tạo project | 1 | Xong | 2026-10-13 | Đã chốt hung.lct.chitieu, JDK 17, Compose, AGP 9 |
| S0-02 | Version Catalog | 1 | Xong | 2026-10-14 | Đầy đủ thư viện dự kiến, KSP, Hilt, Room, Nav3, bundles |
| S0-03 | Convention plugins | 3 | Xong | 2026-10-14 | Đủ 6 plugin chitieu.*, build-logic, SDK 37, AGP 9 |
| S0-04 | Khung module | 1,5 | Xong | 2026-10-15 | Đủ 15 module (8 core, 6 feature, benchmarks), compileSdk 37, assembleDebug thành công |
| S0-05 | Chất lượng mã | 1 | Chưa làm | | |
| S0-06 | CI | 1,5 | Chưa làm | | |
| S0-07 | Design system: màu và chữ | 1,5 | Chưa làm | | |
| S0-08 | Design system: component | 1,5 | Chưa làm | | |
| S0-09 | Bộ câu mẫu | 1,5 | Chưa làm | | |
| S0-10 | Parser: chuẩn hóa và tách từ | 1,5 | Chưa làm | | |
| S0-11 | Parser: số tiền (FR-02) | 2 | Chưa làm | | |
| S0-12 | Parser: thời gian (FR-03) | 1,5 | Chưa làm | | |
| S0-13 | Parser: danh mục và tách khoản (FR-04, FR-07) | 1,5 | Chưa làm | | |
| S0-14 | Đo parser trên JVM (NFR-02) | 1 | Chưa làm | | |
| S0-15 | Tài khoản Google Play | 1 | Chưa làm | | |

## S1 Tuần 3–4 · Dữ liệu, ví, danh mục, onboarding

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S1-01 | Domain model | 1 | Chưa làm | | |
| S1-02 | Room: entity và schema | 2 | Chưa làm | | |
| S1-03 | DAO trả về Flow | 2 | Chưa làm | | |
| S1-04 | Test DAO và khung migration | 1,5 | Chưa làm | | |
| S1-05 | Danh mục mặc định | 1,5 | Chưa làm | | |
| S1-06 | Repository và Hilt | 2 | Chưa làm | | |
| S1-07 | DataStore | 1,5 | Chưa làm | | |
| S1-08 | Onboarding (FR-34) | 3 | Chưa làm | | |
| S1-09 | Màn Ví (FR-23) | 2 | Chưa làm | | |
| S1-10 | Màn Danh mục (FR-24) | 2 | Chưa làm | | |
| S1-11 | Cài đặt cơ bản (FR-35) | 2 | Chưa làm | | |
| S1-12 | Test ViewModel | 1 | Chưa làm | | |

## S2 Tuần 5–6 · Nhập bằng câu từ đầu đến cuối

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S2-01 | Khung app với Navigation 3 | 2,5 | Chưa làm | | |
| S2-02 | Edge-to-edge và predictive back | 1 | Chưa làm | | |
| S2-03 | Cài Roborazzi | 1 | Chưa làm | | |
| S2-04 | Trang chủ: tổng và gần đây | 2,5 | Chưa làm | | |
| S2-05 | Máy trạng thái Nhập nhanh | 2 | Chưa làm | | |
| S2-06 | Ô nhập và thẻ nháp (FR-01, FR-08) | 3 | Chưa làm | | |
| S2-07 | Bộ chọn cho chip | 2 | Chưa làm | | |
| S2-08 | Parser: loại và ví (FR-05, FR-06) | 2 | Chưa làm | | |
| S2-09 | Nhiều khoản, lưu một lần (FR-07) | 1 | Chưa làm | | |
| S2-10 | Giọng nói (FR-13) | 2 | Chưa làm | | |
| S2-11 | Test luồng nhập | 2 | Chưa làm | | |
| S2-12 | Demo và sửa lỗi | 1 | Chưa làm | | |

## S3 Tuần 7–8 · Giao dịch, ngân sách, widget

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S3-01 | Danh sách giao dịch (FR-18) | 2,5 | Chưa làm | | |
| S3-02 | Dữ liệu mẫu 10.000 giao dịch | 0,5 | Chưa làm | | |
| S3-03 | Tìm kiếm FTS (FR-20) | 1,5 | Chưa làm | | |
| S3-04 | Bộ lọc | 1,5 | Chưa làm | | |
| S3-05 | Màn hình sửa (FR-19) | 2,5 | Chưa làm | | |
| S3-06 | Xóa và hoàn tác | 1,5 | Chưa làm | | |
| S3-07 | Ngân sách: dữ liệu | 1,5 | Chưa làm | | |
| S3-08 | Ngân sách: giao diện (FR-25) | 2,5 | Chưa làm | | |
| S3-09 | Activity Nhập nhanh | 1,5 | Chưa làm | | |
| S3-10 | Widget Glance (FR-14) | 3 | Chưa làm | | |
| S3-11 | Danh sách và chi tiết trên tablet | 1,5 | Chưa làm | | |
| S3-12 | Tuyển tester | 0,5 | Chưa làm | | |
| S3-13 | Test và kiểm tra cuộn | 1 | Chưa làm | | |

## S4 Tuần 9–10 · Thống kê và hoàn thiện MVP

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S4-01 | Truy vấn thống kê | 2 | Chưa làm | | |
| S4-02 | State của kỳ thống kê | 1 | Chưa làm | | |
| S4-03 | Biểu đồ | 2,5 | Chưa làm | | |
| S4-04 | Màn Thống kê (FR-27) | 2 | Chưa làm | | |
| S4-05 | Bộ chọn tháng | 1,5 | Chưa làm | | |
| S4-06 | Xuất CSV (FR-30) | 2 | Chưa làm | | |
| S4-07 | Khóa app (FR-32) | 2,5 | Chưa làm | | |
| S4-08 | Khả năng truy cập (NFR-11) | 2 | Chưa làm | | |
| S4-09 | Đa ngôn ngữ (NFR-12) | 1,5 | Chưa làm | | |
| S4-10 | Hiệu năng (NFR-01 đến NFR-03) | 2,5 | Chưa làm | | |
| S4-11 | Rà tổng thể (Cổng 2) | 2 | Chưa làm | | |

## S5 Tuần 11–12, chờ duyệt tuần 13 · Phát hành lên Google Play

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S5-01 | Ký app | 1 | Chưa làm | | |
| S5-02 | R8 và kích thước (NFR-04) | 2 | Chưa làm | | |
| S5-03 | Crashlytics (NFR-05, NFR-08) | 1,5 | Chưa làm | | |
| S5-04 | Chính sách quyền riêng tư | 1,5 | Chưa làm | | |
| S5-05 | Data safety và phân loại nội dung | 1 | Chưa làm | | |
| S5-06 | Trang giới thiệu trên Play | 2 | Chưa làm | | |
| S5-07 | Phát hành tự động | 2 | Chưa làm | | |
| S5-08 | Internal testing | 0,5 | Chưa làm | | |
| S5-09 | Closed testing | 1,5 | Chưa làm | | |
| S5-10 | Sửa lỗi từ tester | 7 | Chưa làm | | |
| S5-11 | Xin quyền Production (tuần 13) | 1 | Chưa làm | | |
| S5-12 | Theo dõi sau phát hành | 1 | Chưa làm | | |

## S6 Tuần 14–15 · AI trên máy và học từ người dùng

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S6-01 | Migration v1.1 | 1 | Chưa làm | | |
| S6-02 | Học từ chỉnh sửa (FR-09) | 2 | Chưa làm | | |
| S6-03 | Tab Từ điển | 1 | Chưa làm | | |
| S6-04 | Gợi ý tự hoàn thành (FR-12) | 3 | Chưa làm | | |
| S6-05 | Kiểm tra Gemini Nano | 1,5 | Chưa làm | | |
| S6-06 | Prompt và schema (FR-10) | 2,5 | Chưa làm | | |
| S6-07 | Đánh giá AI | 1 | Chưa làm | | |
| S6-08 | Hỏi đáp theo luật (FR-29) | 3 | Chưa làm | | |
| S6-09 | Hỏi đáp với Gemini | 2 | Chưa làm | | |
| S6-10 | Test và phát hành nội bộ | 2 | Chưa làm | | |

## S7 Tuần 16–17 · Nhập từ ảnh và lối tắt

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S7-01 | Ô Cài đặt nhanh (FR-15) | 1,5 | Chưa làm | | |
| S7-02 | App shortcuts (FR-15) | 1 | Chưa làm | | |
| S7-03 | Share target (FR-16) | 1,5 | Chưa làm | | |
| S7-04 | OCR ảnh | 2 | Chưa làm | | |
| S7-05 | Đọc màn hình chuyển khoản | 3 | Chưa làm | | |
| S7-06 | Quét hóa đơn (FR-17) | 2 | Chưa làm | | |
| S7-07 | Lấy dòng tổng tiền | 2 | Chưa làm | | |
| S7-08 | Đính kèm ảnh | 2 | Chưa làm | | |
| S7-09 | So sánh kỳ trước (FR-28) | 2 | Chưa làm | | |
| S7-10 | Test và phát hành nội bộ | 2 | Chưa làm | | |

## S8 Tuần 18–19 · Tự động hóa, sao lưu, phát hành v1.1

| Task | Việc | Giờ | Trạng thái | Ngày xong | Ghi chú |
| --- | --- | --- | --- | --- | --- |
| S8-01 | Giao dịch định kỳ: dữ liệu và giao diện | 2,5 | Chưa làm | | |
| S8-02 | Worker định kỳ (FR-21) | 1,5 | Chưa làm | | |
| S8-03 | Quyền và kênh thông báo | 1 | Chưa làm | | |
| S8-04 | Thông báo ngân sách (FR-26) | 1 | Chưa làm | | |
| S8-05 | Nhắc hằng ngày (FR-36) | 1,5 | Chưa làm | | |
| S8-06 | Định dạng file sao lưu (NFR-16) | 1,5 | Chưa làm | | |
| S8-07 | Mã hóa file (NFR-07) | 1,5 | Chưa làm | | |
| S8-08 | Sao lưu lên Google Drive (FR-31) | 3 | Chưa làm | | |
| S8-09 | Khôi phục | 1,5 | Chưa làm | | |
| S8-10 | Ẩn nội dung trong đa nhiệm (FR-33) | 0,5 | Chưa làm | | |
| S8-11 | Test sao lưu và worker | 2 | Chưa làm | | |
| S8-12 | Phát hành v1.1 | 1,5 | Chưa làm | | |

## Nợ kỹ thuật

| Ngày | Task | Mục hoãn | Xử lý trước task |
| --- | --- | --- | --- |

## Ngoại lệ (agent viết hộ)

| Ngày | Task | Phần được viết hộ | Lý do |
| --- | --- | --- | --- |

## Nhật ký sprint

(Cuối mỗi sprint: kết quả so với "Xong khi", việc tốt, việc chậm, thay đổi cho sprint sau.)
