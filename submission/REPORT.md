# Báo cáo Day 08 — 2A202603112 - Nguyen Han Truong Giang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n | Mặc định; không chọn được ngưỡng riêng đạt recall mục tiêu 90% trên gold | `r01` tạo 120 box/8 ảnh ở conf 0.25 (0.32 giây/ảnh); `r02` tạo 54 box/8 ảnh ở conf 0.50 (0.24 giây/ảnh). Giữ YOLO26n để tạo gợi ý detection và đối chiếu ảnh hưởng của confidence; không chọn model chỉ vì số box nhiều. |
| three_wheeler | YOLOE-26s-seg, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | Mặc định khi merge; `conf 0.3` ở lệnh merge | `r03` tạo 5 box/8 ảnh pilot (0.14 giây/ảnh); lần chạy toàn bộ tạo 116 box/36 ảnh ở conf 0.05 (0.18 giây/ảnh). YOLOE được dùng để gợi ý lớp địa phương mà model COCO thường nhầm với car, truck hoặc motorbike. Cần người review vì prompt không bảo đảm đúng lớp. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):

```powershell
& $py .\scripts\merge_prelabels.py .\work\prelabel\yoloe_all\pre.coco.json .\work\prelabel\y26_all\pre.coco.json --thr= --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn MobileSAM (`mobile_sam.pt`, thuộc dòng SAM-1 nhẹ) chạy CPU. `r04` cho 10 mask/10 box trên 2 ảnh, thời gian 1.79 giây/ảnh. Máy đang dùng PyTorch bản CPU (`2.15.0+cpu`, CUDA=False), nên chọn MobileSAM để thử trên máy hiện tại. Không dùng SAM 3 vì quyền truy cập checkpoint bị giới hạn; SAM 2.1 có checkpoint trong thư mục models nhưng ba mask đã nộp được tạo bằng MobileSAM.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Nhầm ngữ nghĩa giữa `three_wheeler` với `car` / `truck` / `motorbike`, hoặc vật ngoài taxonomy như `umbrella`, `traffic light` bị loại khỏi ánh xạ, không thể sửa chỉ bằng tăng/giảm confidence. Cách xử lý là dùng YOLOE với prompt riêng cho `three_wheeler`, kiểm tra lớp bằng đặc điểm phương tiện và sửa nhãn thủ công khi cần.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Không có ví dụ confidence cá thể được lưu trong `model_log.csv`/`run.json` đã cung cấp; log hiện có chỉ ghi tổng số box, thời gian và lớp bị loại, nên không kết luận hoặc tự nêu một ảnh cụ thể.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: `threshold_sweep` dùng 4 ảnh gold để vừa chọn vừa đánh giá, vì vậy precision/recall trên gold là in-sample và có xu hướng lạc quan. Mẫu lớp nhỏ cũng yếu: bicycle có 4 vật, bus 0, truck 1. Không lớp nào có đủ căn cứ để chọn ngưỡng theo lớp đạt recall 90%; vì vậy các lớp giữ mặc định, không diễn giải kết quả này như đánh giá độc lập.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 182 | 79.1% | 230 box máy |
| edited | 28 | 12.2% | 230 box máy |
| deleted | 18 | 7.8% | 230 box máy |
| added | 121 | 36.4% | 332 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Xét số đối tượng phải bổ sung sau khi soát, `person` có 48 added và `motorbike` có 43 added (bảng theo lớp trong `review_stats.md`). Ví dụ rõ nhất là `img_018.jpg`: phân tích đối chiếu ghi nhận pre-label bỏ sót 24 person và 19 motorbike có chiều cao box từ 20 px trở lên. Đây là bằng chứng model bỏ sót nhiều vật thể, không thể chỉ nhìn các box máy đã tạo. Báo cáo không có bảng precision/recall riêng theo lớp nên không kết luận một lớp có tỷ lệ lỗi tổng thể cao nhất.
- So với gold: precision / recall của **máy** là precision 27/35 = 77.1% và recall 27/91 = 29.7% (TP=27, FP=8, FN=64; mẫu số recall là 91 đối tượng gold trong 4 ảnh; nguồn `review_stats.md`). Đây là kết quả in-sample trên gold đã dùng trong bước threshold sweep.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3%; loại còn lọt: `wrong_class` ở `img_017.jpg` và `missing` ở `img_021.jpg` (nguồn `review_stats.md`, mẫu số 12 bẫy). Hai lỗi này cho thấy khi review vẫn có thể tin box có sẵn mà không kiểm tra lại lớp độc lập, hoặc không quét vùng không có box để tìm vật bị bỏ sót. Ở lần review đầu bắt 8/12 = 66.7%; sau khi sửa các lỗi đã phát hiện và review lại, kết quả cập nhật là 10/12.
- Accepted 182/230 = 79.1% không có nghĩa là toàn bộ pre-label đúng: đây là tỷ lệ box máy được giữ nguyên theo tiêu chí ghép IoU, còn bài test bẫy đo khả năng phát hiện lỗi được cài có chủ ý. Kết quả 10/12 = 83.3% đạt ngưỡng nhưng vẫn còn hai bẫy lọt; vì vậy phải vừa đối chiếu hình học vừa kiểm tra lớp và rà toàn ảnh, không chỉ dựa vào số box accepted.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): 30 phút cho gold và 100 phút cho batch.
- Kết luận: **Có, nhưng chỉ dùng làm công cụ gợi ý để người kiểm duyệt sửa và bổ sung, không dùng tự động để chốt nhãn.** Căn cứ: accepted 182/230 = 79.1%, edited 28/230 = 12.2%, deleted 18/230 = 7.8%; đồng thời có 121 đối tượng added/332 đối tượng sau duyệt = 36.4%. Trên 4 ảnh gold, recall máy chỉ 27/91 = 29.7%, cho thấy còn bỏ sót nhiều vật thể.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Trước khi xem box máy, quét toàn bộ ảnh theo thứ tự trái sang phải và trên xuống dưới; sau đó mới đối chiếu pre-label. Khi review, ghi riêng số vật thể added để theo dõi bỏ sót.
  2. Kiểm tra class độc lập với vị trí box: đối chiếu số bánh và cấu trúc xe; không chắc thì đánh dấu `can_review`, không giữ class chỉ vì box nằm đúng chỗ.
  3. Kiểm tra từng box theo mép vật nhìn thấy; box cao dưới 20 px không gán instance riêng theo guideline, và cụm nhỏ không tách được thì xem xét `ignore_region`. Ghi lại mọi box xóa, sửa và thêm vào phiếu review.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---:|---|---|---|
| `img_013.jpg` (tuk-tuk) | MobileSAM (SAM-1 family) | 3 (2 dương, 1 âm) | Không gồm người lái; không có ghi nhận riêng về hàng hóa trong ca này. | Mask phủ mái và thân xe nhưng mép bánh sau và bậc phía trước chưa gọn; mask chiếm 245,541 px = 35.2% ảnh, có 2 polygon. | `tuktuk_img013.jpg` |
| `img_032.jpg` (cyclo) | MobileSAM (SAM-1 family) | 7 (4 dương, 3 âm ở lần chỉnh cuối) | Mask tập trung vào xe xích lô; người đứng cạnh bị loại. Không có ghi nhận riêng về hàng hóa. | Lần đầu click sai làm mask lấy cả mặt đường/bóng/nền; sau chỉnh, mask bám ghế, khung và hai bánh nhưng chi tiết khung nhỏ chưa hoàn hảo. Mask cuối chiếm khoảng 6.5% ảnh. | `cyclo_img032.jpg` |
| `img_023.jpg` (tuk-tuk bị che) | MobileSAM (SAM-1 family) | 6 (3 dương, 3 âm) | Không gồm người mặc áo vàng đứng cạnh; không có ghi nhận riêng về hàng hóa. | Chỉ lấy phần mái/thân xe nhìn thấy; không thể khôi phục phần bị che, một số bánh/chi tiết nhỏ bị thiếu. | `occluded_tuktuk_img023.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc hữu ích là polygon bám đường biên phần nhìn thấy, không lấy bóng đổ, và không cố đoán phần bị che. Quy tắc lỗ nhỏ dưới khoảng 5% giúp tránh vẽ quá nhiều chi tiết khung. Điểm guideline chưa nói đủ rõ là mask phương tiện có bao gồm người lái/hành khách hay hàng hóa hay không; trong các thử nghiệm này mask hướng tới phần xe, người được xem là đối tượng riêng, còn hàng hóa không có ghi nhận đủ để kết luận.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không có kết quả thử Interactor được ghi lại. Các mask được tạo offline bằng `scripts/sam2_points.py` với click dương/âm và checkpoint `mobile_sam.pt`; vì vậy không khẳng định Interactor trong CVAT đã chạy thành công hay so sánh chất lượng hai cách.

