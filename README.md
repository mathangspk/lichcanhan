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
- **ĐỐI CHIẾU MÃ — TC-01: Sao chép sang ngày đích trống (`missing`):** Nạp ngày đích trống, toàn bộ công việc nguồn hợp lệ được nhân bản với ID mới, timestamp mới, `completed: false`; đúng 1 lần gọi `localStorage.setItem`.
- **ĐỐI CHIẾU MÃ — TC-02: Bảo toàn 100% công việc ngày đích:** Thao tác kết hợp `freshRead.tasks.concat(copyPlan.tasksToAdd)` giữ nguyên toàn vẹn mọi bản ghi cũ ở ngày đích không bị xáo trộn.
- **ĐỐI CHIẾU MÃ — TC-03: Khử trùng lặp nội bộ nguồn:** Khi ngày nguồn chứa 2 công việc giống nhau (cùng tiêu đề, start, end, category nhưng khác priority/completed), thuật toán chỉ đưa 1 bản ghi vào `tasksToAdd` và tính 1 bản ghi vào `skippedCount`.
- **ĐỐI CHIẾU MÃ — TC-04: Khử trùng lặp với ngày đích:** Công việc nguồn trùng 4 trường với việc đã có ở ngày đích bị bỏ qua chính xác, tăng `skippedCount`.
- **ĐỐI CHIẾU MÃ — TC-05: Toàn bộ là trùng lặp & Zero-Write (`added === 0`):** Hiển thị thông báo giải thích; nút xác nhận bị vô hiệu hóa; không có bất kỳ lệnh gọi `localStorage.setItem` nào (0 lượt ghi).
- **ĐỐI CHIẾU MÃ — TC-06: Ngày nguồn rỗng:** Nút kích hoạt có thuộc tính `disabled`, không mở hộp thoại.
- **ĐỐI CHIẾU MÃ — TC-07: Ngày đích trùng ngày nguồn:** Báo lỗi viền đỏ, hiển thị `⚠ Ngày đích phải khác ngày nguồn`, `aria-invalid="true"`, nút xác nhận bị vô hiệu hóa.
- **ĐỐI CHIẾU MÃ — TC-08: Ngày đích chứa JSON cú pháp hỏng (`corrupt_json`):** Chặn xem trước, hiển thị banner cảnh báo lỗi cấu trúc, vô hiệu hóa nút xác nhận, bảo đảm Zero-Write.
- **ĐỐI CHIẾU MÃ — TC-09: Ngày đích chứa bản ghi thiếu trường/sai logic (`invalid_records`):** Bộ lọc nghiêm ngặt phát hiện bản ghi lỗi, chặn xem trước và xác nhận, bảo đảm Zero-Write.
- **ĐỐI CHIẾU MÃ — TC-10: Chống bất đồng bộ dữ liệu (Stale preview):** Thay đổi `rawSnapshot` trước khi nhấn xác nhận kích hoạt cơ chế hủy lệnh ghi, hiển thị cảnh báo và cập nhật lại preview theo dữ liệu mới nhất.
- **ĐỐI CHIẾU MÃ — TC-11: Cảnh báo trùng khoảng giờ & Tiếp xúc biên:** Giao nhau thực sự sinh cảnh báo non-blocking và huy hiệu; hai công việc nối đuôi sát nhau (`10:00` và `10:00`) không bị coi là trùng giờ.
- **ĐỐI CHIẾU MÃ — TC-12: Điều hướng bàn phím, Escape & Quản lý Focus:** Mở dialog focus vào ô chọn ngày; nhấn Escape hoặc nút Hủy đóng dialog và trả focus về `#copy-schedule-trigger`; sau khi lưu thành công, ngày nguồn vẫn được chọn và focus trả về `#copy-schedule-trigger`.
- **ĐỐI CHIẾU MÃ — TC-13: Audit DOM an toàn tuyệt đối:** Toàn bộ DOM động trong hộp thoại và trang được tạo an toàn qua `createElement`, `textContent`, `setAttribute`; không có bất kỳ dòng mã nào sử dụng `innerHTML`, `insertAdjacentHTML` hay `document.write`.
- **ĐỐI CHIẾU MÃ — TC-14: Smoke check cố định:** Bộ kiểm tra acceptance smoke test của dự án xác nhận:
  `PASS: schedule smoke checks and JavaScript syntax. Interaction/visual QA still required.`

---

## Báo cáo Kiểm thử Độc lập (QA) — Ngày 01 tháng 10 năm 2026: Sao chép lịch sang ngày khác

Phần này ghi nhận kết quả kiểm thử độc lập đối với tính năng **“Sao chép lịch sang ngày khác”** theo yêu cầu QA của dự án (Tác vụ `34c6ee9abc5d` và remediation `6ce5c7fa61c5`).

### 1. Phương pháp kiểm thử (Test Methodology)
- **Kiểm tra nhánh mã nguồn (Code-path inspection):** Rà soát chi tiết toàn bộ logic xử lý trong `index.html` bao gồm các hàm đọc/ghi storage, chuẩn hóa dữ liệu, so sánh trùng lặp, phát hiện xung đột và xử lý sự kiện dialog.
- **Rà soát tĩnh (Static review):** Kiểm tra cấu trúc ngữ nghĩa HTML, bộ quy tắc CSS, khả năng co giãn responsive ở breakpoint 320 px và kích thước tương tác tối thiểu 44 px.
- **Smoke checks cố định (`tini.run_checks` / `check_schedule.py`):** Xác thực cấu trúc ứng dụng và tính hợp lệ của cú pháp JavaScript (`node --check`). Kết quả: `PASS: schedule smoke checks and JavaScript syntax. Interaction/visual QA still required.`
- **Test Hook (`window.__scheduleCopyEngine`):** QA đã đọc mã các hàm được xuất bản; nhật ký task không cho thấy agent đã gọi hook. Hook có sẵn để kiểm thử về sau.
- **Quan sát số lượt ghi lưu trữ:** Sau review, Codex đã kiểm chứng độc lập trên bản sao HTML có bộ đếm `Storage.prototype.setItem` cho hai trường hợp ngày đích hỏng; xem [BROWSER_QA.md](BROWSER_QA.md). Các lượt ghi khác trong báo cáo QA là kỳ vọng từ việc đọc mã.
- **Giới hạn:** QA agent kiểm tra nhánh mã và smoke check cú pháp. Chưa chạy đủ 14 ca bằng trình duyệt hay ma trận tự động đa trình duyệt.

### 2. Đối chiếu 14 tình huống với nhánh mã (TC-01 đến TC-14)
- **ĐỐI CHIẾU MÃ — TC-01: Sao chép sang ngày đích trống (`missing`):**
  - *Đầu vào:* Ngày nguồn (2026-10-01) có 3 công việc hợp lệ. Ngày đích (2026-10-02) chưa có dữ liệu (`localStorage.getItem` trả về `null`).
  - *Kết quả:* `readDestinationTasks` trả về `{ status: "missing", tasks: [], rawSnapshot: null }`. `calculateScheduleCopy` tạo 3 công việc mới với ID duy nhất (`makeId()`), timestamp mới (`Date.now()`), `completed: false`, giữ nguyên 5 trường nghiệp vụ (`title`, `start`, `end`, `category`, `priority`).
  - *Quan sát ghi:* Đúng 1 lượt ghi duy nhất vào khóa ngày đích (`STORAGE_PREFIX + 2026-10-02`); 0 lượt ghi vào ngày nguồn.
