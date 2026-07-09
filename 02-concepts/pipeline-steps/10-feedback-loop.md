[← Validate Traces](09-validate-traces.md) · [Pipeline Steps](README.md) · [Overview →](../overview.md)

# Bước 10 · Feedback Loop — Vòng phản hồi & tự học (Feedback & Learning)

> **Tóm tắt.** Bug, đề xuất scenario và bài học được đưa vào các **kênh có hồ sơ**, nổi lên qua `/sync`, và nạp lại vào context — **không tạo loop ngầm** mà cải tiến spec & tri thức dự án.
> **Commands:** `/report-bug` · `/propose-scenario` · `/learn` · `/sync` · `/fix-bug`

| | |
|---|---|
| **Giai đoạn** | Cross-cutting (xuyên suốt) |
| **Owner** | 👤 Tester / QA (và tất cả) |
| **Đầu vào** | Bug, gap, định hướng lặp lại |
| **Đầu ra** | `feedback/*`, `project-lessons.md`, Living Docs |
| **HITL** | ⚪ Kênh có hồ sơ |

---

## Mục đích (Purpose)

Framework là pipeline **một chiều** — nhưng vẫn cần đường **phản hồi ngược có kiểm soát** để hệ thống *học* qua thời gian. Nguyên tắc: định hướng/guardrail lặp lại **phải được ghi**, đừng để nằm trong đầu vài senior. Bước này:

- Cho tester/QC một **kênh feedback có hồ sơ spec** (bug, scenario proposal).
- Biến bài học thành **lesson dán tường** (`project-lessons.md`) — nạp lại để không sai lại.
- **Nổi** feedback lên PO/Dev qua `/sync`.

> Đây **không phải loop** để chạy vòng vòng — mỗi phản hồi đi vào spec hoặc tri thức rồi pipeline lại chảy một chiều.

---

## Các kênh (Channels)

| Lệnh | Ai dùng | Kết quả | Đi về đâu |
|------|---------|---------|-----------|
| `/report-bug` | Tester / QC | Bug **spec-anchored** (gồm product-gap từ `/qc-*`) | `feedback/bug-reports/` |
| `/propose-scenario` | Tester / QC | Đề xuất BDD scenario mới | `feedback/bdd-proposals/` |
| `/learn` | Tất cả | Guardrail lesson | `project-lessons.md` (qua step `capture-lesson`) |
| `/fix-bug` | Dev | Sửa lỗi có root-cause + regression test | Code + `@trace.fixes/root_cause/regression` |
| `/sync` | Lead (umbrella) | Pull + submodule + **nổi feedback** + làm mới Living Docs | Chạy hằng ngày |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Tester/QC** | `/report-bug`, `/propose-scenario` — feedback có hồ sơ |
| 👤 **Tất cả** | `/learn` khi thấy định hướng lặp lại |
| 👤 **Dev** | `/fix-bug` — truy gốc → sửa → regression test |
| 👤 **Lead** | `/sync` — đồng bộ umbrella, nổi feedback |
| 🤖 **AI** | Ghi feedback đúng schema, incorporate proposal `accepted` vào `/generate-bdd`, nạp lại lessons |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Bug này gắn với **scenario/spec** nào (spec-anchored)?
- Scenario còn thiếu nào cần đề xuất bổ sung?
- Định hướng nào **lặp lại** đủ để thành lesson dán tường?
- (Umbrella) feedback nào cần **nổi lên** cho PO/Dev?

---

## Framework xử lý thế nào (Mechanics)

1. **`/report-bug` / `/propose-scenario`** — ghi feedback kèm tham chiếu spec vào `feedback/` (spec repo nếu umbrella).
2. **`/learn`** — qua step `capture-lesson`, ghi guardrail vào `project-lessons.md`; các workflow sau **nạp lại** vào context → hệ thống *nhớ*.
3. **`/generate-bdd`** — có thể incorporate scenario proposal đã `accepted`.
4. **`/fix-bug`** — đọc bug spec-anchored → tạo branch `fix/{TICKET}-<slug>` → root cause → sửa (tag `@trace.fixes/root_cause/regression`) → regression test + build verify → commit sau khi user duyệt.
5. **`/sync`** — pull, init submodule, **nổi feedback** lên PO/Dev, bootstrap config service, làm mới Living Docs. An toàn chạy lặp lại.

---

## HITL / Gate

- ⚪ Không gate chặn — nhưng là nơi **con người quyết** điều gì đáng thành spec/lesson.
- `/fix-bug` commit **sau khi user duyệt**.

---

## Ví dụ (Example)

```
QC phát hiện: link reset vẫn dùng được sau khi đổi mật khẩu
  /report-bug  → feedback/bug-reports/BUG-217.md  (@trace tới UC-02/SC-02.1)
  /sync        → nổi lên cho Dev
  /fix-bug BUG-217 → branch fix/BUG-217-invalidate-link
                     → root cause: thiếu invalidate token
                     → fix + regression test + build ✅
  /learn "luôn invalidate one-time token sau khi dùng"
                     → project-lessons.md (nạp lại lần sau)
```

---

## Anti-pattern

- ❌ Sửa định hướng bằng miệng rồi để AI "quên" — phải `/learn`.
- ❌ Bug không gắn spec → khó truy vết, khó regression.
- ❌ Coi feedback loop là "chạy vòng lặp" — nó cải tiến spec/tri thức, pipeline vẫn một chiều.

---

## Hết pipeline (End of pipeline)

Bạn đã đi hết 11 bước (0→10). Xem lại bức tranh lớn ở [Overview](../overview.md), hoặc quay về [danh sách các bước](README.md).
