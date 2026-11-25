# Báo cáo Day 13 — Pham Duc Viet, 2A202603265

## 1. Kết luận trước
- Mã số sinh viên: 2A202603265.
- Task CVAT: `D13-2A202603265`.
- 10 frame của tôi: `s01f07`, `s01f41`, `s02f09`, `s02f42`, `s03f04`, `s03f52`, `s04f01`, `s04f48`, `s05f27`, `s05f44`.
- Môi trường thực hiện: Hệ điều hành Windows, trình duyệt Google Chrome, phiên bản CVAT v2.83.0 (tài khoản admin). MSSV bạn ngồi cạnh: 2A202600128.
- Tổng số cuboid đã gán nhãn: 100 cuboid trên toàn bộ 10 frame. Frame có ít cuboid nhất là `s02f42` và `s03f52` (đều có 6 cuboid) do đây là các đoạn đường thông thoáng, ít phương tiện và các vật thể ở xa hơn 50 m đã được lọc bỏ theo quy định. Công cụ `selfcheck` gắn cờ cảnh báo 29 cuboid trên tổng số 100 cuboid (chủ yếu là cảnh báo mật độ điểm ở xa hoặc khoảng hở đáy). Phần tôi tự tin nhất là vị trí không gian (x, y, z) và căn chỉnh đáy chạm sát mặt đường cho các vật thể ở gần (<30 m); phần còn cảm thấy yếu nhất là ước lượng chiều dài chính xác cho các xe ở cự ly xa (>35 m) khi điểm LiDAR bị thưa.

Cách dùng CVAT (3 dòng):
1. Đổi góc nhìn: Kéo chuột trái để xoay phối cảnh 3D, kéo chuột phải để di chuyển tâm nhìn, lăn con lăn chuột để phóng to/thu nhỏ; nháy đúp chuột vào khoảng trống để đưa góc nhìn về mặc định.
2. Vẽ và chỉnh cuboid: Chọn công cụ vẽ cuboid trên thanh bên trái, chọn label phù hợp, chọn chế độ Shape; rê chuột đến cụm điểm rồi nháy đúp để đặt hộp, sau đó chỉnh vị trí và kích thước trên 3 khung chiếu Top/Side/Front.
3. Hướng và camera: Vào menu Appearance bật Cuboid orientation để xoay trục yaw (+X chỉ về đầu xe/hướng di chuyển); chạy lệnh `python scripts/lab.py preview` để đối chiếu hình chiếu cuboid trên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: `s01f07` và `s01f41`. Ảnh minh chứng: `t1-s01f07-3d.png`, `t1-s01f07-3d-top.png`, `t1-s01f41-3d.png`, `t1-s01f41-3d-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f07` | Phía trước xe là hướng -y (hướng xe chạy tới); trục +x chỉ sang bên trái xe, trục +y chỉ về phía sau đuôi xe | Mặt đường tương đối phẳng, thấp hơn vị trí cảm biến LiDAR khoảng 1,7 m (z ≈ -1,7 m) | ~48 m (xe con ở xa trên trục đường) | Car, Pedestrian, Cyclist | Có các xe con đỗ và di chuyển ở hai bên hông; phía sau có xe bám đuôi quan sát qua cam09 | Cụm điểm phản xạ tán xạ ở mép cỏ/vỉa hè bên phải cách ~35 m |
| `s01f41` | Phía trước xe là hướng -y; trục +x chỉ sang bên trái xe, trục +y chỉ về phía sau đuôi xe | Mặt đường phẳng, cao độ đáy xe ổn định, thấp hơn LiDAR ~1,7 m | ~45 m | Car, Cyclist | Xe lưu thông cùng chiều ở làn bên trái và xe đối diện làn bên phải | Cụm điểm thưa ở rìa xa >50 m khó xác định hình dáng vật thể |

3 điều **chỉ camera** cho biết:
1. Màu sắc xe thực tế (trắng, đen, đỏ, bạc) và trạng thái đèn tín hiệu/đèn pha xe.
2. Chi tiết nhận dạng loại phương tiện và chữ viết (biển số xe, logo, tem dán xe, phân biệt xe taxi và xe cá nhân).
3. Ranh giới rõ ràng của các bề mặt trong suốt như kính chắn gió hoặc vạch kẻ đường, vũng nước mà LiDAR bị xuyên qua hoặc tán xạ.

