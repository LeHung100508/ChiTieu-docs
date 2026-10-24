# Bộ câu mẫu kiểm thử bộ phân tích ngôn ngữ tự nhiên (ChiTieu)

Bộ dữ liệu này là **Single Source of Truth** (nguồn chân lý duy nhất) dùng chung giữa hai nền tảng **Android** (Kotlin) và **iOS** (Swift) để phát triển và kiểm thử bộ phân tích câu tự nhiên theo phương pháp Test-Driven Development (TDD).

Thư mục này nằm trong repo tài liệu `ChiTieu-docs`, được nhúng vào repo Android và iOS dưới dạng git submodule tại `docs/shared/parser/`.

---

## 1. Cấu trúc Schema JSON (`test_cases.json`)

```json
{
  "version": 1,
  "referenceTime": "2026-10-20T10:00:00+07:00",
  "timezone": "Asia/Ho_Chi_Minh",
  "description": "Bộ 200 câu mẫu kiểm thử bộ phân tích ngôn ngữ tự nhiên ChiTieu (Single Source of Truth cho Android và iOS)",
  "totalCases": 200,
  "mockWallets": [
    {
      "key": "cash",
      "name": "Tiền mặt",
      "aliases": ["tiền mặt", "ví tiền mặt", "tien mat"]
    }
  ],
  "testCases": [
    {
      "id": "tc_001",
      "input": "phở bò 45k",
      "tags": ["amount_k", "food"],
      "expected": [
        {
          "type": "EXPENSE",
          "amount": 45000,
          "categoryKey": "food_dining",
          "datetime": "2026-10-20T10:00:00+07:00",
          "wallet": null,
          "sourceWallet": null,
          "targetWallet": null,
          "note": "phở bò"
        }
      ]
    }
  ]
}
```

### Chi tiết các trường:
- **`version`** (`Int`): Phiên bản của bộ câu mẫu (bắt đầu từ `1`).
- **`referenceTime`** (`String` ISO-8601): Mốc thời gian quy chiếu cố định: `2026-10-20T10:00:00+07:00` (Thứ Ba, ngày 20/10/2026). Tất cả các biểu thức thời gian tương đối đều được tính toán từ mốc này.
- **`timezone`** (`String`): Múi giờ chuẩn `Asia/Ho_Chi_Minh` (+07:00).
- **`totalCases`** (`Int`): Tổng số ca kiểm thử trong file (đúng 200).
- **`mockWallets`** (`Array<MockWallet>`): Danh sách ví giả định của người dùng để parser đối chiếu tên hoặc bí danh (FR-06).
- **`testCases`** (`Array<TestCase>`): Danh sách 200 ca kiểm thử. Mỗi ca gồm:
  - **`id`** (`String`): Mã duy nhất dạng `tc_001` đến `tc_200`.
  - **`input`** (`String`): Câu người dùng nhập vào.
  - **`tags`** (`Array<String>`): Các nhãn phân loại kỹ thuật hỗ trợ lọc và đo độ chính xác theo tính năng (ví dụ `amount_slang`, `relative_date`, `multi_comma`, `income`, `transfer`, `wallet_bank`, `edge_case`).
  - **`expected`** (`Array<ParsedItem>`): Mảng các giao dịch dự kiến. Một câu có thể cho ra 1 giao dịch, nhiều giao dịch (khi tách khoản theo FR-07), hoặc 0 giao dịch (khi là câu nhiễu / không có tiền).

### Cấu trúc `ParsedItem`:
| Trường | Kiểu | Mô tả |
| :--- | :--- | :--- |
| `type` | `String` | `"EXPENSE"` (Chi), `"INCOME"` (Thu), `"TRANSFER"` (Chuyển khoản). |
| `amount` | `Long` | Số tiền tính theo đơn vị Đồng (VND), số nguyên dương. |
| `categoryKey` | `String?` | Khóa danh mục chuẩn (`seedKey`). Luôn là `null` khi `type == "TRANSFER"` hoặc khi không xác định được danh mục. |
| `datetime` | `String` | Mốc thời gian dạng ISO-8601 (`YYYY-MM-DDTHH:mm:ss+07:00`). |
| `wallet` | `String?` | Khóa ví thanh toán (`walletKey`) hoặc `null` (nếu dùng ví mặc định). |
| `sourceWallet` | `String?` | Khóa ví nguồn (chỉ dùng khi `type == "TRANSFER"`). |
| `targetWallet` | `String?` | Khóa ví đích (chỉ dùng khi `type == "TRANSFER"`). |
| `note` | `String?` | Nội dung mô tả / tên món trích xuất từ câu, hoặc `null` nếu không có mô tả (chỉ có số tiền). |

---

## 2. Danh sách Ví giả định (`mockWallets`)

