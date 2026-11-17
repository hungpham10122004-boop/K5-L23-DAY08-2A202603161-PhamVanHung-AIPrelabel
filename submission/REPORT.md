# Báo cáo Day 08 — AI hỗ trợ gán nhãn

Học viên: Phạm Văn Hùng, MSHV 2A202603161. Máy: Windows, không GPU (CPU), CVAT Docker local.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (run_id trong model_log.csv) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n (`--size n --imgsz 640`, CPU) | motorbike 0.15; các lớp còn lại mặc định 0.3 | r05: nhanh (0.33 s/ảnh), bắt được person 69 và motorbike 32 ở conf 0.25 trên 8 ảnh pilot. conf 0.5 làm mất 41/69 person và 17/32 motorbike (xe, người ở xa) nên không dùng. Ngưỡng motorbike 0.15 từ threshold_sweep: recall 100%, precision 56% (5 vật gold). person và car không đạt recall 90% (tối đa 88% và 86%) nên giữ mặc định. bicycle (4 vật), bus (0), truck (1) dưới 5 vật gold nên không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r03: YOLO26 không có lớp này. YOLOE tìm đúng tuk-tuk ở img_002 (0.93) và 3 xích lô ở img_032. threshold_sweep: recall 100%, precision 67% (6 vật gold) |

- **Lệnh merge_prelabels.py đã dùng:**
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.15,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
  Kết quả: 350 box cho 24 ảnh batch + 4 gold (person 128, motorbike 123, three_wheeler 52, car 37, bicycle 5, bus 3, truck 2).
- **Phiên bản SAM đã chọn: SAM 2.1 (`sam2.1_b.pt`).** Lý do: cân bằng chất lượng mask và tốc độ, chạy được trên CPU (8.68 s/ảnh, chậm hơn YOLO26n khoảng 26 lần nên chỉ dùng làm mask cho vài vật khó, không pre-label hàng loạt), không cần token như SAM 3.
- **Lỗi không sửa được bằng ngưỡng:** YOLO26 là detector danh mục cố định (COCO), không có lớp `three_wheeler`. Ví dụ img_002: tuk-tuk xanh lá không có box nào; img_032: xích lô bị gọi `bicycle`; img_011: xe ở mép ảnh bị gọi `truck`. Tăng hay giảm ngưỡng đều không đổi được. Cách sửa: dùng YOLOE (gõ chữ) cho lớp này, và người duyệt đổi nhãn tay.
- **Box điểm cao mà sai / điểm thấp mà đúng:** điểm cao mà sai: img_002, xe bán tải trắng bị gọi `truck` 0.73 (YOLO26) và 0.37 (YOLOE gọi three_wheeler). Điểm thấp mà đúng: img_032, xích lô được YOLOE cho 0.48 và 0.27. <<kiểm lại bằng viz của bản n>>
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
- Theo lớp: three_wheeler bị xoá 14/40 box máy (YOLOE ở ngưỡng 0.15 cho nhiều box thừa); motorbike 12 xoá + 13 sửa / 112 box máy (hậu quả ngưỡng 0.15); truck 0 box giữ nguyên (1 xoá, 2 thêm).
- **So với gold** (31 đối tượng): máy TP 24, FP 15, FN 7; precision 61.5% (KTC 95%: 46–75%), recall 77.4% (KTC 95%: 60–89%). Lạc quan vì chọn ngưỡng và chấm trên cùng 4 ảnh.
- Vì sao loại vùng bẫy khỏi profile: bẫy là lỗi nhân tạo được cài sẵn, không phản ánh chất lượng thật của máy.
- **Máy có đáng dùng không?** Accepted 80.9% cho thấy phần lớn box máy dùng được, nhưng recall trên gold chỉ 77.4% và `added` thấp (4.0%) không có nghĩa máy không sót: bẫy `missing` lọt 1/3 cho thấy người duyệt cũng dễ bỏ qua vật máy sót. Thời gian: gán tay gold 15 phút / 4 ảnh = 3.75 phút/ảnh; duyệt có pre-label <<... phút / 24 ảnh = ... phút/ảnh>>. <<kết luận: tiết kiệm bao nhiêu % thời gian>>

## 3. Ảnh bẫy

**Số đo chính thức (vòng 1, trước khi xem đáp án): bắt 7/12 = 58.3% (KTC 95%: 32–81%), dưới ngưỡng 80%.**

| Loại bẫy | Bắt được | Lọt |
|---|---|---|
| extra | 3/3 | |
| missing | 2/3 | img_009 |
| wrong_class | 1/3 | img_008, img_010 |
| loose | 1/3 | img_025, img_014 |