3 điều **chỉ LiDAR** cho biết:
1. Tọa độ không gian 3D và khoảng cách hình học chính xác tuyệt đối (x, y, z tính bằng mét) đến từng bề mặt vật thể.
2. Kích thước 3D thực tế của vật thể (dài, rộng, cao) mà không bị biến dạng do phối cảnh ống kính camera.
3. Tầm quan sát toàn cảnh 360 độ liên tục, không bị điểm mù giữa các góc camera và hoàn toàn không phụ thuộc vào điều kiện ánh sáng (ngày/đêm, ngược sáng).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f09` | Hướng -y (phía trước xe) | Có xe buýt lớn ở làn đối diện cự ly gần (~10,8 m), che khuất một góc nhìn lớn phía sau |
| `s02f42` | Hướng -y (phía trước xe) | Đường thông thoáng, mật độ phương tiện thưa thớt, chỉ có 6 cuboid đủ điều kiện |
| `s03f04` | Hướng -y (phía trước xe) | Xuất hiện nhóm người đi bộ và người đi xe hai bánh di chuyển gần mép đường |
| `s03f52` | Hướng -y (phía trước xe) | Mật độ xe ít, có xe máy (Cyclist) di chuyển cùng chiều bên trái |
| `s04f01` | Hướng -y (phía trước xe) | Mật độ giao thông rất đông đúc (15 cuboid), nhiều xe đỗ sát nhau |
| `s04f48` | Hướng -y (phía trước xe) | Có xe tải/bán tải di chuyển phía trước, các vạch sơn đường phản xạ tia rõ |
| `s05f27` | Hướng -y (phía trước xe) | Góc cua đường, xuất hiện chướng ngại vật ven đường |
| `s05f44` | Hướng -y (phía trước xe) | Tầm nhìn mở rộng, phát hiện xe ở xa tới 48,2 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh minh chứng: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f07`, idx 2 | Car | Có, yaw = 267,4° (mũi tên hướng về đầu xe) | Có, z đáy = -1,92 m (chạm mặt đường) | Đã kiểm tra cả 3 khung và khớp ảnh cam01 | Có (4,99 × 2,21 × 1,74 m, trong khoảng SUV/Sedan) |
| (b) xe buýt hoặc xe tải | `s02f09`, idx 6 | Bus | Có, yaw = 85,0° (ngược chiều, đầu xe hướng về trước) | Có, z đáy = -1,85 m | Đã kiểm đủ 3 view và ảnh camera bên | Có (11,94 × 3,07 × 3,00 m, đúng chuẩn xe buýt lớn) |
| (c) người đi bộ hoặc xe hai bánh | `s03f52`, idx 4 | Cyclist | Có, yaw = 271,2° (hướng theo chiều di chuyển) | Có, z đáy = -1,76 m | Đã kiểm đủ 3 view và ảnh camera | Có (1,57 × 0,68 × 1,57 m, đúng chuẩn Cyclist) |

