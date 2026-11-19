[← /refine-prd](03-refine-prd.md) · [Explain Home](README.md) · [Next: /generate-design-spec →](05-generate-design-spec.md)

# 04 · `/review-context` — Gate chất lượng PRD & BDD (findings có mã)

> **Một câu.** Cổng chất lượng cuối ở thượng nguồn: fan-out review có mã (**P0–P5** cho PRD, **B1–B6** cho BDD), sạch critical mới mở khoá downstream. Có 3 chế độ: phân tích · `--fix` (auto-apply) · `--resume` (áp finding đã accept).

> ⚠️ **Một lệnh, hai đối tượng.** Target là PRD → **PRD Review Mode**; target là `.feature` → **BDD Review Mode**. Nó chạy ở **hai vị trí** trong pipeline (sau PRD, và sau BDD).

---

## Vấn đề giải quyết

Artifact kém ở thượng nguồn phá huỷ hạ nguồn. `/review-context` là **gate bằng findings**: phân loại lỗi theo mã, chặn tới khi sạch critical + người đặt trạng thái `approved`. Khác `/refine-prd` (phán đoán chất lượng 4 lăng kính), đây là **check có mã, có auto-fix**.

---

## Vị trí & tiền đề

- **Vị trí:** sau `/generate-prd`/`/refine-prd` (PRD mode) **và** sau `/generate-bdd` (BDD mode).
- **Gate ra:** sạch critical → người đặt `Status: approved` (PRD) / `@trace.status: approved` (BDD) → mở khoá bước kế.

---

## Input / Output

**Input:** PRD hoặc `.feature` + business-dictionary + core-entities (+ config umbrella cho P0).

**Output:** file findings —
- PRD: `{refinement_dir}/{prd-slug}-review-context-findings.yaml`
- BDD: `{refinement_dir}/{uc-id}-{platform}-review-bdd-findings.yaml`

---

## Các bước xử lý (chi tiết)

### 1 · Detect Review Mode
Target PRD (`.md` ở gốc feature) → PRD mode; target `.feature` → BDD mode.

### 2 · Chạy review-fanout (Phase 1 → 2 → 3)
Giống `/refine-prd` (fan-out song song → completeness-critic → dedup/merge, `GRANULARITY = per-uc`), nhưng DIMENSIONS là các mã check.

**Full vs delta:** lần đầu → FULL; `--full` → FULL, bỏ qua findings cũ; lần sau so `source_version` (PRD `Version` / BDD `@trace.bdd_version`) — chưa đổi → DỪNG; đổi đúng bằng phần chính lệnh này áp (`applied_to_version` khớp) → DELTA; đổi bởi actor khác → FULL + cảnh báo.

**PRD Review Mode — P0–P5:**
| Mã | Check | Ghi chú |
|----|-------|---------|
| **P0** | Umbrella routing (chỉ umbrella) | **Orchestrator chạy** (cần config); là gate check |
| **P1** | Terminology (banned/canonical) | fan-out; **auto-fixable** |
| **P2** | Ambiguity | fan-out; **KHÔNG auto-fixable** (người viết fix) |
| **P3** | Domain conflict (cross-PRD) | **Orchestrator chạy** (cần PRD khác) |
| **P4** | Structural completeness | fan-out; **auto-fixable** (thêm skeleton) |
| **P5** | Custom criteria | fan-out (tuỳ chọn) |

**BDD Review Mode — B1–B6 (đều fan-out):**
| Mã | Check |
|----|-------|
| **B1** | PRD coverage (mọi AC/UC được phủ) |
| **B2** | Terminology & entity |
| **B3** | Gherkin rules (R1–R10) |
| **B4** | Compliance (C.1–C.5) |
| **B5** | Metadata & structural (@trace header, Coverage Matrix) — xem 3 nhóm dưới |
| **B6** | Side-effect completeness |

**B5 — field `@trace.*` chia 3 nhóm theo mức thiệt hại, không phải một danh sách phẳng:**

| Nhóm | Mức | Field |
|---|:---:|---|
| **A — chặn** | `major`, auto-fixable | `id` · **`platform`** · `domain` · `prd` · `prd_version` · `bdd_version` · `status` |
| **B — thông tin** | `minor` | `title` · `revision` · `author` · `created_at` · `business_rules` · `dataset` |
| **C — có điều kiện** | chỉ flag khi đúng điều kiện | `module` (chỉ **umbrella**; giá trị `unknown` hợp lệ) · `api_source` (chỉ `platform = system` + PRD brownfield) |

- **`@trace.service` đã rời nhóm C — giờ bắt buộc MỌI mode (`major`)**: nó là nguồn duy nhất của cột `service` trong sổ TSV. Giá trị hợp lệ: path service · `multi` · `unresolved` · `—` (single-service / spec repo). Umbrella không suy được → `unresolved` kèm finding major (lỗi cấu hình routing thật, không che).

- **`@trace.platform` là major chứ không minor** — thiếu nó thì `/generate-code` không quyết được BE/FE (và **cấm** fallback sang `platform_type`), không định vị được sổ trace, không tìm được design-spec. Auto-fix suy từ segment `bdd/{platform}/` của chính path file, **không đoán**.
- **`@trace.sc_version` của scenario cũng là major** (ngoại lệ trong nhóm scenario-tag vốn minor): thiếu nó thì SC đó **vĩnh viễn hiện `OK`** dù scenario đổi bao nhiêu lần.
- **Nhóm C không được flag khi sai điều kiện.** Trước đây B5 đòi `module` vô điều kiện → mọi `.feature` **đúng-theo-template** ở spec repo mode ăn 2 finding minor, và `--fix` **thêm field bịa** vào header.
- B5 cũng bắt **tag lạc** rò vào BDD canonical: `@trace.uc=` / `@trace.ac=` (vocabulary proposal cũ) và `@proposed` / `@from-test`.

