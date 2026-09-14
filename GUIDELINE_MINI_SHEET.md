# Phiếu quy tắc gán nhãn — Ngày 2

**Họ và tên:** Võ Quốc Dinh<br>
**MSSV:** 2A202602318<br>
**Hình thức:** cá nhân<br>
**Mã cặp:** SOLO

## 1. Phạm vi

- Chỉ gán phương tiện thuộc bốn lớp bên dưới.
- Mỗi phương tiện là một hộp; không gộp nhiều xe.
- Không gán người, xe máy, xe đạp, biển báo hoặc phần phản chiếu.
- Vật thể quá nhỏ hoặc mờ đến mức không thể phân lớp có căn cứ: không đoán; ghi lý do vào nhật ký quyết định.

## 2. Bốn lớp cố định

| Mã | Lớp                 | Gán khi nhìn thấy                                         | Không gán vào lớp này                               |
| --: | -------------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
|   0 | `car` (ô tô con) | sedan, hatchback, SUV, taxi, xe bán tải dùng như xe con  | xe có thùng/ben rõ; thân xe buýt; xe van thân hộp |
|   1 | `truck` (xe tải)  | thùng, ben, sàn hàng hoặc thiết bị công vụ rõ ràng | ô tô con; thân xe buýt; xe van kín một khối       |
|   2 | `bus` (xe buýt)   | thân xe khách dài, nhiều cửa sổ hoặc hàng ghế       | xe van nhỏ; xe tải; ô tô con                         |
|   3 | `van` (xe van)     | thân hộp nhỏ, kín, dùng chở người hoặc hàng        | thân xe buýt; khoang hàng tách biệt như xe tải    |

Thứ tự lớp là cố định: `0 car, 1 truck, 2 bus, 3 van`.

## 3. Hộp giới hạn

- Vẽ sát phần vật thể nhìn thấy.
- Không ước lượng phần bị xe khác che.
- Vật thể chạm mép ảnh vẫn được gán nếu đủ bằng chứng phân lớp.
- Không để hộp chứa nhiều nền hoặc nhiều phương tiện.

## 4. Ba thuộc tính

| Thuộc tính                             | Giá trị                                                         | Ý nghĩa                                   |
| ---------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------- |
| `visibility` (mức nhìn thấy)        | `clear` (rõ), `occluded` (bị che), `unclear` (không rõ) | mức bằng chứng nhìn thấy               |
| `boundary` (quan hệ mép ảnh)        | `inside` (trong ảnh), `truncated` (bị cắt)                 | vật thể có bị mép ảnh cắt hay không |
| `review_state` (trạng thái xem lại) | `confident` (tự tin), `needs_review` (cần xem lại)         | đánh dấu quyết định cần quay lại    |

YOLO không lưu ba thuộc tính này. Vì vậy phải xuất thêm `CVAT for images 1.1` từ cùng công việc.

## 5. Ba tình huống mơ hồ

Hoàn thành trước khi xem bài của người khác hoặc bộ nhãn tham chiếu.

### Tình huống A — xe buýt hay xe van?

- Ảnh và mã vật thể: `drive_038.jpg`, hộp #75, dòng 9 trong nhãn YOLO (xe màu đỏ bên phải, khoảng x 493–581, y 108–207)
- Dấu hiệu nhìn thấy: thân kín liền một khối với cabin, mui cao; chỉ khoảng 3–4 ô cửa sổ bên và một cửa
  trượt; thân ngắn, không có thân khách dài với nhiều hàng cửa sổ/hàng ghế như xe buýt khớp nối #59 trong cùng
  ảnh. Góc dưới bên trái thân xe bị thùng xe tải xanh #74 che một phần.
- Quy tắc áp dụng: `bus` cần "thân xe khách dài, nhiều cửa sổ hoặc hàng ghế"; `van` là "thân hộp nhỏ, kín,
  dùng chở người hoặc hàng". Không quyết định theo màu hay kích thước hộp. Phần bị che không ước lượng thêm.
- Quyết định: `van` (mã 3), `visibility=occluded`, `boundary=inside`, `review_state=confident`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Phóng ảnh 100% đếm cửa sổ/cửa lên xuống; nếu vẫn không thấy rõ chiều
  dài thân, gán `review_state=needs_review`, ghi lý do và quay lại xử lý trước khi xuất, không đoán theo màu.

### Tình huống B — xe tải hay xe van/ô tô con?

- Ảnh và mã vật thể: `drive_033.jpg`, hộp #39, dòng 18 trong nhãn YOLO (xe trắng làn giữa, khoảng x 252–316, y 281–455)
- Dấu hiệu nhìn thấy: cabin phía trước tách rời với thùng hàng hộp phía sau; thùng cao và dài hơn cabin rõ rệt,
  có khe nối giữa cabin và thùng; không có cửa sổ khách dọc thân.