Vòng hiệu chuẩn:
Trong vòng hiệu chuẩn với task chung `D13-HC485087`, khi đối chiếu cuboid xe con, nhóm nhận thấy sự sai lệch chủ yếu nằm ở việc xác định chiều dài phần đầu/đuôi xe bị che khuất và độ cao đáy hộp bị chìm dưới mặt đất. Cả lớp đã thống nhất: luôn căn đáy hộp chạm mặt đường tại điểm tiếp xúc bánh xe (z đáy ≈ -1,7 m đến -1,9 m); đồng thời căn góc yaw dọc thân xe và chỉ về đầu xe dựa vào hình ảnh camera.
Về các vật bỏ qua vì không đủ cơ sở: Tại frame `s01f41` có 1 cụm điểm thưa ở x ≈ 32 m, y ≈ -42 m (<15 điểm) nghi là xe con nhưng camera bị mờ, phân loại `UNCERTAIN_SCOPE` / `LOW_VISIBILITY` nên đã bỏ qua không gán nhãn theo đúng guideline.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Ảnh minh chứng: `t4-1-bong-che.png`, `t4-2-thua-o-xa.png`, `t4-3-phan-xa-thap.png`, `t4-4-diem-la.png`.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `t4-1-bong-che.png` (frame `s02f09`) | Tia LiDAR bị thân xe buýt/xe tải phía trước chắn hoàn toàn, không thể truyền ra phía sau | Xuất hiện một khoảng trống hình nan quạt không có điểm LiDAR nào ngay phía sau lưng vật thể che | Bật cờ `occluded`, bao trọn phần nhìn thấy và phần suy luận hợp lý theo kích thước chuẩn bảng 4.3 |
| 2. Thưa ở xa | `t4-2-thua-o-xa.png` (frame `s05f44`, idx 12) | Mật độ góc quét của các tia LiDAR cố định, càng ra xa thì khoảng cách giữa các tia càng mở rộng | Vật thể ở cự ly >40 m chỉ nhận được vài chục điểm rải rác, không rõ biên dạng | Chỉ gán nhãn nếu thấy được ≥15-20% thể tích và có ảnh camera đối chiếu; nếu không đủ cơ sở thì bỏ qua |
| 3. Phản xạ thấp / ít điểm bất thường | `t4-3-phan-xa-thap.png` (frame `s01f07`) | Bề mặt sơn xe tối màu hoặc kính hấp thụ/tán xạ tia LiDAR ra hướng khác thay vì phản xạ về máy thu | Vật thể ở cự ly gần nhưng số điểm LiDAR thu được ít hơn đáng kể so với xe màu sáng bên cạnh | Cần kết hợp quan sát ảnh camera để xác định hình dáng xe, không thu hẹp kích thước cuboid theo cụm điểm thiếu hụt |
| 4. Điểm lạ | `t4-4-diem-la.png` (frame `s02f35` task chung) | Nhiễu đa phản xạ (multipath reflection) từ mặt đường ướt/gương phản chiếu, hoặc bụi/hơi nước | Cụm điểm trôi nổi lơ lửng trên không trung hoặc nằm sâu dưới lòng đất (z < -3 m) | Không gán nhãn cuboid cho các điểm dị thường này |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603265-bangA.csv` và `submission/D13-2A202603265-tukiem.csv`:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f07`, idx 2, Car | -1,053 | 1,739 | -1,923 | 3,76 | 3,90 | 267,4° | Đáy chạm đường chuẩn (z đáy ≈ -1,92 m) |
| `s02f09`, idx 6, Bus | -0,350 | 3,003 | -1,851 | 10,88 | 10,88 | 85,0° | Đáy chạm mặt đường phẳng (z đáy ≈ -1,85 m) |
| `s03f52`, idx 4, Cyclist | -0,972 | 1.572 | -1,758 | 9,63 | 9,68 | 271,2° | Đáy chạm mặt đường chuẩn (z đáy ≈ -1,76 m) |

(a) Cuboid gần nhất và xa nhất:
- Cuboid gần nhất: `s01f07#2` (Car), `dist_m = 3,76 m`, `r = 3,90 m`.
- Cuboid xa nhất: `s05f44#12` (Car), `dist_m = 48,19 m`, `r = 48,22 m`.

(b) Kích thước l, w, h:
- Người đi xe hai bánh (`s03f52#4`, Cyclist): dài 1,57 m, rộng 0,68 m, cao 1,57 m. So với Bảng 4.3 (xe đạp có người: 1,6–1,9 m; xe máy: 1,8–2,3 m), chiều dài 1,57 m hơi ngắn hơn một chút do người lái hơi khom và bánh trước bị che khuất một phần điểm.
- Xe con (`s01f07#2`, Car): dài 4,99 m, rộng 2,21 m, cao 1,74 m. Kích thước nằm hoàn toàn trong khoảng chuẩn của dòng SUV/Crossover theo Bảng 4.3.

