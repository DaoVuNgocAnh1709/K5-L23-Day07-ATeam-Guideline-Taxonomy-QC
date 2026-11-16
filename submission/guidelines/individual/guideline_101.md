# Guideline gán nhãn — v0.1 — 101

- Phiên bản: v0.1 (bản cá nhân Pha 1)
- Ngày: 2026-11-16
- Người viết / duyệt: 101
- Công cụ: CVAT, xuất COCO 1.0 (tắt Save images)

---

## 1. Mục tiêu

Bộ nhãn này dùng để huấn luyện mô hình phát hiện **người và phương tiện giao thông 2D (Bounding Box)** trên ảnh đường phố, phục vụ bài toán camera giám sát giao thông hoặc hỗ trợ lái xe (ADAS).

Khi gán nhãn cần tránh 3 nhóm lỗi ảnh hưởng trực tiếp đến chất lượng mô hình:
- **Bỏ sót đối tượng thật (False Negative):** Đây là lỗi nguy hiểm nhất. Nếu bỏ sót người đi bộ sang đường hay người đi xe máy (`img_014`, `img_034`), mô hình sẽ không nhận diện được chướng ngại vật trên đường.
- **Gán nhầm vật không thật (False Positive):** Gán nhầm tượng điêu khắc (`img_011`, `img_017`, `img_027`), ma-nơ-canh (`img_033`), đồ chơi (`img_038`), hình in trên biển quảng cáo (`img_018`) hoặc ảnh phản chiếu qua gương (`img_020`) sẽ làm mô hình học sai đặc trưng và báo động giả.
- **Vẽ ranh giới box thiếu nhất quán:** Gộp chung người với xe máy thành 1 box, hoặc kéo box trùm cả đống hàng hóa chất cao trên nóc xe (`img_035`) sẽ làm sai lệch tỷ lệ khung hình và kích thước thực của phương tiện.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp đối tượng (Classes)

Qua khảo sát 40 ảnh thực tế, bộ nhãn gốc `v0` đang thiếu lớp cho **xe buýt cỡ lớn** và **xe 3 bánh tuk-tuk** (xuất hiện nhiều lần trong bộ ảnh). Bản cá nhân này đề xuất giữ 6 lớp cũ và bổ sung thêm `bus`, `tuk_tuk`:

