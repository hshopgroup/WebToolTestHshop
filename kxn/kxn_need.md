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

## 3. Quá trình nâng cấp & Tái cấu trúc (Monolithic + Iframe Bus)
- **Kiến trúc đa cửa sổ (Dashboard):**
  - **Vấn đề:** Trình duyệt chỉ cho phép 1 thẻ mở kết nối Web Serial. Khi muốn mở nhiều màn hình Plotter cùng lúc, trình duyệt sẽ báo lỗi cổng COM đang bận.
  - **Giải pháp:** Giữ nguyên 1 file `Web_SerialPloter_Anti.html` duy nhất. Dùng tham số trên URL (ví dụ: `?hide=plotter`) để ẩn các thành phần không mong muốn. Mở nhiều Iframe trên trang `Dashboard.html` và sử dụng `window.postMessage` làm hệ thống "Iframe Bus" để chia sẻ dữ liệu Serial từ thẻ Master sang các thẻ Slave.
- **Tối ưu Bảng ghi chú (Legend):**
  - Chuyển tính năng bật/tắt hiển thị từng kênh vào Modal "Thiết lập kênh" để mở rộng tối đa không gian vẽ biểu đồ.
- **Nâng cấp Terminal (Phong cách IDE chuyên nghiệp):**
  - **Mốc thời gian (Timestamp):** Thêm ô Checkbox `[x] Time` để gắn nhãn thời gian thực `[HH:MM:SS.ms]` vào từng dòng log.
  - **Cửa sổ kép (Dual View):** Hiển thị song song 2 cửa sổ chữ (ASCII) và mã (HEX) của dữ liệu gửi/nhận. 
  - **Bộ đếm (Counters):** Thêm thanh thống kê tổng số byte TX và RX cùng nút Reset ở tiêu đề Terminal.
- **Trải nghiệm kéo thả (Resizer / Splitter):**
  - **Splitter Dọc:** Nằm giữa bảng điều khiển bên trái (Terminal & Script) và Plotter, hoặc giữa các Iframe trong Dashboard.
  - **Splitter Ngang:** Nằm giữa Terminal và Script List. Khung chứa Script List luôn được cố định nằm dưới Terminal.

## 4. Các lỗi (Bugs) đã phát hiện và xử lý
- **Lỗi không kéo lên được thanh Splitter ngang:**
  - *Nguyên nhân:* Thẻ `<textarea>` của Script List giới hạn chiều cao tối thiểu, làm cho việc kéo thanh Splitter xuống sâu bị đẩy tràn ra ngoài khung, gây kẹt logic tính toán.
  - *Khắc phục:* Bổ sung CSS `flex: 1; min-height: 0;` cho khung Script List và thêm logic ép khung (Clamping) bằng biến giới hạn (min/max) vào sự kiện `mousemove`.
- **Lỗi Terminal không lấp đầy chiều rộng khi chạy độc lập trong Dashboard:**
  - *Nguyên nhân:* Kích thước của Terminal bị khóa cứng ở mức `width: 450px`.
  - *Khắc phục:* Giảm `min-width` xuống `250px`. Thêm logic Javascript tự động chỉnh `width: 100%` cho khung `left-panel` khi tham số URL yêu cầu ẩn toàn bộ Plotter.
- **Tránh nhầm lẫn thao tác Connect trên Dashboard:**
  - *Khắc phục:* Ẩn toàn bộ nút Connect & chọn Baud Rate (bằng tham số `hide=controls`) ở các màn hình Plotter con. Dashboard chỉ duy trì 1 nút Connect duy nhất tại Terminal.
- **Lỗi phải kết nối lại thủ công khi đổi Baud Rate:**
  - *Khắc phục:* Bắt sự kiện thay đổi (`change`) ở menu Baud, tự động ngắt kết nối và gọi lại kết nối mới với cờ `reusePort = true` để giữ nguyên Port đã xin phép từ trình duyệt mà không cần hiện Popup lại.

## 5. Các tính năng mở rộng đã hoàn thành (Mới cập nhật)
- **Bảo vệ chống treo trình duyệt (Anti-freeze):**
  - Giới hạn DOM hiển thị tại 1000 phần tử cho Plotter và Terminal.
  - Tự động lưu (Auto-save) 50,000 dòng log vào file Excel và giải phóng RAM để hệ thống có thể chạy giám sát 24/7 không giật lag.
- **Xuất/Nhập dữ liệu chuẩn Excel (XLSX):**
  - Tích hợp thư viện `SheetJS`.
  - Thay thế toàn bộ tính năng export mặc định sang định dạng `xlsx` (bao gồm Terminal, Plotter và Script List).
- **Quản lý kịch bản đa Sheet (Multi-Sheet Script):**
  - Chuyển Script List thành một phần mềm quản lý kịch bản thu nhỏ.
  - Cho phép tạo, xóa, chuyển đổi các Sheet lệnh trực tiếp trên Web.
  - Hỗ trợ Import/Export file Excel chứa nhiều Sheet với chuẩn 3 cột (Command - Delay - Description).
- **Tuỳ biến Layout linh hoạt:**
  - Thêm hệ thống Checkbox bật/tắt hiển thị từng module (Terminal, Script, Plotter, Guide).
  - Tự động thay đổi kích thước và đẩy Full-width cho Terminal khi Plotter bị ẩn.
  - Tích hợp bộ điều khiển Checkbox tổng lên thanh điều hướng của `Dashboard.html`.