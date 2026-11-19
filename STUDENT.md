# Thông tin học viên

- Họ và tên: Nguyễn Thị Thúy An
- MSHV: 202603094
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: nguyenthithuyan091005-svg
- Repo bài nộp (URL): https://github.com/nguyenthithuyan091005-svg/K5-L23-DAY08-202603094-NguyenThiThuyAn-AIPrelabel
- Commit chốt bài (hash): ec3a0e0
- Máy: Mac chip Apple M2 (8 nhân CPU), RAM 8 GB, macOS 26.4, GPU Apple (MPS)
- CVAT (Docker): v2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 410 / 103 / 116 (mẫu số: 641 box máy; tỉ lệ 64.0% / 16.1% / 18.1%); added: 47 (mẫu số: 560 đối tượng sau duyệt; 8.4%)
- So với gold: precision / recall của **máy** 57.1% / 90.0% (mẫu số: 40 đối tượng gold; KTC 95%: 45–69% / 77–96%)
- Bắt bẫy: 7/12 = 58.3%
- Thời gian: gán tay khoảng 3.8 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label khoảng 6.3 phút/ảnh (150 phút / 24 ảnh)
- Lỗi QC: 25.0% theo ảnh (2/8), 0.9% theo đối tượng (2/231)
- **Quyết định:** Rework — lý do một dòng: ảnh có lỗi 25.0% vượt ngưỡng 2%, bắt bẫy 58.3% dưới 80%, và máy có lỗi lặp ở three_wheeler
