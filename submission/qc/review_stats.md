# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 501; đối tượng sau duyệt: 539; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 447 | 447/501 = 89.2% | 501 box máy |
| edited (sửa) | 29 | 29/501 = 5.8% | 501 box máy |
| deleted (xoá) | 25 | 25/501 = 5.0% | 501 box máy |
| added (vẽ thêm) | 63 | 63/539 = 11.7% | 539 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 58.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 255 | 5 | 7 | 20 |
| bicycle | 5 | 1 | 2 | 2 |
| motorbike | 122 | 14 | 2 | 16 |
| car | 36 | 0 | 2 | 17 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 3 | 0 |
| three_wheeler | 23 | 9 | 9 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 88 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 52 | 20 | 36 | 72.2% (KTC 95%: 61–81%) | 59.1% (KTC 95%: 49–69%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - extra: 2/3
  - loose: 2/3
  - missing: 3/3
  - lọt: img_005.jpg (loose), img_021.jpg (extra), img_033.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
