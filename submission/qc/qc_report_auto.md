# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 6 ảnh (1 ca khó + 5 ngẫu nhiên), 93 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 7/93 = 7.5% | 93 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 5/6 = 83.3% | 6 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 334/443 = 75.4% / 46/443 = 10.4% / 63/443 = 14.2% | 443 box máy |
| Máy: added | 36/416 = 8.7% | 416 đối tượng sau duyệt |
| Bắt lỗi bẫy | 6/12 = 50.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 3 | img_006.jpg, img_013.jpg |
| sai_lop | 2 | img_013.jpg, img_025.jpg |
| sai_vi_tri_kich_thuoc | 1 | img_003.jpg |

## Giới hạn thống kê
- 5 ảnh lỗi trong 6 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **97%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 83.3% > 2%
- bắt bẫy 50% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
