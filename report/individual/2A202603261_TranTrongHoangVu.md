# Báo cáo Day 13 — Tran Trong Hoang Vu, 2A202603261

## 1. Kết luận trước
- **MSSV:** `2A202603261` · **Task:** `D13-2A202603261`
- **10 frame được giao:** `s01f02`, `s01f27`, `s02f01`, `s02f52`, `s03f20`, `s03f59`, `s04f23`, `s04f51`, `s05f01`, `s05f19`
- **Môi trường:** Ubuntu Linux · Trình duyệt: FIREFOX · CVAT:  · MSSV bạn ngồi cạnh: 
- **Tổng quan kết quả:** Đã vẽ **56 cuboid** trên 10 frame; các frame ít nhất là `s02f01`, `s02f52`, `s03f59` (4 cuboid/frame, do chỉ gán các cụm điểm đủ rõ). Phiếu `selfcheck` cờ **14/56 cuboid** (chủ yếu do `width` xe con nhỉnh hơn ngưỡng 2,3 m hoặc nhiều điểm sát mép). Phần tin cậy nhất là định hướng đầu xe (`yaw_deg`) và độ cao đáy trên các frame gần; phần còn yếu nhất là căn sát chiều rộng (`width`) ở các xe hơi thưa điểm hoặc bị che khuất.
- **Cách dùng CVAT (3 dòng):**
  1. Đổi góc nhìn: kéo chuột trái để xoay, kéo chuột phải để dời, cuộn con lăn để phóng to/thu nhỏ; xoay nhìn thẳng từ trên xuống để quan sát toàn cảnh, nháy đúp vào cuboid để căn giữa.
  2. Vẽ và chỉnh cuboid: chọn nút cuboid → chọn class → bấm **Shape**, nháy đúp vào cụm điểm để đặt; chỉnh tâm, kích thước và góc xoay ở khung **Top**, chỉnh độ cao và đáy chạm đường ở **Side** / **Front** (không xoay ở hai khung này).
  3. Hướng và đối chiếu ảnh: bật **Appearance → Cuboid orientation** để căn vạch đỏ `+X` về đầu xe, rồi chạy `python scripts/lab.py preview` để kiểm tra cuboid chiếu lên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ (`s01f02`, `s01f27`). Ảnh: `t1-s01f02-3d.png`, `t1-s01f02-top.png`, `t1-s01f27-3d.png`, `t1-s01f27-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f02` | [CẦN ĐIỀN theo ảnh top: phía trước −y ở cạnh nào, +x (trái) và +y (sau) chỉ về đâu] | [CẦN ĐIỀN: phẳng/dốc, ~1,7 m] | [CẦN ĐIỀN: ~ m] | Car, Truck, Pedestrian | [CẦN ĐIỀN: vật phía sau/hai bên thấy ở cam nào] | [CẦN ĐIỀN] |
| `s01f27` | [CẦN ĐIỀN theo ảnh top: phía trước −y ở cạnh nào, +x (trái) và +y (sau) chỉ về đâu] | [CẦN ĐIỀN: phẳng/dốc, ~1,7 m] | [CẦN ĐIỀN: ~ m] | Car, Truck | [CẦN ĐIỀN: vật phía sau/hai bên thấy ở cam nào] | [CẦN ĐIỀN] |

- **3 điều chỉ camera cho biết:** màu sắc xe/quần áo, chi tiết nhận dạng loại xe (bán tải, taxi, biển báo/đèn tín hiệu), hướng đầu/đuôi xe khi cụm điểm ít tia.
- **3 điều chỉ LiDAR cho biết:** khoảng cách chính xác theo mét (`dist_m`), kích thước 3D thực (`length`, `width`, `height`), vị trí/độ cao mặt đường (`z ≈ −1,7 m`) kể cả trong góc mù giữa các camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f01` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |
| `s02f52` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |
| `s03f20` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |
| `s03f59` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |
| `s04f23` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |
| `s04f51` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |
| `s05f01` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |
| `s05f19` | [CẦN ĐIỀN] | [CẦN ĐIỀN] |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f02`, idx `1` | Car | Đúng (`90,4°`, ngược chiều) | Chạm (`z_bottom = -1,771 m`) | Đã kiểm | `4,14 × 2,23 × 1,61 m` (`length`, `height` đạt; `width` hơi rộng hơn 1,70–2,10 m) |
| (b) xe buýt hoặc xe tải | `s02f01`, idx `1` | Bus | Đúng (`271,2°`, cùng chiều) | Chạm (`z_bottom = -1,964 m`, `gap = -0,13 m`) | Đã kiểm | `12,05 × 3,44 × 3,61 m` (`length` đạt; `width`, `height` hơi lớn hơn 2,50–2,60 × 3,00–3,50 m) |
| (c) người đi bộ hoặc xe hai bánh | `s01f02`, idx `5` | Pedestrian | Đúng (`80,6°`) | Gần chạm (`z_bottom = -1,547 m`, `gap = +0,18 m`) | Đã kiểm | `0,82 × 0,64 × 1,53 m` (nằm trong khoảng `0,4–0,9 × 0,5–0,9 × 1,50–1,95 m`) |

