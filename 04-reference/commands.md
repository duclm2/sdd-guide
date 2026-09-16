[← Docs Home](../README.md) · [Reference](./)

# Reference · Commands — Bảng đầy đủ (Command Catalog)

> 32 slash command, gom theo phase pipeline. Chi tiết cơ chế từng bước → [Pipeline Steps](../02-concepts/pipeline-steps/).

Mọi lệnh chạy chung một **Gate** (model check → target → context-loader → checkpoint). Xem [Khung chung](../02-concepts/pipeline-steps/README.md#khung-chung-mọi-command-common-gate).

---

## 0 · Setup & Foundation

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/setup-ai-first` | Dự án trống/có code | `.agent/`, `.claude/`, `project-context.yaml`, `CLAUDE.md` | Admin/Lead |
| `/generate-architecture` | config + tài liệu (`--from`) + code | `architecture.md` (SSOT kiến trúc, theo tier) | SA/Lead |

## 1 · Discovery

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/define-product` | Ý tưởng (+Figma) | `product-definition/*.md` (8 chặng) | PO |

## 2 · Specification (PRD)

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-prd` | product-definition | PRD draft **mới** | PO |
| `/extend-prd` | **PRD đã có** + `feedback/prd-change-requests/` | PRD v+1 (UC/AC/BR **nối tiếp**, `Status → draft`) | PO |
| `/amend-prd` | **PRD đã có** + ID cần sửa (PO khai) | PRD v+1 (nội dung ID **đổi tại chỗ**, `Status → draft`) | PO |
| `/refine-prd` | PRD | Findings 3 lăng kính DEV/SA/PO | PO+SA+Dev |
| `/review-context` (PRD) | PRD | Findings P0–P5 → `Status: approved` | PO |

> **Chọn lệnh nào cho PRD — bốn nhánh, phân biệt bằng THAO TÁC GHI:**
>
> | Tình huống | Lệnh | Ghi kiểu gì |
> |---|---|---|
> | PRD **chưa tồn tại** | `/generate-prd` | **Write** cả file |
> | **Thêm** UC/AC/BR mới | `/extend-prd` | Edit **add-only** — output là **superset chặt**; đánh số **nối tiếp** |
> | **Đổi** một yêu cầu đang đúng cú pháp | **`/amend-prd`** | Edit **tại chỗ** — output **KHÔNG** phải superset; **không** đánh số mới, **không** xoá row |
> | Sửa vấn đề **review đã soi ra** | `/refine-prd` → Review Board → `--resume` | Edit trong phạm vi finding |
>
> - `/generate-prd` **DỪNG HẲN** trên file đã có — §Guard *"Tồn tại → DỪNG. KHÔNG ghi, KHÔNG hỏi Y/N"*, vì ghi đè mất changelog + **đánh số lại BR** ⇒ hỏng `@trace.business_rules` trong mọi BDD đã sinh, và cả ba mất mát đều không hoàn tác được từ trong lệnh.
> - `/refine-prd` **không** thêm/đổi được theo ý định mới (nó tự cấm đụng section ngoài findings).
> - `/extend-prd` **không** sửa được nội dung cũ, trừ một cửa **phái sinh**: khi phần THÊM làm một BR cũ sai (Bước 3.2).
> - **Xoá hẳn** một BR/AC → không có lệnh, và **có chủ ý**: xoá row biến `@trace.business_rules` trong `.feature` thành `TRACE_ORPHAN` 🔴. Dùng `/amend-prd --retire {ID}` — nó khai tử **tại chỗ**, giữ nguyên row + ID.
>
> ⚠️ **Sửa tay file `.md` là điểm mù tuyệt đối (GAPS-v4 G54).** Mọi drift detector so **nhãn version**, không so nội dung. Sửa mà không bump version ⇒ **0 cờ**. `/validate-traces` Step 3.9 canh cửa sau bằng cờ 🔴 `PRD_UNTRACKED_EDIT` (git diff + git status vs `spec_baseline`); đường sửa là bump version + row changelog nêu UC — tức đúng việc `/amend-prd` làm hộ.

## 3 · Design-Spec (chỉ FE/App)

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-design-spec` | PRD approved + Figma | `design-spec/*.md` | PO/PM |

## 4 · BDD

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-bdd` | PRD approved (+design-spec) | `bdd/**/*.feature` | PO |
| `/review-context` (BDD) | `.feature` | Findings B1–B6 → `@trace.status: approved` | PO/Dev |

## 5 · Tech-Docs

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-tech-docs` | BDD approved + entity catalog | `tech-docs/{TICKET}-tech-design.md` | SA |
| `/review-tech-docs` | tech-design | Findings đa chiều + ký T7 | SA/Lead |

## 6 · Code

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-code` | `.feature` approved + tech-design | Code + `.trace/*.tsv` | Dev |
| `/review-code` | Code | Findings (read-only) | Dev/Lead |
| `/fix-bug` | Bug report | Fix + regression test | Dev |
| `/map-testids` | design-spec + BDD (`--from-code`: + UI code) | §4.5.6 Test Selectors + `@trace.testid_attr` | Người viết tech-doc (`--from-code`: Dev) |
| `/debug` | Mô tả lỗi | Phân tích (read-only) | Dev |

## 7 · Dev self-test

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/dev-gen-test` | Code + `.feature` | Bộ self-test | Dev |
| `/dev-run-test` | Self-test | Kết quả + cột `dev_selftest` | Dev |
| `/dev-smoke-test` | Service/app đang chạy | Kết quả smoke tại chỗ | Dev |

## 8 · QC Automation

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/qc-analyze` | UC + spec | `REQUIREMENT_ANALYSIS.md`, `DOC_GAP.md` + **Guard BR-tag** | QA |
| `/qc-plan` | Analysis | `TEST_PLAN.md` (rủi ro) | QA |
| `/qc-design-test` | Plan + `.feature` + §4.5.6 | `test-cases/*.Test.md` + **Guard SC coverage** | QA |
| `/qc-review-testcase` | Test case | 🛑 Cổng review test case | QA |
| `/qc-automation-assess` | Test case đã duyệt | Quyết định Automatable Y/N + %Automated | QA |
| `/qc-review-script` | Script + Page Object | 🛑 Cổng review script | QA |
| `/qc-design-script` | `.Test.md` reviewed + bảng Automatable | Script Playwright + `qc_status`. **Chạy lại ×2 → 3 nhãn FAIL** (`script-bug`·`product-gap`·`flaky`), 🛑 người xác nhận nhãn | QA |
| `/qc-report` | Kết quả run | Report + evidence + product-gap | QA |

> **Cả sáu trạm chạy một lượt Self-Review trước khi in report**, theo `skills/qc/_shared/self-review-principles.md` (một file dùng chung, không sáu bản sao).
> Self-review **rộng mà mềm**; Guard là phép **đếm** có hệ quả bắt buộc. Không bao giờ dùng self-review làm lý do gỡ một Guard.

## 9 · Quality & Trace

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/validate-traces` | `.trace/*.tsv` + spec/code/test | Ma trận coverage · `trace-report.json` · **`trace-history.jsonl`** (append) · đếm hàng đợi | Dev/QA/Lead |
> **`/validate-traces` có phạm vi (GAPS-v4 G57):** `--domain {d}` · `--prd {TICKET-ID}` · `--uc {UC-ID}`; không cờ nào = toàn bộ. Đây là lệnh **đắt nhất** trong framework (~33k token chỉ dẫn rồi đọc cả repo), nên cờ scope là thứ làm nó chạy được thường xuyên — và `/sync` Step 1e nói cho bạn biết **scope là gì**.
>
> ⚠️ Biên bản có scope **không bao giờ** được coi là đầy đủ: report mang `scope`, và `gate-trace` G2 **fail** nếu `scope.kind !== "all"` — không ngoại lệ, không đếm số domain trên đĩa. Muốn tạo PR thì phải có một lần audit **toàn bộ** đã commit.

| `/validate-traces --realign-prd-version {UC}` | cờ ⓘ `PRD_STALE_REF` | Sửa **chỉ dòng `@trace.*`** — không đụng logic | Dev |
| `/validate-traces --realign-techdoc-revision {UC}` | cờ ⓘ `TECHDOC_STALE_REF` | như trên | Dev |
| `/generate-spec-manifest` | Specs | Mục lục spec cho agent ngoài | SA/Lead |

## 10 · Feedback & Ops

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/report-bug` | Lỗi phát hiện | `feedback/bug-reports/` (spec-anchored) | Tester/QC |
| `/propose-scenario` | Scenario thiếu | **A:** `feedback/bdd-proposals/` → `/generate-bdd` chèn · **B:** `feedback/prd-change-requests/` → PO chạy `/extend-prd` | Tester/QC |
| `/learn` · `/learn --review` | Định hướng lặp lại · rà/retire lesson cũ | `project-lessons.md` | Tất cả |
| `/sync` | (umbrella) | Pull + submodule + nổi feedback + Living Docs | Lead |
| `/update-framework` | — | Sync bản npm mới | Lead |

---

## Ký hiệu trạng thái trong report

- 🛑 checkpoint (dừng, chờ `Y`) · 🔒 gate trạng thái (chặn downstream) · ⚪ read-only
- ⚠️ cảnh báo mềm (không chặn) · ✅ pass · ❌ fail
- 🔴 cờ **chặn PR** — `SEAM_UNWIRED` · `STUB_UNRESOLVED` · `ORPHANED` · `TRACE_ORPHAN`. Build xanh + test từng-UC xanh **không** đủ để bỏ qua chúng.
- 🟠 cờ drift **không chặn PR** — `PRD_DRIFT` · `BDD_DRIFT` · `TECHDOC_DRIFT` · `FE_TECHDOC_DRIFT` · `TECHDOC_STALE_VS_BDD` · `DESIGNSPEC_DRIFT` · `DESIGNSPEC_STALE_VS_BDD`. *"Code chưa theo kịp spec"*, khác *"code đang hỏng"*. **Hết cờ 🔴 ≠ sạch.**
- ⓘ cờ **không phải lỗi** — `PRD_STALE_REF` · `TECHDOC_STALE_REF` (version lệch nhưng changelog không nêu UC này → chỉ con trỏ cũ) · `SEAM_PENDING` · `STUB_PENDING` (owner chưa gen).

---

## CLI maintenance (không phải slash command)

Chạy qua `npx @educa-corp/sdd-framework <flag>`. Tất cả **dry-run mặc định** — thêm `--apply` để thực thi.

| Flag | Việc |
|------|------|
| `--init` | Cài/nâng cấp `.agent/` + shortcut `.claude/commands/`. Backup file bạn đã sửa sang `.agent/.overwritten-*/` và liệt kê ra |
| `--migrate-bdd-platform` | `bdd/*.feature` phẳng → `bdd/{platform}/`. Báo `OCCUPIED` / `LOST SPEC` thay vì đoán |
| `--migrate-specs` | Bố cục artifact-type-first cũ (`specs/prd/`, `specs/bdd/`) → feature-package |
| `--rename-prd-files` | `prd.md` → `{TICKET-ID}-{prd-slug}.md` |
| `--help` | Danh sách đầy đủ |

Trong repo framework: `npm run build` (assemble template + chạy self-check) · `npm run self-check` (chỉ kiểm contract trace).

→ [Trace Schema](trace-schema.md) · [Modules](modules.md) · [Configuration](configuration.md) · [Model Selection](model-selection.md)
