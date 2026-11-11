# Tổng hợp số đo — 10/11/2026, firmware `ab6b2f2`

Log gốc cùng thư mục. Thời gian trong log = giờ máy tính (host) + `millis()` của ESP32 trong dòng `[T]`.

## Hiệu chuẩn cảm biến (02–10)

| Mốc | Kết quả | Log |
|---|---|---|
| Không khí, cấp 3V3 liên tục | 4095 (bão hòa) | 01 |
| Nước, cấp 3V3 liên tục | 1036 → ~1440 sau 4.5 phút, tăng ~100/phút, không ổn định | 02, 03, 04 |
| Nước, GPIO25, chờ 20 ms | 598 → ~190, giảm dần rồi ổn định ~165 ± 8 | 05, 06 |
| Không khí, GPIO25, chờ 20 ms | 2780 (sai — AO chưa nạp xong) | 07 |
| Đường cong AO sau khi bật GPIO25, không khí | τ ≈ 45 ms, đạt 4095 sau ~110 ms | 07 (dòng `[DIAG]`) |
| Không khí, GPIO25, chờ 200 ms | 4095 (15/15 lần) | 08 |
| Nước, GPIO25, chờ 200 ms | 419 → ~580, ổn định từ phút 2; 60 s cuối TB 581, min 539, max 603, σ 12.9 | 09 |

→ `ADC_DRY = 4095`, `ADC_WET = 580`.

## Hiệu năng

| Chỉ tiêu | Kết quả | Log |
|---|---|---|
| Chu kỳ loop, vòng không có HTTP | 1220 ms (59/60 vòng; 1 vòng 1219 ms) | 11 |
| Chu kỳ loop, vòng có ghi channel | 2132–2457 ms (n=5) | 11 |
| 1 request ghi channel `write_fields` (HTTP 200) | TB 977 ms, min 789, max 1210 (n=11) | 01, 11, 13 |
| 1 request ghi channel bị timeout | 5344 ms (HTTP −304), 1 lần | 12 |
| NFR5 — khởi động → gói đầu tiên thành công | TB 3.79 s, min 3.36, max 4.13 (n=7) | 11, 13 |
| Reset khi lần ghi trước < 15 s | lần ghi đầu bị từ chối HTTP −401 (4/4 lần) | 11, 12, 13 |
| NFR9 — treo → watchdog kích | TB 29.98 s, min 29.90, max 30.11 (n=5) | 14, 15 |
| NFR9 — treo → `setup_done` (chạy lại) | TB 32.91 s, min 32.54, max 33.35 (n=5) | 14, 15 |

## NFR1 — đất khô → relay đóng (log 16, chưa cắm 12 V, 6 lần bật + 5 lần tắt)

- Relay chuyển trạng thái ngay sau lần đọc vượt ngưỡng: `pump_on/off` cách đầu vòng lặp 220–221 ms (= thời gian đọc cảm biến), xử lý ≈ 0 ms.
- Độ trễ tổng = chờ tới lần đọc kế tiếp (0 … chu kỳ loop) + 0.22 s.
- 7/11 sự kiện rơi vào vòng thường (dt = 1220 ms) → trễ ≤ 1.44 s.
- 4/11 sự kiện rơi ngay sau vòng có ghi channel (dt = 3208, 4397, 5575, 6567 ms) → trễ tối đa tới 6.8 s.
- Trong log này `write_fields` mất 916–5344 ms (n=12; 1 lần timeout HTTP −304) do WiFi phát từ điện thoại chậm → vòng lặp bị chặn trong lúc chờ HTTP.
- Vùng trễ hoạt động: 1 lần nhúng nông, ADC 1865 → 63 % ≥ 60 % → tắt; nhấc ra → 0 % → bật lại.
- Kết luận: NFR1 (≤ 2 s) đạt khi mạng nhanh/không có request; **không đạt ở trường hợp xấu nhất** vì request HTTP chặn vòng điều khiển.

## H6 — lỗi cảm biến (log 17)

- 22:14:24 rút dây VCC (D25) → ADC 0 → `LỖI cảm biến` ngay lần đọc đầu; 22:14:30 cắm lại → ADC 385 → `Cảm biến hoạt động lại`.
- Lần rút thứ 2 (22:14:39–22:15:05): ADC giảm dần 221 → 41 (tụ ở AO xả chậm qua nước) thay vì về 0 ngay; lần đọc đầu đã < 280 nên vẫn phát hiện ngay.

## H7 / NFR8 — mất WiFi (log 18)

- 22:19:57 lần ghi đầu sau khi mất mạng thất bại (HTTP −301, 3005 ms); 22:20:03 `[WiFi] Mất kết nối!`.
- Offline 126.9 s: `maintainWiFi()` thử kết nối lại mỗi ~11 s, 6 lần gửi bị bỏ qua (`Offline — bỏ qua lần gửi này`).
- **NFR8: chu kỳ loop khi mất mạng TB 1220 ms, min 1220, max 1229 (n=104)** → không bị chặn, đạt ≤ 2 s.
- 22:22:10 `Đã kết nối lại`, lần ghi kế tiếp 22:22:20 thành công (HTTP 200).
- Lần này chưa thử nhấc đầu dò trong lúc offline (chưa có log bơm bật/tắt khi mất mạng).

## NFR2 — lệnh từ xa (log 20, giờ bấm nút: 20_nfr2_thoi_diem_bam_nut.json)

| Lệnh | Bấm lúc | ESP32 đổi chế độ | Trễ |
|---|---|---|---|
| ON | 22:33:32.353 | MANUAL_ON | 7.45 s |
| AUTO | 22:34:09.629 | AUTO | 0.92 s |
| OFF | 22:34:29.643 | MANUAL_OFF | 1.88 s |
| AUTO | 22:34:49.659 | AUTO | 7.51 s |
| ON | 22:35:18.255 | MANUAL_ON | 2.09 s |
| AUTO | 22:35:38.271 | **mất lệnh** — `talkback_execute` HTTP −11 sau 5340 ms | — |

- NFR2: TB 3.97 s, min 0.92, max 7.51 (n = 5) → đạt ≤ 10 s.
- `talkback_execute` (HTTP 200): TB 1529 ms, min 848, max 4617 (n = 31).
- Phát hiện cấu hình: TalkBack bật *Log to Channel* → entry rỗng (status = lệnh) xen vào channel, làm 1 lần ghi của ESP32 bị −401; Field 4 chưa bật → mã lỗi không lưu.

## Ngưỡng lỗi cảm biến (log 19, 21, 22)

- Ngưỡng cũ `ADC_WET − 300` = 280: sau khi khởi động, đầu dò để yên trong nước đọc 196–278 → báo LỖI nhầm (log 19).
- Ngưỡng 100: lần đọc đầu 126 (log 21) — quá sát. Chọn **50**: log 22 đọc 140, không báo lỗi.

Chưa đo: dòng tiêu thụ.
