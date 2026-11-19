# Báo cáo Day 08 Lab — AI hỗ trợ gán nhãn: Pre-label, Review, Calibration

**Học viên:** Lương Đức Mạnh — **MSHV:** 2A202603155 — **Lớp:** K5-L23

## 1. Tour 3 model & Chọn ngưỡng theo lớp (Part 2 & Part 3)
- Đã thử nghiệm 3 họ model trên tập `pilot` (8 ảnh) và toàn bộ 36 ảnh (`submission/model_log.csv`, 6 dòng):
  - **YOLO26 (`yolo26m.pt`):** Tốc độ nhanh trên CUDA (`0.12s/ảnh` khi chạy 36 ảnh), bắt tốt `person` và `motorbike`, nhưng không có lớp `three_wheeler` nên dễ nhận nhầm tuk-tuk/ba gác thành `car`/`truck`.
  - **YOLOE (`yoloe-26s-seg.pt`):** Dùng prompt mở (`tuk-tuk`, `cyclo`, `cargo tricycle`) để tìm riêng lớp `three_wheeler` (`0.06s/ảnh` trên 36 ảnh, sinh 125 box ở `conf=0.05`).
  - **SAM 2.1 (`sam2.1_b.pt`):** Tạo được 16/16 mask trên 2 ảnh mẫu (`1.71s/ảnh`), nhưng chất lượng mask phụ thuộc chặt vào độ chính xác của box/điểm đầu vào.
- **Kết quả `threshold_sweep.py` trên 4 ảnh Gold (28 đối tượng):**
  - Không có lớp nào đạt mục tiêu recall 90% ở ngưỡng `<= 0.5` (`person`: tối đa 83% trên 12 vật; `motorbike`: 60% trên 5 vật; `car`: 88% trên 8 vật). Các lớp `bicycle` (3 vật), `bus` (0), `truck` (0), `three_wheeler` (0) có `< 5` vật nên không chọn ngưỡng riêng.
  - Do đó giữ ngưỡng mặc định `--conf 0.3` (`--thr ""`) khi gộp pre-label.
  - **Giới hạn mẫu nhỏ:** Chỉ có 4 ảnh gold và chọn/chấm trên cùng tập gold nên các chỉ số precision/recall trên gold mang tính lạc quan và ước lượng thô.

## 2. AI Error Profile sau Review 24 ảnh batch (Part 4)
Dữ liệu từ `submission/qc/review_stats.md` (24 ảnh, 506 box máy sau khi loại 21 đối tượng quanh bẫy, 504 đối tượng sau duyệt):
- **accepted (giữ nguyên):** `504/506 = 99.6%` (mẫu số: 506 box máy).
- **edited (sửa):** `0/506 = 0.0%` (mẫu số: 506 box máy).
- **deleted (xoá):** `2/506 = 0.4%` (mẫu số: 506 box máy, thuộc lớp `person`).
- **added (vẽ thêm):** `0/504 = 0.0%` (mẫu số: 504 đối tượng sau duyệt).
- **So với 4 ảnh Gold (28 đối tượng):** Máy đạt Precision `17/73 = 23.3%` (KTC 95%: 15–34%) và Recall `17/28 = 60.7%` (KTC 95%: 42–76%).

## 3. Phân tích 12 ảnh bẫy & Thiên kiến tự động hóa
- **Tổng tỷ lệ bắt bẫy:** `5/12 = 41.7%` (KTC 95%: 19–68%), thấp hơn ngưỡng đạt `80%`.
- Chi tiết theo loại bẫy:
  - `extra` (box thừa): bắt được `3/3` (100%).
  - `wrong_class` (sai lớp): bắt được `2/3` (lọt `img_024.jpg`).
  - `loose` (box lỏng): bắt được `0/3` (lọt `img_013.jpg`, `img_025.jpg`, `img_028.jpg`).
  - `missing` (thiếu vật): bắt được `0/3` (lọt `img_004.jpg`, `img_010.jpg`, `img_017.jpg`).
- **Nhận xét:** Tỷ lệ `accepted = 99.6%` kết hợp với tỷ lệ bắt bẫy `41.7%` là dấu hiệu điển hình của **thiên kiến tự động hóa (automation bias)**: người duyệt dễ phát hiện box thừa (`extra`) nhưng dễ bỏ qua vật máy không vẽ (`missing`) và ít chỉnh lại các box bị lỏng (`loose`).

## 4. Hiệu quả thời gian & 3 quy tắc giảm thiên kiến
- **Thời gian:** Gán tay 4 ảnh gold mất `15 phút` (`3.75 phút/ảnh`); duyệt 24 ảnh có pre-label mất `30 phút` (`1.25 phút/ảnh`), nhanh hơn khoảng `3x` nhưng đánh đổi bằng việc lọt bẫy nếu không tuân thủ quy trình.
- **3 quy tắc giảm thiên kiến tự động hóa áp dụng cho vòng duyệt lại:**
  1. **Ẩn nhãn máy trước khi duyệt:** Luôn ẩn layer pre-label, quét toàn ảnh một lượt để xác định các vật thể chính trước khi bật lại box máy (khắc phục lỗi `missing`).
  2. **Kiểm tra độ khít 4 cạnh box:** Soi riêng các box có kích thước lớn hoặc nền xung quanh nhiều khoảng trống để bắt lỗi `loose`.
  3. **Kiểm tra riêng nhóm lớp hay nhầm (`three_wheeler` vs `car`/`truck`):** Rà soát mọi xe 3 bánh và gom cụm vật `< 20 px` vào `ignore_region`.

