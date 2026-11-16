# Guideline gán nhãn thống nhất — v1.0 — Nhóm ATeam

- Phiên bản: `v1.0` (Đã khóa sau cuộc họp Pha 2 để gán độc lập ở Pha 3)
- Ngày: `2026-11-16`
- Người viết / duyệt: Nhóm `ATeam` (`101`, `102`, `103`, `104`, `105` — điều phối: `101`)
- Công cụ & định dạng xuất: CVAT, xuất `COCO 1.0` (bắt buộc tắt *Save images*)

---

## 1. Mục tiêu

Bộ nhãn dùng để huấn luyện và đánh giá mô hình **phát hiện người và phương tiện giao thông 2D (Bounding Box)** trên ảnh đường phố, phục vụ camera giám sát giao thông và hệ thống hỗ trợ lái xe (ADAS / xe tự hành).

**Ba nhóm lỗi trọng tâm cần kiểm soát:**
1. **Bỏ sót đối tượng thật (`bo_sot` — False Negative):** Bỏ sót người đi bộ sang đường (`img_014`, `img_034`), người đi xe máy, trẻ em ngồi trên xe (`img_010`) hoặc người bị che khuất nặng bên cạnh xe tải (`img_007`, `img_036`) dẫn tới nguy cơ va chạm trực tiếp.
2. **Gán thừa vật không thật (`thua` — False Positive):** Gán nhầm tượng điêu khắc (`img_011`, `img_017`, `img_027`, `img_028`), ma-nơ-canh (`img_033`), đồ chơi (`img_038`), xác vỏ xe trưng bày (`img_024`), hình in trên biển quảng cáo/tranh tường (`img_018`, `img_026`, `img_030`, `img_031`, `img_040`) hoặc ảnh phản chiếu qua gương/kính/mặt đường ướt (`img_013`, `img_015`, `img_020`, `img_033`) khiến hệ thống phanh gấp hoặc báo động giả.
3. **Bất đồng lớp và kích thước box (`sai_lop`, `sai_vi_tri_kich_thuoc`):** Gộp người với xe máy thành 1 box, kéo box trùm theo hàng hóa quá khổ nhô ra ngoài (`img_029`, `img_035`), hoặc phân loại lệch giữa `car` / `van` / `truck` / `bus` / `tuk_tuk` làm sai lệch kích thước vật lý và tụt chỉ số đồng thuận IAA.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp đối tượng (`submission/taxonomy.json` — Phiên bản `v1.0`)

Áp dụng phép thử *"Mô hình phát hiện vật 2D có phải đoán ra thông tin này khi suy luận không?"*, cả nhóm thống nhất mở rộng từ 6 lớp của `v0` lên **8 lớp** (bổ sung `bus` và `tuk_tuk`):

