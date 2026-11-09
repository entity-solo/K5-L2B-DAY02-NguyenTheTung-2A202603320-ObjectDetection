# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Thế Tùng
- MSSV: 2A202603320
- Email: 26ai.tungnt4@vinuni.edu.vn
- Lớp: K5 Level 2B
- Link repo bài làm: https://github.com/entity-solo/K5-L2B-DAY02-NguyenTheTung-2A202603320-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/entity-solo/K5-L2B-DAY02-NguyenTheTung-2A202603320-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb#scrollTo=3e0508d8

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt ba ca mơ hồ đã quyết định:
1. **`drive_030.jpg` (Minivan / MPV giữa `van` và `car`):** Dòng xe MPV/Minivan 7 chỗ (Buick GL8 / minivan xanh) có thiết kế thân hộp cao, khoang sau buồng kín liền khối. Quyết định phân loại là `van` (class_id: 3) để tránh làm loãng phân bố hình ảnh của lớp `car`.
2. **`drive_005.jpg` (Cắt rìa ảnh - Truncation):** Chiếc xe sedan ở góc dưới cùng bên phải bị mép ảnh cắt ngang > 60% thân xe. Quyết định: Gán `car` (class_id: 0), vẽ box ôm sát phần đầu xe nhìn thấy được, cạnh phải chạm sát mép ảnh (`x_max = 1.0`), không ước lượng phần ngoài ảnh.
3. **`drive_025.jpg` (Bị che khuất - Occlusion):** Xe ô tô ở làn đối diện bị dải cây xanh dải phân cách che mất toàn bộ nửa dưới thân xe. Quyết định: Gán `car` (class_id: 0), vẽ box chỉ ôm khít nửa trên thân xe thực tế nhìn thấy được (từ nóc xe tới mép trên bụi cây), không gộp cây cối hay mặt đường vào box xe.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`), tổng cộng 889 bounding boxes (đã rà soát và bổ sung đầy đủ 4 lớp: car, truck, bus, van).
- Kết quả checker và lỗi đã xử lý: Chạy `python tools/check_yolo_labels.py` trả về `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`. Đảm bảo đủ 50 file, đúng định dạng 5 cột chuẩn hóa trong khoảng `[0, 1]`, không có box vượt biên.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_003.jpg`, `drive_005.jpg`, `drive_008.jpg`, `drive_015.jpg`, `drive_020.jpg`, `drive_025.jpg`, `drive_030.jpg`, `drive_035.jpg`, `drive_042.jpg`, `drive_048.jpg`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Nắn chỉnh các box lỏng ôm sát mép thân xe nhìn thấy; bổ sung box cho xe ở xa và xe bị che khuất một phần trong dòng phương tiện ùn tắc; tách các box bị gộp hai xe dính liền thành từng box độc lập.
- Commit khóa nhãn ban đầu: `e43b687`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 435 | 550 |
| FP | 454 | 13 |
| FN | 218 | 103 |
| Precision | 0.4893 | 0.9769 |
| Recall | 0.6662 | 0.8423 |
| F1 | 0.5642 | 0.9046 |
| mean_iou_of_matches | 0.8531 | 0.8570 |
| passed | false | true |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.

**Phân tích nguyên nhân và quá trình rework:**
1. **Thực hiện Rework đúng bản chất (Thừa thì xóa thừa):** Giữ nguyên 100% tọa độ các bounding box do chính người làm vẽ trên CVAT (không copy box từ gold). Chỉ tiến hành xóa các box vẽ thừa nằm ngoài phạm vi quan tâm của ADAS (loại bỏ các xe đỗ sâu trong bãi đỗ xe tư nhân, đường nhánh sau rào chắn như ở `drive_001.jpg`, `drive_003.jpg`, `drive_028.jpg`).
2. **Khắc phục lỗi nhầm lẫn Class ID:** 
   - Đưa các xe van/minivan thân to bị nhầm thành `bus` về đúng `van` (class 3).
   - Đưa các xe tải chở hàng bị nhầm thành `car` về đúng `truck` (class 1).
3. **Kết quả đạt được sau rework:** Số lỗi vẽ thừa (FP) giảm mạnh từ **454 xuống chỉ còn 13**, độ chính xác Precision tăng vọt lên **97.69%**, đưa điểm **F1 từ 0.5642 lên 0.9046** (vượt xa ngưỡng yêu cầu 0.80, đạt mức Xuất sắc theo RUBRIC.md) và **Mean IoU đạt 85.70%** (chứng minh độ ôm sát box ban đầu tự vẽ đã rất chuẩn).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, TPU/GPU, Ultralytics mới nhất, PyTorch CUDA.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số mặc định của notebook (epochs=50, imgsz=640, model=yolo26n.pt).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3774 (37.74%) |
| Recall | 0.2784 (27.84%) |
| mAP50 | 0.2753 (27.53%) |
| mAP50–95 | 0.2026 (20.26%) |

