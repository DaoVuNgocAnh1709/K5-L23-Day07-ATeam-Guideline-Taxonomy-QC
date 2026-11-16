# Guideline gán nhãn — v1.0 — Mã 103

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Người viết / duyệt: Mã 103

## 1. Mục tiêu
- **Mô hình sử dụng**: Mô hình phát hiện vật thể 2D (2D Object Detection) phục vụ hệ thống nhận thức môi trường cho xe tự hành (Autonomous Driving) và hỗ trợ lái nâng cao (ADAS) trong giao thông đô thị.
- **Mục đích**: Nhận diện chính xác tọa độ 2D và phân loại các chủ thể tham gia giao thông (người và phương tiện cơ giới) để tính toán khoảng cách, dự đoán quỹ đạo di chuyển và tránh va chạm.
- **Rủi ro khi sai sót**:
  - *Bỏ sót nhãn (False Negative)* với người đi bộ, người đi xe máy hoặc phương tiện bị che khuất một phần có thể dẫn đến va chạm trực tiếp.
  - *Gán nhầm vật thể giả (False Positive)* như tượng, ma-nơ-canh, ảnh in trên biển quảng cáo hoặc hình phản chiếu qua gương thành vật thể thật khiến xe phanh gấp đột ngột (phantom braking), gây nguy hiểm cho phương tiện phía sau.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật đang tham gia giao thông hoặc xuất hiện trong không gian đường phố ở mọi tư thế (đi bộ, đứng, ngồi, điều khiển phương tiện). | Người đi bộ trên vỉa hè, người lái xe máy, người ngồi sau xe máy, người đạp xe đạp (`img_006`). | Tượng người bằng đồng/đá (`img_011`, `img_017`), ma-nơ-canh trong tủ kính (`img_033`), chân dung trên biển quảng cáo (`img_018`), hình người phản chiếu qua gương chiếu hậu (`img_020`). |
| `car` | Ô tô con chở khách dưới 9 chỗ có thiết kế thân liền khối dùng cho mục đích cá nhân hoặc dịch vụ. | Xe sedan, SUV, hatchback, taxi 4–7 chỗ đang lưu thông hoặc đỗ bên đường. | Xe bán tải có thùng hàng (`truck`), xe van thân hộp chở hàng/khách (`van`), xe mô hình đồ chơi (`img_038`), ô tô con đang được chở trên xe tải chuyên dụng (`img_002`). |
| `truck` | Phương tiện cơ giới đường bộ thiết kế chuyên biệt để chở hàng hóa hoặc kéo rơ-moóc, có khoang chở hàng tách biệt hoặc thùng hở. | Xe tải thùng, xe ben, xe đầu kéo container, xe bán tải (pickup) nguyên bản, xe tải lồng chở ô tô (`img_002`). | Xe bán tải lắp nắp thùng cao ngang nóc cabin (`img_037` — cần reviewer chốt ranh giới với `van`), tượng xe bán tải trưng bày (`img_027`). |
| `van` | Xe ô tô thân hộp liền khối (từ cabin ra khoang sau) dùng để chở hàng nhẹ hoặc chở khách cỡ nhỏ/trung (9–16 chỗ). | Xe tải van chở hàng, xe chở khách 16 chỗ, xe minivan cỡ lớn. | Xe SUV 7 chỗ (`car`), xe buýt cỡ lớn trên 24 chỗ (`img_009`, `img_037`), xe tuk-tuk 3 bánh (`img_004`, `img_035`). |
| `motorcycle` | Phương tiện cơ giới hai hoặc ba bánh có động cơ, điều khiển bằng ghi-đông. | Xe máy số, xe tay ga, xe mô tô phân khối lớn, xe máy điện có bàn để chân, xe ba bánh tuk-tuk (`img_004`, `img_035`). | Xe đạp thường không động cơ (`img_006`, `img_039`), xe xích lô đạp (`img_029`). |

**Xử lý vật không thuộc lớp nào**:
- **Không gán nhãn (Bỏ qua)**: Các đối tượng phi thực tế (tượng, ma-nơ-canh, biển quảng cáo, đồ chơi, hình phản chiếu), phương tiện thô sơ không động cơ (xe đạp, xích lô) và vật thể quá nhỏ (`< 20 px` ở cả hai chiều).
- **Vùng `ignore` & Lớp mới**: Nếu xuất hiện đám đông dày đặc hoặc dãy xe ở quá xa chồng lấn không thể tách rời từng cá thể, hoặc phương tiện chưa có trong danh mục gốc như xe buýt (`img_009`, `img_037`), đánh dấu `can_review=true` để thống nhất sử dụng vùng `ignore` hoặc bổ sung lớp `bus` trong buổi họp calibration.

