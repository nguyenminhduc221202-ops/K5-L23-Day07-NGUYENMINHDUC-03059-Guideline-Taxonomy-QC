# Guideline gán nhãn — v0.1 — 101

- Phiên bản: v0.1-draft
- Ngày: 2026-11-16
- Người viết: Thành viên 101 (Nguyễn Minh Đức)

## 1. Mục tiêu

Bộ dữ liệu nhãn bounding box 2D này được sử dụng để huấn luyện và đánh giá mô hình thị giác máy tính phát hiện đối tượng (người và phương tiện) phục vụ bài toán lái xe tự động / hệ thống an toàn giao thông đường phố.
- Mục tiêu chính: Phát hiện chính xác vị trí, kích thước và phân loại các thực thể tham gia giao thông thực tế.
- Rủi ro an toàn: Bỏ sót người đi bộ, xe máy hoặc nhận diện sai kích thước thực tế của xe tải/ô tô có thể dẫn đến va chạm nghiêm trọng. Việc gán nhầm biển quảng cáo, tượng điêu khắc hoặc hình ảnh phản chiếu là vật thể thật sẽ gây phanh gấp không cần thiết (phantom braking).

## 2. Lớp và thuộc tính

### Lớp đối tượng (Classes)

| Lớp | Định nghĩa (một câu) | Ví dụ thực tế | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô con từ 4 đến 9 chỗ, xe sedan, SUV, hatchback. | Xe taxi, xe ô tô con lưu thông trên đường (`img_040.jpg`). | Xe bán tải lớn (xem `truck`), xe đồ chơi trong tủ kính (`img_038.jpg`). |
| `truck` | Xe tải chở hàng, xe đầu kéo, xe chuyên dụng có cabin và thùng hàng riêng biệt. | Xe chở ô tô (`img_002.jpg`), xe tải nhỏ chở hàng (`img_036.jpg`). | Tượng đài xe bán tải bằng đá (`img_027.jpg`), xe tuk-tuk (`img_004.jpg`). |
| `van` | Xe ô tô thân kín liền khối dạng hộp chở người hoặc chở hàng cỡ nhỏ/vừa. | Xe cứu thương nhỏ, xe van bưu chính liền khung. | Xe buýt lớn chở khách (cần thêm `bus`), xe ô tô con 7 chỗ (xếp vào `car`). |
| `motorcycle` | Xe hai bánh gắn động cơ cơ giới (xe máy, mô tô, xe tay ga). | Xe máy di chuyển trên đường (`img_005.jpg`, `img_026.jpg`). | Xe đạp không gắn động cơ, xe xích lô đạp chân (`img_006.jpg`, `img_009.jpg`). |
| `person` | Con người thật bằng xương bằng thịt (người đi bộ, người lái xe, hành khách). | Người đi bộ qua đường (`img_014.jpg`), người điều khiển xe máy (`img_026.jpg`). | Ma-nơ-canh trong shop (`img_033.jpg`), tượng danh nhân (`img_017.jpg`), ảnh chân dung trên biển pano (`img_018.jpg`). |
| `ignore_region` | Vùng hình chữ nhật đánh dấu khu vực quá đông đúc, mờ nhòe hoặc vật thể quá nhỏ không thể phân định chính xác. | Cụm người/xe ở hậu cảnh xa tít tắp cao dưới 20 px (`img_025.jpg`). | Vật thể rõ nét gần camera. |

*Xử lý vật ngoài taxonomy:*
- Xe tuk-tuk 3 bánh (`img_004.jpg`, `img_035.jpg`): v0 chưa có lớp; đề xuất nhóm bổ sung lớp `tuk_tuk` ở Pha 2, tạm thời chưa ép vào `car` hay `motorcycle`.
- Xe buýt cỡ lớn (`img_032.jpg`, `img_040.jpg`): đề xuất bổ sung lớp `bus`.
- Xe thô sơ không động cơ (xích lô `img_006.jpg`, xe đạp `img_039.jpg`): không gán hoặc đánh dấu `ignore_region` nếu chắn lối.

### Thuộc tính (Attributes)

| Thuộc tính | Kiểu / Mức cố định | Định nghĩa ý nghĩa | Ngưỡng đo lường cụ thể |
|---|---|---|---|
| `occluded_level` | Select: `0`, `1`, `2`, `3` | Mức độ diện tích vật thể bị che khuất bởi vật khác | - `0`: Thấy toàn bộ, diện tích che 0%<br/>- `1`: Che khuất nhẹ, từ 1% đến 30% diện tích<br/>- `2`: Che khuất vừa/nặng, từ 31% đến 70% diện tích<br/>- `3`: Bị che trên 70% diện tích nhưng vẫn đủ đặc trưng nhận diện chắc chắn |
| `truncated` | Select: `no`, `yes` | Vật thể bị cắt cụt bởi mép bức ảnh | - `no`: Toàn bộ vật thể nằm trọn vẹn trong khung hình<br/>- `yes`: Bị mép ảnh cắt qua bất kỳ phần thân xe / người |
| `can_review` | Checkbox: `false`, `true` | Đánh dấu cần hội ý hoặc reviewer kiểm tra lại | - `false`: Tự tin gán theo quy tắc<br/>- `true`: Trường hợp biên gây tranh cãi, chưa thống nhất |

