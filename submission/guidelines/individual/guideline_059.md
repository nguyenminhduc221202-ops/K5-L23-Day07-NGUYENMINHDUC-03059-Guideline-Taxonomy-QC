# Guideline gán nhãn — v0.1-draft — 059

- Phiên bản: `v0.1-draft`
- Ngày: 2026-11-16
- Người viết: Nguyễn Minh Đức - 059 (Reviewer bản cuối, K5-L23-Nhom3-GuidelineQC)

---

## 1. Mục tiêu

Tài liệu này xác lập đặc tả kỹ thuật gán nhãn bounding box 2D độc lập của thành viên **059-Nguyễn Minh Đức** (Pha 1 — Checkpoint CP1), phục vụ xây dựng tập dữ liệu huấn luyện và kiểm thử cho mô hình thị giác máy tính phát hiện đối tượng giao thông đường phố (Object Detection 2D) ứng dụng trong xe tự hành và ADAS:

- **Mục tiêu mô hình:** Định vị không gian chiếm dụng vật lý (Spatial Occupancy) và phân loại chính xác các tác nhân giao thông lưu thông trên đường.
- **Rủi ro an toàn thực tế:**
  - *Va quẹt do tính thiếu kích thước:* Nếu xe chở hàng cồng kềnh (`img_035.jpg`) chỉ vẽ ôm thân xe mà bỏ qua khối hàng phình to bên ngoài, xe tự hành sẽ tính toán khoảng cách an toàn không đủ rộng, dẫn đến va quẹt hàng hóa khi vượt hoặc chuyển làn.
  - *Ảo giác chướng ngại vật (Ghost Objects):* Nếu gán rời rạc 8 chiếc ô tô con trên sàn xe chở xe chuyên dụng (`img_002.jpg`), module lập quỹ đạo (Motion Planner) có thể nhận định sai là có 8 chiếc xe độc lập đang bay lơ lửng trên làn đường.
  - *Phanh gấp ngoài ý muốn (Phantom Braking):* Gán các bức tượng điêu khắc nghệ thuật (`img_017.jpg`, `img_027.jpg`) hay ma-nơ-canh (`img_033.jpg`) thành người/xe thật sẽ khiến xe tự hành nhận diện sai chướng ngại vật tĩnh trên vỉa hè và phanh gấp bất ngờ.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp đối tượng (Classes)

| Lớp | Định nghĩa (một câu) | Ví dụ thực tế từ bộ ảnh | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô con từ 4 đến 9 chỗ ngồi chở người (sedan, SUV, hatchback, taxi). | Xe ô tô lưu thông (`img_024.jpg`, `img_040.jpg`). | Xe bán tải lớn hở thùng (thuộc `truck`), xe mô hình đồ chơi (`img_038.jpg`). |
| `truck` | Xe tải chở hàng, xe đầu kéo kéo rơ-moóc, xe chuyên dụng chở phương tiện. | Xe chuyên chở ô tô (`img_002.jpg`), xe tải nhỏ giao hàng (`img_036.jpg`). | Tượng xe bán tải bằng đá hoa cương (`img_027.jpg`), xe tuk-tuk (`img_004.jpg`). |
| `van` | Xe ô tô thân kín liền khối cỡ nhỏ/vừa có khoang lái liền khoang hàng/khoang khách. | Xe van giao hàng bưu chính (`img_005.jpg`). | Xe buýt lớn chở khách công cộng, xe con 7 chỗ. |
| `bus` | Xe ô tô khách cỡ lớn từ 17 chỗ ngồi trở lên phục vụ vận tải công cộng hoặc du lịch. | Xe buýt công cộng (`img_009.jpg`, `img_031.jpg`). | Xe van cỡ nhỏ, xe tải thùng kín. |
| `tuk_tuk` | Xe ba bánh gắn động cơ cơ giới chở khách/hàng nhẹ (auto rickshaw). | Xe ba bánh tuk-tuk (`img_004.jpg`, `img_035.jpg`). | Xe mô tô hai bánh (`motorcycle`), xe xích lô đạp chân (`img_006.jpg`). |
| `motorcycle` | Xe hai bánh gắn động cơ cơ giới (xe máy số, xe tay ga, mô tô phân khối lớn). | Xe máy di chuyển trên đường (`img_005.jpg`, `img_008.jpg`, `img_026.jpg`). | Xe đạp không động cơ, xe xích lô đạp chân (`img_006.jpg`, `img_009.jpg`). |
| `person` | Con người sinh học bằng xương bằng thịt còn sống (người đi bộ, người lái xe, hành khách). | Người đi bộ qua đường (`img_014.jpg`), người điều khiển xe máy (`img_026.jpg`). | Tượng danh nhân (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), hình người in trên pano quảng cáo (`img_018.jpg`). |
| `ignore_region` | Vùng hình chữ nhật bao phủ khu vực quá đông đúc, mờ nhòe do mưa/chuyển động hoặc vật nhỏ dưới 20 px. | Cụm người xe ở phía xa bờ biển (`img_025.jpg`), cụm mờ nhòe trong mưa (`img_015.jpg`). | Đối tượng rõ nét, kích thước đủ lớn ở tiền cảnh. |

