# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Đậu Văn Thạch |
| MSSV | 2A202602592 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/thach-s/K4-L3-DAY21-DauVanThach-2A202602592-CI-CD-for-AI-Systems./ |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Cấu hình thứ ba được chọn vì có F1 cao nhất (0.7149) và vượt ngưỡng 0.65. Cấu hình thứ hai có accuracy cao nhất (0.8780) nhưng F1 thấp hơn, cho thấy accuracy chưa phản ánh tốt khả năng nhận diện lớp thu nhập cao. Cấu hình thứ nhất có learning rate thấp, ít cây và cây nông nên học chưa đủ. Tăng số cây giúp bù mức đóng góp của từng cây và độ sâu lớn hơn nắm bắt quan hệ phức tạp, dù accuracy không tăng.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Dữ liệu chỉ có khoảng 24,8% mẫu thuộc lớp thu nhập trên 50K. Vì vậy, mô hình luôn dự đoán “thu nhập thấp” vẫn đạt accuracy khoảng 0,752 nhưng F1 lớp dương bằng 0, tức không phát hiện trường hợp thu nhập cao nào. F1 cân bằng precision và recall của lớp dương, phản ánh cả dự đoán dương sai lẫn mẫu dương bị bỏ sót. Lab dùng `f1_score(y_eval, preds)` với lớp dương mặc định để quyết định triển khai. Không dùng weighted F1 vì lớp đa số có thể kéo điểm lên; macro F1 cũng không đo riêng lớp nghiệp vụ quan trọng.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Python cục bộ 3.13 khác Python 3.10 của CI | Thư viện khóa chưa hỗ trợ Python 3.13 | Dùng môi trường tương thích để kiểm thử |
| MLflow từ chối định dạng `skops` | Kiểu nội bộ của GradientBoosting không được tin cậy | Log mô hình bằng `cloudpickle` |
| Accuracy cao nhưng không qua quality gate | Lớp đa số chi phối accuracy | Chọn cấu hình theo F1 lớp dương |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Sau khi thêm `train_batch2`, F1 tăng 0.0205 và accuracy tăng 0.0080. Dữ liệu bổ sung giúp mô hình nhận diện lớp thu nhập cao tốt hơn, đồng thời cải thiện nhẹ độ chính xác tổng thể trên cùng tập đánh giá.
