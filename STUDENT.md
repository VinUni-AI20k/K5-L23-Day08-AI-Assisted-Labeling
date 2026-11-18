# Thông tin học viên

- Họ và tên: Huỳnh Ngọc Gia Hân
- MSHV: 2A202603128
- Lớp (L2 / L3 / khác): L23
- Tài khoản GitHub: suadaunanh
- Repo bài nộp (URL): https://github.com/suadaunanh/K5-L23-Day08-2A202603128-HuynhNgocGiaHan-AI-Assisted-Labeling
- Commit chốt bài (hash): xem commit cuối trên nhánh main (ghi lại sau khi push)
- Máy: macOS, Apple Silicon, 24 GB RAM, GPU qua MPS (không CUDA)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 447 / 29 / 25 (mẫu số: 501 box máy); added: 63 (mẫu số: 539 đối tượng)
- So với gold: precision / recall của **máy** 52/72 = 72.2% / 52/88 = 59.1%
- Bắt bẫy: 9/12 = 75.0%
- Thời gian: gán tay 5.0 phút/ảnh (20 phút / 4 ảnh); duyệt có pre-label 1.7 phút/ảnh (40 phút / 24 ảnh)
- Lỗi QC: 80% theo ảnh (4/5), 9.0% theo đối tượng (10/111)
- **Quyết định:** Rework — lý do một dòng: 4/5 ảnh QC có lỗi (> 2%), 9/10 lỗi là box thừa có hệ thống (three_wheeler điểm thấp) và bắt bẫy 9/12 < 80%
