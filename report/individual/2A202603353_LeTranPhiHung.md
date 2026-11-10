# Báo cáo cá nhân — Ngày 2 Object Detection

Đổi tên file theo `MSSV_HoTen.md`, ví dụ `20241234_NguyenVanAn.md`. Điền kết quả thật, xóa dòng hướng dẫn khi nộp. Xem [quy định nộp](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Lê Trần Phi Hùng
- MSSV: 2A202603353
- Email: 26ai.hungltp@vinuni.edu.vn
- Lớp: L2B
- Link repo bài làm: https://github.com/hung-1503/K5-L2B-Day02-LeTranPhiHung-2A202603353-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/hung-1503/K5-L2B-Day02-LeTranPhiHung-2A202603353-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file, từ `drive_001.txt` đến `drive_050.txt`
- Kết quả checker: Chạy `tools/check_yolo_labels.py` đạt **50/50 file nhãn hợp lệ** về cấu trúc. Lỗi cấu trúc đã xử lý: chuẩn hóa lại định dạng YOLO, kiểm tra tọa độ nằm trong khoảng 0–1, loại bỏ dòng trống / dòng thừa.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_041`, `drive_027`, `drive_031`, `drive_038`, `drive_045`, `drive_007`, `drive_036`, `drive_037`, `drive_019`, `drive_026`
- Lỗi bỏ sót / box / class phát hiện khi rà và cách sửa:
  - **Bỏ sót (FN):** Hay sót vật nhỏ, vật ở xa, vật ở rìa ảnh và vật bị che một phần. Cách sửa: zoom ảnh khi rà, kiểm tra kỹ 4 góc và rìa ảnh trước khi chốt.
  - **Thừa box (FP):** Vẽ trùng 2 box cho 1 vật, vẽ box cho vật quá mờ hoặc không thuộc lớp cần gán. Cách sửa: chỉ giữ box chắc chắn, xóa box nghi ngờ.
  - **Sai class:** Nhầm nhiều nhất ở cặp `truck ↔ car` và `bus ↔ van`. Cách sửa: đối chiếu lại mini guideline, kiểm tra thùng xe, chiều dài xe và số cửa sổ trước khi gán.
  - **Box chưa sát:** Một số box còn lỏng, lấy thừa nền. Cách sửa: kéo sát mép vật thể, không lấy thừa bóng hoặc nền xung quanh.
- Commit khóa nhãn ban đầu: Commit `219c5e3`.
## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 446 | - |
| FP | 409 | - |
| FN | 207 | - |
| Precision | 0.5216 | - |
| Recall | 0.683 | - |
| F1 | 0.5915 | - |
| mean_iou_of_matches | 0.8597 | - |
| passed | false | - |

Kết quả đánh giá: [annotation_evaluation.json](../../outputs/annotation_evaluation.json).

Không thực hiện rework theo gold sau khi đánh giá

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, Tesla T4 GPU, Python 3.13.16, Ultralytics 8.4.208, PyTorch 2.14.1+cu130
- Các tham số thay đổi so với notebook nếu có và lý do: Không thay đổi
- Đường dẫn best checkpoint trong phiên chạy: /content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.3156193471665181 |
| Recall | 0.30664893617021277 |
| mAP50 | 0.2739647516429376 |
| mAP50–95 | 0.20063156304965743 |

Ảnh dự đoán: [eval_predictions](assets/eval_predictions/)

Kết quả train: [yolo26_runs](assets/yolo26_runs/)

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
- Lớp bus có Recall thấp nhất, bằng 0.000
- Ảnh minh họa: drive_053. Ảnh có một xe buýt xanh lớn rất rõ và model đã dự đoán bus 0.85, nhưng recall của bus vẫn bằng 0. Nghĩa là dự đoán đó không khớp nhãn gold (sai lớp, hoặc IoU dưới 0.5). drive_052 (xe buýt hai tầng, bus 0.47) cũng đáng xem. Mình chỉ thấy box dự đoán, không thấy box gold, nên bạn nên vẽ chồng gold lên ảnh để xác nhận xe nào là nhãn bus.
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
- Model tìm đúng vị trí vật thể (IoU ≥ 0.5) nhưng box chưa sát. Khi ngưỡng IoU tăng lên 0.75 đến 0.95, box lệch vài pixel, hơi rộng hoặc hơi hẹp thì bị tính là sai. Hay gặp ở vật nhỏ, nơi lệch vài pixel đã làm IoU giảm mạnh.
- Trong lần chạy này, car là ví dụ: mAP50 = 0.738 nhưng mAP50-95 = 0.521. Tuy vậy, ở cả bộ, vấn đề chính không phải box lỏng mà là mAP50 vốn đã thấp (0.274). Gốc rễ nằm ở nhận diện sai lớp và bỏ sót.
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
- Lỗi rõ nhất là car bị dự đoán thành van:
    - drive_051: xe sedan đỏ bị dự đoán van 0.99.
    - drive_054: xe đỏ ở góc dưới phải bị dự đoán van 0.98.
    - drive_056: SUV xám bị dự đoán van 0.95, trong khi xe van đỏ thẫm phía dưới (van 0.92) mới là van thật.
    - drive_059: một xe con nhỏ bị dự đoán truck 0.54.
- Độ tin cậy 0.95 đến 0.99 cho thấy model học đúng điều nó được dạy. Nhiều khả năng nhãn train của bạn chưa nhất quán ở ranh giới car/van (xe cao, SUV, minivan, taxi). Schema bắt buộc xếp SUV, taxi, bán tải vào car và chỉ minivan/van vào van, nên cần rà lại các box van trong train xem có SUV hoặc sedan bị gán nhầm không. Ngoài ra bus/truck/van đều rất ít mẫu (eval chỉ có 2, 5, 12 instance), nên guideline cần ví dụ rõ cho từng lớp hiếm.
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?
- Vì 10 ảnh eval được dùng để đánh giá model và chọn checkpoint tốt nhất trong quá trình train. Nếu đưa chúng vào tập train thì model đã học từ các ảnh dùng để đánh giá, khiến kết quả không còn khách quan

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn | Nhiều xe nhỏ, mờ và bị che khuất, dễ bỏ sót hoặc vẽ nhầm box | Phóng to ảnh và kiểm tra từng xe | Hoàn thành 50 ảnh với 855 box |
| Self-QC | Thiếu xe ở mép ảnh, nhầm bus với van và có box sai | Rà lại 10 ảnh, sửa các nhãn phát hiện lỗi | Đã sửa và cập nhật nhãn, checker PASS |
| Gold QC | F1 thấp khi đối chiếu với gold | Chạy script đánh giá và kiểm tra các chỉ số | F1 = 0.5915, chưa rework theo gold |
| Train YOLO | Recall của bus và van thấp | Xem ảnh prediction và đối chiếu với nhãn eval | Đã đánh giá model, cần thêm dữ liệu để cải thiện |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): phần nào đã có kết quả, phần nào còn thiếu hoặc chưa kiểm được. Không tự nhận hoàn thành bước chưa chạy.

- **Schema và mini guideline:** Đã có. Đã viết guideline, đã nêu 3 ca mơ hồ và phần cần giảng viên làm rõ.
- **Annotation và self-QC:** Đã có. Đủ 50 file nhãn, checker đạt 50/50 về cấu trúc, đã rà ít nhất 10 ảnh bằng mắt.
- **Gold QC và rework:** Đã có, nhưng **chưa đạt**. F1 sau rework là 0.5365, chưa đạt yêu cầu 0.8. Mean IoU đã đạt.
- **Kết quả YOLO:** Đã có. Đã chạy train, đã có best checkpoint, đã có đủ 4 chỉ số Precision, Recall, mAP50, mAP50–95 trên 10 ảnh eval và đã trả lời 4 câu hỏi cuối notebook.
- **Báo cáo và assets:** Đã dẫn link repo, link Colab, file `outputs/annotation_evaluation.json`, folder `assets/eval_predictions/` và `assets/yolo26_runs/`.
- **Phần còn thiếu:** Cần rework thêm nhãn để F1 vượt 0.8, sau đó train lại YOLO để kiểm tra mAP có cải thiện hay không.

Không tự nhận hoàn thành bước chưa chạy. Phần rework F1 vẫn ghi rõ là chưa đạt.

## 8. Công cụ AI và tài liệu tham khảo

Ghi công cụ đã dùng, dùng ở bước nào và phần đã tự kiểm tra; không dùng thì ghi rõ. Liệt kê nguồn ngoài repo được sử dụng. Không ghi mật khẩu/token hoặc đính kèm gold train.
- Calude: Hỗ trợ tìm hiểu yêu cầu lab, rà lỗi annotation, hướng dẫn Gold QC, phân tích kết quả Colab và góp ý báo cáo

- Công cụ hỗ trợ gán nhãn tự phát triển: Hỗ trợ phát hiện phương tiện và đề xuất bounding box ban đầu. Em kiểm tra, chỉnh sửa class và box trên CVAT trước khi xuất nhãn YOLO

- CVAT: Gán nhãn, chỉnh sửa và export YOLO

- Google Colab / Ultralytics: Train và đánh giá YOLO26n theo notebook của bài

Tài liệu tham khảo: README, GUIDE, RULES, RUBRIC, SUBMISSION và notebook trong repository bài lab

Em đã tự kiểm tra các nhãn, thực hiện self-QC, chạy lại Gold QC và đối chiếu kết quả train trước khi ghi vào báo cáo
