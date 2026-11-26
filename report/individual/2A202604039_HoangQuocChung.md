# Báo cáo Day 13 — Hoàng Quốc Chung, 2A202604039

## 1. Kết luận trước

- **Thông tin chung:** MSSV `2A202604039`, task cá nhân `D13-2A202604039`, bộ 10 frame: `s01f09`, `s01f24`, `s02f01`, `s02f24`, `s03f16`, `s03f34`, `s04f05`, `s04f20`, `s05f21`, `s05f37`. Môi trường làm việc: Ubuntu WSL2 trên Windows, trình duyệt Edge, CVAT v2.81.0.
- **Tổng quan kết quả:** Đã vẽ tổng cộng **53 cuboid**, trong đó frame `s01f09` có nhiều nhất (**8 cuboid**) và 9 frame còn lại mỗi frame có **5 cuboid** (ít nhất là 5 cuboid/frame do tập trung ưu tiên gán các phương tiện và người đi bộ rõ nét trong vùng gần, tránh vẽ đoán các cụm điểm quá thưa ở xa). Công cụ `selfcheck` gắn cờ cảnh báo **2/53 cuboid** (`s01f09 #7` và `s05f37 #2`) đều thuộc nhóm vượt nhẹ ngưỡng chiều cao/chiều rộng tham chiếu của `Car`, không còn lỗi lơ lửng hay chìm đáy.
- **Đánh giá độ tin cậy:** Phần tự tin nhất là **hướng đầu xe (`yaw_deg`)** và **độ cao đáy chạm đường (`z_bottom`)** ở các đối tượng trong phạm vi 30 m; phần còn yếu nhất là **ước lượng kích thước (`width`, `height`) cho các vật thể ở xa trên 25 m** khi cụm điểm LiDAR bị thưa và lẫn điểm phản xạ xung quanh.
- **Cách thao tác CVAT:**
  - *Đổi góc nhìn:* Dùng chuột trái kéo để xoay tự do quanh tâm, chuột phải kéo (hoặc phím `J/L`, mũi tên) để dịch chuyển tâm nhìn tới vật thể ở xa, lăn con lăn chuột để phóng to/thu nhỏ và nháy đúp khoảng trống để đưa góc nhìn về mặc định.
  - *Vẽ và chỉnh cuboid:* Chọn công cụ vẽ cuboid chế độ **Shape**, nháy đúp lên cụm điểm vật thể để tạo hộp, sau đó điều chỉnh sát 6 mặt bằng các tay nắm ở ba khung chiếu phụ **Top**, **Side** và **Front** (tuyệt đối không xoay hộp ở Side và Front).
  - *Bật hướng và đối chiếu camera:* Tích chọn **Cuboid orientation** trong bảng *Appearance* để hiện mũi tên đỏ (`+X`) chỉ về đầu xe; bấm **Save** (`Ctrl + S`) và chạy `python scripts/lab.py preview` để xuất ảnh chiếu đè cuboid lên 7 góc camera trong `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ (`s01f09`, `s01f24`), kèm 4 ảnh minh họa: `t1-s01f09-3d.png`, `t1-s01f09-top.png`, `t1-s01f24-3d.png`, `t1-s01f24-top.png` (trên hai ảnh `-top` đã đánh dấu hướng phía trước `-y`, trục `+x` sang trái và trục `+y` về phía sau).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f09` | Phía trước xe (`-y`) hướng về **cạnh dưới** của ảnh nhìn từ trên xuống; trục `+x` chỉ sang **bên trái** xe (cạnh phải ảnh khi nhìn từ trên xuống), trục `+y` chỉ về **phía sau** xe (cạnh trên ảnh). | Mặt đường tương đối phẳng, thấp hơn cảm biến LiDAR khoảng `1,70 m` (các vòng quét mặt đất quanh `z ≈ -1,70 m`). | Khoảng `56 m` (xe buýt `idx #3` ở `dist_m = 55,72 m`). | Xe con (`Car`), xe buýt (`Bus`). | Có 1 xe con phía sau (`idx #4` ở `20,18 m`) và 4 xe con chạy làn bên phải/ngược chiều (`idx #5–#8` từ `14,57 m` đến `29,69 m`). | Một số cụm điểm nhỏ ở xa ngoài `50 m` và cụm điểm bị dải phân cách che khuất một phần phía làn đối diện. |
