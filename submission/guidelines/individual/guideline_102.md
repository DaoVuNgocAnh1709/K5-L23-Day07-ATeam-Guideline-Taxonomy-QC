# Guideline gán nhãn — v1.0 — Mã 102

- Phiên bản: v1.0-ind-102
- Ngày: 2026-11-16
- Người viết: 102

---

## 1. Mục tiêu

Nhãn phục vụ huấn luyện mô hình thị giác máy tính phát hiện đối tượng (**Object Detection**) phục vụ hệ thống lái xe tự động / hỗ trợ lái (ADAS) và camera giám sát giao thông đô thị.
- **Mục đích**: Nhận diện chính xác vị trí không gian của các tác nhân tham gia giao thông (người, xe con, xe tải, xe van, mô tô/xe máy) trên đường để tính khoảng cách an toàn, tránh va chạm.
- **Rủi ro nguy hiểm**:
  - Bỏ sót (False Negative) người đi bộ hoặc người đi xe hai bánh dẫn tới nguy cơ va quẹt, tai nạn chết người.
  - Nhầm lẫn kích thước xe (vẽ box thiếu phần nhô ra, thùng hàng chở cồng kềnh) gây va quẹt khi chuyển làn hoặc ước lượng khoảng cách hẹp.
  - Báo động giả (False Positive) từ ma-nơ-canh, tượng nghệ thuật, đồ chơi hoặc hình phản chiếu qua gương/cửa kính có thể khiến hệ thống phanh gấp nguy hiểm không cần thiết.

---

## 2. Lớp và thuộc tính

### Lớp

Hệ thống kế thừa taxonomy chuẩn (`car`, `truck`, `van`, `motorcycle`, `person`, `ignore_region`):

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Con người còn sống thực tế ở mọi tư thế (đứng, đi bộ, ngồi, điều khiển xe). | Người đi bộ, người đang cưỡi xe máy/xe đạp, hành khách ngồi trên xe hở. | Tượng (img_017), ma-nơ-canh (img_033), hình ảnh in trên pano/biển quảng cáo (img_018, img_030). |
| `motorcycle` | Phương tiện cơ giới 2 bánh hoặc xe gắn máy (kể cả xe máy kéo theo thùng/sidecar nếu gắn liền). | Xe máy số, xe tay ga, mô tô phân khối lớn, xe ba bánh sidecar gắn xe máy. | Xe đạp thường (nếu taxonomy không có bicycle, người đi xe đạp gán person, xe đạp đưa vào ignore nếu cồng kềnh), ô tô con 3 bánh nhỏ. |
| `car` | Ô tô con chở người dưới 9 chỗ ngồi (sedan, SUV, hatchback, CUV). | Xe taxi sedan (img_040), SUV (img_012, img_034), xe con 4-5 chỗ. | Xe bán tải pickup (xem xét thuộc car hoặc truck/van tùy thùng), xe tuk-tuk (ba bánh máy). |
| `truck` | Xe tải chở hàng mọi kích cỡ (tải nhẹ có thùng hở/thùng kín, xe đầu kéo chuyên dụng). | Xe tải thùng chở hàng (img_007), xe chuyên dụng chở xe ô tô (img_002), xe tải nhỏ Suzuki/Veryca (img_001, img_036). | Xe van kín khung nguyên khối không tách rời cabin và thùng hàng. |
| `van` | Xe ô tô thân liền/thùng kín cỡ vừa phục vụ chở hàng nhẹ hoặc chở khách 9-16 chỗ. | Xe cứu thương, xe transit, xe tải nhỏ cabin liền thùng kín (img_005 phía sau). | Xe SUV cỡ lớn, xe bus cỡ lớn. |
| `ignore_region` | Vùng tập hợp vật thể quá dày đặc không thể tách rời hoặc phương tiện không thuộc taxonomy chính (xe tuk-tuk, xe đạp, đám đông chen chúc, hình phản chiếu lớn). | Đám đông đứng dày đặc ở trạm xe bus (img_032), bãi xe máy xếp sát nhau không rõ từng xe (img_030), xe xích lô/xe ba gác/xe đẩy bán hàng (img_003, img_010, img_023). | Xe rõ ràng đứng đơn lẻ nhìn rõ > 20px. |

