# Guideline gán nhãn — v0.1-draft — 038

- Phiên bản: `v0.1-draft` (theo taxonomy `v1.0`)
- Ngày: 2026-11-16
- Người viết / duyệt: Thành viên 038 / reviewer cuối 059

## 1. Mục tiêu

Hướng dẫn gán bounding box 2D cho người và phương tiện trong ảnh đường phố, phục vụ huấn luyện và đánh giá mô hình phát hiện đối tượng cho hệ thống hỗ trợ lái xe và xe tự hành. Nhãn phải xác định đúng lớp, vị trí và phần nhìn thấy của từng đối tượng.

Bỏ sót người hoặc phương tiện, nhầm lớp xe, hoặc vẽ box sai kích thước có thể làm mô hình ước lượng sai vật cản và khoảng cách an toàn. Gán người/xe trong ảnh quảng cáo, tượng hoặc hình phản chiếu thành vật thể thật có thể gây cảnh báo hay phanh không cần thiết.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con từ 4–9 chỗ, gồm sedan, SUV và hatchback. | Ô tô con trên phố hoặc trong bãi xe (`img_015.jpg`, `img_018.jpg`). | Xe bán tải có thùng chở hàng (`truck`); xe đồ chơi (`img_038.jpg`). |
| `truck` | Xe cơ giới chở hàng có cabin và thùng/chở hàng riêng, gồm xe tải, đầu kéo và bán tải có thùng. | Bán tải chở hàng (`img_001.jpg`), xe tải giao hàng (`img_007.jpg`). | Ô tô con (`car`), xe buýt chở khách (`bus`), tuk-tuk (`tuk_tuk`). |
| `van` | Xe thân kín dạng hộp liền khối cỡ nhỏ hoặc vừa, dùng chở người hay hàng. | Xe giao hàng thân kín cỡ nhỏ/vừa. | Xe buýt lớn (`bus`), xe tải có cabin và thùng hàng tách biệt (`truck`). |
| `bus` | Xe buýt cỡ lớn dùng chở nhiều hành khách, thường từ 17 chỗ trở lên. | Xe buýt đô thị (`img_006.jpg`, `img_037.jpg`). | Xe van nhỏ (`van`), xe tải thùng kín (`truck`). |
| `tuk_tuk` | Xe ba bánh có động cơ, kiểu auto rickshaw, dùng chở khách hoặc hàng. | Tuk-tuk Thái Lan (`img_004.jpg`) và xe chở hàng (`img_035.jpg`). | Mô-tô hai bánh (`motorcycle`), xe xích lô/xe đạp không động cơ. |
| `motorcycle` | Xe hai bánh gắn động cơ, gồm xe máy, mô-tô và xe tay ga. | Xe máy trong dòng giao thông (`img_006.jpg`, `img_010.jpg`). | Xe đạp, xích lô đạp chân, tuk-tuk ba bánh (`tuk_tuk`). |
| `person` | Người thật đang sống, gồm người đi bộ, người lái và hành khách. | Người đi bộ hoặc người ngồi trên xe (`img_001.jpg`, `img_003.jpg`). | Tượng, ma-nơ-canh, người in trên quảng cáo hoặc người chỉ xuất hiện trong phản chiếu. |
| `ignore_region` | Hình chữ nhật đánh dấu một vùng có mục tiêu giao thông quá nhỏ, dày đặc hoặc không thể phân định riêng từng vật. | Một cụm người rất nhỏ ở hậu cảnh (`img_032.jpg`) nếu không thể gán từng người theo ngưỡng bên dưới. | Vật thể rõ, tách được và cao từ 20 px trở lên; không dùng vùng này để bỏ qua đối tượng rõ. |

Vật không thuộc các lớp trên (ví dụ xe đạp, xích lô đạp, động vật, hàng hóa, tượng) thì không gán lớp mới và không ép vào lớp gần giống. Chỉ dùng `ignore_region` cho vùng có mục tiêu cần phát hiện nhưng không thể phân định riêng; không tạo vùng ignore chỉ vì gặp vật ngoài taxonomy.

### Thuộc tính