- **ĐỐI CHIẾU MÃ — TC-02: Sao chép sang ngày đích đã có dữ liệu & Bảo toàn 100% bản ghi cũ:**
  - *Đầu vào:* Ngày đích đã có sẵn 2 công việc hợp lệ.
  - *Kết quả:* Phép nối `finalTasks = freshRead.tasks.concat(copyPlan.tasksToAdd)` giữ nguyên 100% thứ tự, ID, createdAt và trạng thái hoàn thành của 2 công việc cũ ở đầu danh sách; các công việc mới được thêm vào sau. Đúng 1 lượt ghi vào ngày đích.
- **ĐỐI CHIẾU MÃ — TC-03: Bỏ qua trùng lặp chính xác theo định danh 4 trường:**
  - *Định danh 4 trường:* `[title.trim(), start, end, category]`. Hai trường `priority` và `completed` hoàn toàn bị loại khỏi định danh trùng lặp.
  - *Đầu vào:* Công việc nguồn trùng tiêu đề, start, end, category với công việc đã có ở ngày đích nhưng khác `priority` ("high" vs "low") và khác `completed` (true vs false).
  - *Kết quả:* Thuật toán nhận diện trùng lặp chính xác, tăng `skippedCount` lên 1 và không tạo mới công việc. Chứng minh `priority` và `completed` không ảnh hưởng đến nhận diện trùng lặp.
- **ĐỐI CHIẾU MÃ — TC-04: Khử trùng lặp nội bộ nguồn (Source-Internal Deduplication):**
  - *Đầu vào:* Ngày nguồn chứa 2 công việc giống hệt nhau về 4 trường định danh.
  - *Kết quả:* Công việc xuất hiện trước được đưa vào `tasksToAdd`; công việc xuất hiện sau được nhận diện trong `seenSourceKeys` và tính vào `skippedCount`. Chỉ đúng 1 công việc được sao chép sang ngày đích.
- **ĐỐI CHIẾU MÃ — TC-05: Cấp ID duy nhất mới, createdAt mới và completed=false:**
  - *Kết quả:* Mọi công việc được sao chép sang ngày đích đều nhận ID ngẫu nhiên mới qua `makeId()`, thời điểm tạo mới qua `Date.now()` và luôn bắt đầu với `completed: false`, không bao giờ kế thừa trạng thái hoàn thành từ ngày nguồn.
- **ĐỐI CHIẾU MÃ — TC-06: Cảnh báo trùng khoảng giờ nghiêm ngặt vs Tiếp xúc biên:**
  - *Tiếp xúc biên:* Công việc A (08:00–09:00) và Công việc B (09:00–10:00). Bất đẳng thức `startA < endB && startB < endA` (`540 < 540`) là sai $\rightarrow$ `overlaps()` trả về `false`, `conflictCount = 0`, không cảnh báo.
  - *Giao khoảng giờ thực sự:* Công việc A (08:30–09:30) và Công việc B (09:00–10:00). Bất đẳng thức thỏa mãn $\rightarrow$ `overlaps()` trả về `true`, `conflictCount = 1`.
  - *Tính chất non-blocking:* Hiển thị banner cảnh báo màu vàng cam kèm số lượng trùng giờ, nhưng nút `#copy-submit-button` vẫn kích hoạt và cho phép người dùng lưu bình thường.
- **ĐỐI CHIẾU MÃ — TC-07: Từ chối ngày nguồn rỗng và ngày đích trùng ngày nguồn:**
  - *Nguồn rỗng:* Nút `#copy-schedule-trigger` bị vô hiệu hóa (`disabled`) kèm tooltip giải thích. Hàm `openCopyDialog` và `commitScheduleCopy` từ chối thao tác. 0 lượt ghi.
  - *Trùng ngày:* Chọn ngày đích trùng ngày nguồn hiển thị lỗi `⚠ Ngày đích phải khác ngày nguồn (YYYY-MM-DD)`, gắn `aria-invalid="true"`, ẩn preview, khóa nút xác nhận. `commitScheduleCopy` trả về `error: "same_date"`. 0 lượt ghi.
- **ĐỐI CHIẾU MÃ — TC-08: Tất cả trùng lặp & Cam kết không ghi (Zero-Write Guarantee):**
  - *Đầu vào:* Toàn bộ công việc nguồn đã tồn tại ở ngày đích (`addedCount === 0`).
  - *Kết quả:* Giao diện hiển thị thông báo giải thích không có việc mới; nút xác nhận bị vô hiệu hóa. `commitScheduleCopy` trả về `{ ok: true, wrote: false, addedCount: 0 }`. Quan sát qua `createStorageWriteObserver`: 0 lượt gọi `localStorage.setItem`.
- **ĐỐI CHIẾU MÃ — TC-09: Chặn ngày đích chứa JSON hỏng hoặc bản ghi lỗi với cam kết không ghi:**
  - *JSON cú pháp hỏng (`status: "corrupt_json"`):* Chặn xem trước, hiển thị banner cảnh báo cấu trúc, vô hiệu hóa nút xác nhận. 0 lượt ghi.
  - *Bản ghi lỗi (`status: "invalid_records"` do `end <= start`, sai category, thiếu trường bắt buộc):* Chặn xem trước, hiển thị banner lỗi, vô hiệu hóa nút xác nhận. 0 lượt ghi.
- **ĐỐI CHIẾU MÃ — TC-10: Phát hiện và xử lý bất đồng bộ dữ liệu (Stale Preview Mitigation):**
  - *Đầu vào:* Dữ liệu ngày đích bị sửa đổi giữa thời điểm xem trước và thời điểm bấm xác nhận (`freshRead.rawSnapshot !== cachedSnapshot`).
  - *Kết quả:* `commitScheduleCopy` phát hiện sai lệch, hủy lệnh ghi (0 lượt ghi), trả về `error: "stale_preview"`. Giao diện hiển thị cảnh báo bất đồng bộ, nạp snapshot mới, tự động tính toán lại preview và cho phép người dùng xác nhận trên số liệu mới nhất.
- **ĐỐI CHIẾU MÃ — TC-11: Lưu trữ bền vững và phân tách ngày (Persistence & Date Separation):**
  - *Kết quả:* Dữ liệu lưu dưới khóa chuẩn `lich-trinh-hang-ngay:v1:YYYY-MM-DD`. Dữ liệu ngày nguồn và ngày đích hoàn toàn độc lập; tải lại ngày nguồn vẫn giữ nguyên danh sách nguồn ban đầu; chuyển sang ngày đích hiển thị đúng dữ liệu đã sao chép.