### Thuộc tính
1. **`occluded_level`** (Mức độ che khuất — các mức cố định: `0`, `1`, `2`, `3`):
   - `0`: **Không bị che khuất** (`0%` diện tích vật thể bị che).
   - `1`: **Che khuất nhẹ** (`1% – 30%` diện tích ước tính bị vật khác che phủ; vẫn thấy rõ hình dáng chính).
   - `2`: **Che khuất trung bình** (`31% – 70%` diện tích ước tính bị che phủ; vẫn nhận diện chắc chắn loại vật thể).
   - `3`: **Che khuất nặng** (`> 70%` diện tích ước tính bị che phủ nhưng vẫn còn đặc điểm nhận dạng rõ ràng như đầu/vai người hoặc đầu/đuôi xe; nếu bị che `> 90%` và không nhận diện chắc chắn thì bỏ qua).
2. **`truncated`** (Bị cắt bởi mép khung hình — các mức cố định: `true`, `false`):
   - `false`: Toàn bộ vật thể nằm trọn bên trong biên ảnh (cách mép ảnh `>= 2 px`).
   - `true`: Bất kỳ phần nào của vật thể chạm hoặc bị cắt ngang bởi 1 trong 4 cạnh khung ảnh (`< 2 px` tới biên ảnh).
3. **`can_review`** (Cần reviewer kiểm tra lại — các mức cố định: `true`, `false`):
   - `false`: Ca rõ ràng, áp dụng dứt điểm quy tắc trong guideline.
   - `true`: Ca biên gây phân vân giữa hai lớp hoặc cần chốt thêm ở cấp độ nhóm.

## 3. Quy tắc vẽ box
- **RULE-01 (Box ôm sát phần nhìn thấy - Visible Box)**: Vẽ bounding box ôm khít nhất có thể vào đường bao các pixel **nhìn thấy được** của vật thể (sai số không quá `2–3 px`), tuyệt đối không ước lượng phóng rộng box ra phần bị che khuất (không vẽ amodal box).
- **RULE-02 (Một vật có nhiều phần — Người và xe hai bánh)**: Tách thành **2 box độc lập**:
  - 01 box lớp `person` bao trọn toàn bộ cơ thể người lái (từ đỉnh đầu/mũ bảo hiểm xuống hết bàn chân).
  - 01 box lớp `motorcycle` bao trọn chiếc xe (từ bánh trước đến bánh sau, gồm cả phần thân xe nằm sau chân người lái). Nếu có người ngồi sau, vẽ thêm 01 box `person` riêng cho từng người.
