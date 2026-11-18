# Báo cáo Day 08 — 2A202603128 - Huỳnh Ngọc Gia Hân

> Một file cho cả bài. Mọi con số kèm mẫu số và nguồn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / bus / truck | YOLO26m | 0.3 (mặc định `--conf`) | r01/r02: YOLO26 bắt tốt người và xe máy; tăng 0.25 → 0.5 làm mất người nhỏ thật (person 131 → 87 trên 8 ảnh pilot). Sweep không tìm được ngưỡng ≤ 0.5 đạt recall 90% cho person (tối đa 78%, 41 vật gold), motorbike (70%, 23 vật), bicycle (33%, 6 vật); bus (0 vật) và truck (1 vật) quá ít để chọn |
| car | YOLO26m | 0.5 | Sweep: recall 90%, precision 69% trên 10 vật gold |
| three_wheeler | YOLOE-26s-seg, `--classes three_wheeler` (prompt mặc định của taxonomy) | 0.15 | r03: YOLOE ra tuk-tuk đúng 0.93 (img_002) và xe ba bánh 0.92 (img_032), trong khi YOLO26 không có lớp này. Sweep: recall 100%, precision 78% trên 7 vật gold. **Ngưỡng này hoá ra quá thấp** (xem mục 7) |

- Lệnh merge đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM: SAM 2.1 base (`sam2.1_b.pt`)**. Lý do: là bản trong bộ `core` của bài; chạy được trên Mac M-series (MPS, 24 GB RAM) với 2.6 s/ảnh (r04); không cần xin quyền và không nặng 3.4 GB như SAM 3; bản base cân bằng giữa chất lượng và tốc độ hơn bản tiny.
- **Lỗi không sửa được bằng đổi ngưỡng:** tuk-tuk bị YOLO26 gọi `car` 0.42 (img_013) hoặc không có box xe nào (img_002, r01); xe bán tải bị gọi `truck` 0.73 (img_002). Đây là lỗi **ánh xạ lớp** của model COCO, không phải lỗi điểm tin cậy: tăng ngưỡng lên 0.5 (r02) tuk-tuk vẫn không có box. Cách sửa: lấy `three_wheeler` từ YOLOE và cho YOLOE ưu tiên khi chồng box (thứ tự file trong lệnh merge); xe bán tải phải sửa tay khi duyệt và ghi thành quy tắc (CAL-02).
- **Điểm cao mà sai:** img_002 xe bán tải `truck` 0.73; img_024 xe khách 16 chỗ `truck` (sửa thành `bus`). **Điểm thấp mà đúng:** img_032 cụm xích lô bên phải `three_wheeler` 0.27 (r03). **Điểm thấp mà sai:** img_003 `three_wheeler` 0.18–0.25 trên xe máy đỗ và mép ảnh; img_014 0.33 trên quang gánh.
- **Giới hạn:** ngưỡng được chọn và chấm trên cùng 4 ảnh gold (88–90 vật), nên precision/recall của máy trên gold là con số lạc quan; lớp có < 10 vật gold (three_wheeler 7, bicycle 6, truck 1, bus 0) thì ngưỡng gần như ngẫu nhiên. Hậu quả thấy rõ ở QC: ngưỡng 0.15 cho three_wheeler đạt recall 100% trên 7 vật gold nhưng để lọt nhiều box rác ở lô batch.

## 2. AI error profile (từ `qc/review_stats.md`)

Đã loại 30 đối tượng quanh các bẫy (lỗi nhân tạo, không phản ánh máy).

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 447 | 447/501 = 89.2% | 501 box máy |
| edited | 29 | 29/501 = 5.8% | 501 box máy |
| deleted | 25 | 25/501 = 5.0% | 501 box máy |
| added | 63 | 63/539 = 11.7% | 539 đối tượng sau duyệt |