| Lớp | Định nghĩa (một câu) | Ví dụ trong 40 ảnh | Phản ví dụ / dễ nhầm với (không gán vào lớp này) |
|---|---|---|---|
| `car` | Ô tô con chở người từ 4–7 chỗ (sedan, hatchback, SUV, station wagon, taxi). | Ô tô con trên sàn xe tải (`img_002`), SUV xám (`img_012`), sedan đen (`img_023`), wagon đen dưới sàn (`img_024`), taxi (`img_040`). | Mô hình ô tô đồ chơi trong tủ kính (`img_038`), xe đạo cụ hoạt hình Dangermouse (`img_019`), xác vỏ xe vàng đặt trên nóc xe khác (`img_024`), xe bán tải có thùng hàng (`img_001` → `truck`). |
| `truck` | Xe tải chở hàng, xe đầu kéo rơ-moóc và xe bán tải (pickup) có phần thùng hàng tách biệt với cabin lái. | Bán tải trắng/đen (`img_001`, `img_012`), đầu kéo chở ô tô (`img_002`), tải thùng kín/bạt (`img_007`, `img_036`), bán tải cổ (`img_028`), bán tải gắn nắp chụp thùng `Master 75` (`img_037`). | Tượng đài xe bán tải bằng đá (`img_027`), xe van thân liền khối (`img_005`, `img_016` → `van`), xe đồ chơi (`img_038`). |
| `van` | Xe thùng kín cỡ nhỏ có thân liền khối (minivan, panel van, xe khách nhỏ 7–12 chỗ có khoang lái thông với khoang sau). | Xe minivan màu bạc biển `51LD-70-65` (`img_005`), xe van trắng chạy trên phố (`img_016`), xe van nhỏ ở xa (`img_002`). | Xe buýt cỡ trung/lớn từ 16 chỗ trở lên (`img_009`, `img_037` → `bus`), xe bán tải lắp thêm nắp thùng cao (`img_037` → `truck`). |
| `bus` *(Đề xuất thêm)* | Xe buýt hoặc xe khách cỡ trung và lớn chở nhiều người (thân dài, từ 16 chỗ trở lên). | Xe khách kem–nâu ở Huế (`img_009`), xe buýt ban đêm (`img_031`), xe buýt GO Transit xanh lá (`img_037`), xe buýt ở Bangkok (`img_040`). | Xe minivan chở khách cỡ nhỏ (`img_005`, `img_016` → `van`). |
| `tuk_tuk` *(Đề xuất thêm)* | Xe cơ giới 3 bánh có mui che dùng chở khách hoặc chở hàng (tuk-tuk Thái Lan, auto-rickshaw Ấn Độ). | Hai xe tuk-tuk xanh đỗ vỉa hè (`img_004`), tuk-tuk ở Bangkok (`img_016`, `img_040`), auto-rickshaw vàng chở bao tải (`img_035`). | Xích lô 3 bánh đạp bằng sức người (`img_006`, `img_023`, `img_034` → không gán xe), mô tô gắn thuyền sidecar (`img_022` → `motorcycle`). |
| `motorcycle` | Xe cơ giới 2 bánh (xe máy số, xe tay ga, mô tô phân khối lớn) và mô tô có gắn thuyền bên hông (sidecar). | Xe máy đang chạy hoặc đỗ (`img_005`, `img_006`, `img_008`, `img_010`, `img_026`), dàn mô tô độ (`img_011`), mô tô sidecar (`img_022`). | Xe đạp không động cơ (`img_003`, `img_006`, `img_029`), xích lô (`img_023`), xe 3 bánh tuk-tuk (`img_004`, `img_035` → `tuk_tuk`). |
| `person` | Con người thật ở mọi tư thế (đi bộ, đứng, ngồi ăn uống, đang lái hoặc ngồi trên các loại xe). | Khách ngồi quán (`img_001`), người gánh hàng (`img_003`, `img_010`), người đi xe máy (`img_005`, `img_026`), người sang đường (`img_014`, `img_034`), người đạp/ngồi xích lô (`img_023`). | Tượng Predator (`img_011`), tượng đồng cưỡi ngựa (`img_017`), ma-nơ-canh (`img_033`), ảnh người trên biển quảng cáo (`img_018`, `img_030`), búp bê đồ chơi (`img_038`), chuột hoạt hình (`img_019`). |
| `ignore_region` | Vùng bỏ qua (không phải lớp vật thể): dùng bao phủ đám đông người/bãi xe chồng lấp dày đặc không thể tách rời từng chiếc, hoặc cụm vật ở xa cao `< 20 px`. | Đám đông lễ hội/nhà chờ xe buýt (`img_011`, `img_019`, `img_032`), bãi xe máy xếp dày (`img_003`, `img_021`, `img_030`), luồng người/xe nhỏ li ti ở xa (`img_013`, `img_025`, `img_031`, `img_039`). | Không được lạm dụng `ignore_region` để bọc các đối tượng ở gần nhìn rõ và có chiều cao `>= 20 px`. |

**Xử lý các vật nằm ngoài danh mục lớp (Out-of-taxonomy):**
- **Phương tiện thô sơ không động cơ:** Xe đạp (`img_003`, `img_006`, `img_029`), xích lô đạp chân (`img_006`, `img_009`, `img_023`, `img_034`, `img_039`), xe đẩy bán hàng rong và quang gánh (`img_003`, `img_010`, `img_025`) → **Không vẽ box cho chiếc xe**, nhưng người thật đang điều khiển hoặc ngồi trên xe đó vẫn phải gán lớp `person` bình thường.
- **Động vật thật hoặc giả:** Chim đậu trên mũ tượng (`img_017`), tượng ngựa đồng (`img_017`), mô hình ngựa trắng trên thùng bán tải (`img_028`) → **Không gán**.

---

### 2.2. Thuộc tính (Attributes) và ngưỡng đo cụ thể

Mỗi đối tượng (trừ `ignore_region`) cần điền đủ 3 thuộc tính sau trong bảng **Objects** của CVAT:

| Thuộc tính | Giá trị | Ý nghĩa & Ngưỡng định lượng để chấm |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3`<br/>*(Mặc định: `0`)* | Ước lượng theo **% diện tích thực của vật bị vật thể khác ở phía trước che khuất**:<br/>• **`0` (Thấy rõ):** Bị che `0%`, nhìn thấy trọn vẹn phần thân vật trong ảnh.<br/>• **`1` (Che ít):** Bị che từ **`1% – 30%`** diện tích (VD: xe máy bị chân người ngồi che một phần; hàng ô tô tầng trên bị lan can sàn xe che nhẹ ở `img_002`).<br/>• **`2` (Che nhiều):** Bị che từ **`31% – 70%`** diện tích (VD: khách ngồi ăn bị bàn ghế/người đứng trước che nửa người ở `img_001`; ô tô tầng dưới bị khung dầm thép che ở `img_002`; tài xế ngồi trong khoang lái tuk-tuk ở `img_035`).<br/>• **`3` (Che rất nặng):** Bị che **`> 70%` đến `90%`** diện tích nhưng phần lộ ra vẫn đủ để nhận diện chắc chắn đúng lớp. Nếu bị che `> 90%` hoặc chỉ lộ một mẩu nhỏ không phân biệt được lớp gì thì không vẽ box lẻ.<br/>*(Lưu ý: Phải chọn thuộc tính `occluded_level` trong bảng Objects bên phải của CVAT; không bấm phím `Q` vì cờ `occluded` mặc định của CVAT sẽ bị script bỏ qua).* |
| `truncated` | `no`, `yes`<br/>*(Mặc định: `no`)* | Xác định vật có bị cắt ngang bởi 1 trong 4 cạnh mép ảnh hay không:<br/>• **`no`:** Toàn bộ phần nhìn thấy của vật nằm gọn trong khung ảnh (cách mép ảnh tối thiểu `2 px`).<br/>• **`yes`:** Cơ thể người hoặc thân xe chạm sát biên ảnh và bị khung hình cắt mất một phần (ước lượng mất **`>= 5%`** thể tích vật ở ngoài rìa ảnh, VD: đuôi xe bán tải trắng ở mép phải `img_001`, người đi xe máy ở mép dưới `img_005`, `img_026`, `img_040`, đầu/đuôi ô tô ở hai bên mép `img_015`).<br/>*(Lưu ý: Vật nằm giữa ảnh nhưng bị xe khác che khuất thì tính vào `occluded_level`, không chọn `truncated = yes`).* |
| `can_review` | tick / không<br/>*(Mặc định: không)* | Tick chọn khi gặp ca biên chưa chắc chắn về cách chọn lớp, cách lấy ranh giới box hoặc ảnh bị nhòe mạnh (`img_014`, `img_015`, `img_037`) để đánh dấu đưa ra cuộc họp Calibration. Nếu có vấn đề cấp ảnh mà chưa vẽ được box thì ghi vào `submission/qc/image_issues.csv`. |

---

## 3. Quy tắc vẽ box (Gắn mã `R-01` – `R-09`)

1. **`R-01` — Vẽ box ôm sát phần nhìn thấy thực tế (Visible Box) và không vượt biên ảnh:**
   - Kéo 4 cạnh box chạm sát các điểm ngoài cùng của phần thực thể **đang nhìn thấy được** (bao gồm cả mũ bảo hiểm, bàn tay/bàn chân của `person`; gương chiếu hậu, bánh xe, cản trước/sau của phương tiện).
   - Không vẽ phỏng đoán phần bị che khuất hoàn toàn sau tường, biển hiệu hay xe khác. Nếu một chiếc xe bị cột đèn hoặc thanh khung rơ-moóc mảnh chắn ngang ở giữa làm chia đôi phần nhìn thấy (`img_002`), vẫn chỉ vẽ **1 box duy nhất** bao trọn các phần nhìn thấy của chiếc xe đó và tăng `occluded_level`.
   - Khi vẽ sát mép ảnh, tuyệt đối không kéo chuột vượt ra ngoài kích thước pixel của ảnh (đặc biệt chú ý mép dưới `img_026.jpg` có kích thước `784x1024`).

2. **`R-02` — Loại bỏ đối tượng không có thật (tượng, ma-nơ-canh, đồ chơi, hình in, đạo cụ, vỏ xe trưng bày):**
   - Chỉ gán nhãn cho người sống thật và phương tiện cơ giới thật.
   - Không vẽ box cho: mô hình tượng Predator (`img_011`), tượng đồng người cưỡi ngựa (`img_017`), tượng đài xe bán tải trên bệ đá (`img_027`), mô hình ngựa trắng trên thùng xe (`img_028`), ma-nơ-canh trong tủ kính (`img_033`), xe hơi/búp bê đồ chơi (`img_038`), ảnh chân dung người trên biển quảng cáo/băng rôn (`img_018`, `img_030`), vỏ xe hóa trang chuột hoạt hình Dangermouse (`img_019`), và phần xác vỏ xe màu vàng không bánh đặt trên giá nóc ô tô khác (`img_024`).

3. **`R-03` — Không gán ảnh phản chiếu và bộ phận của xe chủ (Ego-vehicle):**
   - Không gán người hoặc phương tiện chỉ xuất hiện qua ảnh phản chiếu trong gương chiếu hậu (`img_020`, `img_036`), phản chiếu trên mặt kính cửa hàng (`img_033`) hoặc bóng đổ dưới mặt đường mưa (`img_013`, `img_015`, `img_031`).
   - Không gán phần gương chiếu hậu, khung cửa hay thân vỏ của chính chiếc xe đặt camera chụp ra bức ảnh (`img_020`).

4. **`R-04` — Tách riêng box người và box phương tiện (Xe máy, Tuk-tuk, Sidecar):**
   - Với người đi xe máy: luôn **vẽ tách riêng** từng box `person` cho mỗi người ngồi trên xe (người cầm lái, người ngồi sau, trẻ em) và **1 box `motorcycle` riêng** cho chiếc xe.
   - Cạnh trên của box `motorcycle` chỉ ôm đến điểm cao nhất của bản thân chiếc xe (gương chiếu hậu hoặc tay lái), **không kéo trùm lên đầu người ngồi trên xe** (`img_005`, `img_006`, `img_008`, `img_010`, `img_016`, `img_026`, `img_035`).
   - Với mô tô gắn thuyền 3 bánh (sidecar ở `img_022`): vẽ 1 box `motorcycle` bao cả thân mô tô lẫn phần thuyền gắn bên hông, đồng thời vẽ riêng 2 box `person` cho 2 người ngồi trên xe.

5. **`R-05` — Quy tắc gán Xe chở xe (Car Carrier — `img_002.jpg`):**
   - Khi một xe tải/đầu kéo chuyên dụng đang chở các ô tô thật khác trên sàn: vẽ **1 box `truck`** ôm toàn bộ xe đầu kéo + khung rơ-moóc, **đồng thời vẽ riêng từng box `car`** cho cả 8 chiếc ô tô con đang nằm trên 2 tầng sàn (gán `occluded_level = 1` cho các xe tầng trên và `occluded_level = 2` cho các xe tầng dưới bị khung sắt che).

6. **`R-06` — Ngưỡng chiều cao tối thiểu `20 px` và cách dùng `ignore_region` cho đám đông:**
   - Theo ngưỡng `min_box_height_px = 20`: các vật thể ở xa có chiều cao phần nhìn thấy **`< 20 px`** thì không vẽ box đối tượng cá nhân.
   - Dùng box `ignore_region` để bao phủ trong 2 trường hợp: (a) cụm người/phương tiện ở xa có chiều cao `< 20 px` (`img_013`, `img_025`, `img_031`, `img_039`), hoặc (b) đám đông người đứng chen chúc / bãi xe máy đỗ hoặc kẹt xe chồng lấp dày đặc không thể tách bạch từng chiếc (`img_003`, `img_008`, `img_011`, `img_019`, `img_021`, `img_030`, `img_032`, `img_034`).
   - **Cách phân ranh giới giữa vùng gán lẻ và `ignore_region` (tránh lỗi `bat_dong_ignore`):** Chỉ gán từng box lẻ (`person`, `motorcycle`, `car`) cho **hàng người/xe đầu tiên ở tiền cảnh nhìn thấy rõ cả phần thân xe/bánh xe phía dưới** và các ô tô lớn nhìn rõ khối. Từ lớp xe thứ 2 trở về phía sau (nơi chỉ lộ đầu/mũ bảo hiểm hoặc bị che khuất phần thân xe `> 70%`), gom chung vào **1 box `ignore_region` lớn nằm ngang**, cạnh dưới của `ignore_region` dừng ngay phía trên hàng xe tiền cảnh (không kéo trùm đè lên `>= 50%` diện tích các box lẻ đã vẽ ở tiền cảnh).

7. **`R-07` — Ranh giới phương tiện khi chở hàng hóa cồng kềnh / quá khổ:**
   - Nếu hàng hóa nằm gọn bên trong lòng thùng xe bán tải (`img_012`), vẽ box `truck` theo khung xe bình thường.
   - Nếu hàng hóa hoặc vật trang trí chất quá khổ, nhô cao hoặc vươn dài vượt hẳn ra ngoài kết cấu khung xe/cơ thể người (như đống bao tải khổng lồ chất trên nóc xe tuk-tuk ở `img_035`, thanh dài màu trắng vác ngang qua đường ở `img_029`, mô hình ngựa đứng trên thùng xe ở `img_028`): **box phương tiện chỉ ôm sát khung vật lý thật của chiếc xe (`tuk_tuk`, `motorcycle`, `truck`)**, không kéo box bao lấy phần hàng hóa/vật chất thêm đó.

8. **`R-08` — Xử lý phương tiện thô sơ ngoài Taxonomy (Xe đạp, Xích lô, Xe hàng rong):**
   - Không vẽ box phương tiện cho xe đạp, xích lô, xe đẩy hàng rong hay quang gánh (`img_003`, `img_006`, `img_009`, `img_010`, `img_023`, `img_029`, `img_034`, `img_039`).
   - Những người thật đang đạp xe, đạp xích lô hoặc ngồi làm hành khách trên xích lô vẫn phải vẽ box `person` đầy đủ (chọn `occluded_level` tương ứng nếu bị mui/khung xích lô che một phần cơ thể).

9. **`R-09` — Phân biệt Xe bán tải gắn nắp thùng cao & Xử lý vật bị nhòe chuyển động (Motion Blur):**
   - Xe bán tải lắp thêm nắp chụp thùng cao phía sau (`Master 75 Truck Cap` ở `img_016`, `img_037`) vẫn có kết cấu cabin đầu xe tách rời khỏi thùng hàng → xếp vào lớp **`truck`** (không gán là `van`).
   - Với các ảnh chụp tối/mưa có phương tiện đang chạy bị nhòe vệt dài do tốc độ màn trập chậm (`img_014`, `img_015`): nếu đối tượng vẫn nhìn ra hình dáng người/xe và cao `>= 20 px` thì vẽ box ôm phần thân chính nhìn thấy (bỏ qua phần bóng mờ kéo đuôi quá loãng) và tick `can_review` để nhóm kiểm tra lại.

---

## 4. Ca khó khảo sát từ 40 ảnh

Trong bảng dưới đây, mỗi tình huống được gắn nhãn **[Loại/Gán theo quy tắc]** (đã rõ để áp dụng ngay) hoặc **[Cần hỏi reviewer / Họp Pha 2]** (ca biên dễ gây bất đồng giữa các thành viên, cần chốt chung trước khi khóa bản v1.0):

| Ảnh | Tình huống thực tế | Rule ID | Quyết định ban đầu của `101` | Lý do |
|---|---|---|---|---|
| `img_001` | Khách ngồi quán buổi tối bị bàn ghế/người đứng trước che nhiều mức; xe bán tải trắng bị cắt đuôi ở mép phải. | `R-01` | **[Gán theo quy tắc]** Gán `person` (`occluded_level = 1, 2`) cho từng khách nhìn rõ phần thân trên; gán `truck` (`truncated = yes`) cho bán tải trắng. | Xe có thùng lửng tách rời cabin là pickup (`truck`); phần đuôi chạm mép ảnh nên `truncated = yes`. |
| `img_002` | Xe đầu kéo Renault chở 8 ô tô con xếp trên 2 tầng sàn rơ-moóc; góc phải có 1 người đứng và 1 xe van nhỏ ở xa. | `R-05` | **[Cần hỏi reviewer / Họp Pha 2]** Vẽ 1 box `truck` cho xe đầu kéo + **8 box `car` riêng** cho 8 xe con trên sàn (`occluded_level = 1` ở tầng trên, `2` ở tầng dưới) + 1 `person` + 1 `van`. | Các xe trên sàn đều là ô tô thật nhìn rõ; cần họp nhóm chốt để tránh người vẽ đủ 9 box xe, người lại chỉ vẽ 1 box `truck`. |
| `img_003` | Người gánh mít, người dắt xe đạp ở gần; phía xa góc trên là dãy xe máy đỗ chồng lấp rất nhỏ. | `R-06`, `R-08` | **[Gán theo quy tắc]** Chỉ gán `person` cho người thật (bỏ qua quang gánh và xe đạp); vẽ `ignore_region` cho dãy xe máy nhỏ ở hậu cảnh. | Xe đạp và gánh hàng nằm ngoài taxonomy; dãy xe ở xa bị chồng lấp và nhiều chiếc cao `< 20 px`. |
| `img_004` | Hai chiếc xe 3 bánh tuk-tuk màu xanh đỗ cạnh nhau ban đêm, có tài xế ngồi trong xe bên phải. | `R-04` | **[Cần hỏi reviewer / Họp Pha 2]** Đề xuất thêm lớp `tuk_tuk` cho 2 xe này, và vẽ riêng 1 box `person` (`occluded_level = 2`) cho người ngồi trong xe. | Bản `v0` chưa có lớp cho xe 3 bánh có mui; nếu ép vào `motorcycle` hay `car` đều làm lệch đặc trưng lớp. |
| `img_005` | Ba người đi trên 2 xe máy bị cắt phần dưới ở mép ảnh; phía sau là xe minivan bạc biển `51LD-70-65`. | `R-01`, `R-04` | **[Gán theo quy tắc]** Tách riêng 3 box `person` (`truncated = yes` ở mép dưới) và các box `motorcycle`; xe màu bạc phía sau gán `van` (`occluded_level = 2`). | Tách riêng người và xe giúp box xe máy ôm chuẩn thân xe; chiếc xe bạc là minivan thân liền khối nên thuộc lớp `van`. |
| `img_006`, `img_023` | Người đi xe đạp (`img_006`) và người đạp xích lô chở khách (`img_006`, `img_023`) lưu thông cùng xe máy, ô tô. | `R-04`, `R-08` | **[Loại theo quy tắc]** Gán bình thường cho `person`, `motorcycle`, `car`; **không vẽ box phương tiện** cho chiếc xe đạp và xích lô (nhưng người đạp và khách ngồi trên xích lô vẫn gán `person`). | Xe đạp và xích lô không có động cơ nên loại khỏi các lớp phương tiện cơ giới. |
| `img_009`, `img_040` | Xe khách cỡ trung ở Huế (`img_009`) và các xe buýt lớn ở Bangkok (`img_040`). | `R-01` | **[Cần hỏi reviewer / Họp Pha 2]** Đề xuất thêm lớp `bus` vào `taxonomy.json` để gán các xe này. | Xe buýt chở khách cỡ lớn không phù hợp với định nghĩa "xe thùng kín cỡ nhỏ" của `van` hay xe chở hàng `truck`. |
| `img_011` | Tượng mô hình quái vật Predator đứng cạnh hàng mô tô độ; phía xa là đám đông người tại lễ hội. | `R-02`, `R-06` | **[Loại theo quy tắc]** **Không gán** tượng Predator; gán `motorcycle` cho các xe độ ở gần, `person` cho người thật đứng rõ, và `ignore_region` cho đám đông ở xa. | Predator là mô hình trưng bày (phản ví dụ của `person`); đám đông phía sau quá nhỏ và đứng chồng lên nhau. |
| `img_012` | Xe bán tải đen đỗ sát mép dưới với thùng sau chất đầy thùng giấy và túi đồ; phía trước có SUV xám và xe tải. | `R-07` | **[Gán theo quy tắc]** Gán `truck` (`truncated = yes`) cho bán tải đen (box bao gọn thân xe và hàng nằm trong lòng thùng), `car` cho SUV xám, `truck` cho xe tải phía trước. | Hàng hóa nằm gọn bên trong thành thùng xe bán tải, không bị nhô cao quá khổ ra ngoài khung thiết kế của xe. |
| `img_014`, `img_015` | Người đi bộ và ô tô đỗ thì nét, nhưng xe máy chạy ngang (`img_014`) và ô tô chạy ngoài mưa (`img_015`) bị nhòe vệt dài (motion blur). | `R-09` | **[Cần hỏi reviewer / Họp Pha 2]** Gán bình thường các người/xe nét; với các xe và người bị nhòe chuyển động vẫn vẽ box ôm phần thân chính nhìn thấy và tick `can_review = true`. | Cần thảo luận nhóm ở Pha 2 để thống nhất cách lấy biên box khi ảnh bị nhòe chuyển động. |
| `img_016` | Trên phố có đủ: 2 xe tuk-tuk, 1 người đi xe tay ga, 1 xe van trắng và 1 xe bán tải gắn khung nắp thùng kín bên trái. | `R-04`, `R-09` | **[Cần hỏi reviewer / Họp Pha 2]** Gán 2 `tuk_tuk`, 1 `van` (xe trắng), 1 `truck` (bán tải lắp thùng kín bên trái), 1 `motorcycle` và tách riêng các box `person`. | Ảnh xuất hiện cùng lúc `van`, `truck` nắp cao và `tuk_tuk`, rất dễ làm các thành viên phân loại khác nhau nếu không chốt trước. |
| `img_017`, `img_027` | Tượng đồng người cưỡi ngựa (`img_017`) và tượng đài xe bán tải cổ đặt trên bệ đá (`img_027`). | `R-02` | **[Loại theo quy tắc — Ca đối chứng]** **Để trống 0 annotation** cho cả hai ảnh (vẫn giữ ảnh trong file xuất COCO). | Toàn bộ là tượng điêu khắc/đài tưởng niệm. Nếu hai người cùng vẽ `person` hoặc `truck` thì điểm IAA vẫn cao nhưng thực chất là **cả hai cùng sai**. |
| `img_018`, `img_030` | Biển quảng cáo in hình người lớn trên bãi đỗ ô tô (`img_018`) và tấm biển in hình người ở góc trên bãi giữ xe máy (`img_030`). | `R-02`, `R-06` | **[Loại theo quy tắc]** **Không gán** hình người trên biển quảng cáo; ở `img_018` gán `car` (`occluded_level = 1, 2`) cho dãy xe sau hàng rào; ở `img_030` gán người bảo vệ thật, hàng xe máy tiền cảnh và bọc `ignore_region` cho biển xe máy chồng lấp phía sau. | Hình in 2D trên bảng hiệu không phải người thật; bãi xe máy ở `img_030` có hàng trăm xe đè lên nhau không thể tách lẻ ở phần xa. |
| `img_019` | Xe đạo cụ hóa trang hình chuột hoạt hình "Dangermouse" màu vàng trong lễ diễu hành; hai bên vỉa hè là biển người đứng xem. | `R-02`, `R-06` | **[Cần hỏi reviewer / Họp Pha 2]** Đề xuất **không gán `car`** cho vỏ xe hoạt hình (hoặc tick `can_review` để nhóm biểu quyết), chỉ gán `person` cho người thật lái bên trong cùng những người đứng rõ ở gần, và bọc `ignore_region` cho đám đông khán giả. | Tuy chạy được trong lễ diễu hành nhưng hình dáng bên ngoài là mô hình nhân vật hoạt hình giả tưởng, đưa vào lớp `car` dễ làm nhiễu đặc trưng. |
| `img_020`, `img_033`, `img_038` | Gương chiếu hậu xe chủ soi bóng ô tô phía sau (`img_020`), ma-nơ-canh cạnh vách kính (`img_033`), tủ kính bày mô hình ô tô đồ chơi và búp bê (`img_038`). | `R-02`, `R-03` | **[Loại theo quy tắc]** Cả 3 ảnh đều **để trống 0 annotation** (giữ nguyên tên ảnh trong COCO). | Gồm toàn bộ các đối tượng bị loại theo quy tắc: gương của xe chủ + ảnh phản chiếu (`img_020`), ma-nơ-canh + bóng kính (`img_033`), đồ chơi (`img_038`). |
| `img_021` | Dòng người và xe máy kẹt cứng tại ngã tư: hàng đầu tiên nhìn rõ cả người lẫn thân xe/biển số, nhưng từ hàng thứ 2 trở về phía xa chỉ thấy biển mũ bảo hiểm và vai áo đè chồng lên nhau. | `R-04`, `R-06` | **[Cần hỏi reviewer / Họp Pha 2]** Gán box lẻ (`person`, `motorcycle`) cho hàng xe đầu tiên sát tiền cảnh và gán `car` cho các ô tô bên trái; gom toàn bộ đám đông xe máy từ hàng thứ 2 trở về xa vào **1 box `ignore_region` ngang** (không kéo đè lên các box tiền cảnh). | Nếu không chốt ranh giới cắt giữa hàng xe tiền cảnh và vùng `ignore_region`, người vẽ sâu vào hàng 2–3 còn người kia bọc `ignore_region` đè lên sẽ bị script báo lỗi `bat_dong_ignore` và `bo_sot_thua`. |
| `img_022` | Ảnh đen trắng chụp mô tô cổ gắn thuyền sidecar bên hông với 2 người ngồi. | `R-04` | **[Gán theo quy tắc]** Vẽ 1 box `motorcycle` ôm cả thân mô tô và thuyền sidecar; vẽ tách 2 box `person` cho người cầm lái và người ngồi bên thuyền. | Sidecar là mô tô 2 bánh gắn thêm thuyền phụ bên hông nên thuộc `motorcycle`, không phải `tuk_tuk`. |
| `img_024` | Trong phòng trưng bày: ô tô AMC Pacer màu đen đỗ dưới sàn, trên giá nóc của nó đặt một bộ khung vỏ xe màu vàng không có bánh. | `R-02`, `R-07` | **[Cần hỏi reviewer / Họp Pha 2]** Gán `car` cho chiếc xe đen dưới sàn (box chỉ ôm thân xe đen) và các ô tô nguyên vẹn xung quanh; **không gán** cái vỏ xe màu vàng trên nóc. | Phần màu vàng trên nóc chỉ là xác vỏ xe rỗng đặt trang trí trên giá nóc, không phải một chiếc ô tô hoàn chỉnh. |
| `img_026` | Hai người ngồi trên xe máy đỏ chạy trong hầm Sài Gòn, bánh xe bị cắt sát mép dưới ảnh (`784x1024`). | `R-01`, `R-04` | **[Gán theo quy tắc]** Tách 2 box `person` (nam lái xe và nữ ngồi sau) + 1 box `motorcycle` (`truncated = yes`), chú ý dừng cạnh dưới đúng biên ảnh. | Cần cẩn thận không kéo lố tọa độ `y > 1024` ở mép dưới và tách rõ 2 người ngồi sát nhau trên xe. |
| `img_028`, `img_029`, `img_035` | Mô hình ngựa trắng đứng trên thùng bán tải (`img_028`), thanh dài trắng vác ngang trên xe máy & xe đạp (`img_029`), bao tải chất cao gấp đôi chiếc tuk-tuk (`img_035`). | `R-07`, `R-08` | **[Cần hỏi reviewer / Họp Pha 2]** Box phương tiện (`truck`, `motorcycle`, `tuk_tuk`) **chỉ ôm sát khung xe thật**, không kéo trùm lên con ngựa giả, thanh dài hay đống bao tải khổng lồ trên nóc. Ở `img_029` bỏ qua chiếc xe đạp bên phải. | Nếu một người chỉ ôm khung xe còn người kia bao cả đống bao tải trên nóc (`img_035`) thì IoU sẽ rất thấp; cần chốt quy tắc `R-07` cho cả nhóm. |
| `img_036`, `img_037` | Xe tải nhỏ CMC Veryca thùng bạt (`img_036`); xe bán tải trắng GO Transit lắp nắp chụp thùng cao `Master 75` đứng cạnh xe buýt xanh (`img_037`). | `R-09` | **[Cần hỏi reviewer / Họp Pha 2]** Gán `truck` cho cả xe tải nhỏ CMC (`img_036`) và xe bán tải trắng lắp nắp thùng `Master 75` (`img_037`); chiếc xe xanh bên trái ở `img_037` gán `bus`. | Ở `img_037`, phần `Master 75` là nắp chụp lắp lên thùng xe bán tải (khoang cabin vẫn tách rời thùng sau), nên bản chất là `truck` chứ không phải `van`. |

---

## 5. QC và phiên bản

### 5.1. Ngưỡng QC áp dụng (Giữ cố định theo `submission/taxonomy.json -> rules`)
- **Chiều cao box tối thiểu (`min_box_height_px`):** `20 px` (vật nhỏ hơn `< 20 px` không gán lẻ, gom vào `ignore_region`).
- **Ngưỡng ghép cặp box khi tính IAA (`match_iou`):** `IoU >= 0.50` (ghép cặp 1–1 giữa hai người gán).
- **Ngưỡng IoU đạt chuẩn của từng cặp box:**
  - Box lớn (chiều cao `>= 50 px`, `large_px = 50`): **`IoU >= 0.85`** (`iou_thr_large`).
  - Box nhỏ (chiều cao từ `20 px` đến `< 50 px`): **`IoU >= 0.70`** (`iou_thr_small`).
- **Ngưỡng đồng thuận toàn lô (IAA) trước Calibration:**
  - IoU trung bình trên các cặp box ghép được: **`>= 0.85`** (`iaa_iou`).
  - Đồng thuận lớp trên các cặp box ghép được: **`>= 95%`** (`iaa_class`).
- **Ngưỡng duyệt QC cuối (`error_rate_img_max`):**
  - Duyệt toàn bộ lô 40/40 ảnh; tỷ lệ ảnh có lỗi **`<= 2%`** (tức `0/40` ảnh lỗi) và không có lỗi lặp hệ thống mới được chốt **Release**.

### 5.2. Quy ước xử lý ca mơ hồ và quản lý phiên bản
- Trong lúc gán độc lập (Pha 3), nếu phân vân về một đối tượng thì vẽ box theo phương án hợp lý nhất và tick `can_review` (hoặc ghi vào `submission/qc/image_issues.csv` nếu chưa vẽ được box), không tự ý hỏi người gán cùng ảnh.
- Các ảnh không có đối tượng thật (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) vẫn giữ nguyên trong Task CVAT để khi xuất COCO có tên trong danh sách `"images"` với `0` annotation.
- Mọi thay đổi quy tắc làm thay đổi cách chọn vật, lớp, ranh giới box hoặc thuộc tính ở Pha 5 đều phải nâng phiên bản lên `v1.1` (`semantic`) và rà soát lại toàn bộ lô 40 ảnh.

| Phiên bản | Thay đổi chính | Lý do (dẫn chiếu ca) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên (6 lớp cơ bản, thuộc tính mô tả chung chung, chưa có mục ca khó). | Bản gốc dùng để khảo sát lỗi mơ hồ. |
| `v0.1` | • Đề xuất thêm 2 lớp `bus`, `tuk_tuk`; bổ sung định nghĩa 1 câu, ví dụ và phản ví dụ cho cả 8 lớp.<br/>• Đặt ngưỡng `%` cụ thể cho `occluded_level` (`0%`, `1–30%`, `31–70%`, `>70%`) và `truncated` (`>= 5%` cắt ở mép ảnh).<br/>• Đưa ra 9 quy tắc vẽ box (`R-01` – `R-09`) và bảng phân tích các ca khó trong 40 ảnh. | Khắc phục các tình huống mà `v0` chưa trả lời được sau khi xem hết 40 ảnh: xe chở xe (`img_002`), tuk-tuk (`img_004`, `img_016`, `img_035`), xe buýt (`img_009`, `img_037`, `img_040`), tượng/ma-nơ-canh/đồ chơi/phản chiếu (`img_011`, `img_017`, `img_020`, `img_027`, `img_033`, `img_038`), bán tải nắp thùng cao (`img_037`) và hàng hóa quá khổ (`img_029`, `img_035`). |
