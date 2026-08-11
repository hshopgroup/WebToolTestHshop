# Yêu cầu phát triển Web Serial Plotter & Terminal

## 1. Mục tiêu chung
- Xây dựng một ứng dụng web tĩnh (chạy trên trình duyệt, lưu trữ trên GitHub Pages) để giao tiếp Serial với vi điều khiển (Arduino, v.v.).
- Khắc phục giới hạn hiển thị tối đa 8 giá trị của Arduino IDE Serial Plotter (hỗ trợ vẽ vô hạn kênh analog).

## 2. Các tính năng cốt lõi yêu cầu
- **Giao tiếp Serial & Biểu đồ Realtime:**
  - Kết nối và ngắt kết nối cổng COM linh hoạt (chống lỗi khóa luồng/mở trùng cổng).
  - Tùy chọn tốc độ truyền **BaudRate** (9600, 115200, v.v.).
  - Vẽ biểu đồ thời gian thực các giá trị nhận được từ thiết bị (các kênh phân tách bằng dấu tab `\t`, xuống dòng `\n`).
- **Khung Terminal / Log (RX & TX):**
  - Hiển thị trực tiếp dữ liệu Arduino gửi lên (RX) lên khung log màu xanh dương.
  - Hiển thị các lệnh gửi đi (TX) từ máy tính xuống thiết bị.
- **Gửi dữ liệu thủ công (Manual Send):**
  - Gõ lệnh/text và gửi xuống phần cứng (hỗ trợ nhấn phím `Enter`).
  - **Hỗ trợ mã HEX:** Cho phép nhập tiền tố `$` để truyền mã HEX nguyên thủy (ví dụ: `$1A` đổi thành byte `0x1A`).
- **Gửi kịch bản theo danh sách (Script List):**
  - Nhập danh sách nhiều lệnh theo từng dòng.
  - Cấu hình thời gian chờ (`delay_ms`) ở cuối mỗi dòng lệnh trước khi thực hiện lệnh tiếp theo.
  - Hỗ trợ dòng chú thích (bắt đầu bằng `#`).
- **Tài liệu & Hướng dẫn:**
  - Tích hợp sẵn phần Hướng dẫn sử dụng trực quan ngay trên giao diện web.