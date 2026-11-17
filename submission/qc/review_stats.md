# Số đo review (det)

- Ảnh tính: 24 (batch); box máy: 401; đối tượng sau duyệt: 507; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 339 | 339/401 = 84.5% | 401 box máy |
| edited (sửa) | 34 | 34/401 = 8.5% | 401 box máy |
| deleted (xoá) | 28 | 28/401 = 7.0% | 401 box máy |
| added (vẽ thêm) | 134 | 134/507 = 26.4% | 507 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 127.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 215 | 11 | 3 | 29 |
| bicycle | 5 | 1 | 3 | 8 |
| motorbike | 71 | 13 | 6 | 58 |
| car | 37 | 1 | 1 | 24 |
| bus | 2 | 0 | 1 | 2 |
| truck | 5 | 0 | 2 | 6 |
| three_wheeler | 4 | 8 | 12 | 7 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 77 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 55 | 8 | 22 | 87.3% (KTC 95%: 77–93%) | 71.4% (KTC 95%: 61–80%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 12/12 = 100.0% (KTC 95%: 76–100%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - loose: 3/3
  - wrong_class: 3/3
  - extra: 3/3

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
