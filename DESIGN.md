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

## 13. Bổ sung thiết kế tương tác (01/10/2026) — Sao chép lịch sang ngày khác

Phần bổ sung này xác định đặc tả thiết kế tương tác chi tiết cho tính năng **“Sao chép lịch sang ngày khác”**, hoàn toàn kế thừa hệ thống thị giác, quy tắc bố cục responsive, chuẩn tiếp cận và các ràng buộc DOM an toàn hiện có của ứng dụng (được nghiệm thu ngày 30/09/2026). Không làm thay đổi hay phá vỡ các chức năng đã có.

### 13.1. Vị trí, hình thức và trạng thái của nút kích hoạt (Copy Trigger)

1. **Vị trí hiển thị:**
   - Đặt trong khung chọn ngày (`section.date-panel`), nằm cạnh khối điều khiển chọn ngày (`div.date-control`) hoặc được gom vào cụm hành động ngày `div.date-actions`.
   - Trên desktop/tablet: Nằm ngang hàng hoặc liền kề với trường chọn ngày xem lịch, căn chỉnh thẳng hàng để người dùng thấy rõ đây là hành động thao tác trên ngày đang xem.
   - Trên mobile (320–767 px): Xếp chồng tự nhiên theo chiều dọc, chiếm toàn bộ chiều rộng (hoặc co giãn linh hoạt theo flex-wrap), đảm bảo không gây tràn chiều ngang.

2. **Cấu trúc ngữ nghĩa & Thuộc tính:**
   - Phần tử: `<button type="button" class="btn btn-secondary" id="copy-schedule-trigger">`
   - Nhãn nút: **“Sao chép lịch sang ngày khác”**
   - Kích thước tương tác: Chiều cao tối thiểu 44 px (`min-height: 44px; padding: 9px 15px;`), phông chữ 750, viền `#8b99aa`, nền trắng, hover `#eef2f7`, `:focus-visible` viền outline 3 px màu `#0b63ce`.
   - Accessible name: `aria-haspopup="dialog"`, `aria-controls="copy-schedule-dialog"`.

3. **Trạng thái của nút kích hoạt:**
   - **Trạng thái khả dụng (Enabled):** Khi ngày nguồn đang xem có ít nhất 1 công việc hợp lệ (`tasks.length > 0`) và không có lỗi đọc dữ liệu hỏng. Nút ở trạng thái bình thường, sẵn sàng bấm mở hộp thoại.
   - **Trạng thái vô hiệu hóa khi ngày nguồn rỗng (Empty Source):** Khi ngày nguồn không có công việc nào (`tasks.length === 0`), nút có thuộc tính `disabled`, giảm độ mờ (`opacity: 0.55`), `cursor: not-allowed`, kèm tooltip/nhãn giải thích: `title="Ngày hiện tại chưa có công việc để sao chép"`.
   - **Trạng thái khi ngày nguồn có dữ liệu hỏng (Corrupt Source):** Nút bị vô hiệu hóa (`disabled`) kèm `title="Dữ liệu ngày nguồn bị hỏng, không thể sao chép"`.

### 13.2. Hộp thoại native `<dialog>` và khả năng tiếp cận bàn phím

1. **Cấu trúc ngữ nghĩa HTML:**
   - Sử dụng thẻ HTML5 chuẩn: `<dialog id="copy-schedule-dialog" class="panel copy-dialog" aria-labelledby="copy-dialog-title" aria-describedby="copy-dialog-desc">`.
   - Khởi tạo mở bằng phương thức native `dialog.showModal()` để tự động kích hoạt cơ chế focus trap của trình duyệt, ngăn tương tác với nội dung nền phía sau.
   - Lớp phủ nền (`::backdrop`):
     ```css
     dialog.copy-dialog::backdrop {
       background: rgba(23, 32, 51, 0.45);
       backdrop-filter: blur(2px);
     }
     ```
   - Định kiểu hộp thoại:
     - Căn giữa màn hình, viền `1px solid var(--border)`, bo góc `var(--radius)` (14 px), nền `var(--surface)` (`#ffffff`), đổ bóng `var(--shadow)`.
     - Kích thước responsive: `width: min(100% - 32px, 520px); max-height: calc(100vh - 48px); overflow-y: auto; padding: 24px;`.
     - Ở màn hình hẹp (<= 390 px): padding 18 px, toàn bộ nút dàn 100% chiều rộng.

