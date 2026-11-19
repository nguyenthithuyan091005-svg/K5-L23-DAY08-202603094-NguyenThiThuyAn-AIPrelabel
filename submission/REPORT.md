# Báo cáo Day 08 — 202603094 - Nguyễn Thị Thúy An

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | person 0.15, motorbike 0.25; bicycle, car, bus, truck dùng 0.3 (mặc định) | Tôi chạy YOLO26 một lần ở conf 0.05 cho cả 36 ảnh (r05), rồi lọc theo từng lớp. Ngưỡng cao 0.5 làm mất người và xe máy nhỏ ở xa (r02: img_019 motorbike từ 7 còn 2). Bicycle chỉ có 4 vật trong gold, bus và truck không có vật nào, còn car không ngưỡng nào đạt recall 90% (cao nhất 83%, 6 vật), nên 4 lớp này để mặc định. |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: tuk-tuk, cyclo, cargo tricycle | 0.15 | YOLO26 không có lớp xe 3 bánh nên hay gọi là car/truck (r01). YOLOE tìm bằng chữ thì thấy được (r03: img_002 điểm 0.93, img_032 cả 3 box đều đúng; r06: 125 box trên 36 ảnh). Trên gold: recall 100%, precision 56% (5 vật). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  - Trả lời: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,motorbike=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: 
  - Trả lời: Tôi chọn SAM 2.1 (`sam2.1_b.pt`). Tôi mới chạy bản này (r04): mask bám khá sát viền xe máy ở img_013 và chạy được trên Mac (MPS), khoảng 8 giây một ảnh. Tôi chưa thử SAM 1 để so sánh. Tôi không dùng SAM 3 vì phải xin quyền truy cập, mà máy tôi chưa đăng nhập HuggingFace. Máy tôi có 8 GB RAM và chạy được bản `sam2.1_b.pt` này.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? 
  - Trả lời: Xe 3 bánh bị máy gọi là car hoặc truck (r01: img_011 bị gọi truck 0.77; r04: img_004 hai box car 0.37 và 0.34, img_013 box car 0.42, đều là tuk-tuk). Máy gọi sai tên chứ không phải thiếu tự tin, nên đổi ngưỡng không giúp được. Tôi sửa bằng cách chạy thêm YOLOE (gõ chữ three_wheeler) rồi gộp vào, và khi review thì tự đổi lại tên lớp cho đúng.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Trả lời: Điểm cao mà sai: img_011 truck 0.77 (thật ra là xe 3 bánh, r01). Điểm vừa mà sai lớp: img_018 three_wheeler 0.53 (thật ra là xe minibus, tôi đổi thành bus) và img_024 truck 0.53 (xe van chở khách, tôi đổi thành bus). Điểm thấp mà đúng: img_032, YOLOE cho 3 box 0.92 / 0.48 / 0.27 và cả 3 đều đúng three_wheeler (r03). Nếu đặt ngưỡng cao thì sẽ mất các box này.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan:
  - Trả lời: Tôi chọn ngưỡng và chấm điểm trên cùng 4 ảnh gold, nên máy trông tốt hơn thực tế. Lớp ít vật (bicycle 4, bus 0, truck 0, three_wheeler 5) thì ngưỡng chỉ là ước lượng sơ sơ.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 410 | 64.0% | 641 box máy |
| edited | 103 | 16.1% | 641 box máy |
| deleted | 116 | 18.1% | 641 box máy |
| added | 47 | 8.4% | 560 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? 
  - Trả lời: Tệ nhất là three_wheeler (xe 3 bánh): chỉ 4/42 box giữ nguyên (khoảng 10%), 12 box phải sửa, 26 box bị xoá, 5 box phải vẽ thêm. Máy hay gán three_wheeler nhầm cho xe lớn hơn (img_003: 2 box tôi đổi thành truck, 4 box xoá; img_013: 2 box đổi thành bus; img_014: 3 box xoá). Các lớp khác khá hơn: person 262/369 (≈ 71%), car 40/52 (≈ 77%), motorbike 94/150 (≈ 63%, có 39 box phải sửa, nhiều ở ảnh đông xe máy). Ghi chú: tôi bỏ 23 đối tượng quanh các bẫy ra khỏi bảng vì đó là lỗi cố ý cài vào. Trong 47 box vẽ thêm có 10 cái chỉ là sửa tên lớp, nên máy sót thật khoảng 37 vật.