(c) Góc `yaw_deg`:
- Xe cùng chiều phía trước (`s01f07#1`): `yaw_deg = 271,1°`, rất sát góc 270° (hướng -y, thẳng về phía trước theo hướng xe chạy).
- Xe ngược chiều (`s01f07#5`): `yaw_deg = 91,7°`, rất sát góc 90° (hướng +y, đối diện lao tới).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f41`, idx 1, Car | 5,01 m | 2700 |
| `s03f52`, idx 3, Car | 15,01 m | 710 |
| `s01f41`, idx 6, Car | 25,11 m | 180 |
| `s02f42`, idx 5, Car | 39,52 m | 109 |

Quy luật mật độ:
Mật độ điểm giảm xấp xỉ theo bình phương khoảng cách: khi khoảng cách tăng gấp ~3 lần (từ 5 m lên 15 m), số điểm thu được giảm gần 4 lần (từ 2700 xuống 710 điểm); khi khoảng cách tăng lên ~40 m (gấp 8 lần), số điểm giảm chỉ còn 109 điểm (giảm ~25 lần). Kết quả này hoàn toàn tương đồng với đường cong lý thuyết trên slide bài giảng (xe con: khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m).

Phần tự kiểm:
- Hai cuboid đã sửa sau khi đối chiếu `preview` / `selfcheck`:
  1. `s01f07#2`: Ban đầu đáy hộp bị đặt hơi thấp (z đáy = -2,15 m, chìm xuống lòng đất), sau khi chạy `selfcheck` có cảnh báo khoảng hở mặt đất, tôi đã nâng đáy lên chạm mặt đường tại z đáy = -1,92 m.
  2. `s04f01#3`: Góc yaw ban đầu bị ngược 180° do nhìn nhầm đuôi xe trên ảnh camera, sau khi chạy `preview` đối chiếu ảnh chiếu camera đã xoay lại 180° để vạch đỏ chỉ đúng đầu xe.
- Cảnh báo giữ nguyên: Cuboid `s01f41#6` (dài 1,83 m, rộng 1,90 m) bị cảnh báo length < width do xe bị che khuất chéo chỉ nhìn thấy cụm điểm phần đuôi và hông sau, tôi giữ nguyên kích thước bám sát cụm điểm quan sát được.

Đổi kiểm:
Đã đổi kiểm chéo với bạn có MSSV 2A202600128 trên frame `s01f07`. Đối chiếu 2 số đo: kích thước xe `s01f07#2` của hai bên lệch nhau dưới 0,15 m (nằm trong dung sai cho phép ±5%); góc yaw lệch nhau ~3° (hoàn toàn khớp về hướng di chuyển).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Ý nghĩa từ góc nhìn Model:
Nhãn 3D cần được đo bằng mét trong hệ tọa độ thế giới thực thay vì pixel trên ảnh 2D vì các mô hình tự hành cần biết chính xác khoảng cách vật lý và kích thước không gian để lập kế hoạch quỹ đạo di chuyển và tính toán khoảng cách phanh an toàn. Vật thưa điểm cần có quy tắc chuẩn riêng (như Bảng kích thước 4.3) nhằm ngăn ngừa mô hình học theo các cụm điểm nhiễu ngẫu nhiên, giúp mô hình suy luận được hình dạng chuẩn xác của vật thể ngay cả khi chỉ nhận được rất ít điểm phản xạ.

(b) Cách dùng AI:
1. Công cụ: Antigravity AI Assistant.
2. Câu hỏi chính: Cách tinh chỉnh hiển thị LiDAR trên CVAT (brightness, contrast), phân tích quy luật mật độ điểm theo khoảng cách và giải thích các cờ cảnh báo của script `selfcheck`.
3. Cách sử dụng kết quả: Áp dụng các mẹo giao diện để thao tác gán nhãn nhanh hơn, kiểm tra tính nhất quán của Bảng A trước khi nộp bài.

(c) Điều còn chưa chắc:
Đối với một số vật thể ở cự ly ranh giới 45–50 m, số lượng điểm LiDAR thu được rất ít (<20 điểm) và hình ảnh camera ở góc xa bị mờ do độ phân giải, việc xác định chính xác chiều dài thực tế của xe vẫn còn mang tính chất ước lượng theo bảng kích thước tham chiếu.
