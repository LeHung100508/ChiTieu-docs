# Rà soát iS0-04 · CI

Ngày rà soát: 2026-11-18 · Nhánh `task/iS0-04-ci` · PR #3 · Trạng thái: **Xong** (đóng sau rà lại lần 2, 2026-11-18)

> Bản rà soát tự viết trước đó (ghi "Xong") đã được thay bằng bản này. Theo luật, mentor rà soát và đổi trạng thái.

## Lệnh đã chạy
Máy không có `gh`, nên mentor đọc trạng thái repo qua API công khai của GitHub.

| Kiểm tra | Kết quả |
| --- | --- |
| `GET …/actions/runs` | Run mới nhất trên PR #3 (`382a39d`) **success**. 6 run trước đó lỗi, trong lúc dò runner và tools-version |
| `GET …/actions/runs/44258085541/jobs` | Job `Build, Lint & Test` chạy trên runner GitHub-hosted (group "GitHub Actions", label `xcode-27`), không phải self-hosted. Mọi step đều success |
| `GET …/ChiTieu-ios/rules/branches/main` | **`[]`**: không có rule nào áp lên `main` |
| `GET …/ChiTieu-ios/branches/main` | `protected: false` |
| Đối chiếu repo Android: `GET …/ChiTieu/rules/branches/main` | Có ruleset `protect-name` ở trạng thái `active` với các rule `deletion`, `non_fast_forward`, `pull_request` và required check |

## Bắt buộc sửa (1)

### 1. `main` chưa được bảo vệ
- **Vì sao:**
  - Tiêu chí của task: "bảo vệ `main` giống bên Android".
  - API công khai cho thấy repo iOS không có ruleset nào đang áp lên `main`, trong khi cùng API đó trả đủ ruleset của repo Android. Như vậy ruleset bên iOS hoặc chưa được tạo, hoặc đang ở chế độ `Disabled`/`Evaluate`, hoặc target không khớp `main`.
  - Hệ quả: bản ghi tiến độ ghi "ruleset bảo vệ `main`" nhưng thực tế vẫn push thẳng lên `main` được, và CI đỏ cũng không chặn được merge.
- **Hướng sửa:**
  - Vào **Settings › Rules › Rulesets** và làm giống ruleset `protect-name` bên Android:
    - Enforcement status: **Active**.
    - Target: default branch.
    - Bật các rule: Restrict deletions, Block force pushes, Require a pull request before merging, Require status checks to pass (chọn check `Build, Lint & Test`).
  - Tự kiểm tra: mở `https://api.github.com/repos/LeHung100508/ChiTieu-ios/rules/branches/main`. Kết quả phải không còn rỗng.

## Nên sửa (3)

### 1. `.swift-format` bị rút gọn, đổi quyết định đã chốt ở iS0-03
- **Vị trí:** `.swift-format`. File giảm từ 81 dòng xuống 7 dòng, chỉ còn `version`, `indentation` và `lineLength`. Thay đổi này nằm trong commit `d5d303d` "tinh giản … tương thích runner CI".
- **Vì sao:**
  - Lý do rút gọn là runner cũ đọc không được file cấu hình mới. Lý do đó đã hết khi chuyển sang runner `xcode-27`, vì runner này dùng cùng toolchain với máy bạn.
  - Bây giờ swift-format chạy theo mặc định của nó. Các lựa chọn của iS0-03 bị bỏ mà không được ghi lại, ví dụ: `fileScopedDeclarationPrivacy`, các rule đặt tắt có chủ đích như `NeverForceUnwrap` hay `UseEarlyExits`, cách nhóm import.
- **Hướng sửa:** chọn một trong hai cách.
  - Khôi phục `.swift-format` của iS0-03, lấy bản trên `main`, rồi để CI chạy lại.
  - Hoặc giữ bản rút gọn, nhưng ghi lý do vào tiến độ như một thay đổi của iS0-03.

### 2. Không kiểm tra checksum của SwiftLint tải về
- **Vị trí:** `.github/workflows/ci.yml:31-33`.
- **Vì sao:**
  - CI tải một file binary từ Internet rồi chạy nó bằng `sudo mv` vào `/usr/local/bin`.
  - Repo public nên mọi PR đều chạy bước này. Nếu file release bị thay thế, hoặc mạng trả về nội dung khác, CI sẽ chạy một binary lạ.
  - `curl -sSL` không có `-f` thì lỗi HTTP vẫn ghi ra file.
- **Hướng sửa:**
  - Ghi cứng SHA-256 của `portable_swiftlint.zip` bản 0.65.1, lấy từ trang release hoặc tự tính một lần.
  - Kiểm tra bằng `shasum -a 256 -c` trước khi giải nén.
  - Thêm `-f` cho `curl`.

