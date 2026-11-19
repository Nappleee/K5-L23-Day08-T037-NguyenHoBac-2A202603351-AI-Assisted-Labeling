# Thông tin học viên

- Họ và tên: Nguyễn Hồ Bắc
- MSHV: 2A202603351
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: Nappleee
- Repo bài nộp (URL): https://github.com/Nappleee/K5-L23-Day08-T037-NguyenHoBac-2A202603351-AI-Assisted-Labeling
- Commit chốt bài (hash): Chưa chốt
- Máy: Windows; GPU chưa ghi nhận
- CVAT (Docker): Không sử dụng Interactor SAM trong CVAT

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 496 / 6 / 18 (mẫu số: 520 box máy); added: 26 (mẫu số: 528 đối tượng).
- So với gold: precision / recall của **máy** 63.5% / 66.2% (47 TP, 27 FP, 24 FN; 71 đối tượng gold).
- Bắt bẫy: 10/12 = 83.3%.
- Thời gian: gán tay chưa có số liệu; duyệt có pre-label chưa có số liệu (`time_log.csv` chỉ có tiêu đề).
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/112).
- **Quyết định:** Release — mẫu QC 0/5 ảnh lỗi và calibration đã chốt 3 ca; giới hạn mẫu nhỏ được ghi trong `REPORT.md`.
