# Rà soát iS0-13 · Đối chiếu với Android

Ngày rà soát: 2026-11-21 · Nhánh `task/iS0-13-doi-chieu-android` · Trạng thái: **Xong** (đóng 2026-11-21 sau rà lại lần 1)

## Kiểm chứng đã làm

| Kiểm chứng | Kết quả |
| --- | --- |
| Đối chiếu bảng `tc_117`–`tc_124` với `test_cases.json` | ❌ **7/8 câu nhập không có trong bộ mẫu** (xem Bắt buộc 1) |
| Tìm trong test Android (`core/parser/src/test`) các câu ở bảng "trường hợp biên" | ⚠️ Chỉ thấy `trà sữa 45k 2 ly` (`AmountParserTest.kt:145`) và `15/12` sang năm mới (`DateTimeParserTest.kt:238`). Các dòng còn lại không có test nào |
| Chạy parser iOS với 11 câu trong bảng và câu cần chốt quy ước | ⚠️ 2 kết luận của cột iOS sai; tìm ra 1 lỗi ở phần agent viết (xem Nên sửa 3) |
| `GET …/actions/runs` | ✅ `main` sau khi merge PR #11 (`d1cf28e`) **xanh** |

## Bắt buộc sửa (2)

### 1. Bảng 8 câu chuyển khoản có câu nhập và kết quả không có thật
- **Vị trí:** `hoc/iS0-13-doi-chieu-android.md`, bảng "Danh sách các câu chênh lệch".
- **Vấn đề:** câu nhập thật trong `test_cases.json` là:

  | id | Câu nhập thật |
  | --- | --- |
  | `tc_117` | chuyển 2tr từ VCB sang ví tiền mặt |
  | `tc_118` | chuyển 500k từ techcombank sang momo |
  | `tc_119` | chuyển 1 củ từ momo sang vcb |
  | `tc_120` | rút tiền mặt 1tr từ atm vcb |
  | `tc_121` | nạp 200k vào momo từ vcb |
  | `tc_122` | chuyển 300k từ ví tiền mặt sang momo |
  | `tc_123` | chuyển khoản 1tr5 từ bidv sang mb |
  | `tc_124` | chuyển 50k từ tech sang momo |

  - Bảng chỉ đúng câu `tc_117`. Các câu như `ck 500k cho mẹ từ momo`, `bắn 200k tiền ăn`, `gửi 2 củ vào sổ tiết kiệm` không có trong bộ mẫu.
  - Cột kết quả iOS cũng không khớp với parser. Ví dụ bảng ghi "ví nguồn `tech`", trong khi khóa ví đúng là `techcombank`. Hay "note `cho mẹ`", trong khi cả 8 câu đều có ghi chú `nil`.
- **Vì sao:** cả task này là tìm chênh lệch **có bằng chứng**. Một bảng có dữ liệu bịa thì bên Android không dùng được, và che mất chênh lệch thật.
- **Hướng sửa:** lấy câu nhập từ `test_cases.json`, lấy kết quả iOS từ test hoặc từ việc chạy parser. Cả 8 câu iOS đều cho ra `TRANSFER`, có đủ ví nguồn và ví đích, ghi chú `nil`.

### 2. Cột Android ở bảng "trường hợp biên" không có bằng chứng; cột iOS có 2 chỗ sai
- **Vấn đề:**
  - 10 dòng đều ghi Android "✅ Khớp hoàn toàn". Nhưng test Android chỉ có `45k 2 ly` và `15/12`. Các dòng như `0912345678`, `ĐỔ XĂNG 50K`, `bánh tráng trộn`, `ca sáng`, nửa đêm, lan ngữ cảnh thì không có test nào chứng minh.
  - Cột iOS: `cho bé đi học 50k` thực tế cho ra **`family`** (khớp `cho bé` có dấu), còn `tao mua sách 80k` cho ra **`education`** (khớp `mua sách`). Bảng ghi cả hai là "không khớp từ điển".
  - Chưa có các câu mà README không quy định, như đã nêu ở bước 3.2.
