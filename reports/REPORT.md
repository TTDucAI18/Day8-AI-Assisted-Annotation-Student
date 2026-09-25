# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Trương Trọng Đức

Công cụ gán nhãn: CVAT local (Docker), xuất Ultralytics YOLO Detection 1.0

Các số liệu dưới đây lấy từ `reports/rounds_table.md`, `outputs/metrics_round*.json` và `outputs/round1_diff.md`. Nhãn tham chiếu test do mô hình tạo, chưa được rà thủ công; vì vậy các chỉ số đo mức khớp với bộ tham chiếu hiện có, không phải chân lý tuyệt đối.

## 1. Dữ liệu và cách chia tập

Pool và test được chia theo thời gian, có vùng đệm, để các ảnh rất gần nhau trong cùng video/camera ít có cơ hội rơi hai bên của phép chia. Nếu chia ngẫu nhiên, các khung hình gần như trùng nhau có thể xuất hiện cả trong train/pool lẫn test. Khi đó model được đánh giá trên cảnh và xe gần như đã thấy, làm kết quả có xu hướng lạc quan hơn và khả năng tổng quát hóa sang đoạn thời gian khác bị phóng đại. Vùng đệm giảm nguy cơ rò rỉ tương quan thời gian này.

## 2. Mô hình khởi đầu lạnh (cold start)

| Vòng | Model | Ảnh train | Box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |

`compare_round0.jpg` cho thấy model khởi đầu bỏ sót một số xe nhỏ/xa và xe ở vùng tối; đồng thời có một số dự đoán lệch hoặc không khớp box tham chiếu. Recall theo kích thước thấp nhất ở nhóm nhỏ (0.182), cao hơn ở nhóm trung bình (0.547) và lớn (0.561), phù hợp với việc xe xa chỉ còn vài điểm đèn khó phát hiện. Cần người rà lại các trường hợp tham chiếu mơ hồ như xe bị cắt ở mép ảnh hoặc vùng chỉ thấy đèn: model có thể đã phát hiện xe thật mà box tham chiếu bỏ sót, hoặc ngược lại box tham chiếu có thể khoanh nhầm phản chiếu/vệt sáng. Không thể quy mọi bất đồng là lỗi model.

## 3. Chiến lược chọn mẫu

Điểm `score = W_U·U + W_A·A + W_D·D` kết hợp ba tín hiệu: bất định dự đoán (`U`), mức mơ hồ/khó rà của box (`A`), và độ xa theo thời gian so với ảnh đã gán nhãn (`D`). Trọng số cân bằng giữa khả năng học từ ảnh khó, giá trị rà soát box mơ hồ và độ đa dạng thời gian. `MIN_GAP_S=2` loại bớt ứng viên nằm quá gần một frame đã chọn để giảm ảnh gần trùng và công rà lặp lại; đây là khoảng cách tối thiểu, không đảm bảo các frame cách hơn 2 giây khác nhau.

Trong `reports/SELECTION.md`, `frame_0182.jpg` (rank 1; U=0.9182, A=1.0000) có nhiều box mơ hồ; `frame_0369.jpg` (rank 2; U=0.9315, A=0.8889) là cảnh đông xe với bất định cao; `frame_0331.jpg` (rank 5; A=1.0000, 47 box dự đoán ban đầu) đáng rà vì số box/mức mơ hồ làm tăng nguy cơ trùng hoặc box giả. `frame_0372.jpg` (rank 6, 148.8 s) dù điểm 0.9101 không được chọn vì cách `frame_0369.jpg` chỉ 1.2 giây, nhỏ hơn ngưỡng gap. Điểm bất định chỉ giúp xếp thứ tự ưu tiên, không chứng minh ảnh sẽ cải thiện model: ảnh có thể chứa nhiễu, box không hữu ích hoặc thông tin trùng lặp.

## 4. Các vòng học chủ động (active learning)

| Vòng | Model | Ảnh train | Box train | AP50 | Δ AP50 so cold start | P@0.25 | R@0.25 | F1 | R small | R medium | R large |
| ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 0 | yolov8n cold start (COCO car+bus+truck) | 0 | 0 | 0.771 | — | 0.925 | 0.489 | 0.640 | 0.182 | 0.547 | 0.561 |
| 1 | yolov8n fine-tune vòng 1 | 12 | 334 | 0.492 | -0.280 | 1.000 | 0.057 | 0.108 | 0.000 | 0.068 | 0.073 |

