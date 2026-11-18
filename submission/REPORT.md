# Báo cáo Day 08 — AI hỗ trợ gán nhãn

Học viên: Phạm Văn Hùng, MSHV 2A202603161. Máy: Windows, không GPU (chạy CPU), CVAT Docker local.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (run_id trong model_log.csv) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n (`--size n --imgsz 640`, CPU) | motorbike 0.15; các lớp còn lại mặc định 0.3 | r05: nhanh (0.33 s/ảnh), bắt được person 69 và motorbike 32 ở conf 0.25 trên 8 ảnh pilot. conf 0.5 làm mất 41/69 person và 17/32 motorbike (xe, người ở xa) nên không dùng. Ngưỡng motorbike 0.15 từ threshold_sweep: recall 100%, precision 56% (5 vật gold). person và car không đạt recall 90% (tối đa 88% và 86%) nên giữ mặc định. bicycle (4 vật), bus (0), truck (1) dưới 5 vật gold nên không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r03: YOLO26 không có lớp này; YOLOE tìm đúng tuk-tuk ở img_002 (0.93) và 3 xích lô ở img_032. threshold_sweep: recall 100%, precision 67% (6 vật gold) |

- **Lệnh merge_prelabels.py đã dùng:**
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.15,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
  Kết quả: 350 box cho 24 ảnh batch và 4 gold (person 128, motorbike 123, three_wheeler 52, car 37, bicycle 5, bus 3, truck 2).
- **Phiên bản SAM đã chọn: SAM 2.1 (`sam2.1_b.pt`).** Lý do: cân bằng chất lượng mask và tốc độ, chạy được trên CPU (8.68 s/ảnh, chậm hơn YOLO26n khoảng 26 lần nên chỉ dùng làm mask cho vài vật khó, không pre-label hàng loạt), không cần token như SAM 3.
- **Lỗi không sửa được bằng ngưỡng:** YOLO26 là detector danh mục cố định (COCO), không có lớp `three_wheeler`. Ví dụ img_002: tuk-tuk xanh lá không có box nào; img_032: xích lô bị gọi `bicycle`; img_011: xe ở mép ảnh bị gọi `truck`. Tăng hay giảm ngưỡng đều không đổi được. Cách sửa: dùng YOLOE (gõ chữ) cho lớp này và người duyệt đổi nhãn tay (CAL-01).
- **Box điểm cao mà sai / điểm thấp mà đúng:** điểm cao mà sai: img_002, xe bán tải trắng bị YOLO26 gọi `truck` 0.73. Điểm thấp mà đúng: img_032, xích lô được YOLOE cho 0.48 và 0.27 (ghi theo cache `yolo26m`, đã đối chiếu `viz` trên pilot). Ngoài ra YOLOE cho điểm thấp (0.18 đến 0.38) cho nhiều box `three_wheeler` thừa, và tôi đã giữ chúng ở lô batch, nên QC vòng 1 phát hiện nhiều box ma.
- **Giới hạn:** chỉ 4 ảnh gold; ngưỡng chọn và chấm trên cùng dữ liệu nên precision/recall của máy trên gold là lạc quan; lớp bus/truck/bicycle ít vật nên không có ngưỡng riêng. Đã chạy YOLO26 cỡ n (không phải m) vì máy không có GPU.

## 2. AI error profile (Part 4, review_stats vòng 1)

Ảnh tính: 24 batch; 298 box máy; 275 đối tượng sau duyệt; accepted khi IoU ≥ 0.9; đã loại 22 đối tượng quanh các bẫy khỏi bảng.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 241 | 80.9% | 298 box máy |
| edited | 23 | 7.7% | 298 box máy |
| deleted | 34 | 11.4% | 298 box máy |
| added | 11 | 4.0% | 275 đối tượng sau duyệt |