*Xử lý ca ngoài taxonomy:*
- Xe tuk-tuk ba bánh (`img_004.jpg`, `img_035.jpg`): Đề xuất bổ sung lớp `tuk_tuk` chính thức ở Pha 2.
- Xe buýt (`img_009.jpg`, `img_031.jpg`): Đề xuất bổ sung lớp `bus` chính thức.
- Xe thô sơ không động cơ (xích lô, xe đẩy bán hàng rong `img_003.jpg`, `img_010.jpg`): Không gán nhãn phương tiện, chỉ gán người bán hàng (`person`).

---

### 2.2. Thuộc tính đối tượng (Attributes)

| Thuộc tính | Kiểu / Các mức cố định | Ý nghĩa | Ngưỡng định lượng cụ thể |
|---|---|---|---|
| `occluded_level` | Select: `0`, `1`, `2`, `3` | Mức độ diện tích bề mặt vật thể bị vật khác che khuất | - `0`: Không bị che khuất (0% diện tích bị che).<br/>- `1`: Che khuất nhẹ (từ 1% đến 30% diện tích bị che).<br/>- `2`: Che khuất vừa/nặng (từ 31% đến 70% diện tích bị che).<br/>- `3`: Bị che khuất rất nặng (> 70% diện tích bị che nhưng vẫn nhận dạng chắc chắn). |
| `truncated` | Select: `no`, `yes` | Vật thể bị cắt cụt do nằm tràn mép bức ảnh | - `no`: Toàn bộ các bộ phận của vật thể nằm trọn trong khung hình.<br/>- `yes`: Vật thể bị ít nhất một trong bốn mép ảnh cắt qua bất kỳ bộ phận nào. |
| `can_review` | Checkbox: `false`, `true` | Đánh dấu ca khó, cần reviewer hoặc cả nhóm hội ý | - `false`: Tự tin gán theo quy tắc.<br/>- `true`: Trường hợp biên gây tranh cãi, chưa thống nhất. |

---

## 3. Quy tắc vẽ Bounding Box

1. **Khái niệm biên vật thể:**
   - Box phải tiếp xúc 4 điểm cực biên (trên, dưới, trái, phải) của phần vật thể nhìn thấy được. Không vẽ đoán phần bị che khuất.
2. **Người và xe máy (`motorcycle` & `person`):**
   - Tách làm 2 box riêng biệt: 1 box `person` ôm từ đỉnh đầu/mũ bảo hiểm xuống chân; 1 box `motorcycle` ôm trọn khung xe, bánh xe và ghi-đông. Vùng giao thoa giữa người và xe không trừ ra.
3. **Xe chuyên dụng chở ô tô (Car carrier - `img_002.jpg`):**
   - **Quy tắc của 059:** Chỉ gán **1 box `truck` duy nhất** bao toàn bộ đầu kéo, rơ-moóc và toàn bộ các xe ô tô con trên hai tầng sàn.
   - *Lý do:* Toàn bộ tổ hợp này di chuyển như một chướng ngại vật thống nhất. Các xe ô tô con trên sàn đóng vai trò là hàng hóa tĩnh, không phải phương tiện đang độc lập tham gia giao thông.
4. **Hàng hóa chở cồng kềnh / Quá khổ (`img_035.jpg`, `img_029.jpg`):**
   - **Quy tắc của 059:** Bounding box của phương tiện phải **kéo rộng bao trùm toàn bộ cả thân xe lẫn khối hàng hóa/bao tải** chất trên nóc và hai bên.
   - *Lý do:* Hệ thống tự hành cần biết ranh giới không gian chiếm dụng thực tế (Spatial Occupancy) để giữ khoảng cách an toàn, tránh va quẹt vào khối hàng nhô ra ngoài.
