# `etax-canhan-hkd` — AI Skill Thuế Cá Nhân & Hộ Kinh Doanh (Việt Nam 2026)

Skill hướng dẫn AI Agent (Antigravity, Claude Code, Cursor, Codex...) hỗ trợ **kiểm tra nợ thuế & tiền nộp thừa, đối chiếu tờ khai/chứng từ đã nộp, tải toàn bộ XML lịch sử, lập hồ sơ `01/ĐNXLNT`, `01/TKN-CNKD`, `01/BĐS`, `01/TB-ĐĐKD`, `08-MST`, `02/QTT-TNCN`**, kiểm tra doanh thu **Hóa đơn điện tử (`hoadondientu.gdt.gov.vn` / phần mềm HĐĐT)**, và tổng hợp **Google Sheets 3 tab** theo quy định thuế Việt Nam 2026 (**NĐ 141/2026/NĐ-CP**, **NĐ 68/2026/NĐ-CP**, **TT 89/2026/TT-BTC**).

---

## 1. Checklist chuẩn bị trước khi bắt đầu (Bắt buộc)

Trước khi nhờ AI Agent chạy quy trình, người dùng cần chuẩn bị sẵn **5 thứ** bên cạnh:

1. **Điện thoại đã cài VNeID (định danh mức 2):** dùng để tự đăng nhập vào `https://dichvucong.gdt.gov.vn/tthc/homelogin` và `https://canhan.gdt.gov.vn` (AI không tự đăng nhập được).
2. **Số điện thoại đăng ký thuế (nhận SMS OTP):** giữ điện thoại bên cạnh vì khi ký nộp **mỗi tờ khai** ở Bước 4 DVC đều đòi **tự nhập CAPTCHA + mã OTP SMS** gửi về số này. *(Cài thêm app **eTax Mobile** nếu cần nộp `01/BK-STK` khi web DVC lỗi 500.)*
3. **Thông tin tài khoản ngân hàng cá nhân chính chủ (trùng tên MST/CCCD):**
   - **Số tài khoản (STK)** + **Tên ngân hàng** + **Tên chủ tài khoản (`VIẾT HOA KHÔNG DẤU`)**.
   - **Bắt buộc phải có** nếu làm `01/ĐNXLNT` (hoàn/bù trừ nộp thừa), **kể cả khi chỉ bù trừ `Hoàn trả = 0 đ`**, vì cổng DVC khóa cứng mục `Tiền mặt`.
4. **Quyết định trước phương án xử lý tiền thừa (nếu có nộp thừa ở Mục I / Cột 10):** chọn **hoàn tiền về STK ngân hàng** hay **chỉ bù trừ sạch nợ Mục A, phần dư `Hoàn trả = 0 đ` giữ lại trên sổ thuế**.
5. **Tài liệu nguồn (tùy thủ tục cần làm):** hợp đồng/chứng từ phát sinh thu nhập, quyền truy cập cổng/phần mềm **Hóa đơn điện tử** (`hoadondientu.gdt.gov.vn`), **Google Drive/Sheets** (nếu muốn AI lưu trữ XML & lập sổ theo dõi), và phần mềm **HTKK** bản mới nhất (chỉ cần nếu nộp bổ sung qua `Nộp XML` khi web DVC lỗi `500`).

---

## 2. Các nút thắt bắt buộc người dùng thao tác (VNeID + Tự nhập CAPTCHA / OTP)

> **Quan trọng:** Cổng Thuế (`dichvucong.gdt.gov.vn`, `canhan.gdt.gov.vn`, `hoadondientu.gdt.gov.vn`) yêu cầu xác thực **VNeID** và có **CAPTCHA chặn tự động**. AI Agent không tự giải CAPTCHA hay tự ký nộp — hệ thống sẽ tạm dừng đúng các điểm sau để bạn thao tác trực tiếp:

