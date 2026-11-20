# Rà soát iS0-10 · Parser: thời gian (FR-03)

Ngày rà soát: 2026-11-20 · Nhánh `task/iS0-10-thoi-gian` · Trạng thái: **Xong** (đóng 2026-11-20)

> Task giải thuật theo mục 2 của luật:
> - **Agent viết:** `DateParser`, `DateParser+Expressions`, `DateParser+Calendar`, `DateParserTests`, phần sửa `AmountParser.isDateNumber`, và file lý thuyết.
> - **Người dùng viết:** phần nối vào `TransactionParser.parse` và phần kiểm `datetime` trong test 200 câu.

## Lệnh đã chạy

| Lệnh | Kết quả |
| --- | --- |
| `swift format lint --recursive --strict ChiTieu Core Features` | ✅ exit 0 |
| `swiftlint lint --strict` | ✅ exit 0 |
| `cd Core && swift test` | ✅ 30 test qua, **không còn known issue nào** |
| `cd Core && xcodebuild -scheme Core-Package … test` (iPhone 18 Pro) | ✅ |
| App: `xcodebuild … build` | ✅ |

## Đối chiếu tiêu chí xong
- ✅ **Test thời gian trên bộ mẫu xanh:** 200/200 câu, cả trong `DateParserTests` lẫn trong test có tham số của `TransactionParser`.
- ✅ **Câu qua nửa đêm và đầu tháng:** đặt `now` là 2026-11-01 00:30 và thử `hôm qua`, `tối qua`, `hôm kia`, `ngày 5`, `ngày 31`, `thứ 2 tuần trước`. Thêm hai trường hợp: ngày 1/3 nói `ngày 30` (ngày không tồn tại), và ngày 2/1/2027 nói `15/12` (sang năm mới).
- ✅ **Inject `Calendar` và thời điểm hiện tại:** `DateParser` không gọi `Date()`.
- ✅ **14 câu ngoài bộ mẫu**, trong đó có các từ dễ nhầm như `sang ví`, `tôi`, `đếm`, `thu 2tr`.

## Phần của người dùng
- **Bước A:** ✅ `Parser.swift` tạo `Calendar` theo `timeZone` rồi gọi `DateParser.resolve`, gán `datetime` cho từng khoản.
- **Bước B:** ✅ Test có tham số kiểm chặt `datetime` cùng với số lượng và số tiền.

## Câu hỏi kiểm tra hiểu (mục 2 của luật)

| Câu | Đánh giá |
| --- | --- |
| 1. So có dấu và không dấu | ✅ Đạt. Có ví dụ đúng: `sang ví`, `tôi ăn phở` |
| 2. `hôm qua ăn sáng 35k, tối xem phim 100k` | ✅ Đạt: luật lan `.day`, rồi ghép buổi `tối` vào ngày được lan |
| 3. `ngày 31` lúc 00:30 ngày 1/11; `ngày 30` lúc 1/3 | ✅ Đạt: lùi về tháng trước. 30/2 không hợp lệ (`isValidDate`), và `calendar.date(from:)` sẽ dồn thành 2/3, một ngày ở tương lai |
| 4. Vì sao inject `now` và `Calendar` | ✅ Đạt |
| 5. `15/10/2027` làm sai số tiền | ✅ Đạt. Ghi chú nhỏ: hàm số trần nay tên là `matchRawNumber` |

## Gợi ý thêm (không chặn)
1. Test có tham số đã bỏ khối `withKnownIssue` của `type`, danh mục và ghi chú. Sang iS0-11, khi làm danh mục và tách khoản, nhớ thêm lại phần kiểm các trường này. Trường nào chưa làm thì để trong `withKnownIssue(isIntermittent: true)`.
2. `parse(referenceTime: Date = Date())` vẫn tự gọi `Date()` khi không truyền tham số. Chỗ này chấp nhận được, vì đây là cửa vào của app và đồng hồ chỉ được đọc ở đó. Sau này ViewModel nên truyền `now` vào, để test UI cũng cho kết quả cố định.
3. `dates[index]` dựa vào việc `resolve` trả về đúng số phần tử. Dùng `zip(amounts, dates)` thì không còn phụ thuộc vào giả định này.

## Kết luận
Đóng được task.
