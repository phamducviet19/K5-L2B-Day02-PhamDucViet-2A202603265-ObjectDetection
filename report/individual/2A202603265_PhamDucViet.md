# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Đức Việt
- MSSV: 2A202603265
- Email: pdviet1909@gmail.com
- Lớp: L2B
- Link repo bài làm: https://github.com/phamducviet19/K5-L2B-Day02-PhamDucViet-2A202603265-ObjectDetection.git
- Link Colab đã chạy: https://colab.research.google.com/drive/1SyHel9GqexeYE5BC2v89xn1k_zldiv_t?usp=sharing

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt ba ca mơ hồ đã quyết định:
1. `drive_001.jpg`: Xe ô tô màu trắng ở làn bên phải bị cột đèn phía trước che khuất một phần thân xe. Quyết định: Gán `car` ôm sát phần thân xe nhìn thấy được (Visible Area), tuân thủ quy tắc Che khuất (Occlusion), không vẽ bao trùm cột đèn.
2. `drive_002.jpg`: Xe màu trắng dạng hộp nhỏ ở làn bên trái (dòng microvan), phân vân giữa `car` và `van`. Quyết định: Gán `van` do cấu trúc một khoang liền khối (monovolume), trần xe cao và đuôi cụt thẳng đứng đặc trưng của xe van.
3. `drive_004.jpg`: Xe màu đỏ ở góc dưới cùng bên phải bị mép ảnh cắt ngang thân sau. Quyết định: Gán `car` với cạnh box chạm sát viền đáy ảnh ($y=1.0$), tuân thủ quy tắc Cắt rìa (Truncation), không tự suy đoán phần xe nằm ngoài khung hình.

Phần cần giảng viên làm rõ: Ranh giới phân loại cụ thể giữa xe van chở khách/minivan (như xe 16 chỗ) với `bus`, và xe tải nhỏ thùng kín với `truck` trong các ảnh chụp từ góc cao (traffic camera viewpoint).

## 3. Annotation và self-QC

