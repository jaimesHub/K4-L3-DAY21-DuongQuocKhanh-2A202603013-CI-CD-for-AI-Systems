# Báo Cáo Kiểm Tra Ảnh 01-mlflow-ui.png

## Thông Tin File

| Tiêu Chí | Kết Quả | Ghi Chú |
|---------|--------|--------|
| Tên file | PASS | `01-mlflow-ui.png` - đúng theo yêu cầu |
| Định dạng | PASS | PNG image data (8-bit/color RGBA, non-interlaced) |
| Dung lượng | PASS | 270 KB < 1 MB |
| Kích thước ảnh | OK | 2866 x 952 pixels - đủ rõ |

---

## Kiểm Tra Nội Dung MLflow UI

### 1. Số Lần Chạy (Runs)

| Tiêu Chí | Kết Quả | Chi Tiết |
|---------|--------|---------|
| Số runs tối thiểu | PASS | Thấy 4 lần chạy: incongruous-wasp-415, delightful-doe-269, trusting-fish-836, industrious-gnu-850 |
| Yêu cầu | >= 3 runs | ✓ Đạt |

---

### 2. Các Cột Metric Cần Thiết

| Cột Yêu Cầu | Tình Trạng | Ghi Chú |
|------------|-----------|--------|
| **f1_score** | ❌ KHÔNG THẤY | Cột này không hiển thị trong ảnh |
| **accuracy** | ❌ KHÔNG THẤY | Cột này không hiển thị trong ảnh |

**Cột hiện tại thấy rõ:**
- Run Name (tên chạy)
- Created (thời gian tạo)
- Dataset
- Duration (thời gian chạy)
- Source (train.py)
- Models (sklearn)

---

### 3. Các Cột Siêu Tham Số (Hyperparameters)

| Cột Yêu Cầu | Tình Trạng | Ghi Chú |
|------------|-----------|--------|
| **n_estimators** | ❌ KHÔNG THẤY | Không hiển thị |
| **learning_rate** | ❌ KHÔNG THẤY | Không hiển thị |
| **max_depth** | ❌ KHÔNG THẤY | Không hiển thị |

---

### 4. Sắp Xếp Theo f1_score

| Tiêu Chí | Tình Trạng | Ghi Chú |
|---------|-----------|--------|
| Sắp xếp giảm dần | ❌ KHÔNG THỂ XÁC NHẬN | Cột f1_score không hiển thị, nên không xác nhận được thứ tự sắp xếp |

Ảnh hiện tại sắp xếp theo "Created" (thời gian tạo), không phải f1_score.

---

### 5. Chất Lượng Ảnh

| Tiêu Chí | Kết Quả | Ghi Chú |
|---------|--------|--------|
| Chữ đọc rõ | PASS | Tất cả text đều đọc rõ ràng, không bị cắt |
| Độ sáng/tương phản | PASS | Giao diện MLflow rõ ràng, không bị xấu |

---

## So Sánh Với Kết Quả Thực Tế (Dữ Liệu Kỳ Vọng)

Theo yêu cầu, 4 lần chạy này phải có các kết quả sau:

| Hyperparameters | f1_score (Kỳ Vọng) | accuracy (Kỳ Vọng) |
|-----------------|-------------------|-------------------|
| n_estimators=150, learning_rate=0.15, max_depth=4 | 0.7182 | 0.876 |
| n_estimators=200, learning_rate=0.10, max_depth=5 | 0.7149 | 0.874 |
| n_estimators=100, learning_rate=0.10, max_depth=3 | 0.7109 | 0.878 |
| n_estimators=50, learning_rate=0.05, max_depth=2 | 0.6051 | 0.846 |

**Kết quả so sánh:** ❌ **KHÔNG THỂ XÁC NHẬN** - vì ảnh không hiển thị các cột f1_score, accuracy, n_estimators, learning_rate, max_depth.

---

## Kết Luận Tổng Hợp

### Tóm Tắt Kết Quả

| Hạng Mục | Trạng Thái | Chi Tiết |
|---------|-----------|---------|
| **File Metadata** | ✓ PASS | Tên, định dạng, dung lượng đúng |
| **Số Runs** | ✓ PASS | >= 3 runs (có 4 runs) |
| **Cột f1_score** | ✗ FAIL | Không thấy trong ảnh |
| **Cột accuracy** | ✗ FAIL | Không thấy trong ảnh |
| **Cột Hyperparameters** | ✗ FAIL | n_estimators, learning_rate, max_depth không thấy |
| **Sắp xếp f1_score** | ✗ FAIL | Không thể xác nhận (cột không hiển thị) |
| **Chất lượng ảnh** | ✓ PASS | Chữ rõ, không bị cắt |

---

## Khuyến Nghị

**❌ Cần chụp lại ảnh.** 

Lý do:
1. Ảnh hiện tại **thiếu các cột metric quan trọng** (f1_score, accuracy) mà README yêu cầu phải thấy rõ.
2. **Thiếu các cột siêu tham số** (n_estimators, learning_rate, max_depth).
3. **Sắp xếp không đúng** - ảnh sắp xếp theo "Created", không phải "f1_score" giảm dần.

### Cách Chụp Lại Đúng

1. Mở MLflow UI tại `http://localhost:5000`
2. Chuyển sang tab **Table** (nếu chưa)
3. Nhấn nút **"Sort: Created"** → chọn **f1_score** → sắp xếp **giảm dần** (↓)
4. Nhấn nút **"Columns"** (góc phải bảng) → tick những cột:
   - ✓ f1_score
   - ✓ accuracy
   - ✓ n_estimators
   - ✓ learning_rate
   - ✓ max_depth
5. Đảm bảo thấy rõ **ít nhất 3-4 runs** với các cột trên
6. Chụp ảnh và lưu lại `01-mlflow-ui.png`

---

## Ghi Chú Kiểm Tra

- **Ngày kiểm tra:** 2026-10-07
- **Người kiểm tra:** Claude Haiku 4.5
- **Đường dẫn ảnh:** `nop-bai/anh-chup-man-hinh/01-mlflow-ui.png`
