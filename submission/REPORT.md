# Báo cáo Day 08 — 2A202603273 - Nguyễn Văn Nhật

> Mọi con số kèm **mẫu số** và nguồn (file nào). Bản nháp do Claude soạn từ số liệu thật trong repo, học viên đã đọc và chỉnh.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / bus / truck | YOLO26**s** (máy chỉ có CPU, không tải `yolo26m`) | 0.3 (mặc định) | r01 vs r02: nâng 0.25 → 0.5 làm số box trên 8 ảnh pilot giảm 195 → 131 (person 111 → 79, motorbike 54 → 35), tức là mất nhiều vật thật. `threshold_sweep`: person và motorbike không đạt recall 90% ở ngưỡng nào ≤ 0.5 (tối đa 88% / 16 vật và 89% / 9 vật) nên giữ mặc định; bicycle (4), bus (1), truck (0 vật gold) quá ít mẫu nên không chọn ngưỡng riêng |
| car | YOLO26s | 0.45 | `threshold_sweep`: recall 100%, precision 73% (8 vật gold) |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r03: YOLO26 không có lớp này (COCO gọi tuk-tuk là car/truck); `threshold_sweep`: recall 100%, precision 78% (7 vật gold) |

- Lệnh `merge_prelabels.py` đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.45,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 521 box (batch + gold).
- **Phiên bản SAM:** SAM 2.1 base (`sam2.1_b.pt`). Lý do: chất lượng mask tốt hơn `sam2.1_t` / `mobile_sam` mà vẫn chạy được trên CPU (~6.9 s/ảnh ở r04, chấp nhận được vì chỉ dùng cho vài vật). SAM 3 cần GPU + quyền truy cập nên không dùng.
- **Lỗi không sửa được bằng đổi ngưỡng:** YOLO26 là detector danh mục cố định (COCO), không có lớp xe ba bánh, nên tuk-tuk luôn ra `car`/`truck` dù ngưỡng nào. Sửa bằng cách lấy `three_wheeler` từ YOLOE (open-vocabulary) và để YOLOE được ưu tiên khi gộp (file YOLOE đứng trước, `--iou 0.6`). Dấu vết còn lại sau gộp: img_013 three_wheeler→car, img_034 three_wheeler→truck, img_024 truck→car ×2 (CAL-02).
- **Điểm thấp mà vẫn giữ / điểm thấp mà sai:** ngưỡng three_wheeler 0.15 rất thấp, nên YOLOE đề xuất nhiều box sai: 25 box three_wheeler bị xoá, 9 accepted, 9 edited (review_stats, theo lớp). Ví dụ img_003 (xoá 5 box three_wheeler), img_010, img_014, img_023 (CAL-01).
- **Giới hạn:** ngưỡng được chọn và chấm trên **cùng 4 ảnh gold** (45–47 vật), nên precision/recall của máy trên gold là con số **lạc quan** (in-sample); 4 lớp không đủ vật để chọn ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

Đã loại 24 đối tượng quanh các bẫy khỏi bảng (lỗi nhân tạo không phản ánh máy).

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 334 | 75.4% | 443 box máy |
| edited | 46 | 10.4% | 443 box máy |
| deleted | 63 | 14.2% | 443 box máy |
| added | 36 | 8.7% | 416 đối tượng sau duyệt |

- **Lớp máy tệ nhất: `three_wheeler`**: 25/43 box bị xoá (58%), 9/43 bị sửa; chỉ 9/43 accepted. Nguyên nhân: ngưỡng YOLOE 0.15 + prompt "cargo tricycle" kéo theo quầy hàng / xe máy chở hàng (img_003, img_010, img_014). **`motorbike`** đứng thứ hai: 23 deleted và 20 added trên 121 box máy; máy sót xe máy nhỏ / bị che (img_009 thêm 3, img_018 thêm 3).
- 9 trong số deleted + added thật ra là **đổi lớp**, nên số vật máy sót thật ≈ 36 − 9 = 27.
- **So với gold:** precision của máy 38/62 = **61.3%** (KTC 95%: 49–72%), recall 38/45 = **84.4%** (KTC 95%: 71–92%), mẫu số 45 đối tượng gold trên 4 ảnh. Recall dưới mục tiêu 90% → `accepted` 75% **không** có nghĩa máy tốt: máy vẫn sót khoảng 1/6 vật.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy: 6/12 = 50%** (KTC 95%: 25–75%), dưới ngưỡng đạt 80% (≥ 10/12).
  - loose 1/3 · wrong_class 2/3 · missing 1/3 · extra 2/3.
  - Lọt: img_009 và img_024 (loose), img_010 (wrong_class), img_013 và img_022 (missing), img_026 (extra).
