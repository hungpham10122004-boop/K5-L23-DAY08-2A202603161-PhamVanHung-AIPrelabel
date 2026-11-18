# Thông tin học viên

- Họ và tên: Phạm Văn Hùng
- MSHV: 2A202603161
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: https://github.com/hungpham10122004-boop
- Repo bài nộp (URL): https://github.com/hungpham10122004-boop/K5-L23-Day08-PhamVanHung-AI-Assisted-Labeling
- Commit chốt bài (hash): c6dd642
- Máy: Windows, không có GPU (chạy CPU)
- CVAT (Docker): 2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 241 / 23 / 34 (mẫu số: 298 box máy); added: 11 (mẫu số: 275 đối tượng)
- So với gold: precision / recall của **máy** 61.5% / 77.4% (mẫu số: 31 đối tượng gold, lạc quan vì chọn ngưỡng và chấm trên cùng 4 ảnh)
- Bắt bẫy: 7/12 = 58.3% (vòng 1, trước khi xem đáp án)
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 0.83 phút/ảnh (20 phút / 24 ảnh)
- Lỗi QC (vòng 3, vòng cuối): 100% theo ảnh (5/5), 11.6% theo đối tượng (8/69)
- **Quyết định:** Rework — lý do một dòng: ảnh có lỗi vượt ngưỡng 2% ở cả ba vòng QC, lỗi bỏ sót lặp lại (6/8 ở vòng 3), bắt bẫy 58.3% dưới 80%
