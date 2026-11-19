# Báo cáo Day 08 — 2A202603351 - Ngueyn Ho Bac


## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / motorbike / car | YOLO26m (`r01`) | mặc định `0.3` | Không ngưỡng nào `<= 0.5` đạt recall 90% trên 4 ảnh gold: person tối đa 85% (27 vật), motorbike tối đa 78% (23 vật), car tối đa 89% (9 vật). Giữ mặc định, không ép tăng ngưỡng vì sẽ tăng bỏ sót. |
| bicycle | YOLO26m (`r01`) | `0.2` | Đạt recall 100% và precision 71% trên 5 vật gold; đây là ngưỡng cao nhất còn đạt mục tiêu recall 90%. |
| bus / truck | YOLO26m (`r01`) | mặc định `0.3` | Gold có 0 vật mỗi lớp, dưới ngưỡng 5 vật để chọn riêng; giữ mặc định theo quy định, không suy luận từ mẫu rỗng. |
| three_wheeler | YOLOE-s (`r03`, prompt `three_wheeler`) | `0.15` | Đạt recall 100% và precision 78% trên 7 vật gold; đây là ngưỡng cao nhất còn đạt mục tiêu recall 90%. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 Base (`sam2.1_b.pt`) cho chất lượng mask tốt hơn MobileSAM trên vật bị che; tốc độ CPU chậm hơn nhưng chỉ làm 3 vật nên chấp nhận được, không cần quyền truy cập gated của SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Nhầm lớp tuk-tuk/xe ba bánh không thể sửa chỉ bằng ngưỡng; dùng YOLOE với prompt `three_wheeler`, gộp kết quả và review lại lớp theo guideline.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): `img_002.jpg` có false positive xe đẩy hàng bị gọi `three_wheeler`; `img_007.jpg` và `img_032.jpg` có xích lô bị bỏ sót. Log hiện không lưu đủ từng confidence để kết luận chắc hơn về cặp điểm cao/điểm thấp.
- Giới hạn: chỉ có 4 ảnh gold. Ngưỡng được chọn và precision/recall của máy đều được tính trên cùng dữ liệu gold, nên các số đo này lạc quan; các lớp ít vật có kết luận rất thô và cần pilot độc lập để xác nhận.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 496 | 95.4% | 520 box máy |
| edited | 6 | 1.2% | 520 box máy |
| deleted | 18 | 3.5% | 520 box máy |
| added | 26 | 4.9% | 528 đối tượng sau duyệt |

