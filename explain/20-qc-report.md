[← ba trạm script](19-qc-run-test.md) · [Explain Home](README.md) · [Next: /qc-smoke-test →](32-qc-smoke-test.md)

# 20 · `/qc-report` — Trạm 9: từ báo cáo thành **phán quyết**

> **Một câu.** Trạm cuối QC: chấm **cả PRD** bằng **hai số coverage + 8 ngưỡng**, ra một `VERDICT`, và đẩy **product-gap** về PO/Dev.

---

## Vấn đề giải quyết

Ba câu mà bản cũ không trả lời được:

| Câu | Bản cũ | Đợt 3 |
|---|---|---|
| *"Coverage bao nhiêu?"* | **một** số — gộp *"đã viết TC"* với *"đã chạy"* | **hai** số in cạnh nhau: `Design 100% · Execution 40%` |
| *"Được release chưa?"* | không ai trả lời — người đọc tự đoán từ *"38/42 pass"* | **8 ngưỡng khai báo trước** → `VERDICT` |
| *"0 fail có phải là tốt?"* | một PRD **chạy 0 test** cũng có 0 fail ⇒ chấm **PASS** | ba ngưỡng tự chặn ca đó ⇒ **FAIL** |

Câu thứ ba là lỗ hổng logic nặng nhất, và framework đã học **chính bài này** ở phía dev — `G55`:
*"ai khẳng định một giá trị dương thì phải kiểm rằng mình được phép khẳng định"*. Trạm này là
**mệnh đề đối ngẫu của nó ở cấp báo cáo**: không kết luận PASS chỉ vì **không có tin xấu**.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (**trạm 9**, cuối), sau `/qc-run-script` và `/qc-run-manualtest`.
- **Target là `{TICKET-ID}`, không phải `{UC-ID}`** *(`D3-TARGET`)* — câu *"được release chưa"* là
  câu **cấp PRD**; không ai hỏi về một UC lẻ. Chấm một UC xanh rồi báo `PASS` trong khi UC bên cạnh
  chưa chạy là đúng lỗ hổng trạm này sinh ra để bịt, chỉ dịch lên một cấp.

---

## Input / Output

**Input: BỐN nguồn** *(bản cũ đọc một)* — sổ trace *(`qc_status` · `qc_owner` · `qc_blocked_by` · `status`)* · `.Test.md` *(`Priority`)* · `AUTOMATION_ASSESSMENT.md` · file bug *(`Severity`)*. Nối nhau qua `@trace.verifies`.

**Output:** `VERDICT` *(`PASS` / `FAIL` / `INCOMPLETE`)* + hai số coverage + 8 ô ngưỡng + risk matrix + evidence; product-gap đẩy về PO/Dev.

---

## Các bước xử lý (chi tiết)

1. **Thu dữ liệu** — duyệt `uc_list`, mở **từng** sổ trace *(một sổ mỗi UC)*, join `.Test.md` qua `@trace.verifies`, đọc `AUTOMATION_ASSESSMENT.md` và file bug. In một **dòng bằng chứng** đã đọc được những gì.
2. **Coverage** — hai số + cờ `INVERSION` *(design < execution ⇒ có SC đã chạy mà không TC approved đứng sau)* + **tách `not_run` theo `qc_owner`**: chưa ai chạy · flaky/quarantine · chờ PO · spec đổi — **bốn chủ khác nhau**.
3. **Risk Matrix** — bảng tra *(trạng thái SC × `Priority`)* cho mọi SC còn `fail`/`skip`/`not_run`.
4. **Gate Decision** — chấm 8 ô rồi ra `VERDICT`: ≥1 ô `❌` → `FAIL` · còn ô `⚠️` (không ô nào `❌`) → `INCOMPLETE` · mọi ô `✅` → `PASS`. **`❌` thắng `⚠️`** — một chỗ trống không làm một kết luận bớt đúng (PRD chưa chạy gì ra `FAIL` nhờ `coverage 0 « 85` và `p0-not-run » 0`, không bị ba ô pass-rate mẫu số 0 che thành `INCOMPLETE`). Ô nào thiếu dữ liệu thì ghi **`⚠️ THIẾU DỮ LIỆU`**, **không ước lượng cho đủ bảng**. Ngưỡng mặc định override được qua `$ARGUMENTS` (vd `--coverage-min=80`).
   - **Stamp `Priority`** — `prd_version` trong `.Test.md` của một UC ≠ PRD hiện tại ⇒ nhãn P0/P1 có thể cũ ⇒ ô #2 · #3 · #6 · #7 · #8 vẫn tính nhưng đánh `⚠️` (không ô nào được `✅`), kèm gợi ý chạy lại `/qc-design-test {UC-ID}`.