Theo FR-06, ví được nhận diện theo tên hoặc bí danh do người dùng đặt. Bộ câu mẫu cung cấp sẵn danh sách ví này để kiểm thử:

| `key` | Tên ví (`name`) | Các bí danh (`aliases`) |
| :--- | :--- | :--- |
| `cash` | Tiền mặt | `tiền mặt`, `ví tiền mặt`, `tien mat` |
| `momo` | MoMo | `momo`, `ví momo` |
| `vcb` | Vietcombank | `vcb`, `vietcombank`, `ngân hàng vcb`, `thẻ vcb` |
| `techcombank` | Techcombank | `tech`, `techcombank`, `tcb` |
| `mb` | MB Bank | `mb`, `mbbank`, `quân đội` |
| `bidv` | BIDV | `bidv` |
| `zalopay` | ZaloPay | `zalopay`, `zalo pay` |
| `shopeepay` | ShopeePay | `shopeepay`, `shopee pay` |
| `viettelpay` | Viettel Money | `viettelpay`, `viettel money` |
| `credit_card` | Thẻ tín dụng | `thẻ tín dụng`, `quẹt thẻ`, `credit`, `the tin dung` |

### Luật ưu tiên nhận diện ví:
1. **Ưu tiên bí danh dài hơn / cụ thể hơn:** Khi nhiều bí danh cùng xuất hiện hoặc lồng nhau, bí danh dài hơn sẽ thắng (ví dụ: `"thẻ vcb"` ưu tiên khớp ví `vcb`, thay vì nhận nhầm thành thẻ chung).
2. **Tránh nhiễu cho từ "thẻ":** Từ `"thẻ"` không đứng độc lập làm bí danh của `credit_card` nhằm tránh nhận nhầm trong các danh từ sản phẩm như `"thẻ điện thoại"`, `"thẻ game"`, `"thẻ cào"`. Bí danh của `credit_card` chỉ khớp khi có từ khóa rõ ràng như `"thẻ tín dụng"` hoặc đi kèm động từ thanh toán `"quẹt thẻ"`.
3. Trong `expected`, các trường `wallet`, `sourceWallet`, `targetWallet` **chỉ ghi `key`** của ví (ví dụ `"vcb"`, `"cash"`, `"techcombank"`), không ghi tên thô hay bí danh.

---

## 3. Quy ước `seedKey` Danh mục Mặc định

Không dùng tên hiển thị tiếng Việt cứng trong bộ test vì tên hiển thị sẽ thay đổi khi người dùng đổi ngôn ngữ ứng dụng (FR-35).

### A. Danh mục Chi tiêu (Expense)
- `food_dining`: Ăn uống, quán xá, cafe, trà sữa, nhậu nhẹt, ăn vặt.
- `groceries`: Đi chợ, siêu thị, thực phẩm tươi sống, đồ nấu nướng gia đình.
- `transportation`: Xăng xe, gửi xe, sửa xe, grab, taxi, vé xe buýt, máy bay, cầu đường.
- `shopping`: Quần áo, giày dép, mỹ phẩm, đồ gia dụng, mua hàng online (shopee, lazada).
- `bills`: Tiền điện, nước, internet/wifi, cước di động/4G, phí dịch vụ chung cư.
- `housing`: Tiền thuê nhà/phòng trọ, đồ nội thất, bảo trì nhà cửa.
- `entertainment`: Xem phim, vé hòa nhạc, game, netflix, spotify, du lịch.
- `health`: Thuốc men, viện phí, khám bệnh, nha khoa, phòng gym, yoga.
- `education`: Học phí, sách vở, khóa học trực tuyến, dụng cụ học tập.
- `family`: Chi cho con cái, bỉm sữa, đồ chơi, biếu ông bà/bố mẹ.
- `pets`: Thức ăn thú cưng, chăm sóc thú cưng, khám thú y.
- `personal`: Cắt tóc, spa, chăm sóc cá nhân.
- `other_expense`: Các khoản chi tiêu thông thường khác không khớp từ khóa rõ ràng.

### B. Danh mục Thu nhập (Income)
- `income_salary`: Tiền lương chính, lương cứng, phụ cấp.
- `income_bonus`: Tiền thưởng dự án, thưởng quý, thưởng tết, hoa hồng bán hàng.
- `income_gift`: Quà tặng bằng tiền, tiền lì xì, người thân cho.
- `income_investment`: Tiền lãi ngân hàng, cổ tức, lợi nhuận đầu tư.
- `other_income`: Các khoản thu nhập khác.

### C. Chuyển khoản (Transfer)
Khi `type == "TRANSFER"`, giao dịch đại diện cho việc điều chuyển số dư giữa 2 ví, không thuộc danh mục thu/chi nào $\rightarrow$ `categoryKey` luôn là `null`.

---

## 4. Quy tắc tính Thời gian trong ngày và Ngày tương đối