| Thuộc tính | Các mức cố định | Nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần diện tích toàn vật bị **vật khác che**: `0` = 0%; `1` = trên 0% đến 30%; `2` = trên 30% đến 70%; `3` = trên 70% nhưng phần còn thấy vẫn đủ để nhận dạng chắc chắn. Nếu không nhận dạng chắc chắn, đánh dấu `can_review` hoặc ghi vấn đề cấp ảnh thay vì đoán lớp. |
| `truncated` | `no`, `yes` | `yes` khi mép ảnh cắt mất bất kỳ phần nào của vật; các trường hợp còn lại là `no`. Cắt mép ảnh không phải là che khuất bởi vật khác. |
| `can_review` | `false`, `true` | `false` khi lớp, box và thuộc tính áp dụng được quy tắc; `true` khi còn mơ hồ về sự tồn tại, lớp, ranh giới hoặc mức che khuất. Nếu không thể vẽ box để gắn cờ, ghi ảnh, vùng và lý do vào nhật ký vấn đề cấp ảnh. |

Không gán thuộc tính cho `ignore_region`. Cờ `occluded` có sẵn trong CVAT là trường riêng; dùng `occluded_level` theo các mức trên để ghi mức che khuất.

## 3. Quy tắc vẽ box

1. **R-01 — Box ôm phần nhìn thấy:** Dùng hình chữ nhật trục ngang/dọc ôm sát bốn cực điểm của phần vật thực sự nhìn thấy, không thêm khoảng đệm và không vẽ bù phần bị khuất. Nếu vật bị mép ảnh cắt, dừng box tại mép ảnh và đặt `truncated=yes`; vẫn đặt `occluded_level` riêng nếu vật đồng thời bị vật khác che.
2. **R-02 — Người và phương tiện là hai đối tượng:** Gán một box `person` cho từng người phân biệt được và một box riêng cho phương tiện có lớp phù hợp. Box người gồm phần cơ thể nhìn thấy, kể cả mũ bảo hiểm; box xe chỉ ôm phương tiện. Hai box có thể chồng lấn ở chỗ người ngồi trên xe. Với xe đạp/xích lô đạp, chỉ gán người nếu đủ điều kiện; không gán phương tiện không động cơ thành `motorcycle`.
3. **R-03 — Phân lớp xe và hàng chở:** Phân biệt `motorcycle` hai bánh, `tuk_tuk` ba bánh có động cơ, xe khách `bus`, xe hàng `truck`, xe thân kín nhỏ/vừa `van` và ô tô con `car` theo cấu tạo nhìn thấy. Box xe ôm phần cấu tạo của xe, không nới ra để bao hàng hóa hoặc vật chở rời. Người đi cùng được gán box riêng. Mô-tô có sidecar gắn liền dùng một box cho cụm xe; không nhầm auto rickshaw ba bánh thành mô-tô.
4. **R-04 — Vật thật và hình ảnh:** Gán phương tiện thật dù đang đỗ, nằm sau hàng rào hoặc bị che một phần. Không gán tượng, ma-nơ-canh, mô hình đồ chơi, ảnh trên biển quảng cáo hay hình phản chiếu trong gương/kính. Hàng hóa và vật/động vật không thuộc taxonomy không nhận lớp phương tiện.
5. **R-05 — Vật nhỏ, đám đông và xe chủ:** Không vẽ box riêng nếu chiều cao phần nhìn thấy dưới `20 px`. Trong đám đông, gán từng người nhận dạng được và đạt ngưỡng; không vẽ một box `person` bao cả nhóm. Chỉ khoanh `ignore_region` quanh phần dày đặc/không thể tách; không để vùng đó che các box rõ bên cạnh. Không gán các bộ phận của chính xe mang camera (như nắp capo hoặc gương ở mép ảnh) thành xe khác. Nếu vật mờ nhưng vẫn đủ nhận dạng thì gán theo R-01; nếu chưa đủ căn cứ thì dùng `can_review` hoặc nhật ký vấn đề cấp ảnh.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_001.jpg` | Người đứng cạnh/thấy một phần sau xe bán tải; xe tải bị cắt ở mép phải. | R-01, R-02 | Box riêng cho từng người và `truck`; box xe dừng tại mép ảnh, `truncated=yes`. Mức che người tính theo phần bị xe che, không theo phần nằm ngoài khung. | Người và xe là hai đối tượng; cắt khung và che khuất là hai thuộc tính khác nhau. |
| `img_003.jpg` | Người bán hàng và người đi đường đứng cạnh xe đạp/xích lô. | R-02 | Gán người đủ rõ; không gán xe đạp hay xích lô đạp vào `motorcycle`. | Chỉ phương tiện có động cơ thuộc các lớp xe hai/ba bánh trong taxonomy. |
| `img_004.jpg` | Hai xe ba bánh có động cơ kiểu auto rickshaw. | R-03 | Mỗi tuk-tuk có một box `tuk_tuk`; người lái chỉ có box `person` nếu nhìn thấy và phân biệt được. | Số bánh và cấu tạo xe phân biệt tuk-tuk với mô-tô hai bánh. |
| `img_015.jpg` | Ô tô trong mưa, có xe chuyển động mờ và xe bị cắt ở hai mép ảnh. | R-01, R-05 | Gán xe còn nhận dạng được theo phần nhìn thấy; xe bị khung cắt đặt `truncated=yes`. Chỉ đặt `can_review=true` nếu nhòe khiến lớp hoặc ranh giới không thể quyết định. | Nhòe chuyển động không tự động làm vật thành ignore; cắt mép không đồng nghĩa che khuất. |
| `img_018.jpg` | Có hình người trên biển quảng cáo và ô tô thật trong bãi phía sau hàng rào. | R-04 | Không gán `person` cho người in trên biển; gán từng `car` thật nếu nhận dạng được. | Hình in không phải người thật; hàng rào chỉ che một phần xe thật. |
| `img_032.jpg` | Cụm người đông ở trạm chờ và một phần ô tô ở mép trái. | R-01, R-05 | Gán từng người phân biệt được có chiều cao box từ 20 px; không gộp thành một box người. Dùng `ignore_region` chỉ cho cụm quá nhỏ/không thể tách; box xe bị ảnh cắt đặt `truncated=yes`. | Ngưỡng kích thước áp dụng cho từng vật; vùng ignore không thay thế nhãn cho người rõ. |
| `img_035.jpg` | Tuk-tuk chất bao hàng cao, người lái và xe máy ở gần. | R-02, R-03 | Box `tuk_tuk` ôm phần xe, không ôm bao hàng; người và mô-tô thật được gán riêng nếu đủ rõ. | Hàng chở không làm thay đổi hình học box xe; từng người/phương tiện giữ nhãn riêng. |
| `img_011.jpg` | Tác phẩm điêu khắc lớn ở tiền cảnh, người thật ở hậu cảnh. | R-04 | Không gán tượng thành `person` hay `motorcycle`; gán người thật ở hậu cảnh nếu đạt ngưỡng 20 px. | Hình dáng giống người/xe không đủ để thành đối tượng thuộc lớp; cần là người hoặc phương tiện thật. |

Nếu gặp ca chưa chốt, đặt `can_review=true` trên box liên quan hoặc ghi vấn đề cấp ảnh nếu chưa thể vẽ box. Dùng các mã `R-01`–`R-05` trong nhật ký QC.

## 5. QC và phiên bản

Áp dụng nguyên ngưỡng trong `submission/taxonomy.json` phiên bản `v1.0`:

- Chiều cao box tối thiểu: `20 px`.
- Ghép cặp box để tính IAA khi IoU `>= 0.50`; ngưỡng đồng thuận IoU theo chiều cao trung bình của cặp box là `>= 0.85` nếu từ `50 px`, và `>= 0.70` nếu dưới `50 px`.
- IAA toàn lô: IoU trung bình `>= 0.85`; đồng thuận lớp `>= 95%`.
- Tỉ lệ ảnh có lỗi khi nghiệm thu: tối đa `2%`.

Box còn mơ hồ được thành viên 038 đánh dấu để reviewer xem. Theo `TEAM.md`, thành viên 029 điều phối chốt ca của nhóm và thành viên 059 review bản cuối. Gỡ cờ `can_review` sau khi đã có quyết định cuối; không để cờ này thay cho quyết định gán nhãn.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Khung guideline ban đầu. | Cần làm rõ lớp, ngưỡng thuộc tính, phần box bị che/cắt và vật ngoài taxonomy. |
| `v0.1-draft` | Viết rõ định nghĩa lớp theo taxonomy `v1.0`, ngưỡng thuộc tính, quy tắc box R-01–R-05 và các ca khó trong ảnh được giao. | `img_001`, `img_003`, `img_004`, `img_015`, `img_018`, `img_032`, `img_035`, `img_011`. |