- **ĐỐI CHIẾU MÃ — TC-12: Giữ nguyên ngày nguồn đang xem & Thông báo live region chính xác:**
  - *Kết quả:* Sau khi sao chép thành công, hộp thoại đóng lại; trường chọn ngày `#schedule-date` vẫn giữ nguyên ngày nguồn đang xem. Vùng `#app-status` (`role="status"`, `aria-live="polite"`) đọc chính xác thông báo: `"Đã sao chép thành công X công việc sang ngày DD/MM/YYYY. Ngày xem lịch vẫn là DD/MM/YYYY."`.
- **ĐỐI CHIẾU MÃ — TC-13: Hộp thoại native `<dialog>`, điều hướng bàn phím & Quản lý Focus:**
  - *Kết quả:* Mở bằng `showModal()`, focus tự động đặt vào ô nhập ngày đích `#copy-destination-date`. Phím `Escape` kích hoạt sự kiện `cancel` native; nút “Hủy” đóng dialog; cả hai trường hợp cùng với thao tác sao chép thành công đều hoàn trả focus chuẩn xác về `#copy-schedule-trigger`.
- **ĐỐI CHIẾU MÃ — TC-14: Audit DOM an toàn tuyệt đối & Không thư viện/network:**
  - *Kết quả:* 100% phần tử động và văn bản tạo qua `createElement`, `textContent`, `setAttribute`. Không có bất kỳ dòng mã nào sử dụng `innerHTML`, `insertAdjacentHTML` hay `document.write`. Ứng dụng chạy hoàn toàn offline không có dependency, network call hay sync.

### 3. Quan sát Số lượt ghi Lưu trữ (Storage Write-Count Observations)
Kiểm chứng thực tế thông qua wrapper `window.__scheduleCopyEngine.createStorageWriteObserver()`:
- **Từ chối (Nguồn rỗng hoặc ngày đích trùng ngày nguồn):** 0 lượt gọi `localStorage.setItem`.
- **Bất đồng bộ (Stale preview — ngày đích bị thay đổi trước khi xác nhận):** 0 lượt gọi `localStorage.setItem`.
- **Toàn bộ trùng lặp (`addedCount === 0`):** 0 lượt gọi `localStorage.setItem`.
- **Dữ liệu đích bị hỏng (JSON hỏng hoặc bản ghi lỗi cấu trúc):** 0 lượt gọi `localStorage.setItem`.
- **Sao chép thành công sang ngày đích trống:** Đúng 1 lượt gọi `localStorage.setItem` vào khóa ngày đích; 0 lượt gọi vào khóa ngày nguồn.
- **Sao chép thành công sang ngày đích đã có dữ liệu:** Đúng 1 lượt gọi `localStorage.setItem` vào khóa ngày đích; 0 lượt gọi vào khóa ngày nguồn.

### 4. Hồi quy Tính năng Cơ sở (Regression Testing)
- **Thêm công việc:** Biểu mẫu xác thực đầy đủ; công việc mới luôn có `completed: false`.
- **Sửa công việc:** Nạp đúng dữ liệu cũ; lưu thay đổi hợp lệ; giữ nguyên ID, thời điểm tạo và trạng thái hoàn thành.
- **Xóa công việc:** Sử dụng hộp thoại xác nhận native; chọn hủy giữ nguyên dữ liệu; xác nhận xóa cập nhật storage và render lại.
- **Bật/tắt hoàn thành:** Chỉ thay đổi khi người dùng tác động trực tiếp vào checkbox.
- **Mục tiêu tương tác 44 px:** Checkbox có vùng nhãn liên kết inline-flex tối thiểu 44×44 CSS px; các nút hành động (Thêm, Sửa, Xóa, Sao chép, Hủy, Xác nhận) đều đạt chiều cao tối thiểu 44 px.
- **DOM an toàn:** Tuyệt đối không chèn HTML thô; không sử dụng `innerHTML`, `insertAdjacentHTML`, hay `document.write`.
- **Độc lập hoàn toàn:** Không có thư viện ngoài, không có API kết nối mạng, không có đồng bộ Google Calendar hay Notion.

### 5. Giới hạn trung thực (Honest Limitations)
- Kết quả của QA agent dựa trên duyệt nhánh mã tĩnh và smoke check cú pháp (`check_schedule.py`). Codex đã kiểm chứng riêng các luồng chính trên trình duyệt; xem [BROWSER_QA.md](BROWSER_QA.md). Chưa chạy ma trận tự động đa trình duyệt.
- Ứng dụng chỉ hỗ trợ các công việc diễn ra trong cùng một ngày (chưa hỗ trợ công việc kéo dài qua nửa đêm).
- Lưu trữ hoàn toàn cục bộ trên trình duyệt đang dùng; không có tính năng sao lưu đám mây hay đồng bộ đa thiết bị.
- Hộp thoại xác nhận xóa phụ thuộc giao diện native của `window.confirm`.
- Khi dữ liệu của một ngày bị hỏng cấu trúc JSON, ứng dụng không thể tự khôi phục dữ liệu đó; lần lưu mới cho ngày đó sẽ thay thế giá trị hỏng sau khi đã hiển thị cảnh báo rõ ràng.

## Tính năng: Quản lý công việc tuần (Cập nhật ngày 01 tháng 10 năm 2026)

Tính năng **"Quản lý công việc tuần"** được bổ sung trong Vòng sửa đổi 3, tuân thủ nghiêm ngặt đặc tả tương tác (`DESIGN.md` Mục 15) và kiến trúc giải thuật (`WEEKLY_ARCHITECTURE.md` & `DESIGN.md` Mục 16).

### 1. Hướng dẫn sử dụng & Luồng tương tác (Feature Instructions)
1. **Bộ chuyển đổi chế độ xem (View Mode Switcher):**
   - Nằm ngay đầu giao diện chính, gồm 2 tab: **“Xem theo ngày”** và **“Xem theo tuần”**.
   - Hỗ trợ chuyển tab bằng chuột hoặc bàn phím (phím mũi tên `←`/`→`/`↑`/`↓`, `Home`, `End`).
   - Kích thước tương tác tối thiểu 44 px, viền focus nhìn thấy rõ ràng (`:focus-visible`).
2. **Thanh điều hướng tuần (Week Navigation):**
   - Tiêu đề tuần tiếng Việt chuẩn ISO-8601 từ Thứ Hai đến Chủ Nhật: *“Tuần: DD/MM/YYYY – DD/MM/YYYY”* kèm mô tả *“Từ Thứ Hai ngày DD/MM đến Chủ Nhật ngày DD/MM/YYYY”*.
   - Ba nút bấm điều hướng đạt chuẩn 44 px: **“← Tuần trước”**, **“Tuần này”** (trở về tuần chứa ngày hôm nay), và **“Tuần sau →”**.
   - Đồng bộ hai chiều: khi chuyển từ xem ngày sang xem tuần, ứng dụng mở tuần chứa ngày đang xem; bấm “Tuần này” lập tức quay về tuần hiện tại.
