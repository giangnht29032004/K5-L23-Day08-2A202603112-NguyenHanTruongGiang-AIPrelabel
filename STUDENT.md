# Thông tin học viên

- Họ và tên: Nguyễn Hán Trường Giang
- MSHV: 2A202603112
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: giangnht29032004
- Repo bài nộp (URL): https://github.com/giangnht29032004/K5-L23-Day08-2A202603112-NguyenHanTruongGiang-AIPrelabel
- Commit chốt bài (hash): Chưa có — cập nhật sau khi tạo commit cuối cùng.
- Máy: Windows, CPU (không có GPU)
- CVAT (Docker): phiên bản v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 182/230 = 79.1% / 28/230 = 12.2% / 18/230 = 7.8% (mẫu số: 230 box máy); added: 121/332 = 36.4% (mẫu số: 332 đối tượng sau duyệt).
- So với gold: precision / recall của **máy**: 77.1% (27/35) / 29.7% (27/91).
- Bắt bẫy: 10/12 = 83.3%.
- Thời gian: gán tay 30 phút / 4 ảnh = 7.5 phút/ảnh; duyệt có pre-label 100 phút / 24 ảnh = 4.17 phút/ảnh.
- Lỗi QC: 0% theo ảnh (0/5), 0% theo đối tượng (0/62).
- **Quyết định:** Release — mẫu QC vòng cuối không ghi nhận lỗi (0/5 ảnh, 0/62 đối tượng) và bắt bẫy đạt 10/12 = 83.3%, vượt ngưỡng 80%; tuy nhiên mẫu nhỏ, cận trên Wilson 95% là 43%, nên chưa thể chứng minh tỷ lệ lỗi toàn lô dưới 2%.
