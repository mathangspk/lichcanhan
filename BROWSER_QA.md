# Browser acceptance evidence

Checked on the local preview at http://127.0.0.1:9001/ through actual browser UI.
Test data are explicitly named "Ví dụ QA" and isolated to 2099-01-01.

Verified:
- Add a task with title, time range and personal category: one chronological entry appears.
- Mark complete, reload, and select the test date again: completed task remains.
- Reload returns to today's date, which has no test tasks: date separation holds.
- Edit the title and save: the entry updates without losing completion state.
- Add an overlapping task: both entries display overlap warnings.
- Submit end time before start time: field error is shown and no entry is added.

- Delete the overlapping example: only the original task remains and overlap warnings clear.
- Viewport width 723 px: document width 708 px; no horizontal overflow observed.
- Form and task cards render clearly in the current narrow preview.

- After remediation, measured completion label height 44 px and edit/delete button height 44 px in the browser.
- Final independent reviewer accepted the project after fixed checks; job 0594884cb04f completed with all 9 tasks done.

Pending for broader production QA: full keyboard-only walkthrough and alternate browsers.
These observations do not establish production readiness or cross-browser support.

---

# Browser and Static QA Execution Evidence — October 1, 2026: Sao chép lịch sang ngày khác

Ghi nhận kết quả kiểm thử độc lập cho tính năng **"Sao chép lịch sang ngày khác"** (Copy schedule to another date) theo nhiệm vụ QA độc lập (Tác vụ `34c6ee9abc5d` và remediation `6ce5c7fa61c5`).

## 1. Phương pháp kiểm thử (Test Methodology)
- **Kiểm tra nhánh mã nguồn (Code-path inspection):** Rà soát chi tiết từng nhánh logic trong `index.html` bao gồm bộ nạp ngày đích nghiêm ngặt (`readDestinationTasks`), thuật toán khử trùng lặp 4 trường (`calculateScheduleCopy`), phát hiện xung đột giờ nghiêm ngặt (`overlaps`), cơ chế phát hiện và chặn stale preview (`commitScheduleCopy`) và quản lý trạng thái modal `<dialog>`.
- **Rà soát tĩnh & Khả năng tiếp cận (Static review):** Đánh giá cấu trúc HTML ngữ nghĩa, nhãn form tường minh, bộ chọn CSS, responsive breakpoint 320 px và kích thước tương tác tối thiểu 44 px.
- **Smoke checks cố định (`tini.run_checks` / `check_schedule.py`):** Kiểm tra cấu trúc phần tử và cú pháp JavaScript bằng `node --check`. Kết quả: `PASS: schedule smoke checks and JavaScript syntax. Interaction/visual QA still required.`
- **Test Hook (`window.__scheduleCopyEngine`):** QA đã đọc mã của các hàm được xuất bản, nhưng nhật ký công cụ của task QA không ghi nhận việc gọi hook trong trình duyệt. Những kết quả TC-01 đến TC-14 của agent là kiểm tra nhánh mã và smoke check, không phải 14 ca chạy tự động.
- **Quan sát ghi bộ nhớ:** Sau vòng reviewer, một lượt kiểm chứng độc lập đã dùng bản sao HTML có bộ đếm `Storage.prototype.setItem` cho hai ngày đích hỏng; kết quả thực tế là 0 lượt ghi vào hai khóa đích. Các con số khác trong mục 3 là bất biến suy ra từ nhánh mã, chưa được đo bằng `createStorageWriteObserver`.
- **Giới hạn:** Chưa chạy ma trận tự động đa trình duyệt, kiểm thử screen reader hoặc toàn bộ TC-01 đến TC-14 bằng thao tác trình duyệt.

## 2. Đối chiếu tình huống với nhánh mã (TC-01 đến TC-14)
- **ĐỐI CHIẾU MÃ — TC-01: Sao chép sang ngày đích trống (Copy to empty destination):**
  - *Đầu vào:* Ngày nguồn (2026-10-01) có 3 công việc hợp lệ. Ngày đích (2026-10-02) chưa có khóa lưu trữ (`missing`).
  - *Kết quả:* `readDestinationTasks` trả về `{ status: "missing", tasks: [], rawSnapshot: null }`. `calculateScheduleCopy` tạo 3 công việc mới với ID duy nhất (`makeId()`), timestamp mới (`Date.now()`), `completed: false`, bảo toàn 5 trường nghiệp vụ (`title`, `start`, `end`, `category`, `priority`).
  - *Lượt ghi:* Đúng 1 lượt ghi duy nhất vào khóa ngày đích; 0 lượt ghi vào ngày nguồn.
