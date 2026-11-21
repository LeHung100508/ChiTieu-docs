# Tiến độ ChiTieu iOS

- **Repo GitHub:** https://github.com/LeHung100508/ChiTieu-ios.git
- **Task hiện tại:** iS1 (chưa chia task; làm sau Android S1)
- **Lượt hiện tại:** Android S1 (iOS iS0 đóng 2026-11-21, đạt Cổng 1; lượt sau: iOS iS1). Ngày 2026-11-12 người dùng đổi thứ tự luân phiên thành iSn ngay sau Sn, để giữ nhịp iOS sau iL. iL đóng ngày 2026-11-12. S0 đóng ngày 2026-11-01, S0-15 dời sang trước S5. Thứ tự luân phiên xem mục 13.3 của `docs/luat-agent.md`
- **Cập nhật lần cuối:** 2026-11-21

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
| iS0-02 | Chia package | 2,5 | Xong | 2026-11-15 | Làm lại từ đầu sau lần 1. Local package `Core` (Model, Parser, Data, DesignSystem, CoreTesting) và `Features` (6 feature), mỗi module có test target; luật phụ thuộc ghi đầu mỗi `Package.swift`; DesignSystem và Feature mặc định MainActor; app chỉ link 6 feature, 4 tab theo wireframe, ⚙ mở Cài đặt |
| iS0-03 | Chất lượng mã | 1,5 | Xong | 2026-11-16 | `.swift-format` (4 space, 120 ký tự) và `.swiftlint.yml` với `swiftlint_version: 0.65.1`; cài bằng release `portable_swiftlint.zip`; lệnh chuẩn có `--strict`, phạm vi ChiTieu, Core, Features (không gồm `luyen-tap/`) |
| iS0-04 | CI | 1,5 | Xong | 2026-11-18 | GitHub Actions trên runner `xcode-27`, checkout kèm submodule, SwiftLint 0.65.1 tải bằng release có kiểm SHA-256, lint `--strict`, `swift test` Core và Features kèm coverage ra Step Summary, build app; `permissions: contents: read`; ruleset `protect-main` active (PR và check `Build, Lint & Test`); PR #3 xanh |
| iS0-05 | Design system: màu và chữ | 1,5 | Xong | 2026-11-19 | `Colors.xcassets` 5 màu sáng/tối đạt WCAG AA; màu thương hiệu iOS `#1E7A4E` theo mockup iOS (chốt 2026-11-19, Android giữ `#1B6B4A`); `AppTypography` dựa trên text style hệ thống (số tiền dùng `.rounded` và `monospacedDigit`); `AppSpacing`, `AppRadius` là `nonisolated`; 3 `#Preview` (Sáng, Tối, AX5); test asset và token chạy trên macOS lẫn iOS. Phần lớn code viết theo ngoại lệ |
| iS0-06 | Design system: component | 1 | Xong | 2026-11-19 | `AmountText` (màu mặc định `textPrimary`), `CategoryChip` (vùng chạm 44 pt trong label, `.isSelected`), `CategoryIconView`, `SummaryCard` theo mockup Tổng quan, `BudgetProgressCard`; `Int64.formattedVND(locale:)` theo `@Environment(\.locale)` (NFR-12); ngưỡng 80%/100% đặt một chỗ, có test; Preview sáng và tối. Chuỗi hard-code hoãn sang iS1 |
| iS0-07 | Model parser và đọc bộ câu mẫu | 2,5 | Xong | 2026-11-20 | `Money(Int64)` (Codable số trần, literal); `ParsedTransaction` với `ParsedField<Value>` (độ tin cậy 0–1 từng trường), ví nguồn/đích cho chuyển khoản; test đọc 200 câu qua `#filePath` (macOS, simulator, CI), giải mã `Date`/`TransactionType`; test có tham số `arguments: try`, `withKnownIssue(isIntermittent:)`. So sánh từng trường hoãn sang trước iS0-09 |
| iS0-08 | Parser: chuẩn hóa và tách từ | 1,5 | Xong | 2026-11-20 | `Tokenizer.tokenize` là cửa vào duy nhất (`normalize` gọi qua): regex literal có capture đặt tên, token word/number/punctuation/newline (`\R` bắt cả `\r\n`), bản bỏ dấu (xử lý `đ`), vị trí theo `Character` trên `raw`; test tham số 8 ca biên (NFD, CRLF). Code viết theo ngoại lệ; người dùng trả lời đạt 5 câu kiểm tra hiểu |
| iS0-09 | Parser: số tiền (FR-02) | 2 | Xong | 2026-11-20 | Giải thuật do agent viết theo mục 2. `AmountParser`: 8 mẫu theo thứ tự ưu tiên, `đ`/`vnd` là ký hiệu không nhân, phần lẻ theo số chữ số, chống tràn số, 5 mức độ tin cậy; 200/200 câu mẫu (Cổng 1) và 19 câu ngoài mẫu; người dùng nối vào `TransactionParser` (`datetime` để `nil` tới iS0-10) và trả lời đạt 5 câu kiểm tra |
| iS0-10 | Parser: thời gian (FR-03) | 1,5 | Xong | 2026-11-20 | Giải thuật do agent viết theo mục 2. `DateParser` inject `now` và `Calendar`; buổi so có dấu; chống ngày tương lai, ngày sai trả rỗng; lan ngữ cảnh và ghép buổi vào ngày được lan; 200/200 câu mẫu, 14 câu ngoài mẫu, nửa đêm, đầu tháng, năm mới; người dùng nối vào `TransactionParser`, kiểm chặt `datetime` và trả lời đạt 5 câu |
| iS0-11 | Parser: danh mục và tách khoản (FR-04, FR-07) | 2 | Xong | 2026-11-21 | Giải thuật do agent viết theo mục 2: `ClauseSplitter`, `CategoryDictionary` (cụm dài thắng, từ đơn ngắn phải có dấu), `WalletMatcher` (bí danh dài thắng), `ClauseAnalyzer` (loại, danh mục, ví, ghi chú cắt từ `raw`), `ParserConfiguration`; 200/200 câu mẫu đủ 6 trường. Người dùng: từ điển vào bundle app dạng tham chiếu, `ChiTieuApp` nạp và ghi lỗi bằng `Logger`, `TransactionParser(configuration:)`, kiểm chặt các trường trong test; trả lời đạt 5 câu |
| iS0-12 | Đo hiệu năng parser (NFR-02) | 1 | Xong | 2026-11-21 | Đo trên Mac, 200 câu mẫu. `ContinuousClock` từng câu: Debug mean ≈ 2,1 ms, p95 2,4 ms, max ≈ 3,0 ms; Release mean 1,9 ms, max 2,3 ms. XCTest `measure` (`XCTClockMetric`): 0,412 s/200 câu ≈ 2,06 ms/câu. Đạt ≤ 50 ms. CI chỉ chặn p95; max có thể vượt trên runner dùng chung. Ghi chú cho iS4: Release chỉ nhanh hơn Debug khoảng 10%, chi phí cố định ở `WalletMatcher` (tách token bí danh mỗi lần gọi) và `DateParser.parse` chạy 2 lần; đo lại trên thiết bị thật |
| iS0-13 | Đối chiếu với Android | 1 | Xong | 2026-11-21 | Đối chiếu 200 câu mẫu và các ca biên tại `docs/shared/parser/doi-chieu-ios-android.md`: iOS 200/200 câu đạt cả 6 trường; Android 192/200 câu (hoãn 8 câu TRANSFER tc_117-tc_124 sang S2-08 theo nợ kỹ thuật). Mọi chênh lệch đều có lý do được ghi nhận. Đã sửa lỗi mất dấu gạch chéo trong ghi chú. Đạt Cổng 1 (CI xanh, số tiền 100%, Preview sáng/tối đạt) |

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
| 2026-11-19 | iS0-06 | Chuỗi hard-code trong các component DesignSystem ("Âm ...", "trên ...", "ngân sách") | iS1, task String Catalog (iS1 chưa chia task) |
| 2026-11-20 | iS0-07 | Khung test 200 câu mới so số lượng giao dịch: cần so từng trường, tính tỉ lệ đúng, đưa câu đã đúng ra khỏi `withKnownIssue` để chặn hồi quy | iS0-09 (đã xử lý 2026-11-20 cho số tiền; các trường khác theo task của chúng) |