*Quy tắc vật không thuộc lớp nào*:
- Xe ba bánh chở khách (Tuk-tuk - img_004, img_035): Không thuộc taxonomy chuẩn (`car`, `motorcycle`, `truck`, `van`). Gán vào `ignore_region` để tránh mô hình học sai đặc trưng.
- Xe thô sơ (xe đạp, xe đẩy hàng, gánh hàng rong - img_003, img_010, img_023, img_029): Chỉ gán box `person` cho người điều khiển; phần xe đẩy/xe đạp nếu gây nhiễu thì gán `ignore_region` hoặc bỏ qua.

### Thuộc tính

| Thuộc tính | Giá trị | Ngưỡng và ý nghĩa cụ thể |
|---|---|---|
| `occluded_level` | `0` | Không bị che khuất (< 5% diện tích bị che, thấy rõ toàn bộ biên dạng). |
| | `1` | Bị che ít (5% – 35% diện tích bị che bởi vật khác nhưng vẫn nhận diện rõ ràng toàn thể). |
| | `2` | Bị che nhiều (36% – 80% diện tích bị che khuất, chỉ thấy một phần đầu, đuôi hoặc thân). |
| | `3` | Bị che nghiêm trọng (> 80% diện tích bị che, hoặc vật thể mờ nhoè trong bóng tối/chuyển động không rõ). |
| `truncated` | `no` | Vật thể nằm trọn vẹn bên trong khung ảnh. |
| | `yes` | Vật thể bị cắt cụt bởi ít nhất một mép viền của bức ảnh (biên giới hạn ảnh). |
| `can_review` | `false` / `true` | Tick `true` khi gặp ca biên giới mơ hồ, tranh chấp quy tắc phân loại để đưa ra hội chẩn nhóm. |

---

## 3. Quy tắc vẽ box

1. **Người đi xe máy (Rider & Motorcycle)**:
   - **Tách riêng 2 box**: 01 box cho `person` và 01 box cho `motorcycle`.
   - **Phần bao**:
     - Box `person`: Bao toàn bộ cơ thể người từ đỉnh mũ bảo hiểm xuống đến phần thấp nhất của chân/bàn chân nhìn thấy được.
     - Box `motorcycle`: Bao trọn vẹn kết cấu xe (bánh xe trước/sau, khung sườn, tay lái, gương, bô xe, biển số).
     - *Thử nghiệm quy tắc*: Áp dụng chuẩn trên img_006 và kiểm tra chéo trên img_008 và img_026: Box person và box motorcycle chồng lấn lên nhau tự nhiên. Nếu xe chở 2 người (img_006 xe Lead, img_026), vẽ 02 box `person` riêng biệt và 01 box `motorcycle`.
2. **Xe chở hàng / Xe chở xe (Car Carrier - img_002)**:
   - Box `truck` lớn: Bao phủ toàn bộ xe kéo, rơ-moóc chuyên dụng và khung giá đỡ chịu tải (tính cả khối hàng bên trong vì đó là kích thước vật lý của xe đang lưu thông).
   - Các xe con nằm trên giá chở: Nếu nhìn rõ biên dạng > 20px, vẽ thêm box `car` (với thuộc tính `occluded_level: 1` hoặc `2`) vì các xe này vẫn mang hình dáng đặc trưng của ô tô con, giúp mô hình nhận diện xe trong mọi tình huống.
3. **Vật thể quá nhỏ (< 20 pixel theo cạnh dài nhất)**:
   - Không gán box riêng lẻ nếu đối tượng mờ ảo, nhỏ hơn 20px ở hậu cảnh xa xăm.
   - Nếu là cụm phương tiện/người ở xa: Gom vào một box `ignore_region` nếu khu vực đó dày đặc.
4. **Vật bị che khuất**:
   - Chỉ vẽ box ôm sát **phần thực tế nhìn thấy** (visible area), không vẽ phóng đại suy đoán phần bị che khuất (amodal).
   - Đặt `occluded_level` tương ứng (1, 2 hoặc 3).
