# Đối chiếu bộ phân tích câu giữa iOS và Android

Tài liệu này là căn cứ dùng chung cho cả hai repo (`ChiTieu-ios` và `ChiTieu` Android) để ghi nhận mức độ tương thích, các câu chênh lệch có giải trình, và các trường hợp biên cần chốt quy ước chung.

---

## 1. Đối chiếu trên bộ 200 câu mẫu (`test_cases.json`)

Mốc chuẩn: `2026-10-20T10:00:00+07:00`, từ điển `category_keywords.json` phiên bản 1.

| Trường kiểm thử | iOS (iS0-11, iS0-13) | Android (S0-13) | Chênh lệch & Lý do |
| --- | :---: | :---: | --- |
| **Số câu đạt** | **200/200** (100%) | 192/200 (96%) | Android hoãn 8 câu `TRANSFER` (`tc_117`–`tc_124`) sang S2-08 (Nợ kỹ thuật) |
| **Số khoản (`count`)** | 229/229 (100%) | 221/221 (100% chi tiêu/thu nhập) | Khớp hoàn toàn ở mọi câu đơn và câu đa khoản |
| **Số tiền (`amount`)** | 229/229 (100%) | 221/221 (100%) | Khớp hoàn toàn; đạt Cổng 1 (≥ 95%) |
| **Thời gian (`datetime`)** | 229/229 (100%) | 221/221 (100%) | Khớp hoàn toàn cả mốc động, giờ mặc định và lan ngữ cảnh |
| **Danh mục (`categoryKey`)** | 229/229 (100%) | 221/221 (100%) | Khớp hoàn toàn theo `seedKey` của từ điển v1 |
| **Ghi chú (`note`)** | 229/229 (100%) | 221/221 (100%) | Khớp hoàn toàn (bỏ khoảng trắng thừa, không phân biệt hoa thường) |
| **Loại (`type`)** | 229/229 (100%) | Chưa kiểm tra (hoãn) | Android hoãn sang S2-08 (FR-05) |
| **Ví (`wallet` / nguồn / đích)** | 229/229 (100%) | Chưa kiểm tra (hoãn) | Android hoãn sang S2-08 (FR-06) |

---

## 2. Danh sách 8 câu chuyển khoản chênh lệch giữa hai bản

Cả 8 câu này nằm trong `test_cases.json` với loại giao dịch `TRANSFER`.

| ID | Câu nhập thật | Kết quả iOS | Kết quả Android | Phân loại & Hướng xử lý |
| --- | --- | --- | --- | --- |
| `tc_117` | chuyển 2tr từ VCB sang ví tiền mặt | 1 khoản: 2.000.000, `TRANSFER`, ví nguồn `vcb`, ví đích `cash`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |
| `tc_118` | chuyển 500k từ techcombank sang momo | 1 khoản: 500.000, `TRANSFER`, ví nguồn `techcombank`, ví đích `momo`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |
| `tc_119` | chuyển 1 củ từ momo sang vcb | 1 khoản: 1.000.000, `TRANSFER`, ví nguồn `momo`, ví đích `vcb`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |
| `tc_120` | rút tiền mặt 1tr từ atm vcb | 1 khoản: 1.000.000, `TRANSFER`, ví nguồn `vcb`, ví đích `cash`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |
| `tc_121` | nạp 200k vào momo từ vcb | 1 khoản: 200.000, `TRANSFER`, ví nguồn `vcb`, ví đích `momo`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |
| `tc_122` | chuyển 300k từ ví tiền mặt sang momo | 1 khoản: 300.000, `TRANSFER`, ví nguồn `cash`, ví đích `momo`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |
| `tc_123` | chuyển khoản 1tr5 từ bidv sang mb | 1 khoản: 1.500.000, `TRANSFER`, ví nguồn `bidv`, ví đích `mb`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |
| `tc_124` | chuyển 50k từ tech sang momo | 1 khoản: 50.000, `TRANSFER`, ví nguồn `techcombank`, ví đích `momo`, note `nil` | Bỏ qua trong `skippedTransferCases` | **Chưa làm theo kế hoạch:** Android hoãn sang S2-08 (FR-05, FR-06) |

*Kết luận:* 100% số câu chênh lệch (8/8) thuộc nhóm **Chưa làm theo kế hoạch** (Android chủ động hoãn sang S2-08 theo Nợ kỹ thuật đã ghi nhận ngày 2026-11-01). Không có câu nào thuộc nhóm **Lỗi** hay **Quy ước chưa rõ**.

---

## 3. Đối chiếu các trường hợp biên ngoài bộ mẫu (Edge cases)

