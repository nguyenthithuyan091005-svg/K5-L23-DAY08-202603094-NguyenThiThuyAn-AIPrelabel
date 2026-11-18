# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 641; đối tượng sau duyệt: 560; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 410 | 410/641 = 64.0% | 641 box máy |
| edited (sửa) | 103 | 103/641 = 16.1% | 641 box máy |
| deleted (xoá) | 116 | 116/641 = 18.1% | 641 box máy |
| added (vẽ thêm) | 47 | 47/560 = 8.4% | 560 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 37.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 262 | 49 | 58 | 11 |
| bicycle | 5 | 1 | 2 | 2 |
| motorbike | 94 | 39 | 17 | 22 |
| car | 40 | 2 | 10 | 1 |
| bus | 3 | 0 | 0 | 4 |
| truck | 2 | 0 | 3 | 2 |
| three_wheeler | 4 | 12 | 26 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 40 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 36 | 27 | 4 | 57.1% (KTC 95%: 45–69%) | 90.0% (KTC 95%: 77–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 1/3
  - extra: 3/3
  - missing: 2/3
  - wrong_class: 1/3
  - lọt: img_005.jpg (loose), img_034.jpg (wrong_class), img_008.jpg (wrong_class), img_006.jpg (loose), img_029.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