5. **Tượng, Ma-nơ-canh, Đồ chơi, Phản chiếu qua gương / Pano tranh ảnh**:
   - **Tượng nghệ thuật / Tượng đài (img_017, img_027)**: KHÔNG gán nhãn `person` hay `car` (hoặc gán `ignore_region` nếu đặt trên đường) vì đây là vật tĩnh vô tri, không di chuyển tham gia giao thông.
   - **Ma-nơ-canh trong tủ kính (img_033)**: KHÔNG gán `person`.
   - **Đồ chơi / Mô hình xe trong tủ kính (img_038, img_024)**: KHÔNG gán `car`/`truck` cho xe đồ chơi thu nhỏ.
   - **Hình ảnh trên Pano / Poster / Biển quảng cáo (img_018, img_030, img_031)**: KHÔNG gán nhãn `person`/`car`.
   - **Ảnh phản chiếu qua gương chiếu hậu (img_020)**: KHÔNG gán box `car` cho ảnh ảo phản chiếu qua gương kính xe (gán `ignore_region` cho toàn bộ mặt kính gương nếu cần tránh nhiễu).
6. **Phương tiện đặc thù**:
   - **Xe Tuk-tuk (img_004, img_035)**: Không thuộc 4 lớp phương tiện tiêu chuẩn (`car`, `truck`, `van`, `motorcycle`). Gán `ignore_region` cho thân xe; gán `person` cho tài xế/hành khách nhìn thấy rõ.
   - **Xe Bus lớn (img_009, img_037)**: Do taxonomy không có lớp `bus`, xe khách/xe bus lớn tạm thời xếp vào `truck` (nếu quy ước nhóm chấp nhận xe vận tải nặng) hoặc gán `ignore_region` chờ nhóm thống nhất. Khuyến nghị cá nhân: Gán `ignore_region` hoặc đề xuất thêm lớp `bus`.
   - **Xe bán tải gắn nắp thùng cao / mui bạt (img_037, img_036, img_007)**: Nếu cabin tách rời thùng chở hàng cồng kềnh phía sau -> gán `truck`. Nếu xe pickup cá nhân thông thường đi lại -> gán `car`.

