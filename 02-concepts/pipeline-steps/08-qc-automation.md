[← Dev self-test](07-dev-selftest.md) · [Pipeline Steps](README.md) · [Next: Validate Traces →](09-validate-traces.md)

# Bước 8 · QC Automation — Dây chuyền kiểm thử 6 trạm (QC Pipeline)

> **Tóm tắt.** Dây chuyền QC tự động 6 trạm: phân rã yêu cầu → lập kế hoạch → thiết kế test case → review → chạy Playwright → report. Ghi `qc_status` **chính thức** + evidence.
> **Commands:** `/qc-analyze` → `/qc-plan` → `/qc-design-test` → `/qc-review-testcase` → `/qc-automation-assess` → `/qc-design-script` → `/qc-review-script` → `/qc-run-script` → `/qc-report`

| | |
|---|---|
| **Giai đoạn** | QC Automation |
| **Owner** | 👤 QA / Tester |
| **Đầu vào** | **Trạm 1–4:** spec (PRD/BDD `approved`) + **hợp đồng test-id §4.5.6** (đóng băng ở bước 5) — **chưa cần code**.<br/>**Trạm 6–8 `/qc-design-script` · `/qc-run-script` thêm:** code đã chạy được — **hai trạm duy nhất** cần |
| **Đầu ra** | Test case, script Playwright, `qc_status`, evidence, product-gap |
| **HITL** | 🟠 Vừa — cổng review case & script trước khi chạy |

---

## Mục đích (Purpose)

Đây là **kiểm thử chính thức** (khác với dev smoke). Dùng module **`qc-playwright`** (Python + pytest-playwright + Page Object) — độc lập với module implementation của dev. Bước này:

- Phân rã yêu cầu thành test case bám scenario, phát hiện **gap tài liệu**.
- Chạy test thật, ghi **`qc_status` chính thức** + **evidence**.
  ⚠️ Nhưng `/qc-run-script` và `/qc-run-manualtest` **đọc cột `status` trước khi ghi `pass`** *(GAPS-v4 G55)*: row `DRIFT`/`ORPHANED` + test xanh → hạ về `not_run`, và **không** đóng bug nào ở lần chạy đó. `fail`/`skip` ghi bình thường.
- Phân loại FAIL thành **ba** nhãn — `script-bug` · `product-gap` · `flaky` — **sau khi đã chạy lại tối đa 2 lần**. Một test đỏ **một lần** chưa nói được nó đỏ vì cái gì.
- Đẩy **product-gap** ngược về PO/Dev.

---

## Input (Đầu vào)

- **UC-ID** + platform (QC pass khoá 1 platform).
- Spec: PRD / `.feature` (từ spec repo, qua `spec_source`).
- **Hợp đồng test-id**: `@trace.testid_attr` (header tech-doc, **tên** thuộc tính) + §4.5.6 Test Selectors (**giá trị** test-id, cột *Serves SC* là chỉ mục ngược). Đã chốt ở [bước 5](05-tech-docs.md) **trước khi có code**.
- Code đã sinh & chạy được — **chỉ `/qc-design-script` và `/qc-run-script` cần**. Bốn trạm đầu (`/qc-analyze` → `/qc-plan` → `/qc-design-test` → `/qc-review-testcase`) chạy **song song với FE** vì chỉ cần spec + hợp đồng test-id. Đó là chỗ hai nhánh của [bước 5](05-tech-docs.md) gặp lại.
- `qc_dir` (working docs của QC) + module `qc-playwright`.

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| `docs/{TICKET-ID}/{platform}/…` | `REQUIREMENT_ANALYSIS.md`, `DOC_GAP.md`, `TEST_PLAN.md`, `test-cases/*.Test.md` — **mỗi loại đúng một file cho cả (PRD × nền)**, các UC là hàng/mục bên trong (cột `UC`) |
| Script Python pytest-playwright | Sinh từ `.Test.md` đã review |
| Cột `qc_status` trong `.trace/…/{UC-ID}-{platform}.tsv` | Trạng thái QC **chính thức** |
| Evidence + report | `/qc-report` — kèm product-gap đẩy về PO/Dev |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **QA/Tester** | Lead: duyệt cổng review case & script; đọc report; phân loại gap |
| 🤖 **AI** | Phân rã yêu cầu, sinh test case + script Playwright, chạy, phân loại FAIL, ghi `qc_status` |
| 👤 **PO/Dev** | Nhận **product-gap** để xử lý (không phải lỗi script) |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Yêu cầu phân rã thành những **test case** nào? Tài liệu có **gap** gì?
- Rủi ro nào cao? Cần hỏi dev điều gì trước khi test?
- Test case & script đã đủ tốt để **chạy** chưa (cổng review)?
- SC nào **PASS/FAIL** chính thức (`qc_status`)? FAIL là **script-bug**, **product-gap**, hay chỉ **flaky**?
- Có business rule nào BDD đã nhắc mà bản phân tích bỏ sót không? Có scenario nào **không** test case nào phủ không?

