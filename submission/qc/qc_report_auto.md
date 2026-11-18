# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 48 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 4/48 = 8.3% | 48 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 3/5 = 60.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 241/298 = 80.9% / 23/298 = 7.7% / 34/298 = 11.4% | 298 box máy |
| Máy: added | 11/275 = 4.0% | 275 đối tượng sau duyệt |
| Bắt lỗi bẫy | 7/12 = 58.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 2 | img_008.jpg, img_018.jpg |
| thua | 1 | img_008.jpg |
| sai_vi_tri_kich_thuoc | 1 | img_031.jpg |

## Giới hạn thống kê
- 3 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **88%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 60.0% > 2%
- bắt bẫy 58% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
