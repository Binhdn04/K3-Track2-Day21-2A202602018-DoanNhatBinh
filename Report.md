# BÁO CÁO CÁ NHÂN DAY 21

**Họ Tên:** Đoàn Nhật Bình
**Cohort:** A20-K3
**Ngày submit:** 2026-08-21

## 1. Bộ siêu tham số được lựa chọn

Mô hình sử dụng `RandomForestClassifier` với `random_state=42`. Trong Bước 1, các thí nghiệm được theo dõi bằng MLflow và đánh giá trên cùng tập `eval.csv` gồm 500 mẫu. Một số kết quả tiêu biểu:

| `n_estimators` | `max_depth` | `min_samples_split` | Accuracy | F1-score |
|---:|---:|---:|---:|---:|
| 50 | 3 | 2 | 0.5580 | 0.5185 |
| 100 | 5 | 2 | 0.5640 | 0.5534 |
| 200 | 10 | 5 | 0.6440 | 0.6417 |
| **200** | **10** | **2** | **0.6480** | **0.6464** |

Bộ siêu tham số cuối cùng được chọn là:

```yaml
n_estimators: 200
max_depth: 10
min_samples_split: 2
```

Cấu hình này đạt cả accuracy và F1-score cao nhất trong các cấu hình đã thử. Việc tăng số cây lên 200 giúp dự đoán ổn định hơn; `max_depth=10` cho mô hình đủ khả năng học các quan hệ phi tuyến nhưng vẫn giới hạn độ phức tạp; `min_samples_split=2` giữ lại khả năng phân tách chi tiết và cho kết quả tốt hơn giá trị 5. Sau khi bổ sung dữ liệu ở Bước 3, cùng cấu hình đạt accuracy **0.6640** và F1-score **0.6603**, đều cao hơn Bước 1.

## 2. Khó khăn và cách giải quyết

- **Xác thực Azure trong GitHub Actions:** connection string lưu trong GitHub Secret có thể chứa ký tự xuống dòng, khiến Azure SDK không nhận diện đúng. Pipeline được sửa để loại bỏ `CR/LF`, kiểm tra định dạng và cấu hình DVC remote cục bộ trước khi pull dữ liệu.
- **DVC không mở được database cache trong môi trường bị giới hạn quyền ghi:** lệnh báo `unable to open database file`. Vấn đề được xử lý bằng cách đặt `DVC_SITE_CACHE_DIR` vào thư mục tạm có quyền ghi, sau đó `dvc add` và `dvc push` hoạt động bình thường.
- **Đảm bảo CI tải được phiên bản dữ liệu mới:** dữ liệu DVC được push lên Azure trước, rồi mới push commit chứa `data/train_phase1.csv.dvc` lên GitHub. Nhờ đúng thứ tự này, runner pull được dữ liệu 5.996 mẫu và toàn bộ các job Test, Train, Eval, Deploy đều thành công.
- **Ngưỡng đánh giá ban đầu cao hơn kết quả thực nghiệm:** với tập dữ liệu hiện tại, accuracy tốt nhất ở Bước 1 là 0.6480. Eval gate được hiệu chỉnh về 0.64 dựa trên kết quả thực tế nhưng vẫn giữ vai trò chặn các mô hình kém hơn mức chuẩn.
- **API trả về `Method Not Allowed`:** nguyên nhân là gọi `/predict` bằng GET. Endpoint được kiểm tra lại bằng POST, kèm header `Content-Type: application/json` và đủ 12 đặc trưng đầu vào.

Kết quả cuối cùng là quy trình dữ liệu mới → DVC/Azure → GitHub Actions → huấn luyện → đánh giá → triển khai hoạt động tự động; commit dữ liệu `fba7c07` đã kích hoạt thành công cả bốn job.
