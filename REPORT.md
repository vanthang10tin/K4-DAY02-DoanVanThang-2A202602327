# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Đoàn Văn Thắng<br>
**MSSV:** 2A202602327<br>
**Hình thức:** Cá nhân<br>
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_022`, `drive_033`, `drive_038`, `drive_008`
- Số vật thể thực tế: 62
- Mã SHA-256 của gói YOLO của bạn: `4f7fff1f94942d766c5cdaf8a757013792248abe222a04e53ecfa48e3102e288`
- Mã SHA-256 của gói CVAT gốc của bạn: `c2e269b12158a1c380595c290cc5e1523eeb24d8a3077d5a85979b272e8a2f94`
- Nguồn đối chiếu: bạn cùng cặp hoặc bộ tham chiếu do người hướng dẫn thực hành cấp: Bộ tham chiếu do người hướng dẫn thực hành cấp (Lab Coach)
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: 15:53

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu:

Hai gói dữ liệu của tôi gồm bản Ultralytics YOLO và bản CVAT for images 1.1 đã được xuất trực tiếp từ cùng một công việc gán nhãn CVAT và được lưu về máy tính với mã băm SHA-256 cố định lần lượt là `4f7fff1f94942d766c5cdaf8a757013792248abe222a04e53ecfa48e3102e288`và `c2e269b12158a1c380595c290cc5e1523eeb24d8a3077d5a85979b272e8a2f94`. Các tệp xuất gốc không hề bị can thiệp hay sửa đổi thủ công, đảm bảo tính độc lập tuyệt đối trước khi tiến hành đối chiếu.

## 2. Quyết định phân lớp

| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| --- | --- | --- | --- |
| `drive_022` (xe buýt trung tâm) | `bus` | Thân xe khách dài, hai khoang cửa sổ lớn, dải kính rộng dọc thân | Thân xe khách dài, nhiều cửa sổ hoặc nhiều hàng ghế -> `bus` |
| `drive_033` (xe tải thùng trắng) | `truck` | Thùng kín chở hàng tách biệt phía sau cabin, chiều cao thùng vượt nóc cabin | Có thùng, ben, sàn chở hàng tách biệt rõ ràng -> `truck` |
| `drive_038` (xe cứu hộ công vụ trắng) | `truck` | Cabin xe tải nhỏ gắn thiết bị cẩu/kéo chuyên dụng phía sau sàn | Phương tiện có thiết bị công vụ, máy móc chuyên dụng trên sàn -> `truck` |
| `drive_008` (xe bán tải đen ngã tư) | `car` | Xe bán tải pickup 4 chỗ nhỏ, di chuyển trong luồng xe con cá nhân | Xe bán tải dùng cho mục đích cá nhân/xe con -> `car` |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Lớp (`class`) định danh bản chất chủng loại của phương tiện (ví dụ: `car` là ô tô con, `bus` là xe buýt), đây là thuộc tính bản thể cố định của đối tượng không thay đổi theo góc máy hay thời điểm quan sát. Ngược lại, thuộc tính (`attribute`) mô tả tình trạng quan sát tức thời của đối tượng trong khung hình cụ thể. Ví dụ: một chiếc ô tô con (`class = car`) khi di chuyển qua ngã tư có thể bị xe tải phía trước che khuất một phần thân sau (`visibility = occluded`), hoặc khi chạy sát mép dưới bức ảnh thì bị mép cắt mất cản trước (`boundary = truncated`). Chiếc xe đó vẫn luôn thuộc lớp `car`, nhưng thuộc tính `visibility` và `boundary` của nó sẽ thay đổi tùy theo góc nhìn và vị trí. Định dạng YOLO chỉ lưu `class_id`, không giữ lại các thuộc tính này (cần xuất `CVAT for images 1.1` để bảo toàn).

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| --- | --- | --- | --- |
| Hộp xe ở góc xa trên `drive_033` | hình học | Phóng ảnh 100% rà soát biên hộp | Thu hẹp hộp ôm sát mép vỏ xe, loại bỏ phần bóng đổ dài trên mặt đường; quy tắc: vẽ sát phần phương tiện nhìn thấy, không chứa nền thừa |
| Xe van màu bạc trên `drive_022` | lớp | So sánh đối chiếu tiêu chí thân hộp một khối | Chuyển từ `car` sang `van`; quy tắc: thân hộp kín một khối, không có cốp sedan/khoang hàng tách biệt -> `van` |
| Xe buýt bị che trên `drive_008` | thuộc tính | Lọc danh sách nhãn kiểm tra thuộc tính | Đổi `visibility` từ `clear` sang `occluded` do phần đầu xe bị xe tải chở đá che khuất; quy tắc: bị che một phần thân -> `occluded` |

- Số hộp `needs_review` trước và sau khi kiểm: Trước khi kiểm: 4 hộp. Sau khi rà soát và phóng 100%: 0 hộp (toàn bộ chuyển sang `confident`).
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: Trên ảnh `drive_033`, có phương tiện ở hậu cảnh rất xa trên cầu vượt bị mờ nhòe điểm ảnh. Quyết định: Phóng 100%, ghi chú lý do không đủ bằng chứng hình học để phân biệt giữa SUV cỡ lớn và van nhỏ, đánh dấu `review_state = confident` chỉ sau khi xác nhận các đường nét nóc xe, đồng thời trao đổi với Lab Coach để thống nhất ngưỡng kích thước tối thiểu nên bỏ qua để tránh gây nhiễu dữ liệu.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `1 0.457445 0.582305 0.123266 0.303234`
- Tên lớp và tọa độ điểm ảnh `xyxy`:
  - Tên lớp: `truck` (class_id = 1)
  - Tọa độ điểm ảnh `xyxy`: `[253.3, 275.6, 332.2, 469.7]` (tính trên kích thước ảnh 640x640: $x_1 = (0.457445 - 0.123266/2) \times 640 = 253.3$, $y_1 = (0.582305 - 0.303234/2) \times 640 = 275.6$, $x_2 = (0.457445 + 0.123266/2) \times 640 = 332.2$, $y_2 = (0.582305 + 0.303234/2) \times 640 = 469.7$).
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Định dạng nhãn YOLO chỉ là một quy ước cú pháp số học thuần túy (5 giá trị số thực nằm trong khoảng hợp lệ [0, 1] và class_id nằm trong khoảng [0, 3]). Trình kiểm tra cú pháp chỉ xác nhận được các con số có đúng định dạng hay không, chứ không thể hiểu được ngữ nghĩa thị giác:
1. Sai lớp: Nhãn ghi class_id = 1 (`truck`) nhưng thực tế đối tượng trong ảnh là xe buýt lớn (`bus`). Dòng nhãn vẫn hợp lệ về cú pháp nhưng sai hoàn toàn về mặt phân loại.
2. Sai phạm vi: Hộp vẽ gộp hai chiếc ô tô con đi sát nhau vào làm một hộp, hoặc vẽ trùm lên người đi xe máy/biển báo. Dòng tọa độ vẫn là 5 con số hợp lệ nhưng vi phạm nghiêm trọng quy tắc mỗi phương tiện một hộp độc lập.
3. Sai hình học: Hộp vẽ quá lỏng lẻo bao trọn cả bóng đổ mặt đường, hoặc vẽ đoán mò phần thân xe bị khuất sau xe khác. Các con số x_center, y_center, width, height vẫn hợp lệ nhưng hình học hộp bị sai lệch so với ranh giới nhìn thấy thực tế của vật thể.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`:
  Trên ảnh kiểm tra `drive_008`, ở ngưỡng tin cậy mặc định `conf = 0.25`, mô hình YOLO11n sau 5 epoch huấn luyện không hiển thị bounding box nào (hoặc các dự đoán thô có confidence cực thấp dưới 0.25), bỏ sót toàn bộ các phương tiện rất rõ ràng trong bối cảnh ngã tư như chiếc xe tải ben chở đá màu đỏ lớn, xe buýt vàng-xanh và đoàn xe con màu trắng/đen đang dừng chờ đèn đỏ.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào?
  Kết quả này gợi ý rằng tập huấn luyện hiện tại có kích thước quá nhỏ (chỉ 3 ảnh với 47 hộp), số lượng epoch quá ít (5 epoch, early stopping kích hoạt) và việc đóng băng 10 tầng đầu khiến mô hình chưa kịp thích nghi với các đặc trưng phương tiện trong tập dữ liệu. Ngoài ra, cần kiểm tra lại độ đồng nhất trong việc gán nhãn giữa 3 ảnh train và 1 ảnh val (đặc biệt là mật độ và kích thước các hộp ở các khoảng cách khác nhau).
