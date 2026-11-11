# Mô tả thay đổi mã nguồn — repo `do_am_dat`

> Repo: https://github.com/duongtuananh2211/do_am_dat (commit `a4c7f99`)
> File chính: `src/main.cpp` (firmware ESP32), `dashboard/src/app.service.ts`, `dashboard/public/index.html`
> Mỗi thay đổi ghi: **vấn đề hiện tại → cần sửa gì → mục báo cáo bị ảnh hưởng**.

## Tóm tắt

| # | Thay đổi | Mức độ | File |
|---:|---|---|---|
| 1 | Tách API key / mật khẩu WiFi khỏi code | 🔴 Bắt buộc | `main.cpp`, `.gitignore` |
| 2 | Cập nhật hằng hiệu chuẩn ADC theo cảm biến thật | 🔴 Bắt buộc | `main.cpp` |
| 3 | Thêm log đo thời gian | 🔴 Bắt buộc (để có số liệu báo cáo) | `main.cpp` |
| 4 | Kết nối WiFi không chặn — mất mạng vẫn tưới tự động | 🟠 Nên làm | `main.cpp` |
| 5 | Giới hạn thời gian chế độ MANUAL_ON | 🟠 Nên làm | `main.cpp` |
| 6 | Tách kênh lệnh khỏi field3 (tránh mất lệnh) | 🟠 Nên làm | `main.cpp`, `app.service.ts` |
| 7 | Phát hiện cảm biến hỏng / bơm chạy mà đất không ẩm lên | 🟡 Tùy chọn | `main.cpp` |
| 8 | Watchdog tự reset khi treo | 🟡 Tùy chọn | `main.cpp` |
| 9 | Dashboard: xử lý `null`, thêm biểu đồ lịch sử | 🟡 Tùy chọn | `app.service.ts`, `index.html` |
| 10 | README | 🔴 Bắt buộc | `README.md` |

> 📝 Làm xong mục nào thì cập nhật mục báo cáo tương ứng. Thay đổi 4–9 nếu **không** làm thì giữ nguyên trong bảng Hạn chế (3.5) của báo cáo.
> 📝 Làm hết thay đổi trước rồi mới **đóng băng code** và đo đạc — sửa code sau khi đo thì phải đo lại.

---

## 1. Tách API key và mật khẩu WiFi 🔴

**Hiện tại.** `src/main.cpp:5-10` ghi thẳng SSID, mật khẩu WiFi, Write/Read API key; repo public nên ai cũng đọc được và ghi được vào channel.

**Sửa.**
1. Trên ThingSpeak: *API Keys → Generate New Write API Key* và *Generate New Read API Key*. Đổi mật khẩu WiFi nếu dùng mạng thật.
2. Tạo `include/secrets.h` (không commit):
   ```cpp
   #pragma once
   #define WIFI_SSID_VALUE      "..."
   #define WIFI_PASSWORD_VALUE  "..."
   #define TS_CHANNEL_ID        0
   #define TS_WRITE_API_KEY     "..."
   #define TS_READ_API_KEY      "..."
   ```
3. Tạo `include/secrets.h.example` (commit) cùng nội dung nhưng giá trị rỗng.
4. Thêm `include/secrets.h` vào `.gitignore`.
5. Trong `main.cpp`: `#include "secrets.h"` và thay các hằng bằng macro trên.
6. Cập nhật `dashboard/.env` với key mới.

> ⚠️ Key cũ vẫn nằm trong lịch sử git — **bắt buộc đổi key** ở bước 1, xóa khỏi code thôi là chưa đủ.

**Báo cáo:** 3.5 — bỏ dòng hạn chế "API key ghi thẳng trong code" (hoặc chuyển sang "đã khắc phục").

## 2. Hiệu chuẩn ADC 🔴

**Hiện tại.** `ADC_DRY = 3300`, `ADC_WET = 1500` (`main.cpp:15-16`) chưa rõ đo từ cảm biến nào.

**Sửa.** Đo ADC thô (log `[Sensor] ADC raw` đã có sẵn) với cảm biến đang dùng (điện trở + LM393, cấp 3.3 V):
- Đầu dò để ngoài không khí → giá trị `ADC_DRY`
- Đầu dò nhúng nước → giá trị `ADC_WET`

Lấy trung bình vài lần đo rồi điền vào hai hằng số. Kiểm tra `ADC_DRY > ADC_WET` (đất ẩm → AO thấp); nếu ngược thì đổi chỗ.

**Báo cáo:** 2.2 (nguyên lý cảm biến), 2.5a — ghi số đo thật.

## 3. Log đo thời gian 🔴

**Mục đích.** Lấy số liệu cho 1.4 và 3.1 (chu kỳ loop, thời gian request, độ trễ).

