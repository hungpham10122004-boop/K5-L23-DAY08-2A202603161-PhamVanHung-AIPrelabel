# Báo cáo Day 08 — <<2A202603161-Phạm Văn Hùng>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / OYOLO26n (--size n --imgsz 640, CPU)LO   _YOLO26n (--size n --imgsz 640, CPU)sw>r05: nhanh (0.33 s/ảnh), bắt được person 69 và motorbike 32 ở conf 0.25. conf 0.5 mất 41/69 person và 17/32 motorbike nên không dùng. Ngưỡng motorbike 0.15 từ threshold_sweep (recall 100%, precision 56%). person và car không đạt recall 90% (tối đa 88% và 86%) nên giữ mặc định. bicycle (4), bus (0), truck (1) dưới 5 vật gold nên không chọn ngưỡng riêng | <<>> |
| three_wYOLOE-26s, prompt: tuk-tuk, cyclo, cargo tricyclept:>0.15> >r03: YOLO26 không có lớp này; YOLOE tìm đúng tuk-tuk img_002 (0.93) và 3 xích lô img_032. threshold_sweep: recall 100%, precision 67% (6 vật gold)> |Lệnh merge_prelabels.py: dán nguyên lệnh bạn chạy ở Part 4 (chưa chạy thì điền sau). Dự kiến:
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.15,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
Phiên bản SAM: SAM 2.1 (sam2.1_b.pt). Lý do: cân bằng chất lượng và tốc độ; chạy được trên CPU (8.68 s/ảnh, chậm hơn YOLO26 khoảng 26 lần) nên chỉ dùng làm mask cho vài vật khó (Part 5), không pre-label hàng loạt; không cần token như SAM 3. Nếu bạn dùng bản khác thì sửa theo.
Lỗi không sửa được bằng ngưỡng: YOLO26 thuộc danh mục COCO cố định nên không có three_wheeler: tuk-tuk img_002 không có box, xích lô img_032 bị gọi bicycle, xe ở mép img_011 bị gọi truck. Tăng hay giảm ngưỡng đều không đổi được. Cách sửa: dùng YOLOE gõ chữ cho lớp này, và người duyệt đổi nhãn tay.
Box điểm cao mà sai, điểm thấp mà đúng: điểm cao mà sai: img_002, xe bán tải trắng bị gọi truck 0.73 (YOLO26); điểm thấp mà đúng: img_032, xích lô được YOLOE cho 0.48 và 0.27. Kiểm lại bằng viz của chính bạn trước khi chép vì số lấy từ cache yolo26m.
Giới hạn: chỉ 4 ảnh gold, chọn ngưỡng và chấm trên cùng dữ liệu nên số lạc quan; lớp bus/truck/bicycle ít vật nên không có ngưỡng riêng. lạc quan: <<>>

## 2. AI error profile (từ `qc/r03: YOLO26 không có lớp này; YOLOE tìm đúng tuk-tuk img_002 (0.93) và 3 xích lô img_032. threshold_sweep: recall 100%, precision 67% (6 vật gold)review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (Ca khó (can_review):
img_020: <<box id 2 : ) ở xa, mờ; không ch bus hay truckắ.. ; chọtruck>>
Gold có bị nhiễm nhãn máy khô Khôngng:ỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