- **Lớp máy tệ nhất: `three_wheeler`**: 9 edited + 9 deleted trên 41 box máy của lớp (23 + 9 + 9) = 18/41 = 44% phải động tay, so với person 12/267 = 4.5%. Nguyên nhân: YOLOE với ngưỡng 0.15 bắn vào xe máy đỗ, gánh hàng, toà nhà (img_003, img_014, img_035); và box tuk-tuk hay lỏng vì gồm cả bóng/mui (img_013). Thứ hai là **`car` bị sót**: 17 added (ví dụ img_024 ô tô trắng sau xe khách không có box).
- 5 trường hợp là đổi lớp (box đúng chỗ, sai lớp), nên số vật máy sót thật ≈ 63 − 5 = 58.
- So với gold, **máy**: precision 52/72 = 72.2% (KTC 95%: 61–81%), recall 52/88 = 59.1% (KTC 95%: 49–69%); mẫu số 88 đối tượng gold. Nghĩa là máy sót khoảng 4/10 vật, chủ yếu người và xe nhỏ ở xa.
- Accepted 89% **không** có nghĩa máy tốt: cùng lúc máy sót 58 vật và recall trên gold chỉ 59%.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **9/12 = 75%** (KTC 95%: 47–91%, dưới ngưỡng 80%). Theo loại: missing 3/3, wrong_class 2/3, extra 2/3, loose 2/3. Lọt: img_005 (loose), img_021 (extra), img_033 (wrong_class).
- Vì sao lọt: tôi bắt hết 3 bẫy **missing** vì vật bị sót dễ thấy khi nhìn ảnh, nhưng lọt cả ba loại bẫy mà **box trông có vẻ hợp lý**: box lỏng (không kiểm 4 cạnh khi box đúng lớp), box thừa trong ảnh đông (img_021 nhiều box, không đếm lại từng box), sai lớp (img_033, không kiểm lại lớp khi box ôm sát). Bước checklist bị bỏ: ẩn nhãn máy (phím H) để nhìn ảnh trước, và kiểm **từng** box thay vì chỉ tìm vật thiếu.
- Accepted 89% cao, bắt bẫy 75% thấp, và QC sau đó còn thấy 9 box thừa: tôi có xu hướng **tin box máy đã có sẵn** và chỉ tập trung tìm vật bị sót. Đây là thiên kiến tự động hoá: duyệt nhanh (1.7 phút/ảnh) nhưng chủ yếu "thêm", ít "xoá".

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay 20 phút / 4 ảnh = **5.0 phút/ảnh**; duyệt có pre-label 40 phút / 24 ảnh = **1.7 phút/ảnh**, nhanh khoảng 3 lần. Thiên lệch: tôi gán gold trước nên đã quen kiểu ảnh và guideline khi duyệt; và 1.7 phút/ảnh thấp hơn mức hướng dẫn (~3 phút/ảnh), phù hợp với việc lọt bẫy.
- Kết luận: **có, nhưng chỉ cho person / motorbike / car**, nơi tỉ lệ phải sửa thấp (person 12/267 = 4.5%). **Chưa đáng tin cho `three_wheeler`** ở ngưỡng 0.15 (18/41 = 44% phải sửa, và là nguồn chính của lỗi QC); và chất lượng lô hiện tại chưa đạt (QC 4/5 ảnh lỗi), nên tốc độ nhanh đã phải trả bằng chất lượng.
- **Ba quy tắc giảm thiên kiến:**
  1. **Ẩn nhãn máy 5–10 giây trước mỗi ảnh** (phím H), đếm số vật mình thấy, rồi mới bật nhãn; kiểm bằng số: tổng box sau duyệt phải khớp ± 10% với số mình đếm.
  2. **Kiểm từng box theo thứ tự lớp → 4 cạnh → có thật không**, và **xoá mọi box `three_wheeler` < 0.3 trừ khi nhìn thấy rõ 3 bánh**; kiểm bằng số: tỉ lệ deleted của three_wheeler trong lần sau và số box thừa trong QC phải giảm về 0.
  3. **Tối thiểu 3 phút/ảnh cho ảnh có ≥ 20 box**, ghi giờ bắt đầu/kết thúc từng ảnh; kiểm bằng `time_log.csv` và bắt bẫy ≥ 10/12 ở vòng duyệt lại.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_032 (xích lô) | SAM 2.1 base | 4 (2 dương: đệm ghế, biển số; 2 âm: người đi bộ, mặt đường) | Không người; gồm đệm và chăn trên ghế (gắn với xe) | Tốt nhất: đủ khung + 3 bánh; khoảng trống giữa nan bánh bị tô kín | `cyclo_img032.jpg` |