| `s01f24` | Phía trước xe (`-y`) hướng về **cạnh dưới** của ảnh nhìn từ trên xuống; trục `+x` chỉ sang **bên trái** xe, trục `+y` chỉ về **phía sau** xe. | Mặt đường phẳng, độ cao mặt đường ổn định quanh mức `z ≈ -1,70 m` so với tâm LiDAR. | Khoảng `35–40 m` (các cuboid gán nhãn rõ nét nhất trong vòng `22,27 m`). | Xe con (`Car`), xe buýt (`Bus`). | Có 3 xe con và 1 xe buýt đi phía sau xe thu dữ liệu (`idx #1–#4` ở khoảng cách `6,42 m` đến `22,27 m`) và 1 xe con phía trước bên trái (`idx #5` ở `6,22 m`). | Các cụm điểm mờ sát lề đường ngoài `40 m` chỉ còn vài điểm rời rạc không đủ nhận diện hình khối. |

- **3 điều chỉ camera cho biết (qua `s01f09` và `s01f24`):**
  1. Phân biệt chính xác đầu xe và đuôi xe thông qua đèn pha trắng phía trước và đèn hậu đỏ phía sau (trên point cloud hai đầu xe gần giống nhau).
  2. Phân loại đúng nhóm phương tiện khi kích thước gần nhau (ví dụ phân biệt xe SUV/van chở người thuộc `Car` với xe bán tải/tải nhẹ thuộc `Truck`, hoặc nhận diện xe buýt hành khách thuộc `Bus`).
  3. Xác nhận màu sắc, trạng thái tín hiệu giao thông và bối cảnh vạch kẻ làn đường, dải phân cách.
- **3 điều chỉ LiDAR cho biết (qua `s01f09` và `s01f24`):**
  1. Tọa độ không gian 3D chính xác (`x`, `y`, `z`) và khoảng cách thực tế tính bằng mét (`dist_m`) tới từng phương tiện trong vùng quét 360°.
  2. Chiều dài, chiều rộng, chiều cao vật lý thực của thân xe mà không bị biến dạng phối cảnh như trên ảnh 2D.
  3. Vùng bóng che thực tế phía sau các xe lớn (các vòng tròn quét mặt đất bị ngắt quãng tạo thành vệt trống phía sau vật cản).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f01` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Có xe buýt rất gần (`idx #3` ở `6,29 m`) thu được tới `12.610` điểm LiDAR, tạo vùng bóng che lớn phía sau. |
| `s02f24` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Có 3 xe ở làn ngược chiều (`idx #1, #2, #5`) hướng mũi tên về phía xe thu dữ liệu (`yaw_deg` ~ `86°`). |
| `s03f16` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Xuất hiện xe tải/bán tải ngay sát bên trái (`idx #3` ở `2,74 m`) với mật độ điểm dày đặc (`8.397` điểm). |
| `s03f34` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Khu vực đô thị có cả người đi bộ (`Pedestrian`) và xe hai bánh có người lái (`Cyclist`) di chuyển gần lề đường. |
| `s04f05` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Mật độ xe con di chuyển cùng chiều khá đều từ `3,62 m` đến `27,02 m`, mặt đường phẳng. |
| `s04f20` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Các xe con tập trung chủ yếu ở phía trước (`y < 0`) trong bán kính `15 m`, cụm điểm rõ nét. |
| `s05f21` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Có xe di chuyển cả phía trước (`3` xe) và phía sau (`2` xe) dọc theo trục đường chính. |
| `s05f37` | Cạnh dưới ảnh (`-y` hướng xuống dưới). | Có xe buýt ở rất xa phía trước (`idx #3` ở `60,66 m`) và một xe lớn ở `35,32 m` (`idx #2`) cụm điểm bắt đầu thưa. |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ba ảnh minh họa: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f09`, `idx #2` | `Car` | Có (`yaw_deg = 267,6°`, chỉ đúng đầu xe hướng `-y`). | Có (`z_bottom = -1,763 m`, `ground_gap_m = 0,03 m`). | Có, đã căn sát trên Top/Side/Front và kiểm tra trên ảnh `preview`. | Có (`l = 4,447 m`, `w = 1,919 m`, `h = 1,565 m`, khớp chuẩn sedan/SUV trong Bảng 4.3). |
| (b) xe buýt hoặc xe tải | `s02f01`, `idx #3` | `Bus` | Có (`yaw_deg = 270,9°`, chỉ đúng đầu xe buýt). | Có (`z_bottom = -1,738 m`, `ground_gap_m = 0,11 m`). | Có, đã kiểm tra trên 3 khung chiếu và ảnh camera `preview`. | Gần chuẩn (`l = 12,493 m`, `h = 3,311 m` nằm trong Bảng 4.3; `w = 2,955 m` hơi rộng hơn ngưỡng 2,50–2,60 m do bao trọn phần gương chiếu hậu hai bên). |
| (c) người đi bộ hoặc xe hai bánh | `s03f34`, `idx #3` | `Pedestrian` | Có (`yaw_deg = 94,4°`, chỉ đúng hướng người đi bộ di chuyển). | Có (`z_bottom = -1,766 m`, `ground_gap_m = -0,01 m`). | Có, đã phóng to kiểm tra sát cụm điểm và đối chiếu ảnh camera. | Có (`l = 0,843 m`, `w = 0,808 m`, `h = 1,543 m`, nằm trọn trong ngưỡng người lớn `0,4–0,9 × 0,5–0,9 × 1,50–1,95 m`). |

