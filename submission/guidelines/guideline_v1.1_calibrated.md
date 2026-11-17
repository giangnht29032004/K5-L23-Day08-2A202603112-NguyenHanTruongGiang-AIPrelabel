# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603112 - Nguyen Han Truong Giang

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện: `bicycle` hay `motorbike`? | Chưa chốt. Chưa có ảnh ví dụ và quyết định cụ thể được xác nhận trong phần review đã ghi nhận; không tự đặt quy tắc khi chưa có ca đủ căn cứ. | — | — |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Với cụm xe máy dày đặc đã gặp, chọn `ignore_region`, không tách box riêng từng xe. | `img_005` | `CAL-02` |
| 3 | Hàng nhô ra khỏi xe đạp / xe ba gác: box và mask có gồm hàng không? | Với box, giữ quy tắc v1.0 mục 4.1.5: không tính hàng hóa nếu nhô ra khỏi xe hơn khoảng 20% chiều dài xe. Quyết định riêng cho mask chưa chốt vì chưa có ca ví dụ và quyết định được xác nhận. | — | — |
| 4 | Người ngồi trong xe ba bánh có mui che: ngưỡng nhìn thấy 20% có phù hợp không? | Chưa hiệu chỉnh ngưỡng bằng dữ liệu đánh giá giữa nhiều người gán nhãn. Tạm giữ quy tắc v1.0 mục 4.1.2; chưa thêm ngưỡng mới. | — | — |
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe không? | Giữ quy tắc đang có ở v1.0 mục 4.2.3: kính xe thuộc vật. Chưa có ca calibration riêng để thay đổi quy tắc này. | — | — |
| 6 | Box các phương tiện chồng nhau nhiều trong ảnh đêm: gộp hay giữ riêng? | Chưa chốt thêm quy tắc. Chưa có ca ảnh và quyết định review cụ thể được xác nhận để làm căn cứ thay đổi v1.0. | — | — |
| 7 | Có đặt mức confidence tối thiểu để chấp nhận box máy không cần sửa không? | Không đặt confidence làm tiêu chí chấp nhận độc lập. Kết quả threshold sweep trên 4 ảnh gold không hỗ trợ chọn ngưỡng riêng đạt mục tiêu recall cho các lớp; các lớp đều giữ mặc định. Theo v1.0 mục 3, `conf` chỉ dùng để ưu tiên thứ tự duyệt, không thay việc nhìn ảnh. | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_018` | Pre-label bỏ sót 43 đối tượng có chiều cao box từ 20 px trở lên: 24 `person` và 19 `motorbike`. Bốn đối tượng dưới 20 px được tách riêng, không tính vào 43 đối tượng này. (`CAL-01`) | YOLO26 (`y26_all`, sau đó được đưa vào bộ pre-label hợp nhất) không tạo box cho các đối tượng này. | Bổ sung các đối tượng bị bỏ sót khi review nhãn cuối; không chấp nhận pre-label chỉ vì các box máy hiện có trông hợp lý. | Bổ sung vào mục 1 và mục 4.1: phải rà toàn bộ ảnh để tìm vật bị bỏ sót; vùng không có box máy không được mặc định là không có đối tượng. |
| `img_017`; `img_003` | Hai bẫy `wrong_class` bị lọt qua review: `truck` bị đổi thành `bus` ở `img_017`; `motorbike` bị đổi thành `bicycle` ở `img_003`. (`CAL-03`) | Đây không phải lỗi dự đoán tự nhiên của model: `inject_traps.py` cố ý thay đổi lớp để kiểm tra khả năng phát hiện lỗi khi review. | Ghi nhận hai bẫy sai lớp không được phát hiện. Khi review, cần kiểm tra lớp độc lập với việc box có nằm đúng vị trí; nếu chưa đủ căn cứ phân biệt thì dùng `can_review`. | Bổ sung vào mục 2 và mục 4.3: đối chiếu đặc điểm phương tiện để xác nhận lớp; không chấp nhận nhãn chỉ vì vị trí box đúng. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt dùng `ignore_region` cho cụm xe máy dày đặc ở gần camera; bổ sung yêu cầu rà toàn ảnh để phát hiện vật bị YOLO26 bỏ sót; bổ sung bước kiểm tra lớp độc lập với hình học box. Các câu hỏi chưa đủ dữ liệu được ghi rõ là chưa chốt, không tự đặt quy tắc mới. | `img_005` (`CAL-02`); `img_018` (`CAL-01`); `img_017`, `img_003` (`CAL-03`). |