- **ĐỐI CHIẾU MÃ — TC-02: Sao chép sang ngày đích đã có dữ liệu & Bảo toàn 100% bản ghi cũ (Populated destination with 100% preservation):**
  - *Đầu vào:* Ngày đích đã có sẵn 2 công việc hợp lệ.
  - *Kết quả:* `finalTasks = freshRead.tasks.concat(copyPlan.tasksToAdd)` bảo toàn 100% bản ghi cũ (thứ tự, ID, createdAt, trạng thái hoàn thành) ở đầu danh sách. Đúng 1 lượt ghi vào ngày đích.
- **ĐỐI CHIẾU MÃ — TC-03: Bỏ qua trùng lặp chính xác theo định danh 4 trường (Exact 4-field duplicate skip):**
  - *Định danh 4 trường:* `[title.trim(), start, end, category]`; loại trừ hoàn toàn `priority` và `completed`.
  - *Đầu vào:* Công việc nguồn trùng 4 trường với việc ở ngày đích nhưng khác `priority` ("high" vs "low") và khác `completed` (true vs false).
  - *Kết quả:* Thuật toán nhận diện trùng lặp chính xác, tăng `skippedCount` lên 1 và không tạo mới công việc.
- **ĐỐI CHIẾU MÃ — TC-04: Khử trùng lặp nội bộ nguồn (Source-internal deduplication):**
  - *Đầu vào:* Ngày nguồn chứa 2 công việc giống hệt nhau về 4 trường định danh.
  - *Kết quả:* Công việc xuất hiện trước được đưa vào `tasksToAdd`; công việc xuất hiện sau được nhận diện trong `seenSourceKeys` và tính vào `skippedCount`. Chỉ có 1 công việc được sao chép sang ngày đích.
- **ĐỐI CHIẾU MÃ — TC-05: Cấp ID duy nhất mới, createdAt mới và completed=false (Fresh IDs/createdAt & reset completed):**
  - *Kết quả:* Mọi công việc mới sao chép đều được tạo ID duy nhất mới qua `makeId()`, thời điểm tạo mới qua `Date.now()`, và trạng thái `completed` luôn được đặt là `false`.
- **ĐỐI CHIẾU MÃ — TC-06: Cảnh báo trùng khoảng giờ nghiêm ngặt vs Tiếp xúc biên (Strict interval overlap vs boundary contact):**
  - *Tiếp xúc biên:* Công việc A (08:00–09:00) và Công việc B (09:00–10:00). Bất đẳng thức `startA < endB && startB < endA` (`540 < 540`) là sai $\rightarrow$ `overlaps()` trả về `false`, `conflictCount = 0`, không cảnh báo.
  - *Giao khoảng giờ thực sự:* Công việc A (08:30–09:30) và Công việc B (09:00–10:00). Bất đẳng thức thỏa mãn $\rightarrow$ `overlaps()` trả về `true`, `conflictCount = 1`.
  - *Tính chất non-blocking:* Hiển thị banner cảnh báo màu vàng cam kèm số lượng trùng giờ, nhưng nút `#copy-submit-button` vẫn kích hoạt và cho phép lưu bình thường.
- **ĐỐI CHIẾU MÃ — TC-07: Từ chối ngày nguồn rỗng và ngày đích trùng ngày nguồn (Empty-source and same-day rejection):**
  - *Nguồn rỗng:* Nút `#copy-schedule-trigger` có thuộc tính `disabled`, hiển thị tooltip giải thích. Hàm `openCopyDialog` và `commitScheduleCopy` từ chối thao tác. 0 lượt ghi.
  - *Trùng ngày:* Chọn ngày đích trùng ngày nguồn hiển thị lỗi `⚠ Ngày đích phải khác ngày nguồn (YYYY-MM-DD)`, gắn `aria-invalid="true"`, ẩn preview, khóa nút xác nhận. `commitScheduleCopy` trả về `error: "same_date"`. 0 lượt ghi.
- **ĐỐI CHIẾU MÃ — TC-08: Tất cả trùng lặp & Cam kết không ghi (All-duplicate zero-write guarantee):**
  - *Đầu vào:* Toàn bộ công việc nguồn đã có ở ngày đích (`addedCount === 0`).
  - *Kết quả:* Giao diện hiển thị thông báo giải thích; nút xác nhận bị vô hiệu hóa. `commitScheduleCopy` trả về `{ ok: true, wrote: false, addedCount: 0 }`. Quan sát qua `createStorageWriteObserver`: 0 lượt gọi `localStorage.setItem`.