## 6. Calibration (Part 6)

- Số quyết định: 3 case được đưa vào bản guideline v1.1 (`CAL-01` đến `CAL-03`); nội dung tương ứng là rà vật bị pre-label bỏ sót ở `img_018`, dùng `ignore_region` cho cụm xe máy dày ở `img_005`, và kiểm tra class độc lập ở `img_017`/`img_003`. Lưu ý: bản `calibration_decisions.csv` trên GitHub được kiểm tra chỉ có dòng tiêu đề; cần bảo đảm bản CSV local có đủ ba dòng này trước khi nộp.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: (1) phải rà toàn ảnh, không coi vùng không có box máy là vùng không có vật thể (`CAL-01`); (2) cụm xe máy dày đặc gần camera đã xác định là không thể tách cá thể thì dùng `ignore_region` (`CAL-02`); (3) xác nhận class dựa trên đặc điểm phương tiện, không chỉ dựa vào vị trí box; nếu chưa chắc thì dùng `can_review` (`CAL-03`).
- Nguyên nhân gốc: kết hợp cả ba yếu tố. Model bỏ sót vật thể và nhầm lớp; guideline v1.0 còn để mở một số tình huống như cụm xe máy dày đặc; người review có lúc dựa vào box máy có sẵn nên chưa quét toàn ảnh hoặc chưa xác nhận class độc lập.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 44); người kiểm: bản thân (**hạn chế**: người kiểm cũng là người đã gán nhãn và review trước đó). Nguồn: `qc_sample.csv`; mẫu có 62 đối tượng.
- Lỗi theo **đối tượng**: 0/62 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%). Nguồn: `qc_report_auto.md`.
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: vòng QC cuối không ghi nhận lỗi nào trong 5 ảnh mẫu. Lịch sử Rework trước đó: lỗi thừa ở `img_003.jpg`, `img_031.jpg`, `img_035.jpg`; lỗi sai lớp ở `img_003.jpg` và `img_024.jpg`. Các ảnh đã được sửa trong `qc_fixed.json` và Release manifest ghi nhận 4 ảnh thay thế.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): với 0/5 ảnh lỗi, cận trên Wilson 95% là 43%. Mẫu 5 ảnh quá nhỏ để chứng minh tỷ lệ lỗi thật của toàn bộ lô 24 ảnh dưới 2%; kết luận Release là theo quy tắc quyết định của lab trên mẫu đã kiểm, không phải chứng minh thống kê cho toàn bộ lô.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script kiểm cấu trúc COCO, taxonomy, số ảnh/đối tượng, một số quy tắc kích thước, IoU để đối chiếu box, thống kê accepted/edited/deleted/added và đếm các bẫy đã bắt. Script không tự biết box có ôm sát vật thật hay không nếu không có người xem ảnh; không phát hiện được vật thể mà cả máy và người cùng bỏ sót; không xác nhận người kiểm có thực hiện đúng checklist hoặc class có đúng ngữ nghĩa ngoài đời hay không.

