# Guideline gán nhãn — v0.1 — 048

- Phiên bản: v0.1-048
- Ngày: 2026-11-16
- Người viết: thành viên 048
- Ghi chú: bản này viết **sau** khi gán 24 ảnh được giao (`submission/qc/assignments/048.txt`), tổng hợp lại các quyết định đã áp dụng trong `submission/labels/048.json`. Bản nháp có AI hỗ trợ trình bày; quyết định và ví dụ đối chiếu với nhãn thật của 048 (xem `individual_notes/048.md`).

## 1. Mục tiêu

Nhãn dùng huấn luyện mô hình phát hiện **người và phương tiện thật đang tham gia giao thông** trên ảnh đường phố 2D (camera giao thông, hỗ trợ lái).

Sai nguy hiểm nhất:
- **Bỏ sót người thật** (đặc biệt người đi xe máy, người đứng sát mép ảnh) → mô hình không phanh/không cảnh báo.
- **Gán nhầm vật không thật** (ma-nơ-canh, tượng, ảnh in, phản chiếu) → mô hình báo động giả, học sai đặc trưng.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người ≤ 9 chỗ: sedan, hatchback, SUV, crossover. | img_018, img_024 (dòng xe con đỗ) | Pickup có thùng hở → `truck` (R-04); xe 7 chỗ dạng hộp có cửa trượt → `van` |
| `truck` | Xe chở hàng có **thùng hàng tách khỏi cabin** (tải, pickup, xe ben, xe chở xe). | img_012 #43 (pickup chở hàng, thùng hở), img_007, img_036 | Xe thùng kín liền khối với cabin → `van` |
| `van` | Xe thân hộp liền khối, cabin và khoang sau chung một vỏ (minivan, xe 12–16 chỗ, xe giao hàng thùng kín cỡ nhỏ). | img_016 #53 (van trắng), img_005 #15 | Pickup → `truck`; xe khách lớn → `bus` |
| `bus` | Xe chở khách cỡ lớn, dài, nhiều hàng ghế, thường có nhiều cửa sổ dọc thân. | img_009, img_037 | Van 16 chỗ → `van` |
| `tuk_tuk` | Xe 3 bánh có mái che chở khách/hàng (xe lam, tuk-tuk). | img_016 #51, #52; img_040 #135 | Xe máy 2 bánh → `motorcycle`; không gộp vào `car` |
| `motorcycle` | Xe 2 bánh có động cơ (xe máy, xe tay ga, mô tô), **chỉ phần xe**. | img_016 #50 (Vespa), img_034 | Xe đạp → không thuộc lớp nào (R-08) |
| `person` | Người thật, đi bộ hoặc đang ngồi/lái trên phương tiện. | img_011, img_021, img_032 (đám đông đi bộ) | Ma-nơ-canh, tượng, người trong ảnh quảng cáo, phản chiếu → **không gán** (R-06) |
| `ignore_region` | Vùng nhiều vật nhỏ/dày đặc không thể vẽ từng box; bỏ sót trong vùng này không tính lỗi. | (048 chưa dùng trong 24 ảnh) | Không dùng để "né" vật khó nhưng vẫn vẽ được |

**Vật không thuộc lớp nào** (xe đạp, xe đẩy, xe kéo tay, động vật): **không gán**. Nếu nghi ngờ nên có lớp mới → ghi vào `image_issues.csv` để nhóm quyết, không tự thêm lớp.

### Thuộc tính

| Thuộc tính | Mức | Ngưỡng (theo phần diện tích vật bị che, ước lượng bằng mắt) |
|---|---|---|
| `occluded_level` | `0` | Thấy rõ, bị che < 10% |
| | `1` | Bị che 10% – < 40% |
| | `2` | Bị che 40% – < 70% |
| | `3` | Bị che ≥ 70% **hoặc** không ước lượng được (chỉ thấy đầu, một phần thân) |
| `truncated` | `yes` / `no` | `yes` khi box chạm mép ảnh (≤ 2 px) **và** vật thật còn tiếp ra ngoài khung. Không tính phần bị che bởi vật khác. |
| `can_review` | tick / không | Tick khi không chắc lớp hoặc không chắc vật có phải là thật. |

