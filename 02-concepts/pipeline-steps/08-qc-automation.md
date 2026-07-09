[← Dev self-test](07-dev-selftest.md) · [Pipeline Steps](README.md) · [Next: Validate Traces →](09-validate-traces.md)

# Bước 8 · QC Automation — Dây chuyền kiểm thử 6 trạm (QC Pipeline)

> **Tóm tắt.** Dây chuyền QC tự động 6 trạm: phân rã yêu cầu → lập kế hoạch → thiết kế test case → review → chạy Playwright → report. Ghi `qc_status` **chính thức** + evidence.
> **Commands:** `/qc-analyze` → `/qc-plan` → `/qc-design-test` → `/qc-review` → `/qc-run-test` → `/qc-report`

| | |
|---|---|
| **Giai đoạn** | QC Automation |
| **Owner** | 👤 QA / Tester |
| **Đầu vào** | UC-ID + spec (PRD/BDD) + code đã chạy |
| **Đầu ra** | Test case, script Playwright, `qc_status`, evidence, product-gap |
| **HITL** | 🟠 Vừa — cổng review case & script trước khi chạy |

---

## Mục đích (Purpose)

Đây là **kiểm thử chính thức** (khác với dev smoke). Dùng module **`qc-playwright`** (Python + pytest-playwright + Page Object) — độc lập với module implementation của dev. Bước này:

- Phân rã yêu cầu thành test case bám scenario, phát hiện **gap tài liệu**.
- Chạy test thật, ghi **`qc_status` chính thức** + **evidence**.
- Phân loại FAIL: **script-bug** (sửa script) vs **product-gap** (giữ FAIL + evidence, **không bao giờ fake-pass**).
- Đẩy **product-gap** ngược về PO/Dev.

---

## Input (Đầu vào)

- **UC-ID** + platform (QC pass khoá 1 platform).
- Spec: PRD / `.feature` (từ spec repo, qua `spec_source`).
- Code đã sinh & chạy được.
- `qc_dir` (working docs của QC) + module `qc-playwright`.

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| `docs/{UC-ID}/…` | `REQUIREMENT_ANALYSIS.md`, `DOC_GAPS.md`, `TEST_PLAN.md`, `test-cases/*.Test.md` |
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
- SC nào **PASS/FAIL** chính thức (`qc_status`)? FAIL là **script-bug** hay **product-gap**?

---

## Framework xử lý thế nào (Mechanics)

Dây chuyền **6 trạm**, output trạm trước là input trạm sau:

| # | Trạm | Việc |
|---|------|------|
| 1 | `/qc-analyze` | Phân rã yêu cầu + phát hiện **gap tài liệu** (`DOC_GAPS.md`) |
| 2 | `/qc-plan` | Đánh giá **rủi ro** + câu hỏi cho dev (`TEST_PLAN.md`) |
| 3 | `/qc-design-test` | Thiết kế **test case** dạng Markdown (`*.Test.md`) |
| 4 | `/qc-review` | 🛑 **Cổng review** hai chiều: test case & script trước khi chạy |
| 5 | `/qc-run-test` | Sinh & chạy **pytest-playwright**, ghi **`qc_status`** chính thức |
| 6 | `/qc-report` | Report + **evidence**, đẩy **product-gap** về PO/Dev |

- Stack QC bắt buộc theo `modules/qc-playwright/stack-profile.yaml`: Python + pytest-playwright + Page Object; mỗi test độc lập; gom theo (role, account) để auth không xen kẽ.

---

## HITL / Gate

- 🛑 `/qc-review` — **cổng review** case & script: không chạy test kém.
- **Không fake-pass**: FAIL là product-gap → giữ nguyên FAIL + evidence, đẩy về PO/Dev.

---

## Anti-pattern

- ❌ Lẫn `qc_status` với `dev_selftest` — hai trục độc lập.
- ❌ Sửa script cho "xanh" khi thực chất là product-gap → giấu lỗi sản phẩm.
- ❌ Chạy `/qc-run-test` khi chưa qua cổng `/qc-review`.

---

## Bước tiếp theo (Next step)

Có `qc_status` → soát độ phủ toàn cục:

➡️ [Bước 9 · Validate Traces — `/validate-traces`](09-validate-traces.md)
