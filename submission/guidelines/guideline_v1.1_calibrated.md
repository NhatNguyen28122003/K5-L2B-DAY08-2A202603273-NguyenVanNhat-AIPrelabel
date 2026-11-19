# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603273 - Nguyễn Văn Nhật

> Chỉ ghi phần thay đổi so với v1.0. Mỗi ca khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Có bàn đạp và khung xe đạp, không có sàn để chân → `bicycle`. Có sàn để chân / yếm xe → `motorbike`, kể cả xe chạy điện. | img_025, img_033 | CAL-03 |
| 2 | Khối xe máy đỗ dày: từng xe hay `ignore_region`? | Mỗi xe cao ≥ 20 px → gán từng xe (kể cả gần camera). Xe cao < 20 px → không vẽ box lẻ; cụm ≥ 5 xe nhỏ sát nhau → **một** `ignore_region`. | img_018, img_023, img_027 | CAL-04 |
| 4 | Người ngồi trong xe ba bánh có mui | Giữ ngưỡng "thấy ≥ 20%" của 4.1.2 cho box `person`. Mask `three_wheeler` **không** gồm người (tài xế, khách); khi dùng SAM phải click âm lên từng người. | img_013, img_017 | CAL-05 |
| 3, 5, 6, 7 | Hàng nhô ra, kính xe, box chồng ban đêm, conf tối thiểu | **Chưa chốt**: lô 24 ảnh không có đủ ca để quyết (chỉ 1 ảnh xe chở hàng là img_009); giữ nguyên v1.0. | — | — |

## 2. Ca khó mới (từ lỗi của máy, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003, img_010, img_014 | Quầy hàng, xe máy chở hàng, xe đẩy bị nhận là xe ba bánh | YOLOE (ngưỡng 0.15) đề xuất `three_wheeler`; 25 box bị xoá | Xoá; chỉ giữ khi thấy kết cấu ba bánh | 4.3: `three_wheeler` phải thấy 3 bánh **hoặc** mui + một bánh trước; box YOLOE luôn kiểm từng cái (CAL-01) |
| img_013, img_024, img_034 | Taxi / bán tải / xe khách nhỏ cạnh tuk-tuk | YOLO26 / YOLOE đổi qua lại `car` / `truck` / `three_wheeler` | Đếm bánh trước: 4 bánh không bao giờ là `three_wheeler`; `truck` chỉ khi có buồng lái + thùng rời | 4.3: thứ tự quyết lớp = số bánh → hình dạng thân (CAL-02) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 1, 2, 4 của mục 6; thêm quy tắc đếm bánh và kiểm từng box `three_wheeler` vào 4.3; mask xe không gồm người (4.2) | CAL-01 … CAL-05 |