| img_002 (tuk-tuk xanh) | SAM 2.1 base | 3 (2 dương: thân, mui; 1 âm: tài xế) | Tài xế phần lớn bị loại; **khách ngồi phía sau bị tô lẫn** | Lấn vào người nhìn qua thành xe hở | `tuktuk_img002.jpg` |
| img_013 (tuk-tuk) | SAM 2.1 base | 3 (2 dương: thân, mui; 1 âm: tài xế) | **Người phía sau (nhìn qua thành hở) bị tô dính**; tài xế còn lấm tấm | Mask 45% ảnh, có mảng rời ở người | `tuktuk_img013.jpg` |

- Đối chiếu guideline 4.2: quy tắc "lỗ nhỏ hơn ~5% thì không vẽ lỗ" giúp chấp nhận mask xích lô tô kín nan bánh. **Thiếu** quy tắc: mask xe có gồm người ngồi trong/sau xe không. v1.0 chỉ nói cho box (mục 4.1.4), nên tôi thêm vào v1.1 (CAL-05): mask xe không gồm người, cần thêm một click âm cho từng người.
- Kinh nghiệm: SAM tốt với vật đặc (thân, mui); với xe có thành hở, 1 click âm là không đủ, cần một click âm cho mỗi người nhìn thấy.
- Interactor SAM trong CVAT: không thử.

## 6. Calibration (Part 6)

- Số quyết định: **6** (`calibration_decisions.csv`): 2 `loi_may`, 3 `cau_hoi_mo`, 1 `bay_lot`. Quan trọng nhất: **CAL-01** (xe ba bánh luôn là `three_wheeler`, không tin lớp YOLO26 gợi ý), vì đây là lỗi lặp lớn nhất của máy.
- Quy tắc mới vào `guideline_v1.1_calibrated.md`: xe 3 bánh luôn `three_wheeler`; xe bán tải = `car`; xe khách ≥ 9 chỗ = `bus`; cụm xe nhoè không tách được → `ignore_region` kể cả khi > 20 px; mask xe không gồm người; khi duyệt kiểm cả 4 cạnh box.
- Nguyên nhân gốc:
  - **Guideline mơ hồ**: CAL-03 (ranh giới car/bus), CAL-04 (cụm xe gần camera), CAL-05 (người trong mask xe).
  - **Máy sai**: CAL-01 (tuk-tuk → car), CAL-02 (bán tải → truck).
  - **Tôi sai**: CAL-06 (bẫy box lỏng lọt). Ngoài ra, tôi **không tick `can_review`** cho box nào trong gold lẫn batch, kể cả ca không chắc như xe khách img_024. Vì vậy mẫu QC có 0 ca khó.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035; 111 đối tượng. Người kiểm: tôi (**hạn chế**: tôi vừa là người duyệt nên dễ bỏ qua lỗi của chính mình).