- Minh chứng nào có thể bác bỏ nhận định của bạn?
  Khi hạ ngưỡng tin cậy `conf` xuống mức 0.05 - 0.10 hoặc cho mô hình chạy thêm nhiều epoch mà không freeze backbone, mô hình bắt đầu đưa ra các hộp bao quanh các xe tải và xe buýt lớn. Điều này chứng minh mô hình đã học được một số đặc trưng cơ bản nhưng điểm confidence chưa đủ cao do thiếu bước học, chứ không phải quy tắc gán nhãn bị sai hoàn toàn.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?
  Bốn bức ảnh chụp từ góc camera tĩnh hoàn toàn không đại diện cho phân phối thực tế của hệ thống giám sát giao thông (vốn bao gồm nhiều điều kiện thời tiết, ban ngày/ban đêm, góc chụp cao/thấp, mật độ phương tiện và các chủng loại xe phong phú từ nhiều vùng miền). Huấn luyện trên 3 ảnh và thử nghiệm trên 1 ảnh chỉ mang tính chất kiểm thử kỹ thuật quy trình (sanity check), các chỉ số mAP hay loss thu được không có giá trị thống kê và tuyệt đối không thể dùng để đánh giá năng lực thực tế của mô hình hay chất lượng của người gán nhãn.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 42