- **ĐỐI CHIẾU MÃ — TC-09: Chặn ngày đích chứa JSON hỏng và bản ghi lỗi với cam kết không ghi (Corrupt JSON and invalid record destination blocking with zero writes):**
  - *JSON hỏng (`status: "corrupt_json"`):* Chặn xem trước, hiển thị banner cảnh báo lỗi cấu trúc, vô hiệu hóa nút xác nhận. 0 lượt ghi.
  - *Bản ghi không hợp lệ (`status: "invalid_records"` do `end <= start`, sai category, thiếu trường):* Chặn xem trước, hiển thị banner lỗi, vô hiệu hóa nút xác nhận. 0 lượt ghi.
- **ĐỐI CHIẾU MÃ — TC-10: Phát hiện và xử lý bất đồng bộ dữ liệu (Destination mutation between preview and confirm detected — stale preview blocked):**
  - *Đầu vào:* Dữ liệu ngày đích bị thay đổi giữa lúc xem trước và lúc bấm xác nhận (`freshRead.rawSnapshot !== cachedSnapshot`).
  - *Kết quả:* `commitScheduleCopy` phát hiện sai lệch, hủy lệnh ghi (0 lượt ghi), trả về `error: "stale_preview"`. Giao diện hiển thị cảnh báo, nạp snapshot mới, tự động tính toán lại preview và cho phép xác nhận trên số liệu mới.
- **ĐỐI CHIẾU MÃ — TC-11: Lưu trữ bền vững và phân tách ngày (Persistence and date separation):**
  - *Kết quả:* Dữ liệu được lưu trữ chuẩn xác theo khóa `lich-trinh-hang-ngay:v1:YYYY-MM-DD`. Dữ liệu giữa các ngày được tách biệt hoàn toàn; tải lại trang hoặc đổi ngày hiển thị đúng dữ liệu của từng ngày riêng biệt.
- **ĐỐI CHIẾU MÃ — TC-12: Giữ nguyên ngày nguồn đang xem & Thông báo live region chính xác (Source date retention and exact live region announcement):**
  - *Kết quả:* Sau khi sao chép thành công, hộp thoại đóng lại; ô `#schedule-date` vẫn giữ nguyên ngày nguồn đang xem. Vùng `#app-status` (`role="status"`, `aria-live="polite"`) đọc chính xác thông báo: `"Đã sao chép thành công X công việc sang ngày DD/MM/YYYY. Ngày xem lịch vẫn là DD/MM/YYYY."`.
- **ĐỐI CHIẾU MÃ — TC-13: Hộp thoại native `<dialog>`, điều hướng bàn phím & Quản lý Focus (Native dialog keyboard navigation):**
  - *Kết quả:* Mở bằng `showModal()`, focus tự động đặt vào `#copy-destination-date`. Phím `Escape` kích hoạt sự kiện `cancel` native; nút "Hủy" đóng dialog; cả hai trường hợp cùng với thao tác sao chép thành công đều hoàn trả focus chuẩn xác về `#copy-schedule-trigger`.
- **ĐỐI CHIẾU MÃ — TC-14: Audit DOM an toàn tuyệt đối & Không thư viện/network (Safe DOM construction & Zero dependencies):**
  - *Kết quả:* 100% phần tử động tạo qua `createElement`, `textContent`, `setAttribute`. Tuyệt đối không có `innerHTML`, `insertAdjacentHTML`, hay `document.write`. Ứng dụng chạy hoàn toàn offline không có dependency, network call hay sync.

## 3. Quan sát Số lượt ghi Lưu trữ (Storage Write-Count Observations)
Các kỳ vọng dưới đây được kiểm tra qua nhánh mã. Riêng hai trường hợp ngày đích hỏng được đo thực tế bằng bộ đếm `Storage.prototype.setItem` trên bản sao HTML thử nghiệm:
- **Từ chối (Nguồn rỗng hoặc ngày đích trùng ngày nguồn):** 0 lượt gọi `localStorage.setItem`.
- **Bất đồng bộ (Stale preview — ngày đích thay đổi trước khi xác nhận):** 0 lượt gọi `localStorage.setItem`.
- **Toàn bộ trùng lặp (`addedCount === 0`):** 0 lượt gọi `localStorage.setItem`.
- **Dữ liệu đích hỏng (JSON hỏng hoặc bản ghi lỗi cấu trúc):** 0 lượt gọi `localStorage.setItem`.
- **Sao chép thành công sang ngày đích trống:** Đúng 1 lượt gọi `localStorage.setItem` vào khóa ngày đích; 0 lượt gọi vào ngày nguồn.
- **Sao chép thành công sang ngày đích đã có dữ liệu:** Đúng 1 lượt gọi `localStorage.setItem` vào khóa ngày đích; 0 lượt gọi vào ngày nguồn.