## 3. Quy tắc vẽ box

1. **Ôm sát phần nhìn thấy (Tight Bounding Box):**
   - Box phải tiếp xúc 4 cạnh ngoài cùng của phần vật thể nhìn thấy (visible part). Không ước lượng vẽ bù phần bị khuất vào không gian trống.
2. **Người đi xe máy / phương tiện hở:**
   - Vẽ tách riêng thành 2 box: 1 box `person` cho người điều khiển (và người ngồi sau) ôm từ đỉnh đầu đến chân/bàn đạp; 1 box `motorcycle` ôm toàn bộ khung xe, bánh xe và tay lái. Phần giao nhau giữa người và xe không trừ ra.
3. **Xe chở phương tiện khác (Car carrier - `img_002.jpg`):**
   - Xe chuyên chở được gán 1 box `truck` bao toàn bộ đầu kéo và rơ-moóc chở hàng.
   - Mỗi chiếc xe ô tô con nằm trên các tầng sàn xe nếu còn nguyên vẹn đặc trưng hình học thì được gán thêm 1 box `car` độc lập tương ứng.
4. **Hàng hóa trên xe máy / tuk-tuk (`img_029.jpg`, `img_035.jpg`):**
   - Bounding box của phương tiện chỉ bao trọn thân xe cơ giới; không kéo rộng box để bao trọn các kiện hàng, sọt hàng cồng kềnh nhô cao hoặc bành trướng quá khổ bên ngoài.
5. **Vật thể phi thực thể (Tượng, Ma-nơ-canh, Tranh ảnh, Phản chiếu):**
   - **Tuyệt đối không gán:** Tượng đồng/tượng đá (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh quần áo (`img_033.jpg`), hình ảnh in trên biển quảng cáo (`img_018.jpg`), hình ảnh phản chiếu qua gương chiếu hậu (`img_020.jpg`).
6. **Ngưỡng kích thước tối thiểu (Min height threshold):**
   - Đối tượng có chiều cao nhỏ hơn 20 pixel thì không gán riêng lẻ từng box; nếu đối tượng ở vị trí then chốt trên làn đường thì gom vào `ignore_region`.

## 4. Ca khó và tình huống biên

| Ảnh | Tình huống | Rule ID (Đề xuất) | Quyết định cá nhân | Lý do căn cứ |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải Renault chở 8 ô tô con trên hai tầng sàn | R-01 | Gán 1 box `truck` và 8 box `car` riêng lẻ | Ô tô con dù là hàng hóa nhưng mang đầy đủ hình dáng, màu sắc xe thật, mô hình 2D phải nhận diện để tránh va chạm. |
| `img_017.jpg` | Tượng công tước Duke of Wellington cưỡi ngựa đội nón chóp giao thông | R-02 | Không gán box nào (bỏ qua cả tượng người và ngựa) | Đây là tác phẩm điêu khắc nghệ thuật bất động, không phải người thật tham gia giao thông; gán vào sẽ khiến xe tự lái nhận diện sai. |
| `img_035.jpg` | Xe ba bánh chở hàng ngập tải (quá khổ) ở Hyderabad | R-03 | Đề xuất bổ sung lớp `tuk_tuk`, box chỉ ôm thân xe | v0 không có lớp xe 3 bánh. Hàng hóa bao tải phình to bất thường phía trên nóc không thuộc kết cấu cơ khí của xe. |
| `img_027.jpg` | Tượng đài đúc hình chiếc xe bán tải bằng đá hoa cương | R-02 | Không gán nhãn `truck` hay `car` | Là đài tưởng niệm cố định trong công viên nghĩa trang, không có khả năng di chuyển hay phát sinh quỹ đạo va chạm. |

## 5. QC và quản lý phiên bản

- Quy tắc quản lý phiên bản: Mọi bổ sung lớp nhãn mới (`bus`, `tuk_tuk`) hoặc điều chỉnh ranh giới phân lớp bắt buộc phải nâng phiên bản guideline lên `v1.0` và cập nhật tương ứng vào `taxonomy.json`.
- Ngưỡng QC áp dụng:
  - Tỷ lệ trùng khớp hình học (IoU) trung bình giữa các người gán: $\ge 0.85$.
  - Tỷ lệ đồng thuận lớp nhãn (Class Agreement): $\ge 95\%$.
  - Tỷ lệ lỗi toàn lô khi nghiệm thu: $\le 2\%$.
- Lịch sử thay đổi:

| Phiên bản | Thay đổi chính | Lý do thực tế từ 40 ảnh |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên | Chưa xử lý được xe 3 bánh, tượng, xe chở xe, ngưỡng thuộc tính còn mơ hồ. |
| `v0.1-draft` | Bổ sung định lượng ngưỡng %, quy tắc tượng, quy tắc xe chở xe, phân tích ca khó | Bản nháp cá nhân thành viên 101 chuẩn bị cho buổi họp Pha 2. |
