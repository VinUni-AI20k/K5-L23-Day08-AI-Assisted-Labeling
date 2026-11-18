# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603128 - Huỳnh Ngọc Gia Hân

> Chỉ ghi phần thay đổi so với v1.0. Mỗi ca khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 + 6 | Khối xe máy dày đặc gần camera / box chồng nhau trong ảnh đêm | Không đếm được từng xe (nhoè, chồng nhau) → **một** `ignore_region` bao cụm, kể cả khi xe cao > 20 px. Xe nào tách rõ vẫn gán riêng. Không gộp box của các xe tách rõ. | img_024 | CAL-04 |
| 4 | Người ngồi trong xe ba bánh có mui, nhìn qua thành xe hở | Giữ ngưỡng box: gán `person` nếu thấy ≥ 20% người. **Mask xe không gồm người** ngồi trong hoặc sau xe. | img_002, img_013 | CAL-05 |
| — | Ranh giới car / bus (không có trong v1.0) | Xe chở khách ≥ 9 chỗ (hàng cửa sổ khách dài) = `bus`; xe gia đình ≤ 8 chỗ = `car`; xe chở khách **không bao giờ** là `truck`. | img_024 | CAL-03 |

**Chưa chốt:**
- Câu 1 (xe đạp điện): chưa gặp ca rõ trong lô này nên chưa có ảnh ví dụ. Tạm áp quy tắc theo hình dáng: có bàn đạp, khung mảnh như xe đạp → `bicycle`; có yếm, sàn để chân → `motorbike`; không rõ → tick `can_review`.
- Câu 3 (hàng nhô ra), câu 5 (kính xe), câu 7 (conf tối thiểu): chưa gặp đủ ca trong 28 ảnh để quyết.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_002 | Tuk-tuk có mui, có người ngồi | YOLO26m gọi `car` 0.42 (img_013) hoặc không ra box xe (img_002) | `three_wheeler`, box lấy từ YOLOE | Mục 2 + 4.3: đếm bánh nhìn thấy; 3 bánh → `three_wheeler` bất kể máy gợi ý gì (CAL-01) |
| img_002 | Xe bán tải có thùng hở | YOLO26m gọi `truck` 0.73 | `car` | Mục 2, dòng `truck`: thêm "không gồm xe bán tải (→ `car`)" (CAL-02) |
| img_024 | Xe khách 16 chỗ | YOLO26m gọi `truck` | `bus` + `can_review` | Mục 2: ranh giới car / bus theo số chỗ (CAL-03) |
| img_005 | Box lỏng lọt khi duyệt (bẫy) | — (bẫy cài) | Kéo 4 cạnh ôm sát | Mục 4.1.1: khi duyệt box máy, kiểm cả 4 cạnh sau khi kiểm lớp (CAL-06) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu mở 2, 4, 6; thêm ranh giới car/bus; xe bán tải = car; xe 3 bánh luôn `three_wheeler`; mask xe không gồm người; bước kiểm 4 cạnh khi duyệt | CAL-01 … CAL-06 |
