# iS0-13 · Đối chiếu parser với Android

## Vì sao cần
Hai bản cùng dùng bộ câu mẫu `docs/shared/parser/`, nhưng mỗi bản tự viết parser riêng. Nếu không đối chiếu, cùng một câu có thể cho ra hai giao dịch khác nhau trên Android và iOS. Người dùng dùng cả hai bản (sao lưu, khôi phục chéo) sẽ thấy dữ liệu không nhất quán. Task này tìm mọi chỗ khác nhau và ghi lại lý do cho từng chỗ.

## Khái niệm chính
- **So theo từng trường, không chỉ "đúng/sai cả câu".** Một câu có thể đúng số tiền nhưng sai danh mục. Bảng đối chiếu cần các cột: số khoản, số tiền, thời gian, loại, danh mục, ví (thường, nguồn, đích), ghi chú.
- **Ba loại chênh lệch, mỗi loại xử lý một kiểu:**
  1. **Chưa làm theo kế hoạch:** một bản chưa tới task đó. Ví dụ Android hoãn chuyển khoản sang S2-08. Chỉ cần ghi lại task sẽ làm, không phải lỗi.
  2. **Lỗi:** một bản làm sai quy ước chung trong README. Sửa ở bản sai (sửa Android thì làm trong repo Android).
  3. **Quy ước chưa rõ:** README không nói gì, hai bản chọn khác nhau. Ví dụ `50000đ`, `45k 2 ly`. Người dùng quyết định, rồi bổ sung README và bộ mẫu để hai bản theo cùng một cách.
- **Bằng chứng phải lấy từ test, không phải đọc code.** iOS lấy danh sách lỗi theo trường từ test bộ mẫu. Android lấy từ báo cáo JUnit (`build/test-results/test/*.xml`) hoặc log của test bộ mẫu.
- **Câu ngoài bộ mẫu.** Bộ mẫu đúng 100% chưa chứng minh hai bản giống nhau (bài học từ iS0-09). Hãy chạy cùng một danh sách câu ngoài mẫu trên cả hai bản. Câu nào khác nhau thì xếp vào một trong ba loại trên.

## So với cách cũ bạn quen
| Cũ (Android) | Mới (iOS) | Khác biệt chính |
| --- | --- | --- |
| JUnit `@Test` + báo cáo XML trong `build/test-results` | Swift Testing; `swift test --xunit-output <file>` xuất báo cáo dạng xUnit | Cả hai đều xuất được báo cáo XML, nên có thể so bằng script |
| `println` trong test, xem bằng `--info` | `print` trong test, hoặc ghi vào `#expect(…, "comment")` | Comment của `#expect` hiện ngay trong báo cáo, gắn với đúng ca lỗi |

## API và cú pháp cần biết
- `swift test --filter <tên>`: chỉ chạy một nhóm test.
- `swift test --xunit-output report.xml`: xuất báo cáo XML.
- `#expect(điều_kiện, "id: trường")`: ghi rõ câu nào sai ở trường nào.
- Phía Android: `./gradlew :core:parser:test` (chạy trong repo Android); báo cáo nằm ở `core/parser/build/test-results/test/`.

```swift
// Ví dụ khác bài toán: gom lỗi theo nhóm rồi in một lần
var failuresByField: [String: [String]] = [:]
failuresByField["price", default: []].append("sp_017")
print(failuresByField.sorted { $0.key < $1.key })
```

## Lỗi hay gặp
1. **Chỉ so tổng điểm** (ví dụ "221/221" với "229/229"). Mỗi bên có thể bỏ qua những câu khác nhau, nên tổng điểm không cho biết câu nào lệch.
2. **Quên các câu bị bỏ qua.** Android đang bỏ 8 câu chuyển khoản (`skippedTransferCases`), và chưa kiểm loại giao dịch và ví. Những câu, những trường này phải ghi vào bảng là "chưa làm theo kế hoạch".
3. **Sửa bộ mẫu dùng chung một mình.** Sửa `test_cases.json` hay README là ảnh hưởng tới cả hai bản. Cần người dùng đồng ý, và cập nhật cả hai repo.
4. **Sửa Android trong phiên iOS.** Lỗi phía Android ghi vào bảng, rồi sửa trong phiên làm việc ở repo Android.

---

## Bảng đối chiếu dùng chung với Android

Toàn bộ các bảng đối chiếu chi tiết đã được đặt tại file dùng chung:
👉 **[doi-chieu-ios-android.md](file:///Users/lehung/Developer/ios-projects/ChiTieu-ios/docs/shared/parser/doi-chieu-ios-android.md)**

Tài liệu này bao gồm:
1. *Bảng đối chiếu 200 câu mẫu:* Tỷ lệ khớp 100% ở 6 trường dữ liệu trên iOS và giải trình 8 câu `TRANSFER` Android hoãn sang S2-08.
2. *Danh sách chi tiết 8 câu chuyển khoản:* Dữ liệu thật từ `test_cases.json` (`tc_117`–`tc_124`) và kết quả thực tế của parser.
3. *Bảng đối chiếu trường hợp biên ngoài bộ mẫu:* Ghi nhận trung thực các câu Android chưa có test ("chưa kiểm") và các kết luận chính xác của iOS.
4. *Danh sách 6 câu cần chốt quy ước chung:* Đề xuất phương án đồng bộ giữa hai nền tảng.

---

## Đánh giá tiêu chí Cổng 1 (Gate 1)

Đối chiếu với điều kiện nghiệm thu Cổng 1 trong [ke-hoach-ios.html](file:///Users/lehung/Developer/ios-projects/ChiTieu-ios/docs/ke-hoach-ios.html#L361):

1. **CI xanh:** ✅ Đạt. Nhánh `main` sau khi merge PR #11 (`d1cf28e`) đã xanh trên GitHub Actions.
2. **Parser nhận đúng số tiền ≥ 95%:** ✅ Đạt. Thực tế iOS đạt **200/200 câu (100%)** trên bộ mẫu dùng chung.
3. **Mọi chỗ khác với bản Android đều có lý do được ghi lại:** ✅ Đạt. Toàn bộ 8 câu chênh lệch `tc_117`–`tc_124` đều được giải trình rõ ràng tại [doi-chieu-ios-android.md](file:///Users/lehung/Developer/ios-projects/ChiTieu-ios/docs/shared/parser/doi-chieu-ios-android.md) (Android chủ động hoãn sang task S2-08 theo Nợ kỹ thuật).
4. **Design system hiện đúng trong Preview ở cả sáng và tối:** ✅ Đạt. Hoàn thành ở **iS0-05** (màu thương hiệu `#1E7A4E`, typography) và **iS0-06** (các component: `AmountText`, `CategoryChip`, `SummaryCard`).

---

## Đọc thêm
- Bảng đối chiếu dùng chung: [doi-chieu-ios-android.md](file:///Users/lehung/Developer/ios-projects/ChiTieu-ios/docs/shared/parser/doi-chieu-ios-android.md)
- Quy ước dùng chung: [README.md](file:///Users/lehung/Developer/ios-projects/ChiTieu-ios/docs/shared/parser/README.md)
- [Swift Package Manager: `swift test`](https://www.swift.org/documentation/package-manager/)
- Tiến độ Android: [tien-do.md](file:///Users/lehung/Developer/ios-projects/ChiTieu-ios/docs/tien-do.md), các dòng S0-11 tới S0-14 và Nợ kỹ thuật S0-13