- Quy tắc áp dụng: `truck` khi có "thùng, ben, sàn hàng" rõ ràng; `van` chỉ khi là "xe van kín một khối", không
  có "khoang hàng tách biệt như xe tải".
- Quyết định: `truck` (mã 1), `visibility=clear`, `boundary=inside`, `review_state=confident`. Áp dụng tương tự
  cho xe thùng kín #30 (dòng 1 trong nhãn YOLO) trong `drive_022.jpg`.
- Nếu vẫn thiếu bằng chứng, bạn sẽ làm gì? Tìm khe tách cabin–thùng khi phóng 100%; nếu không thấy được (ví dụ
  bị che), gán `needs_review`, ghi lý do và xử lý lại trước khi xuất.

### Tình huống C — bị che, bị mép ảnh cắt hay không đủ bằng chứng?

- Ảnh và mã vật thể: `drive_008.jpg`, hộp #13, dòng 24 trong nhãn YOLO (góc phải dưới, khoảng x 620–640, y 565–640)
- Dấu hiệu nhìn thấy khi phóng 100%: chỉ thấy một dải hẹp gồm bánh xe và thân xe màu tối thấp, sát mặt đường;
  phần còn lại nằm ngoài mép phải và mép dưới ảnh; không có xe nào khác che.
- Giá trị `visibility`: `unclear`
- Giá trị `boundary`: `truncated`
- Trạng thái `review_state`: `confident` (đã xem lại sau khi đặt `needs_review` ban đầu)
- Lý do: bị cắt bởi mép ảnh chứ không bị xe khác che, nên `boundary=truncated` và không dùng `occluded`. Phần
  nhìn thấy ít nên `visibility=unclear`, nhưng hình dạng bánh xe và thân thấp đủ căn cứ cho `car`; hộp chỉ vẽ
  sát phần nhìn thấy, không ước lượng phần ngoài ảnh. Ngược lại, các vật thể quá nhỏ/mờ không phân lớp được
  thì không gán (xem nhật ký bên dưới).

### Nhật ký quyết định — vật thể không gán

- `drive_008.jpg`: hai xe nhỏ ở mép trên (x ≈ 483–513, y 0–25) và xe ở đỉnh ảnh (x ≈ 205–215, y 0–8) — bị cắt,
  quá nhỏ/mờ, không phân biệt được `car`/`van` (đã thử gán rồi xóa sau khi xem lại). Vật màu đỏ sát mép phải
  (x ≈ 623–640, y 259–345) — chỉ thấy một dải tối, không đủ căn cứ (đã thử gán rồi xóa). Xe mờ trong hàng xe phía
  trên (x ≈ 380–397, y 3–20) — dính liền với xe #26 phía trước, không tách được thành một phương tiện riêng, xóa
  để tránh gán trùng. Vật phía sau van #2 là xe máy chở hàng — ngoài phạm vi.
- `drive_033.jpg`: hai xe đỗ gần nhà chờ bên trái (x ≈ 150–200, y 95–115) và các xe rất xa ở đầu đường
  (khoảng 10 px, gồm x ≈ 276–287, y 72–87) — quá mờ để phân lớp. Xe mờ phía trước taxi #41 (x ≈ 353–373,
  y 105–128) — nhòe, không tách rõ khỏi #41, xóa để tránh gán trùng. Xe ba bánh/xe đạp chở hàng bên phải —
  ngoài phạm vi.
- Mọi ảnh: không gán người, xe máy, xe đạp, biển báo và phản chiếu trên mặt đường ướt (`drive_038.jpg`).

## 6. Xác nhận tự kiểm tra

- [X] Đã rà đủ bốn ảnh.
- [X] Đã kiểm vật thể thiếu và trùng.
- [X] Đã kiểm lớp và hình học từng hộp.
- [X] Mỗi hộp có đủ ba thuộc tính.
- [X] Đã xử lý mọi hộp `needs_review`.
- [X] Đã hoàn thành ba tình huống trước khi xem nguồn đối chiếu.
- [ ] Nếu làm theo cặp, hai người đã xuất bài độc lập trước khi trao đổi. (không áp dụng — làm cá nhân)
- [X] Nếu làm cá nhân, bài riêng đã được kiểm trước khi nhận bộ tham chiếu.
- [X] Số vật thể thực tế: 81 (drive_008: 25, drive_022: 5, drive_033: 20, drive_038: 31) — 40–60 là mục tiêu khối lượng, không phải điểm cắt.