---

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe chuyên dụng chở 6-8 xe con trên giá | R-01 | Gán 01 box `truck` bao toàn bộ xe tải + giá chở hàng. Gán thêm từng box `car` cho các xe con đặt trên giá. | Giúp mô hình vừa biết kích thước choán chỗ của xe tải lớn, vừa học được đặc trưng ô tô con ở các góc nhìn. |
| `img_004` | 02 xe Tuk-tuk ba bánh Thái Lan | R-02 | Gán `ignore_region` cho khung xe Tuk-tuk. Gán box `person` cho tài xế ngồi lái. | Tuk-tuk không có trong 4 lớp xe của taxonomy; nếu ép vào `car` hay `motorcycle` sẽ làm hỏng bộ phân loại mô hình. |
| `img_009` | Xe bus vận chuyển hành khách 29 chỗ | R-03 | Gán `ignore_region` (đánh dấu `can_review: true`). | Taxonomy v0 thiếu lớp `bus`. Ép vào `truck` hay `car` đều gây tranh cãi ngữ nghĩa. Cần đưa ra họp nhóm. |
| `img_011` | Mô hình quái vật Predator bằng kim loại phế liệu | R-04 | Bỏ qua (không gán nhãn). | Không phải người thật, không phải tác nhân sinh học di chuyển trên đường. |
| `img_017` | Tượng đài kỵ sĩ cưỡi ngựa bằng đồng | R-05 | Bỏ qua không gán nhãn `person`. | Tượng điêu khắc tĩnh, không phải con người tham gia giao thông. |
| `img_018` | Pano áp phích tranh cử có hình chân dung người | R-06 | Bỏ qua không gán nhãn `person`. | Ảnh in 2D trên biển hiệu quảng cáo, không có nguy cơ va chạm vật lý đường bộ. |
| `img_020` | Ô tô phản chiếu trong gương chiếu hậu ô tô | R-07 | Gán `ignore_region` cho diện tích mặt gương phản chiếu. | Là ảnh ảo qua gương, khoảng cách và toạ độ 3D không tương ứng vị trí vật lý trong camera. |
| `img_027` | Bia mộ tạc hình xe bán tải bằng đá granite trên bãi cỏ | R-08 | Bỏ qua (không gán). | Tượng đá bia mộ trong nghĩa trang, không phải xe thật đang lưu thông. |
| `img_030` | Bãi đậu hàng chục xe máy chen chúc sát nhau | R-09 | Gán 01 box `ignore_region` bao trùm khu vực bãi xe dày đặc. | Các xe che khuất nhau quá 80%, không thể vẽ tách bạch từng box riêng lẻ chính xác. |
| `img_032` | Đám đông hơn 30 người đứng chờ xe bus | R-10 | Vẽ `person` cho các cá nhân đứng tách rời ở rìa; các cụm người đan xen đặc kín gán `ignore_region`. | Đảm bảo chất lượng nhãn, tránh box chồng lấn vô nghĩa gây nhiễu NMS khi train model. |
| `img_033` | Ma-nơ-canh nữ không mặc quần áo trong tủ kính | R-11 | Bỏ qua không gán `person`. | Ma-nơ-canh tĩnh trong cửa hàng, không phải con người. |
| `img_035` | Xe Tuk-tuk chở kiện hàng khổng lồ chằng buộc trên nóc | R-12 | Gán `ignore_region` bao cả xe và kiện hàng cồng kềnh; gán `person` cho tài xế. | Xe 3 bánh dị biệt mang hàng quá khổ, không thể xếp vừa vặn vào bất kỳ lớp xe cơ giới chuẩn nào. |
| `img_037` | Xe bán tải lắp cabin nắp thùng cao phía sau đậu cạnh xe bus | R-13 | Gán `truck` cho xe bán tải nắp thùng cao (chuyên dụng chở hàng kĩ thuật). Xe bus gán `ignore_region`. | Thùng xe đã cải tạo thành xe chuyên dụng chở thiết bị, cấu trúc tương tự xe tải nhỏ/light truck. |
| `img_038` | Tủ kính trưng bày hàng chục mô hình xe hơi đồ chơi | R-14 | Bỏ qua toàn bộ (hoặc gán 1 box `ignore_region` toàn bộ tủ). | Đồ chơi tĩnh kích thước nhỏ trong cửa hàng, gán `car` sẽ gây nhiễu nhận dạng xe thực tế. |

---

## 5. QC và phiên bản

- **Ngưỡng QC áp dụng**:
  - IoU threshold: $\ge 0.70$ đối với phương tiện rõ ràng (`car`, `truck`, `van`, `motorcycle`).
  - IoU threshold: $\ge 0.60$ đối với `person` do người có nhiều chi và tư thế vận động biến thiên.
  - Tỷ lệ trùng khớp nhãn lớp (Class Match Rate): $\ge 90\%$.
- **Thẩm quyền quyết định ca mơ hồ**:
  - Các ca gắn cờ `can_review: true` được tổng hợp và đưa ra thảo luận tại buổi họp Calibration nhóm (Pha 2). Quyết định cuối cùng thuộc về Lead/Reviewer của nhóm dựa trên biểu quyết thống nhất.
- **Lịch sử thay đổi**:

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu sơ sài từ giảng viên. | Khởi tạo ban đầu. |
| `v1.0-ind-102` | - Làm rõ quy tắc tách 2 box cho người đi xe máy (`person` + `motorcycle`).<br>- Quy định cụ thể cách xử lý xe cõng xe (img_002), tượng/ma-nơ-canh/ảnh in (img_011, 017, 018, 027, 033, 038), gương phản chiếu (img_020).<br>- Phân loại rõ đối tượng ngoài taxonomy (Tuk-tuk, bus) dùng `ignore_region`. | Giải quyết 14 ca khó phát sinh trong 40 ảnh dataset chuẩn bị cho Pha 2. |
