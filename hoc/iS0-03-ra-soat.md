# Rà soát iS0-03 · Chất lượng mã

Ngày rà soát: 2026-11-16 · Nhánh `task/iS0-03-chat-luong-ma` · Trạng thái: **Xong** (đóng sau rà lại lần 1, 2026-11-16)

> Bản rà soát tự viết trước đó (ghi "Đóng được task") đã được thay bằng bản này. Theo luật, mentor rà soát và đổi trạng thái.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ✅ exit 0, không có cảnh báo |
| `swiftlint lint` (0.65.1, cài bằng Homebrew) | ✅ 0 violations trong 26 file |
| `swift test` cho `Core` và `Features` | ✅ |
| `swift test` cho `luyen-tap/SwiftCoBan` | ✅ 31/31 |
| App: `xcodebuild … build` | ✅ BUILD SUCCEEDED |

Thử nghiệm thêm trong một thư mục nháp: đặt `swiftlint_version: 0.60.0` rồi chạy với SwiftLint 0.65.1.
- `swiftlint lint` chỉ in cảnh báo và trả exit 0.
- `swiftlint lint --strict` trả **exit 2**.

## Bắt buộc sửa (1)

### 1. Phiên bản SwiftLint chưa thật sự được cố định
- **Vị trí:** `.swiftlint.yml:1`. Phiên bản chỉ được ghi trong một dòng comment (`# Phiên bản SwiftLint cố định: 0.65.1`).
- **Vì sao:**
  - Tiêu chí yêu cầu "phiên bản SwiftLint cố định".
  - Một dòng comment không ngăn được ai cả. Máy khác chạy `brew install swiftlint` sẽ nhận bản mới nhất. Runner CI ở iS0-04 cũng vậy.
  - Bản mới có thể thêm hoặc đổi rule. Kết quả là máy này sạch nhưng máy kia đỏ.
- **Hướng sửa:** cần hai lớp.
  1. **Phát hiện lệch phiên bản.** Dùng key `swiftlint_version` trong `.swiftlint.yml`, chạy kèm `--strict`. Như thử nghiệm ở trên, lệch phiên bản khi đó sẽ thành lỗi.
  2. **Cài đúng phiên bản.** Ghi lại một cách cài ra đúng 0.65.1 mà máy nào và CI nào cũng làm được. Ví dụ: tải file release đúng phiên bản từ GitHub, dùng Mint với `Mintfile`, hoặc dùng SPM plugin có phiên bản `exact`. Chọn một cách và ghi lý do.
  - Homebrew không cố định được phiên bản một cách tin cậy, nên chỉ hợp để dùng tạm trên máy cá nhân.

## Nên sửa (2)

### 1. Phạm vi của hai công cụ không khớp, `luyen-tap/` bị format theo
- **Vị trí:**
  - `.swiftlint.yml:3-12` chỉ quét `ChiTieu`, `Core`, `Features` và loại `luyen-tap`.
  - Trong khi đó `swift format` chạy trên `.`. Hệ quả là 17 file trong `luyen-tap/` (code của sprint iL đã đóng) bị đổi format và nằm trong diff của task này.
- **Vì sao:**
  - Commit `[iS0-03]` lẫn thay đổi không liên quan, khó rà và khó revert.
  - Hai công cụ có hai phạm vi khác nhau thì sau này dễ có file chỉ được một công cụ kiểm tra.
  - Mình đã kiểm tra: diff chỉ là format (`-w` còn 24/16 dòng), test `SwiftCoBan` vẫn xanh. Vì vậy không có gì hỏng, chỉ là lẫn phạm vi.
- **Hướng sửa:**
  - Chốt một phạm vi chung. Đề xuất: `ChiTieu Core Features`, giống bên SwiftLint.
  - Trả `luyen-tap/` về như cũ bằng `git restore luyen-tap`.
  - Nếu bạn muốn giữ `luyen-tap/` trong phạm vi lint thì làm ngược lại: thêm nó vào `included` của SwiftLint. Khi đó ghi rõ lý do vào tiến độ, vì luật 13.6 không yêu cầu điều này cho iL.

### 2. Chưa ghi lại lệnh lint chuẩn của repo
- **Vì sao:**
  - `swiftlint lint` không có `--strict` trả exit 0 khi chỉ có cảnh báo. `swift format lint` không có `--strict` cũng vậy. CI ở iS0-04 sẽ xanh dù code còn cảnh báo.
  - Phạm vi đường dẫn (Nên sửa 1) cũng cần được ghi ở một chỗ.
