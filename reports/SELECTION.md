# Vì sao chọn lô này?

Nếu chỉ có ngân sách rà năm ảnh, tôi ưu tiên năm frame đứng đầu trong 50 ứng viên đầu của `outputs/selection_round1.csv`:

| Ưu tiên | Frame | Rank | Điểm | Thời điểm | Lý do |
| ---: | --- | ---: | ---: | ---: | --- |
| 1 | `frame_0182.jpg` | 1 | 0.9591 | 72.8 s | U=0.9182 và A=1.0000; nhiều box mơ hồ nên có cơ hội tìm cả lỗi box lẫn xe bị bỏ sót. |
| 2 | `frame_0369.jpg` | 2 | 0.9324 | 147.6 s | U=0.9315, A=0.8889, 43 box; cảnh đông xe và model phân vân. |
| 3 | `frame_0380.jpg` | 3 | 0.9170 | 152.0 s | U=0.9340, A=0.8333; điểm bất định cao và nằm ở đoạn thời gian khác. |
| 4 | `frame_0326.jpg` | 4 | 0.9155 | 130.4 s | U=0.9310, A=0.8333; thêm cảnh ở một đoạn khác để tránh chỉ rà một vùng thời gian. |
| 5 | `frame_0331.jpg` | 5 | 0.9154 | 132.4 s | A=1.0000 và 47 box; lô dày box khiến việc kiểm trùng và box giả đáng ưu tiên. |

Ba frame trong lô minh họa tiêu chí chọn là `frame_0182.jpg` (rank 1; 18 box mơ hồ), `frame_0369.jpg` (rank 2; U=0.9315) và `frame_0331.jpg` (rank 5; A=1.0000, 47 box). Các điểm và số box lấy từ CSV; `outputs/selection_round1.jpg` cho thấy nội dung ảnh được chọn.

`frame_0372.jpg` có điểm cao 0.9101 nhưng không nằm trong lô. Nó ở giây 148.8, chỉ cách `frame_0369.jpg` 1.2 giây; `MIN_GAP_S=2.0` hạn chế chọn hai frame quá gần nhau trong cùng lô vì nền camera cố định khiến chúng dễ gần trùng và tốn công rà lặp lại. Quy tắc này là giới hạn tối thiểu; các frame cách nhau hơn 2 giây vẫn có thể giống nhau.

Điểm bất định giúp xếp thứ tự ưu tiên, không chứng minh frame sẽ cải thiện mô hình. Việc chọn còn phụ thuộc vào xe thực sự có trong ảnh, chất lượng pre-label, mức trùng cảnh và thời gian người rà phải bỏ ra.
