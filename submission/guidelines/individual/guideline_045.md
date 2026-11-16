# Guideline gán nhãn — — 045

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết: Nguyễn Thế Phách

## 1. Mục tiêu

Bộ nhãn bounding box 2D này phục vụ [mục đích / mô hình sử dụng] trong [bối cảnh]. Nếu bỏ sót hoặc gán nhầm [đối tượng quan trọng], hệ thống có thể [hậu quả cụ thể].

## 2. Lớp và thuộc tính

### Lớp đối tượng (Classes)

| Lớp             | Định nghĩa (một câu)                                                                                          | Ví dụ thực tế                                                                   | Phản ví dụ / Dễ nhầm với                                                                                              |
| --------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `car`           | Xe ô tô con từ 4 đến 9 chỗ, xe sedan, SUV, hatchback.                                                         | Xe taxi, xe ô tô con lưu thông trên đường (`img_040.jpg`).                      | Xe bán tải lớn (xem `truck`), xe đồ chơi trong tủ kính (`img_038.jpg`).                                               |
| `truck`         | Xe tải chở hàng, xe đầu kéo, xe chuyên dụng có cabin và thùng hàng riêng biệt.                                | Xe chở ô tô (`img_002.jpg`), xe tải nhỏ chở hàng (`img_036.jpg`).               | Tượng đài xe bán tải bằng đá (`img_027.jpg`), xe tuk-tuk (`img_004.jpg`).                                             |
| `van`           | Xe ô tô thân kín liền khối dạng hộp chở người hoặc chở hàng cỡ nhỏ/vừa.                                       | Xe cứu thương nhỏ, xe van bưu chính liền khung.                                 | Xe buýt lớn chở khách (cần thêm `bus`), xe ô tô con 7 chỗ (xếp vào `car`).                                            |
| `motorcycle`    | Xe hai bánh gắn động cơ cơ giới (xe máy, mô tô, xe tay ga).                                                   | Xe máy di chuyển trên đường (`img_005.jpg`, `img_026.jpg`).                     | Xe đạp không gắn động cơ, xe xích lô đạp chân (`img_006.jpg`, `img_009.jpg`).                                         |
| `person`        | Con người thật bằng xương bằng thịt (người đi bộ, người lái xe, hành khách).                                  | Người đi bộ qua đường (`img_014.jpg`), người điều khiển xe máy (`img_026.jpg`). | Ma-nơ-canh trong shop (`img_033.jpg`), tượng danh nhân (`img_017.jpg`), ảnh chân dung trên biển pano (`img_018.jpg`). |
| `ignore_region` | Vùng hình chữ nhật đánh dấu khu vực quá đông đúc, mờ nhòe hoặc vật thể quá nhỏ không thể phân định chính xác. | Cụm người/xe ở hậu cảnh xa tít tắp cao dưới 20 px (`img_025.jpg`).              | Vật thể rõ nét gần camera.                                                                                            |

_Xử lý vật ngoài taxonomy:_

- Xe tuk-tuk 3 bánh (`img_004.jpg`, `img_035.jpg`): v0 chưa có lớp; đề xuất nhóm bổ sung lớp `tuk_tuk` ở Pha 2, tạm thời chưa ép vào `car` hay `motorcycle`.
- Xe buýt cỡ lớn (`img_032.jpg`, `img_040.jpg`): đề xuất bổ sung lớp `bus`.
- Xe thô sơ không động cơ (xích lô `img_006.jpg`, xe đạp `img_039.jpg`): không gán hoặc đánh dấu `ignore_region` nếu chắn lối.

### Thuộc tính

| Thuộc tính         | Các mức hợp lệ | Ý nghĩa từng mức / ngưỡng |
| ------------------ | -------------- | ------------------------- |
| `[tên thuộc tính]` | `[giá trị]`    | `[cách xác định]`         |

Vật không thuộc lớp nào: [quyết định của bạn về không gán, `ignore_region`, hay đề xuất lớp mới].

## 3. Quy tắc vẽ box

- **Phần được bao:** [nêu cách xác định biên box quanh vật thể nhìn thấy].
- **Che khuất / cắt mép ảnh:** [nêu cách vẽ và ghi thuộc tính tương ứng].
- **Người và phương tiện:** [nêu cách xử lý khi hai đối tượng chồng lấp hoặc che nhau].
- **Ngoài phạm vi:** [nêu cách xử lý phản chiếu, hình in, tượng, đồ chơi hoặc vật không có lớp].
- **Vật nhỏ / đám đông:** [nêu ngưỡng và khi nào dùng `ignore_region`].

## 4. Ca khó và tình huống biên

| Ảnh           | Tình huống                                                                                                  | Rule ID | Quyết định cá nhân | Lý do |
| ------------- | ----------------------------------------------------------------------------------------------------------- | ------- | ------------------ | ----- |
| `img_001.jpg` | Nhiều người đứng/ngồi gần xe bán tải; người và xe che khuất lẫn nhau, một số phần cơ thể khó tách khỏi nền. |         |                    |       |
| `img_003.jpg` | Đường phố đông người và xe máy; người bán hàng cùng các giỏ hàng lớn che một phần người và phương tiện.     |         |                    |       |
| `img_006.jpg` | Nhiều người đi xe hai bánh ở gần nhau; cần phân biệt xe máy với xe đạp và xem xét vật thể bị cắt ở mép ảnh. |         |                    |       |

Nếu chưa chốt, ghi rõ điều cần hỏi reviewer hoặc đánh dấu `can_review` khi gán nhãn.

## 5. QC và phiên bản

### Ngưỡng QC áp dụng

Theo `submission/taxonomy.json` v1.0 của nhóm:

- Chiều cao box tối thiểu: 20 px.
- Ngưỡng IoU box: 0.85 với vật lớn (từ 50 px); 0.70 với vật nhỏ.
- Ngưỡng IAA: IoU tối thiểu 0.85; đồng thuận lớp tối thiểu 0.95.
- Ngưỡng lỗi tối đa theo ảnh: 2%; ngưỡng IoU để ghép cặp box: 0.50.

### Ca chưa chắc chắn

- Nếu đã vẽ được box nhưng chưa chắc nhãn hoặc thuộc tính, bật `can_review`.
- Nếu chưa thể vẽ box, ghi ảnh, vùng và lý do vào `submission/qc/image_issues.csv` sau khi khóa nhãn.
- Ca chưa chốt được ghi lại để nhóm thống nhất; không tự thay đổi taxonomy hoặc các ngưỡng QC.

### Lịch sử phiên bản

| Phiên bản    | Thay đổi                                                                 | Lý do (ca nào)                                         |
| ------------ | ------------------------------------------------------------------------ | ------------------------------------------------------ |
| `v0`         | Guideline khởi đầu                                                       | Bản gốc trước khi xem xét các tình huống trong bộ ảnh. |
| `v0.1-draft` | Ghi nhận các tình huống cần tự chốt và chuẩn bị đề xuất quy tắc cá nhân. | `img_001.jpg`, `img_003.jpg`, `img_006.jpg`.           |
