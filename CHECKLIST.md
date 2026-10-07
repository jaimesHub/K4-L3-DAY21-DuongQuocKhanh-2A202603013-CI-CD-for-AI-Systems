# CHECKLIST LAB DAY 21 - CI/CD cho AI Systems

**Tổng điểm: 80 + Bonus 20 = 100**

---

## BƯỚC 0: CHUẨN BỊ MÔI TRƯỜNG

- [x] **Cài đặt phần mềm cơ bản** | Kiểm tra: `python --version` (3.10+), `git --version`, `gcloud/aws/az --version` (một trong ba) | Không có bằng chứng cần nộp | 0 điểm
- [x] **Tạo repo GitHub (public)** | Kiểm tra: Repo công khai, clone được | URL repo dùng để nộp bài | 0 điểm
- [x] **Tạo môi trường ảo và cài thư viện** | Chạy: `python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` | Không có lỗi trong quá trình install | 0 điểm
- [x] **Tải dữ liệu** | Chạy: `python prepare_data.py` | Kết quả: 3 file CSV, tỷ lệ lớp > 50K = 24.8% | 0 điểm

---

## BƯỚC 1: THỰC NGHIỆM CỤC BỘ VỚI MLFLOW

**Rubric: 24 điểm**

### MLflow Tracking (12 điểm)

- [x] **Cấu hình MLflow** | Chạy: `export MLFLOW_TRACKING_URI=sqlite:///mlflow.db` | File `mlflow.db` được tạo | ✓ mlflow.db tạo thành công
- [x] **Viết `params.yaml` với bộ tham số mặc định** | File tồn tại tại thư mục gốc: `n_estimators: 100`, `learning_rate: 0.1`, `max_depth: 3` | File đúng format YAML | ✓ File tồn tại, format YAML đúng
- [x] **Viết `src/train.py` hoàn chỉnh** | Chạy: `python src/train.py` không lỗi | File tồn tại, huấn luyện GradientBoostingClassifier, ghi kết quả vào MLflow | ✓ File hoàn chỉnh, chạy thành công
- [x] **Chạy ít nhất 3 lần thí nghiệm** | Chạy: Sửa `params.yaml` 2 lần và chạy lại `python src/train.py` | MLflow UI hiển thị ≥ 3 runs khác nhau | **12 điểm** ✓ 4 runs chạy thành công

### Độ Đo (Metrics) (8 điểm)

- [x] **Ghi F1-score và Accuracy** | Kiểm tra: Mỗi run trong MLflow có cột `f1_score` và `accuracy` | Cả hai metric đều được log | **8 điểm** ✓ Cả 4 runs đều log đủ metrics

### Phân Tích Kết Quả (4 điểm)

- [x] **Phân tích bộ siêu tham số tốt nhất** | Kiểm tra: `params.yaml` được cập nhật với bộ có f1_score cao nhất | f1_score ≥ 0.65, điền vào bảng trong `bao-cao.md` | **4 điểm** ✓ n_estimators=150, lr=0.15, depth=4, F1=0.7182

**Checkpoint:** MLflow UI hiển thị ≥ 3 runs với f1_score, accuracy, n_estimators, learning_rate, max_depth rõ ràng.  
⚠️ **Công việc còn lại cho người dùng**: Chạy `mlflow ui --backend-store-uri sqlite:///mlflow.db`, chụp ảnh → `01-mlflow-ui.png`

---

## BƯỚC 2: PIPELINE CI/CD TỰ ĐỘNG

**Rubric: 44 điểm**

### DVC và Cloud Storage (12 điểm)

- [x] **Tạo bucket trên cloud** | Chạy: `aws s3 mb s3://$BUCKET` (AWS), hoặc `gsutil mb` (GCP), hoặc `az storage container create` (Azure) | Bucket hiển thị trên Cloud Console | ✓ S3 bucket income-lab-667323977010 tồn tại (ảnh 05)
- [x] **Tạo credentials** | Chạy: `aws iam create-user` + `aws iam create-access-key` (AWS), hoặc tương đương GCP/Azure | File credentials (access key hoặc JSON) được lưu an toàn | 0 điểm (dùng GitHub Secrets thay vì lưu file)
- [x] **Cấu hình DVC** | Chạy: `dvc init`, `dvc remote add -d labstore s3://$BUCKET/dvc`, `dvc remote modify labstore region`, `dvc add data/*.csv` | File `.dvc/config` có remote URL đúng | ✓ DVC remote s3://income-lab-667323977010/dvc
- [x] **Push dữ liệu lên cloud** | Chạy: `dvc push` | S3/Cloud Storage Console hiển thị 3 file CSV hoặc dung lượng tương ứng trong `dvc/` | **12 điểm** ✓ dvc/ folder thấy rõ trên S3 (ảnh 05)

### Unit Tests (không tính điểm riêng, nhưng bắt buộc)

- [x] **Viết `tests/test_train.py`** | Chạy: `pytest tests/ -v` | 3 test đều PASS | ✓ 3 PASS (test_train_returns_float, test_report_file_created, test_model_file_created)

### CI/CD Pipeline (16 điểm)

- [x] **Viết `src/serve.py` hoàn chỉnh** | File tồn tại, có `download_model()`, endpoints `/healthz` và `/score` | ✓ Hoàn thành với boto3 S3 download, /healthz trả {"status":"ok"}, /score kiểm tra 10 features + dự đoán, đã chạy trên VM
- [x] **Tạo VM trên cloud** | Chạy: `aws ec2 run-instances ...` | VM hoạt động, có public IP | ✓ EC2 income-api đang chạy, IP 54.87.78.192 (ảnh 04)
- [x] **Cấu hình systemd service trên VM** | SSH vào VM, tạo `/etc/systemd/system/income-api.service` | Service đã enable và đang chạy | ✓ Service income-api hoạt động (ảnh 04)
- [x] **Tạo `.github/workflows/cicd.yml`** | File tồn tại ở `.github/workflows/cicd.yml` | 4 jobs: Unit Test → Train → Quality Gate → Release, triggers on data/**.dvc, src/**.py, params.yaml | ✓ 4 jobs hoàn chỉnh, AWS-specific (boto3 S3 auth, dvc pull, quality gate, SSH release)

### Quality Gate (4 điểm)

- [x] **Quality gate chặn khi f1_score < 0.65** | Kiểm tra: Bước Quality Gate trong cicd.yml so sánh `f1 >= 0.65` | Nếu f1 < 0.65, Release job bị skip | **4 điểm** ✓ cicd.yml có so sánh f1 >= 0.65
- [ ] **(Chưa thử thực tế)** Demonstrate Release bị chặn khi f1 < 0.65 | Ghi chú: chưa có bằng chứng run bị chặn (chỉ có run thành công)

### Triển Khai (Serving) (12 điểm)

- [x] **Push code lần đầu kích hoạt pipeline** | Chạy: `git add . && git commit && git push origin main` | GitHub Actions tab: 4 jobs chạy tới Green | ✓ Run 37657696450 thành công cả 4 jobs (ảnh 02)
- [x] **Start service trên VM** | Chạy: `aws ec2-instance-connect send-ssh-public-key ...` hoặc SSH vào VM, `sudo systemctl start income-api` | Service đang chạy | ✓ Service income-api chạy trên EC2 54.87.78.192
- [x] **API `/healthz` hoạt động** | Chạy: `curl http://VM_IP:8080/healthz` | Kết quả: `{"status": "ok"}` | ✓ curl 54.87.78.192:8080/healthz → {"status":"ok"} (ảnh 04)
- [x] **API `/score` trả về dự đoán đúng** | Chạy: `curl -X POST http://VM_IP:8080/score -H "Content-Type: application/json" -d '{"features": [...]}` | Kết quả: `{"prediction": 0 or 1, "label": "thu_nhap_thap" or "thu_nhap_cao"}` | **12 điểm** ✓ 2 POST requests → [60,2,...,45]→thu_nhap_thap, [28,2,...,45]→thu_nhap_cao (ảnh 04)

**Checkpoint:** 
- 4 jobs tất cả màu xanh. Chụp → `02-actions-buoc-2.png` ✓ PASS
- Lệnh curl thành công. Chụp → `04-curl-api.png` ✓ PASS
- Cloud Storage hiển thị `dvc/` và `artifacts/current/model.joblib`. Chụp → `05-cloud-storage.png` ⚠️ Chưa fix tên file (hiện có 05-cloud-storage_1.png và 05-cloud-storage_2.png)

**Ghi chú (7 tháng 10, 2026):** Các mục chính Bước 2 đã hoàn thành. Công việc còn tồn: (1) Hủy run kẹt 37651842288, (2) Gỡ s3-policy.json(.bak) khỏi repo, (3) Fix tên ảnh 05 (gộp thành một file hoặc đổi tên theo yêu cầu README), (4) Chưa commit thay đổi.

---

## BƯỚC 3: HUẤN LUYỆN LIÊN TỤC

**Rubric: 12 điểm**

- [ ] **Thêm dữ liệu mới** | Chạy: `python append_batch.py` | Kết quả: train_batch1.csv tăng từ 22.361 → 44.722 mẫu | 0 điểm
- [ ] **Cập nhật DVC** | Chạy: `dvc add data/train_batch1.csv` | File `data/train_batch1.csv.dvc` được cập nhật | 0 điểm
- [ ] **Commit dữ liệu vào git** | Chạy: `git add data/train_batch1.csv.dvc && git commit -m "data: ..."` | Commit message trong git log | 0 điểm
- [ ] **Push dữ liệu lên cloud trước git push** | Chạy: `dvc push` rồi `git push origin main` | Cloud Storage có dữ liệu mới | 0 điểm
- [ ] **Pipeline tự động được kích hoạt** | Kiểm tra: GitHub Actions tab, commit message đúng là data commit | 4 jobs chạy và tất cả pass | **12 điểm**

**Checkpoint:** Pipeline được kích hoạt bởi commit dữ liệu (không phải code commit). Chụp → `03-actions-buoc-3.png`

---

## NỘP BÀI (SUBMISSION)

**Rubric: 0 điểm (bắt buộc)**

- [ ] **Điền báo cáo vào `nop-bai/bao-cao.md`** | File có đủ 4 mục: (1) bộ siêu tham số, (2) giải thích F1, (3) khó khăn, (4) so sánh Bước 2 vs 3 | Không vượt quá 1 trang A4 (~ 450-550 từ) | 0 điểm
- [ ] **Nộp 5 ảnh chụp màn hình** | Chính xác tên file: `01-mlflow-ui.png`, `02-actions-buoc-2.png`, `03-actions-buoc-3.png`, `04-curl-api.png`, `05-cloud-storage.png` | Mỗi ảnh < 1 MB | 0 điểm
- [ ] **Commit tất cả lên GitHub** | Chạy: `git add nop-bai/ && git commit && git push` | Thư mục `nop-bai/` hiển thị đầy đủ trên GitHub | 0 điểm
- [ ] **Repo GitHub ở chế độ public** | Kiểm tra: Settings > Visibility > Public | Có thể truy cập repo mà không cần login | 0 điểm
- [ ] **Dán URL repo vào bài nộp trên vlearn.dev** | URL: `https://github.com/USERNAME/REPO_NAME` | Được xác nhận nhân viên chấm | 0 điểm

---

## BONUS: 5 THÁCH THỨC NÂNG CAO (Tối đa 20 điểm)

**Hoàn thành tất cả 5 thách thức = 20 điểm; một phần = tính theo tỷ lệ**

### Bonus 1: Tracking MLflow Từ Xa Với DagsHub (4 điểm)

- [ ] **Tạo tài khoản DagsHub** | Chạy: Kết nối repo GitHub trên dagshub.com | Tài khoản được tạo | 0 điểm
- [ ] **Thêm MLflow environment vào GitHub Secrets** | Kiểm tra: Settings > Secrets có `MLFLOW_TRACKING_URI` và `MLFLOW_TRACKING_PASSWORD` | 2 secrets được thêm | 0 điểm
- [ ] **Cập nhật `cicd.yml`** | File `.github/workflows/cicd.yml` có bước set environment trước khi chạy train | Train job ghi lên DagsHub | **4 điểm**

### Bonus 2: Điều Chỉnh Ngưỡng Quyết Định (4 điểm)

- [ ] **Quét ngưỡng từ 0.1 → 0.9** | Chạy: Trong `src/train.py`, dùng `predict_proba()` và quét threshold | `outputs/report.json` chứa threshold tối ưu và F1 tương ứng | 0 điểm
- [ ] **So sánh với ngưỡng mặc định 0.5** | Kiểm tra: Log hoặc report hiển thị F1(threshold_best) vs F1(0.5) | Chênh lệch được ghi lại | **4 điểm**

### Bonus 3: Báo Cáo Precision/Recall Tự Động (4 điểm)

- [ ] **Tính confusion matrix** | Thêm vào `src/train.py`: tính precision, recall từng lớp | `outputs/detail.txt` được tạo | 0 điểm
- [ ] **Upload artifact trong cicd.yml** | Step `Upload model` thêm `actions/upload-artifact` cho `detail.txt` | Artifact hiển thị trong Actions | **4 điểm**

### Bonus 4: Hoàn Trả Về Phiên Bản Trước (4 điểm)

- [ ] **Tải f1_score trước đó từ cloud** | Thêm bước trong cicd.yml: kéo `outputs/report.json` cũ từ cloud storage | File tồn tại hoặc bỏ qua nếu lần đầu | 0 điểm
- [ ] **So sánh F1 mới vs F1 cũ** | Quality Gate: chỉ deploy khi F1_new >= F1_old | Log ghi lại kết quả so sánh | **4 điểm**

### Bonus 5: Cảnh Báo Lệch Lạc Dữ Liệu (4 điểm)

- [ ] **Kiểm tra tỷ lệ lớp dương** | Thêm vào `src/train.py`: tính tỷ lệ target=1, cảnh báo nếu lệch > 5% so với 24.8% | `outputs/report.json` ghi tỷ lệ | 0 điểm
- [ ] **In cảnh báo rõ ràng** | Khi tỷ lệ lệch, log in WARNING hoặc ERROR | Log GitHub Actions hiển thị cảnh báo | **4 điểm**

---

## BẢNG TỔNG HỢP RUBRIC

| Hạng Mục | Tiêu Chí | Điểm |
|---|---|---|
| Bước 1 - MLflow tracking | ≥ 3 runs với siêu tham số khác nhau | 12 |
| Bước 1 - Độ đo | Cả f1_score và accuracy | 8 |
| Bước 1 - Phân tích | Chọn bộ tối ưu, giải thích F1 | 4 |
| Bước 2 - DVC | Remote cấu hình, dvc push thành công | 12 |
| Bước 2 - CI/CD | 4 jobs Pass | 16 |
| Bước 2 - Quality gate | Release chặn khi f1 < 0.65 | 4 |
| Bước 2 - Serving | /score endpoint trả về dự đoán đúng | 12 |
| Bước 3 - Tự động hóa | Commit dữ liệu kích hoạt pipeline | 12 |
| **Tổng Bắt Buộc** | | **80** |
| Bonus (tất cả 5 thách) | | **20** |
| **TỔNG CỘNG** | | **100** |

---

## 5 ẢNH CHỤP MÀN HÌNH CẦN NỘP

| # | Tên File | Nội Dung | Rubric | 
|---|---|---|---|
| 1 | `01-mlflow-ui.png` | MLflow UI: ≥ 3 runs, f1_score, accuracy, hyperparams rõ ràng | Bước 1: 20 điểm |
| 2 | `02-actions-buoc-2.png` | GitHub Actions: 4 jobs màu xanh (Bước 2) | Bước 2: 16 điểm |
| 3 | `03-actions-buoc-3.png` | GitHub Actions: Lần chạy kích hoạt bởi commit dữ liệu | Bước 3: 12 điểm |
| 4 | `04-curl-api.png` | Terminal: curl /healthz + /score, IP VM rõ ràng | Bước 2: 12 điểm |
| 5 | `05-cloud-storage.png` | Cloud Storage: dvc/ + artifacts/current/model.joblib | Bước 2: 12 điểm |

---

## NỘI DUNG BÁO CÁO (nop-bai/bao-cao.md)

1. **Bộ siêu tham số chọn**: Bảng 3 runs, lý do chọn bộ này (dựa f1, không accuracy)
2. **Giải thích F1 vs Accuracy**: Phân bố lớp, accuracy "luôn trả lời thấp" = 0.752, vì sao F1 hơn
3. **Khó khăn**: 2-3 vấn đề thực tế gặp + cách giải quyết
4. **So sánh Bước 2 vs 3**: Bảng f1/accuracy, nhận xét (F1 có tăng không, tại sao)
5. **Bonus** (nếu làm): Danh sách thách thức hoàn thành

**Giới hạn:** ≤ 1 trang A4 (~ 450-550 từ)

---

## GHI CHÚ

- ✅ Tất cả checkout (`- [ ]`) đánh dấu khi hoàn thành
- ✅ Giữ repo GitHub public để người chấm xem được
- ✅ Không commit `sa-key.json` và GitHub Secrets (đã có trong `.gitignore`)
- ✅ Ảnh < 1 MB, định dạng PNG/JPG, không che thông tin cần chấm
- ✅ Xóa toàn bộ comment hướng dẫn trong `bao-cao.md` trước khi nộp