| Lớp | Định nghĩa (một câu) | Ví dụ trong 40 ảnh | Phản ví dụ / dễ nhầm với (không gán vào lớp này) |
|---|---|---|---|
| `car` | Ô tô con chở người 4–7 chỗ (sedan, hatchback, SUV, station wagon, taxi) có khoang lái và khoang hành khách liền khối. | Ô tô trên sàn xe chở (`img_002`), Mazda CX-5 (`img_012`), Honda Civic (`img_023`), AMC Pacer đen dưới sàn (`img_024`), SUV & taxi (`img_031`, `img_040`). | Mô hình ô tô đồ chơi (`img_038`), vỏ xe vàng không bánh đặt trên giá nóc (`img_024`), xe đạo cụ hoạt hình Dangermouse (`img_019`), xe bán tải có thùng hàng rời (`img_001`, `img_012` → `truck`). |
| `truck` | Xe tải chở hàng, xe đầu kéo rơ-moóc và xe bán tải (pickup, kể cả bán tải lắp nắp chụp thùng hàng) có thùng hàng tách biệt với cabin lái. | Bán tải trắng/đen (`img_001`, `img_012`, `img_028`), đầu kéo chở ô tô (`img_002`), tải Isuzu/CMC (`img_007`, `img_036`), Hilux thùng kín (`img_007`), bán tải gắn nắp chụp `Master 75` (`img_016`, `img_037`). | Tượng đài xe bán tải bằng đá (`img_027`), xe van thân hộp liền khối (`img_005`, `img_016` → `van`), xe đồ chơi (`img_038`). |
| `van` | Xe thùng kín cỡ nhỏ thân liền khối (minivan, panel van, xe khách nhỏ 7–16 chỗ có khoang lái thông với khoang sau). | Minivan bạc `51LD-70-65` (`img_005`), xe van trắng (`img_016`), xe van nhỏ ở xa (`img_002`), xe thùng kín cảnh sát (`img_011`). | Xe buýt cỡ trung/lớn từ 16–20 chỗ trở lên (`img_009`, `img_037` → `bus`), xe bán tải lắp nắp thùng cao (`img_007`, `img_037` → `truck`). |
| `bus` | Xe buýt hoặc xe khách cỡ trung và lớn chở nhiều hành khách (thân dài, từ 16–20 chỗ trở lên). | Xe khách ở Huế (`img_009`), xe buýt ban đêm (`img_031`), xe buýt GO Transit xanh lá (`img_037`), xe buýt ở Bangkok (`img_040`). | Xe minivan chở khách cỡ nhỏ (`img_005`, `img_016` → `van`). |
| `tuk_tuk` | Xe cơ giới 3 bánh có mui che dùng chở khách hoặc chở hàng (tuk-tuk Thái Lan, auto-rickshaw Ấn Độ). | Hai xe tuk-tuk xanh đỗ vỉa hè (`img_004`), tuk-tuk ở Bangkok (`img_016`, `img_040`), auto-rickshaw vàng chở bao tải (`img_035`). | Xích lô 3 bánh đạp bằng sức người (`img_006`, `img_023`, `img_034` → không gán xe), mô tô 2 bánh gắn thuyền sidecar (`img_022` → `motorcycle`). |
| `motorcycle` | Xe cơ giới 2 bánh (xe máy số, xe tay ga, mô tô) và mô tô gắn thuyền bên hông (sidecar). | Xe máy đang chạy hoặc đỗ (`img_005`, `img_006`, `img_008`, `img_010`, `img_026`), dàn mô tô độ (`img_011`), mô tô sidecar (`img_022`). | Xe đạp không động cơ (`img_003`, `img_006`, `img_029`), xích lô (`img_023`), xe 3 bánh tuk-tuk (`img_004`, `img_035` → `tuk_tuk`). |
| `person` | Con người thật ở mọi tư thế (đi bộ, đứng, ngồi, đang điều khiển hoặc ngồi trên xe 2–3 bánh hoặc khoang xe mở). | Khách ngồi quán (`img_001`), người gánh hàng (`img_003`, `img_010`), người đi xe máy (`img_005`, `img_026`), người sang đường (`img_014`, `img_034`), tài xế trong tuk-tuk (`img_004`, `img_035`). | Người ngồi trong cabin kín sau cửa kính của `car`/`van`/`truck`/`bus` (`img_005`, `img_009`, `img_040`); tượng (`img_011`, `img_017`); ma-nơ-canh (`img_033`); ảnh trên biển quảng cáo (`img_018`, `img_030`); búp bê/chuột hoạt hình (`img_019`, `img_038`); mũ bảo hiểm treo trên gương (`img_008`). |
| `ignore_region` | Vùng bỏ qua (không phải vật): dùng bao phủ đám đông/bãi xe chồng lấp dày đặc không tách được từng cá thể, hoặc cụm vật ở xa có chiều cao `< 20 px`. | Đám đông lễ hội/khán giả/nhà chờ buýt (`img_011`, `img_019`, `img_032`), dòng xe máy kẹt cứng/bãi xe phía xa (`img_003`, `img_021`, `img_030`), cụm người/xe `< 20 px` ở xa (`img_013`, `img_025`, `img_031`, `img_039`). | Không dùng `ignore_region` cho vật đơn lẻ nhìn rõ `>= 20 px`, và **không dùng `ignore_region` đè lên tượng, ma-nơ-canh, đồ chơi hay gương phản chiếu**. |

**Quy tắc xử lý vật ngoài Taxonomy (Out-of-taxonomy):**
- **Phương tiện thô sơ / ngoài phạm vi:** Xe đạp (`img_003`, `img_006`, `img_029`, `img_034`, `img_039`), xích lô đạp chân (`img_006`, `img_009`, `img_023`, `img_034`, `img_039`), xe đẩy/xe thồ hàng rong và quang gánh (`img_003`, `img_010`, `img_025`), thuyền trên biển (`img_025`) → **Không vẽ box phương tiện** (và không bọc `ignore_region`), nhưng người thật đang điều khiển hoặc ngồi trên xe đạp/xích lô vẫn gán lớp `person` bình thường.
- **Động vật thật hoặc mô hình động vật:** Chim hải âu (`img_017`), tượng ngựa đồng (`img_017`), mô hình ngựa trắng trên thùng bán tải (`img_028`) → **Không gán**.

---

### 2.2. Thuộc tính (Attributes) và ngưỡng đo thống nhất

Mọi đối tượng thuộc 7 lớp vật thể (trừ `ignore_region`) đều phải gán đủ 3 thuộc tính trong bảng **Objects** của CVAT:

