# Guideline gán nhãn — v0.1-draft — 029

- Phiên bản: `v0.1-draft`
- Ngày: 2026-11-16
- Người viết: Phạm Lâm Anh - 029 (Điều phối nhóm K5-L23-Nhom3-GuidelineQC)

---

## 1. Mục tiêu

Tài liệu này xác lập hướng dẫn gán nhãn bounding box 2D độc lập của thành viên **029-Phạm Lâm Anh** (Pha 1 — Checkpoint CP1), phục vụ việc chuẩn bị dữ liệu huấn luyện và đánh giá mô hình thị giác máy tính phát hiện đối tượng (Object Detection 2D) trong hệ thống hỗ trợ lái xe nâng cao (ADAS) và xe tự hành:

- **Mục tiêu mô hình:** Phát hiện chính xác vị trí, kích thước bao ngoài và phân loại các thực thể tham gia giao thông thực tế trên đường phố trong mọi điều kiện ánh sáng, mật độ và góc chụp.
- **Rủi ro an toàn:**
  - *Bỏ sót (False Negative):* Bỏ sót người đi bộ hoặc người đi xe máy có nguy cơ dẫn đến va chạm gây tai nạn nghiêm trọng.
  - *Nhận diện sai lớp / kích thước:* Gán sai xe tải (`truck`) thành ô tô con (`car`) hoặc vẽ bounding box quá lỏng/quá chặt khiến hệ thống tính toán sai khoảng cách an toàn và thời gian va chạm (TTC - Time to Collision).
  - *Gán nhầm vật thể tĩnh (False Positive):* Gán các bức tượng nghệ thuật, ma-nơ-canh, biển quảng cáo in hình người hoặc hình ảnh phản chiếu qua gương/kính thành người/xe thật sẽ gây hiện tượng phanh gấp ngoài ý muốn (phantom braking), nguy hiểm cho các xe phía sau.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp đối tượng (Classes)

| Lớp | Định nghĩa (một câu) | Ví dụ thực tế từ bộ ảnh | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô con từ 4 đến 9 chỗ ngồi, xe sedan, SUV, hatchback phục vụ chở người cá nhân/gia đình/taxi. | Xe ô tô con chạy trên phố (`img_018.jpg`, `img_024.jpg`, `img_040.jpg`). | Xe bán tải chở hàng (thuộc `truck`), xe đồ chơi mô hình trong tủ kính (`img_038.jpg`). |
| `truck` | Xe cơ giới chở hàng hóa, xe đầu kéo kéo rơ-moóc, xe có cabin và thùng hàng chuyên dụng riêng biệt. | Xe chở ô tô (`img_002.jpg`), xe tải nhỏ chở hàng (`img_036.jpg`). | Tượng xe bán tải bằng đá hoa cương (`img_027.jpg`), xe tuk-tuk ba bánh (`img_004.jpg`). |
| `van` | Xe ô tô thân kín dạng hộp liền khối cỡ nhỏ/vừa, dùng để chở người hoặc chuyển phát hàng hóa. | Xe van giao hàng liền khung (`img_005.jpg`). | Xe buýt lớn chở khách (thuộc `bus`), xe ô tô con 7 chỗ (thuộc `car`). |
| `bus` | Xe ô tô khách cỡ lớn từ 17 chỗ ngồi trở lên, phục vụ vận tải hành khách công cộng hoặc du lịch. | Xe buýt chạy trên đường (`img_009.jpg`, `img_031.jpg`). | Xe van cỡ nhỏ, xe tải thùng kín. |
| `tuk_tuk` | Xe ba bánh gắn động cơ cơ giới chở khách hoặc chở hàng nhẹ, phổ biến ở Nam Á và Đông Nam Á (auto rickshaw). | Xe ba bánh tuk-tuk (`img_004.jpg`, `img_035.jpg`). | Xe mô tô 2 bánh (`motorcycle`), xe xích lô đạp chân (`img_006.jpg`). |
| `motorcycle` | Xe hai bánh gắn động cơ cơ giới (xe máy số, xe tay ga, xe phân khối lớn). | Xe máy di chuyển hoặc đỗ trên đường (`img_006.jpg`, `img_008.jpg`, `img_021.jpg`, `img_030.jpg`). | Xe đạp không gắn động cơ, xe xích lô đạp bằng chân (`img_006.jpg`, `img_009.jpg`). |
| `person` | Con người thật bằng xương bằng thịt đang sống (người đi bộ, người điều khiển phương tiện, người ngồi sau xe). | Người đi bộ qua đường (`img_003.jpg`, `img_014.jpg`), người lái xe máy (`img_026.jpg`). | Tượng đài điêu khắc danh nhân (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), hình người in trên pano quảng cáo (`img_018.jpg`). |
| `ignore_region` | Vùng hình chữ nhật bao phủ khu vực có mật độ quá đông đúc, mờ nhòe xa xăm hoặc vật thể quá nhỏ không thể phân định chính xác. | Cụm người/xe ở hậu cảnh xa tít cao dưới 20 px (`img_025.jpg`). | Các vật thể rõ nét, kích thước đủ lớn ở tiền cảnh và trung cảnh. |

