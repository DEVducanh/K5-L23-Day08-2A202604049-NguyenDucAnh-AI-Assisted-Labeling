# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 428; đối tượng sau duyệt: 427; ngưỡng accepted: IoU >= 0.9
- Đã **loại 21 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 400 | 400/428 = 93.5% | 428 box máy |
| edited (sửa) | 8 | 8/428 = 1.9% | 428 box máy |
| deleted (xoá) | 20 | 20/428 = 4.7% | 428 box máy |
| added (vẽ thêm) | 19 | 19/427 = 4.4% | 427 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 15.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 189 | 0 | 1 | 5 |
| bicycle | 6 | 0 | 2 | 1 |
| motorbike | 125 | 3 | 1 | 7 |
| car | 52 | 0 | 1 | 0 |
| bus | 3 | 0 | 0 | 1 |
| truck | 6 | 0 | 0 | 3 |
| three_wheeler | 19 | 5 | 15 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 33 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 32 | 27 | 1 | 54.2% (KTC 95%: 42–66%) | 97.0% (KTC 95%: 85–99%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 2/3
  - wrong_class: 2/3
  - missing: 3/3
  - lọt: img_008.jpg (loose), img_023.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
