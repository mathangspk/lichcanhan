# Lịch trình hằng ngày

Ứng dụng lịch trình cá nhân bằng tiếng Việt, chạy hoàn toàn trong trình duyệt từ một tệp HTML độc lập.

## Khởi chạy

1. Tải hoặc giữ `index.html` trong cùng thư mục này.
2. Mở trực tiếp `index.html` bằng trình duyệt hiện đại (Chrome, Edge, Firefox hoặc Safari).
3. Không cần cài đặt, máy chủ, tài khoản, kết nối mạng hay thư viện bên ngoài.

Có thể dùng một máy chủ tĩnh cục bộ nếu muốn, nhưng không bắt buộc. Ví dụ với Python:

```bash
python -m http.server 8000
```

Sau đó mở địa chỉ cục bộ mà Python hiển thị.

## Phạm vi tính năng

- Chọn ngày và lưu lịch riêng cho từng ngày.
- Thêm, sửa và xóa công việc.
- Nhập tiêu đề, giờ bắt đầu/kết thúc, danh mục Công ty/Cá nhân và mức ưu tiên.
- Sắp xếp timeline theo giờ bắt đầu, sau đó giờ kết thúc và thứ tự tạo.
- Đánh dấu hoặc bỏ đánh dấu hoàn thành chỉ theo thao tác trực tiếp của người dùng; ứng dụng không tự suy đoán trạng thái.
- Cảnh báo khi khoảng thời gian trùng với việc khác nhưng vẫn cho phép lưu.
- Kiểm tra tiêu đề, ngày, định dạng giờ và yêu cầu giờ kết thúc phải sau giờ bắt đầu.
- Lưu bằng `localStorage` với khóa riêng theo ngày; nếu lưu trữ không khả dụng, ứng dụng tiếp tục hoạt động trong bộ nhớ của phiên hiện tại và hiển thị cảnh báo.
- Xử lý dữ liệu lưu bị hỏng mà không làm sập giao diện.
- Empty state rõ ràng khi ngày chưa có công việc.
- Giao diện responsive, điều hướng bàn phím, focus hiển thị rõ và nhãn form tường minh.

Ứng dụng **không** đồng bộ Google Calendar, Notion hoặc bất kỳ dịch vụ nào. Không có dữ liệu mẫu được tự động thêm; chữ “Ví dụ” trong placeholder chỉ minh họa cách nhập và không phải lịch thật.

## Lưu trữ và quyền riêng tư

Mỗi ngày dùng khóa dạng:

```text
lich-trinh-hang-ngay:v1:YYYY-MM-DD
```

Dữ liệu chỉ nằm trong `localStorage` của trình duyệt hiện tại. Xóa dữ liệu trang/trình duyệt sẽ xóa lịch. Không có yêu cầu mạng và không gửi dữ liệu ra ngoài.

## Kết quả QA — ngày 30 tháng 9 năm 2026

Phạm vi xác minh: đọc/đi theo các nhánh mã nguồn và kiểm tra thủ công tĩnh, kết hợp bộ smoke check cố định của dự án (kiểm tra cấu trúc ứng dụng và cú pháp JavaScript). Các kết quả dưới đây không được mô tả như một bộ kiểm thử trình duyệt tự động đa nền tảng.

