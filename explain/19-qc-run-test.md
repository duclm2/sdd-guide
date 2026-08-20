[← /qc-review](18-qc-review.md) · [Explain Home](README.md) · [Next: /qc-report →](20-qc-report.md)

# 19 · `/qc-run-test` — Trạm 5: Sinh & chạy Playwright, ghi `qc_status`

> **Một câu.** Biến `.Test.md` đã review thành **Python pytest-playwright**, chạy thật, rồi ghi **`qc_status` chính thức** (có evidence) vào trace TSV.

---

## Vấn đề giải quyết

Đây là nơi QC trở thành **chính thức**: chạy test thật trên Playwright, phân loại FAIL (script-bug vs product-gap, **không fake-pass**), và đóng dấu `qc_status` — trạng thái QC authoritative.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 5), sau `/qc-review` (case APPROVED).
- **Stack:** module `qc-playwright` (Python + pytest-playwright + Page Object) — **độc lập** module dev.

---

## Input / Output

**Input:** `.Test.md` đã review + skill `qa-runner` + bảng Test Selectors §4.5.6 (**giá trị** test-id, từ `/map-testids`) + **`@trace.testid_attr`** ở header tech-doc (**tên thuộc tính** chứa chúng).

**Output:** script Python + kết quả + cột `qc_status` trong `.trace/…/{UC-ID}-{platform}.tsv` + panel mirror (`.trace-mirror/`).

> **`@trace.testid_attr` — đọc, KHÔNG suy từ platform.** §4.5.6 cho **giá trị** test-id; field này cho **tên thuộc tính** chứa chúng. `get_by_test_id()` của Playwright mặc định dò `data-testid` **nhưng cấu hình được** — dự án dùng `data-test`/`data-qa` thì phải `set_test_id_attribute("{attr}")` trước, không thì **trượt 100% locator**.
>
> Suy từ platform là **phát biểu lại một sự thật đã ghi ở nơi khác** (`/map-testids` đã phân giải một lần cho cả feature, `/generate-code` đọc chính field đó để emit). Và nó hỏng **im lặng theo kiểu tệ nhất**: test fail `element not found` — trông y hệt một bug sản phẩm, nên QC đi mở bug thay vì sửa selector. Thiếu field → **cảnh báo mềm nêu rõ rủi ro** rồi mới fallback.

---

## Các bước xử lý (chi tiết)

1. **Role & stack** — qc-playwright (`stack-profile.yaml`): Python, pytest-playwright fixture, Page Object; mỗi test độc lập; gom theo (role, account) để auth không xen kẽ.
2. **Skills** — nạp một file skill `qa-runner` theo layer.
3. **Sinh script** từ `.Test.md`; tag `@trace.verifies={UC-ID}-SC{N}`.
4. **Chạy** — phân loại mỗi FAIL: **script-bug** (fix selector/logic) vs **product-gap** (giữ FAIL + evidence, **không bao giờ fake-pass**).
5. **Write Trace State — `qc_status`** (kết quả QC chính thức) + `qc_run_at`, `qc_owner`, `qc_blocked_by`, `last_updated`.

   ⚠️ **ĐỌC cột `status` TRƯỚC KHI GHI `pass`** *(GAPS-v4 G55, đối xứng `/dev-run-test`)*: row `DRIFT`/`ORPHANED` + test xanh → **`not_run`**, không bao giờ `pass`. `fail` và `skip` ghi bình thường — chỉ giá trị **khẳng định** cần giấy phép.

   Và ở lệnh này hậu quả đi **xa hơn** `/dev-run-test`: một `pass` sai còn **đóng một bug** (§Đóng bug đã verify). Nên khi hạ về `not_run`, lệnh **KHÔNG** clear `qc_owner`/`qc_blocked_by` và **KHÔNG** chạy bước đóng bug — đóng bug dựa trên một lần QC chạy trên spec đã đổi là đóng sai.
6. **Đóng bug đã verify** — chạy **TRƯỚC** bước clear cột (xem dưới).
7. **Refresh Panel Mirror** — Living Docs local.

---

## Checkpoint & Gate

- Tiền đề: case đã APPROVED ở `/qc-review`. Script sinh ra → review lại ở `/qc-review` (script) trước PR.

---

## Cơ chế đặc biệt

- **`qc_status` là trục authoritative** — khác `dev_selftest`; có evidence.
- **Không fake-pass** — product-gap giữ nguyên FAIL, đẩy về PO/Dev.
- **Stack QC tách hẳn dev** — `@trace.verifies` nối script ↔ SC.
- **`active_platform` khoá sổ trace** — `qc_status` ghi đúng `{UC-ID}-{platform}.tsv`.
- **Chủ sở hữu bước `🟡 Fixed → 🟢 Closed`.** `/report-bug` mở bug (`🟢 Open`), `/fix-bug` đặt `🟡 Fixed`, và **chỉ QC re-verify mới đóng được** — dev không tự đóng bug của mình.

### Vì sao đóng bug phải chạy TRƯỚC khi clear cột

Khi `qc_status` flip `pass`, lệnh clear `qc_owner`/`qc_blocked_by` về `—`. Nhưng `qc_blocked_by` **chính là con trỏ tới `{BUG-ID}`** — clear xong là mất đường về, và đối chiếu tay cũng không làm được. Nên thứ tự bắt buộc: **đọc `qc_blocked_by` → đóng bug → rồi mới clear**.

| `State` của bug | SC vừa `pass` → làm gì |
|---|---|
| `🟡 Fixed` | → `🟢 Closed` + dòng `Verified: /qc-run-test {today} — {UC-ID}-SC{N} pass` |
| `🟢 Open` (chưa ai fix) | **KHÔNG đóng.** Giữ `Open` + ghi chú kiểm tra lại test |
| `GAP-*` thay vì `BUG-*` | không đụng — spec-gap thuộc PO, không phải QC |

> Ca `Open` là ngoại lệ **có chủ đích**: test pass trên một bug chưa ai fix là dấu hiệu **test sai**, không phải bug hết. Tự đóng ở đây sẽ **chôn một defect thật**.

Bug report đã đổi phải **commit + push** vào spec repo — file local là dead drop, PO/Dev chỉ thấy sau khi push.

---

## 👓 Góc nhìn tối ưu

- **Trạm nặng nhất của QC** — sinh + chạy + phân loại + ghi trace. Chạy thật phụ thuộc môi trường (browser, data, service lên).
- **Phân loại script-bug vs product-gap phụ thuộc AI/reviewer** — sai loại → hoặc giấu lỗi sản phẩm hoặc báo nhầm. Đáng có tiêu chí rõ.
- **Selector phụ thuộc `/map-testids`** — nếu chưa map, script giòn.
- **Chạy lại tốn tài nguyên** — cân nhắc scoped run như dev-run-test.

---

## Kết nối

**Trước:** [`/qc-review`](18-qc-review.md) (case) · **Sau:** [`/qc-report`](20-qc-report.md) rồi [`/qc-review`](18-qc-review.md) (script).