- 7 ca trong deleted + added là **đổi lớp** (máy gán sai lớp, box đúng chỗ), không phải máy bỏ sót; số vật máy **sót thật** ≈ 11 − 7 = 4.
- Theo lớp: three_wheeler bị xoá 14/40 box máy (YOLOE ở ngưỡng 0.15 cho nhiều box thừa); motorbike 12 xoá và 13 sửa trên 112 box máy (hậu quả ngưỡng 0.15); truck 0 box giữ nguyên (1 xoá, 2 thêm), tức máy gần như sai hoàn toàn lớp này.
- **So với gold** (31 đối tượng): máy TP 24, FP 15, FN 7; precision 61.5% (KTC 95%: 46–75%), recall 77.4% (KTC 95%: 60–89%). Lạc quan vì chọn ngưỡng và chấm trên cùng 4 ảnh.
- Vì sao loại vùng bẫy khỏi profile: bẫy là lỗi nhân tạo được cài sẵn, không phản ánh chất lượng thật của máy.
- **Máy có đáng dùng không?** Thời gian: gán tay gold 15 phút / 4 ảnh = 3.75 phút/ảnh; duyệt có pre-label 20 phút / 24 ảnh = 0.83 phút/ảnh, tức nhanh hơn khoảng 78%. Nhưng cần đọc cùng chất lượng: accepted 80.9% cho thấy phần lớn box máy dùng được, song recall của máy chỉ 77.4%, bắt bẫy chỉ 58.3% và QC vòng 1 vẫn ra 22.7% đối tượng lỗi. Vậy phần nhanh một phần do tôi duyệt vội (0.83 phút/ảnh, khuyến nghị khoảng 3 phút/ảnh). Thiên lệch cần nói rõ: 4 ảnh gold tôi gán trước nên đã quen kiểu cảnh này. Kết luận: máy đáng dùng làm điểm xuất phát, nhưng không đủ để bấm Accept; `added` thấp (4.0%) không có nghĩa máy không sót, vì bẫy `missing` cũng lọt 1/3.

## 3. Ảnh bẫy

**Số đo chính thức (vòng 1, trước khi xem đáp án): bắt 7/12 = 58.3% (KTC 95%: 32–81%), dưới ngưỡng 80%.**

| Loại bẫy | Bắt được | Lọt |
|---|---|---|
| extra | 3/3 | |
| missing | 2/3 | img_009 |
| wrong_class | 1/3 | img_008, img_010 |
| loose | 1/3 | img_025, img_014 |

**Phân tích bẫy lọt (bước checklist nào bị bỏ):**
- `wrong_class` (img_008, img_010): hai box là người bị gán `bicycle`. Box nằm đúng chỗ nên mắt thấy "đã có box" và không đọc lại nhãn: bỏ **bước 3 (sai nhãn)**. Bẫy `car` → `three_wheeler` ở img_033 thì bắt được vì nhìn không hợp lý.
- `loose` (img_025, img_014): box chỉ dịch khoảng 20–35 px so với box đúng, khó thấy bằng mắt: bỏ **bước 4 (lỏng)**.
- `missing` (img_009): thiếu một người bị cắt ở **mép trái ảnh**: bỏ **bước 0 (quét ảnh trước khi nhìn nhãn máy)**, nhất là vùng mép.
- Bắt tốt `extra` (3/3, box ma dễ thấy) nhưng yếu ở lỗi tinh tế: đúng với **thiên kiến tự động hóa** (neo vào nhãn có sẵn, tin điểm cao, mệt dần). Thời gian duyệt chỉ 20 phút cho 24 ảnh (0.83 phút/ảnh), nhanh hơn nhiều so với khuyến nghị khoảng 3 phút/ảnh, nên rất có thể tôi bỏ bước 3 và 4.
- **Lưu ý đo:** bẫy chấm theo box gốc của máy. Vòng 2 (sau khi xem `traps_key.json` và sửa lại các ảnh lọt) cho 10/12 = 83.3% nhưng **không tính là bắt bẫy** vì đã biết đáp án; nó chỉ để làm sạch nhãn (`review_stats_round2.*`, `review_stats_round3.*`). Với img_014: box gốc của máy là `three_wheeler` nhưng thực chất là giỏ gánh hàng rong (không thuộc lớp nào), tôi xoá box thay vì kéo lại, nên script tính là lọt dù xử lý đúng bản chất (CAL-06). Hai bẫy còn lọt ở vòng 2 là img_009 (missing) và img_014 (loose).

## 4. Ba quy tắc giảm thiên kiến (kiểm tra được)

1. **Quét ảnh trước, đếm theo lớp, ghi số đếm cho mỗi ảnh, đặc biệt vùng mép ảnh** (bước 0). Kiểm: ở lô bẫy mới, `missing` bắt 3/3 (hiện 2/3).
2. **Đọc nhãn của từng box một lần riêng** (không chỉ nhìn có box hay không), đặc biệt box trên người và xe lớn. Kiểm: `wrong_class` bắt 3/3 (hiện 1/3).
3. **Zoom và đối chiếu cạnh box với biên vật cho mọi box lớn; dành tối thiểu khoảng 3 phút/ảnh và nghỉ 3–5 phút sau mỗi 10 ảnh.** Kiểm: `loose` bắt 3/3 (hiện 1/3), tổng thời gian duyệt từ 70 phút trở lên cho 24 ảnh, và so tỉ lệ bẫy lọt ở nửa sau lô với nửa đầu.

## 5. Mask SAM (Part 5)

Phiên bản SAM: `sam2.1_b.pt` (SAM 2.1), chạy bằng `scripts/sam2_points.py`.