**Quyết định: Release**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không có dấu hiệu phải Hold vì dữ liệu và taxonomy đã qua kiểm tra cấu trúc; các lỗi phát hiện ở vòng QC đầu và vòng hai đã được sửa qua Rework, gồm 3 box thừa/sai lớp trong các ảnh `img_003`, `img_031`, `img_035` và lỗi `bus` đổi thành `car` trong `img_024`; Release vòng ba có 0/5 ảnh lỗi và 0/62 đối tượng lỗi, đồng thời bắt bẫy 10/12 = 83.3%, đạt ngưỡng tối thiểu 80%. Quyết định này vẫn có giới hạn do mẫu nhỏ (cận trên Wilson 43%).
Nếu Rework / Hold, việc cần làm tiếp: không còn Rework/Hold theo kết quả QC cuối; trước khi nộp cần chạy `check_submission.py`, đồng bộ `calibration_decisions.csv` và các báo cáo local lên repo, và kiểm tra lại các cảnh báo box dưới 20 px trong `batch_reviewed.json` theo quy tắc `ignore_region`/vật quá nhỏ.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Có dùng ChatGPT để đọc/giải thích yêu cầu repo, hướng dẫn lệnh PowerShell, xử lý lỗi tham số, phân tích log QC và soạn nội dung guideline/báo cáo. Các lệnh được chạy trong môi trường local; kết quả được đối chiếu bằng output terminal, file COCO/CSV/JSON, ảnh preview và review trực tiếp trong CVAT. AI không tự quyết định nhãn cuối thay người gán.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Theo trình tự log, `gold_manual.json` đã được tạo và chạy `validate_labels.py` trước các lần chạy `prelabel.py`, nên không thấy dấu hiệu gold được tạo từ pre-label. Tuy nhiên, không có time log độc lập xác nhận thời điểm hoàn tất từng ảnh hoặc xác nhận riêng việc không hỏi AI trước khi gán xong; vì vậy đây là kết luận theo thứ tự thao tác đã ghi.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows PowerShell; Python 3.12.10 trong `D:\day08\.venv`; PyTorch 2.15.0+cpu, CUDA=False; Ultralytics 8.4.215; Transformers 5.23.0; dùng CPU. CVAT dùng để tạo task, import/export COCO 1.0 và review nhãn. Ban đầu thiếu `transformers` (đã cài và `check_env.py` chuyển sang SẴN SÀNG); không có GPU khả dụng trong môi trường PyTorch CPU. SAM 3 không dùng do checkpoint Hugging Face bị giới hạn quyền truy cập.
- Dùng cache `data/cache/` thay vì tự chạy? Không có ghi nhận dùng cache; các lần prelabel được chạy trực tiếp bằng `scripts/prelabel.py` trên pilot hoặc toàn bộ 36 ảnh, với model trong `models/`.
- Lỗi / bug script đã gặp (nếu có): `merge_prelabels.py --thr ""` báo `argument --thr: expected one argument`; đã sửa cú pháp thành `--thr=` và lệnh merge chạy thành công. `prelabel.py` cảnh báo các tên lớp ngoài ánh xạ như umbrella/traffic light/backpack/stop sign bị loại; cần đọc cảnh báo này khi review. YOLOE cũng in `FutureWarning` về `torch.jit.load`, nhưng lần chạy vẫn hoàn tất.
- Một điều bạn làm khác lần sau: ghi `time_log.csv` ngay sau từng phần để có số phút/ảnh; chốt rõ việc người lái/hàng hóa có thuộc mask hay không; sao lưu phiếu và báo cáo từng vòng QC; luôn quét toàn ảnh và kiểm tra class độc lập trước khi chấp nhận pre-label.