### 3 · Ghi file findings + Post-Analysis Routing
Sạch critical → nhắc người đặt `approved`; còn critical → giữ draft, sửa.

### Chế độ `--fix` (auto-apply)
Chạy full phân tích rồi **áp ngay** các finding `auto_fixable: true` (không qua Board): PRD (P1 banned term, P1 tech-term reframe, P4 skeleton) · BDD (B2 terminology, B3 R3/R7/R9/R10, B4 C4/C5, B5 header/matrix, B6 side-effect). Finding cần người vẫn để `pending`.
- **Version bump + reset draft:** ≥1 fix áp → PRD bump minor + **reset `Status: draft`** + Changelog; BDD tăng `@trace.bdd_version` 0.1 + **reset `@trace.status: draft`**. Ghi `applied_to_version`.
- **Row changelog PRD phải nêu UC (`{changelog_scope}`)** — gom finding đã áp theo `uc_id` (`""` → `PRD-global`), mỗi nhóm một mệnh đề `{uc_id}: …`; nhóm chỉ gồm fix **không đổi hành vi** (P4 skeleton) gắn hậu tố `[no-behavior]`, còn P1 (banned term, tech-term) tính là **đổi hành vi**. Nhờ vậy `/validate-traces` chỉ gắn 🟠 `PRD_DRIFT` cho UC bị sửa thật, UC khác ở lại ⓘ `PRD_STALE_REF`. Bản cũ ghi `Auto-fix: applied {N}…` không nêu UC → **mọi** UC ăn 🟠 (G52). `--resume` dựng dòng này theo cùng luật; BR/AC luôn đi kèm UC sở hữu.
- **Bump `@trace.sc_version` theo TỪNG scenario** — không phải cả file. Finding **đổi thân scenario** (R3 diễn đạt lại step · R7 thay giá trị · R9 thêm cột data table · R10 thêm Note · B6 thêm side-effect · B2 đổi tên entity/field trong step) → bump `+0.1` cho **đúng SC đó**. Finding chỉ sửa header / Coverage Matrix / gom NHÓM (B4, B5) → **không** bump.
  - Đây là tín hiệu **duy nhất** cho `/validate-traces` biết code của SC đó lỗi thời (`spec_ver != gen_ver` → `DRIFT`). `bdd_version` ở cấp file **không đủ phân giải** để biết SC nào cần regen.
  - Bump vô cớ thì ngược lại: mọi SC hiện `DRIFT` giả và cờ mất giá trị.
  - `--resume` còn cập nhật `spec_ver` trong `.tsv` cho SC vừa bump (giữ `gen_ver` → row hiện `DRIFT` đúng như mong đợi), và **append row** cho SC mới do B1 sinh.

### Chế độ `--resume`
Áp các finding `accepted`/`modified` từ Board; bỏ `rejected`/`deferred`; `needs_discussion` → cảnh báo, bỏ lần này.

---

## Checkpoint & Gate

- Nhãn khai **chặn CỨNG** — nhưng chỉ nổ khi file findings **đã có** finding mang `accepted`/`modified`/`rejected` (đã qua Review Board), hoặc khi chạy `--full` (bỏ findings cũ) / `--resume` (áp fix thẳng vào target). `--yes` **không** bỏ qua được. Lần phân tích đầu → đi thẳng.
- 🔒 **Gate bằng trạng thái** — sạch critical là điều kiện *cần*; người đặt `approved` là điều kiện *đủ*.
- 🛑 Finding cần người → Review Board (accept/reject/modify) → `--resume`.

---

## Cơ chế đặc biệt

- **`--fix` reset con dấu duyệt** — auto-fix làm target đổi sau khi duyệt → status về `draft`, phải duyệt lại (đồng bộ với `/refine-prd`).
- **P0/P3 do orchestrator chạy** (không fan-out) vì cần config/PRD khác; P1/P2/P4/P5 mới fan-out.
- **Ngôn ngữ finding thuần nghiệp vụ** — cả PRD và BDD (BDD đứng trước tech-docs); `quote` được miễn (trích nguyên văn, kể cả Gherkin).

---

## 👓 Góc nhìn tối ưu

- **Chồng lấn với `/refine-prd`** ở PRD mode — cùng review-fanout, cùng review PRD. Ranh giới: `/refine-prd` = 4 lăng kính phán đoán (không auto-fix); `/review-context` = mã P-check (có auto-fix). Đáng review có nên hợp nhất/rõ ranh giới hơn.
- **`--fix` reset draft** đúng về nguyên tắc nhưng tạo vòng "fix → reset → duyệt lại" — với PRD ổn định chỉ cần dọn thuật ngữ, chi phí duyệt lại đáng cân nhắc.
- **B1 coverage phụ thuộc chất lượng PRD** — nếu PRD mơ hồ, "phủ đủ" khó đo. B4 C.2 cố ý dedup với B1.
- **Cùng lệnh 2 mode PRD/BDD** giữ code chung nhưng logic rẽ nhánh dày — điểm cần chú ý khi bảo trì.

---

## Kết nối

**Trước:** [`/refine-prd`](03-refine-prd.md) (PRD) / [`/generate-bdd`](06-generate-bdd.md) (BDD) · **Sau:** PRD approved → [`/generate-design-spec`](05-generate-design-spec.md) (FE/App) hoặc [`/generate-bdd`](06-generate-bdd.md); BDD approved → [`/generate-tech-docs`](07-generate-tech-docs.md).
