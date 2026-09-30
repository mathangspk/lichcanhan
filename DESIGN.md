# Đặc tả UI/UX — Lịch trình hằng ngày

## 1. Phạm vi và nguyên tắc

- Ứng dụng tiếng Việt, chạy độc lập từ một tệp `index.html`; CSS và JavaScript nội tuyến, không thư viện, không mạng, không đồng bộ Google Calendar/Notion.
- Mục tiêu chính: chọn ngày, xem lịch theo thời gian, thêm/sửa/xóa việc, đánh dấu hoàn thành, nhận cảnh báo trùng giờ.
- Dữ liệu lưu cục bộ theo từng ngày. Không suy đoán trạng thái hoàn thành: chỉ đổi trạng thái khi người dùng kích hoạt điều khiển tương ứng.
- Nếu có dữ liệu mẫu, đặt trong khu vực có nhãn rõ **“Dữ liệu ví dụ”** hoặc gắn huy hiệu **“Ví dụ”** trên từng mục; mọi mục mẫu mặc định chưa hoàn thành. Không trình bày dữ liệu mẫu như lịch thật của người dùng.
- Không dùng ảnh, biểu tượng hoặc font từ bên ngoài. Biểu tượng nếu cần dùng ký tự đơn giản kèm nhãn chữ/`aria-label`; không dùng biểu tượng làm tín hiệu duy nhất.

## 2. Kiến trúc trang và ngữ nghĩa

Thứ tự DOM và tab phải khớp thứ tự đọc:

1. Liên kết bỏ qua: **“Bỏ qua đến lịch trình”** (chỉ hiện khi focus).
2. `<header>`:
   - `<h1>Lịch trình hằng ngày</h1>`
   - Mô tả ngắn: **“Sắp xếp công việc và thời gian cá nhân trong ngày.”**
3. `<main>` gồm:
   - `<section aria-labelledby="date-heading">`: tiêu đề **“Chọn ngày”**, `<label for="schedule-date">Ngày xem lịch</label>`, `input type="date"`.
   - Bố cục hai vùng trên desktop:
     - `<section aria-labelledby="form-heading">`: biểu mẫu thêm/sửa.
     - `<section id="schedule" aria-labelledby="timeline-heading">`: lịch trình của ngày đang chọn.
4. Một vùng thông báo dùng `role="status" aria-live="polite" aria-atomic="true"` cho kết quả lưu/xóa/đổi trạng thái; lỗi nhập liệu gắn trực tiếp với trường. Cảnh báo quan trọng có thể dùng `role="alert"` sau thao tác gửi.
5. `<footer>` ngắn: **“Dữ liệu chỉ được lưu trong trình duyệt này.”**

Chỉ có một `<h1>`. Tiêu đề section dùng `<h2>`, tiêu đề việc dùng `<h3>`. Nút phải là `<button type="button">`; nút gửi form là `<button type="submit">`.

## 3. Bố cục responsive

### Mobile: 320–767 px

- Một cột; padding trang 16 px, khoảng cách section 20–24 px.
- Thứ tự: chọn ngày → biểu mẫu → lịch trình.
- Trường tiêu đề chiếm toàn dòng. Giờ bắt đầu/kết thúc có thể thành 2 cột khi đủ chỗ (mỗi cột tối thiểu 120 px), tự xếp một cột ở màn hình rất hẹp.
- Nút chính rộng 100%; nhóm nút thẻ việc cho phép wrap, mỗi vùng bấm cao tối thiểu 44 px.
- Timeline không có chiều rộng cố định và không tạo cuộn ngang. Văn bản dài được xuống dòng (`overflow-wrap: anywhere`).

### Tablet/desktop: từ 768 px

- Container giữa trang, `max-width: 1120px`, padding 24–32 px.
- Thanh chọn ngày nằm trên hai cột.
- Lưới nội dung: form 340–380 px; timeline chiếm phần còn lại; gap 24 px; căn đầu trên. Có thể để form `position: sticky; top: 24px` nhưng phải tắt sticky khi chiều cao viewport thấp hoặc trên mobile.
- Danh sách vẫn là một cột để giữ trật tự thời gian rõ ràng.