| Ảnh | Click | Diện tích mask |
|---|---|---|
| img_025 (xe máy đỗ, bị cắt ở mép phải) | 1 dương | 12045 px (1.7% ảnh) |
| img_013 (tuk-tuk Bangkok, người lái, hành khách) | 2 dương, 1 âm | 317255 px (45.4% ảnh) |
| img_032 (xích lô, người đứng che một phần) | 2 dương, 1 âm | 49959 px (8.5% ảnh) |

Lượt đầu cho img_014 báo "SAM không trả mask": điểm click rơi vào giỏ gánh hàng rong (không phải xe), vì ảnh đó không có tuk-tuk. Tôi thay bằng xích lô ở img_032.

**Nhận xét từng mask** (đối chiếu guideline 4.2: polygon bám biên phần nhìn thấy của vật, người là vật riêng nên không thuộc mask xe):

- **img_013 (tuk-tuk), mask chưa đạt.** Mask phủ đúng khung xe: mui, khung sắt, ghế, thùng và bánh. Nhưng nó **gồm cả người lái** (áo xanh và quần jean, với nhiều lỗ thủng nhỏ rải rác) và **một phần hành khách ngồi sau** (người áo sọc đỏ trắng), trong khi click âm chỉ đặt một điểm trên áo người lái. Theo 4.2 và mục 5 của v1.0 ("người là vật riêng, xe là vật riêng") phải loại họ ra khỏi mask. Lỗi: SAM hiểu "vật" là cả vùng liền màu, nên một click âm không đủ cho nhiều người che trước xe.
- **img_032 (xích lô), mask tốt.** Mask bám khung xe, mui gập màu hồng và hai bánh, và **không gồm** người đàn ông đứng che bên phải (nhờ click âm) hay các xích lô khác. Chỉ hơi lởm chởm ở cạnh dưới giữa hai bánh, sai lệch nhỏ.
- **img_025 (xe máy đỗ bị cắt ở mép phải), mask chấp nhận được.** Một click dương đủ để ra mask phần thân xe nhìn thấy (12045 px, 1.7% ảnh), đúng quy tắc 4.1.3 (vật bị cắt mép vẽ đến mép ảnh, không đoán phần ngoài ảnh). Không có người lái trong ảnh nên không có vấn đề gồm người; biển số của xe bên cạnh không bị lẫn vào mask.

**Quy tắc rút ra:** (1) với xe có người ngồi, cần **một click âm cho từng người** (người lái, từng hành khách), không chỉ một click; (2) sau khi SAM trả mask, luôn nhìn lại phần giữa xe để kiểm tra người có bị tô lẫn vào không; (3) mask đúng chưa cứu được nhãn sai: nếu box ban đầu gán nhầm lớp thì mask vẫn theo lớp sai. Cách làm cho img_013 vòng sau: thêm click âm tại người lái (khoảng x740,y330 và x770,y430) và hành khách (khoảng x470,y250), rồi chạy lại.

## 6. Calibration

- **Số quyết định:** 8 dòng trong `qc/calibration_decisions.csv` (CAL-01 đến CAL-08), gồm `loi_may` (CAL-01, 02, 05, 06), `cau_hoi_mo` (CAL-03, 04), `can_review` (CAL-07) và `bay_lot` (CAL-08).
- **Phân biệt guideline mơ hồ và tôi/máy sai:** CAL-03 và CAL-04 là guideline mơ hồ (câu hỏi mở số 2 và 3 của v1.0), nên vào v1.1. CAL-01, 02, 05, 06 là lỗi của máy, nhưng vẫn thêm quy tắc rõ ràng vào v1.1 để người duyệt không phải đoán. CAL-07 đã có sẵn trong mục 4.3. CAL-08 (person bị gán bicycle, img_008 và img_010) là lỗi của tôi khi duyệt vội, không đổi guideline mà đưa vào quy tắc giảm thiên kiến (mục 4).
- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:** ngưỡng 20 px và trường hợp nhoè cho `ignore_region`; hàng gắn cố định tính vào box xe, hàng nhô ra quá 20% không tính; một vật một box; tuk-tuk và xích lô luôn là `three_wheeler`; xe khách là `bus`; gánh hàng không phải xe. Các câu hỏi chưa chốt (1, 4, 5, 6, 7) giữ nguyên v1.0 vì batch không có đủ ví dụ.

## 7. QC và quyết định

Ba vòng QC, mỗi vòng sửa nhãn trong CVAT, export `qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py`:

| Vòng | Mẫu | Lỗi theo đối tượng | Ảnh có lỗi | Cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật |
|---|---|---|---|---|
| 1 (seed 42) | 8 ảnh (3 ca khó + 5 ngẫu nhiên), 88 đối tượng | 20/88 = 22.7% | 8/8 = 100% | 100% |
| 2 (seed 7) | 5 ảnh ngẫu nhiên, 48 đối tượng | 4/48 = 8.3% | 3/5 = 60% | 88% |
| 3 (seed 11) | 5 ảnh ngẫu nhiên, 69 đối tượng | 8/69 = 11.6% | 5/5 = 100% | 100% |

