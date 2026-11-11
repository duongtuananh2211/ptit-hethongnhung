# Đề bài Bài tập lớn cuối kỳ — Xây dựng các Hệ thống Nhúng

> Nguồn: `debai.pdf` (5 trang)

| Thông tin | Chi tiết |
|---|---|
| Trường | Học viện Công nghệ Bưu chính Viễn thông (PTIT) |
| Khoa / Bộ môn | Khoa Công nghệ thông tin 1 — Bộ môn Khoa học máy tính |
| Môn học | Xây dựng các hệ thống nhúng |
| Năm học | 2026 – 2027 |
| Giảng viên phụ trách | Đinh Xuân Trường |
| Quy mô nhóm | Tối đa 5 người |

---

## 1. Mục tiêu

Xây dựng **một hệ thống nhúng hoàn chỉnh** (mô phỏng hoặc thực tế) gồm đủ:

- [ ] Đặc tả hệ thống rõ ràng
- [ ] Thiết kế phần cứng và sơ đồ hệ thống
- [ ] Thiết kế phần mềm và kiến trúc xử lý
- [ ] Mã nguồn có giải thích logic
- [ ] Đánh giá kỹ thuật và khả năng ứng dụng

## 2. Yêu cầu chung

> ⚠️ **Trừ điểm nặng** nếu:
> - Chỉ mô tả lý thuyết, hoặc chèn mã nguồn mà **không có phân tích kỹ thuật**
> - Sao chép từ các nguồn **không trích dẫn**, hoặc không tổng hợp kiến thức rõ ràng

**Lĩnh vực đề tài** (chọn một):

- IoT, điều khiển, giám sát
- Nông nghiệp thông minh, giao thông, y tế
- Hệ thống cảnh báo, tự động hóa

---

## 3. Cấu trúc báo cáo bắt buộc

### Phần 0 — Phân chia công việc

- Bảng mô tả khối lượng công việc của **từng thành viên** + **tỷ lệ % công việc** của mỗi người.

Mẫu gợi ý:

| Thành viên | MSSV | Công việc phụ trách | Tỷ lệ (%) |
|---|---|---|---|
| | | | |

### Phần 1 — Giới thiệu và đặc tả hệ thống

**Bắt buộc có bảng đặc tả hệ thống:**

| Thành phần | Mô tả |
|---|---|
| Input | Dữ liệu đầu vào (loại, tần suất) |
| Output | Hành động hệ thống |
| Ràng buộc | Thời gian, năng lượng, môi trường |

**Phải phân biệt rõ:**

- **Yêu cầu chức năng** — hệ thống làm gì
- **Yêu cầu phi chức năng** — mô tả định lượng, ví dụ: *độ trễ < 1s*

**Không chấp nhận:**

- Mô tả chung chung, không có số liệu
- Sao chép lý thuyết không liên quan

### Phần 2 — Thiết kế hệ thống

#### 2.1 Sơ đồ khối hệ thống (bắt buộc)

Sơ đồ tổng thể dạng:

```
Sensor → MCU → Communication → Server/UI
```

- Có **hình vẽ riêng** cho sơ đồ khối
- Giải thích chức năng/nhiệm vụ từng thành phần — **ngắn gọn, hạn chế dài dòng**

#### 2.2 Thiết kế phần cứng

- Danh sách linh kiện và chức năng
- **Lý do chọn** từng linh kiện
- Phân tích **phương án thay thế** (nếu có)

**Không chấp nhận:** chỉ mô tả linh kiện mà không có lý do lựa chọn.

#### 2.3 Sơ đồ ghép nối

- Sơ đồ ghép nối, đấu dây rõ ràng
- Giải thích từng kết nối (chỉ nêu lý do ngắn gọn)

#### 2.4 Thiết kế phần mềm

**Bắt buộc** có sơ đồ luồng xử lý hoặc kiến trúc. Ví dụ:

```
main()
 ├── readSensor()
 ├── processData()
 ├── sendData()
 └── controlDevice()
```

#### 2.5 Mã nguồn và giải thích

- Không chỉ chèn code
- **Giải thích logic hoạt động**
- **Liên kết với sơ đồ luồng** (mục 2.4)

**Không chấp nhận:**

- Chỉ chú thích từng dòng code
- Không có mô tả tổng thể

#### 2.6 Kết quả hệ thống

- Giao diện / hình ảnh kết quả khi hệ thống chạy hoàn chỉnh, **có giải thích từng hình**.

### Phần 3 — Đánh giá hệ thống

Phải phân tích:

- **Hiệu năng** — độ trễ, tốc độ
- **Tiêu thụ năng lượng**
- **Độ tin cậy** — hành vi khi lỗi xảy ra
- **Khả năng mở rộng**

Bắt buộc:

- Nêu **hạn chế** của hệ thống
- Đề xuất **cải tiến cụ thể**

### Tài liệu tham khảo

- Lấy từ **nguồn chuẩn** (datasheet, sách, bài báo, tài liệu chính thức của nhà sản xuất…)
- Tránh blog hoặc website chưa được kiểm chứng dữ liệu

---

## 4. Tiêu chí chấm điểm

| Tiêu chí | Điểm | Yêu cầu |
|---|---:|---|
| Nội dung báo cáo | 25% | Đặc tả rõ ràng, có phân tích kỹ thuật |
| Thiết kế phần cứng | 30% | Có sơ đồ khối, sơ đồ kết nối, giải thích lựa chọn |
| Giải thích mã nguồn | 20% | Có logic tổng thể, liên kết với hệ thống |
| Tính ứng dụng | 15% | Có kịch bản thực tế, khả năng mở rộng |
| Hình thức trình bày | 10% | Format chuẩn, thống nhất |
| **Tổng** | **100%** | |

## 5. Checklist trước khi nộp

- [ ] Có sơ đồ khối hệ thống riêng
- [ ] Có giải thích sơ đồ
- [ ] Có kết nối giữa mã nguồn và hệ thống
- [ ] Không sao chép lý thuyết dư thừa
- [ ] Có phân tích kỹ thuật trong kết luận
- [ ] Có bảng phân chia công việc + tỷ lệ % từng thành viên

## 6. Hình thức nộp bài

- **Định dạng:** file PDF
- **Tên file:** `HTN_TX_2026 Nhóm X Lớp NY.pdf`
  - `X` = tên nhóm
  - `Y` = số thứ tự lớp
- **Nhóm:** tối đa 5 người
- Bắt buộc kèm bảng mô tả khối lượng công việc và tỷ lệ % của từng thành viên

---

## Phụ lục — Khung báo cáo gợi ý (theo đề)

```
Phần 0. Phân chia công việc
Phần 1. Giới thiệu và đặc tả hệ thống
  1.1 Giới thiệu đề tài, kịch bản ứng dụng
  1.2 Bảng đặc tả (Input / Output / Ràng buộc)
  1.3 Yêu cầu chức năng
  1.4 Yêu cầu phi chức năng (có số liệu)
Phần 2. Thiết kế hệ thống
  2.1 Sơ đồ khối + giải thích
  2.2 Thiết kế phần cứng (linh kiện, lý do chọn, phương án thay thế)
  2.3 Sơ đồ ghép nối + giải thích kết nối
  2.4 Thiết kế phần mềm (sơ đồ luồng / kiến trúc)
  2.5 Mã nguồn và giải thích logic (liên kết sơ đồ luồng)
  2.6 Kết quả hệ thống (hình ảnh + giải thích)
Phần 3. Đánh giá hệ thống
  3.1 Hiệu năng
  3.2 Tiêu thụ năng lượng
  3.3 Độ tin cậy
  3.4 Khả năng mở rộng
  3.5 Hạn chế và đề xuất cải tiến
Tài liệu tham khảo
```
