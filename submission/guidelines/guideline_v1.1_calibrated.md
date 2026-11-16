# Guideline gán nhãn sau hiệu chuẩn — v1.1 — Nhóm K5-L23-Nhom3-GuidelineQC

- Phiên bản: `v1.1-calibrated` (Bản hiệu chuẩn chính thức sau buổi họp Pha 5 đối chiếu kết quả đo IAA)
- Ngày hiệu lực: 2026-11-16
- Nhóm biên soạn: K5-L23-Nhom3-GuidelineQC
- Người chốt hiệu chuẩn: `029` (Phạm Lam Anh — Điều phối) & `059` (Nguyễn Minh Đức — Reviewer bản cuối)
- Thành viên tham gia: `038`, `045`, `048`

---

## 1. Mục tiêu và bối cảnh sử dụng

Tài liệu này xác lập bộ quy tắc chuẩn hóa sau hiệu chuẩn (Calibration) của nhóm cho tập dữ liệu ảnh đường phố 2D (40 ảnh), phục vụ mô hình thị giác máy tính phát hiện đối tượng (Object Detection 2D) ứng dụng trong ADAS và xe tự hành:

- **Mục tiêu kỹ thuật:** Xác định chính xác vị trí không gian (bounding box 2D ôm sát phần nhìn thấy), phân loại chuẩn xác 8 lớp đối tượng giao thông thực tế và gán đúng các thuộc tính trạng thái che khuất, cắt mép.
- **Rủi ro an toàn thực tế nếu gán sai:**
  - *Bỏ sót đối tượng (False Negative):* Bỏ sót người đi bộ, người đi xe máy hoặc xe cộ đang lưu thông sẽ khiến hệ thống không kích hoạt cảnh báo va chạm hoặc phanh khẩn cấp tự động (AEB), dẫn đến tai nạn thảm khốc.
  - *Gán sai kích thước / ranh giới:* Vẽ bounding box quá lỏng hoặc quá chật, đặc biệt với các phương tiện cồng kềnh, khiến module lập quỹ đạo (Motion Planning) tính toán sai khoảng cách an toàn và thời gian va chạm (TTC).
  - *Gán nhầm vật thể tĩnh phi thực thể (False Positive):* Gán các bức tượng nghệ thuật, ma-nơ-canh trong cửa hàng, hình người trên biển quảng cáo hoặc ảnh phản chiếu qua gương chiếu hậu thành đối tượng thật sẽ gây ra hiện tượng phanh gấp ngoài ý muốn (*phantom braking*).

---

## 2. Đặc tả Lớp và Thuộc tính

Bộ nhãn thống nhất tuân thủ nghiêm ngặt theo định nghĩa trong `submission/taxonomy.json` phiên bản `v1.0` (giữ nguyên schema và ngưỡng rules).

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

---

### 2.2. Danh mục Thuộc tính đối tượng (Attributes)

| Thuộc tính | Kiểu dữ liệu | Giá trị hợp lệ | Ý nghĩa và ngưỡng định lượng chuẩn (Hiệu chuẩn v1.1) |
|---|---|---|---|
| `occluded_level` | Select | `"0"`, `"1"`, `"2"`, `"3"` | Mức độ diện tích bề mặt nhìn thấy bị **vật thể khác che khuất**:<br/>- `"0"`: Không bị che (0% diện tích bị che).<br/>- `"1"`: Che khuất nhẹ (từ 1% đến 30% diện tích bị che).<br/>- `"2"`: Che khuất vừa/nặng (từ 31% đến 70% diện tích bị che).<br/>- `"3"`: Bị che khuất rất nặng (> 70% diện tích bị che nhưng vẫn đủ đặc trưng nhận dạng chắc chắn). |
| `truncated` | Select | `"no"`, `"yes"` | Vật thể bị **cắt cụt bởi mép bức ảnh**:<br/>- `"no"`: Toàn bộ các bộ phận của vật thể nằm trọn trong khung hình.<br/>- `"yes"`: Bất kỳ bộ phận nào của vật thể bị ít nhất một trong bốn mép ảnh cắt qua (box tiếp xúc mép ảnh $\le 2\text{ px}$). |
| `can_review` | Checkbox | `false`, `true` | Cờ đánh dấu ca mơ hồ: Trong bộ nhãn bàn giao cuối cùng sau calibration, **toàn bộ cờ `can_review` đều được giải tỏa về `false`** theo đúng quyết định của nhóm. |

---

## 3. Hệ thống 10 Quy tắc vẽ Bounding Box (Rules)

- **R-01 — Bounding Box ôm sát phần nhìn thấy (Tight Box on Visible Part):**
  Box hình chữ nhật (trục song song với biên ảnh) phải tiếp xúc chính xác 4 điểm cực biên của phần vật thể thực tế nhìn thấy được. Tuyệt đối không phán đoán vẽ bù phần bị che khuất vào khoảng trống không nhìn thấy.

- **R-02 — Tách riêng Người và Phương tiện hở (Person and Open Vehicle Separation):**
  Với xe máy (`motorcycle`) hoặc phương tiện hở:
  - Vẽ **1 box `person`** cho từng người trên xe: ôm từ đỉnh đầu/mũ bảo hiểm xuống hết chân người.
  - Vẽ **1 box `motorcycle`** cho phương tiện: ôm trọn khung xe, bánh xe tiếp đất và tay lái.
  - Vùng giao thoa giữa người và xe không trừ ra; cả hai box đều bao trùm phần của mình.