2. **Cấu trúc nội dung bên trong `<dialog>`:**
   - **Tiêu đề hộp thoại:** `<h2 id="copy-dialog-title">Sao chép lịch sang ngày khác</h2>` (font-size 1.35rem, font-weight 700).
   - **Mô tả ngữ cảnh:** `<p id="copy-dialog-desc" class="dialog-intro">Sao chép toàn bộ công việc từ ngày đang xem sang một ngày đích mới.</p>`
   - **Thông tin ngày nguồn:** Hộp thông tin nhẹ hiển thị ngày nguồn và số lượng công việc:
     - Vi văn bản: **“Ngày nguồn: {Thứ, DD/MM/YYYY} ({N} công việc)”** (ví dụ: *“Ngày nguồn: Thứ Tư, 30/09/2026 (3 công việc)”*).
   - **Biểu mẫu sao chép:** `<form id="copy-schedule-form" method="dialog" novalidate>`:
     - Trường chọn ngày đích:
       - `<label for="copy-destination-date">Chọn ngày đích</label>` (rõ ràng `for`/`id`).
       - `<input id="copy-destination-date" name="destinationDate" type="date" required aria-describedby="copy-dest-hint copy-dest-error">`.
       - `<span class="hint" id="copy-dest-hint">Chọn ngày khác với ngày nguồn ({YYYY-MM-DD}).</span>`.
       - `<span class="field-error" id="copy-dest-error"></span>`.
     - **Khu vực xem trước (Live Preview Panel):** Phân vùng `<section id="copy-preview-section" aria-labelledby="copy-preview-heading" aria-live="polite">` (chi tiết ở mục 13.3).
     - **Cụm nút hành động (Dialog Actions):**
       - `<div class="actions copy-actions">`
       - Nút Hủy: `<button type="button" class="btn btn-secondary" id="copy-cancel-button">Hủy</button>`
       - Nút Xác nhận: `<button type="submit" class="btn btn-primary" id="copy-submit-button">Xác nhận sao chép</button>`

3. **Điều hướng bàn phím và quản lý Focus:**
   - Khi mở hộp thoại: Chuyển focus ngay lập tức tới trường chọn ngày đích (`#copy-destination-date`).
   - Phím `Tab` / `Shift+Tab`: Chu trình di chuyển focus chỉ xoay vòng bên trong hộp thoại (Destination Date Input -> Cancel Button -> Submit Button).
   - Phím `Escape`: Kích hoạt sự kiện `cancel` native của `<dialog>`, đóng hộp thoại và hoàn trả focus ngay lập tức về nút kích hoạt `#copy-schedule-trigger`.
   - Nút **“Hủy”**: Khi nhấn, đóng hộp thoại (`dialog.close()`), dọn dẹp trạng thái xem trước và hoàn trả focus về `#copy-schedule-trigger`.
   - Không gây cuộn trang ngoài ý muốn khi mở hoặc đóng hộp thoại.

### 13.3. Khu vực xem trước kết quả sao chép (Live Preview)

Khu vực xem trước xuất hiện động ngay bên dưới trường ngày đích khi người dùng đã chọn một ngày đích hợp lệ:

1. **Bố cục các số liệu đếm (Preview Counts):**
   Gồm 3 thông số định lượng trực quan, rõ ràng, không mập mờ:
   - **Sẽ thêm mới:** Thẻ thống kê với nhãn **“Sẽ thêm mới: {X} công việc”** (màu xanh thành công `#146c45`, nền `#edf9f2`). Đây là số lượng công việc từ ngày nguồn sẽ được tạo mới tại ngày đích.
   - **Bỏ qua (trùng lặp):** Thẻ thống kê với nhãn **“Bỏ qua (trùng lặp): {Y} công việc”** (màu trung tính `#526078`, nền `#f1f5f9`). Đây là các công việc trùng cả 4 thông tin (tiêu đề, giờ bắt đầu, giờ kết thúc, danh mục) đã có sẵn ở ngày đích hoặc trùng lặp nội bộ trong ngày nguồn.
   - **Trùng khoảng giờ với lịch hiện có:** Thẻ thống kê với nhãn **“Trùng giờ ở ngày đích: {Z} công việc”** (màu cảnh báo `#714500`, nền `#fff7d6`). Đây là các công việc trong số {X} mục mới có khoảng thời gian giao nhau nghiêm ngặt (`startA < endB && startB < endA`) với lịch đã có ở ngày đích. Tiếp xúc tại điểm biên (`endA === startB`) **không** tính là trùng giờ.

2. **Cảnh báo giải thích chi tiết:**
   - Nếu `Z > 0`: Hiển thị banner cảnh báo:
     - **“Chú ý trùng khoảng giờ: Có {Z} công việc trùng giờ với lịch đã có tại ngày đích. Các công việc này vẫn sẽ được sao chép và gắn huy hiệu ‘Trùng giờ’.”**
   - Nếu `Y > 0`: Hiển thị ghi chú:
     - **“{Y} công việc có cùng tiêu đề, thời gian và danh mục đã tồn tại ở ngày đích nên sẽ không được thêm lần thứ hai.”**

