# Guideline v1.1 (Calibrated) — Phần bổ sung sau Calibration Day 08

Tài liệu này bổ sung các quy tắc xử lý ca khó phát sinh từ lỗi máy (`loi_may`) và câu hỏi mở (`cau_hoi_mo`) trong quá trình review 24 ảnh batch và đối chiếu 12 ảnh bẫy.

## 1. Phân biệt `three_wheeler` với `car` / `truck` (nguồn: `loi_may`)
- **Quy tắc:** Mọi phương tiện 3 bánh đặc thù đường phố châu Á (tuk-tuk, xích lô, xe ba gác chở hàng) bắt buộc gán nhãn `three_wheeler`, không giữ nhãn `car` hoặc `truck` do YOLO26 sinh ra.
- **Ảnh ví dụ:** `img_024.jpg`, `img_013.jpg`.

## 2. Xử lý vật thể nhỏ < 20 px và cụm dày đặc (nguồn: `cau_hoi_mo`)
- **Quy tắc:** Không vẽ bounding box rời rạc cho từng người/xe ở hậu cảnh có chiều cao `< 20 px`. Khoanh 1 khung `ignore_region` bao trọn cụm để tránh nhiễu và giảm thời gian duyệt.
- **Ảnh ví dụ:** `img_018.jpg`, `img_027.jpg`, `img_033.jpg`.

## 3. Quy tắc chống thiên kiến tự động hóa cho box lỏng (`loose`) và sót vật (`missing`) (nguồn: `loi_may`)
- **Quy tắc:** Trước khi duyệt từng box, bắt buộc **ẩn lớp nhãn máy** và quét toàn ảnh một lượt để đếm các đối tượng chính; sau đó bật lại nhãn máy để kiểm tra độ khít của viền box (đặc biệt các box bị nới rộng bất thường).
- **Ảnh ví dụ:** `img_013.jpg`, `img_028.jpg`, `img_025.jpg`, `img_004.jpg`.

## 4. Tách người ngồi trên xe và quy ước mask SAM (nguồn: `cau_hoi_mo`)
- **Quy tắc:** Người lái hoặc hành khách nhìn rõ phần đầu/thân (`>= 20 px`) vẫn gán box `person` riêng biệt với box phương tiện. Khi tạo mask SAM cho phương tiện (`three_wheeler` / `motorbike`), dùng điểm âm (`--neg`) để loại bỏ nền hoặc phần không thuộc thân xe.
- **Ảnh ví dụ:** `img_028.jpg`, `img_013.jpg`, `img_004.jpg`.

## 5. Phân biệt "Guideline mơ hồ" và "Lỗi máy / Người duyệt sai"
- **Guideline mơ hồ (`cau_hoi_mo`):** Cần cập nhật quy tắc rõ ràng trong v1.1 (ví dụ: ngưỡng `< 20 px` gom vào `ignore_region` ở `img_018.jpg`, `img_027.jpg`; tách người ngồi trên xe ở `img_028.jpg`).
- **Lỗi máy / Người duyệt sai (`loi_may`):** Quy tắc v1.0 đã có nhưng model nhận nhầm (`three_wheeler` → `car`/`truck` ở `img_024.jpg`) hoặc người duyệt bị thiên kiến tự động hóa bỏ sót bẫy `loose`/`missing` (`img_013.jpg`, `img_004.jpg`). Xử lý bằng quy trình review 4 bước chứ không thay đổi định nghĩa nhãn gốc.

## 6. Lịch sử thay đổi
| Phiên bản | Ngày | Nội dung thay đổi |
|---|---|---|
| `v1.0` | Trước buổi học | Label contract ban đầu (7 lớp phương tiện/người + `ignore_region`) |
| `v1.1` | 2026-11-19 | Bổ sung 4 quy tắc calibration (`C01–C04`) từ `calibration_decisions.csv` kèm ảnh ví dụ `img_0xx` |
