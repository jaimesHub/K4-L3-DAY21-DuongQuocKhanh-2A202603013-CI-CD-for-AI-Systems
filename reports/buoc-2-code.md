# Bước 2 - Tóm Tắt Công Việc Code

## Mô Tả

Phần này tóm tắt những gì đã được hoàn thành để chuẩn bị Bước 2 (CI/CD với AWS). Tất cả code đã viết xong và kiểm tra, nhưng các bước cloud và triển khai vẫn cần người dùng thực hiện thủ công.

---

## 1. requirements.txt - Cập Nhật Cho AWS

**Thay Đổi:**
- Thay `dvc[gs]==3.50.1` bằng `dvc[s3]==3.50.1`
- Thay `google-cloud-storage==2.16.0` bằng `boto3==1.34.49`
- Thêm comment để rõ ràng về mục đích

**Kiểm Tra:**
```bash
pip install -r requirements.txt
pip check  # Kết quả: No broken requirements found
```

---

## 2. src/serve.py - AWS S3 + FastAPI

**Hoàn Thành:**
- Thay thế `google.cloud.storage` bằng `boto3`
- Hàm `download_model()`: Tải model từ S3 (`s3.download_file()`)
- Endpoint `GET /healthz`: Trả về `{"status": "ok"}`
- Endpoint `POST /score`: 
  - Kiểm tra đầu vào (phải có đúng 10 đặc trưng)
  - Gọi `model.predict()` và trả về `{"prediction": 0|1, "label": "thu_nhap_thap"|"thu_nhap_cao"}`
- Xử lý lỗi: Nếu model không tải được, server vẫn khởi động nhưng predictions thất bại

**Ghi Chú:**
- File này được thiết kế để chạy trên VM sau khi model được upload lên S3
- Chưa chạy thử trên VM thực (yêu cầu tài nguyên AWS)

---

## 3. tests/test_train.py - 3 Unit Tests

**Hoàn Thành:**

1. **test_train_returns_float**: Kiểm tra hàm `train()` trả về float trong [0, 1]
   - Tạo dữ liệu tạm: 200 mẫu, 160 train + 40 holdout
   - Gọi `train()` với siêu tham số nhỏ (n_estimators=10, learning_rate=0.1, max_depth=2)
   - Assert kết quả là float và nằm trong [0.0, 1.0]

2. **test_report_file_created**: Kiểm tra file `outputs/report.json` được tạo
   - Gọi `train()` và kiểm tra file tồn tại
   - Assert file chứa `f1_score` và `accuracy`

3. **test_model_file_created**: Kiểm tra file `models/model.joblib` được tạo
   - Gọi `train()` và kiểm tra file tồn tại

**Kết Quả Pytest:**
```
tests/test_train.py::test_train_returns_float PASSED
tests/test_train.py::test_report_file_created PASSED
tests/test_train.py::test_model_file_created PASSED
====== 3 passed in 23.68s ======
```

---

## 4. .github/workflows/cicd.yml - 4 Jobs CI/CD

**Hoàn Thành:**

### Job 1: Unit Test
- Chạy pytest trên thư mục `tests/`
- Trigger: Push lên main với thay đổi `.dvc`, `src/*.py`, hoặc `params.yaml`

### Job 2: Train
- Xác thực AWS: Parse secret `STORAGE_CREDENTIALS` (JSON), set `AWS_ACCESS_KEY_ID/SECRET` và `AWS_DEFAULT_REGION`
- Pull dữ liệu: `dvc pull data/train_batch1.csv.dvc data/holdout.csv.dvc`
- Huấn luyện: `python src/train.py`
- Đọc f1_score từ `outputs/report.json` thành output
- Upload model: `s3.upload_file("models/model.joblib", bucket, "artifacts/current/model.joblib")`
- Lưu report: `actions/upload-artifact`

### Job 3: Quality Gate
- Đọc f1 từ output của job Train
- Nếu f1 < 0.65: exit 1 (chặn deployment)
- Nếu f1 >= 0.65: in thông báo và tiếp tục

### Job 4: Release
- SSH vào VM qua appleboy/ssh-action
- Restart service: `sudo systemctl restart income-api`
- Chờ 5s rồi health check: `curl -sf http://localhost:8080/healthz`
- Nếu fail: exit 1

**Kiểm Tra YAML:**
```
YAML is valid
```

**Ghi Chú:**
- Workflow chưa chạy thực tế (cần push lên GitHub + setup AWS + VM + secrets)
- 4 jobs sẽ run tự động khi có push thỏa điều kiện trigger

---

## 5. reports/buoc-2-aws-huong-dan.md - AWS Setup Guide