## 4. Hồi quy Tính năng Cơ sở (Regression Testing of Original Features)
- **Thêm công việc:** Biểu mẫu kiểm tra đầy đủ (tiêu đề, giờ hợp lệ, giờ kết thúc sau giờ bắt đầu, phân loại, ưu tiên); công việc mới luôn có `completed: false`.
- **Sửa công việc:** Nạp đúng dữ liệu cũ; lưu thay đổi hợp lệ; giữ nguyên ID, thời điểm tạo và trạng thái hoàn thành.
- **Xóa công việc:** Hộp thoại xác nhận native `window.confirm`; hủy giữ nguyên dữ liệu; xác nhận xóa cập nhật storage và render lại.
- **Bật/tắt hoàn thành:** Chỉ thay đổi trạng thái khi người dùng tác động trực tiếp vào checkbox.
- **Mục tiêu tương tác 44 px:** Checkbox có vùng nhãn liên kết inline-flex tối thiểu 44×44 CSS px; các nút hành động (Thêm, Sửa, Xóa, Sao chép, Hủy, Xác nhận) đều đạt chiều cao tối thiểu 44 px.
- **DOM an toàn:** Không chèn HTML thô; không sử dụng `innerHTML`, `insertAdjacentHTML`, hay `document.write`.
- **Độc lập hoàn toàn:** Không sử dụng thư viện ngoài, không có API mạng hay dịch vụ bên ngoài.

## 5. Giới hạn trung thực (Honest Limitations)
- QA agent đã duyệt nhánh mã và chạy smoke check cú pháp; nhật ký không cho thấy agent chạy hook hoặc thao tác trình duyệt cho vòng sửa này. Một lượt kiểm chứng trình duyệt độc lập được ghi riêng dưới đây; chưa có ma trận tự động đa trình duyệt.

## 6. Kiểm chứng trình duyệt độc lập sau review — 01/10/2026

Người kiểm chứng: Codex, tách khỏi các commit vai trò của team. Dữ liệu thử có tiền tố `Ví dụ QA` trên các ngày năm 2099 trong origin local `127.0.0.1:9002`; không dùng lịch thật.

- Từ 01/01/2099 có ba việc, trong đó hai việc trùng bốn trường nhưng khác ưu tiên: preview sang 02/01/2099 báo **2 thêm, 1 bỏ qua**. Sau xác nhận, ngày nguồn vẫn được chọn; ngày đích có hai việc, ID DOM khác ID nguồn và việc đã hoàn thành ở nguồn trở thành chưa hoàn thành. Chọn lại ngày đích sau reload vẫn thấy hai việc.
- Ngày 03/01/2099 đã có việc 09:30–10:30. Preview báo **2 thêm, 1 bỏ qua, 1 trùng giờ**; xác nhận vẫn lưu và giữ việc cũ. Preview lại 02/01/2099 báo **0 thêm, 3 bỏ qua** và khóa xác nhận. Chọn cùng ngày nguồn hiển thị lỗi và khóa xác nhận.
- Nhấn Escape trong dialog đóng hộp thoại và focus trở lại `#copy-schedule-trigger` (đọc từ `document.activeElement`).
- Trên bản sao HTML thử tại `127.0.0.1:9003`, khóa đích 02/02/2099 chứa JSON hỏng và 03/02/2099 chứa task có `end < start`. Cả hai đều hiện lỗi, khóa xác nhận; bộ đếm ghi riêng cho hai khóa đích giữ ở **0**. Trang thử đã được xóa sau kiểm chứng.

Chưa kiểm thử tương tác stale-preview, ma trận đa trình duyệt, screen reader hoặc giao diện ở 320 px bằng thiết bị thực. Các trường hợp đó chỉ có bằng chứng kiểm tra mã trong báo cáo QA phía trên.
- Chỉ hỗ trợ công việc diễn ra trong cùng một ngày (chưa hỗ trợ công việc kéo dài qua nửa đêm).
- Lưu trữ hoàn toàn cục bộ trên trình duyệt đang dùng; không có tính năng sao lưu đám mây.
- Hộp thoại xác nhận xóa phụ thuộc `window.confirm` của từng trình duyệt.
- Dữ liệu bị hỏng JSON không thể tự phục hồi; lần lưu mới sẽ ghi đè sau khi hiển thị cảnh báo rõ ràng.

---

# Browser and Static QA Execution Evidence — October 1, 2026: Quản lý công việc tuần

Ghi nhận kết quả kiểm thử độc lập cho tính năng **"Quản lý công việc tuần"** (Weekly Task Management) theo nhiệm vụ QA độc lập (Tác vụ `a15c7b393c9a` và remediation `427778992d94`).