- Lỗi theo **đối tượng**: 10/111 = 9.0%. Theo **ảnh**: 4/5 = 80% (ngưỡng Release ≤ 2%).
- Top lỗi:
  - **thừa** 9/10: 7 box `three_wheeler` điểm thấp không có xe (img_003 #1, #2, #25; img_014 #151, #152 trên quang gánh; img_035 #547 trên toà nhà), 2 box trùng cùng một xe ba gác (img_003 #23, #26), 1 `person` trên mũ treo (img_031 #475).
  - **bỏ sót** 1/10: xe đạp của người #19 (img_003).
  - Không có sai lớp hay sai vị trí trong mẫu.
- **Giới hạn mẫu nhỏ:** với 4/5 ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 96%. Với chỉ 5 ảnh, ngay cả khi 0/5 ảnh lỗi cũng không chứng minh được ngưỡng 2%. Ở đây số liệu xa ngưỡng, nên kết luận "không đạt" vẫn chắc chắn.
- Script kiểm được: định dạng, nhãn hợp lệ, box < 20 px, bẫy, đếm lỗi. **Không** kiểm được: độ ôm sát thật, vật mà cả máy lẫn tôi cùng bỏ sót, box thừa trông hợp lý. Những lỗi này chỉ thấy bằng mắt khi QC.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (theo cây Hold → Rework → Release):
- **Không Hold**: guideline đủ rõ cho các lỗi tìm được (box thừa không có vật là lỗi rõ ràng, không phải ca mơ hồ), và dữ liệu không hỏng.
- **Rework**: ảnh lỗi 4/5 = 80% > 2%; lỗi có hệ thống ("thừa" 9/10, chủ yếu `three_wheeler` < 0.5); bắt bẫy 9/12 = 75% < 80%, nên lô duyệt chưa đáng tin.

Việc cần làm tiếp:
1. Nâng ngưỡng `three_wheeler` lên khoảng 0.5 (bỏ box rác 0.15–0.48 thấy trong QC), gộp lại.
2. Duyệt lại cả 24 ảnh theo 3 quy tắc ở mục 4, ưu tiên xoá box `three_wheeler` không có 3 bánh và box trùng.
3. Lưu nhãn sửa vào `qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force`, rồi ghi vòng QC thứ hai vào báo cáo.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** có, Claude Code (trong VS Code), với các việc sau:
  - Chạy các script và lệnh của hướng dẫn: prelabel, threshold_sweep, merge, inject_traps, review_stats, sam2_points, build_release, qc_sample, qc_report.
  - Đặt file export của CVAT vào `submission/`, sửa file nhãn cho CVAT v2.74 (danh sách `values` rỗng bị từ chối), commit.
  - Soạn nháp `model_log.csv` (nhận xét từ ảnh viz), `calibration_decisions.csv`, `guideline_v1.1`, REPORT. Tôi đọc lại và chỉnh.
  - Gợi ý toạ độ click cho SAM.
  - Ở bước QC, AI chỉ ra các box đáng ngờ và tôi tự mở ảnh xác nhận từng box (đồng ý #1, #2, #25, #151, #152, #547; tự tìm #19, #22/#23/#26, #475).

  Kiểm tra: chạy `validate_labels`, `check_submission`, xem lại ảnh viz/mask/QC.
- **Gold có bị nhiễm không:** không. 4 ảnh gold do tôi tự vẽ trong CVAT trước khi chạy bất kỳ model nào; sau đó tôi sửa gold một lần (thêm 5 `person` ở img_020) **trước khi** xem kết quả so sánh với máy. Bản sửa không làm đổi ngưỡng.
- **Duyệt batch:** tôi tự duyệt 24 ảnh trong CVAT; `traps_key.json` chỉ được mở sau khi export `batch_reviewed.json`.
- Máy: MacBook Apple Silicon, 24 GB RAM, chạy model bằng MPS (không dùng Colab). CVAT v2.74.1 qua Docker (bản cài từ buổi trước, không phải 2.81). Vướng: CVAT v2.74 báo lỗi "attribute values must be a non-empty array" khi dán `labels_generated.json`, nên phải đổi `values: []` thành `[""]` / `["false"]`. Task gold và batch tạo ngoài project (nhãn dán vào từng task).
- Dùng cache `data/cache/`: không, tự chạy cả 36 ảnh.
- Lỗi script: `check_submission.py` báo "README.md phải có dòng tiêu đề" dù tiêu đề đúng mẫu, vì regex chỉ nhận MSHV toàn chữ số (`\d{5,}`) còn MSHV của tôi là `2A202603128`. Tôi giữ MSHV thật, không sửa script; ngoài lỗi này checker không còn lỗi nào. Gold còn 10 cảnh báo box < 20 px ở img_036 và img_001: tôi giữ nguyên, không đổi sang `ignore_region`, vì đã dùng gold này để chọn ngưỡng.
- Lần sau: tick `can_review` cho ca không chắc ngay khi gán, và chậm lại khi duyệt (khoảng 3 phút/ảnh, ẩn nhãn máy trước).
