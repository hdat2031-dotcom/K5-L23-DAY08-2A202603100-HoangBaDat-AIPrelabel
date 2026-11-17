# Báo cáo Day 08 — 2A202603100 - Hoàng Bá Đạt

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person=0.4, motorbike=0.5, car=0.5, bicycle=0.3, bus=0.3, truck=0.3 | Dựa trên thực nghiệm `r01`, `r02` và kết quả `submission/qc/threshold_sweep.md`. Ngưỡng 0.4–0.5 lọc bỏ box rác hậu cảnh mà vẫn giữ recall >= 90% cho người và xe máy; các lớp ít vật (< 5 vật gold như bicycle, bus, truck) giữ ngưỡng mặc định 0.3. |
| three_wheeler | YOLOE-26s, prompt: "tuk-tuk", "cyclo", "cargo tricycle" | 0.3 (mặc định) | Dựa trên `r03` trong `model_log.csv`. YOLO26 không nhận diện được xe ba bánh Đông Nam Á; YOLOE phát hiện chính xác tuk-tuk và xích lô tại `img_002` và `img_032`. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.4,motorbike=0.5,car=0.5" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn**: `sam2.1_b.pt` (SAM 2.1 ViT-B). Lý do: Cân bằng tối ưu giữa độ mịn của đường biên polygon mask và thời gian suy luận (16.3s/ảnh trên CPU); không gặp rào cản truy cập tài khoản gated như SAM 3, đồng thời cho chất lượng bóc tách chi tiết vượt trội so với MobileSAM trên các vật thể bị che khuất một phần.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car` / `truck`)? Sửa thế nào? Model closed-set COCO hoàn toàn không có nhãn `three_wheeler` trong không gian nhãn huấn luyện, nên dù hạ ngưỡng tin cậy xuống 0.01 hay tăng lên 0.9 thì model vẫn chỉ dự đoán thành `car` hoặc `truck`. Cách sửa: Kết hợp model open-vocabulary (YOLOE-26s) với prompt chữ cụ thể (`tuk-tuk`, `cyclo`, `cargo tricycle`), sau đó đặt file YOLOE ở vị trí ưu tiên cao nhất trong lệnh `merge_prelabels.py` để ghi đè các box `car`/`truck` của YOLO26 khi IoU >= 0.6.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Tại `img_001.jpg`, chiếc xích lô du lịch bị YOLO26 nhận diện là `truck` với điểm tin cậy rất cao `conf=0.88`.
  - Điểm thấp mà đúng: Tại `img_012.jpg`, chiếc tuk-tuk thật chạy trên đường Ratchadamri được YOLOE phát hiện với điểm `conf=0.30`.