- So với gold: 
  - Trả lời: Precision 57.1% (khoảng tin cậy 45–69%), recall 90.0% (77–96%); mẫu số là 40 đối tượng gold (máy tìm đúng 36, thừa 27, sót 4). Nghĩa là máy ít bỏ sót nhưng đưa ra nhiều box thừa, vì ngưỡng tôi chọn thấp (person 0.15, three_wheeler 0.15). Số này đẹp hơn thực tế vì tôi chọn ngưỡng và chấm điểm trên cùng 4 ảnh.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy (.../12); loại nào lọt; vì sao (bạn đã bỏ bước nào của checklist)?
  - Bắt được: 7/12 = 58.3% (khoảng tin cậy 32–81%, thấp hơn mức 80%). Chỉ có 12 bẫy nên con số này không chắc chắn lắm.
  - Loại lọt: box lệch (loose) lọt 2/3: img_005 box person bị dời xuống 37–91 px, img_006 box motorbike nhỏ 51×61 px lệch khoảng 9–16 px. Sai lớp (wrong_class) lọt 2/3: img_008 một person rất nhỏ 16×29 px bị gán bicycle, tôi giữ nguyên; img_034 three_wheeler bị gán motorbike, tôi thấy sai và đổi thành truck, nhưng đáp án là three_wheeler nên bị tính lọt. Thiếu box (missing) lọt 1/3: img_029 thiếu một person sát mép trái. Box thừa (extra) bắt đủ 3/3.
  - Vì sao: Tôi không ghi lại từng bước khi duyệt nên không dám nói mình đã bỏ bước nào. Chỉ biết các ca lọt rơi vào bước 4 (box lỏng, 2 ca), bước 3 (sai nhãn, 2 ca) và bước 1 (bỏ sót, 1 ca); 2 trong 5 ca nằm ở ảnh đông xe máy (img_005, img_029). Tôi duyệt ngắt quãng, không làm một mạch. Lúc duyệt tôi khá mệt, nhiều ca khó, và ảnh có rất nhiều vật nên phải nhìn kỹ từng cái, nên có thể tôi đã bỏ sót mấy lỗi nhỏ.
  - Sau khi chạy review_stats: tôi đã mở lại 5 ảnh bẫy lọt (img_005, img_006, img_008, img_029, img_034) và sửa trong qc_fixed.json (mục 7). Kết quả 7/12 giữ nguyên vì đó là số đo của lần duyệt đầu. Tôi không sửa batch_reviewed.json và không chạy lại review_stats, vì lúc đó tôi đã biết bẫy nằm ở đâu.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt?
  - Trả lời: Accepted 64.0% mà bắt bẫy chỉ 58.3%, nghĩa là giữ nguyên nhiều box chưa chắc là tôi đã kiểm kỹ. Box vô lý thì tôi bắt dễ, nhưng hay để lọt lỗi nhìn qua thấy bình thường (box lệch ít, vật rất nhỏ, người đứng sát mép). Điểm tin cậy của máy cũng không báo trước được mấy lỗi này. Chỉ có 12 bẫy nên kết luận này chưa chắc chắn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`), Thiên lệch (bạn gán gold trước nên đã quen cảnh):
  - Trả lời: Gán tay 5 phút/ảnh (20 phút cho 4 ảnh), duyệt 5 phút/ảnh (120 phút cho 24 ảnh). Hai số này tôi tự báo gần đúng chứ không bấm giờ chính xác. Tôi đã quen kiểu ảnh sau khi gán gold, nhưng ảnh batch lại đông hơn (có 6 ảnh nhiều xe máy), nên hai số khó so sánh công bằng.
- Kết luận:
  - Trả lời: Chỉ đáng dùng cho một vài lớp. Máy giữ nguyên được 64.0% box, nhưng tôi không nhanh hơn (5 phút/ảnh cả hai cách), bắt bẫy mới 58.3% (dưới 80%), và three_wheeler chỉ khoảng 10% box dùng được. Tôi sẽ dùng máy cho person và car (thêm motorbike nếu chịu sửa box). Với three_wheeler thì tự vẽ, hoặc chạy YOLOE rồi duyệt thật kỹ.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy, tự đếm vật trước rồi mới hiện nhãn máy lên. Kiểm tra bằng số bẫy thiếu box bắt được (hiện 2/3).
  2. Gặp vật nhỏ hoặc xe 3 bánh thì phóng to xem lại tên lớp, không chỉ nhìn cái khung. Kiểm tra bằng số bẫy sai lớp bắt được (hiện 1/3).
  3. Nghỉ sau mỗi khoảng 10 ảnh. Mục tiêu bắt bẫy ít nhất 10/12 mới coi là xong lô.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái) | SAM 2.1 (`sam2.1_b.pt`) | 13 (6 dương, 7 âm) | Không| Mask không phủ hết xe| `masks/tuktuk_img013.jpg` |
| img_008 (xe ba gác chở chuối) | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Có gồm chuối (hàng hoá)| Mask không phủ hết xe, có lỗ ở bánh xe | `masks/bagac_chuoi_img008.jpg` |
| img_031 (xe đạp chở hàng) | SAM 2.1 (`sam2.1_b.pt`) | 4 (3 dương, 1 âm) | Có gồm cả giỏ hàng| Bánh xe không mask hết, có chỗ có mask chỗ không | `masks/xedap_hang_img031.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? 
  - Trả lời: Mục 4.2.4 (mỗi vật một polygon, không chồng nhau, vùng chạm thuộc vật phía trước) giúp tôi quyết định mask xe không nên chứa người, vì người là vật riêng. Guideline còn thiếu một quy tắc nói rõ mask có gồm hàng hoá không: mask của img_008 gồm chuối và mask của img_031 gồm cả giỏ, tôi không biết là đúng hay sai (đây là câu hỏi mở số 3, phần "mask thì sao?"). Sau đó tôi đã chốt ở guideline v1.1 câu 3: mask theo cùng quy tắc với box. Mục 4.2.2 (lỗ nhỏ hơn khoảng 5% diện tích thì không vẽ lỗ) giúp tôi coi lỗ ở nan hoa bánh xe là chấp nhận được.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline:
  - Trả lời: Không làm được. SAM ở Part 5 tôi chạy offline bằng script trên máy (`sam2.1_b.pt`), còn trong CVAT thì tôi không có model SAM để dùng nút Interactor: CVAT trên máy tôi (chạy bằng Docker, phiên bản 2.82.1) chưa cài sẵn model AI, và các container CVAT đang chạy không có Nuclio (phần serverless cần để chạy model trong CVAT). Tôi cũng không cài thêm Nuclio và model SAM vào CVAT vì máy tôi chỉ có 8 GB RAM, Docker được cấp khoảng 3,8 GiB mà CVAT đã dùng khoảng 2,5 GiB, nên không còn đủ chỗ cho model SAM; tài liệu của lab cũng nói image Nuclio trên Mac chip Apple Silicon thường chạy rất chậm hoặc lỗi, và cài thêm có thể làm hỏng CVAT đang giữ Task của tôi. Tôi không thử bản CVAT online vì RULES.md không cho gửi ảnh lên dịch vụ AI đám mây khi chưa được giảng viên cho phép. Vì không dùng được nên tôi không có số liệu để so sánh với cách offline (`sam2_points.py`).

