# Quét độc lập trước khi xem pre-label

Frame: frame_0182.jpg

Số xe nhìn thấy bằng mắt: khoảng 27 xe, gồm cả các xe nhỏ ở xa và các xe bị cắt một phần ở mép dưới ảnh.

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe:

1. Khu vực các làn xe ở xa, gần đường chân trời và dải phân cách phía trên-trung tâm ảnh: nhiều xe chỉ còn đèn hậu/đèn pha nhỏ, dễ bị bỏ sót hoặc box quá rộng.
2. Mép dưới bên phải và vùng gần dải phân cách bên phải: có xe tối hoặc bị cắt khỏi khung hình, dễ bị bỏ sót; đồng thời ánh đèn phản chiếu trên mặt đường dễ bị vẽ nhầm thành xe.

Chạy `python3 tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.
