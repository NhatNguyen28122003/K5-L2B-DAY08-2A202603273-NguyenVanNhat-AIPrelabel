# Thông tin học viên

- Họ và tên: Nguyễn Văn Nhật
- MSHV: 2A202603273
- Lớp (L2 / L3 / khác): <<>>
- Tài khoản GitHub: NhatNguyen28122003
- Repo bài nộp (URL): https://github.com/NhatNguyen28122003/K5-L2B-DAY08-2A202603273-NguyenVanNhat-AIPrelabel
- Commit chốt bài (hash): <<>>
- Máy: Windows 11, không GPU (CPU only, torch 2.15 cpu)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 334 / 46 / 63 (75.4% / 10.4% / 14.2%, mẫu số: 443 box máy); added: 36 (8.7%, mẫu số: 416 đối tượng)
- So với gold: precision / recall của **máy** 38/62 = 61.3% / 38/45 = 84.4%
- Bắt bẫy: 6/12 = 50%
- Thời gian: gán tay 7.5 phút/ảnh (30/4); duyệt có pre-label 2.5 phút/ảnh (60/24)
- Lỗi QC: 83.3% theo ảnh (5/6), 7.5% theo đối tượng (7/93)
- **Quyết định:** Rework — lý do một dòng: 5/6 ảnh QC còn lỗi (> 2%) và bắt bẫy 50% < 80%, lỗi lặp ở motorbike / three_wheeler.