| Thuộc tính | Kiểu & Giá trị | Ý nghĩa & Ngưỡng định lượng đã khóa |
|---|---|---|
| `occluded_level` | `select`: `0`, `1`, `2`, `3`<br/>*(Mặc định: `0`)* | Tính theo **% diện tích thực của vật bị vật thể khác ở phía trước che khuất** (không tính phần bị cắt ở mép ảnh và không gộp độ mờ nhòe):<br/>• **`0` (Thấy rõ):** Bị che `0%` (nhìn thấy trọn vẹn phần thân vật trong khung ảnh).<br/>• **`1` (Che ít):** Bị che từ **`1% – 30%`** diện tích (dưới 1/3 vật, VD: xe máy bị chân người lái che nhẹ, ô tô tầng trên bị thanh chắn che nhẹ ở `img_002`, ô tô sau hàng rào thưa ở `img_018`).<br/>• **`2` (Che nhiều):** Bị che từ **`31% – 70%`** diện tích (từ 1/3 đến 2/3 vật, VD: khách ngồi bàn ăn bị che nửa người ở `img_001`, ô tô tầng dưới ở `img_002`, tài xế ngồi trong tuk-tuk `img_004`, `img_035`).<br/>• **`3` (Che rất nặng):** Bị che **`> 70%` đến `90%`** diện tích nhưng phần lộ ra vẫn đủ bộ phận đặc trưng để khẳng định chắc chắn lớp. Nếu bị che `> 90%` hoặc chỉ lộ chi tiết vụn không rõ thân/khung chính (như chỏm tóc góc dưới trái `img_016` hoặc 2 gương xe máy ở sát mép dưới `img_040`) thì **không vẽ box**.<br/>*(Lưu ý kỹ thuật: Chọn trong bảng Objects của CVAT, không dùng phím `Q` vì cờ `occluded` mặc định của CVAT bị script bỏ qua).* |
| `truncated` | `select`: `no`, `yes`<br/>*(Mặc định: `no`)* | Đánh dấu vật bị cắt ngang bởi 1 trong 4 cạnh biên của bức ảnh:<br/>• **`no`:** Toàn bộ phần nhìn thấy của vật nằm gọn trong khung ảnh (cách mép ảnh `>= 2 px`).<br/>• **`yes`:** Cơ thể người hoặc thân xe chạm sát biên ảnh và bị khung hình cắt mất một phần (`>= 5%` thể tích vật nằm ngoài biên ảnh, VD: đuôi bán tải trắng ở mép phải `img_001`, người đi xe máy ở mép dưới `img_005`, `img_026`, `img_040`, hai ô tô ở hai mép `img_015`).<br/>*(Lưu ý: Giá trị bắt buộc là `no` / `yes` theo schema, không phải `true` / `false`).* |
| `can_review` | `checkbox`: `true` / `false`<br/>*(Mặc định: `false`)* | Tick `true` khi gặp ca biên còn phân vân về lớp, ranh giới box hoặc ảnh bị nhòe/tối nặng (`img_014`, `img_015`, `img_019`, `img_037`) để đưa vào cuộc họp Calibration ở Pha 5. Nếu có vấn đề cấp ảnh chưa vẽ được box thì ghi vào `submission/qc/image_issues.csv`. |

---

## 3. Quy tắc vẽ box đã khóa (Mã `R-01` – `R-09`)

1. **`R-01` — Vẽ box ôm sát phần nhìn thấy thực tế (Visible Box) và không vượt biên ảnh:**
   - Bốn cạnh box phải bám sát các điểm cực trị của phần thực thể **nhìn thấy được** (gồm cả mũ bảo hiểm, nón lá, ba lô đang đeo, tay chân của `person`; gương chiếu hậu, bánh xe, cản trước/sau của phương tiện).
   - Không vẽ suy đoán phần bị che khuất hoàn toàn và không bao bóng đổ hay vệt phản chiếu dưới mặt đường ướt (`img_013`, `img_015`, `img_031`).
   - Khi một vật bị thanh hàng rào (`img_018`), cột kính nhà chờ (`img_032`) hoặc khung sắt rơ-moóc (`img_002`) chắn ngang chia thành nhiều phần rời nhau, vẫn vẽ **1 box duy nhất** bao trọn các phần nhìn thấy của vật đó và đặt `occluded_level` tương ứng.
   - Không kéo tọa độ box vượt ra ngoài biên ảnh (đặc biệt chú ý mép dưới `img_026.jpg` có kích thước `784x1024`).

2. **`R-02` — Loại bỏ đối tượng giả, tượng, ma-nơ-canh, đồ chơi, hình in, đạo cụ và vỏ xe trưng bày:**
   - Chỉ gán nhãn cho người sống thật và phương tiện cơ giới thật.
   - **Không vẽ bất kỳ box nào (kể cả `ignore_region`)** cho: tượng Predator (`img_011`), tượng đồng cưỡi ngựa (`img_017`), tượng đài xe bán tải (`img_027`), tượng ngựa trắng trên thùng xe (`img_028`), ma-nơ-canh (`img_033`), mô hình đồ chơi (`img_038`), ảnh người trên biển quảng cáo/áp phích/kính xe (`img_015`, `img_018`, `img_021`, `img_030`, `img_031`, `img_032`, `img_040`), hình nộm chuột và vỏ xe hóa trang hoạt hình Dangermouse (`img_019`), mũ bảo hiểm rỗng treo trên gương (`img_008`), và khung vỏ xe màu vàng không bánh đặt trên giá nóc ô tô khác (`img_024`).