### 13.4. Các trạng thái giao diện chi tiết (UI States) & Vi văn bản (Microcopy)

| Tình huống giao diện | Trạng thái điều khiển & Trình bày thị giác | Vi văn bản hiển thị (Microcopy) | Hành vi nút Xác nhận & Lưu trữ |
| :--- | :--- | :--- | :--- |
| **1. Chưa chọn ngày đích** | Trường ngày trống, vùng xem trước ẩn hoặc hiển thị chỉ dẫn. | Hướng dẫn: *“Vui lòng chọn ngày đích để xem trước kết quả.”* | Nút **“Xác nhận sao chép”** bị vô hiệu hóa (`disabled`). Không có ghi dữ liệu. |
| **2. Ngày đích trùng ngày nguồn** (Same-day error) | Trường ngày có `aria-invalid="true"`, viền đỏ nguy hiểm (`--danger`), thông báo lỗi hiển thị ngay dưới trường. Vùng xem trước ẩn. | Lỗi: *“⚠ Ngày đích phải khác ngày nguồn ({YYYY-MM-DD}).”* | Nút **“Xác nhận sao chép”** bị vô hiệu hóa (`disabled`). Chặn hoàn toàn thao tác gửi. |
| **3. Ngày đích không hợp lệ** (Invalid format) | Trường ngày có `aria-invalid="true"`. | Lỗi: *“⚠ Vui lòng chọn một ngày hợp lệ.”* | Nút **“Xác nhận sao chép”** bị vô hiệu hóa (`disabled`). |
| **4. Ngày đích có dữ liệu hỏng** (Corrupt destination) | Banner lỗi màu đỏ (`notice error`, `role="alert"`). Vùng đếm xem trước bị ẩn hoặc khóa. | Lỗi: *“⚠ Dữ liệu ngày đích trong bộ nhớ trình duyệt bị lỗi cấu trúc (JSON hỏng hoặc bản ghi không hợp lệ). Thao tác sao chép bị chặn để bảo vệ dữ liệu.”* | Nút **“Xác nhận sao chép”** bị vô hiệu hóa (`disabled`). **Quy tắc Zero-write:** Tuyệt đối không ghi đè hay thay đổi bất kỳ byte nào vào `localStorage`. |
| **5. Toàn bộ là công việc trùng lặp** (All-duplicates / Zero-write) | Đếm xem trước: Thêm mới: **0**, Bỏ qua: **{Y}**. Banner thông tin cảnh báo nhẹ màu vàng cam. | Thông báo: *“Tất cả công việc từ ngày nguồn đều đã có ở ngày đích. Sẽ không có công việc nào được thêm mới và không ghi vào bộ nhớ.”* | Nút **“Xác nhận sao chép”** bị vô hiệu hóa (`disabled`) hoặc khi nhấn sẽ đóng hộp thoại mà không thực hiện thao tác `localStorage.setItem` nào (**Zero-write guarantee**). |
| **6. Ngày đích trống hoặc có công việc không trùng** (Hợp lệ hoàn toàn) | Đếm xem trước: Thêm mới: **{X}**, Bỏ qua: **{Y}**, Trùng giờ: **0**. Banner tích cực hoặc thẻ đếm rõ ràng. | Ghi chú: *“Sẵn sàng sao chép {X} công việc sang ngày {ngày đích}.”* | Nút **“Xác nhận sao chép”** kích hoạt (`enabled`). |
| **7. Có công việc trùng khoảng giờ tại đích** (Strict overlap non-blocking) | Thẻ đếm Trùng giờ: **{Z}**. Banner cảnh báo màu vàng cam (`notice warning`, `role="status"`). | Cảnh báo: *“Có {Z} công việc trùng giờ với lịch đã có tại ngày đích. Các công việc này vẫn sẽ được sao chép và gắn huy hiệu cảnh báo.”* | Nút **“Xác nhận sao chép”** vẫn **kích hoạt** (`enabled`). Đây là cảnh báo tương tác, không phải lỗi chặn. |
| **8. Sao chép thành công** | Hộp thoại tự động đóng. **Giữ nguyên ngày nguồn đang xem** trên giao diện chính. Vùng live region phát thông báo thành công. Focus trả về nút kích hoạt. | Live region (`#app-status`): *“Đã sao chép thành công {X} công việc sang ngày {ngày đích}. Ngày xem lịch vẫn là {ngày nguồn}.”* (Nếu có Z > 0 trùng giờ: *“Đã sao chép thành công {X} công việc sang ngày {ngày đích} ({Z} công việc trùng giờ). Ngày xem lịch vẫn là {ngày nguồn}.”*). | Dữ liệu ngày đích được lưu an toàn. Các công việc đích cũ được giữ nguyên 100%. Các công việc mới nhận ID mới, `createdAt` mới, và luôn có `completed: false`. |
| **9. Dữ liệu ngày đích thay đổi trước khi xác nhận** (Stale preview) | Khi người dùng nhấn Xác nhận, ứng dụng đọc lại dữ liệu ngày đích và phát hiện thay đổi hoặc bị hỏng ngoài dự kiến. | Cảnh báo: *“Dữ liệu ngày đích đã thay đổi hoặc không hợp lệ. Vui lòng xem lại kết quả tính toán mới trước khi xác nhận.”* | Hủy lệnh ghi, tính toán lại preview tại chỗ, yêu cầu người dùng xác nhận lại với dữ liệu mới. |