- **PASS — Thêm:** dữ liệu hợp lệ tạo công việc mới; công việc mới luôn khởi tạo `completed: false` và không bị suy đoán là đã hoàn thành.
- **PASS — Sửa:** nút Sửa nạp đúng trường, giữ ID/thời điểm tạo/trạng thái hoàn thành hiện có và lưu các thay đổi hợp lệ.
- **PASS — Xóa và hủy xóa:** hộp thoại xác nhận native giữ nguyên dữ liệu khi chọn hủy; chỉ nhánh xác nhận mới xóa, lưu và render lại.
- **PASS — Bật/tắt hoàn thành:** chỉ thao tác trực tiếp với checkbox mới đổi trạng thái; thay đổi được lưu và checkbox thay thế nhận lại focus.
- **PASS — Persistence:** dữ liệu được tuần tự hóa vào `localStorage`; đường lui bộ nhớ phiên và cảnh báo được dùng nếu storage không khả dụng.
- **PASS — Tách dữ liệu theo ngày:** khóa `lich-trinh-hang-ngay:v1:YYYY-MM-DD` và việc tải lại khi đổi ngày giữ lịch của từng ngày độc lập.
- **PASS — Thứ tự thời gian:** timeline sắp theo giờ bắt đầu, rồi giờ kết thúc, rồi thời điểm tạo.
- **PASS — Trùng giờ và điểm biên:** giao nhau thực sự tạo cảnh báo/huy hiệu nhưng không chặn lưu; hai khoảng chỉ chạm tại giờ kết thúc/bắt đầu không bị tính là trùng; khi sửa, công việc hiện tại được loại khỏi phép so sánh với chính nó.
- **PASS — Empty state:** ngày không có dữ liệu hiển thị trạng thái rỗng; nút thêm trong trạng thái này đưa focus về trường tiêu đề.
- **PASS — Thời gian thiếu/không hợp lệ/bằng nhau/đảo ngược:** các trường thiếu hoặc sai định dạng bị từ chối; giờ kết thúc bằng hoặc sớm hơn giờ bắt đầu đều không được lưu và focus chuyển tới lỗi đầu tiên.
- **PASS — Phục hồi dữ liệu lưu hỏng:** JSON hỏng không làm sập ứng dụng, tạo lịch rỗng kèm banner rõ ràng; lần lưu hợp lệ tiếp theo thay thế giá trị hỏng của ngày đó.
- **PASS — Bàn phím và focus:** phần tử tương tác dùng control native, có thứ tự Tab tự nhiên, nhãn tường minh, focus nhìn thấy rõ và quản lý focus sau sửa/hủy/xóa/đổi trạng thái. Vùng nhãn liên kết với checkbox hoàn thành có mục tiêu nhấn tối thiểu 44×44 CSS px trong khi checkbox native vẫn hiển thị gọn.
- **PASS — Rà soát tràn ngang ở 320 px:** grid dùng `minmax(0, …)`, nội dung dài được wrap, form chuyển một cột và action wrap ở breakpoint hẹp; kiểm tra tĩnh không phát hiện nguồn tràn ngang bắt buộc.
- **PASS — Audit DOM an toàn:** nội dung động/người dùng được tạo bằng `createElement`, `textContent` và `setAttribute`; JavaScript không dùng `innerHTML`, `insertAdjacentHTML` hoặc `document.write`.
- **PASS — Không dependency/network:** toàn bộ CSS và JavaScript nằm trong `index.html`; không import thư viện, không gọi mạng và không có đồng bộ Google Calendar/Notion.
- **PASS — Smoke check cố định:** lệnh acceptance của dự án báo `PASS: schedule smoke checks and JavaScript syntax.` sau remediation.

## Giới hạn kiểm thử và sản phẩm

- Việc xác minh trên được thực hiện bằng code-path/manual inspection và smoke check cố định; chưa có ma trận trình duyệt thực chạy tự động. Hành vi screen reader, persistence qua reload thực tế và render chính xác ở 320 px vẫn nên được kiểm tra thủ công trên các trình duyệt/thiết bị mục tiêu.
- Công việc phải bắt đầu và kết thúc trong cùng một ngày; lịch qua nửa đêm chưa được hỗ trợ.
- Dữ liệu chỉ lưu cục bộ trên một trình duyệt/thiết bị; không có tài khoản, đồng bộ, nhập/xuất hoặc nhắc việc hệ thống.
- Xác nhận xóa dùng `window.confirm` native nên hình thức và trải nghiệm có thể khác giữa các trình duyệt.
- Nếu `localStorage` bị chặn hoặc hết dung lượng, thay đổi mới chỉ được giữ trong bộ nhớ cho đến khi đóng/tải lại trang.
- Khi dữ liệu của một ngày là JSON hỏng, ứng dụng không thể phục hồi nội dung đó; lần lưu mới cho ngày ấy sẽ thay thế giá trị hỏng sau khi đã cảnh báo rõ.