- **Vì sao lọt:** bẫy *loose* và *missing* lọt nhiều nhất (4/6 vật lọt). Hai loại này chỉ lộ ra nếu (a) **ẩn nhãn máy và quét ảnh trước** (bước 0 → bắt *missing*) và (b) **soi từng box** ở bước 4 (*loose*). Không
- **Ý nghĩa:** accepted 75% mà chỉ bắt 50% bẫy nghĩa là phần lớn box được giữ vì *trông hợp lý*, không vì đã kiểm. Đây đúng là thiên kiến tự động hóa: lỗi không hiện trên điểm số (box lệch vài chục px, vật bị bỏ sót) là lỗi bị bỏ qua.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 30 phút / 4 ảnh = **7.5 phút/ảnh** vs duyệt có pre-label 60 phút / 24 ảnh = **2.5 phút/ảnh** (nhanh gấp 3). Thiên lệch: gold gán trước nên lúc duyệt đã quen cảnh / quy tắc → thời gian duyệt hơi lạc quan.
- **Kết luận (nháp):** **có, nhưng chỉ cho person / car / bus** (accepted cao: person 208/230, car 33/37). **Không tin** `three_wheeler` (58% bị xoá) và `motorbike` (nhiều sót) mà phải kiểm từng box. Nhưng tốc độ 2.5 phút/ảnh đi kèm bắt bẫy 50% và QC 5/6 ảnh còn lỗi: phần thời gian tiết kiệm được một phần là do **duyệt quá nhanh**, nên con số 3× là lạc quan. Thời gian thật cho chất lượng Release cần cộng thêm vòng Rework.
- **Ba quy tắc giảm thiên kiến (kiểm tra được):**
  1. **Ẩn nhãn máy 10 giây đầu mỗi ảnh** và đếm nhẩm số vật trước; sau khi bật nhãn, nếu số box lệch > 2 so với số đếm thì duyệt lại ảnh. Đo: tỉ lệ bắt bẫy `missing` phải ≥ 2/3 ở lô sau.
  2. **Mọi box `three_wheeler` và `motorbike` phải được bấm chọn từng cái** trong danh sách object (không duyệt lướt cả ảnh). Đo: `edited` của hai lớp này ≥ số bẫy `loose` cài vào; bắt bẫy `loose` ≥ 2/3.
  3. **Giới hạn tốc độ:** không duyệt nhanh hơn ~2 phút/ảnh với ảnh > 15 box; nghỉ 5 phút sau mỗi 8 ảnh. Đo: cột `minutes` trong `time_log.csv` / số ảnh, và tỉ lệ bắt bẫy ở nửa sau lô không thấp hơn nửa đầu.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk gần camera) | SAM 2.1 b | 3 (2 dương thân + mui, 1 âm lên tài xế) | **Có**: mask vẫn trùm tài xế và cả khách nhìn qua thành hở phía sau, dù có click âm | Nuốt người + một phần bàn ghế nền nhìn xuyên qua khung → cần nhiều click âm hơn | `tuktuk_img013.jpg` |
| img_017 (tuk-tuk bên phải, rõ) | SAM 2.1 b | 4 (3 dương: mui, ghế, bánh trước; 1 âm: bàn nền) | Không (xe trống) | Mask sạch, bám biên tốt; chỉ thiếu một ít mép mui | `tuktuk_front_img017.jpg` |
| img_017 (tuk-tuk giữa, bị người đi bộ che) | SAM 2.1 b | 4 (2 dương, 2 âm: người phụ nữ + tuk-tuk bên cạnh) | Không gồm người đi bộ | **Tràn sang tuk-tuk phía sau** (bánh đỏ bên trái) vì hai xe cùng màu, chạm nhau | `tuktuk_occluded_img017.jpg` |

- **Đối chiếu guideline 4.2:** quy tắc 4.2.4 (hai vật chạm nhau, mỗi vật một polygon, không chồng) giúp nhận ra lỗi tràn ở img_017; 4.2.2 (lỗ thủng) giải thích vì sao SAM nuốt nền nhìn xuyên khung tuk-tuk. **Thiếu:** v1.0 không nói người ngồi trong xe có thuộc mask xe không → đã chốt ở v1.1 (CAL-05: không gồm người).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: thử rồi

## 6. Calibration (Part 6)

- Số quyết định: **5** (`calibration_decisions.csv`: 2 `loi_may`, 3 `cau_hoi_mo`). Quan trọng nhất: **CAL-01**: box `three_wheeler` của YOLOE phải kiểm từng cái, chỉ giữ khi thấy kết cấu ba bánh (lớp này có 58% box bị xoá).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu 1 (xe đạp điện), câu 2 (khối xe máy dày / < 20 px), câu 4 (người trong xe ba bánh, mask không gồm người); thêm quy tắc "đếm bánh trước" ở 4.3. Câu 3, 5, 6, 7 chưa chốt vì không đủ ca.
- Nguyên nhân gốc: CAL-01, CAL-02 là **lỗi máy** (YOLOE ngưỡng thấp, YOLO26 không có lớp); CAL-03, CAL-05 là **guideline mơ hồ** (v1.0 để mở); CAL-04 là **lỗi của tôi** khi duyệt (còn box < 20 px, không dùng `ignore_region`), dù v1.0 mục 4.1.6 đã có quy tắc.