- **R-03 — Người ngồi trong khoang kín (Enclosed Cabins):**
  - Người ngồi trong ô tô con (`car`), xe tải (`truck`), xe van (`van`) kín cửa kính: **Không gán box `person`**.
  - Người ngồi trong khoang hở của xe ba bánh (`tuk_tuk`) hoặc thùng xe bán tải hở: Chỉ gán `person` nếu nhìn thấy rõ phần thân trên và phân biệt được rõ ràng với xe.

- **R-04 — Phân loại xe thùng hở và xe thân kín (Pickup vs Van vs Truck):**
  - Xe bán tải (pickup) có thùng hàng hở phía sau tách rời cabin $\rightarrow$ gán `truck` (`img_001.jpg`, `img_012.jpg`).
  - Xe thùng hộp liền khối với cabin $\rightarrow$ gán `van` (`img_005.jpg`, `img_016.jpg`).

- **R-05 — Xe chuyên dụng chở ô tô (Car Carrier - Hiệu chuẩn `img_002.jpg`):**
  - Toàn bộ khối xe tải đầu kéo và giàn rơ-moóc chuyên dụng chở 8 ô tô con được gán **1 box `truck` lớn** đại diện cho phương tiện vận tải siêu trường.
  - Các xe ô tô con trên sàn xe là hàng hóa vận chuyển cố định theo xe; việc thống nhất dùng 1 box `truck` giúp tránh báo động giả cho bộ lập quỹ đạo lái xe.

- **R-06 — Hàng hóa chở trên phương tiện (`img_029.jpg`, `img_035.jpg`):**
  Bounding box của phương tiện (`motorcycle`, `tuk_tuk`) ôm trọn kết cấu cơ khí của xe. Các kiện hàng xếp quá khổ trên nóc không kéo dãn box phương tiện.

- **R-07 — Tuyệt đối loại trừ vật thể phi thực thể (Non-real & Static Artworks):**
  Tuyệt đối **KHÔNG gán bất kỳ box nào** cho:
  - Tượng đài nghệ thuật danh nhân (`img_017.jpg`), tượng đài xe bán tải bằng đá (`img_027.jpg`).
  - Ma-nơ-canh trong cửa hàng (`img_033.jpg`).
  - Ảnh chân dung, tranh in 2D trên pano quảng cáo (`img_018.jpg`).
  - Ảnh phản chiếu qua gương chiếu hậu (`img_020.jpg`).
  - Mô hình đồ chơi trong tủ kính (`img_038.jpg`).

- **R-08 — Ngưỡng kích thước tối thiểu (Min Box Height Threshold):**
  - Không gán box riêng lẻ cho bất kỳ đối tượng nào có chiều cao phần nhìn thấy nhỏ hơn **20 pixel**.
  - Vùng hậu cảnh xa có nhiều vật thể nhỏ dày đặc thì khoanh vùng `ignore_region` bao phủ (`img_025.jpg`).

- **R-09 — Phương tiện ngoài taxonomy (Out-of-taxonomy Vehicles):**
  Xe đạp không động cơ, xe xích lô đạp chân, xe kéo tay không gán nhãn phương tiện; chỉ gán người (`person`) nếu cao $\ge 20\text{ px}$.

- **R-10 — Xe mang camera / Xe chủ (Ego Vehicle Exclusion):**
  Không gán nắp ca-pô hoặc gương của xe mang camera ở sát mép dưới ảnh.

---

## 4. Ca khó và Tình huống biên sau hiệu chuẩn

Toàn bộ 13 ca biên từ `img_001.jpg` đến `img_040.jpg` đã được giải quyết triệt để và đồng thuận 100% trong `calibration_decisions.csv` (xem chi tiết tại bảng quyết định hiệu chuẩn).

---

## 5. QC và Lịch sử Phiên bản

### 5.1. Ngưỡng kỹ thuật nghiệm thu
- Chiều cao box tối thiểu: $\ge 20\text{ px}$.
- Ngưỡng ghép cặp box IoU: $\ge 0.50$.
- IoU đồng thuận: $\ge 0.85$ (vật lớn $\ge 50\text{ px}$) và $\ge 0.70$ (vật nhỏ $< 50\text{ px}$).
- IAA toàn lô sau hiệu chuẩn: IoU $\ge 0.85$, Class Agreement $\ge 95\%$.
- Tỷ lệ lỗi toàn lô tối đa khi nghiệm thu: $\le 2\%$.

### 5.2. Lịch sử thay đổi

| Phiên bản | Ngày | Người thực hiện | Nội dung thay đổi |
|---|---|---|---|
| `v1.0` | 2026-11-16 | Nhóm 3 (Điều phối `029`) | Bản đồng thuận đầu tiên khóa cho lượt gán độc lập. |
| `v1.1-calibrated` | 2026-11-16 | Nhóm 3 (Chốt: `029`, `059`) | Hiệu chuẩn sau IAA: Chốt quy tắc xe chở xe `img_002.jpg`, giải tỏa cờ `can_review`, tinh chỉnh ranh giới box ôm sát phần nhìn thấy và đồng thuận mức `occluded_level`. |