Che khuất và cắt mép là **hai thuộc tính độc lập**: người bị cắt mép nhưng không bị vật nào che → `occluded_level=0`, `truncated=yes` (ví dụ img_005 #10, #11).

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | Box ôm sát **phần nhìn thấy** của vật, không đoán phần bị che. |
| **R-02** | **Người đi xe máy: 2 box riêng.** Một box `person` cho người lái/người ngồi sau (từ đầu tới chân), một box `motorcycle` cho xe (bánh trước tới bánh sau, gồm cả hàng chở trên xe). Hai box được phép chồng nhau. Mỗi người ngồi trên xe là một box `person`. Ví dụ: img_016 #49 + #50, img_029, img_034. |
| **R-03** | Người ngồi trong ô tô/tuk-tuk: chỉ gán `person` nếu thấy rõ đầu + thân qua khoang hở (tuk-tuk, xe máy); người sau kính ô tô kín → không gán. |
| **R-04** | Pickup có thùng hàng hở phía sau → `truck` (img_012 #43). |
| **R-05** | Xe chở xe / xe chở hàng: gán **xe đang chạy** theo lớp của nó; xe được chở trên thùng **không gán** (không tham gia giao thông). |
| **R-06** | Vật **không thật**: ma-nơ-canh, tượng, ảnh in trên biển quảng cáo, phản chiếu trên kính → **không gán**. Ví dụ img_033 (ma-nơ-canh + gương xe phản chiếu): ảnh không có box nào. |
| **R-07** | Vật cao < 20 px: không vẽ box riêng; nếu là cụm nhiều vật nhỏ thì vẽ một `ignore_region` phủ cụm. |
| **R-08** | Vật ngoài taxonomy (xe đạp, xe đẩy, động vật) → không gán. |
| **R-09** | Đám đông: vẽ box từng người khi mỗi người cao ≥ 20 px và tách được ranh giới; phần quá dày không tách được → `ignore_region`. Ví dụ img_011, img_021, img_032 (vẽ từng người). |
| **R-10** | Xe của người chụp (ego, phần đầu xe/nắp capo ở mép dưới ảnh) → không gán. |

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_033 | Ma-nơ-canh sau kính cửa hàng, phản chiếu gương chiếu hậu của một xe | R-06 | Không gán box nào; ảnh vẫn có trong COCO | Không có người/xe thật trong khung; gán sẽ dạy mô hình nhận ma-nơ-canh là người |
| img_016 | Người đi Vespa chở hàng | R-02 | #49 `person` + #50 `motorcycle` | Tách người và xe để mô hình đếm đúng người tham gia giao thông |
| img_016 | Hai tuk-tuk (trái và phải), đều bị cắt mép | R-04, `truncated` | `tuk_tuk`, `truncated=yes` | Xe 3 bánh có mái, không phải ô tô |
| img_016 | Người mặc áo đỏ đứng sát mép phải, bị cắt | `truncated` | #48 `person`, occ 0, trunc yes | Người thật, chỉ bị cắt mép, không bị che |
| img_012 | Pickup chở hàng ở tiền cảnh, thùng hở | R-04 | #43 `truck`, trunc yes | Có thùng hàng tách cabin |
| img_012 | Xe ở xa phía cuối đường, rất nhỏ (~35×29 px) | R-07 | #45 `van`, occ 2 | Cao ≥ 20 px nên vẫn vẽ box |
| img_005 | Nhiều người đi xe máy chen nhau dừng đèn, người sau chỉ thấy mũ bảo hiểm/mặt | R-01, `occluded_level` | #12, #14 occ 3; #13 occ 2; #10, #11 occ 0 trunc yes | Che ≥ 70% → mức 3; người tiền cảnh chỉ bị cắt mép |
| img_040 | Người đội mũ bảo hiểm ngồi trên xe máy sát camera, phía sau taxi, SUV, tuk-tuk | R-02, `truncated` | #133 `person` trunc yes; #136 `car` (taxi) trunc yes; #135 `tuk_tuk` | Vật lớn bị cắt mép vẫn phải gán |
| img_009, img_037 | Xe buýt | lớp `bus` | `bus` | Dài, nhiều cửa sổ, chở khách |

**Câu hỏi để reviewer / nhóm chốt:**
1. Pickup có nắp thùng kín (canopy) là `truck` hay `van`? (Gặp ở img_016 bên trái — 048 đã gán #54 `van`, **không nhất quán** với R-04 / img_012 #43.)
2. Người lái tuk-tuk / khách ngồi trong tuk-tuk có gán `person` không (img_016)?
3. Có tách lớp riêng cho xe đạp không, hay luôn bỏ qua?

## 5. QC và phiên bản

- Ngưỡng theo `taxonomy.json › rules` (không đổi): IoU ≥ 0.85 (box cao ≥ 50 px) / 0.7 (nhỏ hơn); IAA cùng lớp ≥ 95%; lỗi theo ảnh ≤ 2% mới Release; vật < 20 px theo R-07.
- Ca mơ hồ: tick `can_review` trên box, hoặc ghi `image_issues.csv`; nhóm quyết ở buổi calibration, người chốt ghi vào `calibration_decisions.csv`.
- Mọi thay đổi làm đổi nhãn → tăng phiên bản.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên | — |
| v0.1-048 | Thêm định nghĩa 8 lớp, ngưỡng % cho `occluded_level`, R-01…R-10, bảng ca khó | Các ca gặp khi gán 24 ảnh: img_005, img_012, img_016, img_033, img_040 |