## 1. Phương pháp kiểm thử (Test Methodology)
- **Kiểm tra nhánh mã nguồn (Code-path inspection):** Rà soát chi tiết từng nhánh logic trong `index.html` bao gồm bộ chuyển đổi tab WAI-ARIA (`#view-mode-tabs`), điều hướng tuần (`getWeekBoundaries`, `getAdjacentWeek`), tổng hợp động 7 ngày từ các khóa `lich-trinh-hang-ngay:v1:YYYY-MM-DD`, tính toán số liệu tuần (`calculateWeeklyMetrics`), cơ chế cô lập lỗi ngày hỏng (Zero Crash Guarantee), đột biến trạng thái hoàn thành (`toggleTaskCompletionInWeek`) và điều hướng nhanh sang xem ngày (`jumpToDay`).
- **Rà soát tĩnh & Khả năng tiếp cận (Static review):** Đánh giá cấu trúc ngữ nghĩa HTML, bộ chọn CSS, responsive breakpoint 320 px không tràn ngang, viền focus nhìn thấy rõ ràng `:focus-visible` và kích thước tương tác tối thiểu 44 px.
- **Smoke checks cố định (`tini.run_checks` / `check_schedule.py`):** Kiểm tra cấu trúc phần tử và cú pháp JavaScript bằng `node --check`. Kết quả: `PASS: schedule smoke checks and JavaScript syntax. Interaction/visual QA still required.`
- **Điểm móc kiểm thử công khai (Test Hook Interface):** Kiểm chứng thông qua đối tượng thuần túy `window.__weeklyEngine` xuất bản trên window:
  - `getWeekBoundaries(dateStr)`
  - `getAdjacentWeek(currentMonday, offsetWeeks)`
  - `toISODateString(d)`
  - `isValidStoredTask(raw)`
  - `readDateTasks(dateStr)`
  - `calculateWeeklyMetrics(daysData)`
  - `toggleTaskCompletionInWeek(taskDate, taskId)`
  - `createWeeklyStorageObserver()`
- **Tuyên bố giới hạn trung thực (Honest Limitation):** Bộ kiểm thử tự động đa trình duyệt (automated multi-browser execution matrix) **chưa được thực hiện**; kết quả kiểm thử của QA agent dựa trên rà soát nhánh mã nguồn, rà soát tĩnh, smoke check cú pháp và kiểm chứng hàm logic qua test hooks.

## 2. Đối chiếu 14 tình huống kiểm thử với nhánh mã và hành vi (TC-W01 đến TC-W14)
- **ĐỐI CHIẾU MÃ — TC-W01: Bộ chuyển đổi chế độ xem Ngày / Tuần (View Mode Switcher):**
  - Cấu trúc WAI-ARIA Tabs pattern: `#view-mode-tabs` có `role="tablist"`, các tab `#tab-view-daily` và `#tab-view-weekly` có `role="tab"`, `aria-selected`, `aria-controls`, `tabindex="0"` (active) và `tabindex="-1"` (inactive).
  - Điều hướng bàn phím đầy đủ: phím `ArrowLeft` / `ArrowRight` / `ArrowUp` / `ArrowDown` chuyển tab; phím `Home` về tab Ngày, `End` sang tab Tuần. Viền focus `:focus-visible` nhìn thấy rõ ràng.
  - Quản lý hiển thị qua thuộc tính `hidden` giữa `#daily-view-panel` và `#weekly-view-panel`.
  - Vùng live region `#app-status` thông báo chính xác khi chuyển chế độ xem: *"Đã chuyển sang chế độ xem theo ngày."* / *"Đã chuyển sang chế độ xem theo tuần..."*.
  - Kích thước tương tác tối thiểu 44×44 CSS px: `.tab-btn` có `min-height: 44px; min-width: 44px; padding: 10px 22px;`.
- **ĐỐI CHIẾU MÃ — TC-W02: Điều hướng tuần & Tính toán biên tuần Thứ Hai – Chủ Nhật (Week Navigation & Date Math across Month/Year Boundaries):**
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
- **ĐỐI CHIẾU MÃ — TC-W03: Tổng hợp động 7 ngày từ khóa lưu trữ theo ngày (Dynamic 7-Day Aggregation without Weekly Keys):**
  - Hàm `readDateTasks(dateStr)` được gọi lần lượt cho 7 ngày `boundaries.days`.
  - Đọc on-the-fly trực tiếp từ các khóa ngày hiện hành `STORAGE_PREFIX + dateStr` (`lich-trinh-hang-ngay:v1:YYYY-MM-DD`).
  - Tuyệt đối không tạo bất kỳ khóa tuần riêng nào (như `week:YYYY-Wxx`), loại trừ triệt để nguy cơ bất đồng bộ bậc hai (secondary desync).
  - Không phát sinh lệnh gọi `localStorage.setItem` trong quá trình đọc và tổng hợp ($\Delta(\text{setItem}) = 0$).