**Sửa.** Thêm cờ bật/tắt để bản nộp không bị rối log:
```cpp
#define DEBUG_TIMING 1
```
- Đầu `loop()`: in `millis()` và khoảng cách so với vòng trước → **chu kỳ loop**.
- Bao quanh `ThingSpeak.writeFields(...)` và `ThingSpeak.readLongField(...)` (hoặc lệnh đọc mới ở mục 6): in thời gian chạy → **thời gian 1 request**.
- Trong `readPumpCommand()` khi đổi chế độ: in `millis()` → so với thời điểm bấm nút trên dashboard → **độ trễ lệnh từ xa (NFR2)**.
- Trong `controlPump()` khi bật/tắt bơm: in `millis()` → **độ trễ cục bộ (NFR1)**.
- Cuối `setup()` và sau lần `writeFields` đầu tiên thành công: in `millis()` → **thời gian khởi động (NFR5)**.

**Báo cáo:** 1.4, 3.1.

## 4. Kết nối WiFi không chặn 🟠

**Hiện tại.** `connectWiFi()` (`main.cpp:36-52`) là vòng `while` chỉ thoát khi có mạng. Nó được gọi trong `setup()` và trong `sendToThingSpeak()` (`main.cpp:130`). Mất WiFi → `loop()` dừng hẳn → **bơm đang bật sẽ không bao giờ tắt**.

**Sửa.**
- `setup()`: gọi `WiFi.begin()` rồi đi tiếp, không chờ (hoặc chờ tối đa ~10 s rồi bỏ qua).
- Viết `maintainWiFi()` gọi mỗi vòng `loop()`: nếu `WiFi.status() != WL_CONNECTED` và đã quá `WIFI_RETRY_MS` (VD 10 s) từ lần thử trước → gọi `WiFi.reconnect()`, ghi lại `millis()`, **không chờ**.
- `sendToThingSpeak()` và `readPumpCommand()`: mất WiFi thì `return` ngay (bỏ lời gọi `connectWiFi()`).
- Khi mất mạng mà đang ở MANUAL_ON/MANUAL_OFF: sau một khoảng (VD 60 s offline) chuyển về **AUTO** để hệ thống vẫn tự tưới an toàn.

Kết quả: `readMoisturePercent()` + `controlPump()` chạy mỗi giây bất kể có mạng hay không.

**Báo cáo:** 2.4 (sơ đồ luồng thêm khối `maintainWiFi`), 2.5, 3.3 (dòng "Mất WiFi" → "Ổn"), 3.5 (bỏ hạn chế tương ứng), FR7.

## 5. Giới hạn thời gian MANUAL_ON 🟠

**Hiện tại.** `controlPump()` (`main.cpp:99`) ở MANUAL_ON bật bơm vô hạn. Bơm 1.8 L/phút → quên tắt là ngập.

**Sửa.**
```cpp
const unsigned long MANUAL_ON_MAX_MS = 5UL * 60 * 1000;  // 5 phút ≈ 9 L
unsigned long manualOnSince = 0;
```
- Khi chế độ chuyển sang MANUAL_ON: `manualOnSince = millis()`.
- Trong `controlPump()`: nếu `pumpMode == MANUAL_ON` và `millis() - manualOnSince >= MANUAL_ON_MAX_MS` → đặt `pumpMode = AUTO`, log `[Pump] Hết 5 phút thủ công → về AUTO`.
- Gửi chế độ mới lên ThingSpeak ở lần gửi kế tiếp để dashboard hiển thị đúng.

**Báo cáo:** 2.4 (máy trạng thái thêm cạnh `ON → AUTO: hết 5 phút`), 3.3, 3.5, FR3.

## 6. Tách kênh lệnh khỏi field3 🟠

**Hiện tại.**
- ESP32 ghi field3 = chế độ hiện tại mỗi 20 s (`main.cpp:135`); dashboard cũng ghi field3 = lệnh mới (`app.service.ts:81`).
- Người dùng bấm nút → trước khi ESP32 kịp đọc (chu kỳ 15 s), ESP32 gửi dữ liệu định kỳ với field3 = chế độ **cũ** → lệnh bị ghi đè, mất.
- Dashboard ghi lúc ESP32 vừa ghi → ThingSpeak từ chối (giới hạn 15 s/lần ghi mỗi channel).
- Entry do dashboard tạo chỉ có field3 → `feeds/last.json` có lúc trả field1/field2 rỗng → dashboard hiện độ ẩm `null`.

**Sửa (khuyến nghị: ThingSpeak TalkBack).** TalkBack là hàng đợi lệnh riêng, không dùng chung giới hạn ghi của channel.
1. Trên ThingSpeak: *Apps → TalkBack → New TalkBack*, ghi lại `TalkBack ID` và `TalkBack API Key` (key riêng, khác key channel). Thêm vào `secrets.h` và `.env`.
2. **Dashboard** (`setPumpMode`): thay vì ghi field3, thêm lệnh vào hàng đợi
   `POST https://api.thingspeak.com/talkbacks/<id>/commands.json` với `api_key`, `command_string` = `AUTO` / `ON` / `OFF`.
3. **ESP32** (`readPumpCommand`): gọi
   `POST https://api.thingspeak.com/talkbacks/<id>/commands/execute` với `api_key` (dùng `HTTPClient`).
   Trả về chuỗi lệnh kế tiếp và **xóa nó khỏi hàng đợi**; hàng đợi rỗng → body rỗng → giữ chế độ cũ.
