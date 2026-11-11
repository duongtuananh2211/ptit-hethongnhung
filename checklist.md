# Checklist — BTL Hệ thống nhúng, Nhóm 11

> Đề tài: Hệ thống tưới cây tự động theo độ ẩm đất (ESP32 + ThingSpeak)
> Báo cáo: [bao_cao.md](bao_cao.md) · Đề bài: [debai.md](debai.md)
> Hạn nộp: `[TODO]`

## 1. Làm ngay

- [ ] Hỏi GV: nhóm **6 người** có được không (đề cho tối đa 5)
- [ ] Hỏi GV: số thứ tự lớp **Y** cho tên file
- [ ] ⚠️ Đổi Write/Read API key trên ThingSpeak + đổi mật khẩu WiFi (key cũ vẫn nằm trong commit `a4c7f99` của repo public)
- [x] Xóa key/mật khẩu khỏi `src/main.cpp` → tách ra `secrets.h`, thêm vào `.gitignore` (commit `d8a8318`)
- [x] Tạo TalkBack (ID 58371), điền `secrets.h` + `dashboard/.env`
- [ ] ⚠️ ThingSpeak: tắt *Log to Channel* của TalkBack + bật **Field 4** của channel

## 2. Phần cứng

- [x] Jumper relay ở **HIGH Trigger** — đã thử: relay im lúc khởi động (10/11)
- [x] Relay ăn mức 3.3 V từ GPIO26 — đóng/ngắt đúng 11/11 lần
- [x] Cảm biến cấp 3.3 V — **từ GPIO25** (đổi ngày 10/11, xem 2.2)
- [ ] Lắp diode 1N4007 song song ngược chiều với bơm
- [x] Adapter bơm: 12 V **1 A** (ghi vào 2.2, 3.2, 3.3, 3.5) — khuyến nghị thay ≥ 2 A
- [x] Đo ADC: khô (ngoài không khí) = 4095, ướt (nhúng nước) = 580 → đã sửa `ADC_DRY`/`ADC_WET`, nạp lên board

## 3. Code

- [x] Log đo thời gian `DEBUG_TIMING` (commit `d8a8318`)
- [x] WiFi không chặn, TalkBack, giới hạn MANUAL_ON, phát hiện lỗi, watchdog, biểu đồ dashboard (commit `d8a8318`)
- [x] `README.md`
- [x] Build firmware bằng `pio run` — thành công ở commit `d8a8318` (RAM 14.5 %, Flash 74.1 %); đã nạp lên board ngày 10/11
- [ ] *(Tùy chọn)* `lastSendTime = millis()` sau `writeFields` — nếu sửa thì bỏ dòng tương ứng ở báo cáo 3.3/3.5
- [ ] *(Tùy chọn)* Kiểm tra giới hạn request TalkBack của tài khoản miễn phí (hỏi lệnh mỗi 5 s ≈ 17 000 request/ngày)
- [x] Commit thay đổi cấp nguồn GPIO25 + hiệu chuẩn → `ab6b2f2` trên nhánh `feat/sensor-gpio-power`, đã nạp lên board
- [ ] Merge nhánh vào `master` + push (sau khi đổi key ThingSpeak/WiFi)
- [ ] **Đóng băng code**, commit, ghi mã commit: `________` → cập nhật đầu báo cáo

## 4. Đo đạc (mỗi chỉ tiêu ≥ 5 lần, ghi TB / min / max)

**Hiệu năng** → mục 1.4, 3.1
- [x] Chu kỳ vòng lặp — 1220 ms; vòng có HTTP 2.1–6.6 s
- [x] NFR1 — độ trễ cục bộ: ≤ 1.44 s ở vòng thường, tới 6.8 s khi HTTP chặn (log 16)
- [x] NFR2 — TB 3.97 s, max 7.51 (n=5), 1/6 lệnh mất (log 20)
- [x] NFR4 — sai số độ ẩm (±1 % sau khi ổn định)
- [x] NFR5 — TB 3.79 s (n=7)
- [x] NFR8 — 1220 ms khi mất mạng (n=104), offline 127 s tự kết nối lại (log 18)
- [x] NFR9 — watchdog kích TB 29.98 s, chạy lại sau 32.91 s (n=5)
- [x] `write_fields` TB 977 ms (mạng tốt), 916–5344 ms khi mạng chậm
- [x] `talkback_execute` TB 1529 ms (848–4617, n=31)