## 6. Calibration (Part 6)

- Số quyết định: 
  - Trả lời: 13 quyết định. Quan trọng nhất: CAL-05 (YOLOE hay gán three_wheeler nhầm cho xe lớn như xe tải, xe buýt; đây là lỗi lặp, làm lớp three_wheeler chỉ còn khoảng 10% box dùng được) và CAL-03 (xe van / minibus chở khách là bus, gặp ở 3 ảnh img_013, img_018, img_024).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  - Trả lời: Đã chốt cả 7 câu hỏi mở của v1.0 (xe điện, khối xe máy dày, hàng nhô ra, người trong xe có mui, kính xe, box máy chồng nhau, mức conf) và thêm 2 quy tắc mới: van / minibus chở khách là bus (CAL-03), xe lớn quá mờ dùng ignore_region (CAL-04). Hai ca còn lại (CAL-05, CAL-07) tôi chỉ ghi nhận, chưa thêm quy tắc.
- Nguyên nhân gốc: 
  - Trả lời: Guideline mơ hồ là chính: 9/13 quyết định (CAL-01, 02, 03, 04, 09, 10, 11, 12, 13). Máy sai: 2/13 (CAL-05, 06). Tôi sai khi duyệt (bẫy lọt): 2/13 (CAL-07, 08), mấy ca này không cần đổi guideline.

## 7. QC và quyết định (Part 6)

- Mẫu:
  - Trả lời: 8 ảnh (3 ảnh can_review là img_008, img_013, img_014 + 5 ảnh ngẫu nhiên là img_003, img_006, img_018, img_021, img_035, seed 42), gồm 231 đối tượng. Người kiểm là tôi, cũng là người đã duyệt lô này.