- **Hướng sửa:**
  - Dòng nào Android chưa có test thì ghi **"chưa kiểm"**. Muốn so thật thì chạy câu đó trong phiên làm việc ở repo Android.
  - Sửa hai kết luận iOS.
  - Thêm các câu cần chốt quy ước, với kết quả iOS hiện tại (mentor đã chạy):

    | Câu | iOS hiện tại | Android |
    | --- | --- | --- |
    | `cafe 50000đ` | 50.000 | chưa kiểm |
    | `tiền nhà 1tr05` | 1.050.000 | chưa kiểm |
    | `15/10/2027 mua vé 200k` | 200.000; ngày tương lai bị bỏ | chưa kiểm |
    | `đổ 5 lít xăng` | 500.000 (độ tin cậy 0,4) | chưa kiểm |
    | `chuyển 500k cho vợ` | khoản chi, ghi chú `chuyển cho vợ` | chưa kiểm |
    | `trà sữa 45k 2 ly` | 45.000 | ✅ 45.000 (`AmountParserTest.kt:145`) |

  - Liệt kê những câu nào bạn muốn chốt quy ước chung và đưa vào README cùng bộ mẫu. Việc này sẽ làm ở phiên sau, khi bạn đã đồng ý.

## Nên sửa (3)

### 1. Bảng đối chiếu nên nằm ở chỗ dùng chung
- **Vì sao:** `hoc/` là tài liệu lý thuyết. Bảng đối chiếu là dữ liệu mà phiên làm việc ở repo Android cũng cần đọc, để sửa phía Android. Bước 3.1 đề xuất đặt ở `docs/shared/parser/doi-chieu-ios-android.md`.
- **Hướng sửa:** chuyển ba bảng sang file đó, và để file `hoc/` dẫn link tới.

### 2. Mục Cổng 1 tham chiếu sai task
- Design system được làm ở **iS0-05** (màu và chữ) và **iS0-06** (component), không phải iS0-04 và iS0-08.
- Component tên là **`CategoryChip`**, không phải `ChiTieuChip`.
- Mục "CI xanh" đúng: mentor đã kiểm qua API, `main` sau khi merge PR #11 xanh.

### 3. Lỗi ở phần agent viết: ghi chú bị méo khi ngày bị bỏ
- **Câu:** `15/10/2027 mua vé 200k` cho ra ghi chú **`15 10 2027 mua vé`**. Dấu `/` bị mất.
- **Nguyên nhân:** `ClauseAnalyzer.analyze` loại **mọi** dấu câu khỏi nội dung, nên các token số trở thành những đoạn rời rạc, rồi được nối lại bằng dấu cách. Khi cụm ngày được nhận diện thì không sao, vì cả cụm đã bị loại. Lỗi chỉ lộ ra khi cụm ngày bị bỏ (ngày tương lai, ngày không tồn tại).
- **Hướng sửa (phần agent):** giữ dấu câu nằm giữa một đoạn ghi chú, chỉ cắt dấu câu ở hai đầu đoạn. Khi tìm từ khóa danh mục thì chỉ xét token chữ và số. Thêm câu này vào test ngoài mẫu. Theo mục 2 của luật, agent sửa khi người dùng đồng ý.

## Ghi chú về iS0-12 (đã đóng)
Sau khi iS0-12 đóng, có thêm commit `17d0ed0` "dùng p95 làm ngưỡng chặn, bỏ hard assert maxDuration". Lý do là lần chạy CI trước đó (`03a9654`) đỏ ở bước `Run Core Tests`: trên runner dùng chung, có bật coverage, câu chậm nhất đã vượt 50 ms. Cách xử lý này hợp lý, vì phép đo chính thức ở iS4. Nhưng đây là một quyết định khác với lúc rà soát, nên cần ghi một dòng vào cột ghi chú của iS0-12 trong tiến độ: "CI chỉ chặn p95; max có thể vượt trên runner dùng chung".