## 4. Hệ thống thị giác

### Màu

Dùng biến CSS và bảo đảm tương phản WCAG AA:

- Nền trang: `#F4F7FB`; bề mặt: `#FFFFFF`; chữ chính: `#172033`; chữ phụ: `#526078`.
- Primary: `#2457D6`; hover: `#1946B8`; chữ trên primary: `#FFFFFF`.
- Viền: `#CBD5E1`; focus: `#0B63CE` với outline 3 px và offset 2 px.
- Công ty: chữ/viền `#1D4ED8`, nền nhạt `#EFF6FF`.
- Cá nhân: chữ/viền `#7C3AED`, nền nhạt `#F5F3FF`.
- Thành công/hoàn thành: `#18794E`; cảnh báo: chữ `#7A4B00`, nền `#FFF7D6`, viền `#D89A00`; lỗi: `#B42318`, nền `#FFF1F0`.
- Không truyền đạt category, priority, hoàn thành hoặc lỗi chỉ bằng màu; luôn có nhãn chữ/hình dạng/decoration bổ sung.

### Chữ và khoảng cách

- Font hệ thống: `system-ui, -apple-system, "Segoe UI", sans-serif`.
- Cỡ chữ thân 16 px, line-height 1.5; `h1` 28–36 px; `h2` 20–24 px; nhãn 14–16 px, weight 600.
- Thang khoảng cách: 4, 8, 12, 16, 24, 32 px. Bo góc 10–14 px; bóng nhẹ chỉ để phân lớp, không thay viền.

## 5. Thanh ngày

- Hiển thị input ngày và tiêu đề ngày đã chọn, ví dụ **“Lịch trình — Thứ Tư, 30/09/2026”**.
- Khi đổi ngày: tải đúng dữ liệu của ngày đó, đặt focus hợp lý ở input ngày (không tự nhảy focus), cập nhật timeline và thông báo nhẹ **“Đã mở lịch ngày …”**.
- Nếu trình duyệt không trả giá trị ngày hợp lệ, hiển thị **“Vui lòng chọn một ngày hợp lệ.”**

## 6. Biểu mẫu thêm/sửa

`<form novalidate>` để ứng dụng cung cấp thông báo nhất quán, nhưng vẫn dùng đúng input type:

- **Tiêu đề công việc** — `input type="text"`, required, maxlength hợp lý (đề xuất 120), placeholder **“Ví dụ: Họp nhóm dự án”**. Placeholder chỉ là ví dụ, không thay label.
- **Giờ bắt đầu** — `input type="time"`, required.
- **Giờ kết thúc** — `input type="time"`, required.
- **Danh mục** — `select` hoặc radio, required: **“Công ty”**, **“Cá nhân”**.
- **Mức ưu tiên** — `select`, required: **“Thấp”**, **“Vừa”**, **“Cao”**.
- Nút chính khi thêm: **“Thêm vào lịch”**.
- Chế độ sửa:
  - Tiêu đề section đổi thành **“Chỉnh sửa công việc”**.
  - Nút chính **“Lưu thay đổi”** và nút phụ **“Hủy chỉnh sửa”**.
  - Có dòng ngữ cảnh **“Bạn đang chỉnh sửa: {tiêu đề}”**; tiêu đề người dùng phải được chèn bằng `textContent`, không HTML.
  - Sau hủy, xóa giá trị sửa và trở về form thêm.

### Xác thực

