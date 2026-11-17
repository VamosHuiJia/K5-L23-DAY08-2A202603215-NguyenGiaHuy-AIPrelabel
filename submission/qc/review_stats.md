# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 451; đối tượng sau duyệt: 419; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 347 | 347/451 = 76.9% | 451 box máy |
| edited (sửa) | 31 | 31/451 = 6.9% | 451 box máy |
| deleted (xoá) | 70 | 70/451 = 15.5% | 451 box máy |
| added (vẽ thêm) | 41 | 41/419 = 9.8% | 419 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 34.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 208 | 15 | 25 | 8 |
| bicycle | 5 | 1 | 4 | 0 |
| motorbike | 82 | 6 | 1 | 18 |
| car | 41 | 1 | 11 | 6 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 4 | 4 |
| three_wheeler | 7 | 8 | 24 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 49 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 46 | 23 | 3 | 66.7% (KTC 95%: 55–77%) | 93.9% (KTC 95%: 83–98%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 3/3
  - wrong_class: 2/3
  - missing: 3/3
  - lọt: img_028.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