- Lớp máy tệ nhất là `three_wheeler`: có 11 box bị xoá và chỉ 27 box được giữ nguyên (bảng theo lớp trong `review_stats.md`), cho thấy nhận diện/ánh xạ lớp này kém ổn định hơn các lớp còn lại. File thống kê không gắn từng lỗi với tên ảnh nên không thể dẫn ảnh cụ thể mà không suy diễn.
- So với gold: precision của **máy** 63.5% và recall 66.2% (TP=47, FP=27, FN=24; mẫu số recall: 71 đối tượng gold). Hai trường hợp trong deleted + added là đổi lớp, không phải bỏ sót thật; số sót thật xấp xỉ 24.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3% (KTC 95%: 55–95%, mẫu chỉ 12 bẫy). Loại lọt: `loose` 2/3, cụ thể `img_013.jpg` và `img_009.jpg`; các loại `missing`, `wrong_class`, `extra` đều bắt được 3/3. Nguyên nhân là bước kiểm tra box có ôm sát vật thể chưa được thực hiện đủ nghiêm, và cần quét ảnh trước khi nhìn nhãn máy để tránh bị neo vào box có sẵn.
- Accepted cao nhưng bắt bẫy chỉ vừa vượt ngưỡng (83.3%) cho thấy tôi thường tin các box máy có vẻ hợp lý, nhưng vẫn bỏ sót lỗi hình học tinh tế. Vì vậy accepted cao không đồng nghĩa với kiểm tra độc lập tốt; cần ưu tiên checklist và xem toàn ảnh, nhất là độ ôm sát.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): chưa tính được gán tay và duyệt vì file hiện chỉ có dòng tiêu đề, chưa có bản ghi phút/ảnh. Do đó chưa thể kết luận máy tiết kiệm thời gian hơn gán tay; cần ghi lại hai thời lượng trên cùng số ảnh ở một pilot độc lập. Thiên lệch cần lưu ý: gold được gán trước nên tôi đã quen cảnh.
- Kết luận: **có, nhưng chỉ như pre-label có người duyệt**, không dùng tự động. Lý do: 496/520 box được accepted (95.4%), nhưng precision/recall trên 71 vật gold chỉ là 63.5%/66.2%, còn 24 vật bị sót thật và bắt bẫy chỉ 10/12. Chưa đủ dữ liệu thời gian để khẳng định hiệu quả năng suất.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Trước khi xem nhãn máy, quét toàn ảnh và ghi số vật thể theo từng lớp; kiểm tra được bằng checklist có mục “quét mù” cho 24/24 ảnh.
  2. Với mọi box được giữ, kiểm tra riêng bốn lỗi `missing`, `wrong_class`, `extra`, `loose`; không tính accepted nếu chưa đánh dấu đủ 4 mục, và báo cáo tỷ lệ bắt bẫy theo từng loại.
  3. Đo năng suất bằng cùng một mẫu ảnh độc lập: ghi phút/ảnh cho gán tay và duyệt máy, đồng thời lấy mẫu ngẫu nhiên ít nhất 20% box accepted để kiểm tra lại; chỉ kết luận dùng máy nếu thời gian giảm mà recall/bắt bẫy vẫn đạt ngưỡng.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_002.jpg` | SAM 2.1 Base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Không gồm người lái; không thấy hàng hoá nhô ra khỏi xe trong vùng mask | Xe ba bánh ở giữa nhiều xe gần nhau nên biên phía trái/phía sau còn hơi dính nền và vật phía sau | `masks/tuktuk_img002.jpg` |
| `img_004.jpg` | SAM 2.1 Base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Không gồm người lái; không gồm hàng hoá rời khỏi thân xe | Mask bám khá tốt xe màu hồng nhưng còn lẫn một ít vùng tối/khoảng nền trong phần khung hở | `masks/tuktuk_img004.jpg` |
| `img_013.jpg` | SAM 2.1 Base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Có dính một phần người lái ở vùng tay/chân; hàng nước phía sau không thuộc xe và đã loại phần lớn | Người lái bị che bởi mui và khung nên một điểm âm chưa tách hết người; mask cũng hơi lấn vào khoảng trống trong cabin | `masks/tuktuk_img013.jpg` |

- Đối chiếu guideline mục 4.2: đã áp dụng quy tắc 1 (bám biên phần nhìn thấy), quy tắc 2 (không cần giữ lỗ nhỏ trong khung), quy tắc 3 (bóng đổ không thuộc vật) và quy tắc 4 (các vật chạm nhau phải có polygon riêng, không chồng). Quy tắc 4.2 chưa đủ để tách người lái bị mui che; cần thêm điểm âm hoặc tách người lái thành một polygon `person` riêng. Hàng hoá chỉ được giữ trong mask xe nếu gắn cố định; hàng nhô ra phải tách khỏi mask xe.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thực hiện được vì CVAT Docker chưa triển khai serverless/AI Tools nên không có model Interactor trong menu Models. Đã dùng `scripts/sam2_points.py` offline để mô phỏng cùng thao tác click. Offline cho phép ghi lại chính xác phiên bản, tọa độ và số click, còn Interactor CVAT cho phản hồi mask ngay trên giao diện và có thể sửa tiếp bằng click; chất lượng phụ thuộc cùng model và các điểm đã chọn.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`qc/calibration_decisions.csv`); quan trọng nhất: `CAL-01` chốt tuk-tuk/xích lô là `three_wheeler` dù YOLO26 gọi sai lớp.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: hàng gắn cố định thuộc xe nhưng hàng nhô ra hơn khoảng 20% phải tách; tuk-tuk có mui vẫn là `three_wheeler`; phải kiểm `loose` độc lập với `conf`.
- Nguyên nhân gốc: guideline v1.0 còn mở về hàng hóa; máy nhầm lớp do taxonomy COCO không có `three_wheeler`; người duyệt bị thiên kiến bởi box có sẵn nên bỏ lọt `loose`.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42), 112 đối tượng; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 0/112 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%).
- Top lỗi: không ghi nhận lỗi trong mẫu (`khong` ở cả 5/5 ảnh).
- **Giới hạn mẫu nhỏ**: `qc_report_auto.md` ghi cận trên Wilson 95% là 43% cho tỷ lệ ảnh lỗi thật; 0/5 không chứng minh được ngưỡng 2%. Mẫu cũng không có ảnh `can_review`, nên không đại diện cho các ca khó.
- Script kiểm được tên file, lớp, box trong ảnh, số ảnh, các thống kê và phiếu QC; không kiểm được box có ôm sát vật thật, trường hợp cả người và máy cùng bỏ sót, hay việc tuân thủ checklist.

**Quyết định: Release**, ngày 2026-11-19. Lý do: mẫu QC không có lỗi quan sát được (0/5 ảnh), guideline đã chốt 3 ca calibration, và bắt bẫy đạt 10/12 = 83.3% (trên ngưỡng 80%); kết luận Release có giới hạn lớn do mẫu QC chỉ 5 ảnh và cận trên 95% vẫn là 43%.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Có dùng trợ lý AI để đọc guideline, kiểm tra file và soạn phần báo cáo; kết quả được đối chiếu bằng ảnh mask, script `qc_report.py` và `check_submission.py`.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không; gold được gán trước khi xem pre-label.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows, chạy CPU; không dùng CVAT Interactor. Lệnh `python` trỏ tới môi trường thiếu pip/numpy nên phải chạy script bằng Python Launcher `py -3.12`.
- Dùng cache `data/cache/` thay vì tự chạy? Có, vì cache có sẵn các kết quả model cần cho báo cáo; các bước release/QC vẫn chạy lại từ nhãn trong `submission/`.
- Lỗi / bug script đã gặp (nếu có): Không có lỗi logic của script; lần chạy đầu thất bại do môi trường `python` thiếu `numpy`, sau đó dùng đúng môi trường Python 3.12 có dependency local.
- Một điều bạn làm khác lần sau: ghi thời gian gán tay/duyệt ngay khi thực hiện và lấy mẫu QC có cả ảnh `can_review`, không chỉ mẫu ngẫu nhiên.