- **Vòng hiệu chuẩn:** [CẦN ĐIỀN: cuboid của bạn lệch ở điểm nào so với lớp, cả lớp chốt quy tắc nào]
- **Các vật bỏ qua vì không đủ cơ sở:** [CẦN ĐIỀN: ghi frame, vị trí, mô tả, loại nghi vấn hoặc "không có"]

## 4. Bốn hiện tượng chất lượng (Bước 4)
| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | [CẦN ĐIỀN frame] / `t4-1-bong-che.png` / [tự chụp hay mượn nguồn] | Giả thuyết: tia LiDAR bị vật cản lớn ở gần chặn nên không tới được vùng phía sau | [CẦN ĐIỀN: mô tả vùng trống phía sau vật cản] | Bật `occluded`, cuboid vẫn bao trọn phần kích thước suy luận được |
| 2. Thưa ở xa | [CẦN ĐIỀN frame] / `t4-2-thua-o-xa.png` / [nguồn] | Giả thuyết: mật độ tia LiDAR giảm mạnh theo khoảng cách (chùm tia phân kỳ) | [CẦN ĐIỀN: mô tả xe ở ~30–50 m chỉ còn vài điểm] | Chỉ vẽ nếu thấy khoảng 15–20% thể tích hoặc đủ cơ sở suy luận; không thì bỏ qua, ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | [CẦN ĐIỀN frame] / `t4-3-phan-xa-thap.png` / [nguồn] | Giả thuyết: bề mặt tối màu, kính hoặc góc tới xiên làm tia phản xạ yếu | [CẦN ĐIỀN: mô tả vật ít điểm hơn vật cùng tầm] | Ghi nhận giả thuyết, đối chiếu ảnh camera, không khẳng định chắc chắn nguyên nhân |
| 4. Điểm lạ | [CẦN ĐIỀN frame] / `t4-4-diem-la.png` / [nguồn] | Giả thuyết: nhiễu đa đường (multipath), bụi/mưa hoặc lỗi phản xạ | [CẦN ĐIỀN: mô tả cụm điểm lơ lửng hoặc dưới mặt đất] | Không gán thành vật |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603261-bangA.csv` và `submission/D13-2A202603261-tukiem.csv`.

| Cuboid (frame, idx, class) | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r = √(x²+y²+z²) (m) | yaw_deg (°) | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| `s01f02`, `1`, `Car` | -0,967 | 1,607 | -1,771 | 6,37 | 6,45 | 90,4° | Chạm đường (`z đáy ≈ -1,7 m`, `ground_gap = +0,03 m`) |
| `s03f20`, `4`, `Pedestrian` | -0,248 | 1,813 | -1,155 | 14,97 | 14,97 | 260,3° | Hơi lơ lửng (`z đáy = -1,155 m`, `ground_gap = +0,41 m`) |
| `s03f20`, `6`, `Car` | -1,391 | 1,669 | -2,226 | 22,21 | 22,25 | 79,1° | Thấp hơn -1,7 m nhưng sát mặt đường dốc tại chỗ (`ground_gap = -0,06 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `s05f01`, idx `1` (`Car`) — `dist_m = 5,29 m`, `r = 5,35 m`.
  - Xa nhất: `s04f51`, idx `5` (`Car`) — `dist_m = 32,98 m`, `r = 32,99 m`.
- **(b) So kích thước với bảng 4.3:**
  - Người đi bộ `s01f02`, idx `5` (`Pedestrian`): `l = 0,821 m`, `w = 0,640 m`, `h = 1,527 m` — cả 3 chiều nằm trọn trong khoảng chuẩn người lớn (`0,4–0,9 × 0,5–0,9 × 1,50–1,95 m`).
  - Xe con `s01f02`, idx `1` (`Car`): `l = 4,141 m`, `w = 2,232 m`, `h = 1,607 m` — `length` trong khoảng (`3,8–5,0 m`), `height` xấp xỉ ngưỡng trên (`1,40–1,60 m`), còn `width = 2,232 m` vượt nhẹ ngưỡng `1,70–1,90 m` do vẽ hộp dư nhẹ hai bên mép hoặc bao cả gương chiếu hậu.