3. **`R-03` — Không gán ảnh phản chiếu và bộ phận của xe chủ (Ego-vehicle):**
   - Không gán người hoặc xe chỉ xuất hiện qua gương chiếu hậu (`img_020`, `img_036`), qua kính cửa hàng/kính mũ bảo hiểm (`img_033`, `img_040`) hoặc phản chiếu mặt nước (`img_013`, `img_015`, `img_031`). Ngược lại, vật thật nằm phía sau lớp kính trong suốt (`img_018`, `img_032`) vẫn gán bình thường.
   - Không gán gương chiếu hậu, khung cửa hay thân vỏ của chính chiếc xe gắn camera (`img_020`).
   - **Năm ảnh đối chứng không có đối tượng thật (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`)** bắt buộc để trống **`0` annotation** (không vẽ `ignore_region`) nhưng vẫn giữ nguyên tên ảnh trong file COCO.

4. **`R-04` — Tách riêng box người và phương tiện; quy tắc người ngồi trong xe:**
   - **Luôn tách riêng box người và box xe:** Mỗi người ngồi trên xe máy (cả người lái và từng người/trẻ em ngồi sau ở `img_005`, `img_006`, `img_010`, `img_026`) được gán **1 box `person` riêng**. Chiếc xe máy gán **1 box `motorcycle` riêng** chỉ ôm kết cấu xe (từ bánh xe lên tới tay lái/gương chiếu hậu), không kéo trùm lên đầu người lái.
   - Với xe mô tô gắn thuyền bên hông (`sidecar` ở `img_022`) hoặc gắn sạp hàng cố định (`img_025`): vẽ **1 box `motorcycle`** bao cả thân xe và thùng/sạp gắn liền, tách riêng các box `person`.
   - **Người bên trong phương tiện:** Gán `person` cho tài xế/hành khách ngồi trong khoang xe mở nhìn rõ phần thân trên (`tuk_tuk` ở `img_004`, `img_016`, `img_035`; người lái thật trong xe mui trần `img_019`). **Không gán `person`** cho bóng người mờ ngồi trong cabin kín sau cửa kính của `car`, `van`, `truck`, `bus` (`img_005`, `img_009`, `img_040`).

5. **`R-05` — Quy tắc gán Xe chở xe (Car Carrier — `img_002.jpg`):**
   - Vẽ **1 box `truck`** ôm toàn bộ xe đầu kéo và khung rơ-moóc hai tầng, **đồng thời vẽ riêng 8 box `car`** cho cả 8 chiếc ô tô nằm trên 2 sàn chở (`occluded_level = 1` cho các xe tầng trên và `occluded_level = 2` cho các xe tầng dưới bị khung thép rơ-moóc che).

6. **`R-06` — Ngưỡng chiều cao tối thiểu `20 px` và ranh giới vẽ `ignore_region` cho đám đông:**
   - Đối tượng có chiều cao phần nhìn thấy **`< 20 px`** (`min_box_height_px = 20`) thì không vẽ box đối tượng cá nhân.
   - Dùng `ignore_region` cho: (a) cụm từ 3 vật trở lên ở xa có chiều cao `< 20 px` (`img_003`, `img_006`, `img_013`, `img_025`, `img_031`, `img_034`, `img_039`), hoặc (b) đám đông người đứng chen chúc / dòng xe máy kẹt cứng hoặc bãi xe xếp chồng lấp dày đặc không thể tách rời từng chiếc (`img_008`, `img_011`, `img_019`, `img_021`, `img_030`, `img_032`).
   - **Mốc phân ranh giới (tránh lỗi `bat_dong_ignore`):** Trên ảnh kẹt xe/bãi xe/nhà chờ (`img_021`, `img_030`, `img_032`), chỉ vẽ box lẻ (`person`, `motorcycle`, `car`) cho **hàng người/xe đầu tiên ở tiền cảnh nhìn thấy rõ cả thân xe/bánh xe** (hoặc người đứng tách biệt hẳn ở rìa phải `img_032`) và các ô tô lớn nhìn rõ khối. Từ lớp xe thứ 2 trở về phía xa (nơi chỉ lộ đầu/mũ bảo hiểm hoặc bị che khuất `> 70%`), gom chung vào **1 box `ignore_region` lớn nằm ngang**, cạnh dưới dừng ngay phía trên hàng xe tiền cảnh, tuyệt đối không vẽ box lẻ nằm đè bên trong vùng `ignore_region`.

7. **`R-07` — Ranh giới phương tiện khi chở hàng hóa cồng kềnh / quá khổ:**
   - Hàng hóa nhỏ gọn nằm trong lòng thùng bán tải (`img_012`) hoặc cuộn dây gọn trên baga (`img_016`) nằm trong phạm vi tự nhiên của box xe.
   - Hàng hóa hoặc vật trang trí chất quá khổ nhô cao/vươn dài bất thường ra ngoài khung thiết kế của xe/người (như khối bao tải chất cao gấp đôi thân xe `tuk_tuk` ở `img_035`, thanh nhôm dài 7m vắt ngang ở `img_029`, quang gánh treo dưới chân người ở `img_003`, tượng ngựa trên thùng xe ở `img_028`): **box chỉ ôm sát khung vật lý thật của chiếc xe (`tuk_tuk`, `motorcycle`, `truck`) hoặc cơ thể `person`**, không kéo rộng box bao lấy phần hàng hóa quá khổ đó.

8. **`R-08` — Xử lý phương tiện thô sơ ngoài Taxonomy (Xe đạp, Xích lô, Xe hàng rong):**
   - Không vẽ box phương tiện cho xe đạp, xích lô, xe đẩy/xe thồ hàng rong hay quang gánh (`img_003`, `img_006`, `img_009`, `img_010`, `img_023`, `img_029`, `img_034`, `img_039`).
   - Người thật đang điều khiển hoặc ngồi làm hành khách trên xe đạp/xích lô vẫn vẽ box `person` đầy đủ.

9. **`R-09` — Phân biệt `truck` vs `van` trên xe bán tải độ thùng & Xử lý nhòe chuyển động (Motion Blur):**
   - Mọi xe bán tải có khoang cabin tách biệt với thùng hàng phía sau — kể cả khi lắp thêm khung thùng kín/thùng lạnh (`img_007`) hoặc nắp chụp thùng cao (`Master 75 Truck Cap` ở `img_016`, `img_037`) và xe tải nhỏ mui bạt (`img_036`) — đều gán lớp **`truck`**. Lớp `van` chỉ dùng cho xe thân hộp liền khối từ đầu tới đuôi (`img_005`, `img_011`, `img_016`).
   - Với đối tượng bị nhòe chuyển động do phơi sáng chậm (`img_014`, `img_015`) hoặc nằm trong vùng tối (`img_039`): nếu vẫn nhận diện được lớp và lõi thân cao `>= 20 px` thì vẽ box ôm phần thân chính (không kéo theo vệt bóng mờ loãng) và tick `can_review = true`. Nếu vệt nhòe làm mất hoàn toàn hình dạng thực thể thì dùng `ignore_region`.

---

## 4. Giải quyết các bất đồng từ Pha 1 và Bảng Ca khó đã chốt

### 4.1. Biên bản giải quyết 5 điểm bất đồng chính giữa các bản cá nhân (`101` – `105`)

| # | Điểm bất đồng ở Pha 1 | Ý kiến các thành viên | Quyết định thống nhất ở `v1.0` & Lý do |
|---|---|---|---|
| 1 | **Xử lý xe buýt (`bus`) và xe 3 bánh (`tuk_tuk`)** | • `101`, `104`: Đề xuất thêm 2 lớp `bus` và `tuk_tuk`.<br/>• `102`: Giữ 6 lớp cũ, bọc `ignore_region` lên `tuk_tuk` và `bus`.<br/>• `103`: Tạm gộp `tuk_tuk` vào `motorcycle` và gộp `bus` vào `van`.<br/>• `105`: Tạm không gán ở v0.1, đề xuất họp Pha 2 thêm lớp. | **Thêm `bus` và `tuk_tuk` vào `taxonomy.json` v1.0** (tổng 8 lớp). Lý do: Đây là các phương tiện cơ giới thật xuất hiện nhiều trong bộ ảnh (`img_004`, `img_009`, `img_016`, `img_031`, `img_035`, `img_037`, `img_040`); ép vào `motorcycle`/`van` hay `ignore_region` đều làm sai ngữ nghĩa của mô hình. |
| 2 | **Xe chở xe (`img_002.jpg`)** | • `103`: Chỉ vẽ 1 box `truck` duy nhất trùm cả xe tải và 8 ô tô trên thùng (coi xe trên sàn là hàng hóa).<br/>• `101`, `102`, `104`, `105`: Vẽ 1 box `truck` + vẽ riêng từng box cho 8 xe trên sàn (riêng `105` phân vân 1 xe tầng dưới là `truck` hay `car`). | **Chốt theo `R-05`:** Vẽ **1 box `truck`** ôm đầu kéo + khung rơ-moóc, và vẽ riêng **8 box `car`** cho cả 8 ô tô trên 2 tầng sàn (`occluded_level = 1` ở tầng trên, `2` ở tầng dưới). Lý do: Các xe trên sàn đều mang đầy đủ đặc trưng thị giác của ô tô thật trên đường. |
| 3 | **Ngưỡng `%` của `occluded_level` và giá trị `truncated`** | • `101`, `103`, `104`: `0` (`0%`), `1` (`1–30%`), `2` (`31–70%`), `3` (`>70%`).<br/>• `102`: `0` (`<5%`), `1` (`5–35%`), `2` (`36–80%`), `3` (`>80%`, gộp cả mờ nhòe).<br/>• `105`: `0` (`0–10%`), `1` (`10–50%`), `2` (`50–85%`), `3` (`>85%`).<br/>• `103` ghi `truncated` là `true`/`false`. | **Khóa mốc 1/3 diện tích:** `0` (`0%`), `1` (`1–30%`), `2` (`31–70%`), `3` (`71–90%`, che `> 90%` hoặc chỉ lộ mẩu vụn thì không gán); không trộn độ nhòe vào `occluded_level`; giữ giá trị `truncated` là `no`/`yes` đúng JSON Schema. |
| 4 | **Năm ảnh đối chứng không có vật thật (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`)** | • `102`: Đề xuất vẽ `ignore_region` lên mặt gương (`img_020`) và tủ kính đồ chơi (`img_038`).<br/>• `101`, `103`, `104`, `105`: Không vẽ bất kỳ box nào, để trống `0` annotation. | **Chốt theo `R-02`, `R-03`:** Để trống **`0` annotation** trên cả 5 ảnh (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`), không vẽ `ignore_region` lên vật giả/gương để tránh lệch `bat_dong_ignore`. |
| 5 | **Người ngồi trong xe kín vs khoang mở & Xe hoạt hình `img_019`** | • `104`, `105`: Đề xuất không gán người ngồi sau kính kín của `car`/`van`/`truck`/`bus`, chỉ gán người trong khoang mở (`tuk_tuk`, xe mui trần).<br/>• `img_019`: `101` đề xuất bỏ qua vỏ xe hoạt hình Dangermouse, `105` tạm gán `car` (`can_review`). | **Chốt theo `R-02`, `R-04`:** Không gán người ngồi sau kính kín (`img_005`, `img_009`, `img_040`), chỉ gán người trên xe 2–3 bánh và khoang mở (`img_004`, `img_019`, `img_035`). Với `img_019`, không gán chuột hoạt hình, chỉ gán `person` cho người lái thật (nếu thành viên nào gán vỏ xe vàng thì bắt buộc tick `can_review` để chốt ở Pha 5). |

---

### 4.2. Bảng tra cứu nhanh các Ca khó đã khóa cho Pha 3

| Ảnh | Tình huống | Rule ID | Quyết định khóa ở `v1.0` | Lý do |
|---|---|---|---|---|
| `img_001` | Khách ngồi quán bị che chân/thân dưới; xe bán tải trắng cắt mép phải. | `R-01`, `R-09` | Gán `person` (`occluded_level = 1, 2`) cho khách nhìn rõ đầu-vai; gán `truck` (`truncated = yes`) cho bán tải trắng. | Bán tải có thùng hàng tách rời cabin thuộc `truck`; đuôi xe chạm biên phải nên `truncated = yes`. |
| `img_002` | Xe đầu kéo Renault chở 8 ô tô con trên 2 tầng sàn; góc phải có 1 người đứng và 1 xe van nhỏ. | `R-05` | Vẽ 1 `truck` (đầu kéo + rơ-moóc) + 8 `car` riêng cho 8 xe trên sàn (`occluded_level = 1` tầng trên, `2` tầng dưới) + 1 `person` + 1 `van`. | Thống nhất gán đủ cả xe chở lẫn 8 xe con trên sàn để đồng nhất số lượng box giữa các thành viên. |
| `img_003`, `img_006`, `img_010`, `img_023` | Xe đạp, xích lô, xe thồ hàng rong và quang gánh đi cùng người/xe cơ giới. | `R-04`, `R-08` | **Không vẽ box xe** cho xe đạp, xích lô, xe hàng rong, quang gánh; vẫn gán đầy đủ `person` cho người dắt/đạp/ngồi trên đó; tách riêng `person` và `motorcycle` cho xe máy. | Phương tiện không động cơ nằm ngoài taxonomy nhưng con người thật trên xe vẫn là chủ thể giao thông. |
| `img_004`, `img_016`, `img_035`, `img_040` | Xe 3 bánh tuk-tuk Thái Lan và auto-rickshaw Ấn Độ. | `R-04`, `R-07` | Gán lớp **`tuk_tuk`** cho thân xe; gán riêng `person` (`occluded_level = 1, 2`) cho tài xế/khách nhìn rõ trong khoang. Ở `img_035`, box `tuk_tuk` **chỉ ôm khung cabin và bánh xe**, không bao khối bao tải trên nóc. | Đã bổ sung lớp `tuk_tuk` vào `taxonomy.json` v1.0 và khóa `R-07` không bao hàng hóa quá khổ. |
| `img_005` | Người đi xe máy cắt mép dưới; phía sau là xe minivan bạc `51LD-70-65` có bóng người mờ sau kính. | `R-01`, `R-04` | Tách riêng từng `person` (`truncated = yes` ở mép dưới) và `motorcycle`; gán `van` (`occluded_level = 2`) cho xe bạc; không gán bóng người mờ sau kính xe van. | Minivan thân liền khối thuộc `van`; không đoán bóng người sau cửa kính kín (`R-04`). |
| `img_007`, `img_036`, `img_037` | Bán tải Hilux lắp thùng kín (`img_007`), tải nhỏ Veryca mui bạt (`img_036`), bán tải Silverado lắp nắp chụp `Master 75` đứng cạnh xe buýt xanh (`img_037`). | `R-09` | Cả 3 xe bán tải/tải nhỏ đều gán **`truck`**; xe buýt xanh ở `img_037` gán **`bus`**; chú ý gán cả người bị che nặng (`occluded_level = 3`) sau bánh xe `img_007` và sau mũi xe `img_036` (hoặc tick `can_review`). | Cabin tách biệt với thùng hàng phía sau nên thuộc `truck`, phân biệt rõ với thân liền khối của `van`. |
| `img_008`, `img_021`, `img_030`, `img_032` | Đám đông xe máy kẹt cứng (`img_021`), bãi giữ xe máy (`img_030`), hàng người chờ xe buýt (`img_032`), phố đêm (`img_008`). | `R-02`, `R-06` | Gán lẻ (`person`, `motorcycle`, `car`) cho hàng đầu ở tiền cảnh nhìn rõ thân xe và các cá nhân tách biệt (như bảo vệ `img_030`, 2 người bên phải `img_032`); gom toàn bộ phần đám đông/bãi xe chồng lấp phía sau vào **1 box `ignore_region` ngang**; không gán mũ bảo hiểm treo gương (`img_008`) và ảnh trên biển quảng cáo (`img_021`, `img_030`, `img_032`). | Giữ ranh giới `ignore_region` đồng nhất giữa các thành viên để tránh lỗi `bat_dong_ignore`. |
| `img_009`, `img_031` | Xe khách 29 chỗ ở Huế (`img_009`) và xe buýt ban đêm (`img_031`). | `R-01`, `R-03` | Gán lớp **`bus`** cho xe khách/xe buýt; không gán tranh tường hoạt hình và bóng phản chiếu đèn dưới mặt đường ướt ở `img_031`. | Đã có lớp `bus` chính thức trong `taxonomy.json` v1.0. |
| `img_011`, `img_018`, `img_019` | Tượng Predator & mô tô độ (`img_011`), biển quảng cáo chính trị gia (`img_018`), xe hoạt hình Dangermouse (`img_019`). | `R-02`, `R-06` | Không gán tượng Predator (`img_011`), ảnh trên biển quảng cáo (`img_018`) và chuột hoạt hình (`img_019`); gán `car` sau hàng rào ở `img_018` (`1` box/xe, `occluded_level = 1, 2`); bọc `ignore_region` cho đám đông ở xa (`img_011`, `img_019`). | Loại bỏ triệt để các đối tượng giả/hình in theo `R-02`. |
| `img_014`, `img_015` | Người đi bộ và xe đỗ rõ nét, nhưng xe máy/ô tô đang chạy bị nhòe chuyển động (motion blur). | `R-03`, `R-09` | Gán bình thường các người/xe rõ nét (không kéo box xuống bóng nước ở `img_015`); với xe bị nhòe còn nhận ra hình khối thì vẽ ôm lõi thân chính kèm `can_review = true` (hoặc bọc `ignore_region` cho vệt nhòe mất hẳn biên ở `img_014`). | Đánh dấu `can_review` trên các box nhòe để nhóm đối chiếu và chốt ở Pha 5. |
| `img_017`, `img_020`, `img_027`, `img_033`, `img_038` | Tượng cưỡi ngựa (`img_017`), gương xe chủ (`img_020`), tượng bán tải (`img_027`), ma-nơ-canh (`img_033`), tủ kính đồ chơi (`img_038`). | `R-02`, `R-03` | **Để trống `0` annotation** trên cả 5 ảnh (không vẽ `ignore_region`), giữ nguyên ảnh trong danh sách `"images"` của COCO. | 5 ảnh đối chứng kiểm tra lỗi False Positive và lỗi "hai người cùng sai". |
| `img_022`, `img_025`, `img_026` | Mô tô sidecar (`img_022`), xe máy gắn sạp hàng & đám đông bãi biển (`img_025`), đôi nam nữ đi xe máy trong hầm (`img_026`). | `R-01`, `R-04`, `R-06` | `img_022` và `img_025`: 1 box `motorcycle` ôm cả xe + thuyền/sạp gắn liền, tách riêng `person`; `img_026`: tách 2 `person` + 1 `motorcycle` (`truncated = yes`, dừng đúng `y <= 1024`), bỏ qua tranh vẽ tàu biển trên tường. | Đảm bảo nhất quán cách tách người – xe và không kéo vượt biên ảnh `784x1024`. |
| `img_024`, `img_028`, `img_029` | Vỏ xe vàng đặt trên nóc ô tô đen (`img_024`), tượng ngựa trên thùng bán tải (`img_028`), thanh nhôm 7m vác ngang trên xe máy & xe đạp (`img_029`). | `R-02`, `R-07`, `R-08` | `img_024`: gán `car` cho xe đen dưới sàn, bỏ qua vỏ xe vàng trên nóc; `img_028`: gán `truck` cho bán tải (không bao tượng ngựa); `img_029`: gán `person` + `motorcycle` cho xe máy (không kéo box theo thanh nhôm 7m) và chỉ gán `person` cho người đi xe đạp. | Khóa chặt quy tắc `R-02` (bỏ vỏ xe mô hình/tượng ngựa) và `R-07` (không bao vật mang vác quá khổ). |

---

## 5. QC và phiên bản

### 5.1. Ngưỡng QC bắt buộc (Giữ nguyên trong `submission/taxonomy.json -> rules`)
- **Chiều cao tối thiểu của box (`min_box_height_px`):** `20 px`.
- **Ngưỡng ghép cặp box giữa các thành viên (`match_iou`):** `IoU >= 0.50`.
- **Ngưỡng IoU đạt chuẩn của từng cặp box:**
  - Box cao `>= 50 px` (`large_px = 50`): **`IoU >= 0.85`** (`iou_thr_large`).
  - Box cao từ `20 px` đến `< 50 px`: **`IoU >= 0.70`** (`iou_thr_small`).
- **Ngưỡng đồng thuận toàn lô (IAA) ở Pha 4:**
  - Độ phủ lượt gán: **`100%`** (`120/120` lượt gán với nhóm 5 người, `K = 3` người/ảnh, mỗi người `24` ảnh).
  - IoU trung bình trên các cặp box ghép được: **`>= 0.85`** (`iaa_iou`).
  - Tỷ lệ đồng thuận lớp trên các cặp box ghép được: **`>= 95%`** (`iaa_class`).
- **Ngưỡng nghiệm thu vòng duyệt QC cuối (`error_rate_img_max`):**
  - Duyệt toàn bộ lô `40/40` ảnh; tỷ lệ ảnh có lỗi **`<= 2%`** (`0/40` ảnh lỗi) và không có lỗi lặp hệ thống mới được chốt **Release**.

### 5.2. Quy trình làm việc độc lập ở Pha 3 và Calibration ở Pha 5
- Trong suốt Pha 3, mỗi thành viên chỉ gán trên đúng 24 ảnh được chia trong `submission/qc/assignments/<mã>.txt` theo bản `v1.0` đã khóa này; không trao đổi hay xem nhãn của nhau.
- Các ca còn phân vân khi vẽ thì bật `can_review = true` trên box (hoặc ghi vào `submission/qc/image_issues.csv` nếu chưa vẽ được box).
- Sang Pha 5, mọi ca bất đồng trong `calibration_candidates.csv` sẽ được nhóm họp giải quyết từng dòng vào `calibration_decisions.csv` (9 cột) và nâng cấp guideline lên `v1.1`.

### 5.3. Lịch sử phiên bản Guideline

| Phiên bản | Thay đổi chính | Lý do (dẫn chiếu ca) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên (6 lớp sơ lược, thuộc tính định tính, chưa có quy tắc ca khó). | Bản gốc dùng để khảo sát điểm mơ hồ ở Pha 1. |
| `v0.1` (`101`–`105`) | 5 bản khảo sát độc lập của từng thành viên trên 40 ảnh. | Phát hiện các điểm mơ hồ tự nhiên và sự khác biệt góc nhìn giữa 5 người gán. |
| `v1.0` | • Mở rộng `taxonomy.json` lên 8 lớp (bổ sung `bus`, `tuk_tuk`).<br/>• Giải quyết 5 điểm bất đồng lớn giữa các thành viên (`img_002`, `img_004`/`009`/`035`/`037`, ngưỡng `%` `occluded_level`, 5 ảnh 0 box `img_017`/`020`/`027`/`033`/`038`, ranh giới `ignore_region` ở `img_021`/`030`/`032`).<br/>• Khóa 9 quy tắc `R-01` – `R-09` và tạo `cvat/labels_generated.json` cho Pha 3. | Đảm bảo cả 5 thành viên hiểu và áp dụng duy nhất một bộ quy tắc trước khi bước vào lượt gán mù ở Pha 3. |