- Giới hạn: Việc quét và chọn ngưỡng diễn ra trên chính 4 ảnh gold (in-sample optimization), nên các chỉ số precision / recall của máy trên gold mang tính lạc quan; tập gold chỉ gồm 4 ảnh nên ngưỡng chỉ là ước lượng thực nghiệm thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 339 | 339/401 = 84.5% | 401 box máy |
| edited (sửa) | 34 | 34/401 = 8.5% | 401 box máy |
| deleted (xoá) | 28 | 28/401 = 7.0% | 401 box máy |
| added (vẽ thêm) | 134 | 134/507 = 26.4% | 507 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp `three_wheeler` và `bicycle` là hai lớp máy kém nhất. Với `three_wheeler`, máy thường bị lỏng box quanh phần bánh xe và mui bạt (như tại `img_002.jpg` và `img_013.jpg`). Với `bicycle`, máy dễ nhầm lẫn giữa xe đạp và xe máy ở cự ly xa (như tại `img_018.jpg`).
- So với gold: Precision của **máy** đạt **87.3%** (55/63 TP+FP, KTC 95%: 77–93%) / Recall của **máy** đạt **71.4%** (55/77 đối tượng gold, KTC 95%: 61–80%) (mẫu số: 77 đối tượng gold).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **12/12 = 100.0%** (KTC 95%: 76–100%); loại nào lọt: Không có bẫy nào bị lọt (bắt trọn 3/3 missing, 3/3 loose, 3/3 wrong_class, 3/3 extra nhờ quy trình quét mù trước và đối chiếu từng lớp đối tượng). Bẫy `wrong_class` tại `img_018.jpg` được phát hiện và chỉnh sửa chuẩn xác sau khi rà soát kỹ lớp người đi xe đạp/xe máy.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Nếu `accepted` cao (ví dụ > 95%) mà tỉ lệ bắt bẫy thấp (< 60%), điều đó phản ánh người duyệt mắc thiên kiến tự động hóa nặng nề — chỉ bấm chấp nhận hàng loạt mà không thực sự soi ảnh độc lập. Ngược lại, nếu bắt bẫy cao (>= 90%) song hành cùng tỉ lệ `accepted` hợp lý, chứng tỏ người duyệt thực hiện nghiêm ngặt quy trình kiểm tra 4 bước và có năng lực chốt chặn chất lượng độc lập.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **5.5 phút/ảnh** (22 phút / 4 ảnh gold) vs duyệt có pre-label **3.0 phút/ảnh** (72 phút / 24 ảnh batch). Tiết kiệm được ~45% thời gian trên mỗi ảnh. Thiên lệch: Người duyệt đã tự tay vẽ 4 ảnh gold trước nên đã quen mắt với các góc máy và bối cảnh đường phố châu Á, giúp quá trình duyệt nhanh hơn.
- Kết luận: **RẤT ĐÁNG DÙNG** cho các bài toán gán nhãn dữ liệu đường phố, đặc biệt hiệu quả vượt trội cho các lớp phổ thông như `person`, `motorbike`, `car` (chiếm hơn 85% tổng số đối tượng). Với các lớp xe đặc thù, quy trình phối hợp YOLO26 + YOLOE là bắt buộc.
- **Ba quy tắc giảm thiên kiến của bạn**:
  1. *Quy tắc Blind Scan (Quét mù trước)*: Luôn tắt hiển thị layer nhãn máy trước khi mở ảnh mới, dùng mắt quét toàn cảnh từ trái sang phải trong 15-20 giây để phát hiện các đối tượng tự nhiên trước khi bật nhãn máy lên đối chiếu.
  2. *Quy tắc Cross-Check Person-Vehicle (Kiểm tra chéo cặp đôi)*: Với mọi phương tiện đang lưu thông trên đường (`motorbike`, `bicycle`, xích lô), bắt buộc kiểm tra xem đã có box `person` tương ứng cho người điều khiển chưa, ngăn chặn việc máy chỉ gán xe mà quên người.
  3. *Quy tắc Pomodoro Rest (Nghỉ mắt định kỳ)*: Cứ sau mỗi 8–10 ảnh duyệt, dừng lại nghỉ mắt 30 giây nhìn ra xa để giải tỏa mỏi cơ mắt và thiết lập lại độ nhạy nhận thức, chống trơ giác quan.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Không gồm người lái (đã tách bằng click âm); bao phủ toàn bộ khung xe tuk-tuk và mui | Khi chỉ click 1 điểm dương ở giữa thân xe, SAM có xu hướng lan mask vào lưng người lái; cần thêm 1 click âm trên vai người lái để bóc tách | `tuktuk_img013.jpg` |
| `img_003.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos, 1 neg) | Không gồm người đứng bán hàng; bao gồm thùng xe và hàng hoá gắn liền | Hàng hoá trên thùng xe tạo đường biên zíc zắc phức tạp, SAM phân tách thành 3 polygon nhỏ ở các nan sắt | `tricycle_img003.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 4 click (2 pos, 2 neg) | Không gồm tài xế ngồi phía trước và người đứng bên cạnh; bao trọn thân xe tuk-tuk chính | Vùng bóng đổ dưới gầm xe hơi bị lẹm vào mask nếu không đặt điểm dương cao hơn sàn xe | `tuktuk_img004.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc 4.2.1 (lệch biên <= 3px) và 4.2.4 (vật chạm nhau không chồng lấn, tách người lái thành đối tượng riêng) là kim chỉ nam rất rõ ràng giúp đặt điểm click âm chuẩn xác. Tuy nhiên, guideline v1.0 còn thiếu hướng dẫn cụ thể về việc xử lý khoảng trống hở giữa các thanh nan hoa bánh xe xích lô (quy tắc lỗ thủng).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Interactor SAM trong CVAT mang lại trải nghiệm tương tác trực quan cao (chấm điểm dương/âm ngay trên màn hình canvas); tuy nhiên chạy offline bằng script `sam2_points.py` cho phép lưu trữ vết click chính xác, chạy hàng loạt theo lô và kiểm soát phiên bản hoàn toàn lặp lại được.

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** (từ `CAL-01` đến `CAL-04` trong `submission/qc/calibration_decisions.csv`); quan trọng nhất: `CAL-01` (chuẩn hóa quy tắc gán nhãn xe ba bánh địa phương và nẹp sát biên hình học) và `CAL-02` (phân định ranh giới giữa xe đạp điện và xe máy điện).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Đã bổ sung chi tiết vào các mục 1 (câu hỏi mở), mục 2 (ca khó mới kèm ảnh ví dụ `img_013.jpg`, `img_004.jpg`, `img_018.jpg`, `img_005.jpg`), và mục 3 (lịch sử thay đổi phiên bản v1.1).
- Nguyên nhân gốc: Bắt nguồn từ sự kết hợp của cả 3 yếu tố: giới hạn phân loại của model closed-set đối với xe bản địa (`loi_may`), sự thiếu sót trong định nghĩa phân loại phương tiện điện thế hệ mới (`cau_hoi_mo`), và hiện tượng mỏi mắt trước mật độ giao thông cực lớn (`bay_lot`).

## 7. QC và quyết định (Part 6)

- Mẫu: **6 ảnh** (1 ảnh ca khó `can_review` là `img_013.jpg` + 5 ảnh ngẫu nhiên: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_017.jpg`, `img_033.jpg`), tổng cộng **90 đối tượng**. Người kiểm: Học viên 2A202603100 (**hạn chế**: người kiểm định đồng thời là người review nên có khả năng trùng lặp góc nhìn).
- Lỗi theo **đối tượng**: **0/90 = 0.0%** · theo **ảnh**: **0/6 = 0.0%** (ngưỡng phê duyệt Release <= 2%).
- Top lỗi: Không phát hiện lỗi sai sót nào trong 6 ảnh mẫu kiểm tra.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 ảnh lỗi trên 6 ảnh mẫu, cận trên 95% khoảng tin cậy Wilson của tỉ lệ ảnh lỗi thật là **39%**. Mẫu kích thước 6 ảnh không đủ sức mạnh thống kê để chứng minh toán học rằng toàn bộ tập dữ liệu đạt tỉ lệ lỗi <= 2%; đây là giới hạn tất yếu của kiểm tra mẫu nhỏ trong điều kiện thời gian thực hành giới hạn.
- Script kiểm được / không kiểm được: Script kiểm tra tự động rất tốt về mặt cú pháp dữ liệu (box ngoài ảnh, nhãn lạ ngoài taxonomy, box rác < 20px, cấu trúc file); tuy nhiên script hoàn toàn không thể đánh giá được ngữ nghĩa thực tế (ví dụ: box có thực sự nẹp sát mép xe hay không, model và người có cùng bỏ sót một người đi bộ trong bóng râm hay không).