- **Hiệu chuẩn cá nhân và các vật bỏ qua:**
  - Không thực hiện vẽ trên task chung `D13-HC485087`; toàn bộ quá trình hiệu chuẩn được thực hiện trực tiếp trên task cá nhân thông qua đối chiếu giữa giao diện CVAT, ảnh chiếu `work/xem/*.jpg` (`preview`) và phiếu tự kiểm `tukiem.csv`. Qua các lần kiểm tra, tôi đã điều chỉnh các lỗi thường gặp ban đầu là đặt đáy hộp hơi cao hơn mặt đường (`0,4–0,5 m`) ở xe tầm trung và bao dư chiều rộng do dính điểm phản xạ lề đường, đảm bảo đáy hộp chạm vệt mặt đường quanh `-1,7 m` và vạch đỏ `+X` chỉ đúng về đầu xe.
  - Các vật bỏ qua vì không đủ cơ sở: trong các frame `s01f24`, `s02f01`, `s05f21`, một số cụm điểm ở khoảng cách ngoài `45–50 m` hoặc bị xe buýt/xe tải phía trước che khuất hoàn toàn chỉ còn dưới 10% thể tích (dưới 15–20 điểm rời rạc, thuộc diện `LOW_VISIBILITY` và `UNCERTAIN_SCOPE`) nên đã bỏ qua, không vẽ đoán.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Bốn ảnh minh họa: `t4-1-bong-che.png`, `t4-2-thua-o-xa.png`, `t4-3-phan-xa-thap.png`, `t4-4-diem-la.png`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f01` / `t4-1-bong-che.png` / Task cá nhân `D13-2A202604039` | Xe buýt lớn (`idx #3` ở `6,29 m`) chắn trực tiếp đường truyền thẳng của chùm tia laser từ cảm biến LiDAR về phía trước bên trái. | Phía sau xe buýt xuất hiện một vùng đen trống trơn hình rẻ quạt, các vòng tròn quét mặt đất bị cắt đứt hoàn toàn; vật nằm sau chỉ lộ một góc nhỏ hoặc bị mất hẳn thân. | Nếu vật bị che vẫn nhận diện được (≥ 15–20% thể tích hoặc rõ trên camera), phải bật cờ `occluded` và suy luận kích thước hộp theo Bảng 4.3; nếu mất quá nhiều điểm thì bỏ qua, không đoán. |
| 2. Thưa ở xa | `s03f34` / `t4-2-thua-o-xa.png` / Task cá nhân `D13-2A202604039` | Góc phân kỳ giữa các tia laser tăng tuyến tính theo khoảng cách nên mật độ điểm rơi trên bề mặt vật ở xa giảm mạnh. | Xe con (`idx #4` ở `31,88 m`) chỉ còn `104` điểm (so với `3.863` điểm của xe gần `idx #1` ở `4,89 m`), chủ yếu là vài vệt ngang ở đuôi xe, không thấy rõ bánh xe và viền nóc. | Phải dựa vào vệt mặt đất xung quanh để hạ đáy hộp đúng `-1,7 m` và dùng Bảng 4.3 để giữ chiều dài/rộng hợp lý; với vật ngoài `50 m` hoặc quá thưa không đủ 15–20% thể tích thì không vẽ. |
| 3. Phản xạ thấp / ít điểm bất thường | `s02f01` / `t4-3-phan-xa-thap.png` / Task cá nhân `D13-2A202604039` | Bề mặt sơn tối màu, kính xe nghiêng hoặc góc tới của tia laser quá xiên làm năng lượng phản xạ về đầu thu yếu hơn ngưỡng ghi nhận. | Xe ở tầm trung có phần thân tối màu bị khuyết mảng điểm ở giữa cửa xe hoặc kính chắn gió dù không có vật cản phía trước, trong khi viền cản và bánh xe vẫn hiện rõ. | Không bật `occluded` (vì không bị vật khác che khuất); dùng khung viền các điểm còn lại kết hợp ảnh camera và Bảng 4.3 để bao trọn toàn bộ thân xe. |
| 4. Điểm lạ | `s01f09` / `t4-4-diem-la.png` / Task cá nhân `D13-2A202604039` | Phản xạ đa đường (multipath) qua kính/kim loại bóng, bụi lơ lửng hoặc nhiễu điện tử của cảm biến tạo ra các điểm nằm lệch khỏi bề mặt thật. | Xuất hiện một vài điểm đơn lẻ lơ lửng phía trên nóc xe (`idx #7`) hoặc nằm tách rời ngoài viền thân xe. | Không kéo giãn cuboid để bao các điểm nhiễu lơ lửng này; chỉ căn sát 6 mặt hộp vào bề mặt thân xe thực tế để tránh bị lệch kích thước chuẩn. |

