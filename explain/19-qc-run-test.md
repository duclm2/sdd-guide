[← /qc-design-testdata](31-qc-design-testdata.md) · [Explain Home](README.md) · [Next: /qc-report →](20-qc-report.md)

# 19 · `/qc-design-script` · `/qc-run-script` · `/qc-run-manualtest` — Trạm 6 · 8 · 8b: sinh · chạy · chạy tay

> **Một câu.** Biến `.Test.md` đã review thành **script TypeScript**, chạy thật, rồi ghi **`qc_status` chính thức** (có evidence) vào trace TSV.

---

## Vấn đề giải quyết

> **Trước Đợt 2 · b2 đây là MỘT lệnh `/qc-run-test` gánh BỐN việc**: quyết cái nào automate được · viết mã · chạy · phán một kết quả đỏ là lỗi sản phẩm hay lỗi mã test. Bốn quyết định, không chỗ nào dừng giữa chúng — nên không ai soát được mã trước khi nó chạy, và TC **không** automate được (OTP, sinh trắc học) rơi ra ngoài: không trạm nào ghi `qc_status` cho chúng, rồi `/qc-report` chấm cả PRD là FAIL vĩnh viễn. `/qc-run-manualtest` sinh ra để bịt đúng chỗ đó.


Đây là nơi QC trở thành **chính thức**: chạy test thật (Playwright cho web + API/`system` · WebdriverIO + Appium cho mobile), phân loại FAIL (script-bug vs product-gap, **không fake-pass**), và đóng dấu `qc_status` — trạng thái QC authoritative.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC, **ba trạm** — 6 sinh script · 8 chạy script · 8b chạy tay (song song trạm 8). Giữa 6 và 8 là cổng `/qc-review-script` (trạm 7, xem [trang 18](18-qc-review.md)).
  - `/qc-design-script {TICKET-ID}` — sau `/qc-automation-assess` (trạm 5). Ba chốt, thiếu cái nào → **DỪNG**: `REVIEW_<FEATURE>.md` verdict ≥ `APPROVED_WITH_SUGGESTIONS` · có `TEST_DATA_PLAN.md` (do `/qc-design-testdata` lập — chốt này, không phải số trạm, giữ thứ tự) · có `AUTOMATION_ASSESSMENT.md`.
  - `/qc-run-script {UC-ID}` — sau `/qc-review-script`: `REVIEW_SCRIPT_<FEATURE>.md` verdict ≥ `APPROVED_WITH_SUGGESTIONS`, không có file → **DỪNG**.
  - `/qc-run-manualtest {UC-ID}` — chỉ cần `AUTOMATION_ASSESSMENT.md` (lọc TC `Automatable: N`).
- **Stack:** module QC theo nền (`qc-playwright-ts` · `qc-wdio-appium`, xem `steps/qc-scope.md` §2b) — **độc lập** module dev.

---

## Input / Output

**Input:** `.Test.md` đã review + `AUTOMATION_ASSESSMENT.md` + `TEST_DATA_PLAN.md` (cột `Nguồn`/`Cách ly` của từng `TD-xxx` → fixture) + skill `qa-script-designer` (sinh) / `qa-script-runner` (chạy), chọn **theo nền** (`web` · `mobile` · `system`) + bảng Test Selectors §4.5.6 (**giá trị** test-id, từ `/map-testids`) + **`@trace.testid_attr`** ở header tech-doc (**tên thuộc tính** chứa chúng).

**Output:**
- `/qc-design-script`: spec `{paths.qc_automation_dir}/{web|mobile|system}/tests/{domain}/{prd-slug}/<feature>-<scenario>.spec.ts` + `pages/*.page.ts` (web) · `screens/*.screen.ts` (app) · `api/*.api.ts` (system); điền cột `Script file` trong `AUTOMATION_ASSESSMENT.md`; `TESTABILITY_IMPROVEMENTS.md` khi gặp rào cản. Script đã có giữ đường dẫn cũ, bố cục cũ vẫn chạy kèm ⚠️. **Không chạy test, không ghi sổ trace.**
- `/qc-run-script`: kết quả + cột `qc_status` trong `.trace/…/{UC-ID}-{platform}.tsv` + panel mirror (`.trace-mirror/`).
- `/qc-run-manualtest`: cột `qc_status` + `MANUAL_EVIDENCE.md` (chỉ append).

