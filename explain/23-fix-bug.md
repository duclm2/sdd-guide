[← /generate-spec-manifest](22-generate-spec-manifest.md) · [Explain Home](README.md) · [Next: /debug →](24-debug.md)

# 23 · `/fix-bug` — Workflow sửa bug đầy đủ

> **Một câu.** Sửa bug **có kỷ luật**: truy root cause → fix (tag `@trace.fixes/root_cause/regression`) → regression test → build → commit (2 tầng ở umbrella) → đóng bug report → đề xuất lesson.

---

## Vấn đề giải quyết

Sửa bug ad-hoc dễ tái phát và mất truy vết. Command áp một quy trình: hiểu gốc rễ, sửa có tag, thêm regression test để không lặp, và ghi lesson nếu là lỗi tái diễn.

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (ngoài pipeline tuyến tính).
- **Đặc biệt:** một trong số ít lệnh được sinh code **không cần `.feature` backing** (bug fix).

---

## Input / Output

**Input:** bug report spec-anchored (`{BUG-ID}`) hoặc mô tả lỗi + code liên quan.

**Output:** code fix (tag trace) + regression test + commit + bug report đóng.

---

## Các bước xử lý (chi tiết)

1. **Gate** — xác định service & code liên quan; tạo branch `fix/{TICKET}-{slug}`.
2. **Phase 2 · Root Cause Analysis** — truy nguyên nhân gốc (không vá triệu chứng).
3. **Phase 3 · Fix** — sửa; tag `@trace.fixes` / `@trace.root_cause` / `@trace.regression`.
4. **Phase 4 · Regression Test** — thêm test tái hiện bug để chống tái phát.
5. **Phase 4.5 · Cập nhật sổ trace** — regression test phải hiện lên coverage (xem dưới).
6. **Phase 5 · Build & Commit** — build verify; umbrella **push 2 tầng** (Tầng 1: fix branch trong service submodule nơi code sống; Tầng 2: umbrella pointer).
7. **Phase 5.5 · Đặt `🟡 Fixed`** — nếu fix một `{BUG-ID}` đã file. **Không** đặt `Closed` — bước đó thuộc `/qc-run-script`.
8. **Phase 6 · Đề xuất Lesson** — nếu lỗi tái diễn → `capture-lesson` (L1–L5).

### Phase 4.5 — vì sao `/fix-bug` phải ghi sổ trace

Đây từng là lệnh **duy nhất** sinh test mà không ghi `.tsv`. Hệ quả: `test_count` under-report vĩnh viễn → SC đứng `GAP` dù vừa có regression test → `/validate-traces` khuyên `/dev-gen-test` → dev sinh test **trùng**. Và `dev_selftest` giữ `pass` **cũ trên code đã đổi**.

| Cột | Ghi gì |
|---|---|
| `test_count` | **+=** số test regression (cộng dồn, không ghi đè) |
| `test_classes` | **append** tên class mới, giữ tên cũ |
| `dev_selftest` → `not_run` · `dev_selftest_at` → `—` | code vừa đổi nên tín hiệu self-test cũ hết hiệu lực |
| `last_updated` | hôm nay |

**Hai nhóm cột cấm đụng:** `qc_*` (QC sở hữu — `/qc-run-script` và `/qc-run-manualtest` flip khi re-verify; riêng `/qc-run-script` đóng bug) · `spec_ver`/`gen_ver` (**fix bug không đổi spec** — đụng vào là tạo `DRIFT` giả).

Vì `dev_selftest` bị reset, Next của lệnh là **`/dev-run-test`** để lấy lại tín hiệu xanh, rồi mới tạo PR.

---

## Checkpoint & Gate

- Commit **sau khi user duyệt**.
- Build phải pass.

---

## Cơ chế đặc biệt

- **Root cause bắt buộc** trước fix — chống vá triệu chứng.
- **Regression test bắt buộc** — mỗi bug fix để lại "bằng chứng chống tái phát".
- **Push 2 tầng umbrella** — code ở submodule, pointer ở umbrella.
- **Đóng bug report + đề xuất lesson** — khép vòng feedback.

---

## 👓 Góc nhìn tối ưu

- **Sinh code không cần `.feature`** — ngoại lệ có chủ đích cho bug, nhưng nghĩa là fix có thể lệch spec. Đáng cân nhắc có nên yêu cầu cập nhật BDD nếu fix đổi hành vi.
- **Root cause phụ thuộc AI phân tích** — với bug phức tạp có thể cần `/debug` trước.
- **Push 2 tầng** dễ sai thứ tự (submodule trước, pointer sau) — điểm nhạy cảm.
- **Regression test có gắn `@trace.verifies` không?** — đáng đảm bảo để validate-traces thấy.

---

## Kết nối

**Trước:** [`/report-bug`](25-report-bug.md) / [`/debug`](24-debug.md) / [`/dev-run-test`](13-dev-run-test.md) (fail) · **Sau:** tạo PR + link ticket; lỗi lặp → [`/learn`](27-learn.md).