- **Cách chọn mẫu:** `qc_sample.py` lấy tối đa 3 ảnh có `can_review` và 5 ảnh ngẫu nhiên (vòng 1); các vòng sau 5 ảnh ngẫu nhiên với seed khác để có ảnh mới. Phiếu `qc_review` ghi mỗi lỗi một dòng với `error_type`.
- **Lỗi chủ yếu:** vòng 1 là `thua` (9) và `bo_sot` (9) và `sai_lop` (2); vòng 3 là `bo_sot` (6/8). Lỗi bỏ sót lặp lại nên là lỗi hệ thống, cần sửa quy tắc, không chỉ sửa từng ảnh.
- **Vì sao vòng 3 không tốt hơn vòng 2:** mẫu vòng 3 rơi trúng 3 ảnh rất dày (img_024 giao thông đêm nhoè, img_026 xe máy đỗ sát nhau, img_028 đám đông xe máy), đúng các cảnh mà guideline v1.0 chưa nói rõ (câu hỏi mở số 2). Cần nói rõ: các lỗi `bo_sot` được đếm ở tử số nhưng không nằm trong số đối tượng đã vẽ ở mẫu số, nên tỉ lệ theo đối tượng chỉ là ước lượng thô.
- **Giới hạn của mẫu nhỏ:** với 5 ảnh, ngay cả khi 0 ảnh lỗi thì cận trên 95% vẫn là khoảng 43%, nên ngưỡng 2% không thể được chứng minh bằng mẫu cỡ này. Ba mẫu khác nhau (8, 5, 5 ảnh) trên cùng 24 ảnh cũng không so sánh trực tiếp được.
- **Phiên bản phiếu QC:** phiếu `qc_review` do tôi tự kiểm bằng mắt; phần nháp được Claude soạn từ ảnh QC rồi tôi xác nhận (xem mục 8).
- **Quyết định: Rework.** Lý do khớp số liệu: ảnh có lỗi 100% ở vòng 3 (vượt ngưỡng 2%), bắt bẫy 58.3% dưới ngưỡng 80% (7/12), lỗi bỏ sót lặp lại (6/8 ở vòng 3). Bước tiếp theo: áp dụng guideline v1.1 để xử lý ảnh dày, sửa các lỗi `bo_sot` còn lại, rồi QC lại với mẫu lớn hơn (từ 12 ảnh trở lên) trước khi xem xét Release.

## 8. Ghi chú cá nhân

- **Công cụ AI đã dùng và cách kiểm tra:** dùng Claude làm trợ lý hướng dẫn lab, đọc output các script, soạn khuôn báo cáo, bản nháp `model_log.csv`, phiếu QC và guideline v1.1. Cụ thể:
  - Gold gán tay 4 ảnh bằng CVAT **trước khi** chạy model lên ảnh gold. Sau khi gán xong, tôi gửi ảnh chụp và `gold_manual.json` cho Claude để góp ý; Claude chỉ ra tuk-tuk bị gán nhầm `car`/`bicycle`, taxi gán `bicycle`, thiếu `person` và `ignore_region` đè lên người rõ nét. Tôi sửa lại trong CVAT rồi export lại. Gold không bị nhiễm nhãn máy, nhưng đã được rà bằng AI sau khi gán.
  - Nhận xét trong `model_log.csv` soạn dựa trên cache `yolo26m` và ảnh ghép pilot, đối chiếu với `viz` của bản `yolo26n` của tôi.
  - Phiếu QC (`qc_review_round1`, `round2`, `round3`) được Claude soạn nháp từ ảnh QC; tôi kiểm trong CVAT và đổi `reviewer` thành tên tôi.
  - Review batch: quét ảnh trước, duyệt theo checklist 4 bước; chỉ mở `traps_key.json` sau khi export vòng 1. Vòng 2 và 3 sửa nhãn sau khi xem đáp án (không tính là số đo bắt bẫy).
- **Ca khó (can_review):**
  - img_020: box `truck` ở xa và mờ (góc phải ảnh), không chắc là `truck` hay `three_wheeler` hoặc `bus` (CAL-07).
  - img_011 (pilot): xe bị cắt ở mép trái ảnh, không rõ tuk-tuk hay taxi.
  - img_031: xe đạp chở khay trái cây, box xe có gồm khay không (CAL-04).
- **Gold có bị nhiễm nhãn máy không:** không. Gold gán tay trước khi chạy bất kỳ model nào lên 4 ảnh gold.
