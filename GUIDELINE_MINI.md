# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe chở người cỡ nhỏ (sedan, hatchback, SUV gia đình, crossover thông thường). | Thường từ 4-7 chỗ; phân biệt với van ở dáng xe và trần xe; không có thùng hàng rời như truck. |
| `truck` | Xe tải chở hàng hạng trung và nặng (xe tải thùng, xe ben, đầu kéo container, bồn). | Cabin tách rời với thùng/rơ-moóc hàng phía sau; thân xe to, gầm cao hơn ô tô con. |
| `bus` | Xe khách, xe chở khách công cộng hoặc du lịch cỡ lớn. | Thân dài hình hộp chữ nhật lớn (>= 16-29 chỗ), nhiều hàng kính dọc thân xe. |
| `van` | Xe chở hàng nhẹ hoặc người dạng một khoang liền (monovolume, Ford Transit, cứu thương). | Khoang lái liền với thân xe; trần cao hơn car nhưng nhỏ hơn bus; không có thùng rời như truck. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Ôm sát toàn bộ thân vỏ xe nhìn thấy được (không lấy bóng đổ, mặt đường hay xe bên cạnh).
- Vật thể bị che khuất: Chỉ vẽ box quanh phần nhìn thấy được (visible part) nếu nhận diện chắc chắn được lớp; không tự đoán phần bị che.
- Vật thể bị cắt bởi rìa ảnh: Kéo cạnh box chạm đúng mép viền ảnh; không vẽ box vượt ra ngoài khung hình.
- Vật thể rất nhỏ/mơ hồ: Bỏ qua không gán nếu xe quá xa hoặc quá nhỏ không xác định được lớp; không gán xe máy, xe đạp, người đi bộ, biển quảng cáo.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Xe ô tô màu trắng ở làn bên phải bị cột đèn/biển báo phía trước che khuất một phần thân xe. | Gán `car` ôm sát phần thân xe nhìn thấy được | Tuân thủ quy tắc che khuất (Occlusion): chỉ vẽ box bao quanh phần nhìn thấy (visible area), không vẽ trùm qua cột đèn. |
| `drive_002.jpg` | Xe màu trắng dạng hộp nhỏ ở làn bên trái (dòng microvan), phân vân giữa `car` và `van`. | Gán `van` | Thiết kế một khoang liền khối (monovolume), trần xe cao và đuôi đứng đặc trưng của dòng van chở hàng/khách nhỏ. |
| `drive_004.jpg` | Xe làn đối diện phía trên bên trái bị đèn pha chiếu thẳng gây lóa sáng mạnh, thân xe nhòe tối không rõ biên dạng (car, van hay xe máy). | Bỏ qua không gán nhãn cho các quầng sáng bị lóa | Tuân thủ quy tắc vật thể mờ/thiếu sáng: không tự đoán lớp khi mất biên dạng thân vỏ; tránh gán box vào vệt sáng đèn pha gây nhiễu mô khung hình. |
