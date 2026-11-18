# Thông tin học viên

- Họ và tên: Nguyễn Gia Huy
- MSHV: 2A202603215
- Lớp: L2A
- Tài khoản GitHub: VamosHuiJia
- Repo bài nộp: https://github.com/VamosHuiJia/K5-L23-DAY08-2A202603215-NguyenGiaHuy-AIPrelabel
- Commit chốt bộ kết quả (hash): [e492258662d39083c258844a21551ec274fbae9c](https://github.com/VamosHuiJia/K5-L23-DAY08-2A202603215-NguyenGiaHuy-AIPrelabel/commit/e492258662d39083c258844a21551ec274fbae9c). Đây là bản nhãn, số liệu và báo cáo đã chốt trước phần bổ sung thông tin bàn giao.
- Máy: Windows; NVIDIA RTX 4050 Laptop khoảng 6 GB VRAM; các lượt model chạy CPU với PyTorch 2.15.0+cpu.
- CVAT (Docker): 2.74.1

## Kết quả tóm tắt

- Máy: accepted **347/451 = 76,9%**; edited **31/451 = 6,9%**; deleted **70/451 = 15,5%**. Mẫu số là 451 box máy sau khi loại vùng bẫy; 3 box thuộc vùng ignore không tính deleted nên tổng ba trạng thái không bằng 100%.
- Added: **41/419 = 9,8%**, mẫu số là 419 đối tượng sau duyệt được dùng trong thống kê. Có 7 trường hợp đổi lớp; số sót thật xấp xỉ 34. File export đầy đủ có 438 annotation, gồm 431 đối tượng và 7 vùng ignore.
- So với gold: precision **46/69 = 66,7%**; recall **46/49 = 93,9%**. Gold gồm 4 ảnh, 49 đối tượng và 2 vùng ignore; ngưỡng được chọn trên chính gold nên số đo có thể lạc quan.
- Bắt bẫy: vòng đầu **9/12 = 75,0%**; sau vòng sửa lại **11/12 = 91,7%**. Hai vòng được ghi riêng; vòng sau đã có phản hồi từ vòng đầu.
- Thời gian: gold **30 phút/4 ảnh = 7,50 phút/ảnh**; review **107 phút/24 ảnh = 4,46 phút/ảnh**, gồm 98 phút vòng đầu và 9 phút sửa lại. Mức giảm trung bình khoảng **40,6%**, chỉ so thời gian gán/review trên hai tập ảnh khác nhau.
- Mask: 3 ảnh kết quả SAM 2.1 cho img_013, img_004 và img_032; mỗi vật dùng 2 click dương và 1 click âm. Nhận xét lỗi nằm ở REPORT mục 5.
- Calibration: 5 quyết định đã chốt và 1 ca QC còn mở (CAL-06); xem `submission/qc/calibration_decisions.csv` và guideline v1.1.
- Lỗi QC: **7/8 ảnh = 87,5%**; theo cách đếm của script **8/145 = 5,5%**. Trong 8 lỗi có 1 lỗi vùng ignore; riêng lỗi trên đối tượng thông thường là **7/145 = 4,8%**. Mẫu gồm 3 ca khó và 5 ảnh ngẫu nhiên, seed 42; cận trên Wilson 95% khoảng 98% theo ảnh.
- **Quyết định: Hold** — còn 1 ca chưa chốt phạm vi lớp xe đẩy bán hàng ở img_008; đồng thời các lỗi QC cần được sửa và kiểm lại trước khi có thể Release.

Nguồn: `submission/qc/review_stats.json`, `submission/qc/qc_report_auto.md`, `submission/time_log.csv` và `submission/release/release_manifest.json`. Chi tiết các vòng, giới hạn gold và kế hoạch sửa được ghi trong `submission/REPORT.md`.