| Điểm dừng bắt buộc có người | Bạn tự làm gì | Lưu ý để không mất phiên |
|---|---|---|
| **1. Đăng nhập ban đầu (`VNeID`)** | Mở `https://dichvucong.gdt.gov.vn/tthc/homelogin` (và `https://canhan.gdt.gov.vn`), chọn **Đăng nhập bằng Định danh điện tử (VNeID)** → quét QR / xác nhận OTP trên app VNeID | Chỉ mở **01 tab DVC** (mở tab thứ 2 sẽ văng `HTTP 403`) |
| **2. Tra cứu hồ sơ đã nộp (`/tthc/tchs`)** | Tự gõ **mã CAPTCHA** trên màn hình tra cứu (mỗi lần truy vấn tối đa `30 ngày`) | Chỉ dùng khi cần lấy biên lai `Mã hồ sơ` hoặc bấm `Nộp hồ sơ bổ sung`; dữ liệu nhiều năm kéo ở `iCanhan` để đỡ gõ CAPTCHA |
| **3. Tra cứu / đăng nhập `hoadondientu.gdt.gov.vn`** | Tự gõ **CAPTCHA** đăng nhập cổng HĐĐT (hoặc mở sẵn phần mềm HĐĐT bạn đang dùng) | Agent chỉ đọc/lọc danh sách hóa đơn hợp lệ sau khi bạn đã vào trang |
| **4. `Bước 4: Nộp hồ sơ` của MỖI tờ khai** | Kiểm tra lại tổng số liệu Agent đã điền sẵn → tự gõ **CAPTCHA** + bấm **`Tiếp tục`** + nhập **OTP ký nộp** | Nếu tách nhiều bộ `01/ĐNXLNT` (`<= 25` dòng/bộ), bạn sẽ nhập CAPTCHA + OTP cho **từng bộ** |
| **5. Thủ tục DVC đang lỗi `500`** | Khi khai bổ sung trực tuyến dính `500` → tự kết xuất XML trên **HTKK**; `01/BK-STK` dính `500` → tự nộp trên **eTax Mobile** | Không bấm thử lại vô hạn trên DVC |

---

## 3. Cài đặt Skill cho AI Agent

### Cách A — Cài nhanh bằng `git clone`
```bash
# Cho Antigravity / Claude Code (toàn cục trên máy)
git clone https://github.com/nth1306/etax-canhan-hkd.git ~/.claude/skills/etax-canhan-hkd
```
Hoặc copy nguyên thư mục chứa `SKILL.md` và `references/` vào thư mục `.agent/skills/etax-canhan-hkd/` trong workspace của bạn.

### Cách B — Gửi thẳng link GitHub cho Agent tự cài
> *"Đọc và cài skill `etax-canhan-hkd` từ `https://github.com/nth1306/etax-canhan-hkd` vào thư mục skills giúp mình."*

---

## 4. Các câu lệnh mẫu để dùng ngay

Sau khi bạn **đã đăng nhập xong qua VNeID**, nhắn Agent theo một trong các mẫu sau:

- **Kiểm tra tổng hợp còn nợ thuế & đã nộp đúng chưa (ít CAPTCHA nhất):**
  > *"Mình đã đăng nhập VNeID trên DVC và iCanhan rồi. Dùng skill `etax-canhan-hkd` kiểm tra toàn bộ Mục A (nợ thuế), Mục I (nộp thừa), Cột (9)/(10) và tải hết thông báo/tờ khai/chứng từ từ 2015 đến nay để kiểm tra xem mình còn nợ hay đã nộp đúng chưa."*
- **Bù trừ nợ chéo CQT / Hoàn thuế nộp thừa (`01/ĐNXLNT`):**
  > *"Mình muốn dùng khoản nộp thừa ở [Tên CQT nộp thừa] để bù trừ toàn bộ nợ thuế ở Mục A (hoặc hoàn về STK [Số TK] - Ngân hàng [Tên NH] - Chủ TK [Tên không dấu]). Dư giữ nguyên trên sổ (`Hoàn trả = 0 đ`), chia nhỏ `<= 25` dòng/tờ khai và dừng ở Bước 4 để mình tự nhập CAPTCHA + OTP ký nộp."*
- **Khai hoặc cập nhật hồ sơ thuế cá nhân / HKD (`01/TKN-CNKD`, `01/BĐS`, `01/TB-ĐĐKD`, `08-MST`):**
  > *"Kiểm tra lại thông tin MST / ĐĐKD và các tờ khai hiện có của mình, chuẩn bị hồ sơ [Tên mẫu] tới bước nhập CAPTCHA/ký nộp."*
- **Kiểm tra ngưỡng doanh thu `1 tỷ đồng/năm` (HĐĐT + kinh doanh):**
  > *"Đối chiếu doanh thu hóa đơn hợp lệ trên cổng/phần mềm HĐĐT với các nguồn doanh thu khác để kiểm tra xem có vượt ngưỡng 1 tỷ/năm (NĐ 141/2026) không và cần nộp tờ khai gì."*

---

## 5. Cấu trúc Skill

```text
etax-canhan-hkd/
├── README.md                              # Hướng dẫn cài đặt, VNeID, checklist chuẩn bị & các điểm tự nhập CAPTCHA
├── SKILL.md                               # Quy trình chuẩn, Golden Rules & fast lookup
└── references/
    ├── dvc-bugs-and-workarounds.md        # Lỗi thực tế DVC/iCanhan/HĐĐT/Chrome/Sheets & cách né
    └── vn-tax-2026-rules.md               # Tổng hợp luật thuế 2026 (NĐ 141, NĐ 68, TT 89)
```