## Điểm làm tốt
- Phân loại đúng: cả 8 câu chuyển khoản thuộc nhóm "chưa làm theo kế hoạch" (Android dời sang S2-08). Nhận ra Android chưa kiểm loại giao dịch và ví.
- Có so theo từng trường, không chỉ so tổng điểm.
- Có mục đánh giá Cổng 1.

## Kết luận
Chưa đóng được task. Cần sửa Bắt buộc 1–2 và Nên sửa 1–2. Nên sửa 3 là phần agent sửa, chờ bạn đồng ý. Sau đó sẽ kiểm tra Cổng 1 và đóng sprint iS0.

---

## Rà lại lần 1 (2026-11-21)

> Bản "Rà lại lần 1" tự viết đã được thay bằng bản mentor kiểm chứng dưới đây.

**Lệnh đã chạy:**
- swift-format `--strict` → ✅.
- SwiftLint `--strict` → ✅.
- `swift test` → ✅ 35 test và 1 XCTest `measure`.
- `xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) → ✅.
- Chạy parser với các câu trong bảng biên và bảng cần chốt quy ước → ✅ khớp với `doi-chieu-ios-android.md`.

**Kết quả:**
- **Bắt buộc 1:** ✅ Câu nhập `tc_117`–`tc_124` khớp với `test_cases.json`. Kết quả iOS đúng: `TRANSFER`, có ví nguồn và ví đích (khóa `techcombank` đúng), ghi chú `nil`.
- **Bắt buộc 2:** ✅ Android ghi "chưa kiểm" ở mọi dòng không có test, chỉ trừ `45k 2 ly` và `15/12`. Hai kết luận iOS đã sửa (`family`, `education`). Đã có bảng 6 câu cần chốt quy ước.
- **Nên sửa 1:** ✅ Bảng nằm ở `docs/shared/parser/doi-chieu-ios-android.md`.
- **Nên sửa 2:** ✅ Mục Cổng 1 tham chiếu đúng iS0-05, iS0-06 và `CategoryChip`.
- **Nên sửa 3:** ✅ Người dùng tự sửa `ClauseAnalyzer`: chỉ loại token xuống dòng và token phân cách tại ranh giới, cắt dấu câu ở hai đầu mỗi đoạn ghi chú, và chỉ dùng token chữ/số khi tìm từ khóa danh mục. Câu `15/10/2027 mua vé 200k` cho ra `15/10/2027 mua vé`, có test ngoài mẫu. Bộ mẫu vẫn đúng 200/200.
- **Ghi chú iS0-12:** ✅ Đã thêm "CI chỉ chặn p95" vào tiến độ.

**Gợi ý nhỏ:** trong bảng biên, hai dòng `mua sữa và bánh 50k` và `ĐỔ XĂNG 50K` ghi "đã kiểm qua `ClauseAnalyzerTests`", nhưng test thật dùng câu khác (`mua rau và thịt 80k`; câu `ĐỔ XĂNG 50K` nằm ở `TokenizerTests`, chưa kiểm ghi chú). Kết quả vẫn đúng (mentor đã chạy). Nên sửa lời ghi chú hoặc thêm hai câu này vào test.

## Kiểm tra Cổng 1 (`ke-hoach-ios.html`, iS0 "Xong khi")

| Điều kiện | Bằng chứng | Kết quả |
| --- | --- | --- |
| CI xanh | `GET …/actions/runs`: `main` ở `d1cf28e` (sau PR #11) success | ✅ |
| Parser đúng số tiền ≥ 95% câu mẫu | 200/200 (100%), test `testAccuracyOn200CasesThreshold` và test có tham số | ✅ |
| Mọi chỗ khác Android có lý do | `doi-chieu-ios-android.md`: 8 câu chuyển khoản là "chưa làm theo kế hoạch" (S2-08); loại và ví Android chưa kiểm (S2-08); 6 câu cần chốt quy ước đã liệt kê | ✅ |
| Design system đúng ở sáng và tối | iS0-05: người dùng xác nhận 3 Preview (Sáng, Tối, AX5); iS0-06: mỗi component có `#Preview` sáng và tối | ✅ |

**Kết luận:** Đóng được task. **Đạt Cổng 1.**
