# Báo Cáo Xác Minh Ảnh Bước 2

**Ngày kiểm tra:** 8 tháng 10, 2026

---

## Kết Quả Xác Minh Từng Ảnh

| Ảnh | Yêu Cầu | Kích Thước | Kết Quả | Ghi Chú |
|-----|---------|-----------|--------|---------|
| 01-mlflow-ui.png | MLflow UI: 3+ lần chạy, cột f1_score + accuracy + siêu tham số | 296 KB | ✓ PASS | Hiển thị 4 lần chạy, sắp xếp theo f1_score, thấy rõ accuracy, f1_score, learning_rate, max_depth, n_estimators |
| 02-actions-buoc-2.png | GitHub Actions: 4 jobs (Unit Test, Train, Quality Gate, Release) màu xanh | 269 KB | ✓ PASS | Run #5, commit 2d34d71, cả 4 jobs thành công (tất cả màu xanh), Status: Success |
| 04-curl-api.png | Terminal: /healthz → {"status":"ok"} + /score POST ≥1 mẫu, thấy IP VM | 68 KB | ✓ PASS | IP VM 54.87.78.192 rõ ràng, /healthz trả {"status":"ok"}, 2 POST /score: [60,2,5,2,4,0,1,0,0,45] → thu_nhap_thap, [28,2,14,2,11,0,1,0,0,45] → thu_nhap_cao |
| 05-cloud-storage_1.png | S3 Console: thấy bucket name + dvc/ + artifacts/ | 323 KB | ✓ PASS | Bucket income-lab-667323977010 rõ ràng, thấy dvc/ folder |
| 05-cloud-storage_2.png | S3 Console: artifacts/current/model.joblib | 316 KB | ✓ PASS | Đường dẫn artifacts/ > current/ > model.joblib (483.4 KB) |

---

## Vấn Đề Cần Lưu Ý

### ⚠️ Tên File Ảnh Cloud Storage

README yêu cầu **một file duy nhất** có tên `05-cloud-storage.png`, nhưng hiện tại đã nộp **hai file** riêng biệt:
- `05-cloud-storage_1.png`: hiển thị thư mục gốc bucket (artifacts/, dvc/)
- `05-cloud-storage_2.png`: hiển thị artifacts/current/ với model.joblib

**Khuyến nghị giải quyết (chọn một):**
1. **Gộp thành một ảnh**: Chụp lại một view S3 duy nhất thấy được cả hai đường dẫn (ví dụ: breadcrumb trail hoặc một cửa sổ rộng hơn), lưu lại thành file tên chính xác `05-cloud-storage.png`
2. **Đổi tên file**: Nếu cần nộp riêng hai ảnh, hãy đổi tên thành `05a-cloud-storage-dvc.png` và `05b-cloud-storage-model.png` (theo hướng dẫn trong README)

---

## Kết Luận

✓ **Tất cả ảnh đáp ứng yêu cầu nội dung.**

Các ảnh không lộ thông tin nhạy cảm (access key, secret, token, mật khẩu).

**Việc cần làm:**
1. Quyết định cách xử lý 05-cloud-storage_1.png và 05-cloud-storage_2.png (gộp hoặc đổi tên).
2. Cập nhật tên file để khớp yêu cầu trong README.
