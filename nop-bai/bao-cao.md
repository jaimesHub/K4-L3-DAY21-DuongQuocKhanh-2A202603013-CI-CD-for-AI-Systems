# Báo Cáo Lab Day 21 - CI/CD cho AI Systems


|             |                                                                                                                              |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Họ và tên   | Dương Quốc Khánh                                                                                                             |
| MSSV        | 2A202603013                                                                                                                  |
| Lớp / Khóa  | K4                                                                                                                           |
| Repo GitHub | [https://github.com/jaimesHub/K4-L3-DAY21-DuongQuocKhanh-2A202603013-CI-CD-for-AI-Systems](https://github.com/jaimesHub/K4-L3-DAY21-DuongQuocKhanh-2A202603013-CI-CD-for-AI-Systems) |
| Ngày nộp    |                                                                                                  **8 tháng 10, 2026**        |


---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n\_estimators | learning\_rate | max\_depth | f1\_score | accuracy |
| -------- | ------------- | -------------- | ---------- | --------- | -------- |
| 1        | 100           | 0.10           | 3          | 0.7109    | 0.8780   |
| 2        | 200           | 0.10           | 5          | 0.7149    | 0.8740   |
| 3        | 150           | 0.15           | 4          | 0.7182    | 0.8760   |
| 4        | 50            | 0.05           | 2          | 0.6051    | 0.8460   |

**Bộ siêu tham số đã chọn:** `n_estimators=150`, `learning_rate=0.15`, `max_depth=4`.

**Lý do:** Chọn vì F1 cao nhất (0.7182), vượt ngưỡng 0.65. Accuracy cao nhất (0.8780) không trùng F1 cao nhất, cho thấy accuracy không đáng tin khi mất cân bằng lớp. Từ run 1 sang run 3 (100/0.10→150/0.15) F1 tăng từ 0.7109 lên 0.7182. Run 2 (200/0.10/5) cho F1=0.7149, thấp hơn dù nhiều cây hơn, nhưng run này đồng thời đổi max_depth nên chỉ là gợi ý, chưa kết luận được nguyên nhân. Run 4 (50/0.05/2) F1 thấp nhất (0.6051) vì mô hình quá nhỏ.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Adult mất cân bằng: 24,8% > 50K, 75,2% ≤ 50K. Mô hình "luôn trả lời thấp" có accuracy=0,752 nhưng F1=0 (không phát hiện lớp dương), chứng tỏ accuracy không phản ánh hiệu năng khi mất cân bằng. F1 là trung bình Precision/Recall lớp dương, bắt buộc vừa bắt được thiểu số vừa chính xác. Không dùng average="weighted"/"macro" vì chúng làm loãng hiệu năng lớp dương.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn                                                  | Nguyên nhân                                                                                                              | Cách giải quyết                                     |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| ImportError: FallbackAsyncAdaptedQueuePool không tìm thấy | MLflow 2.13.0 với backend SQLite cần SQLAlchemy 2.0.x; phiên bản SQLAlchemy cài lúc đầu không tương thích nên import lỗi | Cài sqlalchemy==2.0.23 và ghim vào requirements.txt |
| `dvc pull` trên CI lỗi 403 (HeadObject) | Secret chứa AccessKeyId mới nhưng SecretAccessKey của key cũ, cặp key lệch | Tạo lại key và đẩy thẳng cặp đúng vào GitHub Secret |
| Release fail: `/healthz` không lên, `joblib.load` lỗi unpickle | VM cài scikit-learn mới hơn 1.4.2 dùng khi train | Cài lại đúng phiên bản ghim như requirements.txt rồi restart service |


---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

|                              | f1\_score | accuracy |
| ---------------------------- | --------- | -------- |
| Bước 2 (chỉ `train_batch1`)  | 0.7182    | 0.8760   |
| Bước 3 (thêm `train_batch2`) | 0.7297    | 0.8800   |


**Nhận xét:** F1 tăng nhẹ (0.7182→0.7297) nhưng chênh nhỏ, trong khoảng dao động do holdout 500 mẫu (~124 mẫu dương), nên chưa kết luận dữ liệu mới cải thiện mô hình. Hai nửa chia ngẫu nhiên từ cùng nguồn nên cùng phân phối; mô hình siêu tham số cố định đã học gần hết từ 22.361 mẫu đầu. Giá trị thật của Bước 3 là pipeline tự động chạy đúng từ commit dữ liệu đến API phục vụ mô hình mới.

