# CSC4005 Lab 4 Report – CRNN for UrbanSound8K

## 1. Thông tin sinh viên

- Họ tên: Triệu Quốc Anh
- Mã sinh viên: 1671040002
- Lớp: KHMT 16-01
- Link GitHub repo: https://github.com/FIT-DNU-CS-16-01/csc4005-lab4-TrieuQuocAnh
- Link W&B project: https://wandb.ai/noioivedi-dainam-vietnam/csc4005-lab4-urbansound8k-crnn

## 2. Mục tiêu thí nghiệm

Mục tiêu Lab 4 là xây dựng và đánh giá mô hình CRNN trên tính năng log-mel spectrogram cho bài toán phân loại UrbanSound8K. Log-mel spectrogram được sử dụng vì nó giữ được cấu trúc tần số và thông tin thời gian của tín hiệu âm thanh, phù hợp cho việc học đặc trưng âm thanh. CRNN kết hợp khả năng trích xuất đặc trưng cục bộ của CNN với khả năng nắm bắt quan hệ thời gian của RNN, khác với 1D-CNN ở Lab 3 chỉ xử lý chuỗi thời gian một chiều. Kết quả sẽ được đánh giá bằng độ chính xác trên tập validation và test để so sánh hiệu quả với kiến trúc trước đó.

## 3. Cấu hình dữ liệu

| Thành phần | Giá trị |
|---|---|
| Dataset | UrbanSound8K |
| Số lớp | 10 |
| Train folds | 1–8 |
| Validation fold | 9 |
| Test fold | 10 |
| Feature | log-mel spectrogram |
| Sampling rate | 16 kHz |
| Duration | 4 giây |

## 4. Cấu hình mô hình

| Thành phần | Giá trị |
|---|---|
| Model | CRNN |
| CNN blocks | 3 blocks: 16 → 32 → 64 |
| RNN type | GRU |
| Hidden size | 96 |
| Dropout | 0.3 |
| Optimizer | AdamW |
| Learning rate | 0.001 |
| Batch size | 32 |
| Epochs | 25 |

## 5. Kết quả huấn luyện

Điền kết quả tốt nhất từ W&B hoặc `metrics.json`.

| Run | best_val_acc | test_acc | Ghi chú |
|---|---:|---:|---|
| logmel_crnn_gru_baseline |0.93371 |0.7491 | |
| extension_bilstm_crnn |0.65196 |0.70968 | |

## 6. Learning curves

Chèn hình `curves.png`.

Nhận xét:

- Mô hình không có dấu hiệu overfitting nặng. Train loss giảm đều từ ~2.00 xuống ~0.66, và validation loss cũng giảm từ ~1.92 xuống ~0.98.
- Validation loss dao động nhẹ quanh các epoch 13–18 và 22–25 nhưng vẫn giữ xu hướng giảm tổng thể, cho thấy quá trình huấn luyện tương đối ổn định.
- Early stopping không bắt buộc cho run này vì val loss vẫn tiếp tục cải thiện đến cuối, nhưng trong thực tế vẫn nên dùng để dừng trước khi loss ngừng giảm nếu chạy thêm nhiều epoch.

## 7. Confusion matrix

Chèn hình `confusion_matrix.png`.

Nhận xét:

- Lớp phân loại tốt: `gun_shot` (recall 100%), `jackhammer` (recall 90.6%), `air_conditioner`, `engine_idling`, `street_music` đều có hiệu quả cao.
- Lớp dễ bị nhầm: `siren` có recall thấp nhất (48.2%) và đôi khi bị nhầm với `children_playing`; `children_playing` cũng bị nhầm với `dog_bark`, `street_music` và `drilling`.
- Giải thích: tiếng `siren` và `children_playing` đều có biến động tần số và tiếng người/âm thanh môi trường xen kẽ, nên model dễ lẫn. Các cặp như `engine_idling`/`air_conditioner` hoặc `drilling`/`jackhammer` có thành phần tần số thấp giống nhau, nên nhầm lẫn là hợp lý.

## 8. So sánh với Lab 3 1D-CNN

| Tiêu chí | Lab 3: 1D-CNN | Lab 4: CRNN |
|---|---|---|
| Feature chính | log-mel | log-mel |
| Khả năng học pattern cục bộ | Có | Có |
| Khả năng học quan hệ thời gian | Hạn chế (1D) | Tốt hơn (CNN + RNN) |
| Test accuracy | 0.64 | 0.749 |
| Nhận xét | 1D-CNN chỉ học đặc trưng theo chiều thời gian một chiều, dễ bỏ sót mối liên hệ thời gian dài; độ chính xác thấp hơn. | CRNN trích đặc trưng time-frequency bằng CNN rồi học chuỗi thời gian bằng RNN, nên cải thiện accuracy và xử lý tốt hơn những lớp có diễn biến âm thanh theo thời gian. |

## 9. Kết luận

CRNN đã cải thiện so với 1D-CNN khi đạt test accuracy cao hơn và cho thấy khả năng học quan hệ thời gian tốt hơn trên log-mel spectrogram. Kết quả baseline GRU-CRNN khá ổn định với loss giảm đều và validation loss không tăng mạnh, cho thấy mô hình không bị overfit nặng. Run BiLSTM mở rộng cho thấy biến thể RNN khác có thể ảnh hưởng đến hiệu suất nhưng vẫn cần tuning thêm để ổn định. Nếu làm tiếp, em sẽ thử tăng mức regularization, điều chỉnh learning rate scheduler và mở rộng augmentation âm thanh. Ngoài ra, việc dùng attention hoặc ResNet cho phần CNN có thể giúp trích đặc trưng time-frequency tốt hơn.

## 10. Link minh chứng

- GitHub commit cuối:https://github.com/FIT-DNU-CS-16-01/csc4005-lab4-TrieuQuocAnh
- W&B run baseline:https://wandb.ai/noioivedi-dainam-vietnam/csc4005-lab4-urbansound8k-crnn/runs/zgvvh0yj
- W&B run mở rộng:https://wandb.ai/noioivedi-dainam-vietnam/csc4005-lab4-urbansound8k-crnn/runs/riru0dfg
