# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 230; đối tượng sau duyệt: 332; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 182 | 182/230 = 79.1% | 230 box máy |
| edited (sửa) | 28 | 28/230 = 12.2% | 230 box máy |
| deleted (xoá) | 18 | 18/230 = 7.8% | 230 box máy |
| added (vẽ thêm) | 121 | 121/332 = 36.4% | 332 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 116.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 95 | 12 | 5 | 48 |
| bicycle | 2 | 1 | 0 | 5 |
| motorbike | 45 | 14 | 5 | 43 |
| car | 27 | 0 | 1 | 18 |
| bus | 0 | 0 | 1 | 3 |
| truck | 0 | 0 | 1 | 1 |
| three_wheeler | 13 | 1 | 5 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 91 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 27 | 8 | 64 | 77.1% (KTC 95%: 61–88%) | 29.7% (KTC 95%: 21–40%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - wrong_class: 2/3
  - missing: 2/3
  - extra: 3/3
  - lọt: img_017.jpg (wrong_class), img_021.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
