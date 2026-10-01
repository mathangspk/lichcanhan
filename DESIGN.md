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

## 14. Kiến trúc dữ liệu và giải thuật (01/10/2026) — Sao chép lịch sang ngày khác

Đặc tả kiến trúc dữ liệu và giải thuật cho tính năng sao chép lịch được chuẩn hóa chi tiết tại tệp [`COPY_ARCHITECTURE.md`](file:///COPY_ARCHITECTURE.md). Dưới đây là các định nghĩa và ràng buộc cốt lõi dành cho các vai trò triển khai (Frontend) và kiểm thử (QA/Reviewer):

### 14.1. Định danh trùng lặp chính xác 4 trường (4-Field Exact Duplicate Identity)
- Hai công việc bị coi là trùng lặp khi và chỉ khi khớp chính xác cả 4 trường:
  `task.title.trim()` + `task.start` + `task.end` + `task.category`.
- **Ràng buộc:** Trường `priority` và `completed` **không** nằm trong định danh trùng lặp. Khóa định danh tổng hợp: `JSON.stringify([task.title.trim(), task.start, task.end, task.category])`.

### 14.2. Khử trùng tất định nội bộ nguồn & Bảo toàn ngày đích
- **Khử trùng nội bộ nguồn:** Duyệt danh sách ngày nguồn theo thứ tự thời gian (`sortedTasks()`). Bản ghi đầu tiên của mỗi khóa định danh được chọn làm ứng viên sao chép; các bản ghi trùng lặp phía sau trong ngày nguồn bị bỏ qua và tính vào số lượng **Bỏ qua (skipped)**.
- **Bảo toàn ngày đích:** Toàn bộ công việc hợp lệ hiện có ở ngày đích được giữ nguyên 100%. Các ứng viên nguồn có khóa trùng với ngày đích sẽ bị bỏ qua.
- **Bất biến số lượng:** $\text{added} + \text{skipped} = |\text{sourceTasks}|$.

### 14.3. Hợp đồng chuyển đổi bản ghi
- Các trường giữ nguyên 100%: `title`, `start`, `end`, `category`, `priority`.
- Các trường sinh mới hoàn toàn:
  - `id`: Sinh ID duy nhất mới (`makeId()`, UUID v4 hoặc fallback chuỗi ngẫu nhiên). Không dùng lại ID của ngày nguồn.
  - `createdAt`: Sinh thời điểm tạo mới (`Date.now()`).
  - `completed`: **Luôn khởi tạo bằng `false`** (tuyệt đối không suy đoán hoàn thành).

### 14.4. Đọc dữ liệu ngày đích: Phân loại 4 trạng thái nghiêm ngặt
Hàm `readDestinationTasks(destDate)` thanh tra toàn diện và trả về một trong 4 trạng thái:
1. `missing`: Khóa chưa tồn tại (`raw === null`). Coi là mảng rỗng hợp lệ (`tasks = []`).
2. `valid`: Chuỗi JSON parse thành mảng và **100% bản ghi** đều thỏa mãn 10 điều kiện kiểm tra nghiêm ngặt của `isValidStoredTask(record)`.
3. `corrupt_json`: Chuỗi JSON bị lỗi cú pháp ngoại lệ hoặc cấu trúc gốc không phải mảng.
4. `invalid_records`: Là mảng nhưng có ít nhất 1 bản ghi thiếu trường, sai kiểu hoặc sai logic thời gian.

### 14.5. Nguyên tắc Zero-Write khi có lỗi hoặc không có bản ghi mới
- Khi ngày đích rơi vào `corrupt_json` hoặc `invalid_records`:
  - Khóa vùng xem trước và vô hiệu hóa nút Xác nhận (`disabled = true`).
  - Hiển thị banner cảnh báo lỗi cấu trúc dữ liệu.
  - **Zero-Write Guarantee:** Tuyệt đối không gọi `localStorage.setItem` hoặc `localStorage.removeItem`.
- Khi toàn bộ công việc nguồn đều trùng lặp ($\text{added} = 0$):
  - Hiển thị thông báo giải thích và vô hiệu hóa xác nhận / không ghi. Tuyệt đối không gọi `localStorage.setItem`.

### 14.6. Chống bất đồng bộ dữ liệu (Stale Preview Mitigation)
- Khi người dùng nhấn nút Xác nhận: Ứng dụng đọc lại dữ liệu ngày đích từ `localStorage` (`readDestinationTasks(destDate)`).
- Nếu phát hiện chuỗi lưu trữ thô đã thay đổi so với chuỗi lưu lúc xem trước (`freshResult.rawSnapshot !== cachedSnapshot`): Chặn ghi tức thời, hiển thị cảnh báo dữ liệu đã thay đổi, tự động tính toán lại xem trước từ dữ liệu mới và yêu cầu xác nhận lại.

### 14.7. Trùng khoảng giờ nghiêm ngặt (Strict Overlap Math)
- Điều kiện giao nhau nghiêm ngặt: $A_{\text{start}} < B_{\text{end}} \land B_{\text{start}} < A_{\text{end}}$.
- **Tiếp xúc biên không phải trùng giờ:** $A_{\text{end}} = B_{\text{start}}$ hoặc $B_{\text{end}} = A_{\text{start}}$ được tính là liền kề hợp lệ, không tính vào số đếm xung đột.
- Số đếm $Z$: Số lượng công việc trong số các mục mới có khoảng giờ giao nhau với công việc đã có ở ngày đích. Cảnh báo hiển thị dưới dạng thông tin không chặn lưu (non-blocking).

### 14.8. Xác minh số lần ghi LocalStorage không làm lộ dữ liệu
- QA và bài kiểm tra có thể cài đặt wrapper theo dõi trên `Storage.prototype.setItem` để đếm số lần gọi và kiểm tra tên khóa mà không bao giờ ghi nhận hoặc in nội dung `value` của người dùng.
- Toàn bộ 12 bất biến toán học và mã giả mẫu được quy định đầy đủ tại [`COPY_ARCHITECTURE.md`](file:///COPY_ARCHITECTURE.md).

## 15. Đặc tả giao diện và tương tác (01/10/2026) — Quản lý công việc tuần

Đặc tả này bổ sung thiết kế tương tác và giao diện chi tiết cho tính năng **“Quản lý công việc tuần”** (Weekly Task Management), hoàn thiện trên nền tảng ứng dụng lịch trình cá nhân tiếng Việt độc lập (`index.html`). Toàn bộ thiết kế kế thừa hệ thống thị giác, chuẩn tiếp cận WCAG AA, kích thước tương tác tối thiểu 44 px và nguyên tắc an toàn DOM tuyệt đối (zero `innerHTML`) đã được nghiệm thu trong các vòng trước.

---

### 15.1. Bối cảnh, mục tiêu và phân loại phản hồi người dùng (Triage Feedback 07bd3853ccbd)

Phản hồi người dùng `07bd3853ccbd` yêu cầu tính năng quản lý công việc tuần và đặt ra các câu hỏi định hướng. Phân loại kỹ thuật và quyết định thiết kế:
1. **Bản chất tính năng:** Xây dựng **Chế độ xem tổng quan tuần 7 ngày** (Thứ Hai – Chủ Nhật), tổng hợp dữ liệu động từ các khóa ngày sẵn có (`lich-trinh-hang-ngay:v1:YYYY-MM-DD`). Không tạo danh sách việc không gán thời gian độc lập để bảo toàn kiến trúc lịch trình theo mốc thời gian.
2. **Mô hình dữ liệu:** Tổng hợp động trực tiếp tại thời điểm hiển thị từ các khóa ngày; không tạo khóa tuần riêng (`week:YYYY-Wxx`) nhằm tránh rủi ro lệch đồng bộ dữ liệu (secondary desync) và giữ nguyên tính toàn vẹn của mô hình lưu trữ cục bộ.
3. **Phạm vi tương tác trong giao diện tuần:** Cho phép xem tổng quan, điều hướng giữa các tuần, bật/tắt trực tiếp trạng thái hoàn thành công việc (ghi tức thì vào ngày tương ứng), và điều hướng nhanh (Quick Jump) về chế độ xem ngày để thêm/sửa/xóa việc chi tiết.
4. **Xác nhận ngoài phạm vi (Out of Scope):** Tiếp tục tuyệt đối KHÔNG tích hợp đồng bộ Google Calendar/Notion, không có công việc lặp lại định kỳ tự động, không máy chủ backend và không dùng Web Notification API.

---

### 15.2. Bộ chuyển đổi chế độ xem (View Mode Switcher: Xem theo ngày | Xem theo tuần)

Bộ chuyển đổi chế độ xem là điều khiển cấp cao nhất cho phép người dùng chuyển đổi qua lại giữa giao diện xem ngày chi tiết (hiện tại) và giao diện xem tổng quan 7 ngày của tuần.

#### 1. Cấu trúc ngữ nghĩa HTML & ARIA
Sử dụng mô hình WAI-ARIA Tabs pattern chuẩn mực:
```html
<nav class="view-mode-nav" aria-label="Chế độ xem lịch">
  <div class="view-mode-tabs" role="tablist" aria-orientation="horizontal">
    <button type="button" 
            role="tab" 
            id="tab-view-daily" 
            class="tab-btn active" 
            aria-selected="true" 
            aria-controls="daily-view-panel" 
            tabindex="0">
      Xem theo ngày
    </button>
    <button type="button" 
            role="tab" 
            id="tab-view-weekly" 
            class="tab-btn" 
            aria-selected="false" 
            aria-controls="weekly-view-panel" 
            tabindex="-1">
      Xem theo tuần
    </button>
  </div>
</nav>
```

#### 2. Vị trí & Trình bày thị giác
- **Vị trí:** Đặt ngay trong phần mở đầu của `<main>`, phía trên khung chọn ngày hoặc tích hợp hài hòa liền kề tiêu đề chính của khu vực nội dung.
- **Hình thức:** Thanh chuyển tab dạng viên thuốc (segmented pill control) với nền `#e2e8f0`, bo góc 10 px, đệm 4 px.
- **Tab kích hoạt (Active tab):** Nền trắng `#ffffff`, chữ `#172033` (font-weight: 750), đổ bóng nhẹ `0 2px 6px rgba(0,0,0,0.08)`.
- **Tab không kích hoạt:** Chữ `#526078`, hover nền `#edf2f7`, hover chữ `#172033`.
- **Kích thước mục tiêu cảm ứng:** Chiều cao tối thiểu 44 px (`min-height: 44px; padding: 10px 20px;`), diện tích chạm tối thiểu 44×44 CSS px cho mỗi tab.
- **Hiển thị Focus (`:focus-visible`):** Viền outline 3 px màu `#0b63ce` với khoảng cách viền 2 px (`outline-offset: 2px`).

#### 3. Điều hướng bàn phím theo chuẩn WAI-ARIA
- Phím `Tab`: Đưa focus vào tab đang được chọn (`aria-selected="true"`, `tabindex="0"`). Nhấn `Tab` lần nữa sẽ chuyển focus ra khỏi thanh tab vào nội dung bên trong panel hiển thị.
- Phím `Mũi tên Trái (←)` / `Mũi tên Phải (→)` (hoặc `↑` / `↓`): Di chuyển focus sang tab liền kề, tự động cập nhật `aria-selected="true"`, chuyển tab cũ về `aria-selected="false"` kèm `tabindex="-1"`, kích hoạt hiển thị panel tương ứng và phát thông báo qua live region.
- Phím `Home` / `End`: Di chuyển focus tức thì về tab đầu tiên ("Xem theo ngày") hoặc tab cuối cùng ("Xem theo tuần").
- Phím `Enter` / `Space`: Kích hoạt tab nếu dùng cơ chế kích hoạt thủ công.

---

### 15.3. Thanh điều khiển điều hướng tuần (Week Navigation Controls) & Đồng bộ ngày/tuần

Khi chuyển sang tab "Xem theo tuần", giao diện hiển thị thanh công cụ điều hướng tuần với phạm vi ngày từ Thứ Hai đến Chủ Nhật.

#### 1. Cấu trúc ngữ nghĩa & Điều khiển
```html
<section class="panel week-navigation-panel" aria-labelledby="week-heading">
  <div class="week-nav-header">
    <div class="week-heading-group">
      <h2 id="week-heading" class="week-range-title">Tuần: 28/09/2026 – 04/10/2026</h2>
      <p class="week-range-subtitle" id="week-range-desc">Từ Thứ Hai ngày 28/09 đến Chủ Nhật ngày 04/10/2026</p>
    </div>
    <div class="week-nav-actions" role="toolbar" aria-label="Điều hướng tuần">
      <button type="button" class="btn btn-secondary week-nav-btn" id="week-prev-btn" aria-label="Chuyển sang tuần trước">
        ← Tuần trước
      </button>
      <button type="button" class="btn btn-secondary week-nav-btn" id="week-today-btn" aria-label="Về tuần hiện tại">
        Tuần này
      </button>
      <button type="button" class="btn btn-secondary week-nav-btn" id="week-next-btn" aria-label="Chuyển sang tuần sau">
        Tuần sau →
      </button>
    </div>
  </div>
</section>
```

#### 2. Định dạng chuỗi ngày tiếng Việt chuẩn mực
- Tiêu đề dải tuần: **“Tuần: DD/MM/YYYY – DD/MM/YYYY”** (Ví dụ: *“Tuần: 28/09/2026 – 04/10/2026”*).
- Quy ước tuần: Luôn bắt đầu từ **Thứ Hai (Monday)** và kết thúc vào **Chủ Nhật (Sunday)** theo chuẩn ISO-8601 và thói quen sinh hoạt tại Việt Nam.

#### 3. Quy tắc đồng bộ hai chiều giữa Ngày đang chọn và Tuần kích hoạt
- **Từ Ngày sang Tuần:** Khi người dùng đang ở chế độ xem ngày với ngày `D` (ví dụ `2026-09-30`), chuyển sang tab Tuần sẽ tự động tính toán và hiển thị tuần chứa ngày `D` (tuần 28/09/2026 – 04/10/2026).
- **Từ Tuần sang Ngày:** Khi người dùng chuyển tuần (bấm "Tuần trước", "Tuần sau", "Tuần này"), ngày được chọn trong hệ thống cập nhật tương ứng theo thứ trong tuần hoặc ngày đầu tuần (Thứ Hai), đảm bảo không xảy ra hiện tượng lệch ngày.
- **Nút “Tuần này”:** Tính toán dải tuần từ Thứ Hai đến Chủ Nhật bao chứa ngày thực tế hôm nay của hệ thống (`new Date()`).
- **Thông báo Live Region (`#app-status`):** Mỗi lần đổi tuần phát thông báo:
  *“Đã chuyển sang tuần từ {DD/MM/YYYY} đến {DD/MM/YYYY}.”*

---

### 15.4. Bảng tổng kết số liệu tuần (Weekly Summary Metrics Panel)

Đặt ngay trên đầu giao diện tuần, cung cấp bức tranh định lượng tổng thể về tiến độ công việc trong 7 ngày.

#### 1. Cấu trúc hiển thị & 3 số liệu cốt lõi
```html
<section class="panel weekly-summary-panel" aria-labelledby="weekly-summary-title">
  <h3 id="weekly-summary-title" class="summary-title">Tổng quan tuần</h3>
  <div class="weekly-metrics-grid">
    <div class="metric-card metric-total">
      <span class="metric-label">Tổng công việc</span>
      <strong class="metric-value" id="metric-week-total">0 việc</strong>
      <span class="metric-subtext">Trong 7 ngày</span>
    </div>
    <div class="metric-card metric-completed">
      <span class="metric-label">Đã hoàn thành</span>
      <strong class="metric-value" id="metric-week-completed">0 việc</strong>
      <span class="metric-subtext" id="metric-week-percent">0% tiến độ</span>
    </div>
    <div class="metric-card metric-remaining">
      <span class="metric-label">Chưa hoàn thành</span>
      <strong class="metric-value" id="metric-week-remaining">0 việc</strong>
      <span class="metric-subtext">Cần thực hiện</span>
    </div>
  </div>
</section>
```

#### 2. Quy tắc tính toán & Bất biến
- Tổng công việc: $\text{Total} = \sum_{i=1}^7 \text{Tasks}(Day_i)$.
- Hoàn thành: $\text{Completed} = \sum_{i=1}^7 \text{CompletedTasks}(Day_i)$.
- Còn lại: $\text{Remaining} = \text{Total} - \text{Completed}$.
- Tỷ lệ phần trăm: Khi $\text{Total} > 0$, $\text{Percent} = \text{round}((\text{Completed} / \text{Total}) \times 100)\%$; khi $\text{Total} = 0$, hiển thị *“0% tiến độ”*.
- Cập nhật thời gian thực: Khi người dùng bấm checkbox hoàn thành một công việc ở bất kỳ ngày nào trên giao diện tuần, số liệu tại bảng này được cập nhật ngay lập tức mà không cần tải lại trang.

---

### 15.5. Bố cục tổng quan 7 ngày (7-Day Weekly Overview Layout)

#### 1. Bố cục Desktop / Tablet ngang (Viewport ≥ 768 px)
- **Lưới 7 cột (7-Column Grid):**
  ```css
  .weekly-grid {
    display: grid;
    grid-template-columns: repeat(7, minmax(0, 1fr));
    gap: 12px;
    align-items: start;
    width: 100%;
  }
  ```
- **Mỗi cột đại diện cho 1 ngày trong tuần (từ Thứ Hai đến Chủ Nhật):**
  - **Khung tiêu đề cột (`.day-column-header`):**
    - Tên thứ: `<span class="day-name">Thứ Hai</span>` (font-weight: 750, 0.95rem).
    - Ngày tháng: `<time class="day-date">28/09</time>` (màu `#526078`, font-size: 0.85rem).
    - Huy hiệu số lượng: `<span class="badge day-count-badge">3 việc</span>`.
    - Nút điều hướng nhanh: `<button type="button" class="btn btn-secondary btn-sm day-jump-btn" aria-label="Xem chi tiết ngày Thứ Hai, 28/09/2026">Xem ngày</button>` (chiều cao tối thiểu 44 px).
  - **Điểm nhấn ngày hôm nay (`.is-today`):**
    - Cột đại diện cho ngày hôm nay thực tế có viền trên nổi bật `border-top: 4px solid var(--primary)`, nền nhẹ `#f8faff`, huy hiệu nhỏ **“Hôm nay”** (`badge-primary`).
  - **Điểm nhấn ngày đang chọn (`.is-selected`):**
    - Cột tương ứng với ngày đang lưu trong `#schedule-date` có viền nhận diện rõ ràng.
  - **Danh sách công việc trong ngày:**
    - `<div class="day-tasks-list" role="list">` chứa các thẻ công việc thu gọn xếp theo thứ tự thời gian tăng dần (`start` → `end` → `createdAt`).

#### 2. Bố cục Mobile (Viewport 320 px – 767 px): Accordion dọc hoặc Danh sách xếp tầng
Nhằm triệt tiêu 100% rủi ro tràn chiều ngang (horizontal overflow) trên màn hình hẹp 320 px, giao diện chuyển sang bố cục accordion dọc hoặc danh sách xếp tầng 7 ngày thân thiện với ngón tay:
- **Cấu trúc Accordion dễ tiếp cận:**
  ```html
  <div class="day-accordion-group" role="region" aria-label="Danh sách 7 ngày trong tuần">
    <div class="day-accordion-item panel" id="accordion-day-2026-09-28">
      <button type="button" 
              class="day-accordion-trigger" 
              id="trigger-day-2026-09-28" 
              aria-expanded="true" 
              aria-controls="panel-day-2026-09-28">
        <div class="accordion-title-wrap">
          <span class="day-name">Thứ Hai</span>
          <time class="day-date">28/09/2026</time>
          <span class="badge day-count-badge">3 việc</span>
        </div>
        <span class="accordion-icon" aria-hidden="true">▲</span>
      </button>
      <div class="day-accordion-panel" 
           id="panel-day-2026-09-28" 
           role="region" 
           aria-labelledby="trigger-day-2026-09-28">
        <div class="day-panel-actions">
          <button type="button" class="btn btn-secondary btn-sm day-jump-btn" data-date="2026-09-28">
            Xem lịch ngày này →
          </button>
        </div>
        <div class="day-tasks-list" role="list">
          <!-- Compact task cards -->
        </div>
      </div>
    </div>
  </div>
  ```
- **Hành vi trên Mobile:**
  - Nút trigger mở gập đạt chiều cao tối thiểu 48 px, vùng đệm chạm rộng rãi.
  - Hỗ trợ phím `Enter` và `Space` để bật/mở nội dung ngày.
  - Ngày đang chọn hoặc ngày hôm nay mặc định mở (`aria-expanded="true"`); các ngày khác có thể mở/gập linh hoạt.
  - Tuyệt đối không tạo cuộn ngang trang (`overflow-x: hidden`, `width: 100%`). Mọi chuỗi văn bản dài đều được xuống dòng an toàn (`overflow-wrap: anywhere`).

---

### 15.6. Thẻ công việc thu gọn trong giao diện tuần (Compact Task Presentation & Completion Toggle)

Trong giao diện tuần, không gian mỗi ngày hẹp hơn giao diện ngày, do đó thẻ công việc được thiết kế tối giản, cô đọng nhưng giữ trọn vẹn thông tin nhận diện cốt lõi và khả năng tương tác độc lập.

#### 1. Cấu trúc ngữ nghĩa của thẻ công việc thu gọn
```html
<article class="compact-task-card company" id="week-task-card-abc123" role="listitem">
  <div class="compact-task-header">
    <time class="compact-task-time">08:30 – 09:15</time>
    <div class="compact-badges">
      <span class="badge badge-sm badge-company">Công ty</span>
      <span class="badge badge-sm badge-priority">Ưu tiên vừa</span>
      <!-- Gắn thêm nếu trùng giờ: -->
      <!-- <span class="badge badge-sm badge-overlap">Trùng giờ</span> -->
    </div>
  </div>
  <h4 class="compact-task-title">Họp giao ban đầu tuần</h4>
  <div class="compact-completion-control">
    <input type="checkbox" 
           id="week-chk-abc123" 
           class="compact-checkbox" 
           data-task-id="abc123" 
           data-task-date="2026-09-28">
    <label for="week-chk-abc123" class="compact-completion-label">
      <span class="compact-label-text">Đánh dấu ‘Họp giao ban đầu tuần’ là hoàn thành</span>
    </label>
  </div>
</article>
```

#### 2. Kích thước tương tác tối thiểu 44×44 CSS px cho Checkbox hoàn thành
- Để tuân thủ nghiêm ngặt chuẩn tiếp cận WCAG 2.5.5 và sửa chữa dứt điểm lỗi target size từ các vòng trước:
  - Khung bao `.compact-completion-control` có `min-height: 44px; display: flex; align-items: center;`.
  - Phím checkbox native giữ kích thước 20×20 px trực quan.
  - Nhãn `<label>` được gán `display: inline-flex; align-items: center; min-width: 44px; min-height: 44px; cursor: pointer;`.
  - Toàn bộ vùng nhãn là mục tiêu cảm ứng hợp lệ kết nối với checkbox bằng `for`/`id`.

#### 3. Hành vi chuyển đổi hoàn thành (Completion Toggle Contract)
- **Thao tác người dùng:** Người dùng nhấp/chạm vào checkbox hoặc nhãn tương ứng.
- **Xử lý lưu trữ:**
  - Ứng dụng đọc khóa `localStorage` của đúng ngày chứa công việc đó (`lich-trinh-hang-ngay:v1:YYYY-MM-DD`).
  - Đảo giá trị `completed` (`true` ↔ `false`) của công việc mục tiêu; **giữ nguyên 100%** các trường khác (`id`, `title`, `start`, `end`, `category`, `priority`, `createdAt`) và giữ nguyên thứ tự các công việc khác.
  - Thực hiện đúng **1 lượt ghi** vào khóa ngày đó; **0 lượt ghi** vào các ngày khác.
- **Phản hồi giao diện & Trợ năng:**
  - Thẻ công việc chuyển sang phong cách hoàn thành: nền nhạt `var(--success-bg)`, tiêu đề gạch ngang (`line-through`), nhãn đổi thành *“Bỏ đánh dấu hoàn thành cho ‘...’”*.
  - Bảng tổng kết tuần (Summary Panel) cập nhật ngay số lượng hoàn thành và tỷ lệ tiến độ.
  - Vùng live region `#app-status` thông báo rõ:
    *“Đã đánh dấu hoàn thành công việc: {tiêu đề} (ngày {DD/MM/YYYY}).”* hoặc *“Đã bỏ đánh dấu hoàn thành: {tiêu đề} (ngày {DD/MM/YYYY}).”*.

---

### 15.7. Điều hướng nhanh sang giao diện ngày (Quick Day Navigation)

Nhằm tối ưu luồng thao tác người dùng muốn thêm việc mới hoặc chỉnh sửa chi tiết công việc của một ngày bất kỳ trong tuần:
1. **Nút điều hướng nhanh:**
   - Đặt trên đầu mỗi cột ngày (desktop) hoặc trong panel của từng ngày (mobile).
   - Nhãn nút: **“Xem ngày”** (trên desktop gọn) hoặc **“Xem lịch ngày này →”** (trên mobile).
   - Kích thước tương tác: Chiều cao tối thiểu 44 px (`min-height: 44px;`).
   - Accessible name: `aria-label="Xem lịch chi tiết ngày {Thứ}, {DD/MM/YYYY}"`.
2. **Luồng tương tác khi kích hoạt:**
   - Ứng dụng cập nhật giá trị trường `#schedule-date` bằng chuỗi `YYYY-MM-DD` của ngày tương ứng.
   - Chuyển chế độ xem sang tab **“Xem theo ngày”** (`aria-selected="true"` cho tab ngày, kích hoạt panel ngày).
   - Tải và kết xuất lịch trình chi tiết của ngày đó lên form và timeline.
   - Di chuyển focus một cách hợp lý và tường minh tới tiêu đề lịch trình `#timeline-heading` (hoặc ô nhập tiêu đề nếu người dùng muốn thêm việc).
   - Phát thông báo qua live region: *“Đã mở lịch trình chi tiết ngày {Thứ}, {DD/MM/YYYY}.”*.

---

### 15.8. Thiết kế các trạng thái rỗng (Empty States: Ngày rỗng & Toàn tuần rỗng)

Không bao giờ để khoảng trống vô nghĩa hoặc tự ý tạo dữ liệu mẫu khi người dùng chưa có lịch.

#### 1. Trạng thái ngày không có công việc (Day with no tasks)
- Trong cột ngày (desktop) hoặc panel accordion (mobile), nếu ngày đó hợp lệ nhưng chưa có công việc nào (`tasks.length === 0`):
  - Hiển thị card trạng thái rỗng nhẹ nhàng:
    ```html
    <div class="day-empty-box">
      <p class="day-empty-text">Chưa có công việc</p>
    </div>
    ```
  - Viền nét đứt mảnh `#cbd5e1`, nền `#f8fafc`, chữ màu `#526078` (font-size: 0.85rem), căn giữa.

#### 2. Trạng thái toàn bộ tuần không có công việc (Entire week with no tasks)
- Khi cả 7 ngày trong tuần đều có 0 công việc:
  - Phía trên lưới 7 ngày hoặc thay thế vùng lưới hiển thị một banner thông báo rỗng trang trọng:
    ```html
    <div class="panel week-empty-state" role="status">
      <h3 class="week-empty-title">Tuần này chưa có công việc nào</h3>
      <p class="week-empty-desc">
        Tất cả các ngày từ {Thứ Hai, DD/MM} đến {Chủ Nhật, DD/MM/YYYY} hiện đang trống lịch.
      </p>
      <button type="button" class="btn btn-primary" id="week-start-monday-btn">
        Thêm công việc cho Thứ Hai ({DD/MM})
      </button>
    </div>
    ```
  - Nút **“Thêm công việc cho Thứ Hai”** có chiều cao tối thiểu 44 px. Khi bấm sẽ chuyển sang chế độ xem ngày của ngày Thứ Hai đầu tuần và đặt con trỏ focus ngay vào trường `#task-title` của biểu mẫu.

---

### 15.9. Xử lý an toàn ngày hỏng dữ liệu (Graceful Non-Crashing Corrupt Day State)

Ứng dụng phải tuyệt đối kiên cố trước tình huống dữ liệu cục bộ trong `localStorage` bị hỏng cấu trúc (JSON hỏng hoặc bản ghi thiếu trường/sai logic thời gian):

1. **Nguyên tắc cô lập lỗi (Fault Isolation):**
   - Sự cố dữ liệu ở một ngày bất kỳ (ví dụ Thứ Tư bị hỏng JSON) **tuyệt đối không được làm sập toàn bộ giao diện tuần** và **không được ảnh hưởng đến việc hiển thị 6 ngày hợp lệ còn lại**.
   - Bộ nạp tuần duyệt độc lập từng ngày qua bộ nạp nghiêm ngặt (`isValidStoredTask`).

2. **Giao diện cảnh báo ngày bị hỏng:**
   - Cột/panel của ngày bị hỏng hiển thị thẻ thông báo lỗi trực quan:
     ```html
     <div class="day-corrupt-card notice error" role="alert">
       <strong class="corrupt-title">⚠ Dữ liệu ngày bị lỗi</strong>
       <p class="corrupt-desc">
         Dữ liệu lưu trữ của ngày này bị hỏng cấu trúc hoặc bản ghi không hợp lệ.
       </p>
       <button type="button" class="btn btn-secondary btn-sm day-jump-btn" data-date="YYYY-MM-DD">
         Mở ngày để khắc phục
       </button>
     </div>
     ```
   - Nút **“Mở ngày để khắc phục”** cho phép người dùng chuyển sang xem ngày đó ở giao diện ngày, tại đây cơ chế phục hồi sẵn có sẽ cho phép người dùng thêm công việc hợp lệ mới để ghi đè thay thế dữ liệu hỏng.

3. **Tác động đến bảng tổng kết tuần (Summary Panel):**
   - Bảng tổng kết tuần vẫn tính toán tổng số việc từ các ngày hợp lệ, đồng thời hiển thị thêm huy hiệu cảnh báo phụ:
     *“⚠ Có {K} ngày bị lỗi dữ liệu (không tính vào tổng số).”* màu vàng cam/đỏ.

---

### 15.10. Bảng ma trận các trạng thái giao diện và vi văn bản (UI States & Microcopy Matrix)

| Tình huống giao diện | Trình bày thị giác & Điều khiển | Vi văn bản hiển thị (Microcopy) | Hành vi bàn phím, Focus & Live Region |
| :--- | :--- | :--- | :--- |
| **1. Chuyển tab Xem theo tuần** | Tab Tuần kích hoạt (`aria-selected="true"`), ẩn panel ngày, hiện panel tuần. Dải tuần tương ứng với ngày đang chọn. | Tab label: *“Xem theo tuần”*. Tiêu đề: *“Tuần: DD/MM/YYYY – DD/MM/YYYY”*. | Focus chuyển mượt mà; live region: *“Đã chuyển sang chế độ xem theo tuần (từ DD/MM/YYYY đến DD/MM/YYYY).”* |
| **2. Bấm Tuần trước / Tuần sau** | Cập nhật dải tuần lùi/tiến 7 ngày; kết xuất lại 7 cột ngày tương ứng. | Tiêu đề tuần mới: *“Tuần: DD/MM/YYYY – DD/MM/YYYY”*. | Nút bấm giữ focus; live region: *“Đã chuyển sang tuần từ DD/MM/YYYY đến DD/MM/YYYY.”* |
| **3. Bấm Tuần này** | Cập nhật dải tuần về tuần chứa ngày hôm nay thực tế. Cột ngày hôm nay gắn huy hiệu *“Hôm nay”*. | Nút: *“Tuần này”*. Tiêu đề: *“Tuần: DD/MM/YYYY – DD/MM/YYYY”*. | Live region: *“Đã trở về tuần hiện tại.”* |
| **4. Bật/tắt hoàn thành trên thẻ tuần** | Checkbox đổi trạng thái; thẻ đổi màu nền và gạch ngang tiêu đề; số liệu tổng kết cập nhật ngay. | Checkbox label: *“Đánh dấu ‘{tiêu đề}’ là hoàn thành”* / *“Bỏ đánh dấu hoàn thành cho ‘{tiêu đề}’”*. | Checkbox giữ focus; live region: *“Đã cập nhật trạng thái công việc ‘{tiêu đề}’.”* Đúng 1 lượt ghi vào ngày đó. |
| **5. Bấm Xem ngày từ cột tuần** | Chuyển sang tab Xem theo ngày; cập nhật `#schedule-date`; tải form và timeline ngày đó. | Nút: *“Xem ngày”* hoặc *“Xem lịch ngày này →”*. | Focus chuyển tới `#timeline-heading`; live region: *“Đã mở lịch trình chi tiết ngày {Thứ}, {DD/MM/YYYY}.”* |
| **6. Ngày trong tuần không có việc** | Khung ngày hiển thị hộp rỗng viền nét đứt. | *“Chưa có công việc”*. | Người dùng vẫn có thể bấm nút *“Xem ngày”* để thêm việc. |
| **7. Toàn bộ tuần không có việc** | Banner rỗng lớn xuất hiện phía trên hoặc trong lưới tuần. | *“Tuần này chưa có công việc nào. Tất cả các ngày từ DD/MM đến DD/MM hiện đang trống lịch.”* | Nút *“Thêm công việc cho Thứ Hai”* sẵn sàng nhận focus. |
| **8. Một ngày bị hỏng dữ liệu** | Cột ngày đó hiện banner lỗi đỏ; 6 ngày còn lại hiển thị bình thường; bảng tổng kết ghi chú cảnh báo. | *“⚠ Dữ liệu ngày bị lỗi. Dữ liệu lưu trữ của ngày này bị hỏng cấu trúc.”* | Nút *“Mở ngày để khắc phục”* điều hướng an toàn về giao diện ngày. Zero crash. |

---

### 15.11. Tiêu chí tiếp cận bàn phím, WCAG AA, kích thước tương tác 44px và DOM an toàn

1. **Chuẩn kích thước mục tiêu cảm ứng (Target Size Minimum 44×44 CSS px):**
   - Mọi nút bấm chuyển tab (`.tab-btn`): chiều cao tối thiểu 44 px.
   - Các nút điều hướng tuần (`#week-prev-btn`, `#week-today-btn`, `#week-next-btn`): chiều cao tối thiểu 44 px.
   - Các nút điều hướng nhanh (`.day-jump-btn`): chiều cao tối thiểu 44 px.
   - Các nút trigger mở gập accordion trên di động (`.day-accordion-trigger`): chiều cao tối thiểu 48 px.
   - Điều khiển hoàn thành trong thẻ công việc (`.compact-completion-control label`): chiều cao tối thiểu 44 px, chiều rộng tối thiểu 44 px.

2. **Tương phản màu sắc & Hiển thị Focus (WCAG AA):**
   - Toàn bộ văn bản thông tin chính đạt tỷ lệ tương phản tối thiểu 4.5:1 so với màu nền.
   - Huy hiệu trạng thái, huy hiệu danh mục và đường viền điều khiển đạt tối thiểu 3:1.
   - Tất cả các thành phần tương tác đều có hiệu ứng `:focus-visible` với đường viền outline 3 px màu `#0b63ce`, cách phần tử 2 px (`outline-offset: 2px`).

3. **Bố cục Responsive & Kháng tràn ngang ở 320 px (Zero Horizontal Overflow):**
   - Trên desktop/tablet (≥ 768 px): Phân bổ lưới 7 cột co giãn đều với `minmax(0, 1fr)`.
   - Trên mobile (320 px – 767 px): Sử dụng danh sách xếp tầng hoặc accordion một cột dọc với `width: 100%; box-sizing: border-box; overflow-x: hidden;`.
   - Mọi tiêu đề hoặc văn bản dài đều áp dụng `overflow-wrap: anywhere; word-break: break-word;`.

4. **Nguyên tắc an toàn DOM tuyệt đối (Strict Zero-innerHTML Policy):**
   - Toàn bộ quá trình tạo mới và kết xuất các phần tử của giao diện tuần (thanh tab, bảng tổng kết, các cột ngày, thẻ công việc thu gọn, thông báo rỗng, cảnh báo lỗi) phải được thực hiện 100% bằng các API DOM an toàn:
     - `document.createElement(tagName)`
     - `node.textContent = textValue`
     - `element.setAttribute(name, value)`
     - `element.classList.add(...)`
   - **Tuyệt đối cấm** sử dụng `innerHTML`, `insertAdjacentHTML`, `outerHTML`, hoặc `document.write` ở bất kỳ dòng mã nào.


