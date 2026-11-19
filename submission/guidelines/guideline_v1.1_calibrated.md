# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604049 Nguyen Duc Anh

> Chỉ ghi các quyết định bổ sung cho v1.0. Case ID khớp `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Xe máy đỗ thành cụm gần camera: gán từng xe hay `ignore_region`? | Gán từng xe nếu phân biệt được thân/đầu xe. Dùng một `ignore_region` cho cụm từ 5 xe trở lên khi từng xe cao dưới 20 px hoặc không thể tách ranh giới. | `img_006.jpg`, `img_014.jpg` | CAL-02 |
| 3 | Hàng nhô ra khỏi xe đạp/xe ba gác xử lý thế nào? | Hàng/giỏ gắn cố định thuộc xe; hàng rời nhô hơn khoảng 20% chiều dài xe thì không gộp vào box xe. Người bán vẫn là `person` riêng. | `img_031.jpg` | CAL-03 |
| 4 | Người trong xe ba bánh có mui được gán khi nào? | Gán `person` riêng khi nhìn thấy khoảng 20% người trở lên; không nhập người vào nhãn xe. | `img_013.jpg` | CAL-07 |
| 5 | Kính chắn gió/cửa sổ trong suốt có thuộc mask xe không? | Kính gắn liền xe thuộc mask; người nhìn thấy xuyên qua kính vẫn là vật riêng và phải loại khỏi mask xe. | `img_013.jpg` | CAL-04 |
| 7 | Có ngưỡng `conf` chung để chấp nhận box tự động không? | Không. `conf` chỉ giúp sắp thứ tự review; xác nhận cuối dựa vào ảnh, lớp và độ ôm của box. | `img_023.jpg` | CAL-08 |

Hai câu chưa chốt vì không có ví dụ đủ rõ trong lô hiện tại: **#1 xe đạp điện/xe máy điện** và **#6 box chồng lấn trong ảnh ban đêm**. Giữ mở cho tới khi có ảnh phù hợp; không suy diễn quy tắc từ batch ban ngày này.

## 2. Ca khó mới (từ lỗi máy và bẫy lọt)

| Ảnh | Tình huống | Máy / review làm gì | Quyết định và quy tắc bổ sung |
|---|---|---|---|
| `img_004.jpg`, `img_013.jpg` | Tuk-tuk có mui dễ giống ô tô; một xe có thể bị box thành nhiều phần | YOLO26 đã gợi ý sai lớp `car`; review cũng gặp box xe không trọn | Gộp thành một `three_wheeler` cho toàn xe. Nếu số bánh không rõ, bật `can_review`. `CAL-01` |
| `img_008.jpg` | Bẫy `loose` lọt qua review | Lớp có thể đúng nhưng khung quá rộng | Sau khi xác nhận lớp, rà riêng bốn cạnh box và siết theo vật thể nhìn thấy. `CAL-05` |
| `img_023.jpg` | Bẫy `wrong_class` lọt qua review | Box được giữ dù lớp không đúng | Kiểm tra lớp sau khi định vị vật; khi còn nghi ngờ, bật `can_review` và không dựa riêng vào `conf`. `CAL-06`, `CAL-08` |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Chốt quy tắc cụm xe máy, hàng gắn/rời, người trong xe có mui, kính xe và cách dùng `conf`; bổ sung lượt kiểm độ ôm box và lớp | `CAL-01`–`CAL-08`; lỗi quan sát ở `img_008`, `img_023` và quyết định từ các ảnh ví dụ trong batch |
