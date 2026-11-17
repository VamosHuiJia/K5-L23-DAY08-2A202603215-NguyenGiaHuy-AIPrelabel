# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 145 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 8/145 = 5.5% | 145 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 7/8 = 87.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 347/451 = 76.9% / 31/451 = 6.9% / 70/451 = 15.5% | 451 box máy |
| Máy: added | 41/419 = 9.8% | 419 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 3 | img_003.jpg, img_008.jpg, img_017.jpg |
| thua | 3 | img_006.jpg, img_027.jpg, img_034.jpg |
| bo_sot | 2 | img_003.jpg, img_035.jpg |

## Giới hạn thống kê
- 7 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **98%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**HOLD**

- guideline chưa rõ (1 ca) / dữ liệu có vấn đề (0 ảnh) -> giữ lại làm rõ

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