> **`@trace.testid_attr` — đọc, KHÔNG suy từ platform.** §4.5.6 cho **giá trị** test-id; field này cho **tên thuộc tính** chứa chúng. `getByTestId()` của Playwright mặc định dò `data-testid` **nhưng cấu hình được** — dự án dùng `data-test`/`data-qa` thì phải khai `use: { testIdAttribute: '{attr}' }` trong `playwright.config.ts` trước, không thì **trượt 100% locator**. (Cách cấu hình theo nền nằm ở `qa-script-designer/_shared/{web|mobile|api}-conventions.md`.)
>
> Suy từ platform là **phát biểu lại một sự thật đã ghi ở nơi khác** (`/map-testids` đã phân giải một lần cho cả feature, `/generate-code` đọc chính field đó để emit). Và nó là **lỗi ngầm theo kiểu tệ nhất**: test fail `element not found` — trông y hệt một bug sản phẩm, nên QC đi mở bug thay vì sửa selector. Thiếu field → **cảnh báo mềm nêu rõ rủi ro** rồi mới fallback.

---

## Các bước xử lý (chi tiết)

1. **Role & stack** — module theo nền (`stack-profile.yaml`): TypeScript, Playwright/WDIO fixture, Page/Screen/API Object; mỗi test độc lập; gom theo (role, account) để auth không xen kẽ.
2. **Skills** — `/qc-design-script` nạp `qa-script-designer/_shared/{web|mobile|api}-conventions.md` + `file-naming-and-folders.md`, rồi **một** file lane theo nền → tầng. TC lane `job`/`queue` → **DỪNG theo TC** (chưa có bộ sinh script; TC lane `api` cùng UC vẫn sinh). `/qc-run-script` nạp `qa-script-runner/{web|system|mobile}/run.md`.
3. **Sinh script** (`/qc-design-script`) — **chỉ cho TC `Automatable: Y`** (TC `N` bỏ qua, không sinh rồi `skip`); TC `Y` chưa có dòng nào trong `TEST_DATA_PLAN.md` → in `⚠️` kèm danh sách, không lặng lẽ tự nghĩ dữ liệu; tag `@trace.verifies={UC-ID}-SC{N}`; `P0` → `@smoke` + `@regression`, `P1`–`P3` → `@regression`; locator từ §4.5.6 (test-id trước), assertion ở spec; rào cản kỹ thuật → mục `IMPROVE-*` rồi hạ TC về `Automatable: N`. Guard chân thứ tư hợp đồng test-id (mức `warn`). Xong thì điền cột `Script file`.
4. **Chạy** (`/qc-run-script`) — chạy đúng các file ở cột `Script file` (**không tự suy đường dẫn**; cột còn `—` → báo ra). Test đỏ **một lần** chưa nói được nó đỏ vì cái gì → **chạy lại riêng test đó, tối đa 2 lần**, rồi mới phân loại thành **ba** nhãn:

   ```
   đỏ → đỏ → đỏ     ⇒ NHẤT QUÁN       → sang bước điều tra: script-bug | product-gap
   đỏ → xanh        ⇒ KHÔNG NHẤT QUÁN → flaky
   đỏ → đỏ → xanh   ⇒ KHÔNG NHẤT QUÁN → flaky
   ```

   | Nhãn | Khi nào | Hệ quả |
   |---|---|---|
   | `script-bug` | Sai locator / logic test / timing / dữ liệu test | QC tự sửa. **Không** mở bug |
   | `product-gap` | Hành vi thật ≠ spec — defect thật | Mở bug qua `/report-bug`. Giữ `fail` + evidence, **không bao giờ fake-pass** |
   | `flaky` | Không nhất quán ở bước chạy lại — **chưa đủ căn cứ** | **Quarantine**: gắn tag `@quarantine` + loại khỏi lượt chính thức bằng script `package.json` (web: `--grep-invert @quarantine`, tách project `flaky-tests` — **không** `test.skip`; app: một pattern `grep` Mocha kết hợp) + ghi **nghi vấn** nguyên nhân. **Không** mở bug từ một lần chạy hên xui |

   **Điều tra bằng bằng chứng, không đoán** — đọc trace/video/log: timeline, DOM snapshot, network, console **tại thời điểm fail**. Không chắc giữa `script-bug` và `product-gap` → **mời Dev cùng xem trace**.

   🛑 **Người xác nhận TRƯỚC khi hành động** — lệnh in đề xuất kèm evidence cụ thể rồi **dừng chờ**. **Không ghi `qc_status`** cho scenario nào còn FAIL chưa được xác nhận nhãn.