## 7. QC và quyết định (Part 6)

- Mẫu: **6 ảnh** (1 ảnh `can_review` img_025 + 5 ngẫu nhiên, seed 42): img_003, img_006, img_013, img_014, img_025, img_033; 93 đối tượng. Người kiểm: **Claude (AI)**, không phải người duyệt lô (**hạn chế**: AI có thể sót cùng loại lỗi và chỉ nhìn ảnh overlay, không mở CVAT).
- Lỗi theo **đối tượng**: 7/93 = **7.5%** · theo **ảnh**: 5/6 = **83.3%** (ngưỡng Release ≤ 2%). Nguồn: `qc/qc_review.csv`, `qc/qc_report_auto.md`.
- Top lỗi: **bỏ sót** 3 (img_006 hàng xe máy đỗ; img_013 xe van trắng và một xe máy) · **sai lớp** 2 (img_013 xe buýt gán car; img_025 xe tay ga gán bicycle) · **sai vị trí/kích thước** 1 (img_003 box ba gác lỏng) · **thừa** 1 (img_014 kệ gỗ gán three_wheeler). Lỗi lặp lại đúng các lớp đã thấy ở mục 2 (motorbike sót, three_wheeler thừa).
- **Giới hạn mẫu nhỏ:** 5/6 ảnh lỗi → cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **97%**; khoảng tin cậy rất rộng. Với 6 ảnh, kể cả 0 lỗi cũng không chứng minh được tỉ lệ lỗi ảnh ≤ 2%.
- Script kiểm được: box < 20 px, ảnh thiếu, lớp lạ. **Không** kiểm được: box ôm sát hay không, vật mà cả máy lẫn tôi cùng bỏ sót, tôi có làm đúng checklist không.

**Quyết định: Rework**, ngày 2026-11-19. Lý do theo cây: **Hold?** Không: guideline đã được chốt thêm ở v1.1, dữ liệu không hỏng. **Rework?** Có: ảnh lỗi 5/6 = 83% > 2%, lỗi lặp lại theo lớp (motorbike sót, three_wheeler thừa / sai lớp), và bắt bẫy 6/12 = 50% < 80% → lô duyệt chưa đáng tin. Không đủ điều kiện **Release**.
Nếu Rework / Hold, việc cần làm tiếp: duyệt lại img_003, img_009, img_010, img_013, img_022, img_024, img_026 theo guideline v1.1; xoá box < 20 px (img_018, img_023, img_027) và thay bằng `ignore_region`; chạy lại `review_stats` + QC.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Claude Code (Claude Opus) dùng để: đọc đề và lập kế hoạch; chạy các script (`prelabel`, `threshold_sweep`, `merge_prelabels`, `inject_traps`, `review_stats`, `sam2_points`, `build_release`, `qc_sample`); tạo Project / Task trên CVAT local qua API; chọn điểm click SAM; **soạn nháp** `calibration_decisions.csv`, `guideline_v1.1` và báo cáo này từ số liệu `review_stats`. **Gán gold và duyệt batch do tôi tự làm trên CVAT.** **QC 6 ảnh (`qc_review.csv`) do Claude xem ảnh `work/qc/review/` và điền, cột `reviewer` ghi rõ "Claude (AI)"** (theo yêu cầu của tôi) → mục 7 là đánh giá của AI, không phải của tôi. Kiểm tra: Tôi mở lại các img để xem.
- Gold có bị nhiễm không? hông. Trước khi gán gold chỉ chạy model trên 8 ảnh pilot (r01–r03) và img_004, img_013 (r04, ảnh batch), không có ảnh gold nào. Các lần chạy --split all (r05, r06) có cả 4 ảnh gold nhưng chạy sau khi gold_manual.json đã export, với --no-viz. Trước khi gán xong, tôi chỉ hỏi AI ảnh nào là gold và cần vẽ box hay mask, không hỏi nội dung nhãn.
- Máy / GPU / CVAT: Windows 11, chỉ CPU (torch 2.15 cpu), CVAT Docker v2.74.1. YOLO26**s** thay cho YOLO26m vì chưa tải trọng số m.
- Dùng cache `data/cache/`? **Không**, tự chạy toàn bộ.
- Lỗi / bug script đã gặp: không có lỗi script; chỉ gặp phiên đăng nhập CVAT hết hạn khi tạo task.
- Một điều làm khác lần sau: Cố gắng tìm nhiều bug trong ảnh hơn