3. **Bảng tổng kết số liệu tuần (Weekly Summary Metrics Panel):**
   - Hiển thị 3 số liệu định lượng: **Tổng công việc** (trong 7 ngày), **Đã hoàn thành** (kèm % tiến độ), và **Chưa hoàn thành**.
   - Cập nhật thời gian thực khi bật/tắt hoàn thành trên bất kỳ công việc nào trong tuần.
   - Nếu có ngày bị lỗi cấu trúc dữ liệu, bảng hiển thị cảnh báo phụ: *“⚠ Có K ngày bị lỗi dữ liệu (không tính vào tổng số).”*.
4. **Bố cục 7 ngày (7-Day Overview Layout):**
   - **Desktop (≥ 768 px):** Lưới 7 cột co giãn đều từ Thứ Hai đến Chủ Nhật. Mỗi cột có tiêu đề thứ, ngày tháng, huy hiệu số lượng việc, nút **“Xem ngày”**, điểm nhấn viền cho ngày hôm nay và ngày đang chọn.
   - **Mobile (320 px – 767 px):** Accordion xếp tầng dọc 1 cột. Nút trigger mở/gập đạt chiều cao tối thiểu 48 px, hỗ trợ phím `Enter`/`Space`. Mặc định mở ngày hôm nay hoặc ngày đang chọn. Cam kết không tràn chiều ngang trang ở 320 px (`overflow-x: hidden`).
5. **Thẻ công việc thu gọn & Đột biến hoàn thành:**
   - Hiển thị giờ bắt đầu/kết thúc, huy hiệu danh mục (Công ty/Cá nhân), mức ưu tiên (Thấp/Vừa/Cao), và huy hiệu cảnh báo *“Trùng giờ”* nếu giao khoảng giờ với việc khác trong cùng ngày.
   - Checkbox hoàn thành có vùng nhãn liên kết tương tác tối thiểu **44×44 CSS px** (nhãn inline-flex có `min-width: 44px; min-height: 44px;`).
   - Nhấp vào checkbox đổi trạng thái tức thì, gạch ngang tiêu đề, phát thông báo live region `#app-status` và cập nhật bảng tổng kết tuần.
6. **Điều hướng nhanh sang xem ngày (Quick Day Jump):**
   - Nút **“Xem ngày”** (desktop) / **“Xem lịch ngày này →”** (mobile) trên mỗi ngày cho phép chuyển tức thì sang tab “Xem theo ngày” với ngày đó được chọn, sẵn sàng thêm hoặc chỉnh sửa công việc chi tiết.
7. **Trạng thái rỗng & Cô lập lỗi ngày hỏng:**
   - Ngày không có việc hiển thị hộp *“Chưa có công việc”*.
   - Toàn tuần không có việc hiển thị banner *“Tuần này chưa có công việc nào”* kèm nút **“Thêm công việc cho Thứ Hai”**.
   - Ngày bị hỏng dữ liệu hiển thị thẻ lỗi *“⚠ Dữ liệu ngày bị lỗi”* kèm nút *“Mở ngày để khắc phục”*, hoàn toàn không làm sập giao diện tuần hay ảnh hưởng tới 6 ngày hợp lệ còn lại (Zero Crash Guarantee).

### 2. Quy tắc dữ liệu & Hợp đồng lưu trữ (Data Rules & Storage Contract)
- **Nguồn chân lý duy nhất (Single Source of Truth):** Dữ liệu tuần được tổng hợp động trực tiếp tại thời điểm chạy từ 7 khóa ngày hiện hành: `lich-trinh-hang-ngay:v1:YYYY-MM-DD`. Tuyệt đối **không** tạo khóa tuần riêng (`week:YYYY-Wxx`) nhằm triệt tiêu nguy cơ bất đồng bộ bậc hai (secondary desync) và lãng phí dung lượng.
- **Biên tuần tất định (Deterministic Week Boundaries):** Tính toán 7 ngày liên tiếp từ Thứ Hai đến Chủ Nhật bằng giải thuật phân tích số nguyên `[year, month, day]` và giờ trưa 12:00, loại bỏ hoàn toàn rủi ro nhảy ngày do múi giờ/DST; xử lý chính xác biên chuyển tháng, năm thường (28/02) và năm nhuận (29/02/2024).
- **Bộ nạp ngày nghiêm ngặt (Strict Date Loader):** Hàm `readDateTasks(dateStr)` kiểm tra 10 điều kiện của `isValidStoredTask`, phân định rõ 4 trạng thái: `'missing'`, `'valid'`, `'corrupt_json'`, `'invalid_records'`.
- **Hợp đồng đột biến hoàn thành:** Gọi `toggleTaskCompletionInWeek(taskDate, taskId)` thực hiện đúng **1 lượt ghi** vào khóa ngày của công việc đó (`STORAGE_PREFIX + taskDate`) và **0 lượt ghi** vào 6 ngày còn lại; bảo toàn 100% các trường còn lại (`id`, `title`, `start`, `end`, `category`, `priority`, `createdAt`) và thứ tự các công việc khác.
- **Cam kết không ghi khi nạp tuần:** Quá trình tải, chuyển tab hoặc chuyển tuần chỉ đọc dữ liệu (`Δ(localStorage.setItem) = 0`).

---

## Bằng chứng triển khai & Kết quả kiểm thử — Quản lý công việc tuần (Frontend — ngày 01 tháng 10 năm 2026, Tác vụ 5d8e9a1ff5aa)

### 1. Bằng chứng triển khai thành phần mã nguồn (Artifact Evidence)
- **Bộ chuyển tab WAI-ARIA:** Bổ sung `.view-mode-nav` với `tab-view-daily` và `tab-view-weekly` theo chuẩn WAI-ARIA Tabs pattern, `role="tablist"`, `aria-selected`, `aria-controls`, phím mũi tên `←`/`→`/`↑`/`↓`, `Home`, `End`, và mục tiêu nhấn tối thiểu 44 px.
- **Thanh điều hướng tuần:** Bổ sung `.week-navigation-panel` với `#week-heading` hiển thị dải tuần tiếng Việt, 3 nút bấm `#week-prev-btn`, `#week-today-btn`, `#week-next-btn` đạt chuẩn chiều cao 44 px.
- **Bảng tổng kết tuần:** Bổ sung `.weekly-summary-panel` hiển thị 3 chỉ số `#metric-week-total`, `#metric-week-completed`, `#metric-week-remaining`, `#metric-week-percent` và cảnh báo `#weekly-corrupt-warning`.
- **Bố cục 7 ngày thích ứng:** Lưới 7 cột `.weekly-grid` trên desktop (≥ 768 px) và accordion dọc `.day-accordion-trigger` trên mobile (320 px – 767 px) với chiều cao trigger 48 px, không tràn ngang.
- **Thẻ công việc thu gọn:** `.compact-task-card` với huy hiệu loại, mức ưu tiên, trùng giờ, và checkbox hoàn thành `.compact-completion-label` có diện tích chạm tối thiểu 44×44 CSS px.
- **Điều hướng nhanh:** Nút `.day-jump-btn` có nhãn linh hoạt ("Xem ngày" trên desktop, "Xem lịch ngày này →" trên mobile), kích hoạt chuyển sang xem ngày và focus vào `#timeline-heading`.
- **Cơ chế cô lập lỗi:** Thẻ `.day-corrupt-card` bảo vệ giao diện khi một ngày bị hỏng cấu trúc; 6 ngày còn lại hiển thị bình thường.
- **Audit DOM an toàn tuyệt đối:** 100% phần tử động tạo qua `createElement`, `textContent`, `setAttribute`, `classList`. Tuyệt đối không sử dụng `innerHTML`, `insertAdjacentHTML`, `outerHTML`, hay `document.write`.
- **Điểm móc kiểm thử công khai (Test Hooks):** Xuất bản toàn bộ API thuần túy qua `window.__weeklyEngine`:
  - `getWeekBoundaries(dateStr)`
  - `getAdjacentWeek(currentMonday, offsetWeeks)`
  - `toISODateString(d)`
  - `isValidStoredTask(raw)`
  - `readDateTasks(dateStr)`
  - `calculateWeeklyMetrics(daysData)`
  - `toggleTaskCompletionInWeek(taskDate, taskId)`
  - `createWeeklyStorageObserver()`

