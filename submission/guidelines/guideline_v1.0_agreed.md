# Guideline gán nhãn thống nhất — v1.0 — Nhóm K5-L23-Nhom3-GuidelineQC

- Phiên bản: `v1.0` (Khóa cho lượt gán độc lập và đo độ đồng thuận IAA)
- Ngày hiệu lực: 2026-11-16
- Nhóm biên soạn: K5-L23-Nhom3-GuidelineQC
- Danh sách thành viên:
  - `029` (Phạm Lam Anh — Điều phối nhóm)
  - `059` (Nguyễn Minh Đức — Reviewer bản cuối)
  - `038` (Nguyễn Thành Tài)
  - `045` (Nguyễn Thế Phách)
  - `048` (Thành viên 048)

---

## 1. Mục tiêu và bối cảnh sử dụng

Tài liệu này xác lập bộ quy tắc gán nhãn bounding box 2D chuẩn mực và duy nhất của nhóm cho tập dữ liệu ảnh đường phố (40 ảnh), phục vụ bài toán thị giác máy tính phát hiện đối tượng (Object Detection 2D) ứng dụng trong hệ thống hỗ trợ lái xe nâng cao (ADAS) và xe tự hành:

- **Mục tiêu kỹ thuật:** Xác định vị trí không gian (bounding box 2D ôm sát phần nhìn thấy), phân loại chuẩn xác 8 lớp đối tượng giao thông thực tế và gán đúng các thuộc tính trạng thái che khuất, cắt mép.
- **Rủi ro an toàn thực tế nếu gán sai:**
  - *Bỏ sót đối tượng (False Negative):* Bỏ sót người đi bộ, người đi xe máy hoặc xe cộ đang lưu thông sẽ khiến hệ thống không kích hoạt cảnh báo va chạm hoặc phanh khẩn cấp tự động (AEB), dẫn đến tai nạn thảm khốc.
  - *Gán sai kích thước / ranh giới:* Vẽ bounding box quá lỏng hoặc quá chật, đặc biệt với các phương tiện cồng kềnh, khiến module lập quỹ đạo (Motion Planning) tính toán sai khoảng cách an toàn và thời gian va chạm (TTC).
  - *Gán nhầm vật thể tĩnh phi thực thể (False Positive):* Gán các bức tượng nghệ thuật, ma-nơ-canh trong cửa hàng, hình người trên biển quảng cáo hoặc ảnh phản chiếu qua gương chiếu hậu thành đối tượng thật sẽ gây ra hiện tượng phanh gấp ngoài ý muốn (*phantom braking*), cực kỳ nguy hiểm cho các phương tiện chạy phía sau.

---

## 2. Đặc tả Lớp và Thuộc tính

Bộ nhãn thống nhất tuân thủ nghiêm ngặt theo định nghĩa trong `submission/taxonomy.json` phiên bản `v1.0`.

### 2.1. Danh mục 8 Lớp đối tượng (Classes)

