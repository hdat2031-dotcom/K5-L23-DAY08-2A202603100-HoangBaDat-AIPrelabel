# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603100 - Hoàng Bá Đạt

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của reviewer. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Xe có bàn đạp trợ lực cơ học xếp vào `bicycle`; xe thuần điện không có bàn đạp (scooter điện, mô tô điện) xếp vào `motorbike`. | `img_004.jpg` | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc cự ly gần: từng xe hay `ignore_region`? | Xe nhìn thấy >= 20% diện tích thân xe gán box riêng; khối xe đỗ chen chúc che khuất > 80% hoặc chi tiết < 20px vẽ một `ignore_region` bao quanh cụm. | `img_005.jpg` | CAL-04 |
| 3 | Hàng hoá nhô ra khỏi xe: box xe có gồm hàng không? | Nếu hàng hoá cố định trong giỏ/thùng xe hoặc nhô ra <= 20% chiều dài xe thì box xe bao cả hàng. Nếu hàng cồng kềnh nhô ra > 20% thì box xe chỉ ôm thân xe, không mở rộng theo hàng nhô. | `img_003.jpg` | CAL-01 |
| 4 | Người ngồi trong xe ba bánh có mui che: còn thấy >= 20% tính thế nào? | Nếu nhìn thấy rõ đầu và vai hoặc thân trên (>= 20% cơ thể) qua cửa hở thì bắt buộc gán một box `person` riêng; nếu chỉ thấy lờ mờ qua kính chắn gió/vải bạt đục thì bỏ qua. | `img_013.jpg` | CAL-01 |
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe không? | Kính chắn gió trong suốt thuộc về diện tích thân xe (tính vào mask/box của xe); người bên trong nhìn xuyên qua kính vẫn có box `person` độc lập. | `img_020.jpg` | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Tuk-tuk đường phố có người lái và khung mui hở | YOLO26 gọi nhầm thành `car`/`truck`; YOLOE tìm đúng `three_wheeler` nhưng box lỏng phần bánh | Luôn gán `three_wheeler` cho xe 3 bánh địa phương; box nẹp sát điểm tiếp đất của bánh và mép mui bạt; người lái gán box `person` riêng | Bổ sung mục 2 và 4.1.1 (Quy tắc xe ba bánh và nẹp sát biên hình học) |
| `img_004.jpg` | Dãy xe tuk-tuk đỗ sát lề đường bên cạnh xe đạp điện | Model COCO bỏ sót xe điện và tráo lớp tuk-tuk | Phân loại phương tiện điện theo cơ chế vận hành (có/không bàn đạp); tuk-tuk đỗ chồng hàng vẫn tách từng box riêng | Bổ sung mục 2 định nghĩa phương tiện điện |
| `img_018.jpg` | Luồng giao thông xe máy đông đặc, bẫy tráo nhãn `person` thành `bicycle` | Model gán nhãn ma/tráo nhãn người ngồi trên xe máy thành `bicycle` | Quy tắc kiểm tra chéo: Mọi xe máy đang chạy bắt buộc có ít nhất một box `person` (người điều khiển) nằm chồng lên | Bổ sung mục 4.1.4 (Quy trình kiểm tra chéo người lái) |
| `img_005.jpg` | Bãi đỗ xe máy dày đặc che khuất liên hoàn | Model sinh hàng loạt box vụn và sót xe ở xa | Áp dụng ngưỡng diện tích 20px và độ che khuất: Xe rõ gán riêng, cụm xe mờ phía sau vẽ `ignore_region` | Bổ sung mục 4.1.6 (Tiêu chuẩn áp dụng ignore_region) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | Phiên bản khởi đầu |
| v1.1 | Chốt phân định xe đạp điện / xe máy điện (mục 2); tiêu chuẩn hóa box xe ba bánh và người lái (mục 4.1.1 & 4.1.4); quy định cụ thể ngưỡng dùng `ignore_region` cho bãi xe dày đặc (mục 4.1.6) | CAL-01, CAL-02, CAL-03, CAL-04 (rút ra từ `img_013.jpg`, `img_004.jpg`, `img_018.jpg`, `img_005.jpg`) |