---

## Framework xử lý thế nào (Mechanics)

Dây chuyền **6 trạm**, output trạm trước là input trạm sau:

| # | Trạm | Việc | Phép kiểm cơ học |
|---|------|------|---|
| 1 | `/qc-analyze` | Phân rã yêu cầu + phát hiện **gap tài liệu** (`DOC_GAP.md`) | **Guard BR-tag** |
| 2 | `/qc-plan` | Đánh giá **rủi ro** + câu hỏi cho dev (`TEST_PLAN.md`) | — |
| 3 | `/qc-design-test` | Thiết kế **test case** dạng Markdown (`*.Test.md`) | **Guard SC coverage** |
| 4 | `/qc-review-testcase` | 🛑 **Cổng review test case** — verdict `APPROVED`/`NEEDS_FIX` là điều kiện tiên quyết của trạm sau | — |
| 6 | `/qc-review-script` | 🛑 **Cổng review script** — biên bản riêng `REVIEW_SCRIPT_<FEATURE>.md` | — |
| 5–8 | `/qc-design-script` · `/qc-run-script` | Sinh rồi chạy **pytest-playwright**, ghi **`qc_status`** chính thức | **chạy lại ×2 + 3 nhãn FAIL** |
| 6 | `/qc-report` | Report + **evidence**, đẩy **product-gap** về PO/Dev | — |

### Hai Guard cơ học — chống bỏ sót **im lặng**

Trước đây sáu trạm này **không có phép kiểm cơ học nào**: bỏ sót một business rule, hay một scenario không có test case nào, đều xảy ra mà không ai biết. Hai guard đóng đúng hai lỗ đó:

| Guard | Ở đâu | Đối chiếu cái gì | Khi lệch |
|---|---|---|---|
| **BR-tag** | `/qc-analyze` | Tập business rule mà `.feature` **đã gắn tag** (A) ↔ tập rule bản phân tích **sinh ra** (B) | `A ∖ B` ≠ rỗng → **tự bổ sung** từ PRD, in danh sách |
| **SC coverage** | `/qc-design-test` | Mọi scenario **trong phạm vi** ↔ test case trỏ tới nó | Có SC chưa phủ → **viết bù TC ngay**, in danh sách |

**Cả hai in dòng kết quả kể cả khi sạch** (`Guard BR-tag: khớp {n}/{n}`) — guard im lặng khi sạch là guard không ai biết nó tồn tại, nên cũng không ai phát hiện khi nó hỏng.

> **SC coverage không có đường thoát.** Scenario bị gap chặn thì test case **vẫn viết đủ**, mang dấu `🚫 Block` trỏ tới `DOC_GAP.md` — gap là thứ được **ghi vào** test case, không phải cái cớ để không viết.

### Ba nhãn FAIL — chống kết luận vội

Một test đỏ có thể vì **script sai**, vì **sản phẩm sai**, hoặc vì **chạy hên xui**. Gộp ba thứ đó làm một là nói dối theo cả hai hướng: gắn nhầm `script-bug` cho lỗi sản phẩm thật là **giấu bug**; mở bug từ một lần chạy hên xui là **đốt thời gian dev**.