5. **Vật thể phi thực thể (Tượng, Ma-nơ-canh, Tranh ảnh, Phản chiếu):**
   - **Tuyệt đối không gán:**
     - Tượng đài nghệ thuật danh nhân (`img_017.jpg`).
     - Tượng xe bán tải bằng đá (`img_027.jpg`).
     - Ma-nơ-canh quần áo trong cửa hàng (`img_033.jpg`).
     - Ảnh người/xe trên biển quảng cáo pano (`img_018.jpg`).
     - Ảnh phản chiếu của người/xe qua gương chiếu hậu (`img_020.jpg`).
6. **Ngưỡng kích thước tối thiểu:**
   - Chiều cao $< 20\text{ px}$: Bỏ qua không gán hoặc gom vào `ignore_region` nếu là đám đông dày đặc ở xa.

---

## 4. Ca khó và tình huống biên từ 40 ảnh

| Mã ảnh | Tình huống cụ thể | Mã Rule đề xuất | Quyết định cá nhân của 059 | Căn cứ và lý giải | Phân loại |
|---|---|---|---|---|---|
| `img_002.jpg` | Xe tải Renault chở 8 ô tô con hai tầng sàn | R-01 | Chỉ gán **1 box `truck` to** bao toàn bộ | Các xe con là hàng hóa tĩnh trên sàn xe tải. Gán rời 8 xe con sẽ làm mô hình hiểu sai là có 8 xe đang chạy độc lập trên đường. | Quyết định cá nhân |
| `img_035.jpg` | Xe tuk-tuk chở đống bao tải ngập ngụa trên nóc | R-03 | Gán 1 box `tuk_tuk` **bao cả xe lẫn bao tải** | Khối hàng chuyển động cùng xe tạo thành thể tích chiếm dụng thực tế, bắt buộc bao trùm để tránh va quẹt. | Quyết định cá nhân |
| `img_017.jpg` | Tượng đá Wellington cưỡi ngựa đội nón giao thông | R-02 | **Không gán box nào** (0 annotation) | Là tác phẩm điêu khắc nghệ thuật cố định, không phải đối tượng tham gia giao thông; gán vào sẽ gây phantom braking. | Loại theo quy tắc |
| `img_027.jpg` | Tượng đài xe bán tải đúc bằng đá hoa cương | R-02 | **Không gán box nào** | Công trình tưởng niệm cố định trong công viên, không có động cơ hay quỹ đạo di chuyển. | Loại theo quy tắc |
| `img_020.jpg` | Xe và cầu vồng phản chiếu trong gương chiếu hậu | R-04 | **Không gán** ảnh trong gương | Là ảnh ảo 2D quang học, không có chướng ngại vật thực tại vị trí bề mặt gương. | Loại theo quy tắc |
| `img_033.jpg` | Ma-nơ-canh trưng bày quần áo sau cửa kính | R-05 | **Không gán** nhãn `person` | Búp bê vô tri trong nhà, nằm ngoài phạm vi thực thể sinh học cần phát hiện. | Loại theo quy tắc |

---

## 5. QC và quản lý phiên bản

- Ngưỡng chất lượng theo quy định của lab:
  - Tỷ lệ trùng khớp hình học (IoU) trung bình giữa các người gán: $\ge 0.85$.
  - Tỷ lệ đồng thuận lớp nhãn (Class Agreement): $\ge 95\%$.
  - Tỷ lệ ảnh có lỗi toàn lô khi nghiệm thu: $\le 2\%$.
- Lịch sử cập nhật:

| Phiên bản | Thay đổi chính | Căn cứ thực tế |
|---|---|---|
| `v0` | Bản khởi đầu từ giảng viên | Quá sơ sài, thiếu định lượng occluded, chưa có lớp `bus`/`tuk_tuk`. |
| `v0.1-draft` | Bổ sung định lượng %, quy tắc 1 box cho xe chở xe, quy tắc bao trùm hàng hóa quá khổ, loại trừ tượng/ma-nơ-canh | Bản nháp cá nhân độc lập của thành viên 059 chuẩn bị cho họp Pha 2. |
