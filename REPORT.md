
# Báo cáo — Ngày 2: phát hiện vật thể

**Họ và tên:** Võ Quốc Dinh
**MSSV:** 2A202602318
**Hình thức:** cá nhân
**Mã cặp:** SOLO

## 1. Bài độc lập và nguồn dữ liệu

- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: `drive_008`, `drive_022`, `drive_033`, `drive_038`
- Số vật thể thực tế: 81 (`drive_008`: 25, `drive_022`: 5, `drive_033`: 20, `drive_038`: 31)
- Mã SHA-256 của gói YOLO của bạn: `d4a7fb4b4d8bf922b333b05aa18a3d68c91f91b3a00cc98e23e9513cc2a687ce`
- Mã SHA-256 của gói CVAT gốc của bạn: `f4052c9972dd2b6a6778ba63d6f72d9c220e1b87965f6fd06b9fe2f0df0e42a1`
- Nguồn đối chiếu: bộ tham chiếu do người hướng dẫn thực hành cấp
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Nếu làm cá nhân, ghi mã lần phát và thời điểm nhận bộ tham chiếu: `day2-reference-4img-v1` (theo
  `release-manifest.json` trong gói), tệp tải về lúc 12:44 ngày 14/09/2026.

Giải thích vì sao bài của bạn vẫn độc lập trước khi đối chiếu: Bằng chứng độc lập nằm trong chính số liệu đối chiếu: bài của tôi có 81 hộp, còn bộ tham chiếu có 50 hộp. Trong
48 cặp hộp ghép được, cả 13 cặp thuộc nhóm `truck/bus/van` đều khác lớp với tham chiếu, và không cặp nào trùng lớp.
Nếu tôi chép hoặc chỉnh theo bộ tham chiếu, các lớp này phải trùng. Hai gói YOLO và CVAT gốc được xuất từ cùng một
trạng thái công việc: kiểm tra chéo ghép đủ 81/81 hộp, IoU nhỏ nhất giữa hai định dạng là 0,99995.

## 2. Quyết định phân lớp

Mã vật thể dưới đây là số dòng trong tệp nhãn YOLO (`rN`, trùng cột `mine_row` của `comparison_iou.csv`); số `#`
là mã hộp trong CVAT ghi ở phiếu quy tắc.