**Năng lượng** → mục 3.2 — *không có dụng cụ đo: đã điền ước tính từ datasheet + tỷ lệ thời gian HTTP 40 % đo từ log. Nếu có USB tester thì đo lại các dòng dưới*
- [ ] ESP32 chờ, WiFi kết nối
- [ ] ESP32 đang gửi HTTP
- [ ] Relay kênh 1 đóng
- [ ] Bơm chạy (12 V)
- [ ] Tổng phía USB < 500 mA

## 5. Hình ảnh

**Chụp** → mục 2.6
- [ ] H1 — ảnh mạch thật
- [x] H2 — dựng từ log 22 → `hinh/h2-6_h2_khoi_dong.png`
- [x] H3 — dựng từ log 16 → `hinh/h2-6_h3_bom_tu_dong.png`
- [x] H4 — `hinh/h2-6_h4_dashboard_lich_su.png` (150 điểm, 21:50–22:47; chu kỳ relay, chưa có bơm 12 V)
- [x] H5 — `hinh/h2-6_h5_dashboard_bat_bom.jpg` + `hinh/h2-6_h5_serial_lenh_tu_xa.png`
- [x] H6 — dựng từ log 17 → `hinh/h2-6_h6_loi_cam_bien.png` (chưa có ảnh dashboard)
- [x] H7 — dựng từ log 18 → `hinh/h2-6_h7_mat_wifi.png`
- [x] H7 bổ sung: bỏ — log offline đã cho thấy loop + đọc cảm biến chạy đều, `controlPump()` không phụ thuộc WiFi

**Vẽ**
- [x] Sơ đồ đấu dây → `hinh/h2-3_so_do_dau_day.png` (nguồn SVG trong `hinh/src/`), đã chèn mục 2.3
- [x] Xuất 5 sơ đồ mermaid ra ảnh → `hinh/h2-1…`, `hinh/h2-4a…d`, đã chèn mục 2.1, 2.4

## 6. Viết báo cáo (các chỗ `[TODO]` trong bao_cao.md)

- [ ] Phần 0 — % công việc từng người (tổng 100 %)
- [x] 1.1 — bối cảnh sử dụng (bản nháp: ban công/sân thượng, chủ nhà vắng nhà — sửa nếu khác)
- [x] 1.4 — điền NFR1, 3, 4, 5, 8, 9 (còn NFR2, NFR6)
- [x] 2.2 — giá trị `ADC_DRY`/`ADC_WET` + bảng so sánh trôi số đo 3V3 vs GPIO25
- [x] 2.3 — chèn sơ đồ đấu dây (+ dòng diode 1N4007 trong bảng)
- [ ] 2.6 — ảnh mạch thật (H1)
- [x] 2.5 a–g — trích code + giải thích logic theo code `ab6b2f2`
- [ ] 2.6 — chèn H1–H7, mỗi ảnh 1–2 câu giải thích
- [x] 3.1 — điền số đo + nhận xét (còn NFR2, talkback_execute)
- [x] 3.2 — bảng năng lượng, ngân sách USB, năng lượng/ngày (**ước tính**, chưa đo)
- [x] 3.3–3.5 — viết lại theo code `ab6b2f2`
- [x] Kết luận — đã điền số đo; còn NFR2
- [x] Tài liệu tham khảo — IEEE, thêm relay Songle SRD-05VDC-SL-C + opto PC817, có trích dẫn [n] trong bài
- [ ] Đối chiếu chữ in trên relay/opto của module thật với TLTK [8], [9]

## 7. Trước khi nộp

- [ ] Chuyển sang Word/Docs, đánh số hình/bảng, thống nhất font
- [ ] Xóa hết `[TODO]` và ghi chú `📝`
- [ ] Đối chiếu checklist của đề (cuối [bao_cao.md](bao_cao.md))
- [ ] Repo không còn key, link repo trong báo cáo mở được
- [ ] Xuất PDF: `HTN_TX_2026 Nhóm 11 Lớp N__.pdf`
- [ ] Nộp