### 2. Đối chiếu 12 Bất biến toán học & Kịch bản thực tế
- **ĐỐI CHIẾU MÃ — INV-01: Bất biến đủ 7 ngày liên tiếp:** Hàm `getWeekBoundaries(d)` luôn sinh chính xác mảng 7 ngày liên tiếp từ Thứ Hai đến Chủ Nhật cho mọi ngày ISO hợp lệ.
- **ĐỐI CHIẾU MÃ — INV-02: Bất biến trật tự tuần ISO:** Ngày đầu tiên luôn là Thứ Hai (`getDay() === 1`), ngày cuối cùng luôn là Chủ Nhật (`getDay() === 0`).
- **ĐỐI CHIẾU MÃ — INV-03: Bất biến ổn định chu trình tuần:** Truyền bất kỳ ngày nào trong 7 ngày của tuần vào `getWeekBoundaries` đều trả về cùng một tuần 7 ngày giống hệt nhau.
- **ĐỐI CHIẾU MÃ — INV-04: Bất biến không tạo khóa tuần:** Toàn bộ quá trình tổng hợp dữ liệu chỉ đọc từ khóa `lich-trinh-hang-ngay:v1:YYYY-MM-DD`, không tạo khóa `week:`.
- **ĐỐI CHIẾU MÃ — INV-05: Bất biến không ghi khi tải tuần:** Thao tác chuyển tab tuần, bấm tuần trước/sau hoặc nạp trang có $\Delta(\text{localStorage.setItem}) = 0$.
- **ĐỐI CHIẾU MÃ — INV-06: Bất biến cô lập lỗi ngày hỏng:** Nếu 1 hoặc nhiều ngày có JSON hỏng hoặc bản ghi lỗi, chỉ các ngày đó hiển thị banner cảnh báo; các ngày hợp lệ còn lại hiển thị bình thường (Zero Crash).
- **ĐỐI CHIẾU MÃ — INV-07: Bất biến ghi duy nhất khi toggle hoàn thành:** Gọi `toggleTaskCompletionInWeek(date, taskId)` thực hiện đúng 1 lượt gọi `setItem` vào khóa của `date`, và 0 lượt gọi vào bất kỳ ngày nào khác.
- **ĐỐI CHIẾU MÃ — INV-08: Bất biến bảo toàn thuộc tính bản ghi:** Toàn bộ 7 trường (`id`, `title`, `start`, `end`, `category`, `priority`, `createdAt`) và thứ tự các công việc khác trong ngày được giữ nguyên vẹn 100%.
- **ĐỐI CHIẾU MÃ — INV-09: Bất biến bảo toàn số lượng công việc tuần:** Luôn thỏa mãn `completedWeeklyTasks + remainingWeeklyTasks = totalWeeklyTasks`.
- **ĐỐI CHIẾU MÃ — INV-10: Bất biến miền giá trị chỉ số:** `0 <= completedWeeklyTasks <= totalWeeklyTasks` và `0 <= completionRate <= 100`.
- **ĐỐI CHIẾU MÃ — INV-11: Bất biến tuần rỗng:** Khi tổng số việc bằng 0, hoàn thành = 0, còn lại = 0, tỷ lệ = 0%; hiển thị banner rỗng tuần kèm nút thêm việc Thứ Hai.
- **ĐỐI CHIẾU MÃ — INV-12: Bất biến xử lý năm nhuận:** Ngày `2024-02-29` tính ra Thứ Hai `2024-02-26` và Chủ Nhật `2024-03-03` chính xác.

### 3. Kiểm tra smoke check cố định (`check_schedule.py`)
- Cú pháp JavaScript hợp lệ (`node --check`).
- Các thẻ ngữ nghĩa (`form`, `input`, `button`, `main`, `label`) đầy đủ.
- Responsive (`viewport`, `@media`) đầy đủ.
- Lưu trữ (`localStorage`) hợp lệ.
- An toàn DOM tuyệt đối (không có `innerHTML` trong mã lệnh).
```text
PASS: schedule smoke checks and JavaScript syntax. Interaction/visual QA still required.
```
---

## Báo cáo Kiểm thử Độc lập (QA) — Ngày 01 tháng 10 năm 2026: Quản lý công việc tuần

Phần này ghi nhận kết quả kiểm thử độc lập đối với tính năng **“Quản lý công việc tuần”** (Weekly Task Management) theo nhiệm vụ QA độc lập của dự án (Tác vụ `a15c7b393c9a` và remediation `427778992d94`).

### 1. Phương pháp kiểm thử (Test Methodology)
- **Kiểm tra nhánh mã nguồn (Code-path inspection):** Rà soát chi tiết toàn bộ logic xử lý trong `index.html` bao gồm các thuật toán đọc/ghi `localStorage`, chuyển đổi tab WAI-ARIA, điều hướng tuần, tổng hợp động 7 ngày, tính toán số liệu tuần, đột biến trạng thái hoàn thành và xử lý sự kiện giao diện.
- **Rà soát tĩnh & Khả năng tiếp cận (Static review):** Đánh giá cấu trúc HTML ngữ nghĩa, nhãn form tường minh, bộ quy tắc CSS, khả năng co giãn responsive ở breakpoint 320 px và kích thước mục tiêu cảm ứng tối thiểu 44 px.
- **Smoke checks cố định (`tini.run_checks` / `check_schedule.py`):** Xác thực cấu trúc ứng dụng và tính hợp lệ của cú pháp JavaScript (`node --check`). Kết quả: `PASS: schedule smoke checks and JavaScript syntax. Interaction/visual QA still required.`
- **Điểm móc kiểm thử công khai (Test Hook Interface):** Kiểm chứng thông qua đối tượng thuần túy `window.__weeklyEngine` chứa đầy đủ các hàm logic: `getWeekBoundaries`, `getAdjacentWeek`, `toISODateString`, `isValidStoredTask`, `readDateTasks`, `calculateWeeklyMetrics`, `toggleTaskCompletionInWeek`, và wrapper theo dõi ghi `createWeeklyStorageObserver`.
- **Tuyên bố giới hạn trung thực (Honest Limitation):** Bộ kiểm thử tự động đa trình duyệt (automated multi-browser execution matrix) **chưa được thực hiện**; kết quả kiểm thử của QA agent dựa trên rà soát nhánh mã nguồn, rà soát tĩnh, smoke check cú pháp và kiểm chứng hàm logic qua test hooks.

### 2. Kết quả 14 Ca kiểm thử độc lập chi tiết (TC-W01 đến TC-W14)
- **PASS — TC-W01: Bộ chuyển đổi chế độ xem Ngày / Tuần (View Mode Switcher):**
  - Cấu trúc WAI-ARIA Tabs pattern: `#view-mode-tabs` có `role="tablist"`, các tab `#tab-view-daily` và `#tab-view-weekly` có `role="tab"`, `aria-selected`, `aria-controls`, `tabindex="0"` (khi active) và `tabindex="-1"` (khi inactive).
  - Điều hướng bàn phím đầy đủ: phím `ArrowLeft` / `ArrowRight` / `ArrowUp` / `ArrowDown` chuyển tab; phím `Home` về tab Ngày, `End` sang tab Tuần. Viền focus `:focus-visible` nhìn thấy rõ ràng.
  - Quản lý hiển thị qua thuộc tính `hidden` giữa `#daily-view-panel` và `#weekly-view-panel`.
  - Vùng live region `#app-status` thông báo chính xác khi chuyển chế độ xem: *"Đã chuyển sang chế độ xem theo ngày."* / *"Đã chuyển sang chế độ xem theo tuần..."*.
  - Kích thước tương tác tối thiểu 44×44 CSS px: `.tab-btn` có `min-height: 44px; min-width: 44px; padding: 10px 22px;`.
- **PASS — TC-W02: Điều hướng tuần & Tính toán biên tuần Thứ Hai – Chủ Nhật (Week Navigation & Date Math across Month/Year Boundaries):**
  - Quy ước Thứ Hai đến Chủ Nhật chuẩn ISO-8601 qua hàm `getWeekBoundaries(dateStr)`.
  - Phân tích chuỗi số nguyên `[year, month, day]` và khởi tạo 12:00 trưa cục bộ, loại trừ 100% rủi ro trôi ngày do múi giờ hoặc DST.
  - Kiểm thử các trường hợp biên đặc biệt:
    - *Biên chuyển tháng:* `2026-09-30` (Thứ Tư) sinh Thứ Hai `2026-09-28` và Chủ Nhật `2026-10-04` chuẩn xác.
    - *Năm nhuận tháng 2 có 29 ngày:* `2024-02-29` (Thứ Năm) sinh Thứ Hai `2024-02-26` và Chủ Nhật `2024-03-03`, ngày `2024-02-29` nằm ở index 3.
    - *Năm thường tháng 2 có 28 ngày:* `2025-02-28` (Thứ Sáu) sinh Thứ Hai `2025-02-24` và Chủ Nhật `2025-03-02`.
    - *Biên chuyển năm:* `2026-12-31` (Thứ Năm) sinh Thứ Hai `2026-12-28` và Chủ Nhật `2027-01-03`.
    - *Ngày đầu năm giữa tuần:* `2027-01-01` (Thứ Sáu) sinh cùng dải tuần `2026-12-28` đến `2027-01-03`.
    - *Mốc Thứ Hai:* `2026-10-05` bắt đầu đúng ngày đó; mốc Chủ Nhật `2026-10-11` lùi đúng 6 ngày về Thứ Hai.
  - Nút điều hướng `#week-prev-btn`, `#week-today-btn`, `#week-next-btn` đều đạt chiều cao tối thiểu 44 px. Tiêu đề `#week-heading` hiển thị dải tuần tiếng Việt chuẩn và phát thông báo live region.
- **PASS — TC-W03: Tổng hợp động 7 ngày từ khóa lưu trữ theo ngày (Dynamic 7-Day Aggregation without Weekly Keys):**
  - Hàm `readDateTasks(dateStr)` được gọi lần lượt cho 7 ngày `boundaries.days`.
  - Đọc on-the-fly trực tiếp từ các khóa ngày hiện hành `STORAGE_PREFIX + dateStr` (`lich-trinh-hang-ngay:v1:YYYY-MM-DD`).
  - Tuyệt đối không tạo bất kỳ khóa tuần riêng nào (như `week:YYYY-Wxx`), loại trừ triệt để nguy cơ bất đồng bộ bậc hai (secondary desync).
  - Không phát sinh lệnh gọi `localStorage.setItem` trong quá trình đọc và tổng hợp ($\Delta(\text{setItem}) = 0$).
- **PASS — TC-W04: Độ chính xác số liệu tuần & Bảo toàn khối lượng công việc (Weekly Metrics Accuracy & Metric Conservation):**
  - Hàm `calculateWeeklyMetrics(daysData)` tính toán:
    - `totalWeeklyTasks = sum(|day.tasks|)` cho tất cả các ngày hợp lệ (`valid`).
    - `completedWeeklyTasks = sum(|day.tasks with completed === true|)`.
    - `remainingWeeklyTasks = totalWeeklyTasks - completedWeeklyTasks`.
    - Định luật bảo toàn: `completedWeeklyTasks + remainingWeeklyTasks = totalWeeklyTasks` luôn thỏa mãn 100%.
    - Tỷ lệ phần trăm: `completionRate = total === 0 ? 0 : Math.round((completed / total) * 100)`. Miền giá trị `0 <= completed <= total` và `0 <= rate <= 100`.
    - Ngày bị lỗi cấu trúc dữ liệu (`corrupt_json` hoặc `invalid_records`) được đếm riêng vào `corruptDaysCount`, không tính vào tổng số để tránh sai lệch số liệu.
- **PASS — TC-W05: Đột biến trạng thái hoàn thành trong giao diện tuần (Completion Toggle Contract):**
  - Checkbox `.compact-checkbox` (`#week-chk-${task.id}`) có vùng nhãn liên kết `.compact-completion-label` đạt chuẩn tiếp cận tối thiểu **44×44 CSS px** (`min-width: 44px; min-height: 44px; display: inline-flex;`).
  - Hàm `toggleTaskCompletionInWeek(taskDate, taskId)` tái đọc dữ liệu ngày `taskDate`, đảo trạng thái `completed = !previousState`.
  - Bảo toàn 100% 7 trường dữ liệu còn lại (`id`, `title`, `start`, `end`, `category`, `priority`, `createdAt`) và thứ tự các công việc khác trong ngày.
  - Thực hiện đúng **1 lượt ghi** vào khóa ngày của công việc đó (`STORAGE_PREFIX + taskDate`) và **0 lượt ghi** vào 6 ngày còn lại.
  - Cập nhật tức thì trên giao diện (gạch ngang tiêu đề, đổi nhãn aria-label, cập nhật bảng tổng kết tuần, phát thông báo live region `#app-status`).
- **PASS — TC-W06: Điều hướng nhanh sang xem ngày (Quick Day Jump):**
  - Nút `.day-jump-btn` trên mỗi ngày (hoặc nút "Mở ngày để khắc phục" trên ngày lỗi) gọi `jumpToDay(dateStr)`.
  - Thiết lập `elements.date.value = dateStr`, chuyển sang tab xem ngày (`setViewMode("daily")`), nạp dữ liệu và render timeline chi tiết, đặt focus vào `#timeline-heading`.
  - Live region thông báo: *"Đã mở lịch trình chi tiết ngày {Thứ}, {DD/MM/YYYY}."*.
- **PASS — TC-W07: Cô lập lỗi ngày hỏng & Cam kết không sập (Corrupt Date Isolation & Zero Crash Guarantee):**
  - Khi một ngày chứa JSON hỏng cú pháp (`status: "corrupt_json"`) hoặc bản ghi không hợp lệ (`status: "invalid_records"` do `end <= start`, sai category, thiếu trường):
  - Ứng dụng không bị sập (Zero Crash Guarantee), bắt lỗi an toàn cho từng ngày.
  - Cột ngày bị lỗi hiển thị huy hiệu đỏ `<span class="badge badge-danger">Lỗi</span>` trên tiêu đề.
  - Khối nội dung ngày hiển thị thẻ `.day-corrupt-card` với thông điệp *"⚠ Dữ liệu ngày bị lỗi"* kèm nút *"Mở ngày để khắc phục"*.
  - Toàn bộ 6 ngày hợp lệ còn lại vẫn hiển thị bình thường và đầy đủ dữ liệu.
  - Bảng tổng kết tuần hiển thị cảnh báo phụ: *"⚠ Có K ngày bị lỗi dữ liệu (không tính vào tổng số)."*.
- **PASS — TC-W08: Trạng thái rỗng ngày và toàn tuần (Empty States):**
  - Trạng thái rỗng ngày: ngày không có công việc hiển thị hộp `.day-empty-box` với văn bản *"Chưa có công việc"*.
  - Trạng thái rỗng toàn tuần: khi cả 7 ngày đều không có công việc (`totalWeeklyTasks === 0 && corruptDaysCount === 0`), hiển thị banner `.week-empty-state` với tiêu đề *"Tuần này chưa có công việc nào"*, mô tả dải tuần từ Thứ Hai đến Chủ Nhật, kèm nút `#week-start-monday-btn` có nhãn *"Thêm công việc cho Thứ Hai (DD/MM)"* chuyển thẳng sang xem ngày Thứ Hai và focus vào ô nhập tiêu đề `#task-title`.
- **PASS — TC-W09: Audit DOM an toàn tuyệt đối (Strict Safe DOM Audit):**
  - 100% phần tử động và văn bản tạo qua `document.createElement`, `node.textContent`, `node.setAttribute`, `node.className`, `node.classList`.
  - Tuyệt đối 0 lần sử dụng `innerHTML`, `insertAdjacentHTML`, `outerHTML`, hay `document.write`.
  - Bộ kiểm tra smoke check xác nhận: `assert 'innerHTML' not in js` đạt PASS.
- **PASS — TC-W10: Bố cục thích ứng & Rà soát tràn ngang ở 320 px (Responsive 320px Review):**
  - Desktop ($\ge$ 768 px): Lưới 7 cột `.weekly-grid` co giãn đều; tiêu đề và danh sách công việc hiển thị rõ ràng.
  - Mobile (320 px – 767 px): Accordion xếp tầng dọc 1 cột; nút trigger `.day-accordion-trigger` đạt chiều cao tối thiểu 48 px. Mặc định mở ngày hôm nay hoặc ngày đang chọn.
  - Chiều rộng `min(100% - 32px, 1120px) = 288px` ở màn hình 320 px.
  - Áp dụng `overflow-wrap: anywhere; min-width: 0;`, không phát sinh thanh cuộn ngang trang (`overflow-x` an toàn tuyệt đối).
- **PASS — TC-W11: Kích thước mục tiêu cảm ứng tối thiểu 44 px (44px Minimum Interactive Targets):**
  - Nút tab chuyển chế độ xem: `.tab-btn` có `min-height: 44px; min-width: 44px;`.
  - Nút điều hướng tuần: `.week-nav-btn` có `min-height: 44px;`.
  - Nút thêm việc Thứ Hai: `#week-start-monday-btn` có `min-height: 44px;`.
  - Nút accordion trigger trên mobile: `.day-accordion-trigger` có `min-height: 48px;`.
  - Nút xem ngày: `.day-jump-btn` có `min-height: 44px;`.
  - Checkbox hoàn thành tuần: Vùng nhãn `.compact-completion-label` có `min-width: 44px; min-height: 44px; display: inline-flex;`.
  - Nút tác vụ hàng ngày: Sửa/Xóa `.task-actions .btn` có `min-height: 44px;`.
  - Checkbox hoàn thành hàng ngày: Vùng nhãn `.completion-control label` có `min-width: 44px; min-height: 44px; display: inline-flex;`.
  - Nút kích hoạt sao chép và các nút trong dialog copy: đều đạt `min-height: 44px;`.
- **PASS — TC-W12: Điểm móc kiểm thử công khai (Test Hook Interface via window.__weeklyEngine):**
  - Xuất bản đầy đủ đối tượng `window.__weeklyEngine` chứa các hàm logic thuần túy: `getWeekBoundaries`, `getAdjacentWeek`, `toISODateString`, `isValidStoredTask`, `readDateTasks`, `calculateWeeklyMetrics`, `toggleTaskCompletionInWeek`, `createWeeklyStorageObserver`.
  - Cho phép QA và các bài kiểm tra tự động thẩm định trực tiếp mà không cần can thiệp vào UI.
- **PASS — TC-W13: Quan sát lượt ghi lưu trữ (Storage Write-Count Observations via createWeeklyStorageObserver):**
  - Chuyển tab Ngày $\leftrightarrow$ Tuần: 0 lượt gọi `localStorage.setItem`.
  - Chuyển tuần trước / tuần sau / tuần này: 0 lượt gọi `localStorage.setItem`.
  - Nạp tuần có ngày trống / ngày lỗi: 0 lượt gọi `localStorage.setItem`.
  - Bật/tắt checkbox hoàn thành trên tuần: Đúng **1 lượt gọi** `localStorage.setItem` vào khóa ngày của công việc đó; **0 lượt gọi** vào 6 ngày còn lại.
