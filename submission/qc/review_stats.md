# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 401; đối tượng sau duyệt: 403; ngưỡng accepted: IoU >= 0.9
- Đã **loại 20 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 398 | 398/401 = 99.3% | 401 box máy |
| edited (sửa) | 3 | 3/401 = 0.7% | 401 box máy |
| deleted (xoá) | 0 | 0/401 = 0.0% | 401 box máy |
| added (vẽ thêm) | 2 | 2/403 = 0.5% | 403 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 2.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 229 | 0 | 0 | 1 |
| bicycle | 8 | 1 | 0 | 0 |
| motorbike | 90 | 0 | 0 | 1 |
| car | 38 | 1 | 0 | 0 |
| bus | 3 | 0 | 0 | 0 |
| truck | 7 | 0 | 0 | 0 |
| three_wheeler | 23 | 1 | 0 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 61 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 56 | 7 | 5 | 88.9% (KTC 95%: 79–95%) | 91.8% (KTC 95%: 82–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - loose: 3/3
  - wrong_class: 2/3
  - extra: 3/3
  - lọt: img_018.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