5. **Write Trace State — `qc_status`** (kết quả QC chính thức) + `qc_run_at`, `qc_owner`, `qc_blocked_by`, `last_updated`.

   ⚠️ **ĐỌC cột `status` TRƯỚC KHI GHI `pass`** *(GAPS-v4 G55, đối xứng `/dev-run-test`)*: row `DRIFT`/`ORPHANED` + test xanh → **`not_run`**, không bao giờ `pass`. `fail` và `skip` ghi bình thường — chỉ giá trị **khẳng định** cần giấy phép.

   Và ở lệnh này hậu quả đi **xa hơn** `/dev-run-test`: một `pass` sai còn **đóng một bug** (§Đóng bug đã verify). Nên khi hạ về `not_run`, lệnh **KHÔNG** clear `qc_owner`/`qc_blocked_by` và **KHÔNG** chạy bước đóng bug — đóng bug dựa trên một lần QC chạy trên spec đã đổi là đóng sai.
   `flaky` → `qc_status = not_run`, `qc_owner = qc`. Không tạo trạng thái mới: `not_run` đúng nghĩa *"chưa có kết luận"*, và `qc_owner = qc` để nó không rơi vào khoảng không ai nhận.
6. **Self-Review** *(mới)* — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
7. **Đóng bug đã verify** — chạy **TRƯỚC** bước clear cột (xem dưới).
8. **Refresh Panel Mirror** — Living Docs local. Dòng tổng kết bắt buộc có vế `🚧 quarantined` · `🚫 disabled` kể cả khi bằng 0; coverage ghi dạng `X/Y TC đang chạy · Z đang quarantine · W disabled`.
9. **Chạy tay** (`/qc-run-manualtest`) — liệt kê TC `Automatable: N` còn `not_run`, **hỏi từng TC** (không đọc file checklist), **ghi sổ ngay** sau mỗi câu trả lời → chạy lại là tự resume. `pass`/`fail` bắt buộc kèm bằng chứng (ai chạy · ngày · ảnh/mã giao dịch/quan sát cụ thể), append vào `MANUAL_EVIDENCE.md`. Không có nhãn `flaky`; cùng guard `DRIFT`/`ORPHANED` → không ghi `pass`.

Mỗi lệnh append một dòng vào `quality-metrics.jsonl` (`"phase":"qc-design-script"` / `"qc-run-script"` / `"qc-run-manualtest"`).

---

## Checkpoint & Gate

- **`/qc-design-script`: chặn CỨNG khi script đã tồn tại** — ghi đè là xoá bản sửa tay `script-bug` của QC. `--yes` không bỏ qua được. Script có, TC không đổi → bỏ qua im lặng; TC mới → sinh phần thiếu; **file có thay đổi chưa commit** (`git status`) → **DỪNG**, sinh từ trắng phải thêm `--force`. Script sinh ra → soát ở `/qc-review-script` trước khi chạy.
- **`/qc-run-script` · `/qc-run-manualtest`: không chặn** — chỉ ghi cột sổ trace do chính chúng sở hữu, dựng lại được bằng cách chạy lại. Nhưng stamp phiên bản lệch vẫn chặn việc ghi `pass`.
- 🛑 **Xác nhận nhãn FAIL** — cổng chặn được **thêm vào**, trong khi framework vốn đang giảm số cổng chặn (G41). Lý do: **cả hai hướng sai đều không đảo ngược rẻ.** Gắn nhầm `script-bug` cho lỗi sản phẩm thật là **giấu bug** cho tới khi khách gặp; mở bug từ một lần chạy hên xui là **đốt thời gian dev** và làm mòn niềm tin vào QC.

> **Đây KHÔNG phải `retries` trong config test runner.** `retries` tự thử lại rồi báo *"passed on retry"* — nó **che** sự không nhất quán. Ở đây chạy **tách biệt từng lần để quan sát**, vì chính sự không nhất quán mới là thông tin cần.

---

## Cơ chế đặc biệt