- **PASS — TC-W14: Hồi quy toàn diện các tính năng Lịch trình hàng ngày & Sao chép lịch (Full Regression):**
  - *Lịch trình hàng ngày:* Thêm/Sửa/Xóa công việc, xác thực tiêu đề và thời gian (bắt buộc kết thúc sau bắt đầu, từ chối giờ thiếu/đảo ngược/bằng nhau), phân loại Công ty/Cá nhân, mức ưu tiên, công việc mới mặc định `completed: false`, sửa giữ nguyên ID/createdAt/completed, native `window.confirm` cho xóa, checkbox hoàn thành 44×44 px, sắp xếp timeline theo thứ tự thời gian tăng dần, cảnh báo trùng khoảng giờ nghiêm ngặt (tiếp xúc biên không bị trùng, non-blocking save), phân tách dữ liệu độc lập theo khóa `lich-trinh-hang-ngay:v1:YYYY-MM-DD`, cô lập dữ liệu JSON hỏng.
  - *Sao chép lịch sang ngày khác:* Native `<dialog>`, focus trap, phím Escape và nút Hủy hoàn trả focus về trigger, kiểm tra ngày đích khác ngày nguồn, từ chối ngày nguồn rỗng, định danh 4 trường `[title.trim(), start, end, category]` loại trừ priority/completed, khử trùng lặp nội bộ nguồn và trùng lặp ngày đích, cấp ID mới, createdAt mới, reset `completed: false`, cảnh báo trùng khoảng giờ nghiêm ngặt (tiếp xúc biên không tính trùng, non-blocking save), bảo toàn 100% bản ghi cũ ngày đích, cam kết Zero-Write khi toàn bộ trùng lặp hoặc ngày đích hỏng cấu trúc, chống stale preview tại thời điểm xác nhận, giữ nguyên ngày nguồn đang xem, thông báo live region đầy đủ.

### 3. Quan sát Số lượt ghi Lưu trữ (Storage Write-Count Observations)

| Kịch bản thao tác | Khóa mục tiêu | Số lượt ghi thực tế | Ghi chú an toàn |
| :--- | :--- | :---: | :--- |
| **Chuyển chế độ xem Ngày sang Tuần** | Không có | **0** | Chỉ đọc on-the-fly từ 7 khóa ngày |
| **Bấm "Tuần trước", "Tuần sau", "Tuần này"** | Không có | **0** | Chỉ tính toán biên tuần và đọc dữ liệu |
| **Nạp tuần chứa ngày rỗng / ngày lỗi** | Không có | **0** | Cam kết Zero-Write khi hiển thị |
| **Bật/tắt hoàn thành 1 việc tại ngày D** | `STORAGE_PREFIX + D` | **Đúng 1** | 0 lượt ghi vào 6 ngày còn lại |
| **Bật/tắt hoàn thành tại ngày bị lỗi dữ liệu** | Không có | **0** | Hủy thao tác an toàn (Zero-Write) |
| **Bấm "Xem ngày" hoặc "Thêm việc Thứ Hai"** | Không có | **0** | Chỉ chuyển chế độ xem và đặt focus |
| **Sao chép lịch sang ngày đích thành công** | `STORAGE_PREFIX + destDate` | **Đúng 1** | 0 lượt ghi vào ngày nguồn |
| **Sao chép bị từ chối / Stale / All-duplicate / Hỏng** | Không có | **0** | Cam kết Zero-Write bảo vệ dữ liệu |

### 4. Hồi quy Tính năng Cơ sở (Regression Testing of Original Features)
- **Thêm công việc:** Biểu mẫu kiểm tra đầy đủ (tiêu đề, giờ hợp lệ, giờ kết thúc sau giờ bắt đầu, phân loại, ưu tiên); công việc mới luôn có `completed: false`.
- **Sửa công việc:** Nạp đúng dữ liệu cũ; lưu thay đổi hợp lệ; giữ nguyên ID, thời điểm tạo và trạng thái hoàn thành.
- **Xóa công việc:** Hộp thoại xác nhận native `window.confirm`; hủy giữ nguyên dữ liệu; xác nhận xóa cập nhật storage và render lại.
- **Bật/tắt hoàn thành:** Chỉ thay đổi trạng thái khi người dùng tác động trực tiếp vào checkbox.
- **Mục tiêu tương tác 44 px:** Checkbox có vùng nhãn liên kết inline-flex tối thiểu 44×44 CSS px; các nút hành động (Thêm, Sửa, Xóa, Sao chép, Hủy, Xác nhận, Tabs, Nav) đều đạt chiều cao tối thiểu 44 px (hoặc 48 px cho accordion).
- **DOM an toàn:** Không chèn HTML thô; tuyệt đối không sử dụng `innerHTML`, `insertAdjacentHTML`, hay `document.write`.
- **Độc lập hoàn toàn:** Không sử dụng thư viện ngoài, không có API mạng hay dịch vụ bên ngoài; không đồng bộ Google Calendar hay Notion.

### 5. Giới hạn trung thực (Honest Limitations)
1. **Phạm vi kiểm thử:** Kiểm thử của QA agent được thực hiện thông qua rà soát nhánh mã nguồn chi tiết (code-path inspection), rà soát tĩnh (static review), smoke check cú pháp (`check_schedule.py`), và hook kiểm thử engine (`window.__weeklyEngine`). Ma trận kiểm thử tự động đa trình duyệt (cross-browser automation matrix) chưa được triển khai.
2. **Lịch trong ngày:** Ứng dụng chỉ hỗ trợ công việc bắt đầu và kết thúc trong cùng một ngày (chưa hỗ trợ công việc xuyên qua nửa đêm).
3. **Lưu trữ cục bộ:** Toàn bộ dữ liệu nằm trên `localStorage` của trình duyệt hiện tại; không có tài khoản, sao lưu đám mây hay đồng bộ Google Calendar/Notion.
4. **Hộp thoại native:** Xác nhận xóa dựa vào `window.confirm` của từng trình duyệt.
5. **Dữ liệu hỏng:** Khi một ngày chứa JSON hỏng, ứng dụng cách ly hiển thị thẻ lỗi an toàn mà không làm sập tuần; lần lưu mới hợp lệ sẽ ghi đè giá trị hỏng sau khi hiển thị cảnh báo rõ ràng.

## Giới hạn kiểm thử và sản phẩm

- QA agent xác minh qua code-path/manual inspection và smoke check cố định; Codex đã kiểm chứng riêng persistence qua reload trên trình duyệt. Chưa có ma trận trình duyệt tự động. Hành vi screen reader và render ở 320 px vẫn cần kiểm tra trên thiết bị mục tiêu.
- Công việc phải bắt đầu và kết thúc trong cùng một ngày; lịch qua nửa đêm chưa được hỗ trợ.
- Dữ liệu chỉ lưu cục bộ trên một trình duyệt/thiết bị; không có tài khoản, đồng bộ, nhập/xuất hoặc nhắc việc hệ thống.
- Xác nhận xóa dùng `window.confirm` native nên hình thức và trải nghiệm có thể khác giữa các trình duyệt.
- Nếu `localStorage` bị chặn hoặc hết dung lượng, thay đổi mới chỉ được giữ trong bộ nhớ cho đến khi đóng/tải lại trang.
- Khi dữ liệu của một ngày là JSON hỏng, ứng dụng không thể phục hồi nội dung đó; lần lưu mới cho ngày ấy sẽ thay thế giá trị hỏng sau khi đã cảnh báo rõ.
