# Báo cáo QC — ATeam

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC nhãn 2D trong Day 07; tập ảnh tuyển chọn để luyện ca biên giao thông đô thị; độ đại diện cho dữ liệu vận hành thực tế cần được đánh giá thêm khi mở rộng quy mô.
- Nguồn và phép sử dụng dữ liệu: Nguồn ảnh cộng đồng Wikimedia Commons theo giấy phép CC BY-SA / CC BY / CC0, danh mục quyền tác giả đầy đủ tại data/ATTRIBUTION.csv.
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256 nhãn: aa48bf807a2347806884101ae35214b2ae900114b6ca83ba96be8c379b447d92).
- Guideline và taxonomy khi gán / khi bàn giao: Taxonomy v1.0 (8 lớp đối tượng); Guideline v1.0 khi gán độc lập và Guideline v1.1 (Calibrated) sau khi hiệu chỉnh Pha 5.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 lượt hoàn thành trên 40 ảnh (5 thành viên, K=3 lượt gán độc lập cho mỗi ảnh).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 881 cặp box ghép, IoU trung bình 0.868 (vượt ngưỡng yêu cầu 0.85), đồng thuận lớp đạt 98.5% (vượt ngưỡng yêu cầu 95.0%) trên tổng số 881 cặp box ghép.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Box chưa ghép 279/402, loại bỏ do ignore_region 23 ca, bất đồng vùng ignore 269 ca.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_002.jpg (xe tải chở xe con 2 sàn, hai người có thể cùng bỏ sót việc vẽ từng box car cho các xe con cõng trên sàn mà chỉ vẽ 1 box truck lớn; đã đối chiếu và chốt vẽ đủ cả truck lẫn từng car).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý toàn bộ 1560 ca bất thường trong submission/qc/calibration_decisions.csv, áp dụng các quy tắc R-01 đến R-09, người chốt 101, kiểm lại đạt 100%.
- Tác động v1.1 lên nhãn và ảnh: Bổ sung làm rõ quy chuẩn ôm sát biên ngoài cùng cho bounding box (R-09), chuẩn hóa mức occluded_level cho xe con tầng trên (mức 1) và tầng dưới (mức 2) của xe chuyên chở xe (R-02), và quy định vẽ 1 box duy nhất khi nhìn vật xuyên qua kính xe khác (R-03).
- Tham chiếu bằng chứng khi Rework: Đạt chuẩn chất lượng ngay tại Vòng 1, không cần chuyển sang Rework.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 102, 2026-11-17
- Đã duyệt / cả lô: 40/40 ảnh đã duyệt (đạt 100% toàn lô).
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0% (thấp hơn ngưỡng tối đa cho phép 5.0%).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/581 = 0.0%.
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Đã duyệt 38/38 ca khó (0 lỗi), 2/2 ảnh còn lại (0 lỗi); không có hiện tượng lỗi cùng loại lặp lại.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy tự động chặn các box có tọa độ âm hoặc ngoài khung ảnh và thiếu thuộc tính bắt buộc; mắt người cần trực tiếp đối chiếu ảnh để phân biệt người thật đang lái xe với tranh ảnh quảng cáo/tượng đài và xác định ranh giới hàng hóa cồng kềnh trên tuk-tuk.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Lô nhãn đạt chất lượng cao với tỷ lệ lỗi ảnh 0.0%, IoU trung bình 0.868, dữ liệu sạch và nhất quán; sẵn sàng bàn giao cho đội ngũ kỹ sư thị giác máy tính huấn luyện mô hình perception xe tự hành.
- Nếu còn việc: Không còn tồn đọng; toàn bộ các bằng chứng, file nhãn và biên bản kiểm tra đã được đồng bộ hoàn tất.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C001, ảnh img_001.jpg, áp dụng quy tắc R-09, nhãn trong labels_calibrated.json khớp chính xác với ảnh overlay kiểm chứng work/review/round-01/img_001.png.
