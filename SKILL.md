---
name: etax-canhan-hkd
description: Automate and audit Vietnamese individual and household business (HKD/CNKD) tax obligations across dichvucong.gdt.gov.vn (DVC Thuế), canhan.gdt.gov.vn (iCanhan), e-invoice portals (hoadondientu.gdt.gov.vn or taxpayer's HDDT tool), TVPL, and Google Sheets. Use when preparing 08-MST, 01/TB-ĐĐKD, 01/TKN-CNKD, 01/CNKD, 02/CNKD-TNCN-QTT, 01/BĐS (+ 01/BK-BĐS), 01/ĐNXLNT, 01/BK-STK, 02/QTT-TNCN, 03/TK-SDĐPNN, auditing tax debt vs overpayments, verifying past payment receipts, reconciling e-invoice revenue, or troubleshooting DVC portal 500/403/CAPTCHA errors.
category: domain-tax
keywords: [vietnam-tax, hkd, cnkd, dichvucong, icanhan, hoadondientu, 01-dnxlnt, 01-tkn-cnkd, 08-mst, 01-bds]
---

# VN Personal & HKD Tax Operations (DVC, iCanhan, HĐĐT, Google Sheets)

Operational playbook for auditing, preparing, and tracking Vietnamese individual and household business (`HKD`/`CNKD`) tax obligations across government portals and spreadsheets.

## Cách sử dụng & Các nút thắt bắt buộc người dùng (VNeID + Tự nhập CAPTCHA)

1. **Đăng nhập thủ công qua VNeID trước khi gọi Agent:**
   - Người dùng mở Chrome (chỉ **01 tab** cho `dichvucong.gdt.gov.vn`), tự đăng nhập qua **Định danh điện tử (VNeID)** + xác nhận trên app VNeID tại `https://dichvucong.gdt.gov.vn/tthc/homelogin` (và `https://canhan.gdt.gov.vn`, `https://hoadondientu.gdt.gov.vn` nếu cần đối chiếu hóa đơn, hoặc **`https://hokinhdoanh.dkkd.gov.vn`** nếu cần thay đổi tên/ngành nghề/trụ sở trên GCN ĐKHKD). Không mở 2 tab DVC song song (`HTTP 403`).
2. **Các bước bắt buộc người dùng tự nhập CAPTCHA / OTP (Agent PHẢI dừng lại chờ):**
   - **Tra cứu hồ sơ đã nộp (`/tthc/tchs`) & Thông báo/Hóa đơn:** Mỗi lần tìm kiếm trên `/tthc/tchs` (tối đa `30 ngày/lần`) hoặc đăng nhập/tra cứu trên `hoadondientu.gdt.gov.vn` đều bắt **tự gõ CAPTCHA**.
   - **Nộp từng tờ khai (`Bước 4: Nộp hồ sơ`):** Với **MỖI** tờ khai (`01/BĐS`, `01/ĐNXLNT`, `01/TB-ĐĐKD`, `08-MST`, `01/TKN-CNKD`...), Agent điền xong tới màn hình tổng hợp và **dừng chờ người dùng tự nhập CAPTCHA + xác nhận OTP** (nếu tách nhiều bộ `<= 25` dòng `01/ĐNXLNT`, người dùng ký/nhập CAPTCHA từng bộ).
   - **Lỗi hệ thống DVC `500`:** `01/BĐS` bổ sung dính `500` -> người dùng tự kết xuất XML từ **HTKK** để Agent/bạn upload qua `Nộp XML`; `01/BK-STK` dính `500` -> tự nộp trên **eTax Mobile**.
3. **Checklist phải hỏi/yêu cầu người dùng chuẩn bị sẵn trước khi chạy:**
   - **Điện thoại cài VNeID (mức 2)** (để đăng nhập portal) + **SĐT đăng ký thuế bên cạnh để nhận SMS OTP** khi ký nộp từng hồ sơ (cộng app **eTax Mobile** nếu `01/BK-STK` lỗi 500).
   - **STK ngân hàng chính chủ** (`Số TK`, `Ngân hàng`, `Chủ TK VIẾT HOA KHÔNG DẤU`) — bắt buộc cho `01/ĐNXLNT` ngay cả khi `Hoàn trả = 0 đ` vì DVC khóa cứng `Tiền mặt` — và hỏi rõ người dùng muốn **hoàn tiền** hay **chỉ bù trừ (`Hoàn trả = 0 đ`, dư giữ lại trên sổ)**.
   - **Tài liệu đối chiếu (khi cần):** hợp đồng/chứng từ thu nhập, quyền vào cổng/phần mềm HĐĐT, **HTKK** mới nhất (fallback `Nộp XML` nếu khai bổ sung trên web dính `500`), và Google Drive/Sheets nếu lưu trữ hồ sơ.

## Scope & Security Policy

- **Scope:** This skill handles multi-portal tax audits, online form preparation (`08-MST`, `23/ĐK-TCT`, `24/ĐK-TCT`, `01/TB-ĐĐKD`, `01/TKN-CNKD`, `01/CNKD`, `02/CNKD-TNCN-QTT`, `01/BĐS` + `01/BK-BĐS`, `01/ĐNXLNT`, `01/BK-STK`, `02/QTT-TNCN`, `03/TK-SDĐPNN`), e-invoice revenue verification (`hoadondientu.gdt.gov.vn` or taxpayer's e-invoice software), legal threshold checks (`TVPL`), and Google Sheets tax tracking. Does **NOT** handle corporate CIT (`TNDN`), enterprise VAT deduction (`khấu trừ GTGT`), or automated tax payment execution.
- **No Auth / CAPTCHA / Signature Bypass:** NEVER solve CAPTCHAs, intercept OTPs, or click final Submit/Sign buttons (`Nộp tờ khai`, `Ký nộp`). Always prepare forms up to the final review/CAPTCHA screen and pause for the user.
- **No Secret Leakage:** NEVER store passwords, tokens, CCCD/OTP values, or raw cookies in skills, notes, or sheets.
- **Strict Grounding:** Report only figures, receipt IDs (`Mã hồ sơ`), and legal citations directly observed in portal responses, XMLs, or verified screenshots.

## Golden Rules

1. **Single-Tab Lock on `dichvucong.gdt.gov.vn`:** Use **one** browser session/tab on DVC at a time; parallel DVC tabs trigger `HTTP 403` session invalidation.
2. **Read-Only First, File Sequentially:** Audit `/tthc/tracuu-nvt/init` + `/tthc/tchs` XMLs + iCanhan before preparing any new filing.
3. **Verify Overpayment vs Debt:** Distinguish Column `(9) Còn phải nộp` (debt) from Column `(10) Nộp thừa / Tạm nộp` (overpayment credit) before concluding any tax is owed.
4. **Separate Legal Regimes by Year:** Apply `1 tỷ đồng/năm` exemption (**NĐ 141/2026/NĐ-CP** amending **NĐ 68/2026/NĐ-CP**) strictly from `01/01/2026` onward; keep `100 triệu/năm` rules for `2024–2025` unless explicitly instructed.

## Fast & Exact Portal Lookup Strategy (Minimal UI Clicks)

Follow this exact order to eliminate redundant clicks, avoid CAPTCHA fatigue, and achieve 100% dong-level accuracy:

1. **DOM-to-JSON Extraction Over Manual Pagination:** Never visually read or screenshot-scroll multi-page tax tables (e.g., 30+ debt rows). Run one `evaluate_script` to dump the table/modal data (`id`, `cqt`, `dbhc`, `chuong`, `tm`, `ky`, `han`, `soTien`) into JSON, then group by CQT and solve exact subsets offline in Python/JS.
2. **Lowest-Friction Lookup Order (`0` -> `1` CAPTCHA):**
   - **1st (`/tthc/tracuu-nvt/init` on DVC):** Pull complete Mục A debts + Mục I overpayments with exact `ID`s (`0` CAPTCHA once logged in).
   - **2nd (`canhan.gdt.gov.vn` wide sweep):** Query each `Tra cứu` menu once using the widest allowed span (`01/01/2015 – today`; `LPTB <= 1 year`) and retrieve ALL XMLs in one authenticated in-page `fetch()` loop (`§2.2`), avoiding per-row clicks and Chrome's multi-download block.
   - **3rd (`/tthc/tchs` on DVC — last):** Query ONLY the exact `<= 30-day` window or `Mã hồ sơ` needed for supplementary filings or receipt checks, minimizing user CAPTCHA prompts.
3. **Cap & Chunk Large `01/ĐNXLNT` Tables (`<= 25` rows/form):** Split >25 debt rows into sequential `01/ĐNXLNT` submissions (>40 cloned `<select>`/`selectpicker` rows create ~90k DOM nodes that crash Chrome DevTools with `connection closed`). Within each `<= 25`-row form, pre-compute exact `ID`s and partial amounts offline, batch-fill **`8–10` rows per `evaluate_script`** (larger chunks exceed the ~60s MCP timeout from ~1.5s/row synchronous `changeCQThu` AJAX + `selectpicker('refresh')`), and assert `ItongCot10` / `IItongCot13` / `soTienHoanTra` DOM totals before `btnNextStep`.

---

## Workflow 1: Full Tax, Debt & Filing Audit ("Còn nợ không & Đã nộp đúng chưa?")

1. **Check Debt vs Overpayment (`iCanhan` & `DVC`):**
   - On `canhan.gdt.gov.vn` obligations table: separate Column `(9) Còn phải nộp` (real debt, watch `4944` `0.03%/day` late interest accruing) from Column `(10) Nộp thừa / Tạm nộp` (credit — even if labeled `"Tiền chậm nộp"`).
   - On `dichvucong.gdt.gov.vn` `/tthc/tracuu-nvt/init`: extract complete **Mục A** (outstanding debts with `16`-digit `ID`, CQT, ĐBHC, Chương `557/857`, Tiểu mục `1602/1701/1003/4944`, Kỳ, Hạn nộp) and **Mục I** (eligible overpayments). Also check **`/tthc/tra-cuu-thongbao-cqt`** for any CQT tax notices, penalty decisions (`quyết định xử phạt`), or debt enforcement notices.
2. **Verify Past Filings & Payments Are Correct ("Đã nộp đúng chưa?"):**
   - Query ALL `canhan.gdt.gov.vn` audit menus (`01/01/2015 – today`): `Tra cứu thông báo`, `Tra cứu tờ khai`, **`Tra cứu chứng từ` (Giấy nộp tiền KBNN)**, `ĐKT`, `QTT`, `05TH`, `02TH`, `NPT`, `DKT` (`LPTB <= 1 year`). Fetch ALL XMLs in one in-page `fetch()` loop (`§2.2`).
   - Cross-check each payment receipt (`CQT thu`, `Chương`, `Tiểu mục`, `Kỳ thuế`, `ĐĐKD`) against accepted returns: catch **misrouted payments** (accepted in CQT A, cash paid to CQT B -> use `01/ĐNXLNT` cross-CQT offset + late-interest relief under `NĐ 252/2026` & `TT 89/2026`), **failed-ingest returns** (`"Hồ sơ đã chuyển vào hệ thống quản lý thuế không thành công"`, `§2.3`), and **return accuracy** (rental deposits `tiền cọc` wrongly taxed, contract date typos, wrong threshold `1 tỷ` vs `100 triệu`, MST Status `10`).
   - Inspect `/tthc/tchs` (ONLY for needed `<= 30-day` windows; pause for user CAPTCHA) for DVC-only submissions/receipts not mirrored on iCanhan. Archive XMLs/receipts into a dedicated personal Tax Drive folder (separate from unrelated asset/project folders).
3. **Legal Check & Post-Filing Verification (`+5 working days`):**
   - Cross-check legal obligations via `thuvienphapluat.vn` (`NĐ 141/2026/NĐ-CP`, `NĐ 68/2026/NĐ-CP`, `TT 89/2026/TT-BTC`).
   - Remember pending `01/ĐNXLNT` batches keep debts visible in Mục A until processed (`~05 working days`); re-check `/tthc/tracuu-nvt/init` + `/tthc/tra-cuu-thongbao-cqt` after processing to confirm `Mục A = 0 đ` and remaining ledger credits.

## Workflow 2: Registration & Overpayment Filings on DVC

Execute sequentially on the single DVC tab, pausing at `03/04 Nộp hồ sơ` / CAPTCHA / OTP for user signature:

1. **Đổi Tên / Ngành nghề / Thông tin HKD (`hokinhdoanh.dkkd.gov.vn` vs `08-MST`, `TT 86/2024/TT-BTC`, hạn `10 ngày làm việc`):**
   - **Nếu HKD ĐÃ CÓ GCN ĐKHKD** và thay đổi chỉ tiêu in trên GCN (`Tên HKD`, `Ngành nghề chính`, `Trụ sở`, `Vốn`, `Chủ hộ`): **KHÔNG nộp `08-MST` qua DVC Thuế** (CQT trả lỗi **`0525`** `hoSo_TDTTThayDoiGCN=true`). Nộp hồ sơ thay đổi qua **Cổng ĐKHKD quốc gia [`https://hokinhdoanh.dkkd.gov.vn`](https://hokinhdoanh.dkkd.gov.vn)** *(VNeID)* hoặc Một cửa UBND cấp xã/phường để tự liên thông sang Thuế.
   - **Chỉ dùng `08-MST` trên DVC Thuế** cho các thông tin **không nằm trên GCN** (hoặc cá nhân/HKD không có GCN ĐKKD). (`23/ĐK-TCT` tạm ngừng; `24/ĐK-TCT` chấm dứt MST sau khi `Mục A = 0 đ`).
2. **`01/TB-ĐĐKD` (`điểm đ Khoản 4 Điều 8 NĐ 68/2026/NĐ-CP`) — Register HKD Secondary Location (`ĐĐKD`):**
   - For **HKD/CNKD** business locations outside the head office (NOT mandated for individual asset rentals). Verify new 5-digit `ĐĐKD` code is issued in the registry.
3. **`01/BĐS` + `01/BK-BĐS` (`điểm d Khoản 3 & Khoản 4 Điều 8 NĐ 68/2026/NĐ-CP`; DVC `/tthc/tk01-bds-tt50`) — Official vs Supplementary (`Khai bổ sung`):**
   - Individuals with multiple properties may pick **01 CQT** to file combined `01/BĐS` (`2x/year` by `31/07` & `31/01`, or `1x/year` by `31/01` next year) and pay by location ID. If an official return already exists (`G12.18-YYMMDD-XXXXXXXX`), DVC locks `Loại tờ khai`: open `/tthc/tchs` (<= 30 days + user CAPTCHA) -> **`Nộp hồ sơ bổ sung`** -> carry forward all prior items + add new items (exclude refundable security deposits `tiền cọc` per earned-revenue rules in `Điều 3 & 4 NĐ 68/2026`).
   - **If DVC returns `500 TKhaiTypeIdResolver`:** switch to **`Nộp XML`** generated from **HTKK** (`references/dvc-bugs-and-workarounds.md` §1.4).
4. **`01/ĐNXLNT` (`Điều 37 & Điều 42 TT 89/2026/TT-BTC`; `điểm g Khoản 3 Điều 26 NĐ 252/2026/NĐ-CP`) — Refund / Cross-CQT Offset (`Nộp thừa`):**
   - Split submissions strictly **1 receiving CQT per form** (`arrCQThuByCqthue` lock in Mục II), prioritizing verified clean overpayments first. Mục III.1 (`tbtkhaiNT`) offsets nationwide debts across CQTs/chapters/sub-items (**điểm c Khoản 2 & c.2 Khoản 5 Điều 42 TT 89/2026**, exempt from `0.03%/day` interest on the offset amount from overpayment date) and allows partial debt-row amounts (`IIct13 <= IIct12`).
   - Save submitted debt `ID`s after every batch and **strictly exclude** them in subsequent batches. Set `soTienHoanTra = 0` to keep unused credit on the ledger (`"dư cứ để đó"`), or enter exact refund amount; **always fill `Chuyển khoản` (`Chủ TK`, `STK`, `Ngân hàng`) even at `0 đ`** because DVC hard-disables `Tiền mặt` (`§1.7`). Pause at Step 4 CAPTCHA/Submit and record `Mã hồ sơ`.
5. **`01/BK-STK` (`maToKhai=1265`, Sổ tay HKD Cục Thuế `03/2026`) — Bank Account Notification:**
   - Select CQT and `ĐĐKD`. **If `p01-select-cqt` returns `500` or locks radios to `00999`:** instruct user to submit via **eTax Mobile** (§1.5).

## Workflow 3: E-Invoice Revenue Verification (`hoadondientu.gdt.gov.vn` / HĐĐT)

1. Open the taxpayer's e-invoice portal (**`https://hoadondientu.gdt.gov.vn/`** — requires **user CAPTCHA login** — or the taxpayer's authenticated HĐĐT software).
2. Query issued invoices for `01/01/YYYY – DD/MM/YYYY`, strictly separating **valid issued (`Hợp lệ` / `Đã phát hành` / `Đã cấp mã`)** from unissued drafts (`Chưa phát hành`) or cancelled/replaced/adjusted invoices (`Hủy / Thay thế / Điều chỉnh`).
3. Cross-verify total pre-tax & total revenue across invoice list and summary/monthly reports until numbers match `100%`.
4. Combine with any secondary revenue streams to test against the **`1 tỷ đồng/năm`** threshold (`Khoản 1 Điều 1 NĐ 141/2026/NĐ-CP`): `<= 1 tỷ -> 0 đ` VAT/PIT, submit **`01/TKN-CNKD`** by **`31/01` next year** (`TT 89/2026/TT-BTC`, `điểm a Khoản 1 Điều 8 NĐ 68/2026`; use **Mục E** for platform over-withholding). Above `1 tỷ`, VAT applies on total taxable revenue (`điểm c Khoản 2 Điều 12 Luật GTGT 48/2024/QH15`) while PIT applies ONLY on revenue exceeding `1 tỷ` (`Khoản 3 & 4 Điều 7 VBHN 112/VBHN-VPQH`).

## Workflow 4: Google Sheets Tax Dashboard, Ledger & Annual Obligation Checklist

1. **Never overwrite existing tracking tabs.** Use a **3-tab multi-taxpayer architecture** so adding a new transaction or person never requires creating extra tabs:
   - **`Thuế - Tổng hợp` (1 Dynamic Dashboard for ALL taxpayers):** Top selector cell (e.g., `C3` dropdown: `Người nộp thuế A`, `Người nộp thuế B`, `Tất cả`) dynamically drives Taxpayer Profile (`XLOOKUP`), 4 KPI cards (`IF($C$3="Tất cả", SUM(...), SUMIFS(...))`), Year/Category Summary, active Todo table (`FILTER`), and **Annual Obligation Checklist** (`FILTER`).
   - **`Thuế - Phát sinh chi tiết` (1 Unified Master Ledger):** Taxpayer profile table + ĐĐKD/property registry + unified transaction ledger (`$A$20:$L$500`) starting every row with an **`Đối tượng`** dropdown pill, followed by `Năm`, `Ngày / Kỳ`, `Phân loại` dropdown pill, `Nguồn / CQT / Tờ khai`, `Mã ĐĐKD / Tiểu mục`, `Doanh thu`, `Thuế GTGT`, `Thuế TNCN`, `Nợ thuế Mục A`, `Đã nộp / Nộp thừa`, and `Ghi chú`.
   - **`Thuế - Nghĩa vụ & Todo` (Unified Checklist & Annual Obligations Source):** Stores interactive Checkboxes + **`Đối tượng`** dropdown for both one-off Todo actions and recurring annual obligations per taxpayer so `Thuế - Tổng hợp` filters them cleanly without formula/checkbox collisions.
2. **Generate an Annual Obligation Checklist per Taxpayer** derived from their active obligations (each row cites its exact legal basis):
   - **HKD (`<= 1 tỷ đồng/năm`):** **`01/TKN-CNKD`** (`TT 89/2026/TT-BTC`, `điểm a Khoản 1 Điều 8 NĐ 68/2026/NĐ-CP` sửa bởi `NĐ 141/2026`), chốt doanh thu thực tế năm trước hạn **`31/01` năm sau** (kèm `Mục E` nếu hoàn thuế sàn TMĐT khấu trừ thừa; lưu **`Mẫu S1a-HKD` `TT 152/2025/TT-BTC`** tối thiểu 05 năm).
   - **HKD (`> 1 tỷ đồng/năm`):** khai thuế theo quý/tháng (`Điều 8 & 10 NĐ 68/2026/NĐ-CP`) và quyết toán **`02/CNKD-TNCN-QTT`** chậm nhất **`31/03` năm sau** nếu tính theo thu nhập tính thuế.
   - **Cho thuê tài sản (`BĐS`, nếu phát sinh):** **`01/BĐS` + `01/BK-BĐS`** (`điểm d Khoản 3 & 4 Điều 8 NĐ 68/2026/NĐ-CP`), chọn 2 lần/năm (`31/07` & `31/01`) hoặc 1 lần/năm (`31/01` năm sau).
   - **Tiền lương / tiền công (cá nhân tự quyết toán):** **`02/QTT-TNCN`** chậm nhất **`30/04` năm sau** (`08/UQ-QTT-TNCN` nếu ủy quyền trước `31/03`, theo `Điều 10 NĐ 252/2026/NĐ-CP`, `NQ 110/2025/UBTVQH15`).
   - **Đất phi nông nghiệp (`557/1602`, nếu có):** nộp tiền hằng năm chậm nhất **`31/10`**; khai tổng hợp **`03/TK-SDĐPNN`** chậm nhất **`31/03` năm sau** nếu tổng diện tích đất ở vượt hạn mức (`Khoản 1 Điều 25 TT 89/2026/TT-BTC`).
   - **Thay đổi / Bù trừ / Hoàn nộp thừa:** `08-MST` (`TT 86/2024`, hạn `10 ngày`), `01/TB-ĐĐKD` (`điểm đ Khoản 4 Điều 8 NĐ 68/2026`, cho ĐĐKD ngoài trụ sở của HKD), `01/BK-STK`, **`01/ĐNXLNT`** (`Điều 37 & 42 TT 89/2026`, xử lý trong `5 ngày làm việc`).
   - **Kiểm tra định kỳ cuối / đầu năm & hậu kiểm (`+5 ngày làm việc`):** rà soát `iCanhan` Cột `(9)/(10)`, `DVC` `/tthc/tracuu-nvt/init`, `/tthc/tra-cuu-thongbao-cqt`, và đối chiếu doanh thu HĐĐT với ngưỡng `1 tỷ đồng/năm`.
3. Whenever a filing is submitted or hits a DVC `500` error, update the exact `Mã hồ sơ` or error/workaround note in **`Thuế - Tổng hợp`** / **`Thuế - Nghĩa vụ & Todo`**, verify `SUM`/`SUMIFS` ranges (`$20:$500`), wait for **`Saved to Drive`**, and inspect screenshots for `#REF!` or clipping.

## References

- Detailed DVC / iCanhan / HĐĐT / Google Sheets / MCP bug catalog & workarounds: [`references/dvc-bugs-and-workarounds.md`](references/dvc-bugs-and-workarounds.md)
- Verified 2026 VN personal, HKD, multi-property & cross-CQT offset legal rules (TVPL): [`references/vn-tax-2026-rules.md`](references/vn-tax-2026-rules.md)