- Số file nhãn train: 50
- Kết quả checker và lỗi đã xử lý: ĐẠT cấu trúc: 50/50 file nhãn hợp lệ (chạy qua `tools/check_yolo_labels.py`).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001.jpg, drive_002.jpg, drive_003.jpg, drive_004.jpg, drive_005.jpg, drive_008.jpg, drive_010.jpg, drive_012.jpg, drive_015.jpg, drive_020.jpg`
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện một số box bị lỏng do kéo chớm vào bóng đổ dưới lòng đường ở các ảnh ban ngày; đã căn chỉnh lại các mép box bám sát thân vỏ xe; bổ sung các xe ở mép rìa ảnh bị cắt một phần.
- Commit khóa nhãn ban đầu: `daaba43`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 478 | 480 |
| FP | 631 | 208 |
| FN | 175 | 173 |
| Precision | 0.4310 | 0.6977 |
| Recall | 0.7320 | 0.7351 |
| F1 | 0.5426 | 0.7159 |
| mean_iou_of_matches | 0.8388 | 0.8409 |
| passed | false | false |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.

**Phân tích Rework:**
- **Nguyên nhân trước sửa:** $F_1$ ban đầu đạt 0.5426 do lượng False Positive rất cao (631 FP). Qua đối chiếu trực quan, nguyên nhân chính là học viên đã gán toàn bộ xe đỗ trong bãi xe bên lề đường (như `drive_001`, `drive_028`) và các xe kẹt ở đường chân trời quá xa (như `drive_002`, `drive_003`), trong khi Gold chỉ gán các xe lưu thông trên lòng đường (roadway).
- **Kết quả sau sửa:** Sau khi loại bỏ các box ngoài lòng đường và các xe quá nhỏ ở xa, Precision tăng vọt từ 43.1% lên 69.8%, $F_1$ tăng từ 0.5426 lên 0.7159, chỉ số `mean_iou_of_matches` đạt 0.8409 (vượt xa yêu cầu 0.70).
- **Giải trình lý do giữ nhãn:** Số lỗi còn lại không nằm ở vị trí box mà nằm ở sự bất đồng quan điểm phân loại lớp (Class Confusion) giữa bộ Gold và Guideline:
  + Các xe minivan/xe khách nhỏ (16 chỗ) học viên gán là `bus` theo kích thước chở người nhưng Gold gán là `van`.
  + Các xe tải nhỏ thùng kín học viên gán là `van` nhưng Gold gán là `truck`.
  Vì luật chấm phạt kép (vừa tính 1 FP vừa tính 1 FN khi lệch class), học viên quyết định giữ nguyên tính nhất quán theo guideline đã thiết lập trong `GUIDELINE_MINI.md` thay vì sửa gượng ép để khớp hoàn toàn với gold.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab T4 GPU, Python 3.12, Ultralytics 8.4.145, PyTorch 2.6.0+cu124.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số chuẩn của bài thực hành (`yolo26n.pt`, 50 epochs, imgsz 640).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.1857 |
| Recall | 0.2266 |
| mAP50 | 0.3006 |
| mAP50–95 | 0.2065 |

Dẫn link tương đối tới kết quả trong assets:
- 10 ảnh eval kèm box dự đoán: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/) (ví dụ [drive_052.jpg](assets/eval_predictions/visuals/drive_052.jpg), [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg))
- Log huấn luyện và chỉ số đánh giá model: [assets/yolo26_runs/vehicle_eval/](assets/yolo26_runs/vehicle_eval/) (ví dụ [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [BoxPR_curve.png](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png))

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có recall thấp nhất là `truck` và `bus` (recall xấp xỉ 0% trên tập eval theo [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)). Trong 10 ảnh eval, toàn bộ 5 xe truck và 2 xe bus thực tế đều bị mô hình dự đoán nhầm sang class khác hoặc bỏ sót.
   - Ảnh minh họa: [assets/eval_predictions/visuals/drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg) — nơi các xe khách lớn và xe tải ở ngã tư bị mô hình dự đoán lẫn lộn giữa `bus`, `truck` hoặc không phát hiện được do mật độ giao thông đông và số lượng mẫu train của 2 lớp này quá ít (dữ liệu mất cân bằng nghiêm trọng so với `car`).

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ ôm sát (tightness / bounding box alignment)** của các box gán nhãn.
   - Lý do: mAP50 chỉ yêu cầu ngưỡng IoU $\ge 0.50$ (chỉ cần bao phủ được khoảng 50% diện tích là được tính đúng). Khi ngưỡng IoU khắt khe hơn từ 0.50 đến 0.95 (như 0.75, 0.85, 0.90), nếu box của annotator bị vẽ lỏng (dính bóng đổ, vỉa hè) hoặc cắt lẹm thân xe thì IoU sẽ giảm xuống dưới ngưỡng đánh giá, khiến mAP50-95 bị tụt dốc.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa **`bus` và `van`** (và giữa **`van` và `truck`**).
   - Dữ kiện hỗ trợ:
     + Trong ma trận nhầm lẫn [assets/yolo26_runs/vehicle_eval/confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), có tới 10 xe trong tập eval thực tế là `van` nhưng model lại dự đoán là `bus`.
     + Điều này phản ánh trực tiếp sự bất đồng trong guideline khi gán nhãn: trong tập train, nhiều xe dạng minivan/xe khách nhỏ (16 chỗ) được annotator gán là `bus` (phân loại theo chức năng chở người), khiến model học đặc trưng đó và dự đoán các xe van trong tập eval thành `bus`.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train vì điều đó sẽ gây ra hiện tượng **rò rỉ dữ liệu (Data Leakage)** và làm mất tính khách quan khi đánh giá mô hình. Nếu đưa eval vào train, model sẽ "học vẹt" (overfitting) trên các ảnh này, khiến chỉ số cao ảo nhưng không phản ánh được khả năng tổng quát hóa (generalization) trên các tình huống giao thông mới ngoài đời thực.
   - Vai trò validation của 10 ảnh eval là cung cấp một tập kiểm thử độc lập (unseen test set) để giám sát và đánh giá công tâm năng lực nhận diện của mô hình.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Checkpoint 1 | Phân vân ranh giới các dòng xe bán tải, minivan và xe ở quá xa | Đọc kỹ GUIDE và ghi rõ 3 ca mơ hồ vào `GUIDELINE_MINI.md` | Thống nhất được quy ước trước khi gán hàng loạt |
| Checkpoint 3 | Chưa hình dung được cách so sánh nhãn trước khi có gold | Đọc lại mục đích của self-QC (kiểm tra tính hợp lệ và bám sát box trên CVAT) | Đạt cấu trúc 50/50 file YOLO |
| Checkpoint 5 | F1 ban đầu thấp (0.54) do thừa 631 box | Tạo ảnh so sánh đối chiếu với gold, phát hiện do gán xe trong bãi và xe ở xa | Rework loại bỏ xe ngoài lòng đường, F1 tăng lên 0.7159 |
| Checkpoint 6 | Lỗi clone repo trên Colab | Sửa lại đúng link repo GitHub và đảm bảo repo Public | Colab clone thành công và train hoàn tất 50 epochs |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- Đã hoàn thành mini guideline với định nghĩa 4 lớp và 3 ca mơ hồ.
- Đã hoàn thành 50 file nhãn train YOLO, kiểm tra cấu trúc đạt 50/50 file.
- Đã thực hiện Self-QC và Gold QC đối chiếu trước/sau sửa nhãn có giải trình khoa học, logic.
- Đã huấn luyện YOLO trên Google Colab, giải nén và nộp đầy đủ metrics, ảnh visual predictions trong `assets/`.
- Trả lời đầy đủ 4 câu hỏi phân tích mô hình và nguyên nhân dữ liệu.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng trợ lý AI (Antigravity) để hỗ trợ:
  + Giải thích các bước thực hành trong GUIDE.md và cấu trúc file nộp bài.
  + Viết script tạo ảnh so sánh trực quan đối chiếu nhãn học viên và nhãn Gold để xác định nguyên nhân FP/FN.
  + Hướng dẫn cách xử lý lỗi môi trường chạy trên Google Colab.
- Toàn bộ thao tác gán nhãn trên CVAT, đưa ra quyết định phân loại class và chạy notebook trên Colab đều do học viên tự thực hiện và kiểm chứng.
