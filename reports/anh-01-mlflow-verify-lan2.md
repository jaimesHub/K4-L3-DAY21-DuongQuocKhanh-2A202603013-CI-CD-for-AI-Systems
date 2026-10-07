# Báo Cáo Kiểm Tra Lần 2: Ảnh 01-mlflow-ui.png

**Ngày kiểm tra:** 2026-10-07  
**Thời gian:** 12:37 - 12:40 UTC+7

---

## So Sánh Hai Ảnh

| Tiêu Chí | Ảnh Cũ (12:28) | Ảnh Mới từ Chat (12:37) | Kết Luận |
|---------|-------------|----------------------|---------|
| **Tên file nộp** | `01-mlflow-ui.png` | Chưa cập nhật (ảnh trong chat) | ⚠️ Ảnh file chưa thay thế |
| **Sort** | "Created" | "f1_score" | ✓ Ảnh mới đúng, ảnh file sai |
| **Cột f1_score** | ❌ Không thấy | ❌ Không thấy | ✗ FAIL |
| **Cột accuracy** | ❌ Không thấy | ❌ Không thấy | ✗ FAIL |
| **Cột n_estimators** | ❌ Không thấy | ❌ Không thấy | ✗ FAIL |
| **Cột learning_rate** | ❌ Không thấy | ❌ Không thấy | ✗ FAIL |
| **Cột max_depth** | ❌ Không thấy | ❌ Không thấy | ✗ FAIL |
| **Số runs** | 4 runs | 4 runs | ✓ PASS (>= 3) |

---

## Phân Tích Chi Tiết

### 1. **File Metadata**

| Tiêu Chí | Giá Trị | Trạng Thái |
|---------|--------|-----------|
| Đường dẫn file | `nop-bai/anh-chup-man-hinh/01-mlflow-ui.png` | ✓ Đúng |
| Dung lượng | 270 KB | ✓ PASS (< 1 MB) |
| Định dạng | PNG (RGBA, 8-bit) | ✓ PASS |
| Kích thước | 2866 x 952 pixels | ✓ Đủ rõ |
| Lần sửa đổi cuối | Oct 7 12:28 | ⚠️ Chưa cập nhật |

### 2. **Nội Dung MLflow UI - Ảnh File (12:28)**

**Vấn Đề 1: Sắp Xếp Sai**
- ❌ Hiện tại: `Sort: Created` (giảm dần theo thời gian tạo)
- ✓ Yêu cầu: `Sort: f1_score` (giảm dần theo điểm f1)
- **Chi tiết:** Ảnh sắp xếp theo thời gian, không phải theo hiệu suất mô hình

**Vấn Đề 2: Thiếu Cột Metrics**
- ❌ Cột `f1_score` - KHÔNG THẤY
- ❌ Cột `accuracy` - KHÔNG THẤY

**Vấn Đề 3: Thiếu Cột Hyperparameters**
- ❌ Cột `n_estimators` - KHÔNG THẤY
- ❌ Cột `learning_rate` - KHÔNG THẤY
- ❌ Cột `max_depth` - KHÔNG THẤY

**Cột Hiện Có:**
```
✓ Run Name
✓ Created
✓ Dataset
✓ Duration
✓ Source
✓ Models
```

### 3. **Nội Dung MLflow UI - Ảnh Mới từ Chat (12:37)**

**Tiến Bộ:**
- ✓ `Sort: f1_score` - ĐÚNG! Đã sửa sắp xếp

**Vẫn Còn Vấn Đề:**
- ❌ Cột `f1_score` - KHÔNG THẤY (chỉ sắp xếp, chưa hiển thị)
- ❌ Cột `accuracy` - KHÔNG THẤY
- ❌ Cột `n_estimators` - KHÔNG THẤY
- ❌ Cột `learning_rate` - KHÔNG THẤY
- ❌ Cột `max_depth` - KHÔNG THẤY

**Nguyên Nhân:** Chưa nhấn nút **"Columns"** để thêm cột vào bảng. Chỉ sắp xếp theo f1_score nhưng chưa hiển thị cột này.

### 4. **Các Runs Hiện Tại**

| Run Name | Thời Gian | Status |
|----------|-----------|--------|
| incongruous-wasp-415 | 16 min ago | ✓ Success |
| delightful-doe-269 | 16 min ago | ✓ Success |
| industrious-gnu-850 | 17 min ago | ✓ Success |
| trusting-fish-836 | 17 min ago | ✓ Success |

- **Số runs:** 4 (✓ PASS - yêu cầu >= 3)
- **Kích thước dữ liệu:** Đủ để chụp

### 5. **Kết Quả Mong Đợi (Không Xác Nhận Được)**

Không thể so sánh vì các cột f1_score, accuracy, n_estimators, learning_rate, max_depth không hiển thị.

