# Thông tin học viên

- Họ và tên: Lương Đức Mạnh
- MSHV: 2A202603155
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: ldmanh610
- Repo bài nộp (URL): https://github.com/ldmanh610/K5-L23-DAY08-2A202603155-LuongDucManh-AIPrelabel
- Commit chốt bài (hash): 00fbe1a
- Máy: Google Colab (Linux, GPU T4 CUDA) + CVAT Web
- CVAT (Docker): CVAT Web (app.cvat.ai)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 504/506 (99.6%) / 0/506 (0.0%) / 2/506 (0.4%) (mẫu số: 506 box máy); added: 0/504 (0.0%) (mẫu số: 504 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 17/73 = 23.3% / 17/28 = 60.7% (4 ảnh gold, 28 đối tượng)
- Bắt bẫy: 5/12 = 41.7% (KTC 95%: 19–68%)
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 1.25 phút/ảnh (30 phút / 24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/5 ảnh), 0.0% theo đối tượng (0/103 đối tượng)
- **Quyết định:** Rework — lý do một dòng: Tỷ lệ bắt bẫy chỉ đạt 5/12 (41.7% < 80%, lọt 3/3 loose và 3/3 missing) do thiên kiến tự động hóa (accepted 504/506 = 99.6%), cần duyệt lại lô theo Guideline v1.1 trước khi Release.
