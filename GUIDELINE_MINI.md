# Mini guideline — bài làm cá nhân

> Điền trước khi gán nhãn (CP1), giữ lại các quyết định sau khi self-QC. Các ảnh nêu dưới đây là ảnh có thật trong `dataset/images/train/`.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con chở người, thân liền, ≤ 7 chỗ | sedan, SUV, hatchback; thấp, nóc ngắn, không có thùng hàng |
| `truck` | Xe chở hàng, có thùng hàng tách biệt với cabin | xe tải nhỏ/lớn, xe bán tải chở hàng; nhìn thấy thùng hàng phía sau cabin |
| `bus` | Xe khách/buýt lớn, nhiều cửa sổ dọc thân | xe buýt, xe khách giường/ghế; dài, cao, hàng cửa sổ liên tục |
| `van` | Xe van/minivan: thân liền như car nhưng to và cao hơn, không có thùng hàng rời | minivan, xe van chở hàng/người; cao hơn car, nhỏ hơn bus, ít cửa sổ hơn bus |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** bốn cạnh trùng với mép ngoài của phần thân xe nhìn thấy (lốp–nóc–đuôi–đầu), không lấy thêm mặt đường, bóng đổ, cột điện hay xe bên cạnh; sai số cho phép khoảng 1–2 px khi xe ở xa.
- **Vật thể bị che khuất:** vẫn gán một box quanh phần nhìn thấy được nếu còn nhận ra lớp (che sau cột, sau xe khác, bị người đi bộ che). Không đoán phần bị che, không nới box sang vật che; nếu phần nhìn thấy < ~15% thân xe và không xác định được lớp thì bỏ qua và ghi vào bảng dưới.
- **Vật thể bị cắt bởi rìa ảnh:** box chạm đúng rìa ảnh, chỉ ôm phần nằm trong khung, không kéo box ra ngoài để "đủ xe". Vẫn gán lớp nếu phần còn lại đủ nhận diện (nửa đầu xe, nửa đuôi xe).
- **Vật thể rất nhỏ/mơ hồ:** gán nếu nhìn ra được loại xe (thường ≥ ~10 px chiều rộng); xe chỉ còn vài pixel, mờ do xa hoặc nhoè chuyển động thì không gán, tránh box rác. Xe đang đỗ, xe dừng chờ đèn vẫn gán bình thường.
- **Ngoài schema:** không gán người đi bộ, xe đạp, mô tô/xe máy, xe ba bánh, biển báo, đèn tín hiệu, bảng quảng cáo có hình xe, ảnh phản chiếu trong kính.
- Mỗi xe hợp lệ là một box riêng, kể cả khi cùng lớp và chồng nhau; một ảnh không có xe hợp lệ vẫn có file `.txt` rỗng.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_021` | Minivan bạc ở góc dưới trái, thân liền, trần cao, đuôi vuông; dễ bị gán `car` vì kích thước gần sedan. | Gán `van`. | Một khối thân liền từ đầu đến đuôi, trần cao và đuôi vuông đặc trưng van/minivan, không có khoang hành lý thấp như sedan. |
| `drive_047` | Xe van bạc bị cắt bởi rìa trái ảnh, chỉ còn khoảng 2/3 thân xe. | Gán `van`, box chạm rìa trái. | Vẫn nhận diện được lớp nhờ trần cao, thân liền; quy tắc cắt khung: box chỉ ôm phần nhìn thấy, không suy phần ngoài ảnh. |
| `drive_050` | Xe buýt đỏ bị cắt mạnh ở đáy ảnh (chỉ thấy nóc và phần trên thân), đồng thời có nhiều xe rất nhỏ phía xa trên đường. | Xe buýt gán `bus` với box chạm rìa dưới; xe phía xa chỉ gán khi phân biệt được loại, còn lại bỏ qua. | Xe buýt còn đủ đặc trưng (thân dài, trần cao, dãy cửa sổ) nên vẫn gán; xe xa chỉ vài pixel không đủ nhận diện lớp nên không đoán, tránh FP. |