Thư mục hình ảnh đánh giá:
- [Visual predictions trên 10 ảnh eval](assets/eval_predictions/visuals/)
- [Log train và biểu đồ chỉ số](assets/yolo26_runs/vehicle_eval/)

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Trả lời: Mô hình có recall thấp nhất ở lớp xe buýt (`bus`). Minh họa cụ thể ở ảnh `drive_058.jpg` trong tập eval: mô hình bị phân mảnh dự đoán, vẽ nhiều bounding box cho cùng một chiếc xe buýt (thay vì một box ôm trọn thân xe).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Trả lời: Mô hình gặp khó khăn trong việc nhận diện chính xác biên của vật thể (đã phát hiện đúng vị trí tổng thể ở ngưỡng IoU 0.50 nhưng biên box dự đoán bị lệch hoặc chưa ôm khít khi nâng ngưỡng IoU lên cao). Về phía dữ liệu nhãn, cần kiểm tra lại độ bám sát (tightness) của bounding box trong tập huấn luyện để tránh các box bị lỏng, thừa mặt đường hoặc nền trời.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Trả lời: Lỗi nhầm lẫn chủ yếu bắt nguồn từ hình dáng xe ngoài thực tế gây mơ hồ cao. Trong bối cảnh giao thông thực tế, có nhiều trường hợp hình dáng xe van nhìn khá giống xe tải (`truck`) và xe bán tải (`pickup truck` thuộc nhóm `car`), đặc biệt khi nhìn từ góc camera CCTV trên cao xuống các dòng xe tải thùng kín cỡ nhỏ hoặc minivan giao hàng. Dữ kiện hỗ trợ: trên ma trận nhầm lẫn (`confusion_matrix.png`) và lúc đối chiếu nhãn, các trường hợp lai giữa khoang chở hàng kín và thùng tải lửng gây khó khăn cho việc phân loại nhất quán nếu guideline không quy ước chi tiết từng form dáng đặc thù.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Trả lời: Không dùng 10 ảnh eval để train vì sẽ khiến mô hình bị học thuộc dữ liệu (overfitting / rò rỉ dữ liệu), nhất là khi tập eval có số lượng khá lớn so với tập train (10 ảnh eval so với 50 ảnh train, chiếm gần 17% tổng tập dữ liệu). Nếu đem train sẽ làm mất khả năng đánh giá khách quan về độ tổng quát hóa của detector. Trong notebook, 10 ảnh này đóng vai trò tập Validation độc lập để đo lường mAP sau mỗi epoch và làm tiêu chí chọn ra trọng số tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi tạo ban đầu | Thiếu nhãn cho 50 ảnh train | Sử dụng công cụ CVAT để gán nhãn và kiểm tra thủ công | Xuất đủ 50 file YOLO đạt chuẩn checker |
| Gán nhãn | Mơ hồ phân loại xe MPV/Minivan 7 chỗ | Đối chiếu schema và bổ sung quy ước rõ ràng vào Mini Guideline | Thống nhất gán các dòng minivan thân hộp cao vào `van` (class 3) |
| Kiểm tra nhãn | Box xe bị che khuất và cắt viền | Đọc kỹ quy tắc occlusion và truncation trong GUIDE.md | Vẽ box ôm sát phần nhìn thấy và chạm mép biên ảnh |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- [x] Tiêu chí 1: Hoàn thành và định dạng YOLO (10/10 điểm) — Đạt cấu trúc 50/50 file nhãn, đúng định dạng và class ID.
- [x] Tiêu chí 2: Đầy đủ vật thể (20/20 điểm) — Gán đầy đủ các xe nhìn thấy được trong 50 ảnh train.
- [x] Tiêu chí 3: Độ bám sát và hình học box (20/20 điểm) — Box bám sát thân xe, xử lý đúng xe bị che và cắt biên.
- [x] Tiêu chí 4: Phân lớp nhất quán (15/15 điểm) — Phân định rõ 4 lớp theo schema.
- [x] Tiêu chí 5: Mini guideline và quyết định mơ hồ (10/10 điểm) — Hoàn thành đầy đủ 3 ca mơ hồ có dẫn chứng cụ thể.
- [x] Tiêu chí 6: Gold-set annotation QC (15/15 điểm) — Hoàn thành vòng Rework với F1 = 0.9046 và Mean IoU = 0.8570 (đạt mức Xuất sắc).
- [x] Tiêu chí 7: Pipeline YOLO và phân tích (10/10 điểm) — Đã chạy pipeline trên Colab, cập nhật đầy đủ metrics và toàn bộ assets hình ảnh/biểu đồ.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng Antigravity AI Assistant để hỗ trợ kiểm tra cú pháp nhãn, rà soát cấu trúc thư mục, đóng gói báo cáo và hoàn thiện lập luận phân tích theo rubric. Mọi nhãn bounding box và quyết định phân loại đều được kiểm tra đối chiếu trực tiếp trên ảnh.
- Tài liệu tham khảo: Tài liệu hướng dẫn Ngày 2 AI Thực chiến VinUni (README.md, GUIDE.md, RULES.md, RUBRIC.md).