- **Hướng sửa:** ghi hai lệnh lint chuẩn, có `--strict` và đúng phạm vi, vào một chỗ trong repo. Ví dụ: một mục trong README, hoặc comment đầu `.swiftlint.yml`. iS0-04 chỉ cần chép đúng hai lệnh đó.

## Gợi ý thêm để học (3)
1. **Ranh giới giữa hai công cụ.**
   - File lý thuyết (dòng 16) nói swift-format chỉ phụ trách layout. Thực ra `.swift-format` của bạn cũng bật các rule ngữ nghĩa, như `AlwaysUseLowerCamelCase`, `NeverUseForceTry` (đang tắt), `UseShorthandTypeNames`. Các rule này chồng lên SwiftLint (`identifier_name`, `force_try`, `syntactic_sugar`).
   - Chồng lên nhau thì không sao nếu hai bên cùng một ý. Chỉ cần nhớ: khi tắt một rule ở bên này, kiểm tra xem bên kia có đang bật rule tương ứng không.
   - Cặp `trailing_comma` (tắt bên SwiftLint) và `multiElementCollectionTrailingCommas` (bật bên swift-format) là cách chia đúng.
2. **Header "Created by…"** vẫn còn ở mọi file. Nếu muốn dọn, có thể bật rule `file_header` của SwiftLint. Không bắt buộc.
3. **File lý thuyết `iS0-03-chat-luong-ma.md`**, vài chỗ cần chỉnh:
   - Dòng 14: SwiftLint hiện chạy phần lớn rule trên **SwiftSyntax**. SourceKit chỉ cần cho một số rule và cho `swiftlint analyze`.
   - Dòng 31 và 54: lệnh nên có `--strict` và đúng phạm vi. Câu "ghi rõ phiên bản trong tài liệu" chưa phải là cố định phiên bản (xem Bắt buộc 1).

## Điểm làm tốt
- Chia việc giữa hai công cụ có chủ đích:
  - Tắt `trailing_comma`, `line_length`, `trailing_whitespace` bên SwiftLint, để swift-format quản lý với `lineLength: 120`.
  - Bật `force_unwrapping` bên SwiftLint, khớp với checklist 13.8.
- Loại trừ `.build` và `DerivedData`.
- Tự chạy formatter rồi chạy lại test. Cả hai package, `luyen-tap` và app đều vẫn xanh.

## Kết luận
Chưa đóng được task. Cần sửa Bắt buộc 1 và hai mục Nên sửa (hoặc ghi `hoãn <mục>`). Sửa xong thì báo lại, mentor sẽ rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-16)
**Lệnh đã chạy:**
- `swift format lint --recursive --strict ChiTieu Core Features` → exit 0.
- `swiftlint lint --strict` → exit 0, 0 violations trong 26 file.
- Link `portable_swiftlint.zip` của bản 0.65.1 trên GitHub Releases → HTTP 200.

Kết quả từng mục:
- **Bắt buộc 1: đạt.**
  - `.swiftlint.yml:11` có `swiftlint_version: 0.65.1`. Chạy kèm `--strict` thì lệch phiên bản sẽ thành lỗi.
  - Đã ghi lại cách cài đúng phiên bản bằng file release cố định (`.swiftlint.yml:6-8`).
- **Nên sửa 1: đạt.** `luyen-tap/` đã trả về như cũ. Phạm vi của swift-format và SwiftLint giờ trùng nhau: `ChiTieu`, `Core`, `Features`.
- **Nên sửa 2: đạt.** Hai lệnh lint chuẩn, có `--strict`, được ghi ở đầu `.swiftlint.yml`.
- **File lý thuyết:** đã sửa phần SwiftSyntax/SourceKit, thêm `--strict` và `swiftlint_version`.

**Gợi ý cho iS0-04, không chặn:**
- Khi tải binary trên CI, nên kiểm tra SHA-256 của file zip, so với giá trị lấy từ trang release. Như vậy CI không chạy nhầm một file bị thay thế.
- Lệnh `sudo mv … /usr/local/bin` chạy được trên runner macOS. Trên máy cá nhân, `swiftlint_version` sẽ báo ngay nếu bản Homebrew lệch phiên bản.

**Kết luận:** Đóng được task.