- Lỗi xuất hiện ngay sau trường, màu lỗi + biểu tượng/ký hiệu + chữ; input có `aria-invalid="true"` và `aria-describedby` trỏ tới lỗi.
- Khi gửi lỗi, focus trường lỗi đầu tiên; không xóa dữ liệu đã nhập.
- Microcopy:
  - Tiêu đề trống: **“Vui lòng nhập tiêu đề công việc.”**
  - Tiêu đề quá dài: **“Tiêu đề không được vượt quá 120 ký tự.”**
  - Thiếu giờ: **“Vui lòng chọn giờ bắt đầu/kết thúc.”**
  - Sai thứ tự hoặc bằng nhau: **“Giờ kết thúc phải sau giờ bắt đầu.”**
  - Thiếu category/priority: **“Vui lòng chọn danh mục/mức ưu tiên.”**

## 7. Timeline và thẻ công việc

- Tiêu đề timeline gồm ngày đầy đủ và số lượng: **“3 công việc”**. Không gọi mục chưa được người dùng đánh dấu là hoàn thành.
- Danh sách dùng `<ol>` hoặc `<ul>`; sắp theo giờ bắt đầu tăng dần, sau đó giờ kết thúc, rồi thứ tự tạo để ổn định.
- Mỗi `<li>` là một article/card:
  - Cột/nhãn giờ nổi bật: **“08:30–09:15”**.
  - `<h3>` tiêu đề.
  - Huy hiệu category **“Công ty”** / **“Cá nhân”**.
  - Huy hiệu **“Ưu tiên: Cao/Vừa/Thấp”**.
  - Trạng thái chữ **“Chưa hoàn thành”** hoặc **“Đã hoàn thành”**.
  - Checkbox có label cụ thể, ví dụ **“Đánh dấu ‘Họp nhóm’ là hoàn thành”**; khi đã hoàn thành, label đổi phù hợp để có thể bỏ đánh dấu.
  - Nút **“Sửa”** và **“Xóa”**, accessible name nên chứa tiêu đề việc nếu chỉ hiển thị chữ ngắn.
- Mục hoàn thành có nền dịu và tiêu đề gạch ngang, nhưng vẫn giữ tương phản; thứ tự không tự thay đổi.
- Có đường timeline dọc trang trí bằng CSS; đặt `aria-hidden="true"` nếu là phần tử DOM. Giờ và nội dung vẫn phải hiểu được nếu bỏ đường này.

### Xóa

- Để tránh xóa nhầm, dùng hộp xác nhận nội bộ hoặc `window.confirm` với câu **“Xóa ‘{tiêu đề}’? Thao tác này không thể hoàn tác.”**
- Sau khi xóa: cập nhật danh sách và live region **“Đã xóa công việc.”** Không chèn tiêu đề người dùng dưới dạng HTML.

## 8. Trùng thời gian

- Trùng khi `startA < endB && startB < endA`; các việc chạm đầu/cuối (09:00–10:00 và 10:00–11:00) không trùng.
- Khi thêm/sửa một khoảng hợp lệ nhưng trùng việc khác, vẫn cho phép lưu vì đây là cảnh báo, không phải lỗi chặn.
- Hiển thị banner trong form trước nút gửi hoặc ngay sau nhóm giờ:
  - Tiêu đề **“Có lịch trùng giờ”**.
  - Nội dung **“Khoảng thời gian này trùng với: {danh sách tiêu đề và giờ}. Bạn vẫn có thể lưu.”**
- Sau khi lưu, các card liên quan có huy hiệu **“Trùng giờ”** và mô tả hỗ trợ như **“Trùng với 1 công việc khác”**. Không dùng màu cam đơn độc.
- Khi sửa, không so sánh mục với chính nó. Cập nhật cảnh báo khi giờ thay đổi; thông báo qua live region nhưng tránh phát liên tục trên từng phím nếu gây nhiễu (ưu tiên `change`/`blur` hoặc debounce).

## 9. Trạng thái rỗng, lỗi và lưu trữ

### Rỗng

Trong vùng timeline, card rỗng có tiêu đề **“Chưa có công việc nào”**, mô tả **“Hãy thêm công việc đầu tiên cho ngày này.”**, nút **“Thêm công việc”** đưa focus tới trường tiêu đề. Không tự tạo dữ liệu mẫu.