Kết quả kỳ vọng theo yêu cầu:
```
n_estimators=150, learning_rate=0.15, max_depth=4  → f1: 0.7182, acc: 0.876
n_estimators=200, learning_rate=0.10, max_depth=5  → f1: 0.7149, acc: 0.874
n_estimators=100, learning_rate=0.10, max_depth=3  → f1: 0.7109, acc: 0.878
n_estimators=50,  learning_rate=0.05, max_depth=2  → f1: 0.6051, acc: 0.846
```

---

## Bảng Kết Quả - PASS/FAIL

| Hạng Mục | Trạng Thái | Ghi Chú |
|---------|-----------|--------|
| **Tên file** | ✓ PASS | `01-mlflow-ui.png` - đúng |
| **Định dạng & dung lượng** | ✓ PASS | PNG, 270 KB |
| **Số runs >= 3** | ✓ PASS | Có 4 runs |
| **Sắp xếp f1_score** | ✗ FAIL (file) / ⚠️ Bộ phận (chat) | File: sort theo "Created"; Chat: sort f1_score nhưng chưa hoàn toàn |
| **Cột f1_score hiển thị** | ✗ FAIL | Chưa thêm vào bảng |
| **Cột accuracy hiển thị** | ✗ FAIL | Chưa thêm vào bảng |
| **Cột n_estimators hiển thị** | ✗ FAIL | Chưa thêm vào bảng |
| **Cột learning_rate hiển thị** | ✗ FAIL | Chưa thêm vào bảng |
| **Cột max_depth hiển thị** | ✗ FAIL | Chưa thêm vào bảng |
| **Chất lượng ảnh** | ✓ PASS | Chữ rõ, không bị cắt (cả hai ảnh) |
| **Đối chiếu số liệu** | ❓ KHÔNG XÁC NHẬN | Vì không thấy metric/param columns |

---

## Kết Luận

### Tóm Tắt Tình Hình

1. **Ảnh file (`01-mlflow-ui.png` - 12:28):** Chưa được cập nhật, vẫn thiếu cột metrics/params, sắp xếp sai
2. **Ảnh mới (chat - 12:37):** Có tiến bộ (sắp xếp f1_score đúng) nhưng vẫn chưa đủ (chưa hiển thị cột)

### Status Chung

🔴 **FAIL** - Ảnh nộp chưa đáp ứng yêu cầu

---

## Hướng Dẫn Chụp Lại (Chi Tiết)

**Bước 1:** Mở MLflow UI
```
http://localhost:5000
```

**Bước 2:** Vào tab **Experiments** → **Default** → **Table**

**Bước 3:** Sắp xếp theo f1_score
- Nhấn nút `Sort: Created` (hoặc bất kỳ sort hiện tại)
- Chọn `f1_score`
- Chọn **giảm dần** (↓ - Descending)

**Bước 4:** Thêm cột vào bảng
- Nhấn nút **Columns** (góc phải của bảng)
- Tick (✓) những cột sau:
  ```
  ✓ f1_score      (metrics)
  ✓ accuracy      (metrics)
  ✓ n_estimators  (params)
  ✓ learning_rate (params)
  ✓ max_depth     (params)
  ```
- Đóng popup Columns

**Bước 5:** Kiểm tra bảng
- Bảng phải hiển thị tất cả 5 cột trên
- Các runs sắp xếp theo f1_score giảm dần (cao nhất ở trên)
- Thấy rõ >= 3 runs với các cột metric/param

**Bước 6:** Chụp ảnh
- Chụp để thấy rõ: Run Name, f1_score, accuracy, n_estimators, learning_rate, max_depth
- Đảm bảo chữ có thể đọc được, không bị cắt
- Lưu lại `01-mlflow-ui.png` (thay thế file cũ)

**Bước 7:** Nộp
- File lưu tại: `nop-bai/anh-chup-man-hinh/01-mlflow-ui.png`
- Commit & push lên GitHub

---

## Ghi Chú Kỹ Thuật

- **Ảnh file:** `/Users/jaimes/Working/AI20K-IV/track-2/labs/Day21-Track2-Assignment/nop-bai/anh-chup-man-hinh/01-mlflow-ui.png` (270 KB, modified Oct 7 12:28)
- **Ảnh mới từ chat:** `/private/tmp/claude-501/.../images/2.png` (241 KB, created Oct 7 12:37)
- **Báo cáo cũ:** `reports/anh-01-mlflow-verify.md` (Oct 7, kết luận FAIL)
- **Báo cáo lần 2:** `reports/anh-01-mlflow-verify-lan2.md` (tài liệu này)

---

## Người Kiểm Tra

- **AI:** Claude Haiku 4.5
- **Email:** duongquockhanh230596@gmail.com
- **Thời gian:** 2026-10-07 12:40 UTC+7
