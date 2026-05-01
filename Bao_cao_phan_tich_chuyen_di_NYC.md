# Báo cáo Tổng quan: Phân tích Thời gian Chuyến đi Taxi tại NYC

Tài liệu này cung cấp bản tóm tắt từng bước về quy trình xử lý dữ liệu của chúng tôi bằng ngôn ngữ dễ hiểu dành cho doanh nghiệp. Mục tiêu của phân tích này là làm sạch và làm phong phú dữ liệu thô về các chuyến đi taxi để các thuật toán Học máy (Machine Learning) của chúng ta có thể dự đoán chính xác thời gian chuyến đi.

## Bước 1: Khởi tạo & Tải dữ liệu
**Hoạt động:** Thiết lập môi trường làm việc bằng cách tải các công cụ xử lý dữ liệu và bản đồ cần thiết, sau đó nhập dữ liệu thô về các chuyến đi trong quá khứ.
**Nguyên tắc:** Xây dựng một nền tảng vững chắc để có thể xử lý hàng triệu bản ghi dữ liệu một cách hiệu quả nhất.

## Bước 2: Đánh giá dữ liệu ban đầu
**Hoạt động:** Quét nhanh toàn bộ tập dữ liệu để hiểu cấu trúc và kiểm tra xem có thông tin nào bị thiếu không. Dữ liệu không có giá trị trống, nhưng chúng tôi phát hiện ra một số điểm bất thường (ví dụ: có chuyến đi chỉ kéo dài 1 giây hoặc tọa độ GPS nằm tít ngoài đại dương).
**Nguyên tắc:** "Đầu vào chất lượng thấp thì đầu ra cũng vậy" (Garbage in, garbage out). Trước khi xây dựng các mô hình dự đoán, chúng ta phải hiểu rõ chất lượng dữ liệu hiện có để biết cần phải sửa chữa những gì.

## Bước 3: Phân tích ngoại lai (Outlier Analysis)
**Hoạt động:** Phân tích sâu vào các giá trị cực đoan, xem xét 1% các chuyến đi ngắn nhất và 1% dài nhất, cũng như số lượng hành khách. Chúng tôi xác định được các ngưỡng mà tại đó dữ liệu không còn ý nghĩa thực tế.
**Nguyên tắc:** Loại bỏ các trường hợp cá biệt. Một chuyến đi taxi bình thường không thể dưới 1.5 phút hoặc quá 1.5 giờ. Bằng cách thiết lập các ranh giới này, chúng ta tách biệt được các "nhiễu" (như hỏng đồng hồ đo hoặc lỗi GPS) khỏi các chuyến đi thực tế chuẩn mực.

## Bước 4: Làm sạch dữ liệu
**Hoạt động:** Sàng lọc dữ liệu một cách triệt để. Chúng tôi loại bỏ các chuyến đi có thời gian bất hợp lý hoặc có 0 hành khách. Quan trọng hơn, chúng tôi thiết lập một khung tọa độ địa lý (Bounding Box) bao quanh thành phố New York và đối chiếu với bản đồ thực tế (GeoJSON) để thẳng tay loại bỏ mọi điểm đón/trả khách nằm dưới mặt nước.
**Nguyên tắc:** Đảm bảo tính thực tế. Mô hình dự đoán sẽ bị nhiễu nếu nó cố gắng học từ một chiếc taxi... chạy trên mặt biển. Dữ liệu thực tế và chính xác sẽ mang lại các dự đoán có độ tin cậy cao.

## Bước 5: Trích xuất đặc trưng (Tạo thêm dữ liệu thông minh)
**Hoạt động:** Dựa vào dữ liệu thô, chúng tôi tạo ra các biến số mới cung cấp nhiều thông tin hơn (features) để giúp thuật toán học được đặc thù giao thông ở NYC.
*   **Tính toán khoảng cách:** Thay vì tính đường chim bay, chúng tôi tính "Khoảng cách Manhattan" (dạng lưới kẻ ô vuông), vì ô tô phải chạy trên đường phố chứ không thể đâm xuyên qua các tòa nhà.
*   **Phương hướng di chuyển:** Tính toán góc độ định hướng của chuyến đi.
*   **Tốc độ ước tính:** Ước lượng tốc độ giao thông trung bình theo từng khung giờ trong ngày dựa trên dữ liệu lịch sử.
*   **Xoay tọa độ (Tuyệt chiêu):** Chúng tôi xoay các tọa độ trên bản đồ một góc 29 độ bằng công thức toán học.
*   **Tại sao cần xoay?** Lưới đường phố của New York City bị nghiêng tự nhiên khoảng 29 độ. Việc xoay tọa độ giúp các con đường được sắp xếp lại để thẳng đứng và nằm ngang hoàn hảo. Điều này giúp thuật toán Học máy (XGBoost) cực kỳ dễ dàng thấu hiểu cấu trúc thành phố, tăng cường độ chính xác và tránh bị nhiễu bởi các chuyển động chéo phức tạp.

## Bước 6: Áp dụng biến đổi
**Hoạt động:** Áp dụng toàn bộ các quy tắc làm sạch và các hàm tạo đặc trưng ở trên cho cả tập dữ liệu huấn luyện (train) và dữ liệu kiểm thử (test), sau đó lưu lại phiên bản dữ liệu tinh khiết nhất.
**Nguyên tắc:** Tính nhất quán là cốt lõi. Mô hình phải được huấn luyện và kiểm thử trên các tập dữ liệu có cùng định dạng chuẩn hóa, để đảm bảo các dự đoán sẽ luôn ổn định khi được triển khai vào thực tế.

## Bước 7: Đánh giá & Trực quan hóa
**Hoạt động:** Kiểm tra lại tập dữ liệu đã làm sạch và vẽ biểu đồ thể hiện thời gian chuyến đi. Trước khi vẽ, chúng tôi dùng một thủ thuật toán học gọi là "Chuyển đổi Logarit" (Log Transformation) lên các giá trị thời gian.
*   **Tại sao dùng chuyển đổi Log (Tuyệt chiêu):** Phần lớn các chuyến đi taxi thường khá ngắn, nhưng thỉnh thoảng có vài chuyến đi cực dài (ví dụ: do kẹt xe kinh hoàng hoặc đi ra ngoại ô). Điều này khiến biểu đồ thông thường bị kéo lệch thành một cái đuôi rất dài về bên phải. Chuyển đổi Log giúp "nén" các con số khổng lồ này lại và kéo giãn các khoảng cách nhỏ ra.
*   **Ý nghĩa:** Nhờ có thủ thuật này, một biểu đồ lệch, lộn xộn đã được nắn lại thành hình dáng đối xứng tuyệt đẹp: "Hình chuông" (Phân phối chuẩn). Các mô hình Học máy hoạt động hiệu quả nhất và học nhanh hơn hẳn khi biến số mục tiêu có hình chuông, qua đó giảm thiểu đáng kể sai số trong các dự đoán thời gian chuyến đi sau cùng.