**Nội Dung:**
- Chuẩn bị biến môi trường
- Tạo S3 bucket với public access block
- Tạo IAM user với policy tối thiểu (S3 GetObject/PutObject/ListBucket)
- Tạo access key
- Cấu hình DVC remote cho S3 (dvc remote add, dvc add, dvc push)
- Tạo EC2 instance (t2.micro, Ubuntu 22.04)
- SSH setup: cài Python, pip, dependencies
- Sao chép serve.py lên VM
- Tạo systemd service unit (income-api.service)
- Tạo SSH key pair cho GitHub Actions
- Thêm 5 GitHub Secrets: STORAGE_CREDENTIALS, ARTIFACT_BUCKET, SERVER_HOST, SERVER_USER, SERVER_SSH_KEY
- Test API endpoints: /healthz, /score
- Gỡ lỗi và dọn dẹp tài nguyên

---

## Quyết Định Thiết Kế

1. **Boto3 thay vì Botocore trực tiếp:** Boto3 là SDK chính thức của AWS, cung cấp client-level API dễ dùng

2. **S3 thay vì DynamoDB/RDS:** Phù hợp cho artifact storage (file model binary lớn)

3. **EC2 t2.micro (free tier):** Đủ cho inference trên dữ liệu nhỏ, không cần GPU

4. **Systemd service:** Tự động restart khi VM reboot, dễ quản lý

5. **GitHub Actions secrets:** Bảo mật AWS credentials, không hardcode trong code

6. **Quality gate f1 >= 0.65:** Ngưỡng chắt lọc mô hình có chất lượng (thay vì accuracy), phù hợp với phân bố lớp mất cân bằng (24.8% lớp dương)

---

## Công Việc Còn Lại (Cho Người Dùng)

1. **Thiết Lập AWS:**
   - Tạo S3 bucket, IAM user, access key
   - Cấu hình DVC remote, `dvc add/push` dữ liệu

2. **Tạo EC2 + Cấu Hình VM:**
   - Tạo key pair, security group, EC2 instance
   - SSH cài dependencies, sao chép serve.py
   - Tạo systemd service unit

3. **GitHub Setup:**
   - Tạo SSH key để GitHub Actions triển khai
   - Thêm 5 secrets vào repo

4. **Chạy Pipeline:**
   - Push code lên main
   - Theo dõi 4 jobs trong Actions tab
   - Start service trên VM
   - Test /healthz và /score endpoints

5. **Chụp Ảnh:**
   - 02-actions-buoc-2.png: 4 jobs xanh
   - 04-curl-api.png: kết quả curl
   - 05-cloud-storage.png: S3 bucket content

---

## Rủi Ro / Lưu Ý

1. **Credentials bị lộ:** 
   - Không commit `sa-key.json`, access key vào git
   - Dùng `.gitignore` và GitHub Secrets
   - Kiểm tra `git status` trước khi commit

2. **Chi Phí AWS:**
   - EC2 t2.micro free tier (750h/tháng)
   - S3 miễn phí cho 5GB/tháng
   - Dọn dẹp tài nguyên sau lab để tránh phát sinh chi phí

3. **DVC pull trong GitHub Actions:**
   - Cần AWS credentials được set trước
   - Kiểm tra `STORAGE_CREDENTIALS` format JSON chính xác

4. **SSH deploy từ GitHub Actions:**
   - Server SSH key phải không có passphrase
   - Security group phải mở cổng 22 (SSH)
   - authorized_keys phải chứa public key của GitHub Actions

5. **Model inference thất bại:**
   - Nếu model chưa upload lên S3, serve.py vẫn khởi động nhưng /score sẽ lỗi
   - Pipeline phải xanh hoàn toàn trước khi start service

---

## File Được Tạo/Sửa

| File | Trạng Thái | Ghi Chú |
|---|---|---|
| `requirements.txt` | ✓ Hoàn thành | Thay dvc[gs] → dvc[s3], google-cloud-storage → boto3 |
| `src/serve.py` | ✓ Hoàn thành | Đầy đủ /healthz + /score endpoints, boto3 S3 download |
| `tests/test_train.py` | ✓ Hoàn thành | 3 tests, all PASS |
| `.github/workflows/cicd.yml` | ✓ Hoàn thành | 4 jobs, AWS-specific steps |
| `reports/buoc-2-aws-huong-dan.md` | ✓ Hoàn thành | Hướng dẫn AWS từng bước |
| `reports/buoc-2-code.md` | ✓ Hoàn thành | Tài liệu này |

---

Kế tiếp: Người dùng chạy lệnh AWS/VM theo `reports/buoc-2-aws-huong-dan.md`, rồi push code để chạy pipeline.