| Ảnh/vật thể          | Lớp      | Dấu hiệu nhìn thấy                                                                                                       | Quy tắc áp dụng                                                                                            |
| ----------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `drive_022` r4        | `bus`   | xe khách khớp nối hai toa, thân dài, nhiều cửa sổ liên tiếp, biển số tuyến trên kính trước                  | `bus`: "thân xe khách dài, nhiều cửa sổ hoặc hàng ghế"                                             |
| `drive_008` r3        | `truck` | xe ben thùng đỏ chở đầy đất, cabin tách rời với thùng, nhiều trục bánh                                        | `truck`: "thùng, ben, sàn hàng" rõ ràng                                                                |
| `drive_038` r18       | `truck` | xe cứu hộ có cần cẩu và sàn kéo phía sau, cabin riêng                                                              | `truck`: "thiết bị công vụ rõ ràng"; không phải `van` vì không phải thân hộp kín một khối |
| `drive_033` r18 (#39) | `truck` | cabin trước tách với thùng hàng hộp cao và dài hơn cabin, có khe nối cabin–thùng                               | `truck`; loại `van` vì có "khoang hàng tách biệt như xe tải" (Tình huống B)                     |
| `drive_038` r9 (#75)  | `van`   | thân kín liền cabin, mui cao, chỉ 3–4 ô cửa sổ bên và một cửa trượt, thân ngắn hơn hẳn xe buýt cùng ảnh | `van`: "thân hộp nhỏ, kín"; không đủ dấu hiệu `bus` (Tình huống A)                             |
| `drive_008` r5        | `van`   | xe trắng thân hộp một khối, mũi ngắn, kính lớn, không có thùng hay khoang hàng riêng                           | `van`: "thân hộp nhỏ, kín, dùng chở người hoặc hàng"                                              |
| `drive_008` r24 (#13) | `car`   | chỉ thấy dải bánh xe và thân thấp sát mặt đường ở góc phải dưới                                             | `car` theo dáng thân thấp; hộp chỉ vẽ phần nhìn thấy (Tình huống C)                              |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:

Hộp `drive_038` r9 có lớp `van` vì thấy thân kín một khối và cửa trượt. Góc dưới thân xe bị thùng xe tải xanh che
nên `visibility=occluded`, nhưng phần nhìn thấy vẫn đủ căn cứ nên `review_state=confident`. Lớp trả lời câu hỏi
"đây là loại xe gì", còn thuộc tính ghi mức bằng chứng nhìn thấy và độ chắc chắn của quyết định. Ngược lại,
`drive_008` r24 cùng là `car` nhưng `visibility=unclear`, `boundary=truncated`. Trong gói CVAT gốc có 56 `clear`,
18 `occluded`, 7 `unclear`; 64 `inside`, 17 `truncated`; 81 `confident`. Gói YOLO không lưu các giá trị này.

## 3. Tự kiểm tra và sửa nhãn

| Trước khi sửa                                                                                                             | Loại lỗi   | Cách phát hiện                                                                                      | Sau khi sửa và quy tắc                                                                                                                                                             |
| ---------------------------------------------------------------------------------------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `drive_008`: có hộp cho hai xe nhỏ ở mép trên (x ≈ 483–513, y 0–25) và xe ở đỉnh ảnh (x ≈ 205–215, y 0–8) | phạm vi     | phóng 100%: xe bị cắt, quá nhỏ và mờ, không phân biệt được`car`/`van`                 | xóa hộp; quy tắc "quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán", ghi vào nhật ký                                                           |
| `drive_008`: có hộp cho vật màu đỏ sát mép phải (x ≈ 623–640, y 259–345)                                       | phạm vi     | phóng 100% chỉ thấy một dải tối, không có dấu hiệu phương tiện                            | xóa hộp, ghi lý do vào nhật ký quyết định                                                                                                                                    |
| `drive_008`: hộp cho xe mờ trong hàng xe phía trên (x ≈ 380–397, y 3–20)                                           | hình học   | rà vật thể trùng: hộp dính liền với xe#26 phía trước, không tách được thành xe riêng | xóa để tránh gán trùng; quy tắc "mỗi phương tiện là một hộp"                                                                                                            |
| `drive_033`: hộp cho xe mờ phía trước taxi #41 (x ≈ 353–373, y 105–128)                                            | hình học   | rà vật thể trùng: hình nhòe, không tách rõ khỏi#41                                           | xóa để tránh gán trùng                                                                                                                                                          |
| `drive_008` r24 (#13): `review_state=needs_review`                                                                       | thuộc tính | lọc các hộp`needs_review` khi tự kiểm tra                                                       | xem lại ở 100%: bị mép ảnh cắt chứ không bị che, nên`boundary=truncated`, `visibility=unclear`; dáng thân thấp đủ căn cứ cho `car`, chuyển sang `confident` |

- Số hộp `needs_review` trước và sau khi kiểm: số trước (ít nhất 1, là hộp #13) → 0 sau khi kiểm
- Một quyết định chưa đủ bằng chứng và cách bạn xin hỗ trợ: các xe rất xa ở đầu đường `drive_033` không được gán vì không phân biệt được lớp. Tôi ghi tọa độ vào nhật ký quyết định
  và hỏi Lab Coach nên đặt ngưỡng kích thước tối thiểu nào để cả lớp xử lý nhất quán.

## 4. Một dòng nhãn YOLO

- Dòng `class x_center y_center width height`: `1 0.942969 0.457813 0.082812 0.109375` (dòng 1 của
  `drive_022.txt`)
- Tên lớp và tọa độ điểm ảnh `xyxy`: lớp 1 = `truck`; với ảnh 640 × 640:
  `x1 = (0.942969 − 0.082812/2) × 640 = 577.0`, `y1 = (0.457813 − 0.109375/2) × 640 = 258.0`,
  `x2 = (0.942969 + 0.082812/2) × 640 = 630.0`, `y2 = (0.457813 + 0.109375/2) × 640 = 328.0`
  → `[577.0, 258.0, 630.0, 328.0]`. Đây là xe tải nhỏ thùng hộp trắng bên phải ảnh.
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?

Bước kiểm tra chỉ xác nhận dòng có đúng năm số, mã lớp nằm trong 0–3, tâm nằm trong [0, 1] và hộp không vượt
biên ảnh. Nó không nhìn nội dung ảnh. Ví dụ ngay trong bài này: bộ tham chiếu vượt qua đủ các bước kiểm định dạng,
nhưng xe buýt khớp nối `drive_022` r4 lại mang mã lớp `van` (xem mục 6). Tương tự, một dòng hợp lệ vẫn có thể là
xe máy bị gán nhầm (sai phạm vi), hoặc hộp bao cả phần bị che (sai hình học). Chỉ người đối chiếu ảnh với quy tắc
mới phát hiện được các lỗi này.

## 5. Huấn luyện và dự đoán thử

- Ba mã ảnh huấn luyện: `drive_022`, `drive_033`, `drive_038`
- Mã ảnh thẩm định: `drive_008`
- Mô tả một dự đoán trong `detect_result.jpg`: ảnh không có hộp dự đoán nào ở ngưỡng tin cậy 0,25. Khi thử hạ ngưỡng
  xuống 0,01 để chẩn đoán, mô hình vẫn không đưa ra hộp nào, trong khi nhãn của tôi ở `drive_008` có 25 vật thể
  (18 `car`, 1 `truck`, 3 `bus`, 3 `van`). Chỉ số lúc thẩm định: precision 0,0025, recall 0,1667, mAP50 0,0083,
  mAP50-95 0,0042. Lần chạy: YOLO11n (`0ebbc80d…44ee1`), Ultralytics 8.4.145, 8 vòng lặp, hạt giống 42, thiết bị
  CPU, 27,89 giây.
- Dự đoán đó gợi ý cần kiểm lại quy tắc hoặc dữ liệu nào? Kết quả rỗng trước hết là tín hiệu về lượng dữ liệu và
  cấu hình, chưa phải về nhãn: đầu phát hiện được khởi tạo lại cho 4 lớp, 10 lớp đầu bị đóng băng, và chỉ học 8 vòng
  trên 3 ảnh. Về dữ liệu, ba lớp hiếm trong ba ảnh huấn luyện chỉ có 4 `truck`, 7 `bus`, 3 `van`, không đủ để mô hình
  học phân biệt. Ranh giới `van`/`truck`/`bus` (Tình huống A, B) là chỗ cần thêm ảnh ví dụ nhất.
- Minh chứng nào có thể bác bỏ nhận định của bạn? Nếu chạy lại cùng dữ liệu với nhiều vòng lặp hơn, không đóng băng
  lớp, hoặc trên GPU, mà mô hình vẫn không phát hiện được cả những xe `car` lớn và rõ (ví dụ `drive_008` r18), thì
  nguyên nhân không nằm ở số vòng lặp mà ở đường ống dữ liệu, chẳng hạn ánh xạ mã lớp hoặc ghép ảnh–nhãn. Khi đó cần
  kiểm `data.yaml` và thư mục `dataset` do sổ thực hành tạo.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế?

Chỉ có 3 ảnh huấn luyện và 1 ảnh thẩm định, nên một hộp đúng hay sai cũng làm chỉ số dao động rất mạnh. Bốn ảnh có
thể là các khung hình liên quan về thời gian của cùng nguồn video, không đại diện cho góc camera, thời tiết hay ban
đêm. Cấu hình (8 vòng lặp, đóng băng 10 lớp, chạy CPU) chỉ nhằm kiểm tra đường ống dữ liệu chạy thông suốt. mAP ở
đây vì vậy không dùng để chấm người gán nhãn và không nói gì về khả năng dùng mô hình trong thực tế.

## 6. Đối chiếu nhãn

- Số hộp ghép được: 48
- IoU trung bình và trung vị: 0,8699 và 0,8941
- Mức đồng thuận lớp: 72,9% (35/48)
- Số hộp phía bạn không ghép được: 33
- Số hộp phía đối chiếu không ghép được: 2
- Một điểm khác biệt cụ thể: cả 13 cặp khác lớp theo đúng một quy luật: `truck` của tôi ↔ `bus` tham chiếu (5 cặp),
  `bus` ↔ `van` (4 cặp), `van` ↔ `truck` (4 cặp). Không có cặp `truck/bus/van` nào trùng lớp; 35 cặp `car` đều trùng.
  Kiểm tra từng hộp trên ảnh cho thấy nhãn phía tham chiếu trái với dấu hiệu nhìn thấy:

  - xe buýt khớp nối `drive_022` r4 và `drive_008` r6 bị gán `van`;
  - xe ben chở đất `drive_008` r3 bị gán `bus`;
  - xe van trắng `drive_008` r5 bị gán `truck`.

  IoU của các cặp này rất cao (0,86–0,99), nên hai bên cùng khoanh một chiếc xe; chỉ có lớp khác nhau. Quy luật
  hoán vị đều đặn khớp với giả thuyết bộ tham chiếu ghi mã lớp theo thứ tự `car, van, truck, bus` trong khi
  `data.yaml` khai báo `car, truck, bus, van`. Nếu đổi lại theo giả thuyết này, mức đồng thuận lớp là 48/48.
  Đây mới là giả thuyết, cần Lab Coach xác nhận.

  - Các khác biệt khác:
    - 33 hộp chỉ có ở phía tôi, chủ yếu là xe nhỏ ở xa trong hàng xe phía trên (`drive_008` r20–r23, r25) hoặc bị mép ảnh
      cắt. Trong số đó cũng có xe thấy rõ mà tham chiếu không gán, như hai xe buýt `drive_038` r15, r16.
    - 2 hộp tham chiếu không ghép được (`drive_008` hộp 8 và 10) nằm trên thân xe buýt khớp nối, nơi không thấy ô tô
      nào.
    - Cặp IoU thấp nhất, 0,168 (`drive_008` r11 ↔ tham chiếu 15): hộp tham chiếu lệch xuống nóc xe buýt, còn hộp của
      tôi bao phần nhìn thấy của xe bạc phía trên.
- Quy tắc hoặc hành động sửa phát sinh: không đổi lớp theo tham chiếu, vì 13/13 trường hợp đều trái dấu hiệu nhìn thấy
  và trái quy tắc lớp trong phiếu. Tôi báo Lab Coach nghi vấn lệch mã lớp của `day2-reference-4img-v1`, kèm bảng 13 cặp
  trong `comparison_iou.csv` và `comparison_overlay.png`. Đề xuất bổ sung một bước kiểm cho mọi gói nhãn trước khi phát
  hành: ngoài kiểm định dạng, mở ngẫu nhiên vài hộp của mỗi lớp trên ảnh để xác nhận tên lớp khớp nội dung. Về phạm vi,
  tôi đề xuất phiếu quy tắc ghi một ngưỡng kích thước tối thiểu cho xe ở xa, vì phần lớn 33 hộp không ghép được đến từ
  khác biệt này chứ không phải lỗi hình học.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng?

Đồng thuận chỉ đo hai bên giống nhau đến đâu, không đo bên nào khớp với ảnh. Bài này cho thấy cả hai chiều: IoU cao
(0,86–0,99) ở 13 cặp khác lớp chỉ nói hai hộp chồng khít, không nói lớp đúng; và 72,9% đồng thuận lớp ở đây thấp vì nguồn
đối chiếu có lỗi hệ thống, không phải vì nhãn của tôi sai. Ngược lại, nếu hai bên cùng mắc một lỗi, ví dụ cùng bỏ sót
xe bị che hoặc cùng gọi `van` là `car`, đồng thuận vẫn đạt 100% mà nhãn vẫn sai. Ngoài ra, IoU và đồng thuận lớp chỉ
tính trên hộp ghép được, nên không phản ánh 35 hộp không ghép được ở hai phía. Muốn biết nhãn đúng hay sai phải quay
lại ảnh và quy tắc.

## 7. Kiểm tra kho GitHub cá nhân

- [X] Có phiếu quy tắc với ba tình huống mơ hồ.
- [X] Có kết quả kiểm hai gói xuất.
- [X] Có thông tin lần huấn luyện và ảnh dự đoán.
- [X] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [X] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [X] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.

Minh chứng mạnh nhất trong bài và câu hỏi còn lại cho Lab Coach:

Minh chứng mạnh nhất là mẫu hoán vị lớp ở mục 6: 13/13 cặp `truck/bus/van` lệch theo đúng một quy luật, IoU 0,86–0,99,
và mỗi cặp đều kiểm được bằng mắt trên ảnh. Câu hỏi cho Lab Coach: (1) gói `day2-reference-4img-v1` có bị ghi mã lớp
theo thứ tự `car, van, truck, bus` không, và có bản sửa không? (2) Với xe ở xa nhỏ hơn khoảng 20 điểm ảnh, lớp nên gán
hay bỏ qua để bài của các học viên nhất quán?
