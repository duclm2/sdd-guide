[← /generate-prd](02-generate-prd.md) · [Explain Home](README.md) · [Next: /amend-prd →](02c-amend-prd.md)

# 02b · `/extend-prd` — Thêm yêu cầu vào PRD đã duyệt

> **Một câu.** Thêm UC/AC/BR mới vào một PRD **đã ship**, đánh số **nối tiếp** (không bao giờ đánh lại), ghi **add-only** kèm guard sau-ghi, và drain hàng đợi `feedback/prd-change-requests/`.

---

## Vấn đề giải quyết

Đây là bước **xảy ra nhiều nhất sau khi sản phẩm đã sống** — và trước v0.4.3 là bước **duy nhất trong toàn pipeline không có lệnh**.

Ba lệnh chạm PRD đều không dùng được:

| Lệnh | Vì sao không |
|---|---|
| `/generate-prd` | Sinh PRD **mới** từ discovery. **Ghi đè** bản cũ, không có gì chặn |
| `/refine-prd` | Chỉ áp findings từ review, và **tự cấm** đụng section ngoài findings. Findings sinh từ việc soi PRD hiện có → **không có đường nào để một yêu cầu MỚI đi vào** |
| `/define-product` | Bắt đi hết **cả 7 phase** mọi lần; resume chỉ chạy khi `Completed Phase < 7` nên file `completed` không có gì để resume |

Nên PM phải làm tay **5 bước, đều là contract, đều không ai kiểm** — trong đó bước viết dòng changelog là contract **thật**: `/generate-bdd` đọc nó để quyết cập nhật hẹp hay gen lại toàn bộ, và `/validate-traces` đọc nó để lọc báo động oan.

Đây là chỗ framework **thôi bảo vệ bạn** — đúng lúc rủi ro cao nhất: sửa tài liệu đã ký duyệt của một feature đang chạy production.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Specification — nhánh *"PRD đã tồn tại"*.
- **Tiền đề:** có PRD. Không có → dùng `/generate-prd` (feature mới đi từ discovery).
- **Song hành:** `/generate-prd` giờ **từ chối chạy** trên PRD đã có và chỉ sang đây.

---

## Input / Output

**Input:** file PRD + `feedback/prd-change-requests/*.md` (tuỳ chọn) + PO.

**Output:** PRD `v+1` với UC/AC/BR **nối tiếp**, `Status → draft`, row changelog nêu rõ scope; request đã xử → `archived/` + `Status: incorporated`.

---

## Các bước xử lý (chi tiết)

### Bước 1 — Nạp trạng thái
`max_uc` · `max_br` · `max_ac` · danh sách UC hiện có · changelog · `bdd_generated`. Hai guard mềm: PRD đang `draft` (trộn hai việc) · BDD đã sinh (nêu rõ liên kết cũ **không** bị ảnh hưởng vì chỉ đánh số nối tiếp).

### Bước 2 — Drain hàng đợi
Quét `prd-change-requests/`, lọc theo TICKET-ID.

| Status | Xử lý |
|---|---|
| `accepted` | Thành **nguyên liệu** cho Bước 3 — **không** chèn thẳng vào PRD (yêu cầu nghiệp vụ phải qua PO chốt AC/BR đúng tầng) |
| `Open` | Trình PO ở CHECKPOINT kèm **số ngày chờ**; PO chọn từng cái |
| `rejected` / `incorporated` | Bỏ qua |

### Bước 3 — Discovery delta
Tái dùng Phase 1/4/5/6 của `/define-product` (bỏ Phase 0/2/7 vốn là toàn-feature), **cộng phase kiểm va chạm** — phase mà `/define-product` không có, vì lúc discovery lần đầu chưa có gì để va chạm:

| Câu hỏi | Kết quả |
|---|---|
| Phần thêm làm một BR cũ **sai đi** không? | → đây là **sửa** BR cũ, không phải thêm mới → bump **major** |
| Đã có UC/AC nào **phủ một phần** chưa? | → hỏi PO: mở rộng UC cũ hay tạo mới. **Không tự quyết** |
| Cần dữ liệu/năng lực từ đâu khác? | → §1c Phụ thuộc liên service |

Vẫn áp **Discovery Contract**: input của PO (kể cả nội dung request) là **nguyên liệu thô**, không phải câu trả lời thay phỏng vấn.

### Bước 4 — Đánh số nối tiếp
`UC{max+1}` · `BR{max+1}` *(tính trên **toàn PRD**, không reset theo UC)* · `AC{max+1}`.

### Bước 5 — Ghi add-only
Đọc lại file ngay trước khi ghi · **cấm Write cả file** · output là **superset chặt** · **guard sau-ghi** đối chiếu mọi UC/AC/BR/changelog row cũ còn nguyên, mất cái nào → **khôi phục + dừng**.

### Bước 6 — Bump + changelog
Tái dùng **nguyên** `/refine-prd` Phase 3: bump · `Status → draft` · row changelog · rollover >5 row sang `changelog/`.

### Bước 6.5 — Đóng dấu request
`Status: incorporated` + `Incorporated into: v{new}` + chuyển `archived/` + commit spec repo. Request PO **không** chốt → giữ `Open`, `/validate-traces` tiếp tục nhắc.

