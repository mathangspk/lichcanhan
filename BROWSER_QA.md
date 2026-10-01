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
- **Kiểm chứng qua Test Hook (`window.__scheduleCopyEngine`):** Sử dụng các hàm engine thuần túy được xuất bản tại `window.__scheduleCopyEngine` (`calculateScheduleCopy`, `readDestinationTasks`, `commitScheduleCopy`, `isValidStoredTask`, `overlaps`, `getTaskDuplicateKey`).
- **Quan sát số lượt ghi bộ nhớ (`createStorageWriteObserver`):** Sử dụng hook spy quan sát `localStorage.setItem` để kiểm chứng chính xác số lượt ghi trong từng kịch bản mà không làm lộ dữ liệu người dùng.
- **Tuyên bố giới hạn trung thực (Honest Statement):** Kiểm thử tự động đa trình duyệt (automated multi-browser execution matrix) **chưa được thực hiện**. Kiểm thử được tiến hành qua duyệt mã nguồn tĩnh, kiểm tra cú pháp smoke checks, thử nghiệm thủ công trên bản preview và kiểm chứng logic qua hook engine.

## 2. Các ca kiểm thử độc lập (TC-01 đến TC-14)
- **PASS — TC-01: Sao chép sang ngày đích trống (Copy to empty destination):**
  - *Đầu vào:* Ngày nguồn (2026-10-01) có 3 công việc hợp lệ. Ngày đích (2026-10-02) chưa có khóa lưu trữ (`missing`).
  - *Kết quả:* `readDestinationTasks` trả về `{ status: "missing", tasks: [], rawSnapshot: null }`. `calculateScheduleCopy` tạo 3 công việc mới với ID duy nhất (`makeId()`), timestamp mới (`Date.now()`), `completed: false`, bảo toàn 5 trường nghiệp vụ (`title`, `start`, `end`, `category`, `priority`).
  - *Lượt ghi:* Đúng 1 lượt ghi duy nhất vào khóa ngày đích; 0 lượt ghi vào ngày nguồn.
- **PASS — TC-02: Sao chép sang ngày đích đã có dữ liệu & Bảo toàn 100% bản ghi cũ (Populated destination with 100% preservation):**
  - *Đầu vào:* Ngày đích đã có sẵn 2 công việc hợp lệ.
  - *Kết quả:* `finalTasks = freshRead.tasks.concat(copyPlan.tasksToAdd)` bảo toàn 100% bản ghi cũ (thứ tự, ID, createdAt, trạng thái hoàn thành) ở đầu danh sách. Đúng 1 lượt ghi vào ngày đích.
- **PASS — TC-03: Bỏ qua trùng lặp chính xác theo định danh 4 trường (Exact 4-field duplicate skip):**
  - *Định danh 4 trường:* `[title.trim(), start, end, category]`; loại trừ hoàn toàn `priority` và `completed`.
  - *Đầu vào:* Công việc nguồn trùng 4 trường với việc ở ngày đích nhưng khác `priority` ("high" vs "low") và khác `completed` (true vs false).
  - *Kết quả:* Thuật toán nhận diện trùng lặp chính xác, tăng `skippedCount` lên 1 và không tạo mới công việc.
- **PASS — TC-04: Khử trùng lặp nội bộ nguồn (Source-internal deduplication):**
  - *Đầu vào:* Ngày nguồn chứa 2 công việc giống hệt nhau về 4 trường định danh.
  - *Kết quả:* Công việc xuất hiện trước được đưa vào `tasksToAdd`; công việc xuất hiện sau được nhận diện trong `seenSourceKeys` và tính vào `skippedCount`. Chỉ có 1 công việc được sao chép sang ngày đích.
- **PASS — TC-05: Cấp ID duy nhất mới, createdAt mới và completed=false (Fresh IDs/createdAt & reset completed):**
  - *Kết quả:* Mọi công việc mới sao chép đều được tạo ID duy nhất mới qua `makeId()`, thời điểm tạo mới qua `Date.now()`, và trạng thái `completed` luôn được đặt là `false`.
- **PASS — TC-06: Cảnh báo trùng khoảng giờ nghiêm ngặt vs Tiếp xúc biên (Strict interval overlap vs boundary contact):**
  - *Tiếp xúc biên:* Công việc A (08:00–09:00) và Công việc B (09:00–10:00). Bất đẳng thức `startA < endB && startB < endA` (`540 < 540`) là sai $\rightarrow$ `overlaps()` trả về `false`, `conflictCount = 0`, không cảnh báo.
  - *Giao khoảng giờ thực sự:* Công việc A (08:30–09:30) và Công việc B (09:00–10:00). Bất đẳng thức thỏa mãn $\rightarrow$ `overlaps()` trả về `true`, `conflictCount = 1`.
  - *Tính chất non-blocking:* Hiển thị banner cảnh báo màu vàng cam kèm số lượng trùng giờ, nhưng nút `#copy-submit-button` vẫn kích hoạt và cho phép lưu bình thường.