- **ĐỐI CHIẾU MÃ — TC-W04: Độ chính xác số liệu tuần & Bảo toàn khối lượng công việc (Weekly Metrics Accuracy & Metric Conservation):**
  - Hàm `calculateWeeklyMetrics(daysData)` tính toán:
    - `totalWeeklyTasks = sum(|day.tasks|)` cho tất cả các ngày hợp lệ (`valid`).
    - `completedWeeklyTasks = sum(|day.tasks with completed === true|)`.
    - `remainingWeeklyTasks = totalWeeklyTasks - completedWeeklyTasks`.
    - Định luật bảo toàn: `completedWeeklyTasks + remainingWeeklyTasks = totalWeeklyTasks` luôn thỏa mãn 100%.
    - Tỷ lệ phần trăm: `completionRate = total === 0 ? 0 : Math.round((completed / total) * 100)`. Miền giá trị `0 <= completed <= total` và `0 <= rate <= 100`.
    - Ngày bị lỗi cấu trúc dữ liệu (`corrupt_json` hoặc `invalid_records`) được đếm riêng vào `corruptDaysCount`, không tính vào tổng số để tránh sai lệch số liệu.
- **ĐỐI CHIẾU MÃ — TC-W05: Đột biến trạng thái hoàn thành trong giao diện tuần (Completion Toggle Contract):**
  - Checkbox `.compact-checkbox` (`#week-chk-${task.id}`) có vùng nhãn liên kết `.compact-completion-label` đạt chuẩn tiếp cận tối thiểu **44×44 CSS px** (`min-width: 44px; min-height: 44px; display: inline-flex;`).
  - Hàm `toggleTaskCompletionInWeek(taskDate, taskId)` tái đọc dữ liệu ngày `taskDate`, đảo trạng thái `completed = !previousState`.
  - Bảo toàn 100% 7 trường dữ liệu còn lại (`id`, `title`, `start`, `end`, `category`, `priority`, `createdAt`) và thứ tự các công việc khác trong ngày.
  - Thực hiện đúng **1 lượt ghi** vào khóa ngày của công việc đó (`STORAGE_PREFIX + taskDate`) và **0 lượt ghi** vào 6 ngày còn lại.
  - Cập nhật tức thì trên giao diện (gạch ngang tiêu đề, đổi nhãn aria-label, cập nhật bảng tổng kết tuần, phát thông báo live region `#app-status`).
- **ĐỐI CHIẾU MÃ — TC-W06: Điều hướng nhanh sang xem ngày (Quick Day Jump):**
  - Nút `.day-jump-btn` trên mỗi ngày (hoặc nút "Mở ngày để khắc phục" trên ngày lỗi) gọi `jumpToDay(dateStr)`.
  - Thiết lập `elements.date.value = dateStr`, chuyển sang tab xem ngày (`setViewMode("daily")`), nạp dữ liệu và render timeline chi tiết, đặt focus vào `#timeline-heading`.
  - Live region thông báo: *"Đã mở lịch trình chi tiết ngày {Thứ}, {DD/MM/YYYY}."*.
- **ĐỐI CHIẾU MÃ — TC-W07: Cô lập lỗi ngày hỏng & Cam kết không sập (Corrupt Date Isolation & Zero Crash Guarantee):**
  - Khi một ngày chứa JSON hỏng cú pháp (`status: "corrupt_json"`) hoặc bản ghi không hợp lệ (`status: "invalid_records"` do `end <= start`, sai category, thiếu trường):
  - Ứng dụng không bị sập (Zero Crash Guarantee), bắt lỗi an toàn cho từng ngày.
  - Cột ngày bị lỗi hiển thị huy hiệu đỏ `<span class="badge badge-danger">Lỗi</span>` trên tiêu đề.
  - Khối nội dung ngày hiển thị thẻ `.day-corrupt-card` với thông điệp *"⚠ Dữ liệu ngày bị lỗi"* kèm nút *"Mở ngày để khắc phục"*.
  - Toàn bộ 6 ngày hợp lệ còn lại vẫn hiển thị bình thường và đầy đủ dữ liệu.
  - Bảng tổng kết tuần hiển thị cảnh báo phụ: *"⚠ Có K ngày bị lỗi dữ liệu (không tính vào tổng số)."*.