### 13.5. Quy tắc dữ liệu và tính toán dưới góc độ trải nghiệm người dùng

1. **Định danh trùng lặp 4 trường (Duplicate Identity):**
   - Người dùng xem một công việc là "trùng lặp" khi khớp chính xác cả 4 thuộc tính: **Tiêu đề** (`title`), **Giờ bắt đầu** (`start`), **Giờ kết thúc** (`end`), và **Danh mục** (`category`).
   - Mức ưu tiên (`priority`) và trạng thái hoàn thành (`completed`) **không** nằm trong định danh trùng lặp.
   - Các công việc trùng lặp trong chính ngày nguồn (nội bộ nguồn lặp lại) cũng được gom lại để chỉ thêm 1 bản ghi duy nhất sang ngày đích.

2. **Bảo tồn tính toàn vẹn dữ liệu ngày đích:**
   - Tất cả các công việc hợp lệ đang có ở ngày đích phải được giữ nguyên vẹn 100% (không xóa, không ghi đè, không thay đổi ID/thời gian của chúng).
   - Công việc mới sao chép sang: Giữ nguyên `title`, `start`, `end`, `category`, `priority`; được cấp `id` duy nhất mới (UUID / chuỗi ngẫu nhiên); thời điểm tạo `createdAt` mới; và trạng thái hoàn thành **luôn khởi tạo là `false`** (không bao giờ suy đoán hoàn thành).

3. **Cảnh báo trùng giờ nghiêm ngặt (Strict Overlap vs Boundary Touch):**
   - Hai khoảng giờ [A_start, A_end] và [B_start, B_end] chỉ trùng nhau khi `A_start < B_end && B_start < A_end`.
   - Trường hợp tiếp xúc biên: `A_end === B_start` hoặc `B_end === A_start` (ví dụ 09:00–10:00 và 10:00–11:00) được coi là liền kề hợp lệ, **không** tính là trùng giờ và không tăng số đếm {Z}.

### 13.6. Tiêu chí tiếp cận, an toàn DOM và kích thước tương tác

1. **Ràng buộc an toàn DOM (Strict Safe DOM):**
   - Mọi thông tin động trong hộp thoại (tên ngày, số lượng công việc, cảnh báo trùng giờ, danh sách tiêu đề) tuyệt đối phải được tạo bằng các node DOM an toàn: `document.createElement`, gán thuộc tính `setAttribute`, và gán văn bản bằng `node.textContent`.
   - Tuyệt đối **không** dùng `innerHTML`, `insertAdjacentHTML`, `outerHTML` hoặc `document.write` ở bất kỳ đoạn mã JavaScript nào.

2. **Kích thước mục tiêu tối thiểu (44×44 CSS px):**
   - Nút kích hoạt `#copy-schedule-trigger`: chiều cao tối thiểu 44 px.
   - Trường chọn ngày `#copy-destination-date`: chiều cao tối thiểu 44 px.
   - Các nút trong hộp thoại (`#copy-cancel-button`, `#copy-submit-button`): chiều cao tối thiểu 44 px, đệm tay rộng rãi.

3. **Tương phản màu & Hiển thị Focus:**
   - Tất cả văn bản đạt tỷ lệ tương phản tối thiểu WCAG AA (tối thiểu 4.5:1 với văn bản thường, 3:1 với văn bản lớn và thành phần điều khiển).
   - Mọi nút bấm và trường nhập liệu đều có `:focus-visible` với viền outline 3 px màu `#0b63ce` cách 2 px (`outline-offset: 2 px`).

4. **Hành vi Responsive (320 px đến Desktop):**
   - Ở màn hình hẹp (320 px): `<dialog>` có lề hai bên an toàn 16 px (`width: calc(100% - 32px)`), các nút hành động xếp chồng theo chiều dọc (`flex-direction: column; width: 100%`), không xuất hiện thanh cuộn ngang trang hay hộp thoại.