- **RULE-03 (Vật quá nhỏ)**: Nếu cả chiều rộng (`width`) và chiều cao (`height`) của phần nhìn thấy đều **nhỏ hơn 20 px**, **không gán nhãn** vì thiếu đặc trưng hình ảnh cho mô hình học.
- **RULE-04 (Tượng, ma-nơ-canh, đồ chơi, hình in)**: **Không gán nhãn** cho tượng người/xe, ma-nơ-canh trong tủ kính, xe mô hình đồ chơi, hoặc hình người/xe in trên biển quảng cáo, thân xe buýt.
- **RULE-05 (Hình phản chiếu)**: **Không gán nhãn** cho người hoặc phương tiện chỉ xuất hiện qua gương chiếu hậu, kính cửa hàng hoặc vũng nước.
- **RULE-06 (Xe chủ - Ego vehicle)**: Nếu một phần nắp ca-pô hoặc gương của chính xe gắn camera lọt vào mép dưới khung hình, **không gán nhãn** cho xe chủ.
- **RULE-07 (Xe chở xe - Carrier truck)**: Với xe tải chuyên dụng đang chở nhiều ô tô con trên thùng (`img_002`), chỉ gán **01 box `truck`** bao trọn cả xe tải đầu kéo và dàn xe đang được chở trên sàn (coi như hàng hóa cố định trên xe để tránh mô hình nhầm lẫn các xe trên thùng là xe đang lưu thông độc lập).
- **RULE-08 (Đám đông / Hàng xe dày đặc)**: Các cá thể tách biệt rõ ràng vẫn gán từng box riêng; nếu cụm người/xe ở quá xa chồng lấn không thể phân tách ranh giới từng đối tượng, đánh dấu `can_review=true` để áp dụng vùng `ignore`.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải lồng 2 tầng chở 8 ô tô con phía sau | `RULE-07` | **Loại tách lẻ theo quy tắc**: Gán 01 box `truck` trùm cả xe chở và dàn xe trên thùng. | Các ô tô trên sàn là hàng hóa cố định, không tham gia giao thông độc lập; gán lẻ làm sai lệch bài toán dự đoán quỹ đạo. |
| `img_004` | Xe ba bánh tuk-tuk lưu thông trên phố | `RULE-02` | **Cần hỏi reviewer (`can_review=true`)**: Tạm gán `motorcycle` và đánh dấu chờ họp nhóm. | v0 chưa định nghĩa xe 3 bánh có khoang hành khách thuộc `motorcycle`, `car` hay cần lớp riêng. |
| `img_006` | Người đạp xe đạp trên đường | `RULE-02` | **Loại theo quy tắc**: Chỉ gán `person` cho người lái, không gán box cho xe đạp. | Taxonomy gốc chỉ gồm 5 lớp cơ giới/người (`car, truck, van, motorcycle, person`), không có lớp `bicycle`. |
| `img_009` | Xe buýt cỡ lớn lưu thông | `RULE-01` | **Cần hỏi reviewer (`can_review=true`)**: Tạm gán `van` và tick `can_review`. | v0 không có lớp `bus`; cần thống nhất nhóm giữa việc gộp vào `van`/`truck` hay thêm lớp `bus`. |
| `img_011` | Tượng đồng người đứng ngoài trời | `RULE-04` | **Loại theo quy tắc**: Không gán nhãn `person`. | Tượng tĩnh không có khả năng di chuyển; gán `person` gây cảnh báo giả (False Positive) cho xe tự hành. |
| `img_017` | Tượng đá/điêu khắc hình người bên đường | `RULE-04` | **Loại theo quy tắc**: Không gán nhãn `person`. | Tương tự `img_011`, vật thể vô tri không phải chủ thể tham gia giao thông. |
| `img_018` | Chân dung người kích thước lớn trên biển quảng cáo | `RULE-04` | **Loại theo quy tắc**: Không gán nhãn `person`. | Hình ảnh 2D phẳng trên biển hiệu, không phải người thật trong không gian 3D. |
| `img_020` | Hình người/xe phản chiếu trong gương chiếu hậu | `RULE-05` | **Loại theo quy tắc**: Không gán nhãn đối tượng trong gương. | Tọa độ 2D trong gương không phản ánh đúng vị trí thực tế của vật thể trên đường. |
| `img_027` | Tượng xe bán tải trưng bày | `RULE-04` | **Loại theo quy tắc**: Không gán nhãn `truck`/`car`. | Vật trang trí cố định, không phải phương tiện cơ giới hoạt động. |
| `img_029` | Xe xích lô đạp lưu thông trên đường | `RULE-02` | **Loại theo quy tắc**: Chỉ gán `person` cho người đạp và hành khách, bỏ qua khung xe xích lô. | Phương tiện thô sơ không động cơ, nằm ngoài phạm vi 5 lớp cơ giới của guideline. |
| `img_033` | Ma-nơ-canh mặc quần áo đứng sau tủ kính cửa hàng | `RULE-04` | **Loại theo quy tắc**: Không gán nhãn `person`. | Ma-nơ-canh có hình dáng giống người nhưng là vật tĩnh trong cửa hàng. |
| `img_035` | Xe tuk-tuk chở khách góc nhìn khác | `RULE-02` | **Cần hỏi reviewer (`can_review=true`)**: Tạm gán `motorcycle` và tick `can_review`. | Cần đồng nhất cách xử lý với `img_004` trong buổi họp thống nhất nhóm. |
| `img_037` | Xe buýt và xe bán tải lắp nắp thùng cao (Master 75) | `RULE-01` | **Cần hỏi reviewer (`can_review=true`)**: Tạm gán `truck` cho bán tải nắp thùng cao, `van` cho xe buýt và tick `can_review`. | Bán tải gắn nắp thùng cao dễ nhầm với `van`; xe buýt chưa có lớp riêng trong danh mục gốc. |
| `img_038` | Xe ô tô mô hình đồ chơi bày trong tủ kính | `RULE-04` | **Loại theo quy tắc**: Không gán nhãn `car`. | Đồ chơi thu nhỏ, không phải phương tiện giao thông thật. |

## 5. QC và phiên bản
