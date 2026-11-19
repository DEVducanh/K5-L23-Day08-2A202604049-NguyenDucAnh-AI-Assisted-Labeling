# Báo cáo Day 08 — 2A202604049 Nguyen Duc Anh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.50 | Recall 100% trên 9 vật gold; giữ ngưỡng ưu tiên recall (r05) |
| motorbike | YOLO26m | 0.35 | Recall 100% trên 8 vật gold (r05) |
| car | YOLO26m | mặc định 0.30 | Không có ngưỡng ≤0.50 đạt recall 90%; tối đa 86% trên 7 vật gold nên dùng mặc định, không ép ngưỡng (r05) |
| bicycle / bus / truck | YOLO26m | mặc định 0.30 | Dưới 5 vật gold hoặc không có vật gold; dữ liệu không đủ để chọn ngưỡng riêng (r05) |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | 0.15 | Recall 100%, precision 78% trên 7 vật gold; kiểm tra lớp thủ công vì prompt gộp tuk-tuk, xích lô và ba gác (r06) |

- Lệnh `merge_prelabels.py`: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.5,motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`.
- **SAM chọn chính:** SAM 2.1 b; trên CPU mất khoảng 9.15 giây/ảnh end-to-end trong 3 ảnh thử. MobileSAM nhanh hơn (5.73 giây/ảnh) nhưng overlay rộng hơn; chọn SAM 2.1 b khi cần biên gọn hơn. Không có ground truth để so IoU.
- Lỗi không sửa được chỉ bằng đổi ngưỡng: tuk-tuk có thể bị gán `car` (img_004, img_013); cần sửa lớp theo taxonomy thành `three_wheeler`, không chỉ tăng/giảm confidence.
- Ví dụ box sai lớp lọt review: `img_023.jpg` có bẫy `wrong_class`. Báo cáo bẫy không lưu confidence của box nên không gán điểm cao/thấp cụ thể; kết luận là không dùng `conf` thay cho kiểm tra lớp.
- Giới hạn: ngưỡng được chọn trên cùng 4 ảnh gold dùng để chấm. Precision / recall gold (32 TP, 27 FP, 1 FN trên 33 vật gold) có thể lạc quan và chỉ là ước lượng thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 400 | 93.5% | 428 box máy |
| edited | 8 | 1.9% | 428 box máy |
| deleted | 20 | 4.7% | 428 box máy |
| added | 19 | 4.4% | 427 đối tượng sau duyệt |

- Lớp cần sửa nhiều nhất là `three_wheeler`: 15 box bị xóa và 5 box bị sửa. Bảng thống kê không ghi ảnh cụ thể nên chưa kết luận được nguyên nhân theo từng ảnh.
- So với gold, máy có precision **54.2%** (32 TP / 59 dự đoán) và recall **97.0%** (32 / 33 đối tượng gold), theo `qc/review_stats.md`. Mẫu gold chỉ có 4 ảnh.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được **10/12 bẫy (83.3%)**, đạt ngưỡng 80%. Còn lọt `img_008.jpg` (bbox lỏng) và `img_023.jpg` (sai lớp). Thống kê không cho biết bước checklist nào bị bỏ qua.
- Accepted đạt 93.5% nhưng vẫn lọt hai bẫy; tỷ lệ giữ nguyên box máy không tự chứng minh rằng review đã bắt hết lỗi. Cần kiểm tra riêng độ chặt và lớp của từng box.

## 4. Máy có đáng dùng không?

- Thời gian từ `time_log.csv`: gán tay 20 phút / 4 ảnh = **5 phút/ảnh**; review 30 phút / 24 ảnh = **1.25 phút/ảnh**. Đây là so sánh tham khảo vì hai lô ảnh khác nhau; việc gán gold trước cũng giúp quen lớp nhãn và quy tắc.
- Kết luận: **có, dùng làm pre-label để tiết kiệm thời gian nhưng phải review toàn bộ**. Trên gold, recall 97.0% nhưng precision 54.2%; trong batch, 400/428 box được giữ, đồng thời có 20 box bị xóa, 8 sửa và 19 đối tượng được thêm.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn toàn bộ box máy và quét ảnh trước khi hiện nhãn; áp dụng cho đủ 24/24 ảnh.
  2. Với mỗi ảnh, kiểm tra lần lượt vật bị sót, box thừa, sai lớp và box lỏng; dùng `ignore_region` cho cụm vật nhỏ dưới 20 px.
  3. Sau mỗi lô review, đối chiếu export với checklist và chạy lại `review_stats`; xem riêng các bẫy lọt trước khi chốt.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_004.jpg` | SAM 2.1 b vs MobileSAM | 6 mỗi bản (4 dương, 2 âm) | Bao được tuk-tuk xanh; loại người đứng cạnh ra; không thấy hàng hoá cần tách | MobileSAM rộng hơn một chút (6.9% ảnh vs 6.1%); kính/cabin cần rà lại | `tuktuk_img004_sam21.jpg`, `tuktuk_img004_mobile.jpg` |
| `img_013.jpg` | SAM 2.1 b vs MobileSAM | 12 mỗi bản (5 dương, 7 âm) | Xe được tô mask; click âm đặt trên tài xế để tách người khỏi xe; khoang hành khách/hàng hoá cần kiểm tra riêng | MobileSAM mask rộng hơn (33.6% vs 26.8%); nan khung và vùng che khuất còn lỗ/nhiễu | `tuktuk_img013_sam21.jpg`, `tuktuk_img013_mobile.jpg` |
| `img_017.jpg` | SAM 2.1 b vs MobileSAM | 8 mỗi bản (4 dương, 4 âm) | Tuk-tuk phía sau người đi bộ; click âm đặt trên người để giữ họ ngoài mask; không thấy hàng hoá cần tách | Vùng chồng lấp và nan khung còn khó; MobileSAM rộng hơn nhẹ (7.7% vs 7.5%) | `tuktuk_img017_sam21.jpg`, `tuktuk_img017_mobile.jpg` |