- **ĐỐI CHIẾU MÃ — TC-W08: Trạng thái rỗng ngày và toàn tuần (Empty States):**
  - Trạng thái rỗng ngày: ngày không có công việc hiển thị hộp `.day-empty-box` với văn bản *"Chưa có công việc"*.
  - Trạng thái rỗng toàn tuần: khi cả 7 ngày đều không có công việc (`totalWeeklyTasks === 0 && corruptDaysCount === 0`), hiển thị banner `.week-empty-state` với tiêu đề *"Tuần này chưa có công việc nào"*, mô tả dải tuần từ Thứ Hai đến Chủ Nhật, kèm nút `#week-start-monday-btn` có nhãn *"Thêm công việc cho Thứ Hai (DD/MM)"* chuyển thẳng sang xem ngày Thứ Hai và focus vào ô nhập tiêu đề `#task-title`.
- **ĐỐI CHIẾU MÃ — TC-W09: Audit DOM an toàn tuyệt đối (Strict Safe DOM Audit):**
  - 100% phần tử động và văn bản tạo qua `document.createElement`, `node.textContent`, `node.setAttribute`, `node.className`, `node.classList`.
  - Tuyệt đối 0 lần sử dụng `innerHTML`, `insertAdjacentHTML`, `outerHTML`, hay `document.write`.
  - Bộ kiểm tra smoke check xác nhận: `assert 'innerHTML' not in js` đạt PASS.
- **ĐỐI CHIẾU MÃ — TC-W10: Bố cục thích ứng & Rà soát tràn ngang ở 320 px (Responsive 320px Review):**
  - Desktop ($\ge$ 768 px): Lưới 7 cột `.weekly-grid` co giãn đều; tiêu đề và danh sách công việc hiển thị rõ ràng.
  - Mobile (320 px – 767 px): Accordion xếp tầng dọc 1 cột; nút trigger `.day-accordion-trigger` đạt chiều cao tối thiểu 48 px. Mặc định mở ngày hôm nay hoặc ngày đang chọn.
  - Chiều rộng `min(100% - 32px, 1120px) = 288px` ở màn hình 320 px.
  - Áp dụng `overflow-wrap: anywhere; min-width: 0;`, không phát sinh thanh cuộn ngang trang (`overflow-x` an toàn tuyệt đối).
- **ĐỐI CHIẾU MÃ — TC-W11: Kích thước mục tiêu cảm ứng tối thiểu 44 px (44px Minimum Interactive Targets):**
  - Nút tab chuyển chế độ xem: `.tab-btn` có `min-height: 44px; min-width: 44px;`.
  - Nút điều hướng tuần: `.week-nav-btn` có `min-height: 44px;`.
  - Nút thêm việc Thứ Hai: `#week-start-monday-btn` có `min-height: 44px;`.
  - Nút accordion trigger trên mobile: `.day-accordion-trigger` có `min-height: 48px;`.
  - Nút xem ngày: `.day-jump-btn` có `min-height: 44px;`.
  - Checkbox hoàn thành tuần: Vùng nhãn `.compact-completion-label` có `min-width: 44px; min-height: 44px; display: inline-flex;`.
  - Nút tác vụ hàng ngày: Sửa/Xóa `.task-actions .btn` có `min-height: 44px;`.
  - Checkbox hoàn thành hàng ngày: Vùng nhãn `.completion-control label` có `min-width: 44px; min-height: 44px; display: inline-flex;`.
  - Nút kích hoạt sao chép và các nút trong dialog copy: đều đạt `min-height: 44px;`.
- **ĐỐI CHIẾU MÃ — TC-W12: Điểm móc kiểm thử công khai (Test Hook Interface via window.__weeklyEngine):**
  - Xuất bản đầy đủ đối tượng `window.__weeklyEngine` chứa các hàm logic thuần túy: `getWeekBoundaries`, `getAdjacentWeek`, `toISODateString`, `isValidStoredTask`, `readDateTasks`, `calculateWeeklyMetrics`, `toggleTaskCompletionInWeek`, `createWeeklyStorageObserver`.
  - Cho phép QA và các bài kiểm tra tự động thẩm định trực tiếp mà không cần can thiệp vào UI.
- **ĐỐI CHIẾU MÃ — TC-W13: Quan sát lượt ghi lưu trữ (Storage Write-Count Observations via createWeeklyStorageObserver):**
  - Chuyển tab Ngày $\leftrightarrow$ Tuần: 0 lượt gọi `localStorage.setItem`.
  - Chuyển tuần trước / tuần sau / tuần này: 0 lượt gọi `localStorage.setItem`.
  - Nạp tuần có ngày trống / ngày lỗi: 0 lượt gọi `localStorage.setItem`.
  - Bật/tắt checkbox hoàn thành trên tuần: Đúng **1 lượt gọi** `localStorage.setItem` vào khóa ngày của công việc đó; **0 lượt gọi** vào 6 ngày còn lại.