Ở vòng 1, 12 ảnh có 169 box model đề xuất; bản đã rà có 334 box: 140 accepted, 14 edited, 15 deleted và 180 added. Tức nhiều xe bị pre-label bỏ sót; số liệu này mô tả việc sửa nhãn gợi ý, không phải chất lượng model sau train.

AP50 vòng 1 giảm 0.2797 (xấp xỉ 0.280) so với cold start. Ở cùng test, recall giảm ở cả ba nhóm: small 0.182→0.000, medium 0.547→0.068, large 0.561→0.073. Precision tại ngưỡng 0.25 tăng 0.925→1.000 nhưng recall chỉ còn 0.057; model vòng 1 dự đoán rất ít và bỏ sót nhiều. Trong `compare_round1.jpg`, `frame_0050.jpg` giảm từ TP=11, FP=2, FN=7 ở cold start xuống TP=1, FP=0, FN=17 sau fine-tune; đây là ví dụ suy giảm rõ, chưa xác định nguyên nhân chỉ từ ảnh so sánh.

Quan sát độc lập trong `BLIND_SCAN.md` được ghi trước khi xem pre-label: ước lượng khoảng 27 xe ở `frame_0182.jpg`, chú ý xe xa và xe tối/bị cắt ở mép. Log rà sau đó ghi xe con rõ ở làn giữa-trái được giữ (`accepted`); ở `frame_0369.jpg` thêm nhiều xe nhỏ bị bỏ sót (`added`); ở `frame_0331.jpg` xóa một box không khớp xe riêng biệt (`deleted`). Đây lần lượt là quan sát độc lập, quyết định sửa pre-label, và không phải bằng chứng trực tiếp về nguyên nhân metric giảm. Ước lượng đếm bằng mắt 27 so với 25 box cuối ở frame 0182 chỉ là tín hiệu nên đối chiếu lại, không đủ để kết luận chắc chắn còn thiếu hai xe.

Ca khó theo guideline là xe rất nhỏ ở xa hoặc bị cắt một phần: cần phân biệt thân xe nhìn thấy với đèn/ánh phản chiếu trên mặt đường; box chỉ ôm phần xe nhìn thấy, không khoanh riêng vệt sáng. Theo quy ước bài, xe rất nhỏ dưới khoảng 16 px có thể bị bỏ qua; cần áp dụng nhất quán và tránh gán nhầm ánh sáng thành `car`.

## 5. Kết luận và giới hạn

Với kết quả hiện tại, vòng 1 kém cold start theo AP50 và recall cả ba kích thước. Tôi dừng việc gán nhãn vòng tiếp theo để kiểm tra pipeline trước; làm thêm vòng ngay khi model gần như không phát hiện có thể tốn công mà chưa xử lý được nguyên nhân. Trước khi train lại, cần kiểm tra class mapping `car`/class id 0, ghép đúng ảnh với nhãn, chuẩn YOLO normalized `cx cy w h`, kích thước/box sau export, cấu hình fine-tune và confidence; đồng thời xem lại ví dụ dự đoán và nhãn train.

Nếu sau kiểm tra vẫn tiếp tục, hai ứng viên vòng 2 trong `outputs/selection_round2.csv` là:

- `frame_0009.jpg` (rank 3, score 0.7000, model hiện không dự đoán box nào): cần rà toàn ảnh để tìm khả năng bỏ sót toàn bộ xe, chi phí có thể cao; lựa chọn này bổ sung trường hợp model im lặng, nhưng nội dung xe phải được xác nhận bằng người.
- `frame_0071.jpg` (rank 1, score 0.7519; 6 box, 4 mơ hồ): chi phí rà vừa đến cao vì cần xác nhận từng box mơ hồ và tìm xe bị bỏ sót. Không nên chọn thêm `frame_0069.jpg` cùng lô vì chỉ cách 0.8 giây, có nguy cơ cảnh gần trùng.

Đây là ứng viên do thuật toán xếp hạng, chưa được gán nhãn/rà xác nhận; không phải khẳng định chắc chắn chúng chứa nhiều xe hay sẽ cải thiện model. Kết luận metric cũng bị giới hạn bởi test chỉ có 20 ảnh, quy tắc bỏ qua xe quá nhỏ và nhãn tham chiếu do model tạo chưa được người rà thủ công. Do đó, kết quả giảm là cảnh báo cần điều tra, chưa đủ để kết luận hiệu năng thực tế giảm tương ứng trên mọi cảnh.