## 5. Số từ cuboid của tôi (Bước 5)

Số liệu trích xuất từ `submission/D13-2A202604039-bangA.csv` và `submission/D13-2A202604039-tukiem.csv`:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f09`, `#2`, `Car` | `-0,980 m` | `1,565 m` | `-1,763 m` | `5,01 m` | `5,10 m` | `267,6°` | Chạm đường (`z` đáy lệch mặt đường chuẩn `-1,7 m` chỉ `0,063 m`, `ground_gap_m = 0,03 m`). |
| `s01f09`, `#7`, `Car` | `-0,787 m` | `2,392 m` | `-1,983 m` | `27,07 m` | `27,08 m` | `90,5°` | Chạm đường trong ngưỡng cho phép (`z` đáy lệch `-1,7 m` là `0,283 m ≤ 0,3 m`, `ground_gap_m = 0,13 m`). |
| `s01f09`, `#8`, `Car` | `-1,283 m` | `1,715 m` | `-2,140 m` | `29,69 m` | `29,72 m` | `87,0°` | Hơi chìm so với mốc `-1,7 m` (lệch `0,440 m` do độ dốc mặt đường ở xa `29,69 m`, nhưng `ground_gap_m = 0,05 m` sát điểm mặt đất tại chỗ). |

- **(a) Cuboid gần nhất và xa nhất:**
  - *Gần nhất:* `s03f16, #3 (Truck)` với `dist_m = 2,74 m` và `r = 2,83 m`.
  - *Xa nhất:* `s05f37, #3 (Bus)` với `dist_m = 60,66 m` và `r = 60,66 m` (xét riêng trong phạm vi ≤ 50 m thì cuboid xa nhất là `s05f37, #2 (Car)` với `dist_m = 35,32 m` và `r = 35,33 m`).
- **(b) So sánh kích thước với Bảng 4.3:**
  - *Người đi bộ (`s03f34, #3, Pedestrian`):* `l = 0,843 m`, `w = 0,808 m`, `h = 1,543 m` — cả ba chiều đều nằm gọn trong khoảng chuẩn của người lớn theo Bảng 4.3 (`l`: `0,4–0,9 m`, `w`: `0,5–0,9 m`, `h`: `1,50–1,95 m`).
  - *Xe con (`s01f09, #7, Car`):* `l = 3,799 m`, `w = 2,242 m`, `h = 2,392 m` — chiều dài đạt chuẩn (`3,8–5,0 m`), nhưng chiều rộng (`2,242 m`) và chiều cao (`2,392 m`) vượt ngoài khoảng chuẩn của xe con (`w`: `1,70–2,10 m`, `h`: `1,40–2,10 m`) vì xe ở khoảng cách xa `27,07 m` bị lẫn điểm phản xạ phía trên nóc và rìa sườn khiến hộp bị kéo dư.
- **(c) Góc hướng `yaw_deg`:**
  - *Xe đi cùng chiều phía trước (`s01f09, #1, Car`, tại `y = -22,658 m`):* `yaw_deg = 270,1°`, gần như trùng khớp với mốc `270°` (hướng `-y` của xe thu dữ liệu).
  - *Xe đi ngược chiều (`s01f09, #5, Car`):* `yaw_deg = 92,0°`, rất sát mốc `90°` (ngược chiều `180°` so với hướng di chuyển của xe thu dữ liệu).

