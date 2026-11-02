# Prompt dùng với agent – Dự án ChiNhanh

Các prompt dưới đây dùng cùng `docs/luat-agent.md`. Copy nguyên khối vào agent.

## Ba repo

- `ChiTieu` (Android): https://github.com/LeHung100508/ChiTieu
- `ChiTieu-ios` (iOS, chỉ làm trên Mac): https://github.com/LeHung100508/ChiTieu-ios
- `ChiTieu-docs` (tài liệu): https://github.com/LeHung100508/ChiTieu-docs, gắn vào hai repo trên dưới dạng submodule tại `docs/` (mục 13 của luật)

Commit luôn theo hai bước: commit và push trong `docs/` trước, rồi mới commit con trỏ `docs` cùng code trong repo ứng dụng (mục 13.4).

## Repo Android (`ChiTieu`)

Repo đã được khởi động ở sprint S0: có sẵn `CLAUDE.md` (dòng `Nền tảng: Android`) và `AGENTS.md`. Trên máy mới:

1. Clone kèm submodule:

   ```bash
   git clone --recurse-submodules https://github.com/LeHung100508/ChiTieu.git
   ```

   Nếu đã clone mà `docs/` trống: `git submodule update --init`.
2. Cài JDK 17 trở lên (CI dùng 17), hoặc dùng JBR của Android Studio (trên Mac: `JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"`).
3. Mở agent ở gốc repo rồi dán prompt "Mỗi lần sau" bên dưới.

`main` có ruleset: phải qua PR, check "Build & Verify" xanh, dùng Squash and merge. Không push thẳng lên `main`.

## Mỗi lần sau (trên bất kỳ máy nào)

Claude Code tự nạp luật qua `CLAUDE.md`, nên chỉ cần:

```text
Tiếp tục dự án ChiNhanh. Hãy đồng bộ và rà tiến độ theo source code hiện tại (mục 4 của docs/luat-agent.md), đối chiếu với file tiến độ của nền tảng (docs/tien-do.md hoặc docs/tien-do-ios.md), rồi báo task hiện tại và bước tiếp theo. Chưa làm gì thêm cho đến khi tôi trả lời.
```

Nếu muốn agent biết bối cảnh chuyển máy, thêm một dòng:

```text
Lần trước tôi làm trên <tên máy khác>, đã commit [wip] và push. Hôm nay tôi làm trên <tên máy này>.
```

Với công cụ không tự đọc `CLAUDE.md` hay `AGENTS.md`, thêm câu này ở đầu prompt: `Đọc và tuân theo docs/luat-agent.md trước khi làm bất cứ việc gì.`

## Repo iOS (`ChiTieu-ios`)

Chỉ làm trên Mac. Luật dùng chung `docs/luat-agent.md`, phần riêng cho iOS ở mục 13.

### Chuẩn bị một lần (bạn tự làm, thuộc task iL-01)

1. Push hết thay đổi của repo tài liệu (`git -C docs status -sb` phải sạch, khớp `origin/main`).
2. Tạo repo **public** `ChiTieu-ios` trên GitHub, không tick README, .gitignore hay license.
3. Clone và gắn repo tài liệu làm submodule:

   ```bash
   git clone https://github.com/LeHung100508/ChiTieu-ios.git
   cd ChiTieu-ios && git submodule add https://github.com/LeHung100508/ChiTieu-docs.git docs
   ```

4. Trỏ dòng lệnh về Xcode (cần cho `xcodebuild`): `sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`, kiểm tra bằng `xcodebuild -version`.
5. Mở agent ở thư mục `ChiTieu-ios` rồi dán prompt "Lần đầu" bên dưới.

### Lần đầu: khởi động repo iOS

```text
Đây là repo iOS của dự án ChiNhanh (ChiTieu-ios). Repo mới tạo, đã gắn ChiTieu-docs làm submodule tại docs/, chưa có CLAUDE.md. Đọc và tuân theo docs/luat-agent.md trước khi làm bất cứ việc gì, chú ý mục 13: nền tảng iOS, tiến độ ở docs/tien-do-ios.md.

Hãy làm lần lượt:
1. Đồng bộ theo mục 4 và 13.4 (kiểm tra cả submodule docs/).
2. Xác định lượt hiện tại theo mục 13.3, đối chiếu docs/tien-do.md.
3. Bắt đầu Bước 1 của iL-01: được tạo CLAUDE.md, AGENTS.md theo mục 13.9 và docs/tien-do-ios.md (cùng cấu trúc với docs/tien-do.md, dòng "Lượt hiện tại" khớp nhau); phần còn lại (.gitignore, commit, push) tôi tự làm theo hướng dẫn.
```

### Mỗi lần sau

Sau iL-01, repo đã có `CLAUDE.md` (dòng `Nền tảng: iOS`), nên dùng đúng prompt "Mỗi lần sau" ở trên. Trên Mac mới, clone kèm submodule:

```bash
git clone --recurse-submodules https://github.com/LeHung100508/ChiTieu-ios.git
```

Nếu `docs/` ở detached HEAD, agent sẽ yêu cầu `git -C docs switch main` rồi `git -C docs pull` trước khi sửa tiến độ.

## Trong lúc làm

Dùng các câu lệnh ngắn ở mục 10 của luật: `bắt đầu`, `lý thuyết`, `bước tiếp`, `gợi ý`, `xong task`, `hoãn <mục>`, `tạm dừng`, `đồng bộ`, `ngoại lệ: <phần>`.

## Cuối phiên

```text
tạm dừng
```

Agent sẽ cập nhật `docs/tien-do.md` (bản iOS: `docs/tien-do-ios.md`), tóm tắt chỗ đang dừng và đưa lệnh commit `[wip]`. Bạn chạy lệnh đó rồi `git push` để máy khác làm tiếp được.
