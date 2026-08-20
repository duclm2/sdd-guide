[← /generate-design-spec](05-generate-design-spec.md) · [Explain Home](README.md) · [Next: /generate-tech-docs →](07-generate-tech-docs.md)

# 06 · `/generate-bdd` — Sinh kịch bản BDD (.feature)

> **Một câu.** Phân rã PRD thành các file `.feature` (Gherkin) mang `@trace.*`, theo platform (web/app/system), với fan-out per-UC và tổng hợp **System BDD** từ BDD FE/App.

---

## Vấn đề giải quyết

BDD là **anchor cứng** của traceability — mọi code/test link về scenario. Cấu trúc UC/SC quyết ở đây tác động mọi file code sau. Command phân rã PRD → UC → SC bằng Gherkin, gắn trace, và (cho system) tổng hợp contract từ BDD FE/App.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Design, sau PRD approved (+ design-spec nếu FE/App).
- **Gate vào:** cảnh báo mềm nếu PRD chưa `approved` (áp mọi mode).

---

## Input / Output

**Input:** PRD approved + design-spec (FE/App) + `feature.template` + BDD FE/App có sẵn (khi tổng hợp system).

**Output:** `.feature` với header `@trace.*` đầy đủ:
- Spec repo: `bdd/{web|app|system}/{TICKET-ID}-UC{N}-{slug}.feature`
- Umbrella: `bdd/{TICKET-ID}-UC{N}-{slug}.feature`

---

## Các bước xử lý (chi tiết)

1. **Phát hiện Repo Mode** — spec repo (không có `services`) → Platform Selection; umbrella (có `services`) → Service Detection (routing đã resolve ở context-loader; `unresolved` → DỪNG).
2. **Platform Selection** (spec mode) — hỏi web / app / **system**; set từ vựng theo platform (clicks/taps/receives).
3. **System BDD Synthesis** (nếu platform = system) — 5 step:
   - **S0 Brownfield** — `API Source: existing` → skip scan, dùng contract PRD.
   - **S1 Scan** BDD web/app có sẵn → phân loại multi/web-only/app-only/backend-only.
   - **S2 Trích contract** kỳ vọng mỗi platform (triggers, response data, error, business rules).
   - **S3 Cross-Platform Conflict Check** (multi) — so contract giữa platform; conflict (shape/error/rule/field) → 🛑 **CHECKPOINT bắt buộc** với 4 resolution: A union · B platform-hint · C endpoint riêng · D custom. Ghi `@system.resolution`.
   - **S4 Generate** scenario dùng từ vựng business-event (không UI).
4. **Design Spec Gate & Load** (FE/App) — nạp design-spec để lái coverage UI; cảnh báo mềm nếu chưa approved/lỗi thời.
5. **Orchestration Check** — PRD nhiều UC → **fan-out mỗi UC một sub-agent** (`_agent_mode`), sinh song song, gom kết quả.
6. **UC Decomposition** — 🛑 trình danh sách UC/SC để PO chốt.
7. **Generate** — mỗi `.feature` với header @trace (id, platform, service, module, status=draft, prd_version, bdd_version, business_rules, dataset) + scenario theo **R1–R10** + Compliance **C.1–C.5** + **NHÓM grouping** (bắt buộc khi ≥3 SC) + Coverage Matrix.

---

## Checkpoint & Gate

- 🛑 **UC Decomposition** — điểm dừng chính (cấu trúc UC sai kéo cả pipeline).
- 🛑 **Cross-platform conflict** (system multi) — bắt buộc PO resolve.
- Sinh với `@trace.status: draft` → cần `/review-context` sạch critical → đặt `approved`.

---

## Gen lại: hai thứ dễ mất nếu làm sai

**1 · Bump `@trace.sc_version` cho SC có thân đổi.** So bản mới với bản trên disk theo **4 thành phần**: tên `Scenario:` · chuỗi step · data table · `# Side-effects:`.

| Kết quả so | Hành động |
|---|---|
| khác ở bất kỳ thành phần nào | `+0.1` |
| giống hoàn toàn | **giữ nguyên** — bump vô cớ tạo `DRIFT` giả |
| SC mới | `1.0` |

Thay đổi **ngoài** 4 thành phần đó (`@trace.business_rules`, tag `@happy`/`@edge`, comment) **không** bump — chúng không đổi hành vi mà code phải implement. Report in danh sách SC được bump + cảnh báo chúng sẽ hiện `DRIFT`.

**2 · SC biến mất khỏi `.feature` — đừng xoá row `.tsv` nếu đã có code.**

| `implemented_by` | Hành động |
|---|---|
| `—` (chưa có code) | **xoá row** — không có gì mồ côi |
| có giá trị (**đã có code**) | **GIỮ row**, `status = ORPHANED`, giữ nguyên `implemented_by`/`test_*` |

Xoá row của một SC đã có code sẽ làm method đó **vô hình**: không `UNTRACKED`, không `GAP`, không `DRIFT`, không xuất hiện ở report nào — mà vẫn nằm trong code và vẫn được caller gọi. Coverage còn *đẹp hơn* thực tế vì mẫu số nhỏ đi. `ORPHANED` **không tự hết**: người phải chọn xoá code+test, hay đưa scenario trở lại.

---

## Cơ chế đặc biệt

- **System BDD là tổng hợp**, không viết tay — suy từ BDD web+app, giải conflict contract trước khi có tech-docs.
- **Fan-out per-UC** giữ context mỗi sub-agent nhỏ, quét sâu.
- **Từ vựng theo platform** áp âm thầm; không trộn web/mobile trong một file.
- **R1–R10 + C.1–C.5** enforce nghiêm, làm input cho gate B3/B4 của `/review-context`.

---

## 👓 Góc nhìn tối ưu

- **System BDD synthesis là phần phức tạp nhất** — phụ thuộc BDD web/app đã tồn tại & chất lượng. Nếu FE/App BDD lệch, conflict check gánh nặng; nếu thiếu → backend-only suy từ PRD.
- **Conflict resolution ghi vào `@system.resolution`** nhưng quyết định contract thực chốt ở tech-docs (T7). Có thể trùng vai — đáng xem ranh giới BDD-system vs tech-design.
- **Fan-out per-UC** tốn agent với PRD lớn (giống refine-prd) — chi phí ↔ độ đầy đủ.
- **Thứ tự platform quan trọng** — system nên sinh *sau* web/app để có contract tổng hợp; pipeline không ép thứ tự này cứng.

---

## Kết nối

**Trước:** [`/generate-design-spec`](05-generate-design-spec.md) (FE/App) / [`/review-context`](04-review-context.md) (PRD) · **Sau:** [`/review-context {feature}`](04-review-context.md) → [`/generate-tech-docs`](07-generate-tech-docs.md).