Mật độ theo khoảng cách (4 cuboid lớp `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f24`, `#4`, `Car` | `6,42 m` | `1968` |
| `s02f24`, `#1`, `Car` | `14,63 m` | `712` |
| `s05f37`, `#4`, `Car` | `25,50 m` | `245` |
| `s03f34`, `#4`, `Car` | `31,88 m` | `104` |

- **Quy luật mật độ và đối chiếu slide bài giảng:** Khi khoảng cách tăng từ `6,42 m` lên `31,88 m` (xa gấp ~`5,0` lần), số điểm LiDAR bên trong cuboid giảm từ `1968` điểm xuống còn `104` điểm (còn khoảng `5,3%`, tức giảm gần `19` lần). Các mốc thực đo này bám rất sát đường cong lý thuyết trên slide bài giảng (`1968` điểm ở `0–10 m`, `718` điểm ở `10–20 m`, `214` điểm ở `20–30 m` và `72` điểm ở `30–40 m`).
- **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`:**
  - *Các cuboid đã sửa:* Sau lần chạy `sync` đầu tiên có 9 cảnh báo, tôi đã chỉnh sửa nhiều cuboid trên CVAT, tiêu biểu là: (1) hạ đáy hộp `s01f09 #4 (Car)` từ trạng thái lơ lửng `0,46 m` xuống chạm vệt mặt đường (`z_bottom = -1,686 m`, `ground_gap_m = 0,01 m`); (2) thu gọn chiều rộng hộp `s02f01 #3 (Bus)` từ `3,36 m` xuống còn `2,955 m` để loại bỏ điểm nhiễu lề đường và xóa cảnh báo kích thước.
  - *Cảnh báo giữ nguyên và lý do:* Sau lần `sync` thứ hai còn 2 cảnh báo kích thước ở `s01f09 #7 (Car)` (`height = 2,39 m`) và `s05f37 #2 (Car)` (`width = 2,48 m`, `height = 2,35 m`). Tôi giữ nguyên vì đây là các xe thân lớn ở khoảng cách xa (`27,07 m` và `35,32 m`), chiều cao đáy đã chạm đúng mặt đường (`ground_gap_m` lần lượt là `0,13 m` và `0,08 m`) và khung chiếu trên ảnh `preview` đã khớp vị trí xe.
- **Đổi kiểm:** Thực hiện bài lab cá nhân độc lập nên không có bước đổi kiểm chéo với MSSV khác; đã tự đối chiếu chéo trực tiếp toàn bộ thông số trong `bangA.csv` (như `s01f09 #2 Car` có `dist_m = 5,01 m`, `yaw_deg = 267,6°` và `s03f34 #3 Pedestrian` có `height = 1,543 m`, `z_bottom = -1,766 m`) với khung hộp trên CVAT và ảnh chiếu `preview`, kết quả trùng khớp hoàn toàn.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Góc nhìn mô hình học máy:** Nhãn 3D phải đo bằng đơn vị mét thực tế thay vì pixel vì các thuật toán định vị, dự đoán quỹ đạo và lập kế hoạch chuyển động của xe tự lái cần biết khoảng cách va chạm, kích thước hình học thật và vận tốc tương đối trong không gian vật lý 3D (điều mà ảnh 2D bị biến dạng phối cảnh không thể cung cấp trực tiếp). Vật thể thưa điểm cần có quy tắc suy luận kích thước riêng vì nếu gán hộp co rúm theo số điểm ít ỏi ở xa, mô hình sẽ học sai hình dạng vật thể thật; ngược lại nếu gán bừa khi không đủ điểm sẽ tạo nhãn nhiễu làm giảm độ chính xác nhận diện.
- **(b) Cách dùng AI:**
  - *Công cụ:* Gemini.
  - *Câu hỏi chính:* Giải thích quy ước hệ tọa độ cục bộ 3 mũi tên trong CVAT, cơ chế chiếu cuboid 3D lên 7 ảnh camera qua `preview`/`sync`, cách xử lý các cảnh báo trong `tukiem.csv` và rà soát tính nhất quán số liệu báo cáo.
  - *Cách sử dụng kết quả:* Dùng để tra cứu nhanh quy tắc guideline, hiểu nguyên nhân cảnh báo lơ lửng/kích thước trên CVAT và tổng hợp số liệu từ hai file CSV vào báo cáo; toàn bộ thao tác vẽ cuboid và kiểm tra trực quan đều tự thực hiện trên máy cá nhân.
- **(c) Điều còn chưa chắc và lỗi phát hiện sau:** Còn phân vân ở một số phương tiện thân cao tầm xa như `s05f37 #2` (`dist_m = 35,32 m`, kích thước lớn hơn xe con thông thường nhưng chưa rõ thùng hàng phía sau để khẳng định là `Truck` hay `Car` cỡ lớn), và hai xe buýt ở xa (`s01f09 #3` ở `55,72 m` và `s05f37 #3` ở `60,66 m`) vượt nhẹ ngưỡng phạm vi `50 m` của buổi lab dù cụm điểm vẫn khá rõ trên point cloud.