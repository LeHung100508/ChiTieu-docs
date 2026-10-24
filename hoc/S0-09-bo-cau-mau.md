# S0-09 · Bộ câu mẫu (Golden Dataset)

## Vì sao cần
Bộ phân tích ngôn ngữ tự nhiên (Parser) là tính năng cốt lõi tạo nên sự khác biệt của ChiTieu ("ghi khoản chi trong 3 giây, hiểu tiếng Việt đời thường"). Để xây dựng một parser chuẩn xác và ổn định theo phương pháp TDD (Test-Driven Development), ta cần một **bộ dữ liệu kiểm thử vàng (Golden Dataset)** gồm 200 câu thực tế trước khi viết bất kỳ dòng logic parser nào.

Đặc biệt, dự án ChiTieu có cả bản Android (Kotlin) và iOS (Swift). Bộ câu mẫu này được đặt tại `docs/shared/parser/` trong repo tài liệu `ChiTieu-docs` (được gắn làm submodule vào cả 2 repo ứng dụng). Nhờ đó, cả hai nền tảng cùng chạy trên một nguồn dữ liệu duy nhất (Single Source of Truth), đảm bảo kết quả phân tích giống nhau 100%.

## Khái niệm chính

### 1. Cấu trúc Schema JSON
Để hai nền tảng Android và iOS đọc và kiểm thử độc lập mà không bị lệch kết quả do múi giờ hoặc thời gian chạy test:
- **`version`:** Phiên bản bộ dữ liệu (bắt đầu từ `1`).
- **`referenceTime`:** Mốc thời gian cố định dạng ISO-8601 với múi giờ Việt Nam (ví dụ: `2026-10-20T10:00:00+07:00`). Tất cả các từ chỉ thời gian tương đối ("hôm qua", "sáng nay", "thứ hai tuần trước") sẽ được tính toán dựa trên mốc này.
- **`timezone`:** `Asia/Ho_Chi_Minh` (+07:00).
- **`testCases`:** Mảng 200 ca kiểm thử. Mỗi ca gồm:
  - `id`: Mã định danh duy nhất (ví dụ `tc_001` đến `tc_200`).
  - `input`: Câu tiếng Việt tự nhiên người dùng nhập vào.
  - `expected`: Mảng các giao dịch dự kiến được tách ra (mỗi phần tử gồm: `type`, `amount`, `categoryKey`, `datetime`, `wallet`, `note`).
  - `tags`: Nhóm phân loại (ví dụ: `["amount_slang", "single_item"]`).

### 2. Quy ước `seedKey` cho danh mục mặc định
Không dùng tên hiển thị tiếng Việt (như "Ăn uống", "Di chuyển") trong bộ mẫu vì tên hiển thị phụ thuộc ngôn ngữ cài đặt của app (FR-35). Thay vào đó, dùng `seedKey` chuẩn:
- Chi tiêu: `food_dining`, `transportation`, `shopping`, `bills`, `entertainment`, `health`, `education`, `groceries`, `housing`, `other_expense`.
- Thu nhập: `income_salary`, `income_bonus`, `income_gift`, `income_investment`, `other_income`.
- Chuyển khoản: `transfer`.

### 3. Phủ đầy đủ các nhóm ca kiểm thử (FR-02 đến FR-07)
Bộ 200 câu cần phân bổ đa dạng các trường hợp:
1. **Cách nói số tiền của người Việt (FR-02):**
   - Viết tắt nghìn: `45k`, `45 nghìn`, `45 ngàn`, `45.000`, `45000`.
   - Viết tắt triệu: `1tr2`, `1,5tr`, `1.5tr`, `2 triệu`, `2tr`.
   - Tiếng lóng: `2 củ`, `5 lít`, `3 loét`, `năm chục`, `hai trăm`, `nửa củ`.
   - Số tiền lớn, tiền lẻ: `15.500.000`, `25k5`.
2. **Thời gian tương đối và tuyệt đối (FR-03):**
   - Tương đối: `hôm nay`, `hôm qua`, `hôm kia`, `sáng nay`, `tối qua`, `trưa nay`.
   - Ngày trong tuần: `thứ 2 tuần trước`, `chủ nhật rồi`.
   - Tuyệt đối: `ngày 5`, `ngày 15/10`, `hôm 20 tháng 10`.
   - Mặc định: Không nhắc ngày thì lấy `referenceTime`.
3. **Loại giao dịch (FR-05):**
   - Chi tiêu (`EXPENSE`): Mặc định khi không có từ khóa thu/chuyển.
   - Thu nhập (`INCOME`): `lương về 15tr`, `được thưởng 2 củ`, `mẹ cho 500k`.
   - Chuyển khoản (`TRANSFER`): `chuyển 2tr từ VCB sang ví tiền mặt`.
4. **Tách nhiều khoản trong một câu (FR-07):**
   - Dấu phẩy: `phở 45k, cafe 30k`.
   - Liên từ "và", "với": `ăn sáng 35k và đổ xăng 50k`.
   - Xuống dòng: `cơm trưa 40k\ntrà đá 5k`.
5. **Ví nguồn / Ví đích (FR-06):**
   - `bằng momo`, `qua vcb`, `tiền mặt`, `thẻ tín dụng`.
6. **Câu biên và nhiễu (Edge cases):**
   - Câu chỉ có số tiền: `50k`.
   - Câu không có số tiền: `đi ăn phở` (parser phải nhận diện lỗi hoặc không bóc tách được số tiền).
   - Câu nhiều số (số điện thoại, địa chỉ): `mua 2 cốc trà sữa 60k ở 123 Lê Lợi`.

## So với cách cũ bạn quen

| Cách cũ (Unit Test truyền thống) | Cách mới (Data-Driven Testing qua Shared Dataset) |
| --- | --- |
| Viết hàng chục hàm `@Test` trong Kotlin với các câu hard-code rải rác | Toàn bộ dữ liệu nằm trong 1 file JSON độc lập ngôn ngữ, có schema rõ ràng |
| Viết riêng cho Android, sang iOS phải viết lại toàn bộ test case bằng Swift | Dùng chung 1 file JSON qua git submodule, 2 nền tảng chạy cùng bộ test |
| Khó đo tỷ lệ phần trăm chính xác (Accuracy rate) | Dễ dàng chạy vòng lặp tính tỷ lệ: `(số ca pass / 200) * 100%` để kiểm tra tiêu chí Cổng 1 |

## Cấu trúc thư mục của task S0-09
```
docs/shared/parser/
├── README.md           # Mô tả cấu trúc schema JSON và quy ước
└── test_cases.json     # Bộ 200 câu mẫu chi tiết
```
Và trong `core:parser` của Android:
Cấu hình để đọc file `docs/shared/parser/test_cases.json` trong unit test resource.