- **(c) Góc `yaw_deg` theo hướng di chuyển:**
  - Xe cùng chiều phía trước `s01f27`, idx `2` (`Car`, `y = -11,582 m`): `yaw_deg = 271,9°` (rất gần `270°`, tức hướng `−y`).
  - Xe ngược chiều `s01f02`, idx `1` (`Car`, `y = -5,740 m`): `yaw_deg = 90,4°` (rất gần `90°`, tức hướng `+y`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m (m) | Số điểm (`points_in`) |
|---|---:|---:|
| `s05f01`, `1`, `Car` | 5,29 | 4514 |
| `s01f27`, `2`, `Car` | 12,24 | 825 |
| `s04f23`, `4`, `Car` | 18,04 | 618 |
| `s04f51`, `5`, `Car` | 32,98 | 151 |

- **Quy luật mật độ:** Khi khoảng cách tăng gấp ~2,3 lần (từ `5,29 m` lên `12,24 m`), số điểm giảm còn khoảng 18% (từ `4514` xuống `825`), và tới `32,98 m` chỉ còn `151` điểm (~3,3%); xu hướng giảm nhanh này phù hợp với đường cong bài giảng (xe con trung vị: ~1968 ở 0–10 m, ~718 ở 10–20 m, ~214 ở 20–30 m, ~72 ở 30–40 m).
- **Tự kiểm và chỉnh sửa sau `sync`:**
  - Đã sửa ≥ 2 cuboid trong CVAT và chạy lại `sync`:
    1. `s02f01`, idx `2` (`Car`): hạ đáy từ `z_bottom = -1,270 m` (`ground_gap = +0,63 m`) xuống `z_bottom = -2,065 m` (`ground_gap = -0,17 m`) để hết lơ lửng.
    2. `s02f52`, idx `2` (`Car`): hạ đáy từ `z_bottom = -1,682 m` (`ground_gap = +0,56 m`) xuống `z_bottom = -2,061 m` (`ground_gap = +0,18 m`) để sát mặt đường.
    3. `s03f59`, idx `4`: sửa class từ `Car` sang `Truck` (vì kích thước `9,11 × 3,28 × 3,22 m`).
    4. `s01f27`, idx `1` (`Car`): chỉnh lại kích thước và độ cao đáy (`z_bottom = -1,594 m`).
  - Các cảnh báo giữ nguyên và lý do: các cảnh báo `width` nhỉnh hơn `2,3 m` nhẹ (như `s01f27 #1, #2`, `s04f23 #6`) hoặc điểm sát mép (`s01f02 #4`, `s03f20 #1`) được giữ lại sau khi đối chiếu cụm điểm vì bao phần mép ngoài/gương xe hoặc giáp ranh điểm nền xung quanh; `s02f01 #1` (`Bus`) có `7617` điểm sát mép do xe buýt lớn sát dải phân cách/mặt đường.
- **Đổi kiểm với bạn bên cạnh:** [CẦN ĐIỀN: MSSV bạn kiểm chéo, 2 con số đã đối chiếu với bảng A, khớp hay lệch]

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa từ phía mô hình học:**
  - Nhãn 3D dùng đơn vị **mét** thay vì pixel vì xe tự hành cần biết vị trí, khoảng cách và kích thước vật thể trong không gian 3D thực để lập kế hoạch chuyển động và tránh va chạm, không bị biến dạng phối cảnh theo tiêu cự camera.
  - Vật thưa điểm cần quy tắc riêng (ngưỡng ~15–20% thể tích, không đoán mò) để tránh đưa nhãn sai kích thước hoặc hộp giả vào tập huấn luyện, giúp mô hình không học phải nhiễu vị trí/hình học.
- **(b) Cách dùng AI:**
  - Công cụ: trợ lý AI trong IDE.
  - Câu hỏi chính: hướng dẫn lệnh chạy môi trường/CVAT, kiểm tra cấu trúc báo cáo và tổng hợp số liệu từ bảng A / phiếu tự kiểm.
  - Cách dùng kết quả: đối chiếu lại từng con số với `D13-2A202603261-bangA.csv` và `D13-2A202603261-tukiem.csv`, tự thực hiện gán nhãn và chụp ảnh trong CVAT.
- **(c) Điều còn chưa chắc:** Một số hộp ở khung `s01f02` (`#3` dài `2,94 m`, `#7` có `yaw_deg = 0,0°`, `#8` cao `1,08 m`) và các xe ở rìa cụm điểm hơi thưa vẫn còn chênh kích thước so với bảng 4.3, cần kiểm lại trên ảnh camera nếu có thêm thời gian trước hạn nộp.
