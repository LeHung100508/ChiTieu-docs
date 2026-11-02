# S0-15 · Tài khoản Google Play

## Vì sao cần
App chỉ lên được Google Play khi có tài khoản Play Console đã xác minh. Với **tài khoản cá nhân tạo mới**, Google còn bắt chạy closed testing trước khi cho phát hành Production. Cả hai việc đều tốn thời gian chờ mà không rút ngắn được. Vì vậy kế hoạch đặt việc này ở S0, chạy song song với phần code, và xếp nó vào nhóm rủi ro "kẹt ở S5".

## Khái niệm chính
- **Phí đăng ký:**
  - 25 USD, trả **một lần**.
  - Thẻ trả trước (prepaid) không được chấp nhận. Visa và Mastercard thông thường dùng được, nhưng loại thẻ chấp nhận còn tùy quốc gia.
  - Google ghi rõ: nếu thông tin xác minh không hợp lệ thì **không hoàn phí**.
- **Hai loại tài khoản:**
  - *Cá nhân:* dành cho cá nhân, sinh viên, người làm sở thích. Không cần D-U-N-S.
  - *Tổ chức:* dành cho doanh nghiệp, cần số D-U-N-S (đăng ký miễn phí qua Dun & Bradstreet).
  - Hai loại có cùng tính năng cốt lõi, đều bán được trên Play.
- **Xác minh danh tính:** làm ngay trong lúc đăng ký. Có thể cần giấy tờ tùy thân và thẻ thanh toán, **cả hai đều phải đứng tên thật của bạn**.
- **Riêng tài khoản cá nhân, có thêm hai bước:**
  1. Xác minh quyền truy cập một thiết bị Android, qua app **Play Console** trên điện thoại.
  2. Với tài khoản tạo sau ngày 13/11/2023: chạy **closed test với ít nhất 12 tester, mỗi người tham gia liên tục ít nhất 14 ngày**. Nếu một tester rời đi rồi tham gia lại, đếm lại từ đầu. Đủ điều kiện thì mới bấm "Apply for production". Google thường duyệt trong khoảng 7 ngày, có thể lâu hơn.
- **Khai báo tính năng tài chính (Financial features declaration):** mọi app đều phải khai, kể cả app không có tính năng tài chính nào.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| Công ty đã có sẵn tài khoản Play Console, bạn chỉ được thêm vào làm thành viên | Bạn tự là chủ tài khoản | Tự chịu trách nhiệm xác minh, chính sách, phản hồi review của Google |
| Tài khoản tổ chức: phát hành Production ngay | Tài khoản cá nhân mới: 12 tester × 14 ngày rồi mới xin Production | Phải tuyển tester từ S3-12, mời dư khoảng 15–20 người |

## Điểm cần cân nhắc trước khi trả phí
Theo Google, **app cung cấp sản phẩm hoặc dịch vụ tài chính bắt buộc dùng tài khoản tổ chức**. Danh sách Google nêu gồm ngân hàng, cho vay, giao dịch chứng khoán, quỹ đầu tư, ví và sàn tiền mã hóa. Danh sách tính năng trong form khai báo cũng **không có mục riêng** cho app ghi chi tiêu hay lập ngân sách.

ChiNhanh không kết nối ngân hàng, không chuyển tiền, không cho vay, không đầu tư. Vì vậy tài khoản cá nhân **nhiều khả năng** là hợp lệ, và đặc tả cũng đã giả định như vậy. Nhưng Google không nói rõ trường hợp này. Hãy đọc trang chính sách Financial Services trước khi trả phí. Nếu còn phân vân, hỏi bộ phận hỗ trợ Play Console.

## Lỗi hay gặp
1. **Tên trên giấy tờ, trên thẻ và trên hồ sơ không khớp nhau,** dẫn tới xác minh thất bại và không được hoàn phí.
2. **Dùng thẻ trả trước,** bị từ chối thanh toán.
3. **Dùng một tài khoản Google tạm** rồi sau này muốn đổi chủ. Nên dùng tài khoản Google chính, đã bật xác minh 2 bước.
4. **Để tới S5 mới đăng ký,** thì thời gian xác minh cộng với 14 ngày test sẽ đẩy lùi ngày phát hành.
5. **Tưởng tester chỉ cần cài app một lần là đủ.** Thực tế họ phải tham gia closed test **liên tục 14 ngày**; ai rời đi thì đếm lại từ đầu.

## Đọc thêm
- [Tạo và thiết lập tài khoản nhà phát triển](https://support.google.com/googleplay/android-developer/answer/6112435)
- [Chọn loại tài khoản: cá nhân hay tổ chức](https://support.google.com/googleplay/android-developer/answer/13634885)
- [Yêu cầu kiểm thử cho tài khoản cá nhân mới](https://support.google.com/googleplay/android-developer/answer/14151465)
- [Khai báo tính năng tài chính](https://support.google.com/googleplay/android-developer/answer/13849271)
