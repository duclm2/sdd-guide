[← /generate-bdd](06-generate-bdd.md) · [Explain Home](README.md) · [Next: /review-tech-docs →](08-review-tech-docs.md)

# 07 · `/generate-tech-docs` — Sinh Technical Design (full-stack, gộp/PRD)

> **Một câu.** Sinh **một** tech-design full-stack cho cả PRD (backend API/data/DB **và** client), theo chế độ Fresh/Append tăng dần, với self-review REFUTE và cổng ký liên team.

---

## Vấn đề giải quyết

Đây là ranh giới nghiệp-vụ → kỹ-thuật. Tech-design là **API contract** giữa các team (BE viết, FE/App đọc). Chốt sai = đốt budget cả hai phía. Command chuẩn hoá entity/API/dependency và phủ mọi UC trong một doc.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Design (đầu ra kỹ thuật), sau BDD.
- **Gate vào:** cổng chất lượng — BDD phải sạch finding critical + cảnh báo mềm nếu chưa `approved`.

---

## Input / Output

**Input:** các `.feature` (batch tech lead trỏ vào) + review-bdd findings + design-spec (cho §4.5 client) + entity catalog + Figma component catalog.

**Output:** `{specs_dir}/{domain}/{prd-slug}/tech-docs/{TICKET-ID}-tech-design.md` — **một** doc 12 section.

---

## Các bước xử lý (chi tiết)

1. **Bước 1 · Fresh vs Append** — chưa có doc → **FRESH**; đã có → **APPEND** (tăng dần, **không regenerate** để khỏi mất chỉnh tay + sign-off). Đọc §10 UC Coverage → `covered_ucs`; mỗi UC batch phân loại `add-new` / `extend-platform` / `refresh` (hỏi Y/N) / `skip`.
2. **Bước 2 · Cổng Chất lượng** — mỗi feature: tìm `{uc-id}-{platform}-review-bdd-findings.yaml`; còn critical `pending` → **DỪNG** (chạy `--fix`/`--resume` trước); `@trace.status ≠ approved` → cảnh báo mềm Y/N.
3. **Bước 3 · Điều kiện tiên quyết Client** (batch có web/app) — nạp design-spec cho §4.5 (component, Figma map, state, selector); thiếu → §4.5 degraded `[DRAFT — no design-spec]`.
4. **Bước 4 · Brownfield** — `@trace.api_source: existing` → **reverse-document** (mô tả as-is, ghi gap vs BDD, không thiết kế mới); else **greenfield** (thiết kế từ scenario).
5. **CHECKPOINT** — trình kế hoạch (mode, batch, platform, API mode, section sẽ sinh) → chờ Y.
6. **Sinh** doc 12 section: §1 Overview · §2 Architecture · §3 Data Model · §4 API Contracts (+§4.5 client design) · §5 Key Flows (sequence, lane 5.A system/5.B web/5.C app) · §6 Integration · §7 Security · §8 Error Handling · §9 Design Decisions · **§10 UC Coverage** (khoá theo platform×SC) · §11 Cross-cutting · **§12 GAP Register** (ẩn số chưa chốt).
7. **Self-Review Gate — REFUTE** — trước khi ghi, tự phản biện doc (bắt buộc, cả Fresh lẫn Append).
8. **Công bố** (umbrella) — chia sẻ doc vào spec repo dùng chung.

---

## Checkpoint & Gate

- 🛑 CHECKPOINT kế hoạch + xác nhận refresh UC đã phủ.
- 🔒 Cổng chất lượng BDD (chặn nếu còn critical).
- Sinh với `@trace.status: draft` + `@trace.sign_off` (be/fe/app/sa = pending) → `/review-tech-docs` T7.

---

## Cơ chế đặc biệt

- **Append tăng dần, không regenerate** — bảo toàn chỉnh tay & sign-off; §10 UC Coverage là "sổ" theo dõi UC đã phủ.
- **Một doc full-stack** — BE §4 + client §4.5 sinh cùng nhau (bỏ cổng "BE contract trước"); §4.5.4 vẫn phải map endpoint thật ở §4.1.
- **§12 GAP Register** — ghi ẩn số thiết kế chưa chốt (blocker 🔴 chạm UC → generate-code WARN).
- **REFUTE self-review** — vòng tự phản biện trước khi ghi.

---

## 👓 Góc nhìn tối ưu

- **Append phức tạp** — phân loại add-new/extend-platform/refresh/skip theo platform×UC, đánh số sequence trong đúng lane. Đây là logic dày nhất; dễ sai khi nhiều platform. Đáng review độ chắc của việc merge.
- **Một doc/PRD** dễ điều hướng nhưng có thể rất lớn với PRD nhiều UC × nhiều platform → chạm giới hạn context khi Append đọc lại.
- **§12 GAP Register + sign-off** là cơ chế "chốt dần" tốt — nhưng phụ thuộc con người cập nhật trạng thái.
- **Cổng chất lượng chỉ quét batch** (feature tech lead trỏ) — nếu bỏ sót feature, coverage thủng mà không báo.

---

## Kết nối

**Trước:** [`/generate-bdd`](06-generate-bdd.md) + [`/review-context {feature}`](04-review-context.md) · **Sau:** [`/review-tech-docs`](08-review-tech-docs.md).
