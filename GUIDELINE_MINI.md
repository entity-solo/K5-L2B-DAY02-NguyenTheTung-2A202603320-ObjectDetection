# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con cá nhân, xe gia đình từ 4 đến 7 chỗ, xe taxi, sedan, hatchback, SUV, crossover và xe bán tải (pickup cabin nhỏ). | Toyota Vios, Camry, Hyundai Grand i10, Mazda CX-5, Ford Ranger (bán tải cabin kép). |
| `truck` | Xe tải chở hàng hóa, xe ben công trình, xe bồn, container, xe đầu kéo. | Xe tải thùng Isuzu, Hyundai Porter thùng lửng chở hàng, container đầu kéo. |
| `bus` | Xe buýt công cộng nội đô, xe khách liên tỉnh lớn, xe đưa đón học sinh/công nhân, minibus chở khách du lịch. | Xe buýt 2 tầng, xe khách 29-45 chỗ, xe buýt chạy tuyến cố định, xe buýt khớp nối 2 toa. |
| `van` | Xe van, minivan thân hộp kín chuyên chở hàng hóa nhỏ hoặc chở người từ 7-16 chỗ dạng buồng kín. | Wuling Sunshine, Ford Transit, Buick GL8, Toyota HiAce, minivan giao hàng bưu phẩm. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Bounding box ôm sát mép ngoài cùng của phần thân xe nhìn thấy được, không thừa lề đường, mặt nhựa, nền trời hay lấn sang phương tiện bên cạnh.
- **Vật thể bị che khuất:** Nếu bị vật cản lớn (bụi cây, xe khác, công trình) che một phần thân xe thì chỉ vẽ box quanh phần diện tích thân xe thực tế nhìn thấy được, miễn là phần lộ ra còn đủ dấu hiệu nhận diện lớp.
- **Vật thể bị cắt bởi rìa ảnh:** Vẽ box ôm sát phần xe xuất hiện trong ảnh, cạnh box tiếp giáp mép ảnh; tuyệt đối không mở rộng tọa độ ra ngoài khung hình (tọa độ giữ trong khoảng [0, 1]).
- **Vật thể rất nhỏ/mơ hồ:** Bỏ qua các xe ở quá xa cuối tiêu cự (dưới 10x10 pixels) hoặc bị che khuất trên 85% không thể xác định chính xác class_id để tránh làm nhiễu mô hình.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_030.jpg` | Xe MPV/Minivan 7 chỗ (dạng Buick GL8 / minivan xanh) di chuyển ở làn giữa, ranh giới giữa car và van. | Gán nhãn `van` (class_id: 3). | Thân xe dạng hộp cao, khoang sau liền khối đặc trưng của minivan/MPV chở người, theo schema xếp vào `van` để phân biệt với xe con `car`. |
| `drive_005.jpg` | Chiếc xe sedan màu xám ở góc dưới cùng bên phải bị mép ảnh cắt ngang hơn 60% thân xe. | Gán nhãn `car` (class_id: 0), box ôm sát phần nắp capo và bánh trước nhìn thấy được, cạnh phải chạm biên ảnh. | Phần nhìn thấy đủ nhận diện rõ ràng là xe sedan; tuân thủ quy tắc truncation: vẽ phần nhìn thấy, chạm biên ảnh và không đoán phần ngoài ảnh. |
| `drive_025.jpg` | Chiếc xe ô tô màu xám đen di chuyển ở làn đối diện bị dải bụi cây xanh giữa đường che mất toàn bộ nửa dưới thân xe. | Gán nhãn `car` (class_id: 0), box chỉ ôm khít nửa trên thân xe nhìn thấy được (từ nóc xe đến mép trên bụi cây). | Tuân thủ quy tắc occlusion: chỉ vẽ box phần thân xe thực tế nhìn thấy được; không bao gồm cây cối hay mặt đường vào box xe. |
