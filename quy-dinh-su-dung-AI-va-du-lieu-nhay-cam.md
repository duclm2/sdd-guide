# QUY ĐỊNH SỬ DỤNG CÔNG CỤ AI VỚI DỮ LIỆU VÀ HỆ THỐNG NỘI BỘ

**Áp dụng cho:** Toàn bộ nhân sự phòng Phát triển (Dev, QA, DevOps, Data)
**Phiên bản:** 1.0 — Ngày hiệu lực: `[điền ngày]`
**Người ban hành:** `[Trưởng phòng / CTO]`

---

## 1. Mục đích

Công cụ AI (Claude, ChatGPT, Copilot, Cursor, Gemini, các AI agent qua MCP…) là công cụ làm việc được khuyến khích sử dụng. Quy định này không nhằm hạn chế việc dùng AI, mà nhằm xác định **ranh giới rõ ràng về dữ liệu nào được đưa vào AI và quyền truy cập nào AI được phép có**, để tránh:

- Rò rỉ thông tin xác thực (password, API key, token, private key).
- Rò rỉ dữ liệu khách hàng / dữ liệu cá nhân (PII).
- Thao tác phá hoại hoặc thay đổi ngoài ý muốn trên hệ thống production.
- Vi phạm nghĩa vụ bảo mật trong hợp đồng với khách hàng và quy định pháp luật về bảo vệ dữ liệu cá nhân.

## 2. Phân loại dữ liệu

| Mức | Loại dữ liệu | Được đưa vào AI? |
|---|---|---|
| **Đỏ** | Password, API key, secret, token, private key, chuỗi kết nối DB, thông tin VPN, host/IP/port của server production, nội dung file `.env`, `.pem`, `kubeconfig`, credential cloud (AWS/GCP/Azure) | **KHÔNG** — trong mọi trường hợp |
| **Cam** | Dữ liệu thật của khách hàng: CMND/CCCD, số điện thoại, email, địa chỉ, thông tin thanh toán, hồ sơ y tế, log chứa PII, dump/backup DB production | **KHÔNG** — trừ khi đã ẩn danh/masking và được `[Trưởng phòng]` phê duyệt |
| **Vàng** | Source code nội bộ, sơ đồ kiến trúc, DB schema (chỉ cấu trúc), tài liệu thiết kế nội bộ | **CÓ** — chỉ trên công cụ AI đã được duyệt (mục 4) |
| **Xanh** | Code mẫu, dữ liệu giả/synthetic, thông tin công khai, câu hỏi kỹ thuật chung | **CÓ** — tự do |

**Nguyên tắc vàng:** Nếu bạn phải cân nhắc "cái này có nhạy cảm không?" thì mặc định coi là nhạy cảm và hỏi trước khi gửi.

## 3. Các hành vi bị NGHIÊM CẤM

1. **Cấp cho AI quyền truy cập trực tiếp vào database production** — dưới mọi hình thức: MCP server, DB connector, agent tự chạy query, script do AI điều khiển.
2. **Dán credential vào chat AI** — kể cả để "nhờ AI debug lỗi kết nối". Hãy thay bằng giá trị giả: `DB_PASSWORD=<redacted>`.
3. **Upload file cấu hình hoặc dump dữ liệu** chứa dữ liệu mức Đỏ/Cam lên công cụ AI.
4. **Copy kết quả query chứa dữ liệu thật của khách hàng** vào AI để "nhờ phân tích".
5. **Cho AI agent quyền ghi (INSERT/UPDATE/DELETE/DDL)** trên bất kỳ database nào chứa dữ liệu thật.
6. **Dùng AI tự động chạy lệnh trên server** (SSH, kubectl, terraform apply…) mà không có người review từng lệnh trước khi thực thi.
7. **Dùng công cụ AI cá nhân, tài khoản cá nhân, hoặc bản miễn phí** cho công việc liên quan đến dữ liệu mức Vàng trở lên.

## 4. Công cụ AI được phê duyệt

Chỉ sử dụng công cụ AI trong danh sách sau, và **phải đăng nhập bằng tài khoản do công ty cấp**:

| Công cụ | Mục đích được duyệt | Ghi chú |
|---|---|---|
| `[Claude – gói Team]` | Code, tài liệu, phân tích | Được cấp theo danh sách - Khối PTSP do Ms. TrangNT46 quản lý việc cấp phát tài khoản|
| `[Gemini – gói Enterprise]` | Code, tài liệu, phân tích | Được cấp theo danh sách - P.HTHT do Mr. TrungNT4 quản lý việc cấp phát tài khoản|

**Yêu cầu kỹ thuật với mọi công cụ được duyệt:**
- Dùng gói doanh nghiệp có cam kết **không dùng dữ liệu để huấn luyện** và có kiểm soát thời gian lưu trữ.
- Tài khoản quản lý tập trung theo emails/seats, thu hồi được khi nhân sự nghỉ việc.
- Không cài extension/MCP server ngoài danh sách whitelist do DevOps duyệt.

