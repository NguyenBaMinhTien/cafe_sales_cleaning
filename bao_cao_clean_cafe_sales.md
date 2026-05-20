# Báo cáo: Phát hiện & Làm sạch dữ liệu Cafe Sales

---

## 1. Tổng quan file dữ liệu

| Thông tin | Chi tiết |
|---|---|
| File gốc | `dirty_cafe_sales_CleanPractice.csv` |
| Số dòng | 10.000 |
| Số cột | 8 |
| Khóa giao dịch | `Transaction ID` (không bị trống, không bị trùng) |

**Danh sách cột:** `Transaction ID`, `Item`, `Quantity`, `Price Per Unit`, `Total Spent`, `Payment Method`, `Location`, `Transaction Date`

---

## 2. Vấn đề tìm thấy

### 2.1 Giá trị lỗi theo từng cột

Dữ liệu lỗi xuất hiện dưới 3 dạng: **ô trống**, **"UNKNOWN"**, và **"ERROR"**.

| Cột | Ô trống | UNKNOWN | ERROR | **Tổng lỗi** |
|---|---:|---:|---:|---:|
| Transaction ID | 0 | 0 | 0 | **0** |
| Item | 333 | 344 | 292 | **969** |
| Quantity | 138 | 171 | 170 | **479** |
| Price Per Unit | 179 | 164 | 190 | **533** |
| Total Spent | 173 | 165 | 164 | **502** |
| Payment Method | 2.579 | 293 | 306 | **3.178** |
| Location | 3.265 | 338 | 358 | **3.961** |
| Transaction Date | 159 | 159 | 142 | **460** |

### 2.2 Vấn đề cụ thể theo nhóm cột

**Cột số (`Quantity`, `Price Per Unit`, `Total Spent`)**
- Bị lẫn giá trị chuỗi lỗi → không thể tính toán trực tiếp.
- Sau khi loại giá trị lỗi: `Quantity` hợp lệ từ 1–5; `Price Per Unit` thuộc các mức cố định; `Total Spent = Quantity × Price Per Unit` (không có dòng nào sai công thức).

**Cột phân loại (`Item`, `Payment Method`, `Location`)**
- `Payment Method` và `Location` có tỷ lệ lỗi rất cao (~32% và ~40%).
- Không tìm được quy luật để suy luận ngược → không thể điền lại.

**Cột ngày (`Transaction Date`)**
- Ngày hợp lệ nằm trong khoảng 2023-01-01 đến 2023-12-31.
- 460 dòng bị thiếu/lỗi ngày → không tự điền để tránh sai phân tích theo thời gian.

**Bảng giá cố định theo mặt hàng** (rút ra từ các dòng sạch):

| Item | Price Per Unit |
|---|---:|
| Cake | 3.0 |
| Coffee | 2.0 |
| Cookie | 1.0 |
| Juice | 3.0 |
| Salad | 5.0 |
| Sandwich | 4.0 |
| Smoothie | 4.0 |
| Tea | 1.5 |

---

## 3. Phương án làm sạch

### Bước 1 — Chuẩn hóa giá trị lỗi
Đổi tất cả ô trống, `"UNKNOWN"`, `"ERROR"` thành `NaN` để xử lý thống nhất.

### Bước 2 — Chuyển kiểu dữ liệu
- `Quantity`, `Price Per Unit`, `Total Spent` → số thực.
- `Transaction Date` → định dạng ngày `YYYY-MM-DD`.

### Bước 3 — Suy luận giá trị số (có cơ sở rõ ràng)

| Nếu biết | Thì điền |
|---|---|
| `Item` | `Price Per Unit` theo bảng giá |
| `Quantity` + `Price Per Unit` | `Total Spent = Quantity × Price Per Unit` |
| `Total Spent` + `Price Per Unit` | `Quantity = Total Spent / Price Per Unit` (nếu kết quả là số nguyên 1–5) |
| `Total Spent` + `Quantity` | `Price Per Unit = Total Spent / Quantity` (nếu giá nằm trong bảng hợp lệ) |

### Bước 4 — Suy luận `Item` (chỉ khi giá đơn trị)

| Price Per Unit | Suy ra Item |
|---|---|
| 1.0 | Cookie |
| 1.5 | Tea |
| 2.0 | Coffee |
| 5.0 | Salad |

> Không suy luận khi giá trùng nhiều mặt hàng: 3.0 có thể là Cake hoặc Juice; 4.0 có thể là Sandwich hoặc Smoothie.

### Bước 5 — Các cột không thể suy luận
- `Payment Method`, `Location`: giữ nguyên là `"Unknown"`.
- `Transaction Date` bị thiếu: giữ `NaN`, đánh dấu để xem lại thủ công.

### Bước 6 — Thêm cột audit
- `Needs Review`: đánh dấu dòng còn thiếu cột quan trọng sau khi clean.
- `Cleaning Notes`: ghi lại thao tác đã xử lý trên từng dòng.

---

## 4. Kết quả sau làm sạch

| Chỉ số | Giá trị |
|---|---:|
| Tổng số dòng giữ nguyên | 10.000 |
| Dòng không cần xem lại | 9.514 |
| Dòng cần xem lại thủ công | 486 |
| Dòng có đủ 3 cột số hợp lệ | 9.974 |
| Dòng sai công thức `Total Spent` | 0 |
| Ngày giao dịch hợp lệ sau clean | 9.540 |
| Ngày giao dịch còn thiếu | 460 |

**Giá trị còn để `"Unknown"` sau khi clean:**

| Cột | Số dòng |
|---|---:|
| Item | 480 |
| Payment Method | 3.178 |
| Location | 3.961 |


File kết quả: `cleaned_cafe_sales.csv`