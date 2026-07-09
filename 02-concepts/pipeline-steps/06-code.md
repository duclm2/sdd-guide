[← Tech-Docs](05-tech-docs.md) · [Pipeline Steps](README.md) · [Next: Dev self-test →](07-dev-selftest.md)

# Bước 6 · Code — Sinh mã nguồn (Code Generation)

> **Tóm tắt.** Sinh code từ `.feature approved` + tech-design, gắn `@trace` ở boundary, verify build, cập nhật trace state — với checkpoint "AI đã hiểu đúng chưa" trước khi chạy.
> **Commands:** `/generate-code` · `/review-code` · `/fix-bug`

| | |
|---|---|
| **Giai đoạn** | Implementation |
| **Owner** | 👤 Developer |
| **Đầu vào** | `.feature approved` + tech-design + CLAUDE.md |
| **Đầu ra** | Code có `@trace` + trace row trong `.tsv` |
| **HITL** | 🟡 Mỏng — 🛑 comprehension checkpoint |

---

## Mục đích (Purpose)

Code là **hệ quả của spec, không phải nguồn**. Bước này biến scenario thành code sao cho:

- Mỗi boundary mang **`@trace`** → đọc code biết ngay phục vụ scenario nào.
- **Drift per-UC** → không đập code refactor mù; chỉ chạm phần lệch.
- **Build verify** sẵn (≤3 retry) → không nhận code không build được.
- Không sinh code cho file **không** có `.feature` backing (trừ `/fix-bug`, `/debug`).

---

## Input (Đầu vào)

- **`.feature approved`** (hoặc UC-ID) — target.
- **Tech-design** `approved` (§4 làm nguồn contract cho shape DTO/endpoint/error).
- `CLAUDE.md` §2 (thứ tự layer, package strategy) + §3 (coding standards) + §5 (error handling) — **service overlay thắng**.
- `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` — để so drift.

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| File code | Theo thứ tự layer của stack, tag `@trace.implements/source` ở boundary |
| `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` | Trace row cập nhật: `status`, `implemented_by`, `bdd_version`… |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Dev** | Xác nhận comprehension checkpoint; xem report; gọi `/review-code` nếu cần |
| 🤖 **AI** | Phát hiện drift, sinh code theo layer, tag trace, build verify, ghi `.tsv` |
| 👤 **Lead** | `/review-code` (read-only) khi cần soát kỹ |

---

## Câu hỏi cần trả lời (Questions this step answers)

- So với lần sinh trước: SC nào **mới**, **drifted**, hay **synced** (bỏ qua)?
- Code phải theo **layer/package/convention** nào của stack này?
- Contract (DTO/endpoint/error) lấy từ đâu — tech-design §4 hay System BDD?
- Build có **pass** không?

---

## Framework xử lý thế nào (Mechanics)

1. **Drift detection** — so `bdd_version`/spec với `.tsv`: phân loại **new (UNTRACKED)** / **DRIFT** / **OK (synced-skip)**.
2. 🛑 **Comprehension checkpoint** (mềm): *"{X} new, {Y} drifted, {Z} synced-skip — Proceed?"* → tránh AI hiểu sai mà vẫn chạy.
3. **Scope Lock** — chỉ implement UC target; code của UC khác trong file dùng chung là **bất khả xâm phạm** (đọc `@trace.implements` để bảo toàn, không xoá).
4. **Generate** theo **thứ tự layer** (vd Controller → Facade → Service → Repository) từ CLAUDE.md §2; tag `@trace` chỉ ở **boundary** (controller/handler), shared code dò qua import chain.
5. **Build verify** — chạy `{conventions.build_command}`, ≤3 retry.
6. **Ghi trace row** vào `.tsv` (trong spec repo nếu umbrella — thao tác ghi liên-repo).

**Phase cho Frontend:**
| Phase | Ý nghĩa |
|-------|---------|
| `--phase=ui` | FE Phase 1 — sinh UI + layer **mock API** từ System BDD contract |
| `--phase=integration` | FE nối vào **BE contract thật** (đọc tech-design từ spec repo) |
| *(default)* | Backend — sinh từ tech-design §4 |

**`/review-code`** — read-only, chỉ báo findings (không tự sửa: "AI tự fix tự review" = lặp lỗi).
**`/fix-bug`** — sửa lỗi có root-cause + regression test; xem [Feedback Loop](10-feedback-loop.md).

---

## HITL / Gate

- 🛑 **Comprehension checkpoint** — điểm dừng chính: Dev xác nhận phân loại drift đúng trước khi sinh.
- Build phải **pass** trước khi commit.
- `/review-code` là gate read-only tuỳ chọn — không auto-apply fix.

---

## Ví dụ (Example)

```
/generate-code UC-02
  Drift: 1 new (SC-02.3), 1 drifted (SC-02.1 bdd_version 1.0→1.1), 2 synced-skip
  🛑 Proceed? (Y/N) → Y
  → sinh/patch code, @trace.implements ở PasswordResetController
  → build verify ✅ (attempt 1)
  → cập nhật .trace/auth/quen-mat-khau/UC-02-web.tsv
```

---

## Anti-pattern

- ❌ Sinh code cho file không có `.feature` backing (ngoài fix-bug/debug).
- ❌ Tái tạo file dùng chung "chỉ gồm scenario UC này" → xoá nhầm nghiệp vụ UC khác.
- ❌ Tag `@trace` mọi file → tag explosion; chỉ tag boundary.
- ❌ Lưu version trong code — version chỉ ở spec; code dùng `.tsv`.

---

## Bước tiếp theo (Next step)

Code build được → dev tự kiểm nhanh:

➡️ [Bước 7 · Dev self-test — `/dev-gen-test` · `/dev-run-test` · `/dev-smoke-test`](07-dev-selftest.md)
