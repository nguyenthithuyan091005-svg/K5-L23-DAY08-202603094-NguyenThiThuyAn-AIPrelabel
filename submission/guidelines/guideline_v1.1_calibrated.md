# Label contract — Phố châu Á — BỔ SUNG v1.1 — 202603094 - Nguyễn Thị Thúy An

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | `motorbike` khi xe không có bàn đạp và chạy hoàn toàn bằng động cơ. Xe có bàn đạp + động cơ trợ lực: chưa gặp trong batch, chưa chốt | img_025 (xe điện sát mép phải) | CAL-01 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? Mask thì sao? | Giữ quy tắc v1.0 mục 4.1.5 (hàng nhô ra khỏi xe từ khoảng 20% chiều dài xe trở lên thì không tính phần nhô), làm rõ thêm: hàng xếp trong thùng, giỏ hoặc khay gắn cố định tính vào xe; chiều dài xe đo từ mép ngoài bánh trước đến mép ngoài bánh sau, phần nhô đo từ mép bánh ra đầu hàng; bạt, vải phủ trên thùng tính vào xe nếu không buông quá ngưỡng 20%; mask theo cùng quy tắc với box | img_008, img_031 | CAL-02 |
| 2 | Khối xe máy dày đặc gần camera: từng xe hay `ignore_region`? | Vẽ từng xe khi cao từ 20 px trở lên và tách được khỏi xe bên cạnh (thấy bánh, đèn hoặc tay lái riêng); không đạt thì dùng `ignore_region` bao cả cụm (mốc từ 5 vật giữ như v1.0) | img_024, img_025 | CAL-09 |
| 4 | Người trong xe ba bánh có mui: ngưỡng "còn thấy ≥ 20%" có hợp lý không? | Đổi sang tiêu chí dễ kiểm hơn: gán `person` khi thấy rõ đầu hoặc vai-thân trên; chỉ thấy tay, chân, mảng áo thì không gán; không chắc thì gán lớp tin nhất và tick `can_review`. Nếu không áp dụng được thì quay lại mốc 20% của v1.0. Chưa đo được độ lệch giữa hai người đánh giá vì chỉ có một người gán nhãn | img_013 | CAL-10 |
| 5 | Kính chắn gió, cửa sổ xe: thuộc mask xe hay không? | Thuộc xe (mask xe gồm kính). Hình phản chiếu trên kính không phải `person`: chỉ gán vật và người thật. Người thật nhìn thấy trực tiếp vẫn gán `person` theo quy tắc ở câu 4 | img_013 (kính chắn gió tuk-tuk) | CAL-13 |
| 6 | Box máy chồng nhau quá nhiều (ảnh đêm): gộp hay giữ? | Giữ riêng từng xe khi còn nhận ra hai xe; IoU giữa hai box từ 0.5 trở lên chỉ là dấu hiệu phải xem kỹ. Chỉ gộp thành một box khi không tách được là hai xe; từ 3 xe không tách được thì dùng `ignore_region`. Trong batch chỉ có 1 cặp motorbike IoU từ 0.5 trở lên (img_026, IoU 0.57), tôi giữ riêng hai box | img_026 | CAL-11 |
| 7 | Mức conf tối thiểu để chấp nhận box máy không cần sửa? | Mốc 0.5 chỉ để ưu tiên: dưới 0.5 xem kỹ từng box, từ 0.5 xem lướt nhưng vẫn kiểm theo checklist; không tự chấp nhận theo conf (từ 0.5 trở lên vẫn có 31/155 = 20% box phải sửa hoặc xoá; giữ nguyên 124/155 = 80% so với 76/151 = 50% ở dưới 0.5) | số liệu `qc/review_stats.md` | CAL-12 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_018, img_024 | Xe van / minibus chở khách (CAL-03) | img_018: YOLOE gán `three_wheeler` 0.53; img_024: YOLO26 gán `truck` 0.53 | `bus` | Mục 2 (`bus`): xe van hoặc minibus chở khách (thân dài, một hàng cửa sổ dọc hông, chở nhiều người) gán `bus`, không gán `car` hay `truck`. Không chắc thì chọn lớp tin nhất và tick `can_review` (mục 4.3) |
| img_013 | Một xe trắng lớn bị mờ, bị che (CAL-04) |  | `ignore_region` | Mục 2 (`ignore_region`) và mục 4.1.6: ngoài cụm vật nhỏ, được dùng `ignore_region` cho một xe lớn khi xe quá mờ hoặc bị che đến mức không xác định được lớp và ranh giới xe. Không dùng cho vật nhìn rõ chỉ vì khó vẽ |
| img_003, img_013, img_018 | Xe lớn bị gán nhầm `three_wheeler` (CAL-05) | YOLOE gán `three_wheeler` cho xe tải / xe buýt | đổi lại `truck` / `bus` |  |
| img_034 | `truck` hay `three_wheeler` khi xe che bạt có hình dáng giống xe tải (CAL-07, bẫy lọt) | bẫy gán `motorbike` | `truck` |  |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt 7 câu hỏi mở của v1.0: xe điện (câu 1), khối xe máy dày (2), hàng nhô ra (3), người trong xe có mui (4), kính xe (5), box máy chồng nhau (6), mức `conf` (7). Thêm quy tắc cho 2 ca khó mới: van/minibus chở khách là `bus`, xe lớn bị mờ dùng `ignore_region`. Ghi nhận thêm 2 ca (xe lớn bị máy gán nhầm `three_wheeler`, `truck` hay `three_wheeler` khi xe che bạt có hình dáng giống xe tải), chưa thêm quy tắc | CAL-01, 02, 09, 10, 13, 11, 12 (câu hỏi mở 1–7); CAL-03, 04 (quy tắc mới); CAL-05, 07 (ghi nhận) |
