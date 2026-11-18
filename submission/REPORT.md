# Báo cáo Day 08 — 2A202603215 - Nguyễn Gia Huy

Bài làm gồm 4 ảnh gold gán tay, 24 ảnh batch được review trên CVAT và 3 ảnh mask SAM cho các vật khó. Các số liệu dưới đây lấy từ nhật ký chạy, nhật ký thời gian và các file trong `submission/qc/`. Kết quả cuối cùng là **Hold** vì còn một ca chưa chốt phạm vi nhãn và mẫu QC vẫn có lỗi cần sửa.

## 1. Model và recipe (Part 2–3)

Em thử YOLO26 ở hai ngưỡng 0,25 và 0,5, dùng YOLOE tìm xe ba bánh, sau đó dùng box YOLO26 làm prompt cho SAM 2.1. Bốn lượt khảo sát tương ứng `r01`–`r04` trong `model_log.csv`.

| Lượt | Model và cấu hình | Tập ảnh | Kết quả | Thời gian chạy trung bình |
|---|---|---|---|---|
| r01 | YOLO26m, conf 0,25 | 8 ảnh pilot | 215 box | 1,35 giây/ảnh |
| r02 | YOLO26m, conf 0,5 | 8 ảnh pilot | 139 box | 1,08 giây/ảnh |
| r03 | YOLOE-26s, conf 0,25; chỉ tìm three_wheeler | 8 ảnh pilot | 8 box | 0,46 giây/ảnh |
| r04 | YOLO26m, conf 0,3 → SAM 2.1 (`sam2.1_b.pt`) | img_004, img_013 | 16 box, cả 16 có mask | 8,03 giây/ảnh |

Khi tăng ngưỡng YOLO26 từ 0,25 lên 0,5, tổng box giảm 76/215 = 35,3%. Một số box thừa được loại, như box bicycle conf 0,25 ở `img_015`, nhưng box motorbike conf 0,47 của xe máy tiền cảnh ở `img_030` cũng bị mất. Vì vậy em không chọn một ngưỡng chung chỉ dựa trên việc ảnh trông ít box hơn.

Sau khảo sát, em chạy hai model trên toàn bộ 36 ảnh ở conf 0,05 để lấy ứng viên cho bước quét ngưỡng: YOLO26 tạo 1.669 box (0,82 giây/ảnh), YOLOE tạo 125 box three_wheeler (0,48 giây/ảnh), tương ứng `r05` và `r06`. Các lượt này dùng `--no-viz`; nhận xét thị giác của tour model dựa trên các ảnh đã lưu ở `r01`–`r04`.

Ngưỡng cuối lấy từ `qc/threshold_sweep.md`:

| Lớp | Model + cỡ | Ngưỡng | Lý do |
|---|---|---:|---|
| person | YOLO26m | 0,35 | Trên 18 đối tượng gold: recall khoảng 94%, precision khoảng 55%; giữ độ phủ rồi xóa box thừa khi review. |
| bicycle | YOLO26m | 0,20 | Trên 5 đối tượng gold: recall 100%, precision khoảng 71%; mẫu vừa đủ ngưỡng tối thiểu nên kết luận còn yếu. |
| motorbike | YOLO26m | 0,50 | Trên 11 đối tượng gold: recall khoảng 91%, precision 100% tại ngưỡng gợi ý. |
| car | YOLO26m | 0,30 mặc định | Có 8 đối tượng gold nhưng không ngưỡng nào trong miền quét đến 0,5 đạt recall 90%; mức cao nhất khoảng 88%. |
| bus | YOLO26m | 0,30 mặc định | Gold không có bus; không đủ dữ liệu để chọn ngưỡng riêng. |
| truck | YOLO26m | 0,30 mặc định | Gold không có truck; không đủ dữ liệu để chọn ngưỡng riêng. |
| three_wheeler | YOLOE-26s | 0,15 | Trên 7 đối tượng gold: recall 100%, precision khoảng 78%; YOLOE bổ sung lớp xe ba bánh mà YOLO26 không có. |