---

## Checkpoint & Gate

- 🛑 Nhãn khai **chặn CỨNG** (sửa PRD đã duyệt) — `--yes` **không** bỏ qua được.
- ⚠️ PRD đang chưa `approved` → cảnh báo mềm, hỏi `Y/N` (không chặn).
- 🛑 **CHECKPOINT trước khi ghi** — hiện: PRD hiện tại · phần thêm · **cái cũ bị sửa** · request được đưa vào · version mới · **BDD ảnh hưởng**.
- 🛑 **Guard sau-ghi** — mất bất kỳ ID cũ nào là **chặn cứng**, khôi phục file.
- `Status` reset về `draft` → bắt buộc qua `/review-context` + PO duyệt lại. Không có đường đi tắt.

---

## Cơ chế đặc biệt

### Hai ràng buộc cứng

**① Không đánh lại BẤT KỲ ID cũ nào.** Đánh lại — kể cả "cho gọn" — sẽ phá `@trace.business_rules` trong mọi `.feature` đã sinh, dòng "AC liên quan" của từng UC, và mọi cross-reference từ PRD khác trỏ tới AC/BR cụ thể.

Số bị bỏ trống (do UC bị xoá ở version trước) **để trống vĩnh viễn** — ID đã từng tồn tại có thể còn bị tham chiếu ở BDD, code, bug report, hoặc PRD khác.

**② Dòng changelog phải nêu rõ UC/AC/BR.**

| | |
|---|---|
| ✅ **Đúng** | `thêm UC7 (xuất nhiều file): AC12-AC14, BR21-BR23; UC3: sửa BR8 (nâng giới hạn 5→20)` |
| ❌ **Sai — mơ hồ** | `cập nhật theo yêu cầu mới` |
| ❌ **Sai — BR không kèm UC sở hữu** | `…; sửa BR8 (nâng giới hạn 5→20)` (thiếu `UC3:`) |

Dòng mơ hồ làm **mất cả hai** bộ lọc cùng lúc: `/generate-bdd` khuyến nghị gen lại **toàn bộ**, **và** `/validate-traces` gắn `PRD_DRIFT` 🟠 cho **mọi** UC thay vì chỉ UC mới — lỗi có cảnh báo.

Ca thiếu UC sở hữu còn nguy hiểm hơn — là lỗi ngầm: `/validate-traces` khớp theo *"UC này có trong tập bị ảnh hưởng?"*, UC3 không có trong tập nên bị xếp ⓘ `PRD_STALE_REF` dù BR8 vừa đổi hành vi; rồi `--realign-prd-version` (chỉ từ chối UC 🟠) dán lại nhãn, che mất thay đổi chưa ai implement. Vì vậy mỗi mệnh đề **phải mở đầu bằng UC sở hữu** — thành contract `changelog_row_contract` trong `bin/trace-schema.json`; nội dung không thuộc UC nào ghi `PRD-global`.

### Vì sao là lệnh riêng, không phải `/generate-prd --extend`

Hai chế độ **ngược nhau về thao tác ghi**: `/generate-prd` **Write cả file**, lệnh này **chỉ Edit add-only**. Trộn vào một file là một file hai hành vi — người đọc chọn nhầm.

---

## 👓 Góc nhìn tối ưu

- **Bề mặt mới lớn nhất, nhưng phần mới thật chỉ có 4:** nạp trạng thái · drain hàng đợi · đánh số nối tiếp · kỷ luật add-only. Gate, context-loader, business-language guard, discovery phases, version-bump, report-footer đều **tái dùng** thứ đã chạy.
- **Kỷ luật add-only copy nguyên từ `/generate-code`** — cùng bài toán (thêm vào artifact chung mà không xoá phần của người khác), nên copy giải pháp đã kiểm chứng thay vì phát minh lại.
- **Chất lượng changelog là điểm yếu duy nhất** — nó quyết định chất lượng của hai bộ lọc downstream. Đáng đo: dòng changelog thực tế có nêu đủ scope không?
- **Ranh giới với `/refine-prd`** dễ lẫn nếu không đọc header: *"PRD có **vấn đề** gì"* vs *"PRD **thiếu** cái gì mới"*. Và với [`/amend-prd`](02c-amend-prd.md): lệnh này chỉ THÊM (cửa sửa BR cũ chỉ mở khi phần thêm làm BR cũ sai); muốn **đổi** một yêu cầu mà không thêm gì mới → `/amend-prd`.

---

## Kết nối

**Trước:** [`/propose-scenario`](26-propose-scenario.md) Case B (ghi request) · hoặc PO trực tiếp.
**Sau:** [`/refine-prd`](03-refine-prd.md) → [`/review-context`](04-review-context.md) → PO duyệt → [`/generate-bdd`](06-generate-bdd.md) **cho UC MỚI và UC cũ bị sửa BR/AC** (báo cáo tách riêng danh sách "UC BỊ SỬA nội dung" — cấm `--realign` cho chúng); chỉ UC cũ **không đổi gì** mới dùng `/validate-traces --realign-prd-version {UC-ID}`.
