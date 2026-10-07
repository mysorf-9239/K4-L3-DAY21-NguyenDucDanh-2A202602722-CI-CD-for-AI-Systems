# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Đức Danh |
| MSSV | 2A202602722 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/mysorf-9239/K4-L3-DAY21-NguyenDucDanh-2A202602722-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần 3 đạt F1 cao nhất và vượt ngưỡng 0.65. Lần 1 có accuracy cao hơn một chút nhưng F1 thấp hơn, cho thấy accuracy không phản ánh tốt khả năng nhận diện lớp thu nhập cao. Bộ cây sâu hơn với nhiều estimator cải thiện F1 trên holdout.

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Chỉ 24,8% dữ liệu thuộc lớp thu nhập trên 50.000 USD. Một mô hình luôn dự đoán “thu nhập thấp” vẫn đạt accuracy khoảng 75,2%, nhưng F1 của lớp dương bằng 0 vì không phát hiện được trường hợp thu nhập cao nào. F1 kết hợp precision và recall của lớp dương nên phù hợp hơn để quyết định có triển khai mô hình hay không. Bài lab dùng `f1_score` mặc định cho nhãn dương, không dùng `average="macro"` hoặc `average="weighted"`, để quality gate phản ánh đúng lớp cần phát hiện.

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow lỗi khi khởi chạy | SQLAlchemy 2.1 và setuptools mới không tương thích với MLflow 2.13 | Khóa `sqlalchemy<2.1` và `setuptools<81` trong requirements. |
| Không tạo được khóa JSON cho service account | Chính sách GCP chặn service account key | Dùng GitHub OIDC/Workload Identity Federation; gắn service account riêng vào VM. |
| DVC không ghi được cache mặc định trên macOS | Cache hệ thống nằm ngoài vùng ghi | Cấu hình cache local trong `/private/tmp`. |

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (22.361 mẫu) | 0.7149 | 0.8740 |
| Bước 3 (44.722 mẫu) | 0.7354 | 0.8820 |

**Nhận xét:** Khi thêm batch dữ liệu cùng nguồn, F1 tăng 0.0205 và accuracy tăng 0.0080 trên holdout cố định. Kết quả này cho thấy lần chạy cụ thể cải thiện nhẹ; điều quan trọng của Bước 3 là commit con trỏ DVC kích hoạt lại pipeline và cập nhật model trên VM.