Prompt YOLOE là `tuk-tuk`, `cyclo`, `cargo tricycle`, cùng ánh xạ vào `three_wheeler`. Ở `img_002`, YOLOE tìm được tuk-tuk xanh giữa ảnh, nhưng cũng gán nhầm xe bán tải trắng thành three_wheeler. Kết quả vẫn phải được kiểm lại, không dùng trực tiếp làm nhãn cuối.

Lệnh gộp đã dùng:

```powershell
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,bicycle=0.2,motorbike=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

Lệnh tạo 536 box trước khi chèn bẫy. File nhập vào CVAT chỉ chứa 24 ảnh batch; file đầy đủ còn có gold để đo máy.

Em chọn **SAM 2.1, trọng số `sam2.1_b.pt`** vì đã chạy được trên CPU của máy và tạo được mask từ box hoặc click. Ở `img_004`, mask tách được phần lớn người và xe khỏi nền; ở `img_013`, mask xe vẫn dính người lái. Đây là phiên bản chính cho cả tour model và ba lượt click ở Part 5; chưa có phép so sánh trực tiếp với SAM 1 hoặc SAM 3.

Đổi ngưỡng không giải quyết được việc tuk-tuk bị YOLO26 gọi thành car/truck. Ví dụ ở `img_011`, tuk-tuk vẫn mang nhãn truck với conf 0,77 ở cả hai ngưỡng; cần sửa lớp theo guideline hoặc dùng YOLOE bổ sung ứng viên. Cũng tại ảnh này, YOLOE tìm được xe ba bánh với conf khoảng 0,28, cho thấy điểm thấp chưa chắc sai và điểm cao chưa chắc đúng.

Giới hạn quan trọng là em chọn ngưỡng và đánh giá máy trên cùng 4 ảnh gold. Các số precision/recall vì vậy có thể lạc quan, đặc biệt với lớp ít mẫu. Tình trạng tham khảo minh họa trước khi chốt gold được ghi ở mục 8.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---|---|
| accepted | 347 | 347/451 = 76,9% | 451 box máy |
| edited | 31 | 31/451 = 6,9% | 451 box máy |
| deleted | 70 | 70/451 = 15,5% | 451 box máy |
| added | 41 | 41/419 = 9,8% | 419 đối tượng sau duyệt |

Nguồn là `qc/review_stats.json` và `.md` sau vòng sửa lại. Script loại 27 đối tượng ở quanh các bẫy khỏi bảng để lỗi được chèn không bị tính thành lỗi tự nhiên của máy. Các mẫu số 451 và 419 là sau bước lọc này, khác số nhãn trong file export.

File `batch_reviewed.json` có 438 annotation, gồm 431 đối tượng và 7 vùng ignore. Ba box máy nằm trong vùng ignore không bị tính là deleted, nên accepted + edited + deleted là 448/451, không cộng thành đúng 100%. Đây là cách xử lý vùng ignore trong script, không phải ba box bị thiếu khỏi file nhãn.

| Lớp | accepted | edited | deleted | added |
|---|---:|---:|---:|---:|
| person | 208 | 15 | 25 | 8 |
| bicycle | 5 | 1 | 4 | 0 |
| motorbike | 82 | 6 | 1 | 18 |
| car | 41 | 1 | 11 | 6 |
| bus | 2 | 0 | 1 | 2 |
| truck | 2 | 0 | 4 | 4 |
| three_wheeler | 7 | 8 | 24 | 3 |

Three_wheeler cần nhiều công sửa nhất: trong 39 box được tính trạng thái của lớp này, có 24 box bị xóa và chỉ 7 box được giữ nguyên. Ở `img_002`, YOLOE gán xe bán tải thành three_wheeler; ở `img_032`, kết quả vẫn còn bỏ sót xe bị che. Với motorbike, 18 đối tượng được thêm cho thấy cần chú ý bước tìm vật thiếu, không chỉ chỉnh các box đã có.

Trong deleted và added có 7 trường hợp đổi lớp. Vì vậy không coi cả 41 added là 41 vật bị bỏ sót; số sót thật theo cách ghép của script xấp xỉ 41 − 7 = 34.

So với gold, máy có TP = 46, FP = 23, FN = 3: **precision 46/69 = 66,7%**, **recall 46/49 = 93,9%**. File gold có 51 annotation nhưng 2 là ignore_region, nên mẫu số recall là 49 đối tượng. Khoảng tin cậy 95% được script in là 55–77% cho precision và 83–98% cho recall. Gold không nằm trong task batch-review nên không có số residual/new error của nhãn gold sau duyệt.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Vòng đầu bắt được **9/12 = 75,0%**, chưa đạt ngưỡng 80%:

| Loại bẫy | Vòng đầu | Sau sửa lại |
|---|---|---|
| extra | 2/3 | 3/3 |
| loose | 3/3 | 3/3 |
| wrong_class | 2/3 | 2/3 |
| missing | 2/3 | 3/3 |
| Tổng | 9/12 | 11/12 |

Ba bẫy lọt ban đầu là `img_004` (extra), `img_025` (missing) và `img_028` (wrong_class). Sau 9 phút sửa lại và export, kết quả tăng thành **11/12 = 91,7%**; bẫy sai lớp ở `img_028` vẫn còn lọt theo script. Khoảng tin cậy 95% của kết quả sau sửa được in là khoảng 65–99%.

Kết quả vòng đầu cho thấy bước kiểm box thừa, vật thiếu và sai lớp chưa đủ hiệu quả. Không có log chi tiết từng thao tác để khẳng định em đã bỏ bước nào ở từng ảnh; các bẫy lọt là bằng chứng để siết lại đúng ba bước này. Box trông hợp lý hoặc có conf cao vẫn có thể sai.

Accepted vòng đầu là 350/451 = 77,6% nhưng bắt bẫy chỉ 75%. Do đó accepted cao không chứng minh review tốt: lỗi máy có thể vẫn được giữ nguyên, còn vật mà cả máy và người cùng bỏ sót thì không hiện trong số accepted. Kết quả 11/12 là sau khi đã biết phản hồi vòng đầu, không thay thế số 9/12 ban đầu và không được xem là một lần đo mù độc lập.

## 4. Máy có đáng dùng không?

Thời gian từ `time_log.csv`:

| Công việc | Số ảnh | Thời gian | Trung bình |
|---|---:|---:|---:|
| Gán tay gold | 4 | 30 phút | 7,50 phút/ảnh |
| Review vòng đầu | 24 | 98 phút | 4,08 phút/ảnh |
| Sửa lại sau thống kê bẫy | Cùng lô 24 ảnh | 9 phút bổ sung | Tính vào tổng review |
| Review gồm cả sửa lại | 24 | 107 phút | 4,46 phút/ảnh |

So với 7,50 phút/ảnh gán tay, 4,46 phút/ảnh review thấp hơn khoảng 40,6%. Đây chỉ là so sánh trên hai tập ảnh khác nhau: số đối tượng, độ khó và thao tác không giống nhau; sau khi gán gold em cũng đã quen kiểu cảnh. Tổng 107 phút không gồm thời gian cài đặt, tải model, inference hoặc làm mẫu QC, nên không coi 40,6% là mức tiết kiệm của toàn bộ quy trình.

Em thấy pre-label có ích để tạo bộ box ban đầu, nhất là person và motorbike. Tuy nhiên, 18 added ở motorbike và nhiều box three_wheeler bị xóa cho thấy vẫn phải kiểm từng ảnh. Với xe ba bánh, YOLOE giúp mở rộng lớp nhưng cần kiểm sai lớp, box lỏng và xe bị che. Kết quả QC còn lỗi nên mức giảm thời gian chưa đủ để kết luận nhãn đã sẵn sàng sử dụng.

Ba quy tắc áp dụng cho lần review tiếp theo:

1. Trước khi nhìn box, ẩn nhãn và quét đủ 100% ảnh khoảng 15 giây; với ảnh đông thì lần lượt quét bốn góc. Sau đó tìm vật thiếu trước khi chỉnh box có sẵn.
2. Kiểm riêng lớp của 100% box xe ba bánh và các box car/truck ở gần chúng. Không bỏ qua kiểm tra vì conf cao; đối chiếu ảnh và định nghĩa lớp trước khi giữ nhãn.
3. Sau mỗi khoảng 10 ảnh thì nghỉ ngắn, rồi rà lại các ca can_review. Nếu lần kiểm bẫy đầu dưới 80% thì review lại lô, ghi riêng thời gian bổ sung và kết quả từng vòng.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 (`sam2.1_b.pt`) | 3: 2 dương, 1 âm | Còn dính một phần người lái và người phía sau. Chưa đánh giá riêng trường hợp chở hàng. | Mask lấy được phần lớn mái, thân và bánh xe nhưng vẫn tô vào người. Một click âm ở người lái chưa đủ để tách sạch. | [tuktuk_img013.jpg](masks/tuktuk_img013.jpg) |
| img_004 | SAM 2.1 (`sam2.1_b.pt`) | 3: 2 dương, 1 âm | Không dính hai người đứng cạnh xe. Chưa đánh giá riêng trường hợp chở hàng. | Mask tách được tuk-tuk xanh dương phía trước khỏi các xe phía sau, nhưng bỏ sót biển TAXI gắn trên mái. | [tuktuk_img004.jpg](masks/tuktuk_img004.jpg) |
| img_032 | SAM 2.1 (`sam2.1_b.pt`) | 3: 2 dương, 1 âm | Không dính người đứng bên phải. Không có ca hàng hoá được kiểm riêng trong lượt này. | Mask chọn được ghế, khung phía trước và hai bánh lớn; một số khoảng trống trong khung cũng bị tô kín. Chưa đo diện tích từng lỗ nên chưa kết luận tất cả các khoảng này đều vi phạm ngưỡng khoảng 5%. | [cyclo_img032.jpg](masks/cyclo_img032.jpg) |

- em dùng cùng SAM 2.1 cho cả ba vật. Mỗi lượt có hai điểm dương trên xe và một điểm âm trên người đứng cạnh hoặc người lái. Các lệnh đã chạy:

  ```powershell
  python scripts/sam2_points.py --image img_013.jpg --pos "400,480" --pos "600,110" --neg "720,300" --sam sam2.1_b.pt --out submission/masks --name tuktuk_img013
  python scripts/sam2_points.py --image img_004.jpg --pos "720,400" --pos "650,260" --neg "465,350" --sam sam2.1_b.pt --out submission/masks --name tuktuk_img004
  python scripts/sam2_points.py --image img_032.jpg --pos "400,350" --pos "325,480" --neg "565,350" --sam sam2.1_b.pt --out submission/masks --name cyclo_img032
  ```

- Diện tích mask do script in ra lần lượt là 317.255/698.368 pixel ở `img_013` (45,4% ảnh), 92.185/698.368 pixel ở `img_004` (13,2%) và 50.773/589.824 pixel ở `img_032` (8,6%). Cả ba lượt đều trả về một polygon. Đây là diện tích và số polygon, không phải điểm chính xác của mask.
- Mục 4.2 của guideline giúp kiểm phần nhìn thấy, bóng đổ, khoảng trống và vùng hai vật chạm nhau. Với `img_013`, phần người còn bị tô vào mask xe là lỗi phân đoạn; quy tắc tách người và xe đã có, không cần đổi định nghĩa lớp. Với `img_032`, cần kiểm từng khoảng trống theo ngưỡng xấp xỉ 5% đã quy định, không tự coi mọi lỗ đều phải khoét.
- Em bổ sung cách phân biệt bề mặt kính với khoang xe hở trong `CAL-02`, vì giữ kính trong mask không có nghĩa là được tô cả người nhìn qua cạnh hở. `CAL-04` và `CAL-05` ghi lại các bước rà mask người/xe, bộ phận gắn cố định và lỗ trong khung.
- Ba ảnh trên là kết quả của các lượt click đã chạy, còn giữ lỗi để phân tích. Chưa có lượt sửa mask bổ sung hoặc kết quả đối chứng bằng Interactor SAM trong CVAT được đưa vào báo cáo này.

## 6. Calibration (Part 6)

- Bảng `qc/calibration_decisions.csv` có 6 ca: 5 quyết định đã chốt từ Part 5–6 và 1 ca còn mở phát sinh khi QC (`CAL-06`). Có 3 ca `loi_may` và 3 ca `cau_hoi_mo`. Các ca đều dẫn ảnh đã quan sát, không lấy ví dụ giả định thay cho kết quả chạy.
- `CAL-01`: YOLO26 gán tuk-tuk thành `car` ở `img_004` và `img_013`. Em giữ định nghĩa `three_wheeler` của v1.0, bổ sung các ảnh này vào phần nhắc lỗi. Đây là lỗi máy, không phải lý do để đổi taxonomy.
- `CAL-02`: chốt câu hỏi mở số 5 về kính xe. Giữ bề mặt kính gắn trên xe trong mask xe, kể cả khi nhìn xuyên qua kính. Khoang hở không có kính phải được phân biệt với bề mặt kính; người hoặc nền nhìn qua khoang hở không tự động thuộc mask xe.
- `CAL-03`: chốt câu hỏi mở số 7 về điểm tin cậy. Không có mức `conf` nào cho phép bỏ qua kiểm tra ảnh. Ở `img_011`, YOLO26 gán sai tuk-tuk thành `truck` với conf 0,77, trong khi YOLOE tìm được xe này dưới lớp `three_wheeler` với conf khoảng 0,28. Điểm tin cậy chỉ dùng để ưu tiên kiểm tra; ngưỡng giữ pre-label vẫn theo kết quả Part 3.
- `CAL-04`: mask xe còn dính người ở `img_013`, còn ở `img_004` thì bỏ biển TAXI gắn trên mái. Em giữ quy tắc tách người và xe, đồng thời bổ sung bước rà các bộ phận gắn cố định trước khi chấp nhận mask.
- `CAL-05`: dùng `img_032` làm ví dụ rà lỗ trong khung xích lô. Giữ ngưỡng xấp xỉ 5% của v1.0; chỉ yêu cầu loại các lỗ lớn, không tự đặt một ngưỡng mới. Do chưa đo từng lỗ, ca này ghi nhận việc cần kiểm tiếp chứ chưa coi toàn bộ mask là sai.
- `CAL-06`: trong QC, các xe đẩy bán trái cây ở `img_008` chưa được xác nhận rõ cấu trúc ba bánh và phạm vi áp dụng lớp. Ca này còn mở, được tính là 1 guideline gap; chưa tự tính cả ba box là lỗi sai lớp.
- Phần bổ sung nằm trong `guidelines/guideline_v1.1_calibrated.md`, có đối chiếu `case_id` và lịch sử thay đổi. Các câu hỏi 1, 2, 3, 4 và 6 của v1.0 chưa được chốt thêm trong lần calibration này; trạng thái và giới hạn được ghi riêng trong v1.1.
- Nguyên nhân được tách thành hai nhóm: nhầm lớp, dính người hoặc thiếu bộ phận là lỗi máy; cách xử lý kính nhìn xuyên qua và việc có được chấp nhận chỉ theo `conf` là các điểm cần làm rõ trong guideline. Các quyết định này không thay tên lớp, thuộc tính hay ngưỡng trong `taxonomy.json`.

## 7. QC và quyết định (Part 6)

- Mẫu gồm 8/24 ảnh batch, với 3 ca khó có `can_review` (`img_006`, `img_027`, `img_035`) và 5 ảnh ngẫu nhiên (`img_003`, `img_008`, `img_014`, `img_017`, `img_034`), seed 42. Có 145 đối tượng không tính `ignore_region`, theo `qc/qc_sample.csv`. Đây là kiểm tra detection; ba ảnh mask SAM không nằm trong mẫu số 145 này.
- Phiếu `qc/qc_review.csv` ghi reviewer là Nguyễn Gia Huy. Việc kiểm dựa trên ảnh gốc và ảnh box có ID trong `work/qc/review/`. Đây là kiểm tra trên cùng bộ nhãn, chưa có vòng xác nhận độc lập bởi người thứ hai nên không dùng để kết luận độ nhất quán giữa hai người.
- Theo đầu ra `qc/qc_report_auto.md`, số đối tượng/vùng có lỗi là **8/145 = 5,5%** và số ảnh có ít nhất một lỗi là **7/8 = 87,5%**, vượt ngưỡng ảnh lỗi 2%. Trong 8 lỗi có 1 lỗi vùng `ignore_region` (#316 ở `img_027`), trong khi mẫu số 145 không tính các vùng ignore. Vì vậy chỉ số theo đối tượng là quy ước đếm của script; riêng 7 lỗi trên đối tượng thông thường tương ứng **7/145 = 4,8%**. Quyết định theo chất lượng nhãn dùng tỷ lệ theo ảnh, không đánh đồng hai cách đếm.

| Loại lỗi | Số | Ví dụ đã đối chiếu |
|---|---:|---|
| Sai vị trí/kích thước | 3/8 lỗi | `img_003` #3 cắt mép mái bạt bên trái; `img_008` #89 cắt phần vai/áo còn nhìn thấy; `img_017` #137 cắt phần trên biển TAXI và dư nền phía dưới. |
| Thừa | 3/8 lỗi | `img_006` #74 trùng người đã có #77; `img_027` #316 là vùng ignore bao một ô tô nhỏ riêng lẻ; `img_034` #394 trùng người đã có #416. |
| Bỏ sót | 2/8 lỗi | `img_003` thiếu xe đạp của người áo hoa phía sau giữa ảnh; `img_035` thiếu một xe máy đỗ ngay sau chiếc #433. |

- `img_014` chưa phát hiện lỗi chắc chắn trong vòng kiểm này. Không lấy việc hai box của hai xe đỗ chồng nhau làm bằng chứng nhãn trùng. Với các chi tiết quá mờ hoặc bị che, chỉ ghi lỗi khi có đủ bằng chứng; chưa khẳng định đã tìm hết mọi lỗi nhỏ trong ảnh.
- Có 1 ca còn mở: các xe đẩy bán trái cây #85/#87/#88 ở `img_008` chưa được xác nhận rõ có thuộc `three_wheeler` hay không (`CAL-06`). Ca này được tính vào `--guideline-gap 1`; chưa tính ba box là ba lỗi sai lớp. Không ghi nhận ảnh hỏng đến mức không dùng được trong mẫu (`--data-problem 0`).
- Lệnh tạo báo cáo: `python scripts/qc_report.py --guideline-gap 1`. Không sửa tay file báo cáo tự động.
- **Giới hạn mẫu nhỏ:** với 7/8 ảnh có lỗi, cận trên Wilson 95% được script in là khoảng **98%**. Mẫu chỉ có 8 ảnh và gồm cả các ảnh được chọn vì khó, nên không suy rộng rằng 87,5% của toàn bộ batch chắc chắn có lỗi. Mẫu này cho thấy còn lỗi đáng sửa, không chứng minh được chất lượng dưới 2%.
- Kết quả bắt bẫy hiện tại là 11/12 = 91,7%, nhưng đó là sau vòng sửa lại 9 phút dựa trên kết quả vòng đầu. Vòng đầu chỉ đạt 9/12 = 75%. Hai số cần được giữ riêng; 11/12 không thay thế kết quả review ban đầu và không phải kiểm định độc lập.
- Script kiểm được cấu trúc, ID, số lỗi ghi trong phiếu, tỷ lệ lỗi và cận trên thống kê. Script không tự xác nhận lớp đúng trong ảnh, biên box ôm sát, cấu trúc bánh xe bị che hoặc vật mà cả người duyệt và máy cùng bỏ sót. Các lỗi trên được xác định bằng đối chiếu ảnh, không suy ra chỉ từ `accepted` hay điểm tin cậy.

**Quyết định: Hold**, ngày **2026-11-17**. Theo thứ tự Hold → Rework → Release, dừng ở Hold vì còn 1 ca chưa chốt phạm vi lớp trong mẫu QC. Đồng thời 7/8 ảnh có lỗi vượt ngưỡng 2%, nên ngay cả khi giải quyết xong ca mở thì nhãn vẫn cần Rework; chưa đủ điều kiện Release.

Việc cần làm tiếp:

1. Xác minh và chốt cách gán các xe đẩy ở `img_008`; cập nhật `CAL-06` và v1.1 theo kết quả, không đổi lớp chỉ để làm đẹp số đo.
2. Sửa các lỗi trong phiếu trên CVAT, đặc biệt box trùng và vật bị bỏ sót; xuất các ảnh đã sửa thành `submission/labels/qc_fixed.json` theo quy trình QC.
3. Chạy lại `build_release.py`, tạo mẫu QC vòng sau bằng `qc_sample.py --n 5 --seed 42 --force`, rồi kiểm và điền phiếu mới. Chỉ dùng `--force` sau khi đã lưu nhận xét và số liệu của vòng hiện tại; lệnh này ghi đè phiếu kiểm.
4. Cập nhật lại kết quả và quyết định sau vòng sửa. Các lỗi của ba mask SAM ở mục 5 vẫn cần xử lý riêng nếu muốn dùng các mask đó như nhãn hoàn chỉnh.

## 8. Ghi chú cá nhân

- **Công cụ hỗ trợ và cách kiểm tra:** em dùng ChatGPT/Codex để giải thích quy trình lab, đọc thông báo lỗi, hướng dẫn lệnh chạy.
- Em gán 4 ảnh gold và review 24 ảnh batch trên CVAT Docker, export theo COCO 1.0. Các số đo được tính từ những file nhãn đã export; thời gian gán và review được ghi trong `time_log.csv`. em đối chiếu file nhãn bằng `validate_labels.py`, xem ảnh box/mask và so các tỷ lệ với đầu ra trong `submission/qc/`.
- **Gold bị nhiễm ở img_001:** trước khi chốt ảnh này, em đã hỏi ChatGPT/Codex cách gán người và xe trên ảnh, đồng thời xem ảnh minh họa box được tạo từ công cụ. Nhãn vẫn được vẽ tay trên CVAT, nhưng ảnh này không còn độc lập với gợi ý đã xem. Vì vậy không coi cả 4 ảnh gold là một tập đối chứng hoàn toàn sạch; số đo trên gold cần được đọc cùng hạn chế này và việc chọn, chấm ngưỡng trên cùng 4 ảnh. Chưa có tập gold thay thế để định lượng riêng mức ảnh hưởng.
- Môi trường: Windows, Python 3.11.9, PyTorch 2.15.0+cpu, Ultralytics 8.4.215, CVAT Docker 2.74.1. Máy có RTX 4050 Laptop khoảng 6 GB VRAM nhưng các lượt trong nhật ký chạy bằng CPU vì PyTorch hiện tại là bản CPU. Không ghi các thời gian này như kết quả chạy CUDA.
- Ban đầu thiếu OpenCV và Ultralytics, sau đó đã cài thư viện và tải `yolo26m.pt`, `yoloe-26s-seg.pt`, `sam2.1_b.pt`. YOLOE tự cài thêm CLIP ở lượt đầu; cảnh báo `torch.jit.load` không làm lượt chạy thất bại. Tên menu CVAT có thể khác hình trong tài liệu do khác phiên bản.
- Không dùng `data/cache/` thay cho các lượt chạy trong bài. Kết quả được tạo từ các lệnh đã chạy và ghi trong `work/runs.csv`; các lớp ngoài taxonomy như umbrella, bottle, handbag được loại theo ánh xạ của đề.
- Không sửa mã script để thay số đo hoặc đổi mục `rules` trong taxonomy. Bản nộp giữ kết quả Hold, các lỗi QC còn lại và giới hạn của ba mask; chưa có `qc_fixed.json` hoặc vòng QC sau sửa để khẳng định đã giải quyết hết.
- Lần sau em sẽ dành riêng một lượt rà vật thiếu và sai lớp trước khi chỉnh biên, đồng thời ghi lại ca mơ hồ ngay khi gặp. Như vậy phần calibration có căn cứ rõ hơn và hạn chế phải quay lại sửa sau khi xem kết quả bẫy.
