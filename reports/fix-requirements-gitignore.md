# Báo cáo: Sửa requirements.txt và .gitignore

**Ngày:** 2026-10-07

## Tóm tắt
Đã thêm `sqlalchemy` vào `requirements.txt` và `mlruns/` vào `.gitignore` để sửa các phụ thuộc bị thiếu và quản lý file cục bộ của MLflow.

---

## 1. Thêm sqlalchemy vào requirements.txt

### Vấn đề
- `mlflow==2.13.0` với SQLite backend yêu cầu `sqlalchemy` nhưng package này không có trong `requirements.txt`
- Môi trường local đã cài thủ công `sqlalchemy==2.0.23`
- Thiếu dependency này có thể gây lỗi khi cài đặt lại môi trường

### Giải pháp thực hiện
1. Kiểm tra phiên bản thực tế: `.venv/bin/pip show sqlalchemy` → **Version: 2.0.23**
2. Thêm dòng `sqlalchemy==2.0.23` vào `requirements.txt` (sau `mlflow==2.13.0`)

### Bằng chứng
```bash
$ pip check
No broken requirements found.
```
✓ Không có xung đột giữa mlflow/sqlalchemy sau khi thêm dependency

---

## 2. Thêm mlruns/ vào .gitignore

### Vấn đề
- Thư mục `mlruns/` đang untracked ngoài `.gitignore`
- Chứa artifacts từ các MLflow runs cục bộ
  - `mlruns/0/` chứa 4 run subdirectories: `15016...`, `3e0da...`, `a679a...`, `b7ac6...`
  - Đây là output tạm thời của MLflow tracking (tương tự `mlflow.db`, `mlartifacts/`)

### Giải pháp thực hiện
- Thêm `mlruns/` vào `.gitignore` (dòng 2, sau `mlflow.db`) để tránh commit artifacts cục bộ

### Lý do
- `mlruns/` là output tạm thời từ việc chạy training cục bộ
- Không cần commit vào repository vì mỗi lần chạy sẽ tạo artifacts mới
- Giữ repository sạch và tránh conflict khi làm việc nhóm

---

## 3. Khuyến nghị về .python-version

### Tình trạng hiện tại
- File `.python-version` untracked
- Nội dung: `3.11.15` (phiên bản Python được sử dụng)

### Khuyến nghị
**Nên commit** `.python-version` nếu:
- Dự án sử dụng `pyenv` để quản lý phiên bản Python
- Muốn đảm bảo tất cả team members sử dụng cùng phiên bản Python
- Tránh lỗi về compatibility giữa môi trường khác nhau

**Quyết định:** Do không chắc chắn về policy của team, khuyến nghị **NỀN GIỮ** file này (không xoá, không ignore) để team lead quyết định.

---

## Kết quả
```bash
$ git status --short
M .gitignore          # Thêm mlruns/
 M params.yaml       # (không thay đổi bởi fix này)
 M requirements.txt   # Thêm sqlalchemy==2.0.23
 M src/train.py      # (không thay đổi bởi fix này)
?? .python-version   # Chờ quyết định
?? CHECKLIST.md      # (khác phạm vi fix này)
?? reports/          # (này là thư mục chứa báo cáo này)
```

---

## Kiểm tra
- ✓ sqlalchemy==2.0.23 được thêm vào requirements.txt
- ✓ `pip check` không báo xung đột
- ✓ mlruns/ được thêm vào .gitignore
- ✓ mlruns/ không còn xuất hiện trong `git status`