## Ngoại lệ (agent viết hộ)

| Ngày | Task | Phần được viết hộ | Lý do |
| --- | --- | --- | --- |
| 2026-11-18 | iS0-04 | Step `Report Coverage` trong `.github/workflows/ci.yml` | Người dùng xác nhận `ngoại lệ:` cho phần script bash/jq xuất bảng coverage |
| 2026-11-18 | iS0-05 | Setup `Colors.xcassets`, `Colors.swift` và `resources` trong `Core/Package.swift` | Người dùng xác nhận `ngoại lệ:` cho bước setup bảng màu |
| 2026-11-18 | iS0-05 | Viết `Typography.swift` và `LayoutTokens.swift` trong `Core/DesignSystem` | Người dùng xác nhận `ngoại lệ:` cho bước token kiểu chữ, khoảng cách và bo góc |
| 2026-11-19 | iS0-05 | Sửa colorset `BrandPrimary` (sáng `#1E7A4E`) và `BrandTintBackground` (sáng `#E6F5EC`); sửa `DesignSystemPreview` (chữ trên nền nhấn theo color scheme, thẻ `separator`, bỏ `cardPrimary`) | Người dùng xác nhận `ngoại lệ:` trong phiên mentor |
| 2026-11-20 | iS0-08 | Viết `Token.swift`, `TextNormalizer.swift`, `Tokenizer.swift` và test | Người dùng xác nhận `ngoại lệ:` sau khi xem khung mẫu |

