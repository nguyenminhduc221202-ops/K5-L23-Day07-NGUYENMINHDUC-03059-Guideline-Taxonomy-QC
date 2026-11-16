# Báo cáo QC — K5-L23-Nhom3-GuidelineQC

Báo cáo nghiệm thu chất lượng dữ liệu gán nhãn 2D bounding box (Pha 6 — Checkpoint CP6).

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và kiểm soát chất lượng nhãn bounding box 2D cho bài toán phát hiện người và phương tiện phục vụ nghiên cứu, huấn luyện và đánh giá mô hình ADAS/xe tự hành. Giới hạn độ đại diện: tập 40 ảnh tuyển chọn các ca biên phức tạp, chưa đánh giá phân phối vận hành thực tế.
- Nguồn và phép sử dụng dữ liệu: Dữ liệu ảnh nguồn có bản quyền mở Creative Commons (CC BY, CC BY-SA, CC0) từ Wikimedia Commons và các tác giả ghi công đầy đủ tại `data/ATTRIBUTION.csv`.
- Manifest và checksum nhãn cuối: Quản lý tại `submission/qc/batch_manifest.json` với SHA-256 nhãn cuối: `be39bb76a83a0bcd0faa21c771aea8ed066b0fc2a6ba8ccb9d9b31914bdc26ef`.
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập dùng guideline v1.0 và taxonomy v1.0; khi bàn giao dùng guideline v1.1-calibrated và taxonomy v1.0 (giữ nguyên taxonomy và rules, hiệu chuẩn quy tắc và ngưỡng).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 lượt (đủ 100%, 5 thành viên gán chéo, mỗi ảnh đúng 3 người gán).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 484 cặp box ghép được (mẫu số 484 cặp), IoU trung bình ban đầu đạt 0.841, tỷ lệ đồng thuận lớp đạt 97.7% (vượt ngưỡng >= 95%).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 217 box bên A chưa ghép, 197 box bên B chưa ghép, 0 box loại do ignore, không có bất đồng vùng ignore_region.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Ảnh img_002.jpg (xe tải chở 8 ô tô con hai tầng sàn), cả hai người có thể cùng gán 1 box truck to trùm toàn bộ hoặc cùng bỏ sót ô tô con; đối chiếu quy tắc R-05 để thống nhất nhận diện một khối vận tải siêu trường.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý 810 ca trong calibration_candidates.csv, dẫn chiếu tại submission/qc/calibration_decisions.csv với các mã quy tắc R-01 đến R-04, người chốt 029 và 059, toàn bộ kiểm lại đạt "đã xác nhận".
- Tác động v1.1 lên nhãn và ảnh: Phạm vi hiệu chuẩn 36 ảnh ca khó, loại bỏ toàn bộ cờ can_review về false, chuẩn hóa ranh giới box ôm sát vật thể và đồng thuận mức occluded_level; giữ nguyên taxonomy v1.0 nên không cần đổi mapping lớp.
- Tham chiếu bằng chứng khi Rework: Vòng 1 đạt nghiệm thu toàn bộ lô, không phát sinh lỗi vượt ngưỡng nên không cần chuyển vòng Rework sang vòng 2.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: Thành viên 029 (Phạm Lam Anh), ngày 2026-11-16
- Đã duyệt / cả lô: 40/40 ảnh (duyệt toàn bộ 100% lô ảnh)
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0% (đạt ngưỡng cho phép <= 2%)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/338 = 0.0% (338 đối tượng đều hợp lệ)
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó (36 ảnh): 0/36 lỗi (0.0%); Phần còn lại (4 ảnh): 0/4 lỗi (0.0%). Không phát sinh lỗi lặp; dẫn phiếu chi tiết tại submission/qc/reviews/round-01/qc_review.csv.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn tự động lỗi sai format, thiếu ảnh, sai tên lớp, box âm hoặc rỗng (validate_labels.py); người đọc ảnh phát hiện các ca ngữ nghĩa phức tạp như ma-nơ-canh sau kính (img_033), tượng đài đá (img_017, img_027) hay xe đồ chơi trong tủ (img_038).

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Dữ liệu đạt 100% chỉ số nghiệm thu (tỷ lệ lỗi 0.0% <= 2%, đồng thuận lớp 97.7% >= 95%), bàn giao cho Giảng viên / Coach lớp K5-L23 để phục vụ nghiên cứu và thực hành bài toán phát hiện vật thể giao thông 2D.
- Nếu còn việc: Không còn ca tồn đọng (0 guideline-gap, 0 data-problem).
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C026 ảnh img_002.jpg áp dụng rule R-05 tại labels_calibrated.json annotation #2 khớp hoàn toàn với quy tắc trong guideline_v1.1_calibrated.md.