- IoU trung bình và trung vị:
  - IoU trung bình (`mean_iou`): 0.717747 (71.77%)
  - IoU trung vị (`median_iou`): 0.752101 (75.21%)
- Mức đồng thuận lớp (`class_agreement`): 0.642857 (64.29%)
- Số hộp phía bạn không ghép được (`unmatched_mine`): 20
- Số hộp phía đối chiếu không ghép được (`unmatched_comparison`): 8
- Một điểm khác biệt cụ thể:
  Trên ảnh `drive_022`, với chiếc xe khách lớn đi ở làn giữa (mine row 1 ghép với comparison row 2, đạt IoU hình học rất cao 0.911808):
  - Phía tôi gán nhãn là: `bus`
  - Phía đối chiếu gán nhãn là: `van`
  Nguyên nhân: Chiếc xe này có thiết kế lai giữa thân van khối lớn và thân xe khách mini. Tôi phân loại là `bus` vì xe có dải cửa kính dài cho hành khách, trong khi phía đối chiếu coi là `van` vì đầu xe liền khối.
  Bên cạnh đó, phía tôi gán thêm 20 hộp phương tiện ở hậu cảnh xa trên `drive_033` và `drive_038` mà phía đối chiếu đã bỏ qua do quá nhỏ.
- Quy tắc hoặc hành động sửa phát sinh:
  Cần bổ sung tiêu chí cụ thể vào Phiếu quy tắc (`GUIDELINE_MINI_SHEET.md`):
  1. Quy định rõ tiêu chí phân định `bus` và `van`: Xe chở khách từ 16 chỗ trở lên hoặc có trên 3 hàng ghế/cửa sổ kính dọc thân xe thì tính là `bus`; xe thân hộp ngắn dưới 16 chỗ thì phân vào `van`.
  2. Bổ sung ngưỡng kích thước tối thiểu: Các phương tiện ở hậu cảnh xa có kích thước cạnh nhỏ hơn 15 pixel hoặc không thể phân biệt rõ đèn/kính/bánh xe thì không gán nhãn nhằm tránh gây nhiễu cho mô hình.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?
  Mức đồng thuận (Agreement/IoU) chỉ phản ánh mức độ trùng khớp trong việc diễn giải và áp dụng quy tắc giữa hai người gán (hoặc giữa người gán và bộ tham chiếu), chứ không đại diện cho chân lý thực tế (Ground Truth tuyệt đối). Nếu cả hai phía cùng hiểu sai quy tắc (ví dụ: cùng đoán mò phần thân xe bị che khuất, cùng vẽ trùm bóng đổ xe trên đường, hoặc cùng nhầm lẫn giữa xe bán tải và xe tải nhỏ), thì độ đồng thuận lớp và IoU vẫn có thể đạt mức rất cao (thậm chí 90-100%), trong khi tất cả các nhãn đó đều bị sai so với thực tế vật lý.

## 7. Kiểm tra kho GitHub cá nhân

- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

- Minh chứng mạnh nhất: Hai gói xuất YOLO và CVAT gốc của tôi đạt độ nhất quán tuyệt đối về số lượng (62 hộp trên cả 4 ảnh) và hình học, bảo toàn đầy đủ cả 3 thuộc tính `visibility`, `boundary`, `review_state`, đồng thời bước đối chiếu phân tích rõ ràng nguyên nhân sai lệch phân lớp (IoU trung vị đạt 75.21%, chỉ ra cụ thể tranh chấp lớp bus/van và ngưỡng kích thước vật thể ở xa).
- Câu hỏi còn lại cho Lab Coach: Đối với các phương tiện công vụ có khung gầm xe tải nhỏ nhưng gắn trang thiết bị chuyên dùng (như xe kéo cứu hộ giao thông trên ảnh `drive_038`), trong các bài toán công nghiệp thực tế, ta nên gán vào lớp `truck` hay nên tạo một lớp chuyên biệt riêng (special vehicle) để tránh làm suy giảm độ chính xác nhận diện của các xe tải thương mại thông thường?
