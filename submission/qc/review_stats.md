# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 443; đối tượng sau duyệt: 416; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 334 | 334/443 = 75.4% | 443 box máy |
| edited (sửa) | 46 | 46/443 = 10.4% | 443 box máy |
| deleted (xoá) | 63 | 63/443 = 14.2% | 443 box máy |
| added (vẽ thêm) | 36 | 36/416 = 8.7% | 416 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 27.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 208 | 9 | 13 | 2 |
| bicycle | 3 | 2 | 0 | 5 |
| motorbike | 76 | 22 | 23 | 20 |
| car | 33 | 4 | 0 | 4 |
| bus | 3 | 0 | 0 | 0 |
| truck | 2 | 0 | 2 | 2 |
| three_wheeler | 9 | 9 | 25 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 45 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 38 | 24 | 7 | 61.3% (KTC 95%: 49–72%) | 84.4% (KTC 95%: 71–92%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 6/12 = 50.0% (KTC 95%: 25–75%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 1/3
  - wrong_class: 2/3
  - missing: 1/3
  - extra: 2/3
  - lọt: img_024.jpg (loose), img_010.jpg (wrong_class), img_009.jpg (loose), img_013.jpg (missing), img_026.jpg (extra), img_022.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