| Nhóm | Trường hợp biên | Câu mẫu kiểm thử | Kết quả iOS | Kết quả Android | Đánh giá |
| --- | --- | --- | :---: | :---: | :---: |
| **Số tiền** | Số ghép, số tắt | `tai nghe 1tr2`, `85k5`, `củ rưỡi`, `hai trăm rưỡi` | ✅ 1.200.000, 85.500, 1.500.000, 250.000 | chưa kiểm | iOS đã kiểm qua `AmountParserTests` |
| **Số tiền** | Tiếng lóng | `mua điện thoại 4 củ`, `5 lít`, `3 loét`, `năm chục` | ✅ 4.000.000, 500.000, 300.000, 50.000 | chưa kiểm | iOS đã kiểm qua `AmountParserTests` |
| **Số tiền** | Chống nhận nhầm | `0912345678`, `mua 2 ly trà sữa 60k`, `nhà 123` | ✅ Chỉ lấy 60.000; bỏ qua SĐT, số lượng | chưa kiểm | iOS đã kiểm qua `AmountParserTests` |
| **Thời gian** | Nửa đêm & đầu tháng | Lúc `00:30 2026-11-01` nói: "hôm qua mua thuốc 60k" | ✅ Lùi về `2026-10-31 00:30` | chưa kiểm | iOS đã kiểm qua `DateParserTests` |
| **Thời gian** | Chuyển năm mới | Lúc `2027-01-02` nói: "15/12 mua quà 300k" | ✅ Lùi về `2026-12-15 10:00` | ✅ Ra 15/12/2026 (`DateTimeParserTest.kt:238`) | Khớp hoàn toàn giữa 2 bản |
| **Thời gian** | Lan ngữ cảnh | "hôm qua ăn phở 45k, tối xem phim 100k" | ✅ Khoản 2 lan ngày hôm qua, ghép buổi tối | chưa kiểm | iOS đã kiểm qua `DateParserTests` |
| **Danh mục** | Cụm từ dài nhất | `bánh tráng trộn 25k` | ✅ Khớp `food_dining` | chưa kiểm | iOS đã kiểm qua `ClauseAnalyzerTests` |
| **Danh mục** | Từ đơn ngắn có cụm | `cho bé đi học 50k` | ✅ Khớp `family` (khớp cụm `cho bé`) | chưa kiểm | iOS đã kiểm qua `ClauseAnalyzerTests` |
| **Danh mục** | Từ đơn ngắn có cụm | `tao mua sách 80k` | ✅ Khớp `education` (khớp cụm `mua sách`) | chưa kiểm | iOS đã kiểm qua `ClauseAnalyzerTests` |
| **Danh mục** | Từ đơn ngắn đứng 1 mình | `cho 100k`, `ca 60k`, `tao 30k` (không dấu) | ✅ Không khớp từ điển (fallback `other_expense`) | chưa kiểm | Tránh nhận nhầm động từ không dấu |
| **Tách khoản** | 'và' trong 1 khoản | `mua sữa và bánh 50k` | ✅ 1 khoản, note: `mua sữa và bánh` | chưa kiểm | iOS đã kiểm qua `ClauseAnalyzerTests` |
| **Ghi chú** | Giữ nguyên chữ hoa | `ĐỔ XĂNG 50K` | ✅ Note cắt từ raw: `ĐỔ XĂNG` | chưa kiểm | iOS đã kiểm qua `ClauseAnalyzerTests` |
| **Ghi chú** | Giữ dấu câu trong note | `15/10/2027 mua vé 200k` | ✅ Note: `15/10/2027 mua vé` (giữ dấu `/`) | chưa kiểm | Ngày tương lai bị bỏ, giữ nguyên cụm ngày trong note |

---

## 4. Các câu cần chốt quy ước chung

Các câu dưới đây phản ánh những trường hợp thực tế chưa được nêu rõ ràng trong `README.md`. Bảng này ghi nhận cách xử lý hiện tại của iOS và đối chiếu với Android để làm cơ sở cập nhật `README.md` và `test_cases.json` trong tương lai:

| Câu nhập | iOS hiện tại | Android | Hướng đề xuất quy ước |
| --- | --- | --- | --- |
| `cafe 50000đ` | 50.000 | chưa kiểm | Hỗ trợ ký hiệu `đ`/`vnd` viết liền ngay sau số không dấu cách |
| `tiền nhà 1tr05` | 1.050.000 | chưa kiểm | Nhận diện số ghép phần lẻ có số 0 đứng trước (`1tr05` = 1.050.000đ) |
| `15/10/2027 mua vé 200k` | 200.000; ngày tương lai bị bỏ, note `15/10/2027 mua vé` | chưa kiểm | Khi ngày bị hủy (tương lai/sai), giữ nguyên cả cụm và dấu phân cách `/` trong `note` |
| `đổ 5 lít xăng` | 500.000 (độ tin cậy 0,4) | chưa kiểm | Từ lóng `lít` (100k) đi sau động từ tiêu hao/hành vi thì vẫn nhận là tiền nhưng hạ độ tin cậy |
| `chuyển 500k cho vợ` | khoản chi tiêu thường, note `chuyển cho vợ` | chưa kiểm | Từ `chuyển` nếu không đi kèm 2 ví nguồn/đích thì coi là chi tiêu thông thường |
| `trà sữa 45k 2 ly` | 45.000 | ✅ 45.000 (`AmountParserTest.kt:145`) | Số lượng đi liền đơn vị phân loại (`2 ly`, `3 cái`) bị loại bỏ, không nhận nhầm là tiền |