5. **Evidence theo nền** — Playwright HTML Report + Trace *(web·system)* hoặc Allure *(app)*. Không dashboard viết tay.
6. Tách **product-gap** → in sẵn các lệnh `/report-bug {UC-ID} …` (không tự tạo report). Scenario nhãn **`flaky`** **không** thành bug — nó là *chưa có kết luận*, `qc_status` để `not_run`. `script-bug` không được file.
7. **Self-Review** rồi append một dòng `"phase":"qc-report"` vào `quality-metrics.jsonl`.
> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard — chính file `self-review-principles.md` ghi rõ ranh giới đó ngay ở đầu.


---

## Checkpoint & Gate

- **CHECKPOINT: chặn thường** (`--yes` bỏ qua được) — report sinh hoàn toàn từ lần chạy gần nhất, không ô nào người điền.
- **Không gate chặn PR** — `VERDICT` là **tín hiệu kỹ thuật**, không phải quyết định release. QC đưa tín hiệu, Lead/PM giữ quyền quyết cuối.
- Nhưng report **in cả bộ ngưỡng đã dùng** — đó là cách duy nhất phân biệt *"FAIL vì chất lượng"* với *"FAIL vì ai đó đặt ngưỡng quá chặt"*, và nó làm việc **hạ ngưỡng** thành một quyết định **thấy được** *(cùng lập luận `G-GATE`)*.

---

## Cơ chế đặc biệt

- **Product-gap routing** — khép vòng feedback: lỗi sản phẩm từ QC về đúng người quyết.
- **Evidence-first** — báo cáo kèm bằng chứng (khác dev smoke).
- **Coverage kép** — `design` và `execution` **không bao giờ gộp**. `skip` và `not_run` **không** vào `execution`: một SC `skip` là một SC **không có oracle nào kiểm chứng**.
- **`@quarantine` không làm coverage đẹp lên** — SC bị cách ly vẫn đếm vào `not_run`. Cách ly là để **lượt chạy không bị nhiễu**, không phải để thoát khỏi luật *"Untested ≠ Pass"*.
- **`INCOMPLETE` là trạng thái thứ ba, và nó quan trọng** — thiếu dữ liệu thì **không bao giờ** mặc định thành `PASS`.

---

## 👓 Góc nhìn tối ưu

- **Product-gap → PO/Dev** nhưng cơ chế nối với `/report-bug` không tự động — dễ đứt. Đáng tích hợp để gap thành bug spec-anchored có hồ sơ.
- **Report tách khỏi run** (trạm riêng) — linh hoạt nhưng thêm một lệnh phải nhớ gọi.
- **Sau report → `/validate-traces`** để làm mới Living Docs (`qc_status`) — chuỗi này phụ thuộc người chạy đủ.

---

## Kết nối

**Trước:** [`/qc-run-script` + `/qc-run-manualtest`](19-qc-run-test.md) · **Sau:** [`/validate-traces --prd {TICKET-ID}`](21-validate-traces.md) (làm mới Living Docs), rồi tạo PR — `FAIL`/`INCOMPLETE` thì xử phần `❌`/`⚠️` trước; product-gap → [`/report-bug`](25-report-bug.md).