- **PASS — TC-07: Từ chối ngày nguồn rỗng và ngày đích trùng ngày nguồn (Empty-source and same-day rejection):**
  - *Nguồn rỗng:* Nút `#copy-schedule-trigger` có thuộc tính `disabled`, hiển thị tooltip giải thích. Hàm `openCopyDialog` và `commitScheduleCopy` từ chối thao tác. 0 lượt ghi.
  - *Trùng ngày:* Chọn ngày đích trùng ngày nguồn hiển thị lỗi `⚠ Ngày đích phải khác ngày nguồn (YYYY-MM-DD)`, gắn `aria-invalid="true"`, ẩn preview, khóa nút xác nhận. `commitScheduleCopy` trả về `error: "same_date"`. 0 lượt ghi.
- **PASS — TC-08: Tất cả trùng lặp & Cam kết không ghi (All-duplicate zero-write guarantee):**
  - *Đầu vào:* Toàn bộ công việc nguồn đã có ở ngày đích (`addedCount === 0`).
  - *Kết quả:* Giao diện hiển thị thông báo giải thích; nút xác nhận bị vô hiệu hóa. `commitScheduleCopy` trả về `{ ok: true, wrote: false, addedCount: 0 }`. Quan sát qua `createStorageWriteObserver`: 0 lượt gọi `localStorage.setItem`.
- **PASS — TC-09: Chặn ngày đích chứa JSON hỏng và bản ghi lỗi với cam kết không ghi (Corrupt JSON and invalid record destination blocking with zero writes):**
  - *JSON hỏng (`status: "corrupt_json"`):* Chặn xem trước, hiển thị banner cảnh báo lỗi cấu trúc, vô hiệu hóa nút xác nhận. 0 lượt ghi.
  - *Bản ghi không hợp lệ (`status: "invalid_records"` do `end <= start`, sai category, thiếu trường):* Chặn xem trước, hiển thị banner lỗi, vô hiệu hóa nút xác nhận. 0 lượt ghi.
- **PASS — TC-10: Phát hiện và xử lý bất đồng bộ dữ liệu (Destination mutation between preview and confirm detected — stale preview blocked):**
  - *Đầu vào:* Dữ liệu ngày đích bị thay đổi giữa lúc xem trước và lúc bấm xác nhận (`freshRead.rawSnapshot !== cachedSnapshot`).
  - *Kết quả:* `commitScheduleCopy` phát hiện sai lệch, hủy lệnh ghi (0 lượt ghi), trả về `error: "stale_preview"`. Giao diện hiển thị cảnh báo, nạp snapshot mới, tự động tính toán lại preview và cho phép xác nhận trên số liệu mới.
- **PASS — TC-11: Lưu trữ bền vững và phân tách ngày (Persistence and date separation):**
  - *Kết quả:* Dữ liệu được lưu trữ chuẩn xác theo khóa `lich-trinh-hang-ngay:v1:YYYY-MM-DD`. Dữ liệu giữa các ngày được tách biệt hoàn toàn; tải lại trang hoặc đổi ngày hiển thị đúng dữ liệu của từng ngày riêng biệt.
- **PASS — TC-12: Giữ nguyên ngày nguồn đang xem & Thông báo live region chính xác (Source date retention and exact live region announcement):**
  - *Kết quả:* Sau khi sao chép thành công, hộp thoại đóng lại; ô `#schedule-date` vẫn giữ nguyên ngày nguồn đang xem. Vùng `#app-status` (`role="status"`, `aria-live="polite"`) đọc chính xác thông báo: `"Đã sao chép thành công X công việc sang ngày DD/MM/YYYY. Ngày xem lịch vẫn là DD/MM/YYYY."`.
- **PASS — TC-13: Hộp thoại native `<dialog>`, điều hướng bàn phím & Quản lý Focus (Native dialog keyboard navigation):**
  - *Kết quả:* Mở bằng `showModal()`, focus tự động đặt vào `#copy-destination-date`. Phím `Escape` kích hoạt sự kiện `cancel` native; nút "Hủy" đóng dialog; cả hai trường hợp cùng với thao tác sao chép thành công đều hoàn trả focus chuẩn xác về `#copy-schedule-trigger`.
- **PASS — TC-14: Audit DOM an toàn tuyệt đối & Không thư viện/network (Safe DOM construction & Zero dependencies):**
  - *Kết quả:* 100% phần tử động tạo qua `createElement`, `textContent`, `setAttribute`. Tuyệt đối không có `innerHTML`, `insertAdjacentHTML`, hay `document.write`. Ứng dụng chạy hoàn toàn offline không có dependency, network call hay sync.

## 3. Quan sát Số lượt ghi Lưu trữ (Storage Write-Count Observations)
Xác minh thực tế thông qua wrapper `window.__scheduleCopyEngine.createStorageWriteObserver()`:
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
- Kiểm thử được thực hiện bằng duyệt nhánh mã tĩnh, smoke checks cú pháp (`check_schedule.py`), và hook kiểm thử engine; chưa chạy ma trận kiểm thử tự động đa trình duyệt (cross-browser automation matrix).
- Chỉ hỗ trợ công việc diễn ra trong cùng một ngày (chưa hỗ trợ công việc kéo dài qua nửa đêm).
- Lưu trữ hoàn toàn cục bộ trên trình duyệt đang dùng; không có tính năng sao lưu đám mây.
- Hộp thoại xác nhận xóa phụ thuộc `window.confirm` của từng trình duyệt.
- Dữ liệu bị hỏng JSON không thể tự phục hồi; lần lưu mới sẽ ghi đè sau khi hiển thị cảnh báo rõ ràng.
