# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

<!--
HƯỚNG DẪN - đọc rồi XÓA TOÀN BỘ các khối chú thích này sau khi điền xong:

  - Giới hạn: KHÔNG QUÁ 1 TRANG A4, tương đương khoảng 450 - 550 từ nội dung.
  - Chỉ điền vào các chỗ ___ và các ô trong bảng. Không thêm mục mới.
  - Viết bằng câu hoàn chỉnh, không gạch đầu dòng cụt lủn.
  - Kiểm tra độ dài sau khi đã xóa hết chú thích:
        wc -w nop-bai/bao-cao.md
    và xem trước bản in bằng cách mở file trên GitHub rồi Ctrl+P / Cmd+P.
-->

| | |
|---|---|
| Họ và tên | ___ |
| MSSV | ___ |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/___/___ |
| Ngày nộp | ___ |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

<!-- Khoảng 120 - 150 từ. Điền kết quả thật từ MLflow UI ở Bước 1, tối thiểu 3 lần chạy. -->

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.10 | 3 | 0.7109 | 0.8780 |
| 2 | 200 | 0.10 | 5 | 0.7149 | 0.8740 |
| 3 | 150 | 0.15 | 4 | 0.7182 | 0.8760 |
| 4 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |

**Bộ siêu tham số đã chọn:** `n_estimators=150`, `learning_rate=0.15`, `max_depth=4`.

**Lý do:** Bộ siêu tham số n_estimators=150, learning_rate=0.15, max_depth=4 được chọn vì đạt F1 Score cao nhất (0.7182), vượt quá ngưỡng yêu cầu 0.65. Lần chạy có accuracy cao nhất (0.8780 ở 100/0.10/3) không trùng với lần có F1 cao nhất, cho thấy accuracy không phải chỉ số đáng tin cậy khi dữ liệu mất cân bằng lớp. Trên dữ liệu này, tăng n_estimators từ 100 lên 150 kèm tăng learning_rate từ 0.10 lên 0.15 cải thiện F1 từ 0.7109 lên 0.7182. Tuy nhiên, nếu chỉ tăng n_estimators đến 200 mà giữ learning_rate ở 0.10, F1 chỉ đạt 0.7149, thấp hơn bộ đã chọn, cho thấy cân bằng giữa hai tham số này là quan trọng. Bộ 50/0.05/2 có F1 thấp nhất (0.6051) vì mô hình quá nhỏ và tốc độ học quá chậm.

<!--
Trả lời trong phần Lý do:
  - Vì sao bộ này tốt hơn các bộ còn lại (dựa trên f1_score, không phải accuracy)?
  - Lần chạy có accuracy cao nhất có trùng với lần có f1_score cao nhất không?
    Nếu không, điều đó nói lên điều gì?
  - Bạn quan sát thấy đánh đổi nào giữa n_estimators và learning_rate?
-->

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

<!-- Khoảng 120 - 150 từ. -->

Tập dữ liệu Adult có phân bố lớp mất cân bằng: chỉ 24,8% mẫu có thu nhập > 50K, còn 75,2% có thu nhập ≤ 50K. Với phân bố như vậy, một mô hình đơn giản "luôn trả lời thu nhập thấp" sẽ đạt accuracy = 0,752, trông rất cao nhưng thực chất nó không bao giờ phát hiện được lớp dương (F1 = 0). Accuracy bị lớp đa số chi phối, khiến nó không phản ánh thực chất hiệu năng của mô hình trên bài toán này. Ngược lại, F1 Score là trung bình điều hòa của Precision và Recall của lớp dương, bắt buộc mô hình phải vừa bắt được lớp thiểu số (recall cao) vừa dự đoán chính xác (precision cao). Do đó, F1 đảm bảo mô hình thực sự học được bài toán. Không dùng average="weighted" hay average="macro" là vì hai phương pháp này sẽ làm loãng hoặc che đi hiệu năng thực của lớp dương bằng cách cộng hưởng tính toán với lớp đa số, trái với mục tiêu đặt ngưỡng trên lớp thiểu số.

<!--
Cần nêu được:
  - Phân bố lớp của tập dữ liệu (tỷ lệ lớp thu nhập > 50K) và hệ quả của nó.
  - Accuracy của một mô hình luôn trả lời "thu nhập thấp" là bao nhiêu, vì sao con số
    đó gây hiểu nhầm.
  - F1 của lớp dương đo điều gì mà accuracy không đo được.
  - Vì sao KHÔNG dùng average="weighted" hay average="macro" khi gọi f1_score.
-->

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

<!-- Nêu 2 - 3 khó khăn thật, mỗi ô một câu ngắn. -->

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| ImportError: FallbackAsyncAdaptedQueuePool không tìm thấy | MLflow 2.13.0 với backend SQLite cần SQLAlchemy 2.0.x; phiên bản SQLAlchemy cài lúc đầu không tương thích nên import lỗi | Cài sqlalchemy==2.0.23 và ghim vào requirements.txt |
| ___ | ___ | ___ |
| ___ | ___ | ___ |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

<!-- Lấy số liệu từ bảng ở mục 3.6 của tasks/buoc-3.md. -->

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | ___ | ___ |
| Bước 3 (thêm `train_batch2`) | ___ | ___ |

**Nhận xét:** ___

<!--
Một câu trả lời trung thực kiểu "f1 giảm 0,01 vì dữ liệu mới cùng phân phối, không mang
thêm thông tin mới" được đánh giá cao hơn kết luận sai rằng thêm dữ liệu luôn tốt hơn.
-->

---

## 5. Phần Bonus Đã Thực Hiện (nếu có)

<!-- Xóa cả mục 5 nếu không làm bonus. Mỗi bonus tối đa 1 dòng. -->

- [ ] Bonus 1 - Tracking MLflow từ xa với DagsHub: ___
- [ ] Bonus 2 - Điều chỉnh ngưỡng quyết định: ___
- [ ] Bonus 3 - Báo cáo precision / recall tự động: ___
- [ ] Bonus 4 - Hoàn trả về phiên bản trước: ___
- [ ] Bonus 5 - Cảnh báo lệch lạc dữ liệu: ___
