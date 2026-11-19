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