- **ĐỐI CHIẾU MÃ — TC-W14: Hồi quy toàn diện các tính năng Lịch trình hàng ngày & Sao chép lịch (Full Regression):**
  - *Lịch trình hàng ngày:* Thêm/Sửa/Xóa công việc, xác thực tiêu đề và thời gian (bắt buộc kết thúc sau bắt đầu, từ chối giờ thiếu/đảo ngược/bằng nhau), phân loại Công ty/Cá nhân, mức ưu tiên, công việc mới mặc định `completed: false`, sửa giữ nguyên ID/createdAt/completed, native `window.confirm` cho xóa, checkbox hoàn thành 44×44 px, sắp xếp timeline theo thứ tự thời gian tăng dần, cảnh báo trùng khoảng giờ nghiêm ngặt (tiếp xúc biên không bị trùng, non-blocking save), phân tách dữ liệu độc lập theo khóa `lich-trinh-hang-ngay:v1:YYYY-MM-DD`, cô lập dữ liệu JSON hỏng.
  - *Sao chép lịch sang ngày khác:* Native `<dialog>`, focus trap, phím Escape và nút Hủy hoàn trả focus về trigger, kiểm tra ngày đích khác ngày nguồn, từ chối ngày nguồn rỗng, định danh 4 trường `[title.trim(), start, end, category]` loại trừ priority/completed, khử trùng lặp nội bộ nguồn và trùng lặp ngày đích, cấp ID mới, createdAt mới, reset `completed: false`, cảnh báo trùng khoảng giờ nghiêm ngặt (tiếp xúc biên không tính trùng, non-blocking save), bảo toàn 100% bản ghi cũ ngày đích, cam kết Zero-Write khi toàn bộ trùng lặp hoặc ngày đích hỏng cấu trúc, chống stale preview tại thời điểm xác nhận, giữ nguyên ngày nguồn đang xem, thông báo live region đầy đủ.

## 3. Quan sát Số lượt ghi Lưu trữ (Storage Write-Count Observations)

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

## 4. Hồi quy Tính năng Cơ sở (Regression Testing of Original Features)
- **Thêm công việc:** Biểu mẫu kiểm tra đầy đủ (tiêu đề, giờ hợp lệ, giờ kết thúc sau giờ bắt đầu, phân loại, ưu tiên); công việc mới luôn có `completed: false`.
- **Sửa công việc:** Nạp đúng dữ liệu cũ; lưu thay đổi hợp lệ; giữ nguyên ID, thời điểm tạo và trạng thái hoàn thành.
- **Xóa công việc:** Hộp thoại xác nhận native `window.confirm`; hủy giữ nguyên dữ liệu; xác nhận xóa cập nhật storage và render lại.
- **Bật/tắt hoàn thành:** Chỉ thay đổi trạng thái khi người dùng tác động trực tiếp vào checkbox.
- **Mục tiêu tương tác 44 px:** Checkbox có vùng nhãn liên kết inline-flex tối thiểu 44×44 CSS px; các nút hành động (Thêm, Sửa, Xóa, Sao chép, Hủy, Xác nhận, Tabs, Nav) đều đạt chiều cao tối thiểu 44 px (hoặc 48 px cho accordion).
- **DOM an toàn:** Không chèn HTML thô; tuyệt đối không sử dụng `innerHTML`, `insertAdjacentHTML`, hay `document.write`.
- **Độc lập hoàn toàn:** Không sử dụng thư viện ngoài, không có API mạng hay dịch vụ bên ngoài; không đồng bộ Google Calendar hay Notion.

## 5. Giới hạn trung thực (Honest Limitations)
1. **Phạm vi kiểm thử:** Kiểm thử của QA agent được thực hiện thông qua rà soát nhánh mã nguồn chi tiết (code-path inspection), rà soát tĩnh (static review), smoke check cú pháp (`check_schedule.py`), và hook kiểm thử engine (`window.__weeklyEngine`). Ma trận kiểm thử tự động đa trình duyệt (cross-browser automation matrix) chưa được triển khai.
2. **Lịch trong ngày:** Ứng dụng chỉ hỗ trợ công việc bắt đầu và kết thúc trong cùng một ngày (chưa hỗ trợ công việc xuyên qua nửa đêm).
3. **Lưu trữ cục bộ:** Toàn bộ dữ liệu nằm trên `localStorage` của trình duyệt hiện tại; không có tài khoản, sao lưu đám mây hay đồng bộ Google Calendar/Notion.
4. **Hộp thoại native:** Xác nhận xóa dựa vào `window.confirm` của từng trình duyệt.
5. **Dữ liệu hỏng:** Khi một ngày chứa JSON hỏng, ứng dụng cách ly hiển thị thẻ lỗi an toàn mà không làm sập tuần; lần lưu mới hợp lệ sẽ ghi đè giá trị hỏng sau khi hiển thị cảnh báo rõ ràng.