*Xử lý các thực thể ngoài taxonomy ban đầu:*
- Xe tuk-tuk ba bánh (`img_004.jpg`, `img_035.jpg`): Không gán gượng ép vào `car` hay `motorcycle`; đề xuất nhóm chính thức chốt lớp `tuk_tuk` ở Pha 2.
- Xe buýt công cộng (`img_009.jpg`, `img_031.jpg`): Đề xuất thống nhất đưa lớp `bus` vào taxonomy chính thức.
- Phương tiện thô sơ không động cơ (xích lô đạp chân `img_006.jpg`, xe đạp): Không gán nhãn vật thể; nếu nằm chắn giữa luồng giao thông thì đánh dấu `ignore_region`.

---

### 2.2. Thuộc tính đối tượng (Attributes)

| Thuộc tính | Kiểu / Các mức cố định | Ý nghĩa | Ngưỡng định lượng cụ thể |
|---|---|---|---|
| `occluded_level` | Select: `0`, `1`, `2`, `3` | Mức độ diện tích bề mặt vật thể bị vật khác che khuất | - `0`: Không bị che khuất (0% diện tích bị che).<br/>- `1`: Che khuất nhẹ (từ 1% đến 30% diện tích nhìn thấy bị che).<br/>- `2`: Che khuất vừa/nặng (từ 31% đến 70% diện tích bị che).<br/>- `3`: Bị che khuất rất nặng (> 70% diện tích bị che nhưng vẫn còn đủ đặc trưng nhận dạng chắc chắn). |
| `truncated` | Select: `no`, `yes` | Vật thể bị cắt cụt do nằm ở mép bức ảnh | - `no`: Toàn bộ các bộ phận của vật thể nằm trọn trong khung hình.<br/>- `yes`: Vật thể bị mép ảnh cắt qua bất kỳ bộ phận nào. |
| `can_review` | Checkbox: `false`, `true` | Đánh dấu ca mơ hồ / trường hợp biên cần reviewer hoặc nhóm thảo luận lại | - `false`: Tự tin gán theo đúng quy tắc.<br/>- `true`: Trường hợp biên, khó xác định lớp hoặc ranh giới box. |

---

## 3. Quy tắc vẽ Bounding Box

1. **Ôm sát phần nhìn thấy (Tight Box on Visible Area):**
   - Bounding box phải tiếp xúc chính xác 4 điểm cực biên (trên, dưới, trái, phải) của **phần vật thể thực tế nhìn thấy được**.
   - Tuyệt đối không phán đoán vẽ bù phần bị che khuất vào khoảng trống không nhìn thấy.

2. **Người điều khiển và phương tiện hở (Xe máy / Mô tô):**
   - **Tách riêng thành 2 box độc lập:**
     - 1 box `person` cho người điều khiển (và người ngồi sau xe): Ôm từ đỉnh đầu/mũ bảo hiểm xuống hết bàn chân.
     - 1 box `motorcycle` cho xe máy: Ôm trọn khung xe, ghi-đông, đèn xe và hai bánh xe tiếp đất.
   - Vùng giao nhau giữa thân người và thân xe không trừ ra mà cả hai box đều bao trùm phần của mình.

3. **Xe chuyên chở phương tiện khác (Car carrier - `img_002.jpg`):**
   - Xe tải chuyên chở hai tầng: Quyết định của tôi tại Pha 1 là **vẽ duy nhất 1 box `truck` lớn** bao trùm toàn bộ xe tải đầu kéo và toàn bộ giàn xe ô tô chở trên rơ-moóc.
   - Các xe ô tô con trên sàn xe: Xem chúng là hàng hóa chuyên chở cố định của xe tải, không vẽ bóc tách lẻ từng box `car` để tránh gây nhiễu cho mô hình về kích thước tổng thể của xe siêu trường siêu trọng. (Đưa ca này ra thảo luận với nhóm ở Pha 2).

4. **Hàng hóa chở cồng kềnh trên xe (`img_029.jpg`, `img_035.jpg`):**
   - Box của phương tiện (`motorcycle`, `tuk_tuk`) chỉ ôm trọn kết cấu cơ khí của xe; **không kéo rộng box** để bao các bao tải, kiện hàng cồng kềnh xếp nhô cao vượt khung xe.

5. **Vật thể phi thực thể (Tượng, Ma-nơ-canh, Tranh ảnh, Phản chiếu):**
   - **Tuyệt đối không gán:**
     - Tượng đài điêu khắc đá/đồng (`img_017.jpg`, `img_027.jpg`).
     - Ma-nơ-canh trưng bày quần áo (`img_033.jpg`).
     - Hình ảnh in trên áp phích, biển quảng cáo pano (`img_018.jpg`).
     - Hình ảnh phản chiếu của người/xe qua gương chiếu hậu hoặc bề mặt kính (`img_020.jpg`).

6. **Ngưỡng kích thước tối thiểu (Min Box Height):**
   - Không gán box riêng lẻ cho các vật thể có chiều cao dưới **20 pixel**.
   - Nếu khu vực xa tập trung nhiều người/xe nhỏ hơn 20 px nhưng nằm trong hành lang di chuyển, đánh dấu một vùng `ignore_region`.

---

## 4. Ca khó và tình huống biên từ 40 ảnh

| Mã ảnh | Tình huống cụ thể | Mã Rule đề xuất | Quyết định gán của 029 | Căn cứ và lý do | Phân loại |
|---|---|---|---|---|---|
| `img_002.jpg` | Xe tải Renault chở 8 xe ô tô con trên hai tầng sàn | R-01 | Gán duy nhất **1 box `truck` to** bao toàn bộ xe và giàn ô tô trên sàn (không gán lẻ `car`) | Xem toàn bộ các ô tô con là hàng hóa chuyên chở trên rơ-moóc; box cần ôm trọn khối xe tải chở hàng để xe tự hành nhận diện đây là một khối phương tiện siêu trường. | Cần hỏi reviewer |
| `img_017.jpg` | Tượng đá công tước Wellington cưỡi ngựa đội nón chóp giao thông | R-02 | **Không gán** bất kỳ box nào | Là tượng đài nghệ thuật cố định, không có ý thức tham gia giao thông. Gán vào sẽ gây lỗi phanh gấp. | Loại theo quy tắc |
| `img_020.jpg` | Người lái xe và xe nhìn qua gương chiếu hậu | R-04 | **Không gán** ảnh phản chiếu trong gương | Hình ảnh trong gương là ảnh ảo 2D quang học, không tồn tại vật thể thực ở vị trí tọa độ của gương. | Loại theo quy tắc |
| `img_027.jpg` | Tượng đài bằng đá hoa cương tạc hình chiếc xe bán tải | R-02 | **Không gán** nhãn `truck` hay `car` | Là công trình điêu khắc bất động trong khuôn viên, không có quỹ đạo chuyển động. | Loại theo quy tắc |
| `img_035.jpg` | Xe ba bánh tuk-tuk chở bao tải hàng hóa ngập ngụa ở Hyderabad | R-03 | Gán 1 box `tuk_tuk`, box chỉ ôm thân xe | Xe 3 bánh là lớp riêng biệt cần thêm. Hàng hóa chất cao không thuộc kích thước tiêu chuẩn của xe. | Cần hỏi reviewer |
| `img_038.jpg` | Mô hình xe ô tô đồ chơi xếp trong tủ kính | R-05 | **Không gán** nhãn `car` | Xe mô hình đồ chơi kích thước siêu nhỏ trong nhà, nằm ngoài miền ứng dụng đường phố (ODD). | Loại theo quy tắc |

---

## 5. QC và quản lý phiên bản

### 5.1. Ngưỡng chất lượng áp dụng
- **Độ trùng khớp hình học (IoU) trung bình giữa 2 annotator:** $\ge 0.85$ (đối với box $\ge 50\text{ px}$) và $\ge 0.70$ (đối với box $< 50\text{ px}$).
- **Độ đồng thuận phân lớp (Class Agreement):** $\ge 95\%$.
- **Tỉ lệ lỗi tối đa trên toàn lô ảnh khi nghiệm thu:** $\le 2\%$.

### 5.2. Lịch sử thay đổi

| Phiên bản | Người sửa | Nội dung thay đổi | Căn cứ thực tế |
|---|---|---|---|
| `v0` | Giảng viên | Bộ guideline khung ban đầu | Bản khởi tạo còn mơ hồ về xe 3 bánh, tượng, xe chở xe và ngưỡng thuộc tính. |
| `v0.1-draft` | 029 | - Đề xuất bổ sung lớp `bus` và `tuk_tuk`.<br/>- Lượng hóa ngưỡng `occluded_level` theo % diện tích.<br/>- Xác lập quy tắc tách người/xe máy, quy tắc xe chở xe, quy tắc loại tượng/ảnh ảo.<br/>- Đưa ra phân tích 6 ca khó tiêu biểu từ 40 ảnh. | Khảo sát độc lập trên 40 ảnh tại Pha 1 trước khi họp chốt v1.0 nhóm. |