```
đỏ → đỏ → đỏ     ⇒ NHẤT QUÁN       → điều tra bằng evidence: script-bug | product-gap
đỏ → xanh        ⇒ KHÔNG NHẤT QUÁN → flaky
đỏ → đỏ → xanh   ⇒ KHÔNG NHẤT QUÁN → flaky
```

| Nhãn | Khi nào | Hệ quả | `qc_status` |
|---|---|---|---|
| `script-bug` | Sai locator / logic test / timing / dữ liệu | QC tự sửa, **không** mở bug | — (sửa rồi chạy lại) |
| `product-gap` | Hành vi thật ≠ spec — defect thật | Mở bug qua `/report-bug`, giữ evidence | `fail` |
| `flaky` | Không nhất quán qua các lần chạy lại | Cách ly + ghi **nghi vấn** nguyên nhân. **Không** mở bug | `not_run`, `qc_owner = qc` |

> **Đây KHÔNG phải `retries` trong config test runner.** `retries` tự thử lại rồi báo *"passed on retry"* — nó **che** sự không nhất quán. Ở đây chạy **tách biệt từng lần để quan sát**, vì chính sự không nhất quán mới là thông tin cần.
>
> `flaky` → `not_run` chứ không phải một trạng thái mới: nó đúng nghĩa *"chưa có kết luận"*. Và `qc_owner = qc` để nó không rơi vào khoảng không ai nhận.

**Người xác nhận trước khi hành động** — lệnh in đề xuất kèm evidence cụ thể rồi **dừng chờ**. Không ghi `qc_status` cho scenario nào còn FAIL chưa được xác nhận phân loại.

### Self-Review — mỗi trạm tự soát trước khi in report

Cả sáu trạm nạp chung `skills/qc/_shared/self-review-principles.md` và chạy một lượt tự soát trước khi in report.

> ⚠️ **Self-review KHÔNG thay Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**. Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một guard — file đó ghi rõ ranh giới này ngay ở đầu.

- Stack QC bắt buộc theo `modules/qc-playwright/stack-profile.yaml`: Python + pytest-playwright + Page Object; mỗi test độc lập; gom theo (role, account) để auth không xen kẽ.
- **Locator lấy từ hợp đồng, không dò DOM**: thứ tự ưu tiên là §4.5.6 → `@trace.testid_attr` → mới tới các cách khác. Skill `qa-runner` đã bỏ hết chỉ dẫn "dò DOM trước".

---

## HITL / Gate

- 🛑 `/qc-review-testcase` · `/qc-review-script` — **hai cổng review riêng**: không chạy test kém, và trạm sau đọc được verdict của ĐÚNG vai nó cần.
- 🛑 **Xác nhận phân loại FAIL** — mỗi FAIL phải được người chốt nhãn trước khi ghi `qc_status`. Đây là cổng chặn hiếm hoi được **thêm vào** (framework vốn đang giảm số cổng), vì **cả hai hướng sai đều không đảo ngược rẻ**.
- **Không fake-pass**: FAIL là product-gap → giữ nguyên FAIL + evidence, đẩy về PO/Dev.

---

## Anti-pattern

- ❌ Lẫn `qc_status` với `dev_selftest` — hai trục độc lập.
- ❌ Sửa script cho "xanh" khi thực chất là product-gap → giấu lỗi sản phẩm.
- ❌ Chạy `/qc-design-script` khi chưa qua cổng `/qc-review-testcase`.
- ❌ **Kết luận từ một lần chạy đỏ** — chưa loại nhiễu thì chưa phân biệt được `flaky` với lỗi thật.
- ❌ **Tự dò selector từ DOM** thay vì đọc §4.5.6 — script giòn, dev đổi một class là vỡ mà không ai báo.
- ❌ Dùng self-review làm lý do **bỏ qua** một Guard.

---

## Bước tiếp theo (Next step)

Có `qc_status` → soát độ phủ toàn cục:

➡️ [Bước 9 · Validate Traces — `/validate-traces`](09-validate-traces.md)
