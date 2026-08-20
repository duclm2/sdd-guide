[← /extend-prd](02b-extend-prd.md) · [Explain Home](README.md) · [Next: /review-context →](04-review-context.md)

# 03 · `/refine-prd` — Tinh chỉnh PRD qua 3 lăng kính

> **Một câu.** Fan-out review PRD qua **3 lăng kính DEV / SA / PO**, chạy **vòng lặp completeness-critic** để hội tụ đầy đủ trong một lần, rồi sinh file findings cho PO accept/reject ở Review Board.

---

## Vấn đề giải quyết

Một lượt review đơn không bao giờ liệt kê hết vấn đề — model dừng ở mức "đủ", nên mỗi vòng sau lại lòi lỗi mới (**đập chuột chũi**). `/refine-prd` ép review **hội tụ trong một lần chạy**, bắt lỗi nghiệp vụ *trước* khi truyền xuống BDD, giữ altitude & ngôn ngữ nghiệp vụ.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Specification (sau `/generate-prd`).
- **Tiền đề:** có PRD draft.
- **Đặc biệt:** có **Resume Mode** (`--resume`) áp findings đã accept và bump version PRD.

---

## Input / Output

**Input:** PRD + core-entities + business-dictionary.

**Output:** `{refinement_dir}/{prd-slug}-findings.yaml` — findings với `lens` (DEV/SA/PO), severity, `quote`+`uc_id` (để Review Board jump-to-source), `suggestion`, `resolution_edge_cases`, `status`.

---

## Các bước xử lý (chi tiết)

Chạy qua step **review-fanout** với tham số `GRANULARITY = per-uc`:

### Phase 1 — Fan-out song song theo dimension
- **DIMENSIONS = 3 lăng kính** (mỗi lăng kính một sub-agent, context window mới, quét toàn PRD chỉ qua lăng kính đó):
  | Lăng kính | Soi gì |
  |-----------|--------|
  | **DEV** (cơ chế nghiệp vụ) | BR + Business Logic đã đủ & không mơ hồ để build không phải đoán chưa? Nhánh nghiệp vụ thiếu, điều kiện biên, đường lỗi bỏ ngỏ |
  | **SA** (thông suốt & nhất quán) | Luồng nghiệp vụ thông suốt trên cả feature/domain? Tương tác UC, quan hệ entity, vòng đời trạng thái, ai-làm-gì |
  | **PO** | Scope khoanh vùng? Priority? Success metric? Rủi ro scope creep? |
- ⚠️ **Nguyên tắc DEV & SA: đọc bằng mắt kỹ thuật, VIẾT bằng lời nghiệp vụ** — chỉ nêu *cái nghiệp vụ còn thiếu/mơ hồ* + đặt câu hỏi làm rõ; KHÔNG đề xuất cơ chế kỹ thuật.
- `GRANULARITY = per-uc` → luôn fan-out `DIMENSION × UC` (+ phạm vi PRD-global), bỏ ngưỡng cả-file → **lần đầu quét sâu**. Agent cap = 12/wave, gom batch UC nếu vượt.

### Phase 2 — Vòng lặp completeness-critic
- Spawn một critic đọc **toàn PRD** + danh sách findings đã có (slim) → liệt kê **chỉ vấn đề mới** (gap, mâu thuẫn, edge/negative path thiếu, **vi phạm altitude/role-boundary**: cơ chế nằm trong AC, AC lặp lại BR…).
- Lặp tới khi **2 vòng liên tiếp 0 finding mới** hoặc cap **3 vòng**. Ghi `convergence_rounds`.

### Phase 3 — Dedup / xung đột / merge
- Khử trùng (giữ suggestion phong phú hơn, severity cao hơn); merge được thì merge, loại trừ nhau → một finding `needs_discussion`; sắp theo severity; gán ID `F001…`; map dimension → `lens`; ghi **một** file findings.

### Full vs Delta
- Lần đầu (chưa có findings file) → **FULL**. Lần sau so `prd_version`: chưa đổi → DỪNG; đổi do chính resume này (`applied_to_version` khớp) → **DELTA** (chỉ UC đã đổi + UC mới); đổi bởi actor khác → **FULL** + cảnh báo.

### Resume Mode (`--resume`)
- Áp finding theo `status` (`accepted`/`modified`), bump version PRD, ghi `applied_to_version`. `needs_discussion` chặn resume tới khi người quyết.

---

## Checkpoint & Gate

- 🛑 **Review Board** — PO accept/reject/modify **từng** finding (không auto-apply). Finding lifecycle: `pending → accepted|modified|rejected|needs_discussion|deferred → applied`.
- `recommendation`: critical≥1 → `BLOCKED`; major≥1 → `NEEDS_REVISION`; else `APPROVED_WITH_MINOR_CHANGES`.

---

## Cơ chế đặc biệt

- **Không có `--fix` mode** (khác `/review-context`) — finding 3 lăng kính là phán đoán DEV/SA/PO, **bắt buộc qua người** ở Board; `auto_fixable` chỉ là gợi ý quick-accept.
- **`resolution_edge_cases`** — phân tích bậc-hai (chỉ critical/major): "nếu chốt phương án này thì đẻ ra edge case gì?" → PO thấy trước khi accept (advisory, không chặn).
- **QA lens đang DISABLED** (comment trong file) — có hướng dẫn bật lại nếu cần.

---

## 👓 Góc nhìn tối ưu

- **Đây là command tốn agent/token nhất phía thượng nguồn** — `per-uc` × 3 lăng kính × (UC+1) + tới 3 vòng critic. `AGENT_CAP=12` là núm chỉnh chính. Với PRD lớn, đây là điểm cần cân đối chi phí ↔ độ đầy đủ.
- **Completeness-critic tới 3 vòng** — điểm đáng đo: thực tế hội tụ ở vòng mấy? Nếu thường 1–2 vòng thì cap 3 hợp lý.
- **Full/delta logic phức tạp** (`applied_to_version` tracking) — mạnh nhưng nhiều nhánh; dễ rơi về FULL khi có actor khác sửa PRD (vd `/review-context` xen giữa).
- **Ranh giới với `/review-context`** — cả hai đều review PRD, dùng chung review-fanout. `/refine-prd` = phán đoán chất lượng nghiệp vụ (3 lăng kính); `/review-context` = check có mã P0–P5 + auto-fix. Chồng lấn có chủ đích hay có thể gộp?

---

## Kết nối

**Trước:** [`/generate-prd`](02-generate-prd.md) hoặc [`/extend-prd`](02b-extend-prd.md) · **Sau:** mở Review Board → cập nhật PRD → [`/review-context`](04-review-context.md).
