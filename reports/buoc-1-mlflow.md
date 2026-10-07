# Báo Cáo Bước 1: Thực Nghiệm Cục Bộ Với MLflow

## Tóm Tắt

Bước 1 được thực hiện thành công với 4 lần chạy thí nghiệm sử dụng `GradientBoostingClassifier` trên tập dữ liệu Adult. Mỗi lần chạy sử dụng bộ siêu tham số khác nhau. Kết quả được theo dõi bằng MLflow và lưu trữ cục bộ dưới dạng SQLite.

---

## Bảng Tóm Tắt Các Run

| Lần | n_estimators | learning_rate | max_depth | F1 Score | Accuracy | Nhận Xét |
|-----|---------|---------|---------|---------|---------|---------|
| 1 | 100 | 0.10 | 3 | 0.7109 | 0.8780 | Tham số mặc định |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 | Tham số nhỏ, F1 thấp |
| 3 | 200 | 0.10 | 5 | 0.7149 | 0.8740 | Tham số lớn, F1 cao |
| 4 | 150 | 0.15 | 4 | **0.7182** | 0.8760 | **TỐT NHẤT** |

---

## Bộ Siêu Tham Số Chọn

### Thông Tin

- **n_estimators**: 150
- **learning_rate**: 0.15
- **max_depth**: 4

### Kết Quả

- **F1 Score**: 0.7182 (vượt ngưỡng 0.65)
- **Accuracy**: 0.8760

### Lý Do Chọn

1. **F1 Score cao nhất**: Bộ tham số này đạt F1 = 0.7182, cao hơn các bộ khác
2. **Vượt ngưỡng chất lượng**: F1 >= 0.65 là yêu cầu bắt buộc để triển khai ở Bước 2
3. **Cân bằng hiệu suất**: Accuracy đạt 0.8760, cho thấy mô hình không chỉ giỏi ở lớp dương mà còn dự đoán đúng lớp âm

---

## Giải Thích: Tại Sao Dùng F1 Thay Vì Accuracy

### Bài Toán Mất Cân Bằng Lớp

Tập dữ liệu Adult có phân bố lớp **mất cân bằng**: chỉ 24.8% mẫu thuộc lớp dương (thu nhập > 50K), 75.2% là lớp âm (thu nhập <= 50K).

### Vấn Đề Với Accuracy

Một mô hình đơn giản "luôn trả lời thu nhập thấp" (lớp âm) sẽ đạt:
- **Accuracy = 0.752** (trông rất cao, đánh lừa người phát triển)
- **F1 Score = 0.000** (không bắt được một trường hợp thu nhập cao nào)

Mô hình này hoàn toàn vô dụng vì nó không bao giờ dự đoán lớp dương, nhưng accuracy vẫn cao.

### Tại Sao F1 Tốt Hơn

F1 Score là **trung bình điều hòa** giữa Precision và Recall của lớp dương:

```
F1 = 2 * (Precision * Recall) / (Precision + Recall)
```

Với bài toán mất cân bằng:
- F1 **bắt buộc** mô hình phải bắt được cả lớp dương (recall cao) và dự đoán chính xác (precision cao)
- Mô hình "luôn trả lời thấp" sẽ có F1 = 0, không thể lừa được
- Ngưỡng F1 >= 0.65 đảm bảo mô hình thực sự học được bài toán

### Bằng Chứng Từ Thí Nghiệm

Lần 2 của chúng ta (n_estimators=50, learning_rate=0.05, max_depth=2) có:
- **Accuracy**: 0.8460 (thấp hơn mặc dù không phải lúc nào cũng xấu)
- **F1 Score**: 0.6051 (thấp, cho thấy mô hình yếu trong việc nhận diện lớp dương)

---

## Lệnh Chạy

```bash
# Cấu hình MLflow (chỉ cần một lần)
export MLFLOW_TRACKING_URI=sqlite:///mlflow.db
export MLFLOW_ARTIFACT_ROOT=./mlartifacts

# Lần 1: Tham số mặc định
python src/train.py

# Lần 2: Giảm n_estimators và learning_rate
# (sửa params.yaml)
python src/train.py

# Lần 3: Tăng n_estimators và max_depth
# (sửa params.yaml)
python src/train.py

# Lần 4: Cân bằng siêu tham số (TỐT NHẤT)
# (sửa params.yaml)
python src/train.py

# Xem MLflow UI
mlflow ui --backend-store-uri sqlite:///mlflow.db
# Truy cập http://localhost:5000
```

---

## Vấn Đề Gặp Phải

### 1. Lỗi SQLAlchemy Import

**Triệu chứng**: `ImportError: cannot import name 'FallbackAsyncAdaptedQueuePool'`

**Nguyên nhân**: Phiên bản SQLAlchemy 2.1.3 không tương thích với MLflow 2.13.0

**Giải pháp**: Cài đặt SQLAlchemy 2.0.23 (phiên bản tương thích)
```bash
pip install --upgrade sqlalchemy==2.0.23
```

### 2. Tạo Thư Mục outputs/ Và models/

**Triệu chứng**: Lỗi khi lưu model hoặc report

**Giải pháp**: Code đã sử dụng `os.makedirs(..., exist_ok=True)` để tự động tạo thư mục nếu chưa có

---

## Việc Người Dùng Cần Làm Thêm

1. **Chụp ảnh MLflow UI**: Chạy `mlflow ui --backend-store-uri sqlite:///mlflow.db` và chụp lại giao diện hiển thị ít nhất 3 run, lưu thành `nop-bai/anh-chup-man-hinh/01-mlflow-ui.png`

2. **Điền báo cáo**: Cập nhật file `nop-bai/bao-cao.md` với:
   - Bộ siêu tham số chọn: n_estimators=150, learning_rate=0.15, max_depth=4
   - F1 Score đạt được: 0.7182
   - Giải thích vì sao dùng F1 thay vì accuracy
   - Bất kỳ khó khăn gặp phải trong quá trình thực hiện

3. **Chuẩn bị cho Bước 2**: Các file sau đã được tạo và sẵn sàng:
   - `src/train.py`: Script huấn luyện hoàn chỉnh
   - `params.yaml`: Cơ hình tối ưu đã được cập nhật
   - `outputs/report.json`: Chứa F1 và accuracy
   - `models/model.joblib`: Lưu model huấn luyện cuối cùng

---

## File Liên Quan

- `src/train.py`: Script huấn luyện hoàn chỉnh
- `params.yaml`: Cơ hình tối ưu (đã cập nhật)
- `outputs/report.json`: Kết quả F1 và accuracy
- `models/model.joblib`: Model huấn luyện cuối cùng
- `mlflow.db`: Cơ sở dữ liệu MLflow (SQLite)
- `mlartifacts/`: Artifacts từ MLflow (model files)

---

**Ngày thực hiện**: 2026-10-07  
**Trạng thái**: Hoàn thành