## 5. Kết quả tạo Mask bằng SAM (Part 5)
- Sử dụng **SAM 2.1 (`sam2.1_b.pt`)** qua `scripts/sam2_points.py` với 2 điểm dương (`--pos`) và 1 điểm âm (`--neg`) cho mỗi vật thể:
  - `submission/masks/tuktuk_img013.jpg`: `317,255 px` (`45.4%` ảnh, 1 polygon).
  - `submission/masks/obj2_img004.jpg`: `41,081 px` (`5.9%` ảnh, 1 polygon).
  - `submission/masks/obj3_img032.jpg`: `16,205 px` (`2.7%` ảnh, 1 polygon).
- **Nhận xét:** SAM 2.1 cho đường viền sắc nét chỉ với 3 click (2 dương, 1 âm). Tuy nhiên với vật thể phức tạp như tuk-tuk có người lái hoặc bị che khuất (`img_013.jpg`), cần đặt thêm điểm âm vào vùng nền/người lái để mask không bị tràn ra ngoài thân xe theo đúng guideline.

## 6. Calibration Guideline & Kết quả QC (Part 6)
- Đã tổng hợp 4 quyết định calibration trong `submission/qc/calibration_decisions.csv` (2 ca `loi_may`, 2 ca `cau_hoi_mo`) và cập nhật `submission/guidelines/guideline_v1.1_calibrated.md`.
- Đã lấy mẫu QC ngẫu nhiên `5/24` ảnh batch (`103` đối tượng) trong `submission/qc/qc_sample.csv` và kiểm tra trong `submission/qc/qc_review.csv` (tỷ lệ lỗi QC trên mẫu kiểm `0/5` ảnh = `0%`, `0/103` đối tượng = `0%`). Tuy nhiên cỡ mẫu QC nhỏ (5 ảnh) và vẫn còn 11 cảnh báo box `< 20 px` từ `validate_labels.py`.

## 7. Quyết định chất lượng lô dữ liệu: **Rework**
- **Quyết định:** **Rework** (không Release ngay).
- **Lý do (kèm mẫu số):** Dù tốc độ duyệt tăng (`1.25 phút/ảnh` so với `3.75 phút/ảnh` gán tay) và QC 5 ảnh mẫu không ghi nhận lỗi nghiêm trọng (`0/5` ảnh), tỷ lệ bắt bẫy chỉ đạt **`5/12 = 41.7%`** (dưới ngưỡng bắt buộc `80%`, lọt `3/3` bẫy `loose` và `3/3` bẫy `missing`), đồng thời tỷ lệ giữ nguyên quá cao (`504/506 = 99.6%`) và còn 11 box `< 20 px` chưa gom vào `ignore_region`.
- **Hành động tiếp theo:** Duyệt lại toàn bộ 24 ảnh theo Guideline v1.1 (áp dụng 3 quy tắc chống thiên kiến), thay 11 box `< 20 px` bằng `ignore_region`, sau đó đo lại `review_stats` trước khi Release.

## 8. Ghi chú & Giới hạn
- Cỡ mẫu gold (4 ảnh, 28 đối tượng) và mẫu QC (5 ảnh, 103 đối tượng) còn nhỏ nên khoảng tin cậy 95% rộng (`19–68%` cho bắt bẫy; `42–76%` cho recall trên gold). Mọi kết luận đều được đọc kèm mẫu số và danh sách ca lỗi cụ thể.

### Bài nâng cao (Mục 1 `docs/ADVANCED.md`): So sánh 3 model Open-Vocabulary cho lớp `three_wheeler` (8 ảnh `pilot`, CUDA)
| Model | Ngưỡng (`--conf`) | Số box `three_wheeler` | Tốc độ (`sec/img`) | Nhận xét |
|---|---:|---:|---:|---|
| **YOLOE (`yoloe-26s-seg.pt`)** | `0.25` | `8` | `0.18s` | Nhanh nhất (~3–4×), ít box thừa nhất, phù hợp chạy pre-label hàng loạt |
| **Grounding DINO (`grounding-dino-tiny`)** | `0.30` | `16` | `0.71s` | Tìm được nhiều xe ba bánh hơn YOLOE nhưng chậm hơn ~4× |
| **OWLv2 (`owlv2-base-patch16-ensemble`)** | `0.20` | `44` | `0.59s` | Recall cao nhất nhưng sinh nhiều box thừa (false positive) ở ngưỡng `0.20` |

*Lưu ý:* Ngưỡng điểm giữa 3 model không so sánh trực tiếp 1-1 được; cần chọn model và ngưỡng theo từng lớp trên dữ liệu thực tế.
