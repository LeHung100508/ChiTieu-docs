# Rà soát iL-11 · Debug

Ngày rà soát: 2026-11-10 · Nhánh `task/iL-11-debug` · Trạng thái: **Xong** (đóng sau rà lại lần 1)

## Lệnh đã chạy
- `xcodebuild -project luyen-tap/SachDoc/SachDoc.xcodeproj -scheme SachDoc -destination 'platform=iOS Simulator,name=iPhone 17' build` → **BUILD FAILED**:
  `BookDetailView.swift:26:16: error: cannot find 'Tôi' in scope`
- Lỗi cài vào ở `BookStore.swift:23` đã được gỡ: dòng này đang là `books[index] = book`. Ghi chú "cần khôi phục" trong tiến độ đã cũ.

## Bắt buộc sửa (2)

### 1. Code không build được
- **Vị trí:** `BookDetailView.swift:26`, chữ `Tôi` gõ nhầm vào giữa chuỗi modifier.
- **Vì sao:** app không build thì không chạy được.
- **Hướng sửa:**
  - Xóa chữ đó.
  - Build lại trước khi báo xong task. Phím tắt Cmd+B, hoặc chạy lệnh ở mục "Lệnh đã chạy".

### 2. View Hierarchy Debugger chưa được dùng vào việc cụ thể
- **Vị trí:** `docs/hoc/iL-11-debug.md:58`.
- **Vì sao:** tiêu chí của task là *dùng* các công cụ cho một lỗi thật. Ghi chép hiện tại chỉ mô tả những gì nhìn thấy (`UIHostingController`, cell, nút). Nó chưa trả lời câu hỏi nào, và không liên quan gì tới lỗi.
- **Hướng sửa:** chọn một câu hỏi mà View Hierarchy trả lời được, rồi ghi lại số liệu đọc được. Hai gợi ý:
  - Vùng chạm của nút tròn trong `BookRowView` thực tế rộng bao nhiêu pt? Có đạt 44 pt theo checklist không?
  - Khung của badge chữ lớn bao nhiêu? Cách A và cách B trong preview thí nghiệm khác nhau thế nào?

  Ghi theo dạng: câu hỏi → chọn view nào trong cây → Size Inspector cho số gì → kết luận.

## Nên sửa (1)

### 1. Sửa vài chỗ chưa chính xác trong `docs/hoc/iL-11-debug.md`
- **Dòng 10, về `p`:** từ Xcode 15, `p` là alias của `dwim-print`. Lệnh này đọc biến trực tiếp giống `v` khi có thể, chỉ biên dịch thành biểu thức khi cần. Nó không in "cấu trúc bộ nhớ thô".
- **Dòng 9, về `po`:** `po` ưu tiên `debugDescription` (`CustomDebugStringConvertible`), sau đó mới tới `description`.
- **Dòng 27:** ví dụ `expr book.title = "Tên Mới"` sẽ không chạy trong `updateBook(_:)`, vì `book` là tham số `let`. Hãy tự thử để xem LLDB báo gì, rồi đổi ví dụ cho đúng.
- **Dòng 57:** ghi thêm kết quả của lệnh `expr books[index] = book`. Sau lệnh đó, khi bấm Continue thì dòng lỗi `books[0] = book` vẫn chạy. Vậy cuối cùng mảng ra sao? Câu trả lời cho thấy `expr` chỉ dùng để *thử giả thuyết*, không thay được việc sửa code.

## Gợi ý thêm để học (2)
1. Thử thêm một breakpoint có **Action** dạng `po book.title` và tick "Automatically continue". Đây là cách thay `print()` mà không cần build lại.
2. `BookRowView.swift:64` có thêm một dòng chỉ chứa khoảng trắng.

## Điểm làm tốt
- Nhật ký đi đủ chuỗi: triệu chứng (cảnh báo ID trùng của `ForEach`) → breakpoint có điều kiện → LLDB → nguyên nhân → cách sửa. Giải thích đúng vì sao gán đè `books[0]` làm hai phần tử trùng UUID.
- Lỗi cài vào sát với code thật của SachDoc, không phải ví dụ tách rời.
- Đã căn lại thụt lề của `BookRowView`, xử lý luôn góp ý còn sót từ iL-09.

## Kết luận
Chưa đóng được task. Sửa 2 mục Bắt buộc, cùng mục Nên sửa (hoặc `hoãn <mục>`), rồi báo lại để rà phần thay đổi.

---

## Rà lại lần 1 (2026-11-11)
`xcodebuild … build` → **BUILD SUCCEEDED**, không có cảnh báo từ code.
- Bắt buộc 1: đã xóa chữ thừa ở `BookDetailView.swift:26`.
- Bắt buộc 2: View Hierarchy đã trả lời được một câu hỏi cụ thể. Nút tròn đo được 26×26 pt, chưa đạt 44 pt; đã ghi kèm cách khắc phục cho các màn sản phẩm sau này.
- Nên sửa: đã sửa phần `p`/`dwim-print`, `po`/`debugDescription`, ví dụ `expr` với tham số `let`, và đã ghi kết quả sau khi Continue.

**Kết luận:** Đóng được task.
