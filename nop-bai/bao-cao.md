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
| Họ và tên | Đậu Văn Thạch |
| MSSV | 2A202602592 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/thach-s/K4-L3-DAY21-DauVanThach-2A202602592-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

<!-- Khoảng 120 - 150 từ. Điền kết quả thật từ MLflow UI ở Bước 1, tối thiểu 3 lần chạy. -->

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Cấu hình thứ ba được chọn vì đạt F1 cao nhất (0.7149) và vượt ngưỡng triển khai 0.65. Cấu hình thứ hai có accuracy cao nhất (0.8780) nhưng F1 thấp hơn, cho thấy accuracy không phản ánh đầy đủ khả năng nhận diện lớp thu nhập cao. Cấu hình thứ nhất kết hợp learning rate thấp, ít cây và cây nông nên học chưa đủ, chỉ đạt F1 0.6051. Việc tăng số cây giúp bù mức đóng góp của từng cây và mô hình sâu hơn nắm bắt được quan hệ phức tạp, dù accuracy không tăng tương ứng.

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

Dữ liệu chỉ có khoảng 24,8% mẫu thuộc lớp thu nhập trên 50K. Vì vậy, một mô hình luôn dự đoán “thu nhập thấp” vẫn đạt accuracy khoảng 0,752 nhưng F1 của lớp dương bằng 0, tức là không phát hiện được trường hợp thu nhập cao nào. F1 cân bằng precision và recall của lớp dương, nên phản ánh cả dự đoán dương sai lẫn các mẫu dương bị bỏ sót. Lab vì thế dùng `f1_score(y_eval, preds)` với lớp dương mặc định để quyết định triển khai. Không dùng weighted F1 vì lớp đa số có thể kéo kết quả lên; macro F1 cũng không trực tiếp đo riêng lớp nghiệp vụ quan trọng là thu nhập cao.

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
| Python cục bộ là 3.13, khác Python 3.10 của CI | Các phiên bản thư viện khóa trong đề chưa hỗ trợ Python 3.13 | Giữ phiên bản khóa cho CI và dùng môi trường tương thích riêng để kiểm thử cục bộ |
| MLflow mới từ chối định dạng `skops` cho mô hình cây | Kiểm tra an toàn không tin cậy kiểu nội bộ của GradientBoosting | Chỉ định rõ định dạng `cloudpickle` khi log mô hình |
| Accuracy cao nhưng cấu hình yếu không qua quality gate | Dữ liệu mất cân bằng khiến accuracy bị lớp đa số chi phối | So sánh theo F1 của lớp dương và chọn cấu hình đạt ngưỡng |

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
