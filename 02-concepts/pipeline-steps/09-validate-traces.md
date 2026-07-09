[← QC Automation](08-qc-automation.md) · [Pipeline Steps](README.md) · [Next: Feedback Loop →](10-feedback-loop.md)

# Bước 9 · Validate Traces — Ma trận độ phủ (Coverage Matrix)

> **Tóm tắt.** Check **read-only** độ phủ giữa **spec ↔ code ↔ test**, gồm cả PRD version drift. Chỉ ra chỗ chưa phủ — không sửa gì.
> **Command:** `/validate-traces`

| | |
|---|---|
| **Giai đoạn** | Quality (xuyên suốt) |
| **Owner** | 👤 Dev / QA / Lead |
| **Đầu vào** | `.trace/*.tsv` + spec + code + test |
| **Đầu ra** | Ma trận coverage + phân loại từng SC |
| **HITL** | ⚪ Read-only |

---

## Mục đích (Purpose)

Traceability chỉ có giá trị khi **kiểm được**. Bước này cho một **bức tranh toàn cục**: scenario nào đã có code, có test, hay còn hở — để không "tưởng xong mà chưa xong". Vì là **read-only**, chạy lúc nào cũng an toàn.

- Đối chiếu spec ↔ code ↔ test cho từng SC.
- Phát hiện **PRD version drift** (spec đổi mà code chưa regen).
- Chỉ ra **gap** chưa phủ để lên kế hoạch bù.

---

## Input (Đầu vào)

- `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` (mọi UC × platform).
- Spec (`.feature`), code (`@trace.implements`), test (`@trace.verifies`).

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| Ma trận coverage spec ↔ code ↔ test | Trạng thái từng SC + `code_coverage` tổng |
| (Tuỳ chọn) `trace-report.md` | Báo cáo tổng hợp |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Dev/QA/Lead** | Đọc ma trận, quyết bù gap / regen drift |
| 🤖 **AI** | Quét trace, phân loại SC, dựng ma trận (không sửa file) |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Scenario nào **chưa có code** (UNTRACKED)? Chưa có test (GAP)?
- Code nào **lỗi thời** so với spec (DRIFT)?
- Độ phủ tổng thể (`code_coverage`) bao nhiêu?

---

## Framework xử lý thế nào (Mechanics)

Phân loại mỗi SC theo **thứ tự ưu tiên** (rule sớm thắng):

| # | Trạng thái | Điều kiện |
|---|-----------|-----------|
| 1 | **UNTRACKED** | `gen_ver == —` — scenario chưa từng sinh code |
| 2 | **DRIFT** | có `implemented_by` **và** `spec_ver != gen_ver` — spec đổi sau codegen → **regen trước khi test** |
| 3 | **GAP** | có `implemented_by` **và** (`test_count == — / 0`) — có code, chưa test |
| 4 | **OK** | `spec_ver == gen_ver`, có `implemented_by`, `test_count > 0` |

> **Vì sao DRIFT xét trước GAP:** một SC đã có code, chưa test, **và** spec vừa drift phải hiện `DRIFT` (không phải `GAP`) — vì `/generate-code` xử `GAP` = "skip codegen" còn `DRIFT` = "regenerate". Nếu GAP thắng, code lỗi thời bị bỏ qua và test sinh trên code cũ.

---

## HITL / Gate

- ⚪ **Read-only** — bỏ qua checkpoint ghi-file. Đây là công cụ *quan sát*, không phải *hành động*.

---

## Ví dụ (Example)

```
/validate-traces auth
  UC-01  SC-01.1  OK
  UC-01  SC-01.2  GAP        (có code, chưa test)
  UC-02  SC-02.1  DRIFT      (spec_ver 1.1 ≠ gen_ver 1.0 → regen)
  UC-02  SC-02.3  UNTRACKED  (chưa sinh code)
  code_coverage = 7/9 (78%)
```

---

## Anti-pattern

- ❌ Coi validate-traces là "chạy xong là fix xong" — nó chỉ báo cáo; hành động ở `/generate-code` (regen DRIFT) và QC (bù GAP).
- ❌ Bỏ qua DRIFT rồi test trên code cũ.

---

## Bước tiếp theo (Next step)

Gap/drift phát hiện → quay lại `/generate-code` (regen) hoặc bù test; lỗi/thiếu từ QC → vòng phản hồi:

➡️ [Bước 10 · Feedback Loop — `/report-bug` · `/propose-scenario` · `/learn` · `/sync`](10-feedback-loop.md)