- Lỗi theo **đối tượng**:
  - Trả lời: theo đối tượng 2/231 = 0.9%; theo ảnh 2/8 = 25.0%. Ngưỡng Release là không quá 2%, nên theo ảnh là cao hơn nhiều.
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại:
  - Trả lời: Chỉ có một loại lỗi là sai lớp (sai_lop), 2 lỗi: img_003 box three_wheeler ở mép phải ảnh (không chắc, tôi tính là lỗi nghi ngờ) và img_008 box một person rất nhỏ ở góc trái trên bị gán bicycle. Các loại khác (bỏ sót, thừa, sai vị trí, sai thuộc tính, sai mask) là 0 trong mẫu này.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`):
  - Trả lời: 2 ảnh lỗi trong 8 ảnh mẫu thì cận trên 95% của tỉ lệ ảnh lỗi thật là 59%, nên mẫu này không thể chứng minh ngưỡng 2%. Một trong hai lỗi là ca không chắc. Trong 8 ảnh mẫu có 5 ảnh chứa bẫy (img_003, img_006, img_008, img_014, img_035) nên mẫu không hoàn toàn độc lập với phép đo bẫy, và 2 ảnh đông (img_018, img_021) tôi chỉ kiểm bằng mắt trên ảnh overlay nên có thể sót lỗi nhỏ.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật):
  - Trả lời: Script kiểm được số đếm, tỉ lệ lỗi theo đối tượng và theo ảnh và tỉ lệ bắt bẫy. Script không kiểm được box có ôm sát không, những vật mà cả tôi lẫn máy cùng bỏ sót, và việc tôi có duyệt đúng theo checklist hay không, vì tôi vừa là người duyệt vừa là người kiểm.

**Quyết định** (Release / Hold / Rework), ngày, lý do (đi theo Hold → Rework → Release):
  - Quyết định: Rework (ngày 2026-11-19)
    - Không phải Hold: 7 câu hỏi mở đã chốt trong guideline v1.1, dữ liệu không hỏng.
    - Là Rework vì: ảnh có lỗi 2/8 = 25.0% (ngưỡng 2%); bắt bẫy 7/12 = 58.3% (cần ít nhất 80%); máy lặp lỗi ở lớp three_wheeler.
Nếu Rework / Hold, việc cần làm tiếp:
  - Đã làm: sửa trên CVAT, export thành qc_fixed.json (13 ảnh), release còn 571 đối tượng. Gồm 5 bẫy lọt, xe trắng mờ ở img_013 thành ignore_region, và xoá 12 box dưới 20 px. Lưu ý: số đo review, bắt bẫy và QC vẫn tính trên nhãn trước khi sửa.
  - Còn phải làm: duyệt lại cả lô theo checklist và guideline v1.1, xem lại box three_wheeler ở mép phải img_003, QC lại với mẫu lớn hơn 8 ảnh.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không):
  - Trả lời: Tôi dùng Claude Code (trợ lý AI). Dùng để: kiểm tra môi trường và tải model, chạy các lệnh Part 2, hỏi cách hiểu guideline ở các ca khó lúc review (xe điện, hàng nhô ra, xe van, vùng ignore), tổng hợp số đo từ `review_stats.md`, và soạn nháp câu chữ báo cáo từ số liệu, nhật ký và quyết định của tôi trong quá trình làm bài. Cách kiểm tra: nhận xét trong `model_log.csv` tôi tự nhìn ảnh rồi sửa theo những gì tôi thấy, số liệu trong báo cáo được đối chiếu với `review_stats.md`, quyết định gán nhãn từng ca là tôi chọn. 
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)?
  - Trả lời: Không, tôi tự làm 100%
- Máy / GPU / CVAT dùng; vướng mắc cài đặt:
  - Trả lời: Máy Mac chip Apple M2 (8 nhân CPU), RAM 8 GB, macOS 26.4, không có GPU rời; chạy model trên GPU Apple (MPS). Phần mềm: Python 3.12.1, torch 2.15.0, ultralytics 8.4.215. Model dùng: YOLO26m (`yolo26m.pt`), YOLOE-26s (`yoloe-26s-seg.pt`), SAM 2.1 (`sam2.1_b.pt`). CVAT chạy bằng Docker 29.9.2, CVAT bản 2.82.1 (server và giao diện cùng bản), gồm 18 container, Docker được cấp 4 CPU và khoảng 3,8 GiB RAM. Vướng mắc: lúc đầu CVAT bị lag vì Docker chỉ có 4 CPU và khoảng 3,8 GiB RAM, mấy container phân tích (clickhouse, vector, grafana) dùng nhiều CPU; sau đó tôi thấy CVAT trả lời nhanh hơn.
- Dùng cache `data/cache/` thay vì tự chạy? 
  - Trả lời: Không. Tôi tự chạy YOLO26m và YOLOE trên cả 36 ảnh (r05, r06 trong `work/runs.csv`).
- Lỗi / bug script đã gặp (nếu có):
  - Trả lời: Chưa có lỗi script nào mà tôi ghi lại được.
- Một điều bạn làm khác lần sau:
  - Trả lời: Tôi sẽ bấm đồng hồ thật để ghi `time_log` (lần này tôi chỉ ước chừng 20 phút và 2 giờ), và ghi lại từng ảnh đã duyệt theo checklist để biết mình bỏ bước nào.
