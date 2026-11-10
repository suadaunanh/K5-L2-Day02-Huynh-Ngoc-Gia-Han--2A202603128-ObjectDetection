# Báo cáo cá nhân — Ngày 2 Object Detection

Đổi tên file theo `MSSV_HoTen.md`, ví dụ `20241234_NguyenVanAn.md`. Điền kết quả thật, xóa dòng hướng dẫn khi nộp. Xem [quy định nộp](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Huỳnh Ngọc Gia Hân 
- MSSV: 2A202603128
- Email: huynh.han.ctc@gmail.com
- Lớp: L2A
- Link repo bài làm: https://github.com/suadaunanh/K5-L2-Day02-HuynhNgocGiaHan-2A202603128-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1IfwMQO2B7hbVH4cbla99VheNTB-PmKnP#scrollTo=pNpBtqZcJqFr

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ.

- Cách áp dụng schema: 4 lớp theo data.yaml (0 car, 1 truck, 2 bus, 3 van). Phân biệt chính: van là thân hộp liền khối, không có cabin tách khỏi thùng; truck có cabin tách riêng với thùng hàng; bus thân dài, nhiều hàng cửa sổ. Box ôm sát phần nhìn thấy, xe bị che/cắt khung vẫn gán nếu nhận ra lớp, không gán mô tô/xe máy, phản chiếu, bảng quảng cáo.
- Các ca mơ hồ đã quyết định:
drive_027 (x≈0.80, y≈0.49): xe nhỏ thân hộp, cửa lùa → van.
drive_009 (x≈0.29, y≈0.38): xe tải bưu chính thùng kín → truck (cabin tách khỏi thùng nên không phải van).
drive_009 (x≈0.67, y≈0.57): xe buýt thành phố đỏ trắng → bus.
drive_029 (x≈0.02, y≈0.79): vật thể tối màu sát mép trái, không nhận ra là xe → không gán.
- Phần cần giảng viên làm rõ: thứ tự lớp trong gold train và dataset/labels/eval có vẻ là car/van/truck/bus, khác với data.yaml (xem mục 4). Đã báo thầy và được thầy xác nhận nhãn chuẩn có sai sót.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 (dataset/labels/train/, gán bằng CVAT, export YOLO 1.1).
- Kết quả checker và lỗi đã xử lý: python3 tools/check_yolo_labels.py → ĐẠT cấu trúc: 50/50 file nhãn hợp lệ (cả bản khóa ban đầu và bản sau sửa). Lỗi đã xử lý: bản export CVAT nằm trong thư mục con obj_train_data/ kèm obj.data, obj.names, train.txt; chỉ chép 50 file .txt vào dataset/labels/train/. Lần chấm đầu ra F1 = 0 do dataset/labels/train/ còn trống, đã chép nhãn vào rồi chạy lại.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: drive_008, drive_009, drive_010, drive_026, drive_027, drive_028, drive_029, drive_030, drive_041, drive_045.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Bỏ sót xe ở ảnh đông xe: vẽ thêm box (số box so với bản export CVAT trước khi sửa) ở drive_041 (11 → 19 box), drive_045 (12 → 19), drive_027 (22 → 28), drive_009 (19 → 22), drive_028 (10 → 13), drive_026 (34 → 36), drive_030 (24 → 26).
drive_029: sửa lại lớp các box, sau sửa khớp 17/19 xe trong gold, không còn box sai lớp.
Box của xe nhỏ ở xa còn rộng hơn xe (IoU 0.2–0.48 ở drive_045, drive_041) → bổ sung quy tắc "phóng to ảnh khi vẽ" vào guideline.
- Commit khóa nhãn ban đầu: 96b7326 — "Add YOLO labels and guideline" (09/11/2026 22:54).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 376 | 413 |
| FP | 247 | 368 |
| FN | 277 | 240 |
| Precision | 0.6035 | 0.5288 |
| Recall | 0.5758 | 0.6325|
| F1 | 0.5893 | 0.5760 |
| mean_iou_of_matches | 0.8550 | 	0.8376 |
| passed | false | false |

Dẫn tới `outputs/annotation_evaluation.json` trong repo. Nếu rework, ghi ảnh, lỗi đã kiểm, quyết định sửa/giữ và lý do. Không suy nguyên nhân chỉ từ FP/FN. Không có rework thì ghi rõ.

Kết quả: outputs/annotation_evaluation_before.json (trước sửa) và outputs/annotation_evaluation.json (sau sửa).

Rework: sửa 10 ảnh liệt kê ở mục 3, chủ yếu vẽ thêm xe bị sót. Recall tăng 0.576 → 0.633 (FN giảm 277 → 240), nhưng FP tăng vì nhiều box mới vẽ cho xe nhỏ ở xa bị rộng hoặc lệch (IoU < 0.5), nên F1 gần như không đổi. Mean IoU vẫn đạt (> 0.70).

Lý do F1 thấp — nhãn gold lệch thứ tự lớp so với data.yaml: Khi đối chiếu các box trùng vị trí (IoU ≥ 0.5) nhưng khác lớp, lỗi lặp lại theo quy luật cố định: bus của tôi ↔ gold id 3 (42 box), van ↔ gold id 1 (18 box), truck ↔ gold id 2 (15 box). Kiểm tra bằng mắt cho thấy gold train và dataset/labels/eval có vẻ dùng thứ tự car/van/truck/bus, khác với data.yaml (car/truck/bus/van):

drive_027 (x≈0.80, y≈0.49): xe thân hộp nhỏ, cửa lùa (van) → gold ghi id 1 (truck).
drive_009 (x≈0.67, y≈0.57): xe buýt thành phố đỏ trắng → gold ghi id 3 (van).
Eval drive_052: xe buýt 2 tầng → labels/eval ghi id 3 (van).
Đã báo với 

Đối chiếu: nếu đọc gold theo thứ tự car/van/truck/bus, F1 bản trước sửa là 0.660 và bản sau sửa là 0.667 (thay vì 0.589 / 0.576). Như vậy một phần điểm thấp đến từ gold; phần còn lại là lỗi của tôi: bỏ sót xe và box thừa ở xe nhỏ phía xa (drive_041, drive_027, drive_024, drive_012, drive_050), box xe nhỏ còn rộng.

Quyết định: giữ nguyên lớp theo schema data.yaml, không đổi lớp để khớp gold và không sửa gold.
## 5. Kết quả YOLO

- Runtime: Google Colab, GPU Tesla T4 (15 GB), Ultralytics 8.4.208, Torch 2.14.1+cu130, Python 3.13.16.
- Các tham số thay đổi so với notebook nếu có và lý do: giữ nguyên notebook (yolo26n.pt, epochs=50, imgsz=640, batch=-1, seed=20260909). Colab clone repo ở commit ebb040b, tức train bằng bộ nhãn khóa ban đầu 96b7326 (539 car, 26 truck, 58 bus, 0 van).
- Đường dẫn best checkpoint trong phiên chạy: /content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt 

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.888 |
| Recall | 0.306 |
| mAP50 | 0.318 |
| mAP50–95 | 0.211 |

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.

- drive_052: xe buýt 2 tầng giữa ảnh bị đoán là car 0.39, box chỉ phủ đầu xe. Sai lớp và box thiếu.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa? truck (0/5) và van (0/12) có recall = 0. Model gán gần như mọi xe là car: xe buýt 2 tầng ở drive_052 bị dự đoán car 0.39 và box chỉ phủ đầu xe; xe buýt xanh ở drive_053 bị tách thành nhiều box car (0.30, 0.58, 0.88); xe buýt vàng bên trái drive_058 và xe thân hộp đỏ ở drive_056 (car 0.95) cũng bị gán car. Nguyên nhân: dữ liệu train lệch nặng (539 car so với 26 truck, 58 bus) và bộ nhãn train không có box van nào nên model không thể học lớp này. 
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box? Với car, mAP50 0.705 nhưng mAP50–95 chỉ 0.418: model tìm đúng xe nhưng box chưa khít, rớt ở ngưỡng IoU 0.75–0.95. Cần kiểm: cạnh box có ôm sát thân xe hay lấn mặt đường/bóng xe (đặc biệt xe nhỏ ở xa); xe lớn có bị tách thành nhiều box (drive_053); xe bị che/cắt khung có được vẽ đúng phần nhìn thấy; nhãn train có nhất quán độ chặt (khi chấm annotation, box xe nhỏ của tôi thường rộng hơn gold 1.5–2 lần).
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét? Ranh giới van / truck / bus. Dữ kiện: (a) khi chấm annotation, các box đúng vị trí nhưng khác lớp theo quy luật cố định bus ↔ gold 3, van ↔ gold 1, truck ↔ gold 2; (b) xem ảnh: gold gán id 3 cho xe buýt (drive_009, eval drive_052) và id 1 cho xe hộp nhỏ (drive_027), trái với data.yaml; (c) tập eval dùng cùng cách gán nên kể cả khi model nhận ra xe buýt cũng bị tính sai lớp, góp phần làm recall van = 0. Ngoài ra bộ nhãn train ban đầu thiếu hẳn lớp van, cho thấy định nghĩa van trong guideline lúc gán nhãn chưa được áp dụng nhất quán.
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì? 10 ảnh eval phải là dữ liệu model chưa thấy để đo khả năng tổng quát hóa; đưa vào train sẽ rò rỉ dữ liệu và làm chỉ số cao ảo (notebook kiểm tra và in "eval không đi vào train"). Trong notebook, data.yaml đặt val: images/eval: sau mỗi epoch YOLO tính P/R/mAP trên 10 ảnh này để chọn best.pt, sau đó best_model.val() và bước predict trực quan cũng chạy trên tập này. Lưu ý: vì best.pt được chọn theo chính tập eval nên chỉ số có thể hơi lạc quan; chỉ 10 ảnh và 2 xe buýt nên chỉ số theo lớp dao động mạnh.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
| Gold QC |Nhiều box đúng vị trí nhưng sai lớp theo quy luật cố định | Cắt ảnh từng ca để xem; thử đọc gold theo thứ tự lớp khác | Gold có vẻ dùng car/van/truck/bus; F1 đối chiếu ≈ 0.66–0.67; labcoach xác nhận gold sai; giữ nhãn theo schema |
| Rework | Thêm xe nhưng F1 không tăng | Phân tích từng box mới: nhiều box xe nhỏ bị rộng (IoU < 0.5) | Recall tăng 0.576 → 0.633, FP tăng; cần vẽ sát hơn ở xe nhỏ |
| Train YOLO | Recall truck/van = 0 | Kiểm tra phân bố lớp của bộ nhãn train | Bộ train có 0 van, 26 truck; dữ liệu lệch nặng về car |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): phần nào đã có kết quả, phần nào còn thiếu hoặc chưa kiểm được. Không tự nhận hoàn thành bước chưa chạy.
| Tiêu chí |	Tự đánh giá |
| Hoàn thành và định dạng YOLO (10) |	Đạt: 50 file, checker chạy đạt |
| Đầy đủ vật thể (20) |	Một phần: còn bỏ sót xe ở ảnh đông xe (recall 0.63) |
| Độ bám sát box (20) |	Khá: mean IoU 0.84; box xe nhỏ ở xa còn rộng |
| Phân lớp nhất quán (15)	| Một phần: lớp gán theo data.yaml; bộ khóa ban đầu chưa gán van | 
| Mini guideline (10) |	Đạt: có định nghĩa lớp, quy tắc box, 4 ca mơ hồ có lý do |
| Gold-set QC (15) |	Chưa đạt ngưỡng F1 0.80; có vòng rework và giải thích nguyên nhân (gold lệch thứ tự lớp, thầy xác nhận) |
| Pipeline YOLO và phân tích (10) |	Đạt: Colab chạy được, không dùng eval để train, có metrics, ảnh eval và nhận xét lỗi |

## 8. Công cụ AI và tài liệu tham khảo

Ghi công cụ đã dùng, dùng ở bước nào và phần đã tự kiểm tra; không dùng thì ghi rõ. Liệt kê nguồn ngoài repo được sử dụng. Không ghi mật khẩu/token hoặc đính kèm gold train.

Claude (Anthropic): chạy check_yolo_labels.py và evaluate_annotations.py, thống kê lỗi theo từng ảnh và từng lớp, cắt ảnh để xem các ca sai lớp (phát hiện gold lệch thứ tự lớp), gợi ý nội dung quy tắc chung / bảng ca mơ hồ trong guideline và soạn báo cáo này. Các số liệu lấy trực tiếp từ output của script và notebook Colab.
CVAT để gán nhãn; Google Colab + Ultralytics để train/eval.
Không dùng nguồn ngoài repo. Không đính kèm gold train.