- **`qc_status` là trục authoritative** — khác `dev_selftest`; có evidence.
- **Không fake-pass** — product-gap giữ nguyên FAIL, đẩy về PO/Dev.
- **Chạy lại để QUAN SÁT, không để cho qua** — ba nhãn FAIL tách *chưa đủ căn cứ* (`flaky`) ra khỏi *đã đủ căn cứ* (`script-bug` / `product-gap`). Trước đây hai nhãn gộp cả ba tình huống, nên một test hên xui hoặc bị gán bừa `script-bug` (giấu bug), hoặc thành một bug ma gửi cho dev.
- **Stack QC tách hẳn dev** — `@trace.verifies` nối script ↔ SC.
- **`active_platform` khoá sổ trace** — `qc_status` ghi đúng `{UC-ID}-{platform}.tsv`.
- **Chủ sở hữu bước `🟡 Fixed → 🟢 Closed`.** `/report-bug` mở bug (`🟢 Open`), `/fix-bug` đặt `🟡 Fixed`, và **chỉ QC re-verify mới đóng được** — dev không tự đóng bug của mình.

### Vì sao đóng bug phải chạy TRƯỚC khi clear cột

Khi `qc_status` flip `pass`, lệnh clear `qc_owner`/`qc_blocked_by` về `—`. Nhưng `qc_blocked_by` **chính là con trỏ tới `{BUG-ID}`** — clear xong là mất đường về, và đối chiếu tay cũng không làm được. Nên thứ tự bắt buộc: **đọc `qc_blocked_by` → đóng bug → rồi mới clear**.

| `State` của bug | SC vừa `pass` → làm gì |
|---|---|
| `🟡 Fixed` | → `🟢 Closed` + dòng `Verified: /qc-run-script {today} — {UC-ID}-SC{N} pass` |
| `🟢 Open` (chưa ai fix) | **KHÔNG đóng.** Giữ `Open` + ghi chú kiểm tra lại test |
| `GAP-*` thay vì `BUG-*` | không đụng — spec-gap thuộc PO, không phải QC |

> Ca `Open` là ngoại lệ **có chủ đích**: test pass trên một bug chưa ai fix là dấu hiệu **test sai**, không phải bug hết. Tự đóng ở đây sẽ **chôn một defect thật**.

Bug report đã đổi phải **commit + push** vào spec repo — file local là dead drop, PO/Dev chỉ thấy sau khi push.

---

## 👓 Góc nhìn tối ưu

- **Cụm nặng nhất của QC** — sinh + chạy + phân loại + ghi trace (giờ tách ba lệnh, có cổng soát script ở giữa). Chạy thật phụ thuộc môi trường (browser, data, service lên).
- **Phân loại vẫn phụ thuộc người đọc evidence** — nhưng giờ có **tiêu chí và thứ tự bắt buộc**: chạy lại ×2 → đọc trace/log → đề xuất nhãn → **người xác nhận**. Chỗ mềm còn lại là ranh giới `script-bug` ↔ `product-gap` khi hành vi nằm ở **biên của spec**; đường ra đã ghi rõ là **mời Dev cùng xem trace**, không đoán cho xong.
- **Selector bám hợp đồng `/map-testids`** — §4.5.6 đã được chốt **trước** `/generate-code`, nên trạm này có bảng selector sẵn. Ca còn lại: §4.5.6 rỗng (dự án cũ chưa từng chạy `/map-testids`) → script quay về dò DOM và **giòn**.
- **Chạy lại tốn tài nguyên** — ×2 cho mỗi FAIL là chi phí thật. Đánh đổi có chủ đích: rẻ hơn nhiều so với một bug ma gửi cho dev, hoặc một defect thật bị gán `script-bug` rồi chôn.
- **`flaky` không có cột riêng trong sổ trace** — nó nằm ở `qc_status = not_run` + `qc_owner = qc` + nghi vấn ghi trong report. Cố ý: thêm một giá trị enum mới sẽ kéo theo sửa schema, lint, mọi lệnh đọc cột đó — trong khi `not_run` đã đúng nghĩa *chưa có kết luận*. Cái giá là **không truy được lịch sử flaky bằng máy**, phải đọc report.

---

## Kết nối

**Trước:** [`/qc-automation-assess`](30-qc-automation-assess.md) + [`/qc-design-testdata`](31-qc-design-testdata.md) → `/qc-design-script` → [`/qc-review-script`](18-qc-review.md) (script) → `/qc-run-script` ∥ `/qc-run-manualtest` · **Sau:** [`/qc-report {TICKET-ID}`](20-qc-report.md).