Muốn dùng công cụ mới → gửi yêu cầu cho `[P. HTHT]` để đánh giá trước khi dùng.

## 5. Con đường an toàn (thay thế cho hành vi bị cấm)

Quy định chỉ hiệu quả khi có cách làm thuận tiện tương đương. Phòng sẽ cung cấp:

| Nhu cầu thực tế | Cách làm đúng |
|---|---|
| Cần AI hiểu dữ liệu để viết query | Cung cấp **schema + dữ liệu mẫu giả** (10–20 dòng synthetic), không dùng dữ liệu thật |
| Cần AI truy vấn DB để debug | Dùng **DB dev/staging đã masking**, tài khoản **read-only**, không chứa PII |
| Cần AI đọc log lỗi | Chạy log qua bộ **scrub/redact** trước khi dán (che token, email, số điện thoại) |
| Cần credential trong quá trình dev | Lấy từ **secret manager** (`[Vault / AWS Secrets Manager / 1Password]`), không hardcode, không copy vào chat |
| Cần AI review code có config | Dùng file `.env.example` với giá trị giả |

Trách nhiệm chuẩn bị: `[P. HTHT]` cung cấp DB staging đã masking và pipeline sinh dữ liệu giả trước ngày `[điền deadline]`.

## 6. Kiểm soát kỹ thuật bắt buộc

`[P. HTHT]` triển khai và duy trì:

1. **Quét secret:** cài `gitleaks` hoặc `trufflehog` ở pre-commit hook và CI pipeline. Pipeline **fail** nếu phát hiện secret.
2. **Audit log:** bật audit log trên DB production và lưu tối thiểu `[90]` ngày.
3. **Kiểm tra cấu hình AI/MCP:** rà soát định kỳ `[hàng tháng]` các file cấu hình MCP, extension IDE, và integration của AI agent trên máy nhân viên.
4. **Chặn `.env` khỏi git:** `.gitignore` chuẩn hóa cho mọi repo, kèm kiểm tra tự động.

## 7. Xử lý sự cố rò rỉ

Nếu bạn (hoặc đồng nghiệp) đã đưa dữ liệu mức Đỏ/Cam vào AI — **kể cả khi nghĩ là vô hại**:

1. **Báo ngay** cho `[Trưởng phòng / kênh #security]` trong vòng **1 giờ**. Không tự xử lý im lặng.
2. **Xoay (rotate) toàn bộ credential đã bị phơi nhiễm** ngay lập tức — coi như đã bị compromise, không cần chờ bằng chứng bị lạm dụng.
3. Xóa cuộc hội thoại trên công cụ AI (lưu ý: xóa chat **không** đảm bảo dữ liệu đã bị xóa khỏi mọi bản lưu trữ — vì vậy bước 2 là bắt buộc, không phải tùy chọn).
4. `[P. HTHT]` rà soát audit log để đánh giá phạm vi ảnh hưởng và lập báo cáo trong `[3]` ngày làm việc.

**Chính sách không trừng phạt khi báo cáo trung thực:** người tự giác báo cáo sự cố sẽ không bị xử lý kỷ luật. Việc che giấu sự cố mới là hành vi bị xử lý.

## 8. Chế tài

| Mức | Hành vi | Xử lý |
|---|---|---|
| Lần 1 | Vi phạm do chưa nắm rõ, không gây rò rỉ dữ liệu khách hàng | Nhắc nhở + đào tạo lại, ghi nhận nội bộ |
| Lần 2 | Tái phạm | Khiển trách bằng văn bản, ảnh hưởng đánh giá hiệu suất |
| Nghiêm trọng | Cố ý, hoặc gây rò rỉ dữ liệu khách hàng / mất mát dữ liệu production | Xử lý kỷ luật theo nội quy công ty, có thể chấm dứt hợp đồng và xem xét trách nhiệm pháp lý |

## 9. Đào tạo và cam kết

- Buổi phổ biến quy định: `[ngày]`, bắt buộc tham dự.
- Toàn bộ nhân sự phòng ký xác nhận đã đọc và hiểu quy định (mục 10).
- Nhân sự mới: ký cam kết trong tuần đầu onboarding.
- Rà soát và cập nhật quy định: `[6 tháng/lần]` hoặc khi có công cụ AI mới.

## 10. Xác nhận

Tôi đã đọc, hiểu và cam kết tuân thủ quy định này.

| Họ tên | Vị trí | Ngày | Ký |
|---|---|---|---|
| | | | |
| | | | |

---

*Thắc mắc về quy định hoặc cần phê duyệt ngoại lệ: liên hệ `[tên/kênh liên hệ]`.*
