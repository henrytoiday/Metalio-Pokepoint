# CrossPoint cho Metalio E-Ink 4 — bản Việt hoá v19 by Huymelon

Firmware thay thế cho máy đọc sách **Metalio E-Ink 4** (ESP32-S3, màn e-ink 3,97"), dựng trên mã nguồn mở [CrossPoint Reader](https://github.com/crosspoint-reader/crosspoint-reader).

> **Phát triển bởi Huymelon.** Đây là dự án cá nhân, chia sẻ miễn phí, **không phục vụ mục đích thương mại**. Không bán lại firmware này hay dùng nó để kiếm lời.
>
> 🤝 **Rất mong anh em coder cùng góp sức!** Cứ fork, tạo branch mới để thử tính năng, sửa lỗi rồi gửi Pull Request. Chi tiết ở mục [Cùng xây dựng](#6-cùng-xây-dựng).

> ⚠️ Chỉ dành cho **Metalio E-Ink 4**. Không cài lên Xteink X3/X4 hay máy khác. Hãy sao lưu trước khi cài (xem mục 1).

## Có gì trong bản v19

- **Đọc to (TTS)** bằng Google Cloud Text-to-Speech, có giọng Chirp3-HD nam và nữ. Máy ngắt nghỉ tự nhiên ở dấu phẩy và dấu chấm, đọc liền mạch qua các trang. Wi-Fi tự kết nối ngầm, trang sách vẫn giữ nguyên. Chỉ nút Home (màu cam) mới dừng đọc.
- **Tai nghe Bluetooth**: tìm và kết nối ngay trên máy, ở bảng điều khiển nhanh, ô "Tai nghe".
- **Ghi chú bằng giọng nói**: nói vào micro, máy chuyển thành chữ và lưu vào `My Clippings.txt`.
- **Từ điển** Anh–Việt và Anh–Anh hiển thị phiên âm IPA đầy đủ.
- **Màn hình chờ sạch**, không còn bóng trang cũ. Màn này có bìa sách, tiến độ đọc, giờ hoàng đạo, việc nên và kiêng, kèm thông điệp mỗi ngày.
- Giao diện **"Máy quét cổ"**.

## Trong repo này

| Đường dẫn | Dùng để |
|---|---|
| `firmware/crosspoint-v19.bin` | Cập nhật từ thẻ nhớ ngay trên máy (máy đã chạy CrossPoint) |
| `firmware/crosspoint-v19-factory.bin` | Nạp qua cáp USB, dùng khi cài lần đầu hoặc cứu máy |
| `firmware/SHA256SUMS.txt` | Mã kiểm tra để chắc file tải về không bị lỗi |
| `sd-card/ai.json.example` | File mẫu để nhập khoá Google Cloud |
| `sd-card/oracle.txt` | Bộ thẻ "Thông điệp hôm nay" |
| `docs/metalio-sleep-ghosting.md` | Ghi chép kỹ thuật về cách làm sạch màn hình chờ |

---

## 1. Sao lưu trước khi cài

### 1a. Sao lưu thẻ nhớ (bắt buộc)

Thẻ nhớ chứa sách, tiến độ đọc, cài đặt, Wi-Fi đã lưu và khoá API của bạn.

1. Tháo thẻ ra cắm vào máy tính. Cách khác: trên máy chọn **Truyền tập tin → Ổ USB** rồi cắm cáp.
2. Chép **toàn bộ** thẻ vào một thư mục trên máy tính, kể cả thư mục ẩn `.crosspoint`. Trên macOS, bấm `Cmd + Shift + .` trong Finder để hiện thư mục ẩn.

### 1b. Sao lưu firmware hiện tại (rất nên làm, nhất là lần đầu)

Bản sao này giúp bạn quay về **firmware gốc của hãng** bất cứ lúc nào.

1. Cài Python 3, rồi cài esptool:
   ```bash
   pip3 install esptool
   ```
2. Bật máy và cắm cáp USB-C có truyền dữ liệu. Tìm cổng kết nối:
   - macOS: `ls /dev/cu.usbmodem*`
   - Windows: xem trong Device Manager, mục **Ports (COM & LPT)**, ví dụ `COM5`.
3. Đọc toàn bộ 16 MB bộ nhớ ra file:
   ```bash
   python3 -m esptool --chip esp32s3 --port /dev/cu.usbmodemXXXX read_flash 0 0x1000000 metalio-backup.bin
   ```
   Lệnh chạy mất vài phút. Kết quả là file `metalio-backup.bin` nặng đúng 16.777.216 byte. Hãy cất file này ở nơi an toàn.

Nếu esptool báo không kết nối được, hãy thử cáp hoặc cổng USB khác. Kiểm tra thêm là máy đang bật và **không** ở chế độ Ổ USB.

---

## 2. Cài bản mới (v19)

Trước khi cài, kiểm tra file tải về có nguyên vẹn không:

```bash
cd firmware && shasum -a 256 -c SHA256SUMS.txt
```

### Cách A: cập nhật từ thẻ nhớ (máy đã chạy CrossPoint)

1. Chép `crosspoint-v19.bin` vào **thư mục gốc** thẻ nhớ.
2. Trên máy, vào **Cài đặt → Hệ thống → Cập nhật firmware từ thẻ SD**.
3. Chọn `crosspoint-v19.bin`. Máy sẽ kiểm tra file, ghi và tự khởi động lại.
4. **Đừng tắt máy hay rút thẻ** trong lúc đang ghi.

Sách, cài đặt và khoá API nằm trên thẻ nên được giữ nguyên.

### Cách B: nạp qua cáp USB (lần đầu cài, hoặc khi máy không khởi động)

```bash
python3 -m esptool --chip esp32s3 --port /dev/cu.usbmodemXXXX write_flash 0x0 crosspoint-v19-factory.bin
```

Nạp xong, bấm nút nguồn hoặc rút cáp ra cắm lại để máy khởi động.

> Nếu chỉ muốn nạp phần ứng dụng, giữ nguyên bootloader và bảng phân vùng:
> `write_flash 0x10000 crosspoint-v19.bin`

### Quay về firmware gốc của hãng

```bash
python3 -m esptool --chip esp32s3 --port /dev/cu.usbmodemXXXX write_flash 0x0 metalio-backup.bin
```

---

## 3. Nhập khoá Google Cloud (cho đọc to và ghi chú giọng nói)

Khoá này **không** nằm trong firmware. Máy đọc khoá từ file `ai.json` trên thẻ nhớ.

### 3a. Tạo khoá

1. Vào <https://console.cloud.google.com>. Tạo một project mới, ví dụ `metalio-reader`.
2. Vào **Billing** và gắn một tài khoản thanh toán. Google bắt buộc bước này kể cả khi bạn chỉ dùng trong hạn mức miễn phí. Bạn có thể đặt **Budget alert** để được báo khi phát sinh phí.
3. Vào **APIs & Services → Library** và bật **(Enable)** hai API:
   - **Cloud Text-to-Speech API**: dùng cho đọc to.
   - **Cloud Speech-to-Text API**: dùng cho ghi chú giọng nói. Nếu ghi chú báo lỗi `403`, nghĩa là API này chưa được bật.
4. Vào **APIs & Services → Credentials → Create credentials → API key**.
5. Bấm vào khoá vừa tạo, mở **API restrictions**, chọn **Restrict key** và chỉ tích hai API trên. Bấm **Save**. Bước này giúp khoá có lỡ lộ ra thì cũng không bị dùng vào việc khác.
6. Sao chép khoá. Khoá có dạng `AIza...`.

Hạn mức miễn phí mỗi tháng, theo bảng giá Google tại thời điểm viết:

| Loại | Miễn phí mỗi tháng |
|---|---|
| Giọng Chirp3-HD | 1 triệu ký tự (khoảng vài cuốn sách) |
| Giọng WaveNet | 4 triệu ký tự |
| Speech-to-Text | 60 phút |

Hạn mức có thể thay đổi, hãy xem lại bảng giá trên trang Google Cloud.

### 3b. Tạo file `ai.json`

1. Mở `sd-card/ai.json.example` bằng một trình soạn văn bản thuần như TextEdit (chế độ Plain Text), Notepad hoặc VS Code.
2. Thay `DAN_KHOA_GOOGLE_CLOUD_VAO_DAY` bằng khoá của bạn. Giữ nguyên dấu ngoặc kép.
3. Lưu thành **`ai.json`** và đặt vào **một** trong hai chỗ trên thẻ nhớ:
   - `/.crosspoint/ai.json` (nên dùng, vì là thư mục ẩn), hoặc
   - `/ai.json` ở thư mục gốc.

```json
{
  "tts_provider": "google",
  "google_key": "AIza...khoá của bạn...",
  "gemini_key": "",
  "voice_vi": "",
  "voice_en": "",
  "speaking_rate": 1.0
}
```

| Trường | Ý nghĩa |
|---|---|
| `tts_provider` | Để `"google"`. Nếu Google Cloud lỗi và có `gemini_key`, máy chuyển sang Gemini. |
| `google_key` | Khoá Google Cloud (bắt buộc). |
| `gemini_key` | Khoá Gemini (không bắt buộc), chỉ dùng làm dự phòng. |
| `voice_vi`, `voice_en` | Để trống thì máy dùng giọng chọn trong Cài đặt. Muốn ép một giọng cụ thể, điền tên đầy đủ, ví dụ `"vi-VN-Chirp3-HD-Charon"` hoặc `"vi-VN-Wavenet-A"`. |
| `speaking_rate` | Tốc độ đọc, từ `0.25` đến `4.0`. Mặc định là `1.0`; muốn chậm hơn thì đặt `0.9`. |

Muốn **đổi giọng** ngay trên máy: vào **Cài đặt → Trình đọc → Giọng đọc** và chọn Kore, Aoede hoặc Leda (nữ), hay Charon, Puck, Orus hoặc Fenrir (nam).

> 🔒 **Không bao giờ** đưa `ai.json` lên GitHub hay gửi cho người khác. Repo này đã chặn file đó trong `.gitignore`. Nếu lỡ để lộ khoá, hãy vào Credentials, bấm **Regenerate key** hoặc xoá khoá rồi tạo khoá mới.

---

## 4. Nhập ngày giờ sinh cho "Thông điệp hôm nay"

Màn hình chờ kiểu **Bìa + Tiến độ + Thông điệp** luôn hiện giờ hoàng đạo trong ngày. Khi có ngày sinh, máy hiện thêm phần riêng cho bạn:

- Quan hệ ngũ hành giữa ngày hôm nay và mệnh của bạn.
- Tam hợp hoặc lục xung với tuổi của bạn.
- Dòng **Nên** và **Kiêng** trong ngày.
- Thẻ thông điệp riêng, khác với người sinh ngày khác.

### 4a. Bật màn hình chờ này

Vào **Cài đặt → Hiển thị → Màn hình ngủ**, chọn **Bìa + Tiến độ + Thông điệp**.

### 4b. Nhập ngày sinh qua trang web của máy (dễ nhất)

1. Trên máy, chọn **Truyền tập tin**, rồi kết nối Wi-Fi hoặc tạo điểm phát.
2. Trên điện thoại hay máy tính dùng chung mạng, mở địa chỉ hiện trên màn hình máy, ví dụ `http://192.168.1.23`.
3. Mở trang **Cài đặt (Settings)**, tìm nhóm **Hiển thị** và ô **Ngày giờ sinh (DD/MM/YYYY HH:MM)**.
4. Nhập theo **dương lịch**, theo một trong hai dạng:
   - `25/08/1995`: chỉ có ngày sinh.
   - `25/08/1995 14:30`: có cả giờ sinh, dùng giờ 24h. Giờ sinh giúp thẻ thông điệp sát hơn.
5. Lưu lại. Lần tới khi máy vào chế độ chờ, phần riêng sẽ xuất hiện.

### 4c. Cách khác: sửa thẳng trên thẻ nhớ

Tắt máy, cắm thẻ vào máy tính, mở `/.crosspoint/settings.json` và sửa giá trị `birthInfo`:

```json
"birthInfo": "25/08/1995 14:30",
```

Chỉ sửa giá trị trong ngoặc kép, đừng xoá dấu phẩy hay các dòng khác.

Nếu nhập sai định dạng, màn hình chờ sẽ hiện lời nhắc nhập ngày giờ sinh thay cho phần riêng.

### 4d. Tuỳ biến bộ thẻ thông điệp

Chép `sd-card/oracle.txt` vào **thư mục gốc** thẻ nhớ. Mỗi dòng trong file là một thẻ:

```
Nội dung thẻ | Nguồn (không bắt buộc)
上善若水 :: Thượng thiện nhược thủy — điều thiện cao nhất giống như nước. | Lão Tử
# Dòng bắt đầu bằng dấu # là ghi chú
```

Bạn thêm, bớt hay sửa tuỳ ý; máy chọn một thẻ mỗi ngày. Chữ Hán chỉ hiện khi phông đọc sách đang dùng có chữ Hán.

---

## 5. Gặp lỗi?

| Hiện tượng | Cách xử lý |
|---|---|
| Đọc to báo `429` | Hết lượt tạm thời. Máy tự thử lại; nếu vẫn lỗi, chờ vài phút. |
| Đọc to báo `403` hoặc không có tiếng | Kiểm tra khoá trong `ai.json` và xem đã bật **Cloud Text-to-Speech API** chưa. |
| Ghi chú giọng nói báo `403` | Bật **Cloud Speech-to-Text API** và thêm API này vào giới hạn của khoá. |
| Không có Wi-Fi | Lưu mạng trước trong **Truyền tập tin → Kết nối mạng**. Khi đọc to, máy tự dùng các mạng đã lưu. |
| Màn hình chờ còn bóng mờ | Xem `docs/metalio-sleep-ghosting.md`. Kiểm tra trên thẻ không có file `sleepdiag.txt`, hoặc file đó chỉ chứa `0`. |

## 6. Cùng xây dựng

Dự án mở cho mọi người. Ý tưởng nhỏ đến đâu cũng đáng giá, ví dụ:

- Giọng đọc offline (eSpeak-NG, kho âm tiết tiếng Việt)
- Dịch nhanh một đoạn văn bằng AI
- Thêm phông chữ hoặc giao diện
- Làm cho máy chạy nhanh và bền pin hơn

Cách tham gia:

1. **Fork** repo này về tài khoản của bạn.
2. Tạo **branch mới** cho từng việc, đặt tên rõ ràng: `git checkout -b feature/giong-doc-offline` hoặc `fix/loi-tai-nghe`.
3. Commit với lời nhắn ngắn gọn, nói rõ đã thay đổi gì.
4. Gửi **Pull Request** về branch `main`, mô tả bạn đã thử trên máy thật như thế nào.
5. Gặp lỗi hoặc có ý tưởng mà chưa tự làm được? Cứ mở một **Issue**.

Vài nguyên tắc chung:

- Không đưa khoá API, mật khẩu Wi-Fi hay dữ liệu cá nhân vào commit.
- Câu chữ tiếng Việt trên máy cần viết tự nhiên, đừng dịch từng chữ từ tiếng Anh.
- Giữ tinh thần phi thương mại của dự án.

## Tác giả và giấy phép

- **Phát triển: Huymelon.** Dự án cá nhân, phi thương mại, không liên quan đến hãng Metalio.
- Dựng trên mã nguồn mở [CrossPoint Reader](https://github.com/crosspoint-reader/crosspoint-reader) và FreeInk SDK. Mã nguồn gốc giữ nguyên giấy phép MIT của các dự án này (xem `LICENSE`). Cảm ơn các tác giả gốc.
- Khi chia sẻ lại, vui lòng ghi nguồn **Huymelon** cùng các dự án gốc, và không dùng vào mục đích thương mại.