| Lớp | Định nghĩa chuẩn xác | Ví dụ điển hình từ bộ 40 ảnh | Phản ví dụ / Dễ nhầm lẫn |
|---|---|---|---|
| `car` | Xe ô tô con từ 4 đến 9 chỗ ngồi phục vụ chở người (sedan, hatchback, SUV, crossover, xe taxi). | Xe ô tô con lưu thông hoặc đỗ (`img_018.jpg`, `img_024.jpg`, `img_040.jpg`). | Xe bán tải có thùng hàng hở (xếp vào `truck`); xe mô hình đồ chơi trong tủ kính (`img_038.jpg` — **không gán**). |
| `truck` | Xe cơ giới chở hàng hóa có cabin và thùng hàng tách biệt (xe tải, xe đầu kéo kéo rơ-moóc, xe bán tải/pickup có thùng hàng hở, xe chuyên dụng chở xe). | Xe tải chở ô tô (`img_002.jpg`), xe tải giao hàng (`img_007.jpg`, `img_036.jpg`), xe bán tải chở hàng (`img_001.jpg`, `img_012.jpg`). | Xe van thân hộp kín liền khối (`van`); xe buýt khách lớn (`bus`); tượng đài xe bán tải bằng đá (`img_027.jpg` — **không gán**). |
| `van` | Xe ô tô thân hộp kín liền khối cỡ nhỏ hoặc vừa, cabin và khoang chở hàng/chở khách chung một vỏ kết cấu. | Xe van giao hàng bưu chính, xe cứu thương nhỏ (`img_005.jpg`, `img_016.jpg`). | Xe buýt lớn chở khách (`bus`); xe tải có thùng hàng tách cabin (`truck`); xe 7 chỗ gia đình (`car`). |
| `bus` | Xe ô tô chở khách cỡ lớn công cộng hoặc du lịch, chiều dài lớn, nhiều hàng ghế, thường từ 17 chỗ ngồi trở lên. | Xe buýt đô thị lưu thông trên đường (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`). | Xe van nhỏ chở khách (`van`); xe tải thùng kín (`truck`). |
| `tuk_tuk` | Xe ba bánh gắn động cơ cơ giới có mái che, kiểu auto rickshaw, dùng chở khách hoặc chở hàng nhẹ. | Xe ba bánh tuk-tuk Thái Lan (`img_004.jpg`), xe tuk-tuk chở hàng ở Ấn Độ (`img_035.jpg`). | Xe mô-tô hai bánh (`motorcycle`); xe xích lô đạp chân (`img_006.jpg` — **không gán**). |
| `motorcycle` | Xe hai bánh gắn động cơ cơ giới (xe máy số, xe tay ga, mô-tô phân khối lớn). | Xe máy di chuyển trong dòng giao thông (`img_005.jpg`, `img_006.jpg`, `img_008.jpg`, `img_026.jpg`). | Xe đạp không động cơ; xe xích lô đạp chân (`img_006.jpg`, `img_009.jpg`). |
| `person` | Con người sinh học thật bằng xương bằng thịt đang sống (người đi bộ, người lái xe, người ngồi sau xe). | Người đi bộ qua đường (`img_003.jpg`, `img_014.jpg`), người điều khiển xe máy (`img_026.jpg`). | Tượng đài danh nhân (`img_017.jpg` — **không gán**); ma-nơ-canh cửa hàng (`img_033.jpg` — **không gán**); hình in quảng cáo (`img_018.jpg` — **không gán**). |
| `ignore_region` | Vùng hình chữ nhật đánh dấu khu vực quá đông đúc, mờ nhòe do mưa/chuyển động hoặc cụm vật thể quá nhỏ không thể phân định riêng. | Cụm người xe ở phía xa bờ biển (`img_025.jpg`), cụm mờ nhòe xa xăm trong mưa (`img_015.jpg`). | Đối tượng rõ nét, kích thước đủ lớn ở tiền cảnh và trung cảnh. |

*Xử lý vật thể ngoài taxonomy:*
- **Phương tiện thô sơ không động cơ** (xe đạp, xe xích lô đạp chân, xe đẩy hàng rong, động vật): **Tuyệt đối KHÔNG gán nhãn phương tiện** và không gán ép vào `motorcycle`. Nếu có người điều khiển/người đẩy rõ nét, chỉ gán riêng box `person` cho người đó.
- Không tự ý thêm lớp mới ngoài 8 lớp đã thống nhất trong `taxonomy.json`.

---

### 2.2. Danh mục Thuộc tính đối tượng (Attributes)

| Thuộc tính | Kiểu dữ liệu | Giá trị hợp lệ | Ý nghĩa và ngưỡng định lượng chuẩn |
|---|---|---|---|
| `occluded_level` | Select | `"0"`, `"1"`, `"2"`, `"3"` | Mức độ diện tích bề mặt nhìn thấy bị **vật thể khác che khuất**:<br/>- `"0"`: Không bị che (0% diện tích bị che).<br/>- `"1"`: Che khuất nhẹ (từ 1% đến 30% diện tích bị che).<br/>- `"2"`: Che khuất vừa/nặng (từ 31% đến 70% diện tích bị che).<br/>- `"3"`: Bị che khuất rất nặng (> 70% diện tích bị che nhưng vẫn đủ đặc trưng nhận dạng chắc chắn). |
| `truncated` | Select | `"no"`, `"yes"` | Vật thể bị **cắt cụt bởi mép bức ảnh**:<br/>- `"no"`: Toàn bộ các bộ phận của vật thể nằm trọn trong khung hình.<br/>- `"yes"`: Bất kỳ bộ phận nào của vật thể bị ít nhất một trong bốn mép ảnh cắt qua (box tiếp xúc mép ảnh $\le 2\text{ px}$). |
| `can_review` | Checkbox | `false`, `true` | Đánh dấu ca mơ hồ cần hội ý nhóm hoặc reviewer kiểm tra lại:<br/>- `false`: Tự tin gán theo đúng quy tắc.<br/>- `true`: Trường hợp biên gây tranh cãi về ranh giới box hoặc phân lớp. |

> [!IMPORTANT]
> **Độc lập giữa Cắt mép và Che khuất:** Cắt mép ảnh (`truncated`) và che khuất bởi vật khác (`occluded_level`) là hai thuộc tính hoàn toàn độc lập. Một đối tượng ở mép ảnh không bị vật nào che thì gán `truncated="yes"` và `occluded_level="0"`. Vùng `ignore_region` không mang thuộc tính.

---

## 3. Hệ thống Quy tắc vẽ Bounding Box (Rules)

Mọi annotator trong nhóm bắt buộc tuân thủ 10 quy tắc kỹ thuật sau:

- **R-01 — Bounding Box ôm sát phần nhìn thấy (Tight Box on Visible Part):**
  Box hình chữ nhật (trục song song với biên ảnh) phải tiếp xúc chính xác 4 điểm cực biên của phần vật thể thực tế nhìn thấy được. Tuyệt đối không phán đoán vẽ bù phần bị che khuất vào khoảng trống không nhìn thấy.

- **R-02 — Tách riêng Người và Phương tiện hở (Person and Open Vehicle Separation):**
  Với xe máy (`motorcycle`) hoặc phương tiện hở:
  - Vẽ **1 box `person`** cho từng người trên xe (người lái và người ngồi sau): ôm từ đỉnh đầu/mũ bảo hiểm xuống hết chân người.
  - Vẽ **1 box `motorcycle`** cho phương tiện: ôm trọn khung xe, bánh xe tiếp đất và tay lái.
  - Vùng giao thoa giữa người và xe không trừ ra; cả hai box đều bao trọn phần của mình.

- **R-03 — Người ngồi trong khoang kín (Enclosed Cabins):**
  - Người ngồi trong ô tô con (`car`), xe tải (`truck`), xe van (`van`) kín cửa kính: **Không gán box `person`**.
  - Người ngồi trong khoang hở của xe ba bánh (`tuk_tuk`) hoặc thùng xe bán tải hở: Chỉ gán `person` nếu nhìn thấy rõ phần thân trên và phân biệt được rõ ràng với xe.

- **R-04 — Phân loại xe thùng hở và xe thân kín (Pickup vs Van vs Truck):**
  - Xe bán tải (pickup) có thùng hàng hở phía sau tách rời cabin $\rightarrow$ gán `truck` (`img_001.jpg`, `img_012.jpg`).
  - Xe thùng hộp liền khối với cabin (kể cả có cửa trượt hoặc chở hàng bưu chính) $\rightarrow$ gán `van` (`img_005.jpg`, `img_016.jpg`).

- **R-05 — Xe chuyên dụng chở ô tô (Car Carrier - `img_002.jpg`):**
  - Đối với xe tải chở ô tô hai tầng: Toàn bộ xe đầu kéo và giàn rơ-moóc chuyên dụng được gán **1 box `truck`** bao quát toàn bộ khối xe chuyên chở.
  - *Thống nhất của nhóm:* Các xe ô tô con trên sàn xe tải được xem là khối hàng hóa vận chuyển cố định trên phương tiện chuyên dụng, không tham gia giao thông độc lập. Tuy nhiên, do các xe con này có đầy đủ đặc trưng thị giác 2D, nhóm ghi nhận sự khác biệt quan điểm giữa các thành viên để đưa vào theo dõi tại buổi hiệu chuẩn Pha 4–5.

- **R-06 — Hàng hóa chở trên phương tiện (`img_029.jpg`, `img_035.jpg`):**
  Bounding box của phương tiện (`motorcycle`, `tuk_tuk`) ôm trọn kết cấu cơ khí của xe. Đối với các kiện hàng hóa cồng kềnh xếp nhô cao quá mức (như bao tải trên nóc xe tuk-tuk `img_035.jpg` hay sọt hàng sau xe máy `img_029.jpg`), không kéo rộng box để bao trùm khối hàng nhô ra ngoài trừ khi khối hàng gắn liền với thùng xe và là ranh giới chiếm dụng không gian bắt buộc.

- **R-07 — Tuyệt đối loại trừ vật thể phi thực thể (Non-real & Static Artworks):**
  Tuyệt đối **KHÔNG gán bất kỳ box nào** cho:
  - Tượng đài nghệ thuật danh nhân, tượng ngựa đá (`img_017.jpg`).
  - Tượng đài xe bán tải đúc bằng đá hoa cương (`img_027.jpg`).
  - Ma-nơ-canh trưng bày trong cửa hàng (`img_033.jpg`).
  - Ảnh chân dung, tranh vẽ 2D, decal in trên biển quảng cáo pano (`img_018.jpg`).
  - Ảnh phản chiếu của người hoặc xe qua gương chiếu hậu (`img_020.jpg`).
  - Mô hình xe ô tô đồ chơi, búp bê trưng bày trong tủ kính (`img_038.jpg`).

- **R-08 — Ngưỡng kích thước tối thiểu (Min Box Height Threshold):**
  - Không gán box riêng lẻ cho bất kỳ đối tượng nào có chiều cao phần nhìn thấy nhỏ hơn **20 pixel**.
  - Nếu ở hậu cảnh xa xuất hiện cụm người/xe có kích thước $< 20\text{ px}$ dày đặc không thể phân định riêng, sử dụng một vùng hình chữ nhật `ignore_region` để bao phủ cụm đó (`img_025.jpg`).

- **R-09 — Phương tiện ngoài taxonomy (Out-of-taxonomy Vehicles):**
  Xe đạp không động cơ, xe xích lô đạp chân, xe kéo, xe đẩy tay không được gán bất kỳ nhãn phương tiện nào; chỉ gán người (`person`) nếu người đó cao $\ge 20\text{ px}$ (`img_003.jpg`, `img_006.jpg`).

- **R-10 — Xe mang camera / Xe chủ (Ego Vehicle Exclusion):**
  Không gán các bộ phận của chính chiếc xe mang camera (như nắp ca-pô, gương chiếu hậu mép dưới ảnh) thành đối tượng chướng ngại vật phía trước.

---

## 4. Bảng ca khó và Tình huống biên điển hình (Edge Cases)

| Mã ảnh | Tình huống cụ thể | Mã Rule | Quyết định thống nhất của nhóm | Căn cứ kỹ thuật & Lý do an toàn |
|---|---|---|---|---|
| `img_001.jpg` | Người đứng cạnh đuôi xe bán tải chở hàng; xe tải bị cắt ở mép phải ảnh | R-01, R-02, R-04 | Gán 1 box `truck` (`truncated=yes`) và các box `person` riêng lẻ cho từng người | Người và xe tải là các thực thể độc lập; cắt mép và che khuất là hai thuộc tính tách rời. |
| `img_002.jpg` | Xe tải Renault chở 8 ô tô con trên hai tầng sàn | R-05 | Gán 1 box `truck` bao toàn bộ xe chuyên chở; theo dõi ca gán ô tô con trên sàn ở Pha 4 | Xe chở xe là một khối vận tải thống nhất; tránh làm mô hình hiểu sai là có 8 xe đang chạy lơ lửng. |
| `img_003.jpg` | Người bán hàng đứng cạnh xe đạp và sọt hàng cồng kềnh trên phố | R-02, R-09 | Gán box `person` cho người bán hàng; **không gán** xe đạp | Xe đạp không có động cơ, không nằm trong taxonomy; chỉ quan tâm người tham gia giao thông. |
| `img_004.jpg` | Hai xe ba bánh có động cơ kiểu auto rickshaw | R-03 | Gán mỗi xe 1 box `tuk_tuk`; chỉ gán `person` cho người lái nếu nhìn thấy rõ | Phân loại chính xác xe 3 bánh có động cơ, không ép vào `car` hay `motorcycle`. |
| `img_011.jpg` | Tác phẩm điêu khắc nghệ thuật lớn ở tiền cảnh, người đi bộ ở hậu cảnh | R-07, R-08 | **Không gán tượng**; gán từng `person` thật ở phía sau nếu cao $\ge 20\text{ px}$ | Tượng là vật tĩnh cố định; tránh gây lỗi phantom braking cho xe tự hành. |
| `img_015.jpg` | Ô tô lưu thông trong mưa lớn, nhiều xe bị nhòe chuyển động | R-01, R-08 | Gán xe nhận dạng được theo phần nhìn thấy; cụm mờ xa xăm dùng `ignore_region` | Mưa và nhòe chuyển động không biến vật thể rõ thành ignore; chỉ ignore vùng không thể tách. |
| `img_017.jpg` | Tượng đá công tước Wellington cưỡi ngựa đội nón chóp giao thông | R-07 | **Tuyệt đối KHÔNG gán bất kỳ box nào** (0 annotation) | Là tác phẩm điêu khắc bất động, không tham gia giao thông. |
| `img_018.jpg` | Ảnh chân dung in trên biển pano quảng cáo; ô tô thật sau hàng rào | R-01, R-07 | **Không gán người in trên biển**; gán từng box `car` thật phía sau hàng rào | Hình in 2D không phải người thật; hàng rào chỉ che khuất một phần ô tô thật. |
| `img_020.jpg` | Người lái xe và khung cảnh phản chiếu qua gương chiếu hậu | R-07 | **Không gán** ảnh phản chiếu trong gương | Hình ảnh trong gương là ảnh ảo 2D quang học, không có vật thể tại vị trí gương. |
| `img_027.jpg` | Tượng đài tưởng niệm bằng đá hoa cương tạc hình xe bán tải | R-07 | **Tuyệt đối KHÔNG gán** nhãn `truck` hay `car` | Công trình kiến trúc điêu khắc cố định trong công viên, không di chuyển. |
| `img_033.jpg` | Ma-nơ-canh trưng bày quần áo sau cửa kính ở Glasgow | R-07 | **Tuyệt đối KHÔNG gán bất kỳ box nào** (0 annotation) | Ma-nơ-canh nhân tạo trong nhà không phải con người thật tham gia giao thông. |
| `img_035.jpg` | Xe tuk-tuk chở các bao tải hàng hóa cồng kềnh ngập ngụa trên nóc | R-03, R-06 | Gán 1 box `tuk_tuk` ôm sát thân xe cơ khí; gán riêng người lái và người đi xe máy gần đó | Phân tách rõ ràng giữa cấu tạo xe và hàng hóa rời chở theo. |
| `img_038.jpg` | Bộ sưu tập mô hình xe ô tô đồ chơi trong tủ kính cửa hàng | R-07 | **Tuyệt đối KHÔNG gán bất kỳ box nào** (0 annotation) | Đồ chơi thu nhỏ trong nhà, nằm ngoài miền ứng dụng (ODD) giao thông đường phố. |

---

## 5. Quy trình Quản lý Chất lượng (QC) và Lịch sử Phiên bản

### 5.1. Các ngưỡng kỹ thuật nghiệm thu bắt buộc (từ `submission/taxonomy.json`)
- **Chiều cao tối thiểu của bounding box:** $\ge 20\text{ pixel}$.
- **Ngưỡng IoU tối thiểu để ghép cặp box (Match IoU):** $\ge 0.50$.
- **Ngưỡng IoU đồng thuận hình học theo chiều cao trung bình:**
  - Đối với box lớn ($\ge 50\text{ px}$): IoU $\ge 0.85$.
  - Đối với box nhỏ ($< 50\text{ px}$): IoU $\ge 0.70$.
- **Chỉ số đồng thuận toàn lô (IAA Target):**
  - IoU hình học trung bình toàn lô: $\ge 0.85$.
  - Tỷ lệ đồng thuận phân lớp (Class Agreement): $\ge 95\%$.
- **Tỷ lệ ảnh có lỗi tối đa cho phép khi nghiệm thu bàn giao:** $\le 2\%$.

### 5.2. Quy trình xử lý ca mơ hồ
- Khi annotator gặp trường hợp phân vân chưa thể áp dụng dứt khoát 10 quy tắc trên: Bật cờ `can_review=true` trên box đó.
- Nếu không thể vẽ được box do chất lượng ảnh quá kém: Ghi nhận thông tin vào `submission/qc/image_issues.csv` (cột: `image,annotator,region,note`).
- Người điều phối (`029`) và Reviewer bản cuối (`059`) sẽ triệu tập họp nhóm tại Pha 5 (Calibration) để chốt quyết định chính thức cho từng ca trong `calibration_decisions.csv` trước khi nghiệm thu.

### 5.3. Lịch sử thay đổi phiên bản

| Phiên bản | Ngày | Người thực hiện | Nội dung thay đổi chính | Căn cứ thực tế |
|---|---|---|---|---|
| `v0` | 2026-11-16 | Giảng viên | Bản guideline khung khởi đầu | Tài liệu starter, chưa định lượng che khuất, thiếu quy tắc cho xe 3 bánh, tượng đài và xe chở xe. |
| `v0.1-draft` | 2026-11-16 | Các thành viên (`029`, `038`, `045`, `048`, `059`) | 5 bản guideline độc lập cá nhân của từng thành viên | Khảo sát độc lập trên cả 40 ảnh tại Pha 1 trước khi họp nhóm. |
| `v1.0` | 2026-11-16 | Toàn thể Nhóm 3 (Điều phối: `029`) | - Thống nhất chuẩn hóa 8 lớp đối tượng (bổ sung chính thức `bus` và `tuk_tuk`).<br/>- Định lượng % diện tích cho `occluded_level`.<br/>- Xác lập hệ thống 10 quy tắc vẽ box kỹ thuật (`R-01` $\rightarrow$ `R-10`).<br/>- Thống nhất bảng xử lý 13 ca khó điển hình từ 40 ảnh. | Bản đồng thuận chính thức sau cuộc họp Pha 2, khóa làm căn cứ gán nhãn độc lập và đo IAA. |