### 3. Chưa giới hạn quyền của `GITHUB_TOKEN`
- **Vị trí:** `.github/workflows/ci.yml`, cấp workflow.
- **Vì sao:** job chỉ cần đọc code. Nếu không khai báo `permissions`, token nhận quyền mặc định của repo. Nguyên tắc là cấp quyền tối thiểu, nhất là với repo public.
- **Hướng sửa:** khai báo `permissions:` ở cấp workflow, chỉ cho quyền đọc nội dung.

## Gợi ý thêm để học (3)
1. **`swift test` chạy trên macOS chứ không phải trên iOS.**
   - Hiện ổn, vì `Core` và `Features` có khai báo platform macOS 26.
   - Khi `DesignSystem` (iS0-05, iS0-06) hoặc các Feature dùng API chỉ có trên iOS, `swift test` trên macOS sẽ không build được. Lúc đó cần chuyển phần đó sang `xcodebuild test -scheme <Package>-Package -destination 'platform=iOS Simulator,…'`.
   - Nên chốt chuyện này ngay ở iS0-05.
2. **Merge từ nay phải đi qua PR.** Khi ruleset đã bật, lệnh `git merge` rồi `git push` lên `main` như các task trước sẽ bị từ chối. Đóng task bằng cách thêm commit `[iS0-04] …` trên nhánh task, rồi merge PR trên GitHub (Create a merge commit).
3. **File lý thuyết `iS0-04-ci.md`:**
   - Bảng so sánh ghi `actions/checkout@v4`, trong khi workflow dùng `v6`.
   - Bảng ghi "`xcode-select` chọn phiên bản Xcode", nhưng workflow không dùng `xcode-select`. Phiên bản Xcode do label runner quyết định.
   - "Không bị … data race" chỉ đúng với lỗi mà compiler bắt được ở Swift 6 language mode. Build không phát hiện data race lúc chạy.

## Điểm làm tốt
- Runner đúng đời Xcode 27, có bước in phiên bản toolchain ra log.
- Có `concurrency` với `cancel-in-progress`.
- Checkout kèm submodule `docs/`.
- Dùng lại đúng hai lệnh lint chuẩn của iS0-03, có `--strict`.
- Coverage được xuất ra `$GITHUB_STEP_SUMMARY`.
- Không có bí mật nào trong workflow. Trigger đúng: PR vào `main` và push lên `main`.
- Thử, lỗi, sửa đều có commit `[wip]` rõ ràng. Tools-version 6.2 được khôi phục sau khi đổi runner.

## Kết luận
Chưa đóng được task. Cần bật ruleset cho `main` (Bắt buộc 1) và xử lý ba mục Nên sửa (hoặc ghi `hoãn <mục>`). Sửa xong thì báo lại, mentor sẽ rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-18)
**Kiểm tra:**

| Kiểm tra | Kết quả |
| --- | --- |
| `GET …/rules/branches/main` | Đủ `deletion`, `non_fast_forward`, `pull_request` và `required_status_checks` (`Build, Lint & Test`) |
| `GET …/rulesets` | `protect-main`: **active** |
| Tự tải `portable_swiftlint.zip` 0.65.1 và tính SHA-256 | Khớp `c1e429b0…94d0` trong workflow |
| `swift format lint --strict` với `.swift-format` đã khôi phục | Local exit 0 |
| `swiftlint lint --strict` | Local exit 0 |
| Parse YAML workflow | Hợp lệ |

**Kết quả từng mục:**
- **Bắt buộc 1:** ✅ đạt.
- **Nên sửa 1:** ✅ đạt. `.swift-format` trùng bản trên `main` (81 dòng).
- **Nên sửa 2:** ✅ đạt. Có `curl -f` và `shasum -a 256 -c`.
- **Nên sửa 3:** ✅ đạt. Có `permissions: contents: read`.

**Còn thiếu:** các sửa đổi này chưa được commit và push. Run CI cuối cùng trên PR #3 vẫn là `382a39d`, tức là workflow cũ. Bước checksum, `permissions` và `.swift-format` đầy đủ chưa từng chạy trên runner.

**Kết luận:** chưa đóng được task. Chỉ chờ một điều kiện: push lên PR #3 và CI xanh.

## Rà lại lần 2 (2026-11-18)
`GET …/actions/runs`: run `44266137827` trên PR #3, commit `78ad4fb`, kết quả **success**.
- Mọi step đều success, kể cả `Install SwiftLint 0.65.1` (có kiểm tra SHA-256) và hai bước lint dùng `.swift-format` đầy đủ.
- Workflow mới (có `permissions`, checksum, cấu hình iS0-03) đã chạy xanh trên runner.

**Kết luận:** Đóng được task.