Dựa trên mốc quy chiếu: `referenceTime = 2026-10-20T10:00:00+07:00` (Thứ Ba, ngày 20/10/2026 lúc 10:00 AM).

### A. Trạng từ chỉ buổi / thời gian đứng ở đầu một khoản:
Khi từ chỉ buổi đứng độc lập ở đầu một khoản (hoặc đầu câu/mệnh đề), luôn tính là các mốc giờ trong **ngày hôm nay** (`2026-10-20`), không phụ thuộc vào giờ hiện tại của `Clock` để đảm bảo tính tổng quát:
- **"sáng" / "sáng nay":** `2026-10-20T08:00:00+07:00`.
- **"trưa" / "trưa nay":** `2026-10-20T12:00:00+07:00`.
- **"chiều" / "chiều nay":** `2026-10-20T15:00:00+07:00`.
- **"tối" / "tối nay":** `2026-10-20T20:00:00+07:00`.

Chỉ khi câu có từ chỉ quá khứ rõ ràng thì mới tính lùi ngày:
- **"tối qua":** `2026-10-19T20:00:00+07:00`.
- **"đêm qua":** `2026-10-19T22:00:00+07:00`.
- **"hôm qua":** `2026-10-19T10:00:00+07:00`.
- **"hôm kia":** `2026-10-18T10:00:00+07:00`.
- **"thứ 2 tuần trước" / "thứ hai tuần trước":** `2026-10-12T10:00:00+07:00`.
- **"ngày 5" / "mùng 5":** `2026-10-05T10:00:00+07:00`.
- **"15/10" / "ngày 15 tháng 10":** `2026-10-15T10:00:00+07:00`.
- **"hôm nay" / Không nhắc thời gian:** `2026-10-20T10:00:00+07:00`.

### B. Từ chỉ buổi trong tên món ăn / hành vi
Các cụm từ như `"ăn sáng"`, `"cơm trưa"`, `"ăn trưa"`, `"ăn tối"`, `"cf sáng"` là **nội dung chi tiêu (mô tả khoản chi)**, **không phải trạng từ chỉ thời gian**.
- Do đó, chúng **không tự gán lại giờ**, mà giữ nguyên giờ của ngữ cảnh đó (mặc định `10:00:00+07:00`).
- Ví dụ:
  - `"ăn sáng 30k"` $\rightarrow$ `datetime: 2026-10-20T10:00:00+07:00`, `note: "ăn sáng"`.
  - `"cơm trưa 40k"` $\rightarrow$ `datetime: 2026-10-20T10:00:00+07:00`, `note: "cơm trưa"`.
  - `"ăn tối 300k"` $\rightarrow$ `datetime: 2026-10-20T10:00:00+07:00`, `note: "ăn tối"`.
  - Trừ khi có trạng từ thời gian đi kèm: `"sáng nay ăn bún chả 40k"` $\rightarrow$ `datetime: 2026-10-20T08:00:00+07:00`, `note: "ăn bún chả"`.

---

## 5. Quy tắc trích xuất `note` (Ghi chú mô tả)

1. `note` là phần mô tả khoản chi còn lại sau khi đã tách bỏ số tiền, từ khóa thời gian và từ khóa ví.
2. Nếu câu chỉ có số tiền mà không có nội dung mô tả (ví dụ `"50k"`, `"100 nghìn"`, `"1tr5"`): `note = null` (không gán chuỗi số tiền làm note).
3. Trạng từ chỉ thời gian khi đã được tách làm mốc thời gian thì **không** giữ lại trong `note`.
   - Ví dụ: `"sáng phở 45k"` $\rightarrow$ `note: "phở"` (bỏ chữ "sáng").
   - `"hôm qua ăn ốc 150k"` $\rightarrow$ `note: "ăn ốc"` (bỏ chữ "hôm qua").
   - `"tối ăn lẩu 200k"` $\rightarrow$ `note: "ăn lẩu"` (bỏ chữ "tối").
4. Động từ hành động ("ăn", "mua", "uống", "đóng", "trả"): giữ lại nếu đi liền với danh từ mô tả (ví dụ `"mua áo"`, `"ăn phở"`, `"đổ xăng"`).
5. Chuyển khoản (`TRANSFER`): Nếu câu chỉ nêu hành vi chuyển tiền thuần túy giữa hai ví mà không nêu mục đích chi tiêu (ví dụ `"chuyển 2tr từ VCB sang ví tiền mặt"`, `"rút tiền mặt 1tr từ atm vcb"`), `note = null`. Chỉ khi câu có nêu mục đích hoặc người nhận (ví dụ `"chuyển 500k cho vợ"`), `note` mới nhận giá trị `"cho vợ"`.
6. Khi kiểm thử parser, trường `note` được so khớp chuẩn hóa (bỏ khoảng trắng thừa, không phân biệt hoa thường).
