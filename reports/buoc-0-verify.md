# BƯỚC 0: KIỂM CHỨNG MÔI TRƯỜNG

**Ngày kiểm tra:** 2026-10-07  
**Người kiểm tra:** Claude Code - Verification Agent

---

## 1. PHẦN MỀM CƠ BẢN

| Tiêu Chí | Kỳ Vọng | Kết Quả | Trạng Thái | Chứng Cớ |
|---|---|---|---|---|
| Python version | >= 3.10 | 3.11.15 | ✓ PASS | `python --version` → Python 3.11.15 |
| Git version | Có sẵn | 2.53.0 | ✓ PASS | `git --version` → git version 2.53.0 |
| Cloud CLI | gcloud hoặc aws hoặc az | aws-cli/2.33.26 | ✓ PASS | `aws --version` → aws-cli/2.33.26 Python/3.13.11 |

**Tóm tắt:** Tất cả phần mềm cơ bản đã cài đặt đầy đủ.

---

## 2. REPOSITORY GIT

| Tiêu Chí | Kỳ Vọng | Kết Quả | Trạng Thái | Chứng Cớ |
|---|---|---|---|---|
| Git remote được cấu hình | Có SSH URL | git@github.com:jaimesHub/K4-L3-DAY21-... | ✓ PASS | `git remote -v` → 2 remotes (fetch/push) |
| Nhánh hiện tại | main | main (up to date) | ✓ PASS | `git status` → "On branch main" |
| Repo visibility | public | PUBLIC | ✓ PASS | `gh repo view --json visibility` → {"visibility":"PUBLIC"} |
| Working tree | Clean hoặc có untracked | 2 untracked (.python-version, CHECKLIST.md) | ✓ PASS | `git status` → không có staging changes |

**Tóm tắt:** Repository GitHub đã cấu hình đúng, công khai, và sẵn sàng.

---

## 3. MÔI TRƯỜNG ẢO (.venv)

| Tiêu Chí | Kỳ Vọng | Kết Quả | Trạng Thái | Chứng Cớ |
|---|---|---|---|---|
| Thư mục .venv tồn tại | Có | Có tại .venv/ | ✓ PASS | `test -d .venv` → tồn tại |
| mlflow | Cài đặt | mlflow (2.13.0) | ✓ PASS | `python -c "import mlflow"` → success |
| scikit-learn | Cài đặt | sklearn (1.4.2) | ✓ PASS | `python -c "import sklearn"` → success |
| pandas | Cài đặt | pandas (2.2.2) | ✓ PASS | `python -c "import pandas"` → success |
| dvc | Cài đặt | dvc (3.50.1) | ✓ PASS | `python -c "import dvc"` → success |
| pytest | Cài đặt | pytest (8.2.0) | ✓ PASS | `python -c "import pytest"` → success |
| fastapi | Cài đặt | fastapi (0.111.0) | ✓ PASS | `python -c "import fastapi"` → success |
| uvicorn | Cài đặt | uvicorn (0.29.0) | ✓ PASS | `python -c "import uvicorn"` → success |
| joblib | Cài đặt | joblib (1.4.2) | ✓ PASS | `python -c "import joblib"` → success |
| PyYAML | Cài đặt | PyYAML (6.0.1) | ✓ PASS | `python -c "import yaml"` → success |

**Tóm tắt:** Tất cả 9 thư viện bắt buộc đã cài đặt trong .venv với phiên bản phù hợp.

---

## 4. DỮ LIỆU (DATA FILES)

### 4.1 Kiểm Tra Sự Tồn Tại

| File | Dung Lượng | Trạng Thái |
|---|---|---|
| data/train_batch1.csv | 550K | ✓ Tồn tại |
| data/holdout.csv | 12K | ✓ Tồn tại |
| data/train_batch2.csv | 550K | ✓ Tồn tại |

### 4.2 Kiểm Tra Số Dòng và Cột

| File | Số Dòng | Kỳ Vọng | Trạng Thái | Số Cột | Kỳ Vọng | Trạng Thái |
|---|---|---|---|---|---|---|
| train_batch1.csv | 22,361 | 22,361 | ✓ PASS | 11 | 11 | ✓ PASS |
| holdout.csv | 500 | 500 | ✓ PASS | 11 | 11 | ✓ PASS |
| train_batch2.csv | 22,361 | 22,361 | ✓ PASS | 11 | 11 | ✓ PASS |

### 4.3 Kiểm Tra Tỷ Lệ Target

| File | Positive (>50K) | Số Mẫu | Tỷ Lệ % | Kỳ Vọng | Trạng Thái |
|---|---|---|---|---|---|
| train_batch1.csv | 5,539 | 22,361 | 24.8% | 24.8% | ✓ PASS |
| holdout.csv | 124 | 500 | 24.8% | 24.8% | ✓ PASS |
| train_batch2.csv | 5,545 | 22,361 | 24.8% | 24.8% | ✓ PASS |

### 4.4 Cột Dữ Liệu

Tất cả 3 file đều có 11 cột giống hệt:
- age, workclass, education_num, marital_status, occupation, relationship, sex, capital_gain, capital_loss, hours_per_week, **target** (10 features + 1 target)

✓ **PASS**

### 4.5 .gitignore Check

```
data/train_batch1.csv   ✓ Trong .gitignore
data/holdout.csv        ✓ Trong .gitignore
data/train_batch2.csv   ✓ Trong .gitignore
```

---

## 5. TÓMSẮT KẾT QUẢ

| Thành Phần | Số Kiểm Tra | PASS | FAIL | Tỷ Lệ |
|---|---|---|---|---|
| Phần mềm cơ bản | 3 | 3 | 0 | 100% |
| Repository Git | 4 | 4 | 0 | 100% |
| Môi trường ảo (.venv) | 10 | 10 | 0 | 100% |
| Dữ liệu (Data Files) | 8 | 8 | 0 | 100% |
| **TỔNG** | **25** | **25** | **0** | **100%** |

---

## 6. KẾT LUẬN

✓ **BƯỚC 0 ĐÃ HOÀN THÀNH ĐẦY ĐỦ**

### Tóm tắt xác minh:

1. **Phần mềm cơ bản:** Python 3.11.15 (>= 3.10), Git 2.53.0, AWS CLI 2.33.26 — Sẵn sàng
2. **Git Repository:** Remote cấu hình đúng, repo PUBLIC, nhánh main clean — Sẵn sàng
3. **Môi trường ảo:** .venv tồn tại, tất cả 9 thư viện bắt buộc đã cài đặt (mlflow, sklearn, pandas, dvc, pytest, fastapi, uvicorn, joblib, pyyaml) — Sẵn sàng
4. **Dữ liệu:** 3 file CSV đầy đủ (train_batch1: 22.361, holdout: 500, train_batch2: 22.361 mẫu), 11 cột, tỷ lệ target=1 chính xác 24.8%, tất cả trong .gitignore — Sẵn sàng

### Khuyến nghị bước tiếp theo:

Có thể tiến hành **BƯỚC 1: THỰC NGHIỆM CỤC BỘ VỚI MLFLOW** mà không gặp cản trở môi trường.

---

**Xác nhận:** Tất cả kiểm tra đều vượt qua. Môi trường sẵn sàng cho các bước tiếp theo.