- Đối chiếu guideline 4.2: điểm dương đặt trên thân xe, điểm âm trên tài xế/người chồng lấp; dùng nhiều click và box để gợi ý vùng xe. SAM 2.1 cho mask gọn hơn trong ba ảnh này; MobileSAM nhanh hơn nhưng diện tích mask rộng hơn. Cả hai vẫn cần rà biên nhìn thấy, khoảng trống lớn của khung và người/hàng hoá trước khi coi là nhãn cuối. So sánh bằng quan sát overlay, chưa có ground truth để tính IoU.
- Interactor SAM trong CVAT: chưa thử; tạo mask offline bằng `scripts/sam2_points.py`.

## 6. Calibration (Part 6)

- Có **8 quyết định** trong `qc/calibration_decisions.csv`, gồm lỗi máy, câu hỏi mở và hai bẫy lọt. Quan trọng nhất: `CAL-01` chốt tuk-tuk là `three_wheeler`; `CAL-05` và `CAL-06` xử lý box lỏng và sai lớp lọt review.
- `guidelines/guideline_v1.1_calibrated.md` bổ sung cách xử lý cụm xe máy, hàng gắn/rời, người trong xe có mui, kính xe, `conf` và kiểm lại bốn cạnh box. Câu hỏi xe điện và ảnh ban đêm còn mở vì batch này không có ví dụ đủ rõ.
- Nguyên nhân gốc là vừa có **nhầm lớp của máy** (tuk-tuk/car), vừa có **bước review chưa bắt hết lỗi** (`img_008`, `img_023`); các ca mơ hồ được viết thành quy tắc để giảm bất đồng lần sau.

## 7. QC và quyết định (Part 6)

- Mẫu cuối: **5 ảnh** (0 `can_review` + 5 ngẫu nhiên, seed 42), **88 đối tượng**; ảnh: `img_003`, `img_006`, `img_014`, `img_031`, `img_035`. Người kiểm: Codex hỗ trợ đối chiếu overlay và ảnh gốc; giới hạn là không phải reviewer độc lập.
- Lượt đầu có **6 lỗi / 92 đối tượng = 6.5%** trên **3/5 ảnh = 60%**: box thừa ở `img_003`, box quá nhỏ ở `img_006`, xe xích lô bị chia box ở `img_035`. Đã sửa 3 ảnh qua `labels/qc_fixed.json`, dựng lại release và kiểm tra lại cùng mẫu. Lượt cuối: **0/88 đối tượng = 0%** và **0/5 ảnh = 0%** (`qc/qc_report_auto.md`).
- Sau sửa, không còn loại lỗi rõ nào trong mẫu cuối; lỗi phát hiện ban đầu là **thừa** (3 box) và **sai vị trí/kích thước** (3 box).
- **Giới hạn mẫu nhỏ:** dù 0/5 ảnh lỗi, cận trên Wilson 95% là **43%**; mẫu nhỏ không thể chứng minh tỷ lệ thật ≤2%. Mẫu cũng có 0 ảnh `can_review`, nên không đại diện cho ca khó nhất.
- Script kiểm được cấu trúc, lớp đã khai báo, độ lệch nhãn và số đo; không phát hiện vật bị bỏ sót mà cả model và người cùng bỏ, không đánh giá được độ ôm sát nếu không xem ảnh, và không chứng minh người review đã theo checklist.

**Quyết định: RELEASE**, ngày **2026-11-19**. Lý do: sau rework, mẫu cuối 0/5 ảnh có lỗi và 0/88 đối tượng có lỗi; bắt bẫy 10/12 = 83.3% (đạt ngưỡng 80%). Release có giới hạn rõ: n=5 cho cận trên Wilson 43%, nên chưa chứng minh được tỷ lệ thật dưới ngưỡng 2%.
Không còn lỗi quan sát được trong mẫu cuối; tiếp tục lấy mẫu lớn hơn trước khi dùng lô này như một release có bảo đảm thống kê.

## 8. Ghi chú cá nhân

- **Dùng AI:** Codex hỗ trợ chạy script SAM/QC, xem overlay, sửa các lỗi QC đã xác định và soạn phần diễn giải; quyết định lớp/mask được đối chiếu với ảnh gốc và guideline. QC sample do Codex hỗ trợ kiểm bằng mắt, không phải reviewer độc lập.
- Gold có bị nhiễm không: **không chắc**; không có ghi chép thời điểm đủ để xác nhận gold đã hoàn tất trước khi xem pre-label. Vì vậy, số đo gold cần được đọc thận trọng.
- Máy / GPU / CVAT: Windows, PyTorch CPU-only (CUDA không khả dụng), CVAT Docker v2.81.0. Đã gặp lỗi mount vì `grafana_conf.yml` là thư mục thay vì file; khôi phục file cấu hình đã giải quyết lỗi.
- Có dùng cache `data/cache/` làm nguồn dự đoán ở một số bước; merge, review, thống kê và QC chạy trên workspace này.
- Lỗi / bug đã gặp: Compose báo mount file/directory không khớp; sau đó nhận diện đúng thư mục sai và khôi phục config.
- Lần sau sẽ ghi thời điểm hoàn tất gold và provenance từng file dự đoán ngay trong `model_log.csv`, đồng thời nhờ reviewer thứ hai kiểm tra QC.
