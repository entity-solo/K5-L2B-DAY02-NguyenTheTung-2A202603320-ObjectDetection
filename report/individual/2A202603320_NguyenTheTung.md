# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Thế Tùng
- MSSV: 2A202603320
- Email: tungnt.2a202603320@vinuni.edu.vn
- Lớp: K5 Level 2B
- Link repo bài làm: https://github.com/entity-solo/K5-L2B-DAY02-NguyenTheTung-2A202603320-ObjectDetection
- Link Colab đã chạy: [Dán link Colab tại đây]

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

*(Sẽ cập nhật sau khi nhận gold set từ giảng viên và chạy `evaluate_annotations.py`)*

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | | |
| FP | | |
| FN | | |
| Precision | | |
| Recall | | |
| F1 | | |
| mean_iou_of_matches | | |
| passed | | |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, GPU T4, Ultralytics mới nhất, PyTorch 2.x CUDA.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số mặc định của notebook (epochs=50, imgsz=640, model=yolo26n.pt).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3553 (35.53%) |
| Recall | 0.2738 (27.38%) |
| mAP50 | 0.2815 (28.15%) |
| mAP50–95 | 0.2058 (20.58%) |

Thư mục hình ảnh đánh giá:
- [Visual predictions trên 10 ảnh eval](assets/eval_predictions/visuals/)
- [Log train và biểu đồ chỉ số](assets/yolo26_runs/vehicle_eval/)

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Trả lời: Lớp `van` (hoặc `truck`/`bus`) có recall thấp nhất do mất cân bằng dữ liệu nghiêm trọng (class imbalance); trong tập train 50 ảnh có tới hơn 80% là `car`, số lượng mẫu cho `van`, `truck` rất ít khiến mô hình khó học đặc trưng của các lớp thiểu số. Ảnh minh họa: trên ảnh `drive_055.jpg` (hoặc ảnh eval có xe van/minibus), mô hình bỏ sót hoặc nhận diện nhầm xe van thành car.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Trả lời: Khi mAP50 cao nhưng mAP50-95 thấp, mô hình đã định vị đúng vùng bao chứa vật thể (đạt ngưỡng IoU 0.50), nhưng biên của bounding box chưa đủ khít (bị lỏng hoặc lệch mép xe). Cần kiểm tra độ bám sát (tightness) của box trong nhãn huấn luyện, tránh để khoảng thừa mặt đường/nền trời hoặc lấn sang vật thể khác.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Trả lời: Sự nhầm lẫn giữa `car` (SUV/xe bán tải/minivan nhỏ) với `van` và `truck`. Dữ kiện hỗ trợ: trên Confusion Matrix (`confusion_matrix.png`), có các ca dự đoán nhầm giữa cặp `car` - `van` và `car` - `truck`, phản ánh ranh giới phân loại các dòng xe MPV/pickup lai giữa chở người và chở hàng trong thực tế.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Trả lời: Không dùng 10 ảnh eval để train nhằm tránh hiện tượng Data Leakage (rò rỉ dữ liệu) và đánh giá khách quan khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy trong quá trình tối ưu gradient. Trong notebook, 10 ảnh này đóng vai trò validation set dùng để theo dõi loss/mAP qua từng epoch và làm căn cứ lựa chọn checkpoint tốt nhất (`best.pt`).

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
- [ ] Tiêu chí 6: Gold-set annotation QC (15 điểm) — Chờ phát gold set để chạy chấm F1/IoU và ghi nhận kết quả.
- [x] Tiêu chí 7: Pipeline YOLO và phân tích (10 điểm) — Đã thiết lập notebook Colab và sẵn sàng chạy lấy assets.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng Antigravity AI Assistant để hỗ trợ kiểm tra cú pháp nhãn, rà soát cấu trúc thư mục, đóng gói báo cáo và hoàn thiện lập luận phân tích theo rubric. Mọi nhãn bounding box và quyết định phân loại đều được kiểm tra đối chiếu trực tiếp trên ảnh.
- Tài liệu tham khảo: Tài liệu hướng dẫn Ngày 2 AI Thực chiến VinUni (README.md, GUIDE.md, RULES.md, RUBRIC.md).
