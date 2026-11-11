# BÀI LÀM THI GIỮA KỲ – HỆ THỐNG NHÚNG

**Họ tên:** ............................ **MSSV:** ................ **STT:** 26 → **ĐỀ SỐ 1** (26 chia 5 dư 1)

> Tên file nộp: `26_MSSV_Họ và tên.pdf`

---

## Câu 1 (2 điểm). Thời gian thực cứng và thời gian thực mềm

**1. Phân biệt**

| Tiêu chí | Thời gian thực cứng | Thời gian thực mềm |
|---|---|---|
| Thời hạn hoàn thành | Bắt buộc, **không được trễ** | Cho phép **thỉnh thoảng trễ** |
| Kết quả trễ | **Vô giá trị**, hệ thống coi như thất bại | **Vẫn dùng được**, chỉ giảm chất lượng |
| Mức nguy hiểm | Có thể gây hại người, hỏng thiết bị | Chỉ gây khó chịu cho người dùng |

**2. Ví dụ (lò vi sóng)**

- **Cứng:** khi người dùng mở cửa, phải **ngắt magnetron (bộ phát vi sóng) ngay** trong vài ms.
- **Mềm:** **cập nhật màn hình** đếm ngược thời gian, phát tiếng bíp khi nấu xong.

**3. Hậu quả khi bị trễ**

- **Cứng bị trễ:** vi sóng rò ra ngoài khi cửa đang mở, gây **bỏng, nguy hiểm cho người dùng**.
- **Mềm bị trễ:** màn hình nhảy số chậm, bíp trễ vài trăm ms, người dùng hơi khó chịu nhưng **thức ăn vẫn chín, không nguy hiểm**.

---

## Câu 2 (2.5 điểm). Kiến trúc Harvard và Von Neumann

**1. So sánh**

| Tiêu chí | Harvard | Von Neumann |
|---|---|---|
| Bộ nhớ, bus | **Tách riêng** bộ nhớ và bus lệnh / dữ liệu | **Dùng chung** một bộ nhớ, một bus |
| Đọc lệnh và đọc/ghi dữ liệu | **Đồng thời** trong cùng một chu kỳ | **Lần lượt**, phải chờ nhau (nút cổ chai Von Neumann) |
| Ưu điểm | Tốc độ, thông lượng cao | Đơn giản, rẻ, chia bộ nhớ linh hoạt |

**2. Lựa chọn: kiến trúc Harvard** (ví dụ chip DSP, ARM Cortex-M4, dsPIC)

**3. Giải thích**

- Giải mã âm thanh là luồng dữ liệu liên tục, mỗi chu kỳ vừa **lấy lệnh** vừa **lấy mẫu âm thanh và hệ số** để nhân-cộng.
- Ghi dữ liệu cảm biến chạy song song **không tranh bus** với việc đọc lệnh.
- Nhờ vậy đủ thông lượng, **âm thanh không bị giật** và **không mất mẫu cảm biến**.

---

## Câu 3 (2.5 điểm). Super-loop và RTOS

**1. Dùng super-loop: điều khiển từ xa TV (remote)**

- Chỉ có vài việc chạy **tuần tự**: quét phím, mã hóa lệnh, phát tia hồng ngoại, rồi ngủ.
- **Không có tác vụ song song**, yêu cầu thời gian không gắt.
- Chip nhỏ, RAM ít, cần **rẻ, đơn giản, dễ debug**. Dùng RTOS là thừa và tốn bộ nhớ.

**2. Dùng RTOS: flycam (drone)**

- Có **nhiều tác vụ chạy song song**: giữ thăng bằng (khoảng 1000 lần/giây), nhận lệnh từ tay điều khiển, đọc GPS, giám sát pin.
- Các tác vụ có **deadline và mức ưu tiên khác nhau**, cần RTOS **ngắt ngang (preemption)** để chạy tác vụ quan trọng trước.
- Nếu dùng super-loop, một tác vụ chậm (như xử lý GPS) làm trễ vòng giữ thăng bằng, có thể khiến **drone mất cân bằng và rơi**.

---

## Câu 4 (3 điểm). Thiết bị ngoài trời: -5°C đến 45°C, mưa, bụi

**a) Đề xuất**

- **Cấp độ IP: IP65**
- **Vật liệu vỏ: nhôm đúc, sơn tĩnh điện**

**b) Giải thích**

**Chọn IP65:**

- Số **6** = **kín bụi hoàn toàn**, số **5** = **chống tia nước từ mọi hướng**, đủ cho mưa to và mưa tạt.
- **IP54 không đủ:** bụi vẫn lọt vào một phần, chỉ chống nước bắn nhẹ.
- **IP67 là thừa:** thiết bị không bị ngâm nước, mà IP67 đắt hơn và kín hoàn toàn nên khó tản nhiệt.

**Chọn nhôm:**

- **Tản nhiệt tốt:** nắng 45°C có thể làm trong vỏ nóng trên 60°C, nhôm giúp linh kiện không quá nhiệt.
- **Bền ngoài trời:** chịu tia UV, không giòn ở -5°C, chống gỉ tốt khi sơn tĩnh điện, nhẹ hơn thép.
- **Không chọn nhựa ABS** vì bị tia UV làm giòn và nứt, tản nhiệt kém. **Không chọn thép không gỉ** vì nặng và đắt, chỉ cần khi đặt ở vùng biển hoặc nơi có hóa chất.

**Lưu ý thêm:**

- Nếu nơi lắp đặt có thể bị ngập thì nâng lên **IP67**.
- Vỏ nhôm chắn sóng, nên **gắn anten ra ngoài** nếu thiết bị dùng Wi-Fi hoặc 4G.

---

*HẾT*