## Nhật ký sprint

(Cuối mỗi sprint: kết quả so với "Xong khi", việc tốt, việc chậm, thay đổi cho sprint sau.)

### iL · 2026-11-02 → 2026-11-12 (khoảng 1,5 tuần, kế hoạch 2 tuần)
- **Kết quả so với "Xong khi" (Cổng 0):** đạt. `luyen-tap/SwiftCoBan` có 31 test, 10 suite, tất cả xanh. SachDoc đã chạy trên iPhone thật. Người dùng tự giải thích được 5 khái niệm. Với ARC, actor isolation và identity, mentor phải hỏi lại một lượt. 12/12 task xong, không có nợ kỹ thuật.
- **Việc tốt:** theo người dùng, mọi thứ đều ổn.
- **Việc chậm:** người dùng không thấy việc nào chậm.
- **Thay đổi cho sprint sau:** người dùng chưa muốn đổi gì.
- **Quan sát của mentor:** cần nhắc từ S1 và iS0.

### iS0 · 2026-11-14 → 2026-11-21 (khoảng 1 tuần, kế hoạch 2 tuần)
- **Kết quả so với "Xong khi" (Cổng 1):** đạt.
  - CI trên `main` xanh.
  - Parser đúng số tiền ở 200/200 câu mẫu, và đúng cả 6 trường ở 229/229 khoản.
  - Mọi chênh lệch với Android có lý do trong `docs/shared/parser/doi-chieu-ios-android.md`.
  - Design system đúng ở sáng và tối.
  - 13/13 task xong.
- **Nợ kỹ thuật:** chuỗi hard-code trong DesignSystem, dời sang iS1.
- **Việc tốt:** theo người dùng, mọi việc đều ổn.
- **Việc chậm:** người dùng không thấy việc nào chậm.
- **Thay đổi cho sprint sau:** người dùng không muốn đổi gì.
- **Ghi nhận của mentor (để người dùng tham khảo):**
  - Nhiều lần báo xong khi lint hoặc test chưa xanh: iS0-07 (swift-format), iS0-11 (test không biên dịch, lint đỏ hai lần), iS0-12 (lint).
  - Bản tự rà soát thường ghi "Đóng được" sớm.
  - Bảng đối chiếu iS0-13 lúc đầu có dữ liệu không có thật.
  - Đề xuất: trước khi báo xong, chạy hai lệnh lint chuẩn và `swift test`.
- **6 câu cần chốt quy ước chung** (`50000đ`, `1tr05`, ngày tương lai trong ghi chú, `5 lít xăng`, `chuyển 500k cho vợ`, `45k 2 ly`): chờ người dùng quyết định trước khi sửa README và bộ mẫu.
  - Để mentor đổi cột trạng thái trong tiến độ. Người dùng đã nhiều lần tự ghi Xong trước khi rà soát.
  - Build và chạy test trước khi báo xong. Ở iL-11 có lần báo xong khi code còn BUILD FAILED.
  - Khi tự viết tài liệu `hoc/`, đối chiếu với tài liệu chính thức. Tài liệu của iL-10 và iL-11 có vài chỗ sai về API.
  - Ở Cổng 0, trả lời bằng lời của mình, không dán.
  - Chủ đề cần ôn lại ở iS0: điều kiện để một `struct` là `Sendable`; optional chaining làm phẳng kiểu kết quả.

### iS0 · 2026-11-13 → 2026-11-21 (khoảng 1,2 tuần, kế hoạch 2 tuần)
- **Kết quả so với "Xong khi" (Cổng 1):** đạt. CI xanh trên cả nhánh và `main`; parser nhận đúng số tiền 200/200 câu mẫu (100% ≥ 95% theo NFR-02/Cổng 1); mọi khác biệt với bản Android (8 câu chuyển khoản hoãn sang S2-08) đều được ghi nhận và giải trình rõ ràng; design system hiển thị đúng trong Preview ở cả sáng và tối. Hoàn thành 13/13 task.
- **Việc tốt:** (chờ người dùng chia sẻ)
- **Việc chậm:** (chờ người dùng chia sẻ)
- **Thay đổi cho sprint sau:** (chờ người dùng chia sẻ)
- **Quan sát của mentor:**
  - Tốc độ port parser rất tốt, tận dụng hiệu quả thế mạnh Swift (pattern matching, Regex literal, Swift Testing).
  - Đã xử lý trọn vẹn 200/200 câu với đủ 6 trường dữ liệu (sớm hơn Android ở khoản nhận diện ví và chuyển khoản).
  - Cần duy trì thói quen chạy lint (`swift-format`, `swiftlint`) trước khi tạo PR để tránh đỏ CI do lỗi format.