**Phân tích bẫy lọt (bước checklist nào bị bỏ):**
- `wrong_class` (img_008, img_010): hai box là người bị gán `bicycle`. Box nằm đúng chỗ nên mắt thấy "đã có box" và không đọc lại nhãn: bỏ **bước 3 (sai nhãn)**. Bẫy car→three_wheeler ở img_033 thì bắt được vì nhìn không hợp lý.
- `loose` (img_025, img_014): box chỉ dịch khoảng 20–35 px so với box đúng, khó thấy bằng mắt: bỏ **bước 4 (lỏng)**.
- `missing` (img_009): thiếu một người bị cắt ở **mép trái ảnh**: bỏ **bước 0 (quét ảnh trước khi nhìn nhãn máy)**, nhất là vùng mép.
- Bắt tốt `extra` (3/3, box ma dễ thấy) nhưng yếu ở lỗi tinh tế: đúng với **thiên kiến tự động hóa** (neo vào nhãn có sẵn, tin điểm cao, mệt dần). <<xác nhận lại trải nghiệm của bạn: bạn nhớ mình duyệt thế nào, mệt ở ảnh thứ mấy?>>
- **Lưu ý đo:** bẫy chấm theo box gốc của máy. Vòng 2 (sau khi xem `traps_key.json`, sửa lại 5 ảnh lọt) cho 10/12 = 83.3% nhưng **không tính là bắt bẫy** vì đã biết đáp án; nó chỉ để làm sạch nhãn (`review_stats_round2.*`). Hai bẫy còn lọt ở vòng 2 (img_009, img_014): <<có phải do box bạn vẽ khác box gốc không?>>.

## 4. Ba quy tắc giảm thiên kiến (kiểm tra được)

1. **Quét ảnh trước, đếm theo lớp, ghi số đếm cho mỗi ảnh, đặc biệt vùng mép ảnh** (bước 0). Kiểm: ở lô bẫy mới, `missing` bắt 3/3 (hiện 2/3).
2. **Đọc nhãn của từng box một lần riêng** (không chỉ nhìn có box hay không), đặc biệt box trên người/xe lớn. Kiểm: `wrong_class` bắt 3/3 (hiện 1/3).
3. **Zoom và đối chiếu cạnh box với biên vật cho mọi box lớn; nghỉ 3–5 phút sau mỗi 10 ảnh** để giảm mệt mỏi. Kiểm: `loose` bắt 3/3 (hiện 1/3) và so tỉ lệ bẫy lọt ở nửa sau lô với nửa đầu.

## 5. Mask SAM (Part 5)

- Phiên bản SAM: `sam2.1_b.pt`; số click mỗi ảnh: <<...>>
- Mask có gồm người lái / hàng hoá không (đối chiếu guideline 4.2): <<...>>
- Lỗi và quy tắc rút ra: <<...>>

## 6. Calibration (Part 6)

- Số quyết định: <<... dòng trong calibration_decisions.csv>>
- Phân biệt guideline mơ hồ và tôi/máy sai: <<...>>
- Quy tắc mới đã vào guideline_v1.1_calibrated.md: <<...>>

## 7. QC và quyết định

- Mẫu QC và cách chọn: <<...>>
- Lỗi theo ảnh: <<.../... = ...%>>; theo đối tượng: <<.../... = ...%>>
- Giới hạn mẫu nhỏ (cận trên 95%): <<...>>
- **Quyết định: <<Release / Hold / Rework>>**: <<lý do khớp số liệu>>

## 8. Ghi chú cá nhân

- **Công cụ AI đã dùng và cách kiểm tra:** dùng Claude làm trợ lý hướng dẫn lab, đọc output các script, soạn khuôn báo cáo và bản nháp `model_log.csv`. Cụ thể:
  - Gold gán tay 4 ảnh bằng CVAT **trước khi** chạy bất kỳ model nào lên ảnh gold. Sau khi gán xong, tôi gửi ảnh chụp và `gold_manual.json` cho Claude để góp ý; Claude chỉ ra tuk-tuk gán nhầm `car`/`bicycle`, taxi gán `bicycle`, thiếu `person`, và `ignore_region` đè lên người rõ nét. Tôi tự kiểm lại và sửa trong CVAT rồi export lại. Gold không bị nhiễm bởi nhãn máy, nhưng đã được rà bằng AI sau khi gán.
  - Nhận xét trong `model_log.csv` được soạn dựa trên cache `yolo26m` và ảnh ghép; tôi đối chiếu lại với `viz` của bản `yolo26n` của mình. <<xác nhận>>
  - Review batch: quét ảnh trước, duyệt theo checklist 4 bước; chỉ mở `traps_key.json` sau khi export vòng 1. Vòng 2 sửa 5 bẫy lọt sau khi xem đáp án (không tính là số đo).
- **Ca khó (can_review):**
  - img_020: box truck ở xa và mờ (góc phải ảnh), không chắc là `truck` hay `three_wheeler`/`bus`.
  - img_011 (pilot): xe bị cắt ở mép trái ảnh, không rõ tuk-tuk hay taxi.
  - <<ảnh khác trong batch>>
