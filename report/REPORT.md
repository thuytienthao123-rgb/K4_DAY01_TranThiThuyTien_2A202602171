# Báo cáo bài thực hành Ngày 1 – Đọc nhãn từ đầu ra YOLO11

**Ngày chạy: 11/09/2026**

**Runtime Colab:** CPU/GPU 

**Python / PyTorch / Ultralytics: 3.13.15/ 2.11.0+cu128/ 8.4.145**

**Checkpoint:** `yolo11n-cls.pt`, `yolo11n.pt`, `yolo11n-seg.pt`

**Thay đổi so với notebook nguồn:** Không 

> ZIP do notebook tạo có tên `KX-DAY01-report.zip`. Giải nén rồi đặt trực tiếp `REPORT.md` vào
> `day1\_lab\_outputs/` vào thư mục `report/` của repository tạo từ template. Không ghi họ tên, MSSV,
> email, số điện thoại hoặc dữ liệu cá nhân khác. Nộp link repository trên VLearn; tài khoản VLearn xác
> định người nộp.

## 1\. Phân loại ảnh – prediction cấp ảnh

Nguồn evidence: `classification\_predictions.json`, sample `traffic`.

* Record hạng 1: class\_id: 468, class\_name: cab, rank: 1, score: 0.510915, taxonomy\_name: ImageNet-1K.
* Record này mô tả các chủ thể nhận diện chính chiếm ưu thế trong toàn bộ khung ảnh.
* Định nghĩa class list mà checkpoint có thể dự đoán: do người xây dựng tập dữ liệu ImageNet và người huấn luyện mô hình định nghĩa.
* Cần giữ cả ID, tên lớp và tên taxonomy vì: máy dùng ID để định nghĩa, xử lý; Tên để người dùng hiểu, taxonomy là nguồn gốc của tập nhãn hoặc có thể xem để xá định ngữ cảnh
* Nếu ảnh có nhiều chủ thể, guideline cần quy định chọn đối tượng chính, quan trọng nhất để gán nhãn duy nhất.
* Model score không phải ground truth vì model score là kết quả đánh giá mà máy tính trả về, còn ground truth là tập nhãn do con người xác nhận là đúng và dùng làm mốc đánh giá.

## 2\. Phát hiện vật thể – lớp và box cho từng object

Nguồn evidence: `detection\_predictions.json` và `visuals/detection\_predictions.png`, sample `kitchen`.

* Một record: class\_name: person, score: 0.769676, bbox\_xyxy: \[0.08, 256.79, 18.39, 313.12], bbox\_width: 18.32, bbox\_height: 56.33.
* Diễn giải vị trí box bằng lời: box nằm ở vùng vị trí theo tọa độ x: 0.08 đến 18.39 pixel; vùng vị trí theo tọa độ y: 256.79 đến 313.12 pixel.
* So sánh số prediction ở hai threshold: Ở ngưỡng 0.35 phát hiện 11 vật thể. Khi giảm ngưỡng xuống 0.20, số lượng vật thể tăng lên (17 vật thể).
* Điều thay đổi đối với độ bao phủ và khối lượng reviewer cần xem: Ngưỡng thấp tăng độ bao phủ (ít sót) nhưng reviewer phải tốn nhiều công sức loại bỏ các hộp sai.
* Quy tắc box chặt: Hộp phải bao sát mép ngoài cùng của vật thể, không thừa quá 2 pixel và không thiếu bộ phận nào.
* Với object bị che khuất/cắt mép cần guideline quy định gán nhãn phần nhìn thấy hoặc ước lượng phần bị che.

## 3\. Phân đoạn theo từng đối tượng – polygon cho mỗi instance

Nguồn evidence: `segmentation\_predictions.json` và `visuals/segmentation\_prediction.png`, sample `kitchen`.

* Một record: instance\_id: traffic-001, class\_name: bus, score: 0.925745, số điểm: 120, polygon\_xy: \[148.0, 189.0]
* Polygon bổ sung chi tiết gì so với box? Polygon cung cấp hình dáng chính xác của biên (contour), phân biệt rõ vật thể với nền.
* `instance\_id` dùng để phân biệt các cá thể khác nhau trong cùng một lớp. Và `instance\\\_id` không phải là ID định danh toàn cầu duy nhất cho mọi ảnh.
* Đề xuất một quy tắc biên mask: Đường biên phải đi qua các điểm ảnh ranh giới của vật thể, đảm bảo độ trơn và không lấn sang vật thể khác.
* Với vùng mờ/tiếp xúc/che khuất cần escalation để quyết định ranh giới khi biên vật thể bị nhòe (motion blur) hoặc thiếu sáng.

## 4\. Vòng đời và kiểm tra chất lượng

`ảnh thô → guideline → ground truth → huấn luyện → prediction → QC/rework`

|Tác vụ|Đơn vị/định dạng ground truth|Lỗi hoặc điểm mơ hồ quan sát được|Annotator làm gì?|Reviewer xem gì?|
|-|-|-|-|-|
|Phân loại ảnh|Nhãn đơn|Ảnh có nhiều vật thể|Chọn nhãn chính|Kiểm tra tính phù hợp|
|Phát hiện vật thể|Bounding box|Box chồng lấn|Vẽ box sát mép|Kiểm tra độ chặt/ sót|
|Instance segmentation|Polygon/Mask|Biên vật thể phức tạp|Chấm điểm bao quanh|Kiểm tra độ mịn của biên|

## 5\. An toàn dữ liệu

* Một quy tắc bảo vệ dữ liệu: Không upload ảnh/ thông tin cá nhân, chia sẻ dữ liệu cho bên thứ ba, sao chép dữ liệu.
* Nếu thấy ảnh hoặc dữ liệu không đúng phạm vi, tôi sẽ dừng và báo cho: Quản lý dự án hoặc Mentor hướng dẫn

## 6\. Danh sách bằng chứng

* \[ ] `classification\_predictions.json`
* \[ ] `detection\_predictions.json`
* \[ ] `segmentation\_predictions.json`
* \[ ] `IMAGE\_ATTRIBUTION.md`
* \[ ] `visuals/classification\_top5.png`
* \[ ] `visuals/detection\_predictions.png`
* \[ ] `visuals/segmentation\_prediction.png`
* \[ ] Ô validation cuối notebook báo `PASS`.
* \[ ] Không có họ tên, MSSV hoặc dữ liệu nhạy cảm trong báo cáo/output.

