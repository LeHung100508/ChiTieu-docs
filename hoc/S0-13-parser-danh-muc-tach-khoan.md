# S0-13 · Parser: danh mục và tách khoản (FR-04, FR-07)

## Vì sao cần
Đây là bước ghép các mảnh S0-10, S0-11, S0-12 thành pipeline hoàn chỉnh. Câu `sáng phở 45k, trưa cơm 35k và cafe 29k` phải ra **3 nháp** (FR-07). Mỗi nháp có danh mục đoán từ từ khóa (FR-04), note cắt từ câu gốc, và **độ tin cậy cho từng trường**. Độ tin cậy dùng để tô nổi chip chưa chắc (FR-08) và để quyết định có gọi Gemini hay không (FR-10).

## Khái niệm chính
- **Từ điển từ khóa là dữ liệu, không phải code.**
  - File JSON nằm ở `docs/shared/parser/`, cạnh bộ câu mẫu, khóa theo `seedKey`. Bản iOS đọc cùng file này.
  - `core:parser` đọc file **lúc chạy app**, tức là resource của source set `main`, không phải `test`.
  - Chỉ được đóng gói **đúng file từ điển**, không kèm `test_cases.json` hay README vào APK (NFR-04).
- **Tách khoản dựa trên token có sẵn.**
  - S0-10 đã có token dấu câu và xuống dòng. Ranh giới khoản là dấu `,`, `;`, xuống dòng và từ nối "và", "với".
  - Không phải chữ "và" nào cũng là ranh giới: `cơm gà và canh 50k` là **một** khoản. Một cách kiểm: chỉ cắt khi cả hai phía đều có số tiền.
- **Ngữ cảnh thời gian có thể lan sang khoản sau.**
  - Trong bộ mẫu: `hôm qua ăn sáng 35k, ăn trưa 45k` thì cả hai khoản đều là hôm qua (`tc_147`); `sáng nay cafe 25k, mua bánh 15k` thì cả hai đều 08:00 (`tc_148`).
  - Nhưng ở `sáng phở 45k, trưa cơm 35k, cafe 29k` (`tc_126`), khoản cuối lại không nhận "trưa".
  - Luật lan ngữ cảnh **chưa có trong README** → cần chốt ở đầu task.
- **So khớp danh mục theo cụm dài nhất.**
  - Quét các cụm token và ưu tiên cụm dài nhất: `trà sữa` thắng `trà`.
  - So trên bản bỏ dấu để người gõ không dấu vẫn khớp. Bản bỏ dấu dễ gom nhầm (`cá`/`cà`/`ca`, `bàn`/`bạn`), nên có thể so bản có dấu trước và cho độ tin cậy cao hơn.
- **Độ tin cậy có nhiều mức, không chỉ 0 hoặc 1.** Gợi ý cách chấm:
  - khớp có dấu > chỉ khớp không dấu > không khớp, rơi vào `other_expense`;
  - số tiền có đơn vị > số trần;
  - thời gian nói rõ > thời gian mặc định.

  Ngưỡng cụ thể sẽ chọn ở S6-07. Lúc này chỉ cần thứ tự hợp lý và nhất quán.
- **Note theo README mục 5.** Note là phần còn lại của khoản sau khi bỏ các token đã dùng (số tiền, thời gian, ví). **Giữ** động từ như "mua", "ăn" (`mua bánh mì`). Cắt từ `raw` theo vị trí token để giữ dấu và chữ hoa.

## So với cách cũ bạn quen
| Cũ | Mới | Khác biệt chính |
| --- | --- | --- |
| Map từ khóa viết cứng trong code hoặc trong `strings.xml` | JSON dùng chung đặt trong repo tài liệu, đọc qua classloader | Hai nền tảng cùng một nguồn; sửa từ điển không phải sửa code |
| `split(",")` trên chuỗi | Tách trên danh sách token có vị trí | Không làm mất vị trí so với câu gốc, nên cắt được note |
| Kết quả chỉ có giá trị | Mỗi trường có giá trị kèm độ tin cậy | UI biết chip nào cần tô nổi |

## API và cú pháp cần biết
- **`sourceSets.main.resources`:** dùng `srcDir(...)` để thêm thư mục và `include(...)` để lọc file. Lưu ý `include` áp cho **cả** source set resource đó.
- **Đọc resource:** `javaClass.classLoader.getResourceAsStream(...)` chạy được cả trên JVM lẫn Android, vì resource Java trong thư viện được gộp vào APK.
- **Giải mã JSON:** `Json.decodeFromString<T>()` hoặc `decodeFromStream`. Lúc này `kotlinx-serialization-json` phải là `implementation`, không còn chỉ là `testImplementation`.
- **Chỉ nạp một lần:** dùng `by lazy` để nạp từ điển lúc dùng lần đầu, đúng tinh thần NFR-02.

Ví dụ minh họa (bài toán khác: đóng gói một file `colors.json`):
```kotlin
sourceSets {
    main {
        resources {
            srcDir(rootProject.file("shared/theme"))
            include("colors.json")
        }
    }
}
private val palette: Map<String, String> by lazy { loadJson("colors.json") }
```

## Lỗi hay gặp
1. **Đóng gói cả thư mục `docs/shared/parser`** vào APK, kéo theo bộ câu mẫu và README.
2. **Cắt "và" một cách máy móc**, tách `cơm gà và canh 50k` thành hai khoản, trong đó một khoản không có số tiền.
3. **Bỏ mất ngữ cảnh thời gian** của khoản đầu khi xử lý các khoản sau, hoặc ngược lại: lan ngữ cảnh khi không nên.
4. **Note sai quy tắc:** bỏ động từ, lấy chữ số tiền làm note, hoặc lấy note từ `text` chữ thường thay vì từ `raw`.
5. **Câu không có số tiền vẫn tạo nháp.** Bộ mẫu ghi rõ `tc_199` và `tc_200` phải ra 0 nháp.

## Đọc thêm
- [Đặc tả · FR-04, FR-07, FR-08](../dac-ta-android.html)
- [README bộ câu mẫu · mục 3 (seedKey) và mục 5 (note)](../shared/parser/README.md)
- [Gradle · Source sets và resources](https://docs.gradle.org/current/userguide/java_plugin.html#source_sets)
- [kotlinx.serialization · JSON](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/json.md)
