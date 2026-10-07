# Báo Cáo Xác Minh Ảnh 01-mlflow-ui.png

**Ngày kiểm tra:** 7 tháng 10, 2026  
**Thời gian sửa đổi file:** 12:39 (mới hơn 12:37 ✓)  
**Kích thước file:** 296 KB (< 1 MB ✓)  
**Định dạng:** PNG 2864x958 pixels

---

## Kiểm Tra Yêu Cầu

### Yêu cầu từ README.md

| Yêu Cầu | Trạng Thái | Ghi Chú |
|---------|-----------|---------|
| Ít nhất 3 lần chạy với siêu tham số khác nhau | ✓ PASS | Hiện 4 lần chạy |
| Cột `f1_score` rõ ràng | ✓ PASS | Hiển thị đầy đủ, có sắp xếp giảm dần |
| Cột `accuracy` rõ ràng | ✓ PASS | Hiển thị đầy đủ |
| Cột `n_estimators` | ✓ PASS | Hiển thị: 150, 200, 100, 50 |
| Cột `learning_rate` | ✓ PASS | Hiển thị: 0.15, 0.1, 0.1, 0.05 |
| Cột `max_depth` | ✓ PASS | Hiển thị: 4, 5, 3, 2 |
| Chữ đọc được và rõ ràng | ✓ PASS | Tất cả chữ đều rõ ràng, dễ đọc |

---

## So Sánh Dữ Liệu Thực Tế

### Bảng So Sánh Chi Tiết

| Run Name | n_est | lr | md | accuracy (Ảnh) | f1_score (Ảnh) | accuracy (Kỳ Vọng) | f1_score (Kỳ Vọng) | Khớp |
|----------|-------|----|----|-----------------|------------------|--------------------|--------------------|-----|
| incongruous-wasp-415 | 150 | 0.15 | 4 | 0.876 | 0.7181818 | 0.876 | 0.7182 | ✓ |
| delightful-doe-269 | 200 | 0.1 | 5 | 0.874 | 0.7149321 | 0.874 | 0.7149 | ✓ |
| industrious-gnu-850 | 100 | 0.1 | 3 | 0.878 | 0.7109004 | 0.878 | 0.7109 | ✓ |
| trusting-fish-836 | 50 | 0.05 | 2 | 0.846 | 0.6051282 | 0.846 | 0.6051 | ✓ |

**Kết quả:** Tất cả 4 lần chạy khớp chính xác với giá trị kỳ vọng.

---

## Kết Luận

**TRẠNG THÁI: ✓ PASS HOÀN TOÀN**

Ảnh 01-mlflow-ui.png đáp ứng đầy đủ tất cả yêu cầu:
- File được cập nhật mới (12:39)
- Kích thước hợp lệ (296 KB)
- Chứa 4 lần chạy với siêu tham số khác nhau
- Hiển thị đầy đủ các cột metrics và parameters cần thiết
- Dữ liệu hiển thị trong ảnh khớp chính xác với giá trị kỳ vọng
- Chất lượng ảnh tốt, chữ rõ ràng dễ đọc

Ảnh sẵn sàng sử dụng cho báo cáo bào-cao.md.
