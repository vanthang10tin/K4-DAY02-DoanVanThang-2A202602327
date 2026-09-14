# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Doan-Van-Thang<br>
**MSSV:** 2A202602327<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp | Gán khi nhìn thấy | Không gán vào lớp này |
| ---: | --- | --- | --- |
| 0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
| 1 | `truck` (xe tải) | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối |
| 2 | `bus` (xe buýt) | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế | xe van nhỏ; xe tải; ô tô con |
| 3 | `van` (xe van) | thân hộp nhỏ, kín, dùng chở người hoặc hàng | thân xe buýt; khoang hàng tách biệt như xe tải |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
| --- | --- | --- |
| `visibility` (mức nhìn thấy) | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy |
| `boundary` (quan hệ mép ảnh) | `inside` (trong ảnh), `truncated` (bị cắt) | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại) | đánh dấu quyết định cần quay lại |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_022`, xe khách lớn di chuyển ở làn giữa khung hình.
- Dấu hiệu nhìn thấy: Thân xe dài, có hai khoang cửa sổ kính rộng kéo dài dọc thân xe, dải kính nóc và đèn trần xe chở khách, nhưng phần đầu xe hơi dốc nhẹ giống dòng xe van thương mại cỡ lớn.
- Quy tắc áp dụng: Thân xe khách dài, nhiều cửa sổ hoặc nhiều hàng ghế -> `bus`.
- Quyết định: Phân lớp `bus`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì?: Phóng ảnh lên 100% để quan sát kỹ cấu trúc chia khoang ghế và vị trí cửa lên xuống hành khách; nếu vẫn còn phân vân giữa van cỡ lớn và bus mini, tạm thời đặt `review_state = needs_review` và ghi rõ nghi vấn vào nhật ký quyết định trước khi chốt nhãn.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_038`, xe cứu hộ công vụ màu trắng ở tiền cảnh.
- Dấu hiệu nhìn thấy: Cabin xe tải nhỏ độc lập, phía sau là sàn ben mở gắn thiết bị cơ khí chuyên dụng cẩu kéo xe, không có thùng kín chở hàng kiểu van và không phải khoang khách của ô tô con.
- Quy tắc áp dụng: Phương tiện có thùng, ben, sàn chở hàng hoặc thiết bị công vụ rõ ràng -> `truck`.
- Quyết định: Phân lớp `truck`.
- If vẫn thiếu bằng chứng, bạn sẽ làm gì?: Kiểm tra điểm nối giữa cabin điều khiển và sàn công vụ phía sau; nếu kết cấu khung gầm và sàn tải tách biệt mang thiết bị kỹ thuật thì xếp vào `truck`.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008`, ô tô con màu trắng đi ở mép dưới cùng của khung hình.
- Dấu hiệu nhìn thấy khi phóng 100%: Cản trước và hai bánh trước đã vượt ra ngoài biên ảnh, phần nhìn thấy bao gồm kính lái, nóc xe, kính sau và toàn bộ đuôi xe sedan rất sắc nét.
- Giá trị `visibility`: `clear` (phần thân xe nằm trong ảnh không bị phương tiện nào che khuất).
- Giá trị `boundary`: `truncated` (vật thể bị mép dưới của ảnh cắt ngang).
- Trạng thái `review_state`: `confident`.
- Lý do: Mặc dù bị mép ảnh cắt mất một phần cản trước, nhưng diện tích nhìn thấy chiếm hơn 75% thân xe và các đặc trưng nhận dạng xe sedan rất rõ nét, đủ căn cứ phân lớp `car` một cách chắc chắn mà không cần suy đoán phần bị cắt.

## 6. Xác nhận tự kiểm tra

- [x] Đã rà đủ bốn ảnh.
- [x] Đã kiểm vật thể thiếu và trùng.
- [x] Đã kiểm lớp và hình học từng hộp.
- [x] Mỗi hộp có đủ ba thuộc tính.
- [x] Đã xử lý mọi hộp `needs_review`.
- [x] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [x] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi.
- [x] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [x] Số vật thể thực tế: 62 — 40–60 là mục tiêu khối lượng, không phải điểm cắt.