**Quyết định: RELEASE**, ngày 2026-11-17. Lý do:
1. *Không HOLD*: Các câu hỏi mở của guideline v1.0 đã được hiệu chuẩn và chốt cụ thể thành guideline v1.1; không có ảnh nào bị hỏng hay lỗi dữ liệu gốc.
2. *Không REWORK*: Tỉ lệ ảnh lỗi trong mẫu QC đạt 0.0% (dưới ngưỡng tối đa 2%); tỉ lệ bắt bẫy đạt 100.0% (12/12 bẫy, vượt ngưỡng tối thiểu 80%); không ghi nhận lỗi sai lệch mang tính hệ thống.
3. *Đủ điều kiện RELEASE*: Bộ nhãn lô batch 24 ảnh cùng 4 ảnh gold đáp ứng đầy đủ tiêu chuẩn hợp đồng nhãn và sẵn sàng đưa vào pipeline huấn luyện mô hình.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Antigravity làm trợ lý phân tích dữ liệu, tự động hóa chạy các pipeline suy luận mô hình (YOLO26, YOLOE, SAM 2.1), quét ngưỡng tối ưu, và kiểm tra tính toàn vẹn của dữ liệu theo đúng checklist đề bài.
- Gold có bị nhiễm không: Hoàn toàn không bị nhiễm bẩn; 4 ảnh gold được phân tích và chuẩn hoá hoàn toàn theo con mắt chuyên gia độc lập trước khi chạy sweep.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GPU NVIDIA GeForce 4GB VRAM (kết hợp CPU fallback), Docker CVAT v2.83.1. Đã giải quyết triệt để lỗi xung đột đường dẫn môi trường Python (`day06.pth`).
- Dùng cache `data/cache/` thay vì tự chạy? Chạy trực tiếp model trên các tập đại diện ở Part 2 (`t_y26`, `t_y26_c05`, `t_yoloe`, `t_sam`) để quan sát trực quan hiện tượng; sử dụng cache toàn bộ ảnh (`y26_all`, `yoloe_all`) ở Part 3 và Part 4 để đảm bảo tính nhất quán tuyệt đối và tối ưu hóa thời gian thực thi.
- Lỗi / bug script đã gặp: Gặp lỗi ký tự mã hóa UTF-8 trong một số chuỗi PowerShell có chứa dấu ngoặc kép và đường dẫn; đã xử lý an toàn bằng cách truyền tham số bọc chuẩn.
- Một điều bạn làm khác lần sau: Sẽ tăng kích thước tập gold lên 10-15 ảnh để việc quét ngưỡng ở Part 3 có độ tin cậy thống kê cao hơn, giảm biên độ lạc quan in-sample.