### Lưu thành công

- Live region: **“Đã thêm công việc.”**, **“Đã lưu thay đổi.”**, **“Đã cập nhật trạng thái.”**
- Sau khi thêm thành công: reset form, giữ nguyên ngày, focus hợp lý về tiêu đề hoặc mục mới; ưu tiên không gây cuộn bất ngờ.

### Lỗi lưu/đọc

- Nếu localStorage không khả dụng: banner lỗi **“Không thể lưu dữ liệu trong trình duyệt. Thay đổi có thể mất khi đóng trang.”** Ứng dụng vẫn dùng dữ liệu trong bộ nhớ cho phiên hiện tại nếu có thể.
- Dữ liệu JSON hỏng: không làm sập trang; hiển thị **“Không thể đọc dữ liệu đã lưu cho ngày này. Bạn có thể bắt đầu lại bằng cách thêm công việc mới.”** Không âm thầm tuyên bố dữ liệu đã được xóa.
- Nếu không chọn ngày: vô hiệu hóa hoặc chặn gửi kèm lỗi rõ ràng.

## 10. Bàn phím, focus và hỗ trợ tiếp cận

- Mọi chức năng dùng được bằng Tab/Shift+Tab, Enter và Space theo hành vi native; không tạo card click toàn bộ hoặc `div` giả nút.
- Luôn có `:focus-visible` rõ (outline 3 px, không bị clipping). Không xóa outline nếu chưa có thay thế.
- Khi bấm **“Sửa”**, focus vào trường tiêu đề và thông báo chế độ sửa. Khi **“Hủy chỉnh sửa”**, trả focus về nút Sửa đã kích hoạt nếu mục còn tồn tại.
- Sau xác nhận xóa, focus tới mục kế tiếp, mục trước, hoặc tiêu đề timeline nếu danh sách rỗng.
- Không dùng phím tắt chữ cái toàn cục. Không tự động focus khi tải trang.
- Checkbox, input, select và button cao tối thiểu 44 px trên cảm ứng; label liên kết bằng `for`/`id`.
- Chuyển động ngắn và tôn trọng `prefers-reduced-motion: reduce`.

## 11. Nội dung mẫu (nếu triển khai)

Ưu tiên empty state thay vì tự nạp mẫu. Nếu frontend cần minh họa, đặt mẫu trong vùng tách biệt có tiêu đề **“Dữ liệu ví dụ — chưa phải lịch của bạn”** và nút rõ ràng **“Thêm ví dụ vào ngày đang chọn”**. Không tự chèn vào lịch, không đánh dấu hoàn thành sẵn, và cho biết hành động sẽ lưu vào trình duyệt.

Ví dụ microcopy được phép:

- **“Ví dụ: Họp nhóm dự án, 09:00–09:30, Công ty, Ưu tiên vừa, Chưa hoàn thành.”**

## 12. Tiêu chí nghiệm thu giao diện

- Hoạt động ở 320 px mà không cuộn ngang; desktop có phân cấp hai cột rõ ràng.
- Đủ label hiển thị, heading đúng cấp, landmark và live region; tab order tự nhiên; focus luôn thấy rõ.
- Các trạng thái rỗng, lỗi validation, lỗi storage, trùng giờ, thêm, sửa và hoàn thành đều có microcopy tiếng Việt, không dựa riêng vào màu.
- Timeline đúng thứ tự thời gian; card thể hiện giờ, title, category, priority, trạng thái và đủ điều khiển.
- Dữ liệu ví dụ (nếu có) được ghi nhãn rõ và chưa hoàn thành; không suy đoán hoàn thành.
- Không dependency, network, tích hợp/sync; không dùng `innerHTML`, `insertAdjacentHTML` hoặc `document.write` trong JavaScript. Nội dung người dùng chỉ đưa vào DOM bằng `textContent`, thuộc tính an toàn và API tạo node.
