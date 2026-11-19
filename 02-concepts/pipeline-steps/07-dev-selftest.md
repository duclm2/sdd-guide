[← Code](06-code.md) · [Pipeline Steps](README.md) · [Next: QC Automation →](08-qc-automation.md)

# Bước 7 · Dev Self-test — Tự kiểm nhanh của Dev (Developer Smoke)

> **Tóm tắt.** Dev tự sinh & chạy bộ **smoke test nhanh** ngay sau khi code, đặt cột `dev_selftest`. Đây là smoke của **dev**, độc lập với QC chính thức.
> **Commands:** `/dev-gen-test` → `/dev-run-test` → `/dev-smoke-test`

| | |
|---|---|
| **Giai đoạn** | Dev self-check |
| **Owner** | 👤 Developer |
| **Đầu vào** | Code vừa sinh (bước 6) + `.feature` |
| **Đầu ra** | Test smoke + cột `dev_selftest` trong `.tsv` |
| **HITL** | 🟡 Mỏng |

---

## Mục đích (Purpose)

Trước khi đẩy sang QC chính thức, Dev cần một vòng **kiểm nhanh tại chỗ** để bắt lỗi hiển nhiên — không phải bộ test đầy đủ. Bước này:

- Sinh bộ **self-test** bám scenario, chạy nhanh theo platform.
- Đặt cột **`dev_selftest`** để biết SC nào đã qua smoke của dev.
- Cho phép **thử tại chỗ** trên service/app đang chạy (`/dev-smoke-test`).

> **`dev_selftest` ≠ `qc_status`.** Hai trục **độc lập**: dev smoke (nhanh, tự kiểm) vs QC chính thức (Playwright · WebdriverIO + Appium, evidence). Không lấn quyền nhau.
>
> ⚠️ **Nhưng "độc lập" chỉ đúng với `status` về KẾT QUẢ CHẠY, không đúng về QUYỀN KHẲNG ĐỊNH** *(GAPS-v4 G55)*. `pass` mang nghĩa *"scenario này đã được nghiệm thu theo spec **hiện tại**"* — nên trên row `DRIFT`/`ORPHANED`, `/dev-run-test` và `/qc-run-script` và `/qc-run-manualtest` **không được** ghi `pass`; chúng hạ về `not_run`. `fail`/`skip` thì ghi bình thường. `lint-trace` **T12** bắt trạng thái `DRIFT + pass` ở sổ thật, bất kể ai ghi ra.

---

## Input (Đầu vào)

- Code đã sinh + build được (bước 6).
- `.feature` target (UC-ID / platform).
- `active_module` + `platform_type` (chẩn lỗi theo stack).

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| File test smoke | Bộ tự-kiểm nhanh do `/dev-gen-test` sinh |
| Cột `dev_selftest` trong `.trace/…/{UC-ID}-{platform}.tsv` | Kết quả smoke của dev |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Dev** | Chạy self-test, đọc chẩn lỗi, sửa code nếu fail |
| 🤖 **AI** | Sinh test smoke, chạy + chẩn lỗi theo platform, set `dev_selftest`, thử tại chỗ |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Code vừa sinh có **chạy được / pass smoke** không?
- Lỗi (nếu có) nằm ở đâu, do platform gì?
- SC nào đã qua vòng tự kiểm của dev (`dev_selftest`)?

---

## Framework xử lý thế nào (Mechanics)

| Lệnh | Việc |
|------|------|
| `/dev-gen-test` | Sinh bộ self-test nhanh bám scenario |
| `/dev-run-test` | Chạy + **chẩn lỗi theo platform**, set cột `dev_selftest` |
| `/dev-smoke-test` | Thử **tại chỗ** trên service/app đang chạy — **không ghi file nào** (không đụng sổ trace) |

- Route theo `active_platform` (flat hoặc map-theo-platform) để chạy đúng service.
- Trace ghi vào `.tsv` (spec repo nếu umbrella).

---

## HITL / Gate

- 🟡 Mỏng — Dev chủ động chạy; không có gate chặn downstream. Đây là bước *tự tin trước khi giao QC*.

---

## Anti-pattern

- ❌ Coi `dev_selftest` là thay QC chính thức — nó chỉ smoke, không có evidence.
- ❌ Bỏ qua vì "build đã pass" — build pass ≠ hành vi đúng.

---

## Bước tiếp theo (Next step)

Smoke ổn → chuyển sang dây chuyền QC chính thức:

➡️ [Bước 8 · QC Automation — `/qc-analyze` … `/qc-report`](08-qc-automation.md)
