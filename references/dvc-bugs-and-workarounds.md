# DVC, iCanhan, HĐĐT & Google Sheets — Bugs, Gotchas & Workarounds

Verified production patterns and failure modes encountered during VN personal and household business (`HKD`/`CNKD`) tax operations on `dichvucong.gdt.gov.vn`, `canhan.gdt.gov.vn`, e-invoice portals (`hoadondientu.gdt.gov.vn` / HĐĐT), and Google Sheets.

---

## 0. Fast & Exact Lookup Patterns (DVC & iCanhan)

- **DVC Obligation Ledger (`/tthc/tracuu-nvt/init`):** Contains authoritative payable `ID`s (`16`-digit keys) needed to reconcile or exclude items across `01/ĐNXLNT` batches. Extract via DOM script (`td` -> JSON array) rather than multi-page screenshots.
- **DVC Filed History (`/tthc/tchs`):** Strict `<= 30-day` filter + per-query CAPTCHA. Reserve `/tthc/tchs` strictly for fetching known `Mã hồ sơ` receipts or launching `Nộp hồ sơ bổ sung` (`01/BĐS`); do NOT sweep multi-year histories on `/tthc/tchs` when `canhan.gdt.gov.vn` already holds multi-year returns/notices without CAPTCHA.
- **iCanhan Multi-Year Sweep (`canhan.gdt.gov.vn`):** Run in a separate tab (never touches DVC's `JSESSIONID`). Set `Từ ngày = 01/01/2015`, `Đến ngày = DD/MM/YYYY` across `Tra cứu thông báo`, `Tra cứu tờ khai`, `Tra cứu chứng từ`, `ĐKT`, `QTT`, `05TH`, `02TH`, `NPT`, `DKT` (`LPTB` capped at 1 year/query). Fetch XML payloads via same-origin `fetch()` inside the tab so 10–20+ files complete in seconds instead of blocking on Chrome's multiple-download guard.
- **Large Form Cap & Chunking (`01/ĐNXLNT` `<= 25` rows/form, `8–10` rows/chunk):** Although DVC's `#tbtkhaiNT` table accepts 70+ rows server-side, cloning `>40` `<select>`/`selectpicker` rows creates ~90,000 accessibility/DOM nodes that hang Chrome DevTools `includeSnapshot` / `navigate_page` (`connection closed: calling "tools/call": client is closing`). **Always split large debt sets into `<= 25` rows per `01/ĐNXLNT` submission**, pre-calculate exact `ID`s + partial `IIct13` overrides offline, add/fill rows in **chunks of `8–10` rows** with native `change` event dispatch (chunks >20 rows exceed the ~60s MCP tool timeout from ~1.5s/row synchronous `changeCQThu` AJAX + `selectpicker('refresh')`), and assert `IItongCot13 <= ItongCot10` before `btnNextStep`.

---

## 1. `dichvucong.gdt.gov.vn` (DVC Thuế)

### 1.1 Strict Single-Tab Session Lock, Concurrency & HTTP 403 Timeout
- **Symptom:** Opening a second tab on `dichvucong.gdt.gov.vn` invalidates `JSESSIONID` (`HTTP 403`, redirect to `/tthc/homelogin`); running a second Chrome DevTools subagent concurrently (even on Drive or Sheets) steals window focus/tab state during CAPTCHA/OTP or triggers `403`.
- **Fix:** Maintain **exactly ONE** browser subagent/tab on `dichvucong.gdt.gov.vn`. Never run parallel Chrome DevTools subagents while a DVC form or user CAPTCHA/OTP is active. If `403` occurs, pause and ask the user to re-authenticate via **VNeID** on `/tthc/homelogin`.

### 1.2 `/tthc/tchs` 30-Day Date Range Limit
- **Symptom:** Searching `Tra cứu hồ sơ đã nộp` (`/tthc/tchs`) with `Từ ngày – Đến ngày` > 30 days fails validation or returns empty results.
- **Fix:** Always narrow search windows to `<= 30 days` (e.g., `01/09/2026` – `30/09/2026`) and search by exact `Mã hồ sơ` when known. Note: `/tthc/tchs` requires user **CAPTCHA** entry.

### 1.3 `01/BĐS` (`NĐ 68/2026/NĐ-CP`; DVC `/tthc/tk01-bds-tt50`) `Loại tờ khai` Locked & Duplicate `Lần đầu` Block
- **Symptom:** On `/tthc/tk01-bds-tt50/thong-tin-chung`, `Loại tờ khai` is `disabled` (locked to `Tờ khai chính thức`). Submitting a second `Lần đầu` `01/BĐS` for the same tax year fails with: `"Đã tồn tại tờ khai chính thức được chấp nhận. Vui lòng kê khai tờ khai bổ sung."`
- **Fix:** Launch supplementary filing (`Khai bổ sung`) from `/tthc/tchs` -> locate the accepted official `01/BĐS` filing -> click **`Nộp hồ sơ bổ sung`**. Carry forward all previously declared locations/revenues AND add the new location (`ĐĐKD`) or fix contract dates in the same supplementary return.

### 1.4 `01/BĐS` Supplementary Online Filing HTTP 500 (`TKhaiTypeIdResolver`)
- **Symptom:** Clicking `Nộp hồ sơ bổ sung` -> `Kê khai trực tuyến` -> `Tiếp tục` triggers:
  `500 on POST https://dichvucong-service.gdt.gov.vn/tmdtservice/api/khaithuetmdt/pre-lap-tkhai`
  `Handler dispatch failed: java.lang.NoClassDefFoundError: Could not initialize class vn.com.seatechit.tkhai.TKhaiTypeIdResolver`
- **Fix:** DVC Java backend bug (`tmdtservice`). Do **NOT** retry blindly in a loop. Switch to **`Nộp XML`** (generate the supplementary `01/BĐS` XML using **HTKK** latest version and upload via the same popup) or pause and record the error for retry after DVC maintenance.

### 1.5 `01/BK-STK` (`maToKhai=1265`) Radio Lock & HTTP 500
- **Symptom:** Bank account notification (`/tthc/tk01-bk-stk/p01-select-cqt`) locks location radios to `00999` and `POST .../p01-select-cqt` returns HTTP `500` `"Đã có lỗi hệ thống xảy ra!"`.
- **Fix:** Stop and instruct the user to retry later or submit `01/BK-STK` via **eTax Mobile**.

### 1.6 ĐĐKD Issuance (`01/TB-ĐĐKD`) vs Declaration Sync Lag
- **Symptom:** After `01/TB-ĐĐKD` is submitted, the new 5-digit `ĐĐKD` code appears immediately in the ĐĐKD registry, but `cap-ma-ddkd` or dropdown sync on follow-up forms may temporarily return `{}` or lag.
- **Fix:** Verify the new `ĐĐKD` code in the registry first, then wait or use `Khai bổ sung` / XML filing.

### 1.7 `01/ĐNXLNT` Validation Rules (`arrCQThuByCqthue`, Partial Lock, Bank STK at `0 đ`)
- **Single Receiving-CQT Lock (`arrCQThuByCqthue`):** Step 1's selected receiving tax office (`Kính gửi`) strictly filters/validates Mục II (`tbtkhaiBT`). Submitting Mục II overpayments belonging to multiple CQTs in one form fails `checkTkhaiChinhCN()`. **Fix:** Split overpayments into **1 submission per Mục II CQT**; Mục III.1 (`tbtkhaiNT`) accepts debts from ANY CQT/province/chapter nationwide (**Điều 42 TT 89/2026**).
- **Mục II Row Lock vs Mục III.1 Partial Offset:** `checkTbodyTableBT()` enforces `toNumber(Ict10) === toNumber(Ict10_nvt)` on each Mục II overpayment row (cannot partially consume an overpayment row by lowering column 10; excess over Mục III.1 stays on the ledger when `soTienHoanTra = 0`). Mục III.1 (`IIct13`) **does** allow partial offset (`IIct13 <= IIct12`), leaving `IIct14` for a subsequent batch. Strictly record and **exclude** already-submitted debt `ID`s in follow-up batches (`01/ĐNXLNT` pending items remain selectable in `tbtkhaiNT` until CQT processes the prior submission).
- **`Tiền mặt` Hard-Disabled & Mandatory Bank Account (`STK`) Even at `0 đ`:** On `ke-khai-noi-dung-cn`, `resetInfoTTinNntIII(type)` sets `tienMattmp.disabled = true` for ALL refund brackets (`<= 0`, `<= 5M`, `> 5M`), while `checkTkhaiChinhCN()` blocks `Tiếp tục` unless `Chuyển khoản` (`chuyenKhoantmp`) is checked AND all 3 fields (`tenChuTaiKhoan`, `taiKhoanSo`, `taiNganHang`) are filled — **even when `soTienHoanTra = 0 đ`**. **Fix:** Always obtain the taxpayer's bank details before preparing `01/ĐNXLNT`.
- **Optional Attachments & Receipt Screen:** `upload-tai-lieu` marks supporting docs optional (`batBuoc = "false"`). `05 Kết quả` only shows `THÀNH CÔNG` + `Mã hồ sơ` (`G12.18-...`); download/verify receipts via `/tthc/tchs` (requires CAPTCHA).

### 1.8 `08-MST` Rejected With Error `0525` (`hoSo_TDTTThayDoiGCN=true`) -> Use `hokinhdoanh.dkkd.gov.vn`
- **Symptom:** Submitting `08-MST` on `dichvucong.gdt.gov.vn` to change an HKD's business name, primary sector (`ngành nghề chính`), head office address, capital, or owner fails with notice `maTBao=916`, **`Lỗi 0525`**: `"NNT không được nộp HS đến CQT. Đề nghị NNT nộp HS đến CQ cấp giấy phép"`.
- **Cause & Fix:** When an HKD holds a business registration certificate (`GCN ĐKHKD`), those core fields fall under the one-stop inter-agency linkage (`một cửa liên thông ĐKHKD – ĐKT`, flag `hoSo_TDTTThayDoiGCN=true`). Do **NOT** resubmit `08-MST` to CQT; instead, log in via VNeID on **`https://hokinhdoanh.dkkd.gov.vn`** (or file at the ward/commune `UBND` one-stop desk) to amend the `GCN ĐKHKD` — the approved update syncs automatically to the tax authority. Reserve `08-MST` strictly for non-GCN fields or taxpayers without a `GCN ĐKHKD`.

---

## 2. `canhan.gdt.gov.vn` (iCanhan), XML Archival & Overpayment Forensics

### 2.1 Column `(9) Còn phải nộp` vs Column `(10) Nộp thừa / Tạm nộp`
- **Symptom:** iCanhan displays a non-zero figure (even labeled `"Tiền chậm nộp"`) that looks like tax due at a glance.
- **Fix:** Always inspect column headers and cross-check DVC `/tthc/tracuu-nvt/init`:
  - **Mục A / Column `(9)`**: `Còn phải nộp` (actual debt).
  - **Mục I / Column `(10)`**: `Nộp thừa / Tạm nộp` (overpayment / advance credit eligible for **`01/ĐNXLNT`** refund or offset under **Điều 42 TT 89/2026/TT-BTC**).

### 2.2 Batch XML Retrieval Beyond Chrome's Multi-Download Guard
- **Symptom:** Clicking multiple `Tải về` buttons in `Tra cứu thông báo` / `Tra cứu tờ khai` on `canhan.gdt.gov.vn` only writes the 1st `thongbao.xml` / `ICANHAN<MST>.xml` to `~/Downloads` before Chrome blocks subsequent per-origin automatic downloads; also iCanhan only serves `.xml` (no PDF export).
- **Fix:** Execute authenticated same-origin `fetch()` calls inside the active browser tab for each row, capture raw XML payloads + metadata (`maGiaoDich`, `soThongBao`, `ngayGui`, `kyTinhThue`, `noiNop`), and write them locally with descriptive names (`YYYY-MM-DD_<Type>_<NoticeNo>_tx-<TxID>.xml`). When uploading to Google Drive Web without touching unrelated code repos, keep Tax Archives in a dedicated personal root folder (separate from unrelated asset or business folders).

### 2.3 Forensic Audit of Historical Overpayments (`"Không thành công"` vs Misrouted Payments)
- **Trap:** Assuming all `Mục I / Cột 10` overpayments are safe cash refunds without checking submitted return XMLs on iCanhan.
- **Pattern Found:** Historical Treasury payments (e.g. prior years) often sit as "overpayments" ONLY because the matching return (`tx-...`) ended in `"Hồ sơ đã chuyển vào hệ thống quản lý thuế không thành công"` (Treasury collected money, backend recorded `0` tax due). Conversely, misrouted payments (tax return accepted in CQT A, money paid to CQT B) are 100% verified credits. **Fix:** Download and inspect historical return XMLs FIRST, then **prioritize clean misrouted overpayments** before touching legacy failed-ingest credits (which may trigger supporting-document review).

---

## 3. Legal & Accounting Classification Traps

### 3.1 2026 `1 tỷ đồng/năm` Threshold vs 2024–2025 `100 triệu/năm`
- **Rule:** **NĐ 141/2026/NĐ-CP** (amending **NĐ 68/2026/NĐ-CP**) raises the HKD/individual VAT & PIT exemption threshold to **`1 tỷ đồng/năm`** and abolishes license fee (`lệ phí môn bài`), effective **`01/01/2026`**.
- **Trap:** `2024–2025` remain governed by the old **`100 triệu/năm`** threshold (TT 40/2021). **NEVER** amend `2024–2025` upward without explicit user confirmation, as it triggers `10%` VAT+PIT plus `0.03%/day` late-payment interest.

### 3.2 `01/TKN-CNKD` Mandate Even When `<= 1 tỷ` (TT 89/2026)
- **Rule:** **TT 89/2026/TT-BTC** (effective `01/07/2026`, repealing TT 80/2021) + **Điểm a Khoản 1 Điều 8 NĐ 68/2026** require submitting annual actual revenue notice **`01/TKN-CNKD`** by **`31/01` of the following year** even when revenue `<= 1 tỷ` (`0 đ` tax). Use **Mục E** on `01/TKN-CNKD` to claim refund/offset of platform over-withholding (e.g., e-commerce or ride-hailing/delivery platforms).

### 3.3 Non-Taxable Security Deposits (`Tiền cọc`) & Period Allocation
- **Trap:** Counting a refundable security deposit (`tiền cọc`) as taxable revenue inflates declared totals (`Điều 3 & Điều 4 NĐ 68/2026/NĐ-CP`, `VBHN 112/VBHN-VPQH`). Only count actual earned revenue/rent attributable to the tax period (`01/BĐS + 01/BK-BĐS` or `01/TKN-CNKD`).

---

## 4. E-Invoice Portal Verification (`hoadondientu.gdt.gov.vn` / Phần mềm HĐĐT)
- **Trap:** Relying on a single summary count may mix unissued drafts (`Chưa phát hành`) or cancelled/replaced/adjusted invoices (`Hủy / Thay thế / Điều chỉnh`) with valid issued invoices; also `hoadondientu.gdt.gov.vn` login/queries require user **CAPTCHA**.
- **Fix:** Pause for user CAPTCHA if querying `hoadondientu.gdt.gov.vn`, or filter in the taxpayer's authenticated HĐĐT tool by `01/01/YYYY – DD/MM/YYYY` keeping ONLY valid issued invoices (`Hợp lệ` / `Đã phát hành` / `Đã cấp mã`), and cross-check detail list totals against monthly/summary reports until `100%` matched.

---

## 5. Google Sheets Tracking (3-Tab Multi-Taxpayer Architecture)
- **Trap:** Overwriting user's existing tabs, spawning extra tabs per taxpayer, placing interactive `Checkboxes` directly inside dynamic `FILTER(...)` output cells, using truncated `SUMIFS` bounds when appending rows, or polling Gemini side-panel `wait_for` text with 45–55s timeouts (`Timed out after waiting 50000ms`).
- **Fix:** Use the unified 3-tab structure (`Thuế - Tổng hợp` selector `C3` + KPI/FILTER views, `Thuế - Phát sinh chi tiết` master ledger with `Đối tượng` dropdown pills, `Thuế - Nghĩa vụ & Todo` static rows + checkboxes). Keep generous formula bounds (e.g. `$A$20:$L$500`), verify **`Saved to Drive`**, and visually inspect screenshots across all selector values (`Person 1`, `Person 2`, `Tất cả`) for `#REF!`, `#ERROR!`, or clipping.

---

## 6. Chrome DevTools MCP & Google Drive Upload Workarounds
- **`upload_file` Sandbox Block (`Error: Access denied: path ...`):** Chrome DevTools `upload_file` rejects paths outside the active conversation workspace (e.g., `~/Downloads/` or `scratch/`). **NEVER copy personal tax files into an unrelated software repo workspace** just to satisfy `upload_file`. Instead, in Google Drive Web, hook `HTMLInputElement.prototype.click`, click `New -> File upload`, construct `File` objects via `DataTransfer` from JSON payloads, assign `input.files = dt.files`, and dispatch `change`.
- **Blocked `chrome:` URLs (`chrome://downloads`):** `navigate_page` throws `Error: Navigating to chrome: URLs is not allowed.` Do not attempt to inspect downloads via `chrome://`; use in-page `fetch()` + local scratch extraction (§2.2).
- **Large DOM Snapshot Hangs (`connection closed`):** Calling `includeSnapshot: true` or `navigate_page` on pages with huge DOM trees (~90k nodes from 70+ Bootstrap `selectpicker` rows) hangs CDP until `connection closed: calling "tools/call": client is closing`. Keep forms `<= 25` rows (§0) and prefer `includeSnapshot: false` during repetitive clicks.
- **Transient Chrome Connection Errors (`Could not connect to Chrome`, `Execution context was destroyed`, `No page found`):** When a page navigates mid-script or DevTools restarts, call `list_pages` + re-select the target page and inspect live DOM state rather than blindly retrying form mutations.
