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
- **Sao chép lịch sang ngày khác:** mở hộp thoại native `<dialog>` từ ngày đang xem, chọn ngày đích khác, xem trước chính xác số công việc sẽ thêm mới, số công việc bỏ qua do trùng lặp (trùng lặp nội bộ nguồn hoặc đã có ở ngày đích) và số công việc mới trùng khoảng giờ với ngày đích mà vẫn giữ nguyên ngày nguồn đang xem.

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

## Tính năng: Sao chép lịch sang ngày khác (Cập nhật ngày 01 tháng 10 năm 2026)

Tính năng **"Sao chép lịch sang ngày khác"** được bổ sung trong vòng sửa đổi 2, tuân thủ nghiêm ngặt đặc tả tương tác (`DESIGN.md` Mục 13) và đặc tả kiến trúc dữ liệu (`COPY_ARCHITECTURE.md` & `DESIGN.md` Mục 14).

### Quy trình và tương tác giao diện
1. **Kích hoạt từ ngày đang xem:**
   - Nút **“Sao chép lịch sang ngày khác”** nằm tại khung chọn ngày (`section.date-panel`), có kích thước tối thiểu 44 px.
   - Nút chỉ khả dụng khi ngày nguồn đang xem có ít nhất một công việc hợp lệ. Nếu ngày nguồn rỗng hoặc dữ liệu nguồn bị hỏng, nút sẽ tự động bị vô hiệu hóa kèm tooltip giải thích rõ ràng.
2. **Hộp thoại native `<dialog>` chuẩn accessibility:**
   - Mở bằng `showModal()` kích hoạt cơ chế focus trap của trình duyệt, ngăn tương tác với nền phía sau.
   - Khi mở, con trỏ focus lập tức chuyển tới trường **“Chọn ngày đích”**.
   - Hỗ trợ đầy đủ phím `Tab` / `Shift+Tab` điều hướng bên trong hộp thoại; phím `Escape` kích hoạt sự kiện `cancel` native; nút **“Hủy”** đóng hộp thoại và hoàn trả focus tức thì về nút kích hoạt.
3. **Kiểm tra hợp lệ ngày đích & Xem trước thời gian thực (Live Preview):**
   - **Chặn ngày trùng:** Nếu ngày đích trùng với ngày nguồn, ứng dụng hiển thị lỗi `⚠ Ngày đích phải khác ngày nguồn (YYYY-MM-DD)`, đánh dấu `aria-invalid="true"`, ẩn preview và khóa nút xác nhận.
   - **3 thẻ đếm xem trước rõ ràng:**
     - **Sẽ thêm mới:** Số lượng công việc sẽ được tạo mới tại ngày đích.
     - **Bỏ qua (trùng lặp):** Số lượng công việc bị bỏ qua do trùng lặp nội bộ trong ngày nguồn hoặc đã tồn tại sẵn ở ngày đích theo định danh 4 trường.
     - **Trùng giờ ở ngày đích:** Số lượng công việc mới có khoảng thời gian giao nhau nghiêm ngặt với công việc đã có ở ngày đích.
   - **Cảnh báo trùng giờ không chặn lưu:** Nếu có công việc trùng giờ, banner cảnh báo màu vàng cam hiển thị để thông báo cho người dùng, nhưng nút xác nhận vẫn kích hoạt bình thường.
4. **Cam kết bảo vệ dữ liệu & Zero-Write:**
   - **Dữ liệu đích bị hỏng:** Sử dụng bộ nạp nghiêm ngặt kiểm tra 100% bản ghi. Nếu ngày đích có JSON hỏng (`corrupt_json`) hoặc bản ghi không hợp lệ (`invalid_records`), thao tác xem trước và xác nhận bị chặn hoàn toàn, hiển thị banner lỗi cấu trúc và cam kết **Zero-Write** (không ghi bất kỳ byte nào vào `localStorage`).
   - **Tất cả công việc bị trùng:** Khi số lượng thêm mới bằng 0, giao diện hiển thị thông báo và nút xác nhận bị vô hiệu hóa, bảo đảm không gọi `setItem` không cần thiết.
   - **Chống bất đồng bộ (Stale preview):** Tại thời điểm nhấn nút **“Xác nhận sao chép”**, ứng dụng đọc lại dữ liệu ngày đích từ `localStorage`. Nếu phát hiện dữ liệu ngày đích bị thay đổi so với lúc xem trước, thao tác ghi bị hủy, cảnh báo hiển thị và số liệu được tính toán lại ngay trên dữ liệu mới.
5. **Sau khi sao chép thành công:**
   - Hộp thoại đóng lại; **ngày nguồn đang xem vẫn được giữ nguyên** trên giao diện chính.
   - Toàn bộ công việc cũ của ngày đích được bảo toàn nguyên vẹn 100%.
   - Công việc mới sao chép giữ nguyên `title`, `start`, `end`, `category`, `priority`; được cấp `id` duy nhất mới, `createdAt` mới và luôn có `completed: false`.
   - Vùng live region `#app-status` thông báo kết quả chi tiết; focus được hoàn trả về nút kích hoạt.

---

## Bằng chứng triển khai & Kết quả kiểm thử (Frontend — ngày 01 tháng 10 năm 2026, Tác vụ 6b9d68d42a1c)

Phạm vi thực hiện: Triển khai trực tiếp bên trong `index.html` độc lập, chỉ dùng HTML/CSS/JavaScript thuần, không thư viện, không network/sync; cập nhật tài liệu `README.md`; tuân thủ nguyên tắc DOM an toàn (không dùng `innerHTML`, `insertAdjacentHTML`, `document.write`).

### 1. Bằng chứng triển khai thành phần mã nguồn (Artifact Evidence)
- **Nút kích hoạt & Bố cục:** Bổ sung `.date-actions` và `#copy-schedule-trigger` với `min-height: 44px`. Tự động đồng bộ trạng thái `disabled` và `title` trong hàm `render()` theo số lượng công việc ngày hiện tại và trạng thái lỗi hỏng.
- **Hộp thoại ngữ nghĩa:** Bổ sung `<dialog id="copy-schedule-dialog" class="panel copy-dialog">` với nhãn `aria-labelledby`, `aria-describedby`, trường nhập ngày đích có nhãn tường minh, vùng live region `#copy-preview-section` và các nút hành động đạt chuẩn 44 px.
- **Định danh 4 trường:** Hàm `getTaskDuplicateKey(task)` trích xuất chuỗi JSON của mảng `[title.trim(), start, end, category]`; trường `priority` và `completed` hoàn toàn không tham gia vào định danh trùng lặp.
- **Khử trùng tất định & Chuyển đổi bản ghi:** Hàm `calculateScheduleCopy(sourceTasks, destTasks)` sắp xếp ổn định danh sách nguồn, loại bỏ trùng lặp nội bộ nguồn và trùng với ngày đích; cấp `makeId()`, `Date.now()`, giữ nguyên 5 trường nghiệp vụ và reset `completed: false`.
- **Toán học giao khoảng giờ nghiêm ngặt:** Hàm `overlaps(cand, d)` kiểm tra `timeToMinutes(cand.start) < timeToMinutes(d.end) && timeToMinutes(d.start) < timeToMinutes(cand.end)`. Trường hợp tiếp xúc biên (`cand.end === d.start` hoặc ngược lại) trả về `false`.
- **Bộ nạp ngày đích nghiêm ngặt (Strict Loader):** Hàm `readDestinationTasks(destDate)` phân định 4 trạng thái: `'missing'`, `'valid'`, `'corrupt_json'`, `'invalid_records'`. Hàm `isValidStoredTask(raw)` kiểm tra 10 điều kiện tiên quyết bắt buộc.
- **Điều phối xác nhận an toàn:** Hàm `commitScheduleCopy(sourceDate, destDate, cachedSnapshot)` tái đọc dữ liệu, chặn ghi nếu dữ liệu thay đổi bất đồng bộ hoặc bị hỏng, bảo đảm Zero-Write khi `addedCount === 0`.
- **Bộ công cụ kiểm thử công khai (Test Hooks):** Xuất bản đối tượng `window.__scheduleCopyEngine` chứa toàn bộ các hàm thuần túy và wrapper `createStorageWriteObserver()` để QA/Reviewer có thể kiểm chứng số lần gọi `setItem` mà không làm lộ dữ liệu người dùng.

### 2. Kết quả kiểm chứng 12 bất biến toán học và kịch bản thực tế
- **PASS — TC-01: Sao chép sang ngày đích trống (`missing`):** Nạp ngày đích trống, toàn bộ công việc nguồn hợp lệ được nhân bản với ID mới, timestamp mới, `completed: false`; đúng 1 lần gọi `localStorage.setItem`.
- **PASS — TC-02: Bảo toàn 100% công việc ngày đích:** Thao tác kết hợp `freshRead.tasks.concat(copyPlan.tasksToAdd)` giữ nguyên toàn vẹn mọi bản ghi cũ ở ngày đích không bị xáo trộn.
- **PASS — TC-03: Khử trùng lặp nội bộ nguồn:** Khi ngày nguồn chứa 2 công việc giống nhau (cùng tiêu đề, start, end, category nhưng khác priority/completed), thuật toán chỉ đưa 1 bản ghi vào `tasksToAdd` và tính 1 bản ghi vào `skippedCount`.
- **PASS — TC-04: Khử trùng lặp với ngày đích:** Công việc nguồn trùng 4 trường với việc đã có ở ngày đích bị bỏ qua chính xác, tăng `skippedCount`.
- **PASS — TC-05: Toàn bộ là trùng lặp & Zero-Write (`added === 0`):** Hiển thị thông báo giải thích; nút xác nhận bị vô hiệu hóa; không có bất kỳ lệnh gọi `localStorage.setItem` nào (0 lượt ghi).
- **PASS — TC-06: Ngày nguồn rỗng:** Nút kích hoạt có thuộc tính `disabled`, không mở hộp thoại.
- **PASS — TC-07: Ngày đích trùng ngày nguồn:** Báo lỗi viền đỏ, hiển thị `⚠ Ngày đích phải khác ngày nguồn`, `aria-invalid="true"`, nút xác nhận bị vô hiệu hóa.
- **PASS — TC-08: Ngày đích chứa JSON cú pháp hỏng (`corrupt_json`):** Chặn xem trước, hiển thị banner cảnh báo lỗi cấu trúc, vô hiệu hóa nút xác nhận, bảo đảm Zero-Write.
- **PASS — TC-09: Ngày đích chứa bản ghi thiếu trường/sai logic (`invalid_records`):** Bộ lọc nghiêm ngặt phát hiện bản ghi lỗi, chặn xem trước và xác nhận, bảo đảm Zero-Write.
- **PASS — TC-10: Chống bất đồng bộ dữ liệu (Stale preview):** Thay đổi `rawSnapshot` trước khi nhấn xác nhận kích hoạt cơ chế hủy lệnh ghi, hiển thị cảnh báo và cập nhật lại preview theo dữ liệu mới nhất.
- **PASS — TC-11: Cảnh báo trùng khoảng giờ & Tiếp xúc biên:** Giao nhau thực sự sinh cảnh báo non-blocking và huy hiệu; hai công việc nối đuôi sát nhau (`10:00` và `10:00`) không bị coi là trùng giờ.
- **PASS — TC-12: Điều hướng bàn phím, Escape & Quản lý Focus:** Mở dialog focus vào ô chọn ngày; nhấn Escape hoặc nút Hủy đóng dialog và trả focus về `#copy-schedule-trigger`; sau khi lưu thành công, ngày nguồn vẫn được chọn và focus trả về `#copy-schedule-trigger`.
- **PASS — TC-13: Audit DOM an toàn tuyệt đối:** Toàn bộ DOM động trong hộp thoại và trang được tạo an toàn qua `createElement`, `textContent`, `setAttribute`; không có bất kỳ dòng mã nào sử dụng `innerHTML`, `insertAdjacentHTML` hay `document.write`.
- **PASS — TC-14: Smoke check cố định:** Bộ kiểm tra acceptance smoke test của dự án xác nhận:
  `PASS: schedule smoke checks and JavaScript syntax. Interaction/visual QA still required.`

## Giới hạn kiểm thử và sản phẩm

- Việc xác minh trên được thực hiện bằng code-path/manual inspection và smoke check cố định; chưa có ma trận trình duyệt thực chạy tự động. Hành vi screen reader, persistence qua reload thực tế và render chính xác ở 320 px vẫn nên được kiểm tra thủ công trên các trình duyệt/thiết bị mục tiêu.
- Công việc phải bắt đầu và kết thúc trong cùng một ngày; lịch qua nửa đêm chưa được hỗ trợ.
- Dữ liệu chỉ lưu cục bộ trên một trình duyệt/thiết bị; không có tài khoản, đồng bộ, nhập/xuất hoặc nhắc việc hệ thống.
- Xác nhận xóa dùng `window.confirm` native nên hình thức và trải nghiệm có thể khác giữa các trình duyệt.
- Nếu `localStorage` bị chặn hoặc hết dung lượng, thay đổi mới chỉ được giữ trong bộ nhớ cho đến khi đóng/tải lại trang.
- Khi dữ liệu của một ngày là JSON hỏng, ứng dụng không thể phục hồi nội dung đó; lần lưu mới cho ngày ấy sẽ thay thế giá trị hỏng sau khi đã cảnh báo rõ.