4. ESP32 vẫn ghi field3 = chế độ **hiện tại** (chỉ để báo trạng thái); dashboard **không** ghi vào channel nữa → hết lỗi `null` và hết đụng giới hạn 15 s.
5. Có thể giảm `READ_INTERVAL_MS` xuống ~5 s vì TalkBack không dùng chung giới hạn ghi của channel → độ trễ lệnh từ xa giảm.

> 📝 Kiểm tra lại giới hạn tần suất gọi TalkBack của tài khoản miễn phí trước khi giảm chu kỳ đọc.

**Phương án nhẹ hơn (nếu không làm TalkBack).** Dashboard ghi lệnh vào **field4**, ESP32 đọc field4, field3 chỉ do ESP32 ghi. Khắc phục được việc ghi đè lệnh, nhưng vẫn còn giới hạn 15 s và entry thiếu field1/field2 → phải làm thêm mục 9.

**Báo cáo:** 2.1, 2.4 (kiến trúc: thêm khối TalkBack), 2.5c, 2.5e, 3.1 (NFR2), 3.3, 3.5.

## 7. Phát hiện cảm biến hỏng / bơm chạy vô ích 🟡

**Hiện tại.** Rời dây cảm biến → ADC trôi tùy ý, `constrain` vẫn cho ra 0–100 % → hệ thống tin số sai. Bồn hết nước → bơm chạy mãi ở AUTO vì độ ẩm không lên.

**Sửa.**
- Trong `readMoisturePercent()`: nếu ADC < `ADC_WET - 300` hoặc > `ADC_DRY + 300` (tùy chỉnh) → đặt `sensorFault = true`, trả về `-1`.
- Trong `controlPump()`: nếu `sensorFault` ở chế độ AUTO → **tắt bơm**.
- Thêm `AUTO_ON_MAX_MS` (VD 3 phút): AUTO bật bơm quá thời gian này mà độ ẩm không tăng ≥ 5 % → tắt bơm, đặt cờ lỗi (bồn hết nước / ống tắc).
- Gửi mã lỗi lên **field4** (0 = bình thường, 1 = cảm biến lỗi, 2 = bơm chạy không hiệu quả); dashboard hiển thị cảnh báo.

**Báo cáo:** 2.5, 3.3 (dòng "Cảm biến hỏng" → có xử lý), 3.5.

## 8. Watchdog 🟡

**Sửa.** Bật Task Watchdog của ESP-IDF (có sẵn trong Arduino-ESP32):
```cpp
#include <esp_task_wdt.h>
// setup():  esp_task_wdt_init(30, true); esp_task_wdt_add(NULL);
// loop():   esp_task_wdt_reset();
```
Treo > 30 s (VD request HTTP kẹt) → ESP32 tự khởi động lại; relay khởi tạo LOW nên bơm tắt an toàn.

> 📝 Kiểm tra cú pháp `esp_task_wdt_init` theo phiên bản Arduino-ESP32 đang dùng (bản 3.x đổi sang struct cấu hình).

**Báo cáo:** 3.3, 3.5.

## 9. Dashboard 🟡

**Sửa.**
- `getStatus()` (`app.service.ts`): nếu vẫn dùng phương án field3/field4 (chưa làm TalkBack), đọc `feeds.json?results=5` và lấy entry gần nhất **có field1** thay vì `feeds/last.json`.
- Thêm `GET /api/history?results=N` đọc `feeds.json?results=N` → trả mảng `{time, moisture, pump}`.
- `index.html`: vẽ biểu đồ độ ẩm + trạng thái bơm theo thời gian (Chart.js từ CDN). Ảnh này dùng luôn cho H4/H5 của báo cáo.
- Hiển thị cảnh báo lỗi (mục 7) nếu có.

**Báo cáo:** 2.5e, 2.6, 3.5 (bỏ hạn chế "dashboard chỉ có giá trị mới nhất").

## 10. README 🔴

Nội dung tối thiểu:
- Mô tả hệ thống, sơ đồ khối
- Linh kiện + bảng đấu dây (copy từ báo cáo 2.2, 2.3)
- Cách tạo `secrets.h` từ `secrets.h.example`, nạp firmware bằng PlatformIO
- Cách cấu hình ThingSpeak (channel, field 1–3, TalkBack nếu dùng)
- Cách chạy dashboard: `cp .env.example .env`, `npm install`, `npm run start:dev`

---

## Thứ tự làm đề xuất

1. Mục **1** (đổi key) — ngay.
2. Mục **4, 5, 6** — sửa logic.
3. Mục **7, 8, 9** — nếu còn thời gian.
4. Mục **2, 3** — hiệu chuẩn và thêm log đo.
5. Test toàn bộ → **đóng băng code**, ghi mã commit → đo đạc, chụp ảnh.
6. Mục **10** — README.
7. Cập nhật các mục báo cáo bị ảnh hưởng (cột "Báo cáo" ở từng mục).
