# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Trần Ngọc Khánh |
| MSSV | 2A202602923 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/trankhanh6162/K4-L3-DAY21-TranNgocKhanh-2A202602923-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần chạy 3 có F1 cao nhất và vượt quality gate 0.65 nên được chọn dù accuracy thấp hơn lần 2. Điều này cho thấy accuracy cao nhất không đồng nghĩa khả năng nhận diện lớp thu nhập cao tốt nhất. Cấu hình đầu có learning rate thấp, ít cây và cây nông nên F1 chỉ đạt 0.6051. Khi giữ learning rate 0.1, tăng số cây và độ sâu giúp F1 tăng rõ rệt. Mức tăng từ lần 2 sang lần 3 nhỏ, thể hiện lợi ích biên giảm khi mô hình phức tạp hơn.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Chỉ khoảng 24,8% mẫu thuộc lớp thu nhập trên 50K nên dữ liệu mất cân bằng. Mô hình luôn dự đoán “thu nhập thấp” vẫn đạt accuracy khoảng 0,752 nhưng không phát hiện trường hợp thu nhập cao nào, tương ứng F1 lớp dương bằng 0. F1 kết hợp precision và recall, phản ánh khả năng tìm đúng lớp dương và hạn chế dự đoán dương sai. Lab tính `f1_score(y_eval, preds)` cho lớp dương. Không dùng `average="weighted"` vì lớp đa số kéo điểm lên; cũng không dùng macro vì ngưỡng 0.65 được thiết kế riêng cho lớp dương.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow lỗi khi mở SQLite store | SQLAlchemy 2.1 bỏ API MLflow 2.13 đang dùng | Khóa dependency `sqlalchemy<2.1` để local và CI tái tạo cùng môi trường |
| Không tạo được service-account JSON key | Project áp chính sách `iam.disableServiceAccountKeyCreation` | Dùng Workload Identity Federation cho GitHub và attached service account cho VM |
| Push ban đầu không kích hoạt Actions | Actions của repository fork chưa được bật lại hoàn toàn | Reset quyền Actions ở cấp repository và xác nhận run được kích hoạt bằng commit `.dvc` |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Sau khi tăng dữ liệu huấn luyện từ 22.361 lên 44.722 mẫu, F1 tăng khoảng 0,0205 và accuracy tăng 0,0080. Batch mới cùng phân phối nhưng cung cấp thêm ví dụ để mô hình khái quát tốt hơn; mức tăng vừa phải, không phải bằng chứng rằng thêm dữ liệu luôn bảo đảm cải thiện.
