# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603161 - Phạm Văn Hùng

> Chỉ gồm phần thay đổi so với v1.0. Mỗi ca khó có ảnh ví dụ (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Xe máy dày đặc: từng xe hay `ignore_region`, ngưỡng cụ thể? | Xe và người cao **từ 20 px** vẽ từng cái (`person` và xe là hai box riêng). Cụm từ **5 vật trở lên** cao **dưới 20 px**, hoặc **nhoè** đến mức không tách được từng cá thể (ảnh phơi sáng dài ban đêm), dùng **một** `ignore_region` bao cụm. Không dùng `ignore_region` cho vật rõ chỉ vì khó vẽ | img_018, img_026, img_028 (dày), img_024 (nhoè) | CAL-03 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Khay, rổ, giỏ **gắn cố định** vào xe thì **tính** vào box xe. Hàng xếp thêm **nhô ra quá ~20% chiều dài xe** thì **không** tính (đúng với 4.1.5). Mask theo cùng quy tắc | img_031 | CAL-04 |

**Chưa chốt** (batch này không có đủ ví dụ để đo): câu 1 (xe đạp điện), 4 (người trong tuk-tuk có mui), 5 (kính chắn gió), 6 (box chồng nhau trong ảnh đêm), 7 (ngưỡng `conf` tối thiểu). Giữ nguyên v1.0 cho các câu này; sẽ chốt khi có ví dụ.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_017, img_004, img_002, img_032 | Tuk-tuk, xích lô | YOLO26 không có lớp `three_wheeler`: bỏ sót tuk-tuk (img_002), gọi xích lô là `bicycle` (img_032); YOLOE bỏ sót tuk-tuk bị che (img_017) | Luôn gán `three_wheeler`; người duyệt đổi nhãn tay | Mục 2 và mục 5: tuk-tuk, xích lô luôn là `three_wheeler` bất kể máy gọi gì (CAL-01) |
| img_003, img_008, img_010, img_035 | Nhiều box trùng trên một xe | YOLOE ở ngưỡng 0.15 ra nhiều box `three_wheeler` điểm thấp chồng nhau (ví dụ img_003 #17 #18 #19, img_035 #284 #285) | Một xe một box bao trọn từ mui xuống bánh; xoá box trùng hoặc ma | Mục 4.1, thêm điều 8 (một vật một box) (CAL-02) |
| img_013, img_018 | Xe buýt | YOLO26 gán `car` (img_013 #77) hoặc YOLOE gán `three_wheeler` (img_018 #102) | Xe khách, minibus là `bus` | Mục 2: ví dụ `bus` gồm minibus màu trắng vàng (CAL-05) |
| img_014 | Gánh hàng rong | YOLOE gọi giỏ và túi trên đòn gánh là `three_wheeler` | Gánh hàng, giỏ treo trên đòn gánh không phải xe; không gán, người bán là `person` | Mục 2: cột "Không gồm" của `three_wheeler` (CAL-06) |
| img_020 | Xe tải ở xa, mờ | YOLO26 gán `truck` | Chọn lớp tin nhất và tick `can_review` | Không đổi, đã có mục 4.3 (CAL-07) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt ngưỡng 20 px và trường hợp nhoè cho `ignore_region`; phân biệt hàng gắn cố định và hàng nhô ra; thêm điều 8 (một vật một box); luôn gán `three_wheeler` cho tuk-tuk/xích lô; xe khách là `bus`; gánh hàng không phải xe | CAL-01 đến CAL-06 |
