# Label contract — Phố châu Á — BỔ SUNG v1.1

> Chỉ ghi phần bổ sung so với v1.0. Các quyết định dưới đây khớp với `qc/calibration_decisions.csv`; ảnh ví dụ dùng đúng tên ảnh trong batch.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng hóa trên xe đạp/ba gác có thuộc xe không? | Hàng gắn cố định thuộc mask/box của xe; phần nhô ra khỏi xe hơn khoảng 20% chiều dài xe tách khỏi mask/box xe. | `img_035.jpg` | `CAL-02` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg`, `img_004.jpg` | Tuk-tuk/xích lô có người lái và mui che | YOLO26 gọi tuk-tuk là `car`/`truck`; mask SAM có thể dính người lái | Gán `three_wheeler` cho xe; gán người lái thành `person` riêng khi còn thấy đủ phần người. | Mục 4.1.4 và 4.3: ưu tiên cấu trúc/số bánh nhìn thấy, không giữ nhãn sai của máy. |
| `img_013.jpg`, `img_009.jpg` | Box `loose` lọt qua khi duyệt | YOLO26 tạo box có vẻ hợp lý nhưng không ôm sát phần nhìn thấy | Trước khi chấp nhận phải quét ảnh và kiểm riêng `missing`, `wrong_class`, `extra`, `loose`; `conf` chỉ dùng để ưu tiên duyệt. | Mục 4.1.1 và checklist review: kiểm hình học độc lập với điểm tin cậy. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc hàng hóa; chuẩn hóa xử lý tuk-tuk/xích lô; bắt buộc kiểm box `loose` độc lập với `conf`. | `CAL-01`, `CAL-02`, `CAL-03` |
