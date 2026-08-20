[← /review-tech-docs](08-review-tech-docs.md) · [Explain Home](README.md) · [Next: /review-code →](10-review-code.md)

# 09 · `/generate-code` — Sinh mã nguồn từ BDD

> **Một câu.** Sinh code từ một `.feature` approved + tech-design, phát hiện **drift per-UC**, gắn `@trace` ở boundary, nối/lấp **seam & stub** (chống mồ côi), verify build, ghi trace state — với comprehension checkpoint.

---

## Vấn đề giải quyết

Code là **hệ quả của spec**. Command biến scenario thành code sao cho: mỗi boundary có trace, chỉ chạm phần drift (không đập refactor mù), không sinh code không có `.feature` backing, và build được trước khi commit.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Implementation, sau BDD + tech-docs.
- **Gate vào:** cảnh báo mềm DS1 (BDD chưa approved), DS2 (design-spec), DS3 (tech-doc contract BE), **DS4** (client contract §4.5.4) + **DS5** (FE reuse gate) — DS4/DS5 chỉ khi wire API thật (`integration`/`fe_full`). Xem [Hành vi theo mode](#hành-vi-theo-mode-phase--platform).

---

## Input / Output

**Input:** `.feature` (hoặc UC-ID) + tech-design §4 + CLAUDE.md §2/§3/§5 + `.trace/…/{UC-ID}-{platform}.tsv` + `_seams.tsv`.

**Output:** file code (tag `@trace` boundary) + trace row `.tsv` + cập nhật `_seams.tsv`.

---

## Các bước xử lý (chi tiết)

1. **Guard BDD & Design Spec** — DS1/DS2/DS3 cảnh báo mềm nếu nguồn chưa chốt.
2. **Phase Detection** — `--phase=ui` (FE Phase 1: UI + mock API; **mock source hybrid** = ưu tiên tech-doc §4 contract, fallback System BDD) · `--phase=integration` (thay mock bằng API thật) · **`fe_full`** (default trên `web`/`app`: UI + real API một lần, bỏ mock) · default BE (`system`, full backend). Chi tiết: [Hành vi theo mode](#hành-vi-theo-mode-phase--platform).
3. **Figma Dev Mode MCP Check** (sinh UI) — đọc frame Figma độ trung thực cao cho codegen.
4. **Read Trace State** — so `bdd_version`/spec với `.tsv` → phân loại **new / drifted / synced-skip**.
5. **Package Placement** — đặt code đúng `code_base_package` + strategy (by-layer/by-feature), chống phân mảnh.
6. **Seam & Stub Ledger** (`_seams.tsv`) — trước khi tạo chỗ giả lập, ghi sổ:
   - `seam` = gọi ra port cross-UC (UC khác sở hữu) · `stub` = method trắng nội-feature (logic thuộc BDD khác của chính feature).
   - Trạng thái: `PENDING` (chưa có hàng thật) · `READY` 🔴 (hàng thật đã có nhưng chưa nối) · `RESOLVED`.
   - Vá 2 lỗi kinh điển: **hàm trắng mồ côi** (no-op khi ghép luồng) và **hàm thật mồ côi** (đẻ hàm mới thay vì lấp stub cũ).
7. **File Scan** — quét file hiện có để quyết CREATE/EXTEND/FILL/SKIP.
8. **CHECKPOINT — Code Generation Plan** — 🛑 trình: scenarios (X new, Y drifted, Z synced-skip) · CREATE (mới) · **EXTEND (ADD-ONLY, cấm full Write)** · **FILL** (lấp stub tại chỗ, không đẻ method song song) · SKIP + danh sách member GIỮ NGUYÊN (UC khác). **Scope Lock**: chỉ đọc/implement UC target.
9. **Branch** `feature/{TICKET}-{slug}`.
10. **Generate** theo **thứ tự layer từ CLAUDE.md §2**; tag `@trace.implements/source` ở boundary.
11. **Mock/Integration** — layer mock (`--phase=ui`) · thay adapter thật (`--phase=integration`) · wire real thẳng (`fe_full`). Nhánh wire API thật đi qua **DS4** (kiểm §4.5.4 đủ + **SRC-CHAIN** vét nguồn, gộp-hỏi thay vì hỏi live).
12. **Self-Review (3 vòng)** → **Build Verify** (`{build_command}`, ≤3 retry) → **Write Trace State** → **Refresh Panel Mirror** (Living Docs umbrella) → **Commit** (sau khi duyệt).

---

## Hành vi theo mode (Phase × Platform)

`generate-code` rẽ nhánh theo `--phase` và platform của `.feature`. Bốn mode:

- **`--phase=ui`** — FE Phase 1: chỉ UI + mock, không chạm API thật (prototype nhanh).
- **`--phase=integration`** — FE Phase 2: **không** sinh lại UI, chỉ **thay** mock bằng real adapter.
- **`fe_full`** — default (không flag) trên feature `web`/`app`: UI **+** wire API thật trong một lần, bỏ lớp mock trung gian.
- **default BE** — không flag trên `system`: full backend.

| Bước / Gate | `--phase=ui` | `--phase=integration` | `fe_full` (default web/app) | default BE (`system`) |
|---|:--:|:--:|:--:|:--:|
| **DS1** — BDD `approved` guard | ✅ | ✅ | ✅ | ✅ |
| **DS2** — design-spec guard (FE) | ✅ | ✅ | ✅ | — |
| **DS3** — BE contract gate (§4) | — | — | — | ✅ |
| **DS4** — client contract gate (§4.5.4) | ⏭️ cố ý | ✅ | ✅ | — |
| **DS5** — FE component/service reuse gate | ⏭️ | ✅ | ✅ | — |
| **Sinh UI** (Figma MCP, component, state) | ✅ | ⏭️ giữ UI cũ | ✅ | — |
| **Test Selectors** (emit test-id) | ✅ | ⏭️ | ✅ | — |
| **Mock API Layer** | ✅ | ⏭️ | ⏭️ | — |
| **Integration Phase** (real API adapter) | ⏭️ | ✅ | ✅ | — |
| **Wire-up API** | dùng **mock** | **lật** mock→real | wire **thẳng** real | — (BE là API) |
| **SRC-CHAIN** (vét nguồn trước khi hỏi) | ✅¹ | ✅ | ✅ | ✅ |
| **Nguồn shape/contract** | §4 nếu có, else System BDD | §4.5.4 → §4.1–4.3 | §4.5.4 → §4.1–4.3 (+ design-spec) | §4 (DS3, nguyên văn) |
| **Output adapter** | `{UC}MockApiAdapter` | `{UC}ApiAdapter` (giữ mock) | `{UC}ApiAdapter` (không mock) | — |
| **Trace `fe_phase`** | `ui` | `integrated` | `integrated` | `—` |

<sub>✅ chạy · ⏭️ bỏ qua (có chủ đích) · — không áp dụng. ¹SRC-CHAIN là quy tắc chung, active mọi mode; chỉ được **gom-hỏi qua DS4** ở integration/fe_full.</sub>

**Bất đối xứng đáng nhớ:**
- **DS4** có ở mọi nhánh wire API thật (integration + fe_full), **trừ** `ui` (cố ý) và BE (BE dùng DS3).
- **Mock API Layer** là thứ **duy nhất** riêng `--phase=ui`.
- **Sinh UI** có ở `ui` và `fe_full`, **không** ở `integration` (giả định UI đã có từ pha ui).
- **`fe_full` = `ui` + `integration`** gộp một lần (bỏ mock), nhưng **vẫn giữ safety gate DS4** — không phải đường tắt kém an toàn.

---

## Checkpoint & Gate

- 🛑 **Comprehension checkpoint** (Code Generation Plan) — điểm dừng chính; Dev xác nhận drift + scope đúng.
- Build phải pass trước commit.

---

## Cơ chế đặc biệt

- **Seam & Stub Ledger** — cơ chế tinh vi nhất: giải quyết vấn đề "sinh code từng BDD rời tạo hàm mồ côi". `READY` là cờ 🔴 để lần sau nối hàng thật vào.
- **Scope Lock + ADD-ONLY + FILL** — bảo toàn code UC khác trong file dùng chung; EXTEND chỉ Edit thêm, FILL lấp stub tại chỗ.
- **Mock source hybrid** — mock shape ưu tiên contract thật để đỡ rework lúc integration.
- **SRC-CHAIN + DS4** — "vét cạn nguồn trước khi hỏi": khi §4.5.4 thiếu, quét lần lượt PRD/BDD/design-spec/core-entities/mock adapter (skip-if-answered) rồi **gộp mọi gap vào một checkpoint** thay vì hỏi live từng câu. DS4 là cổng đối xứng với DS3 của BE cho nhánh FE wire API thật (integration/fe_full).
- **DS5 — FE reuse gate** ("discover & reuse trước khi dựng mới"): ở nhánh wire API thật, nếu **không có mock adapter** (chạy `--phase=integration` một mình trên app brownfield) nhưng **quét thấy component/service FE đang chạy** → **hỏi reuse/new** thay vì im lặng dựng adapter song song, mồ côi. Chọn *reuse* → EXTEND/wire vào code có sẵn.
- **3 vòng self-review + build ≤3 retry** — hai lớp tự kiểm trước khi giao.

---

## 👓 Góc nhìn tối ưu

- **Command nặng nhất về logic thực thi** — 12 bước, nhiều nhánh (phase, drift, seam, scope lock). Đây là nơi bug/tối ưu tác động trực tiếp code chất lượng.
- **Seam ledger phụ thuộc kỷ luật ghi sổ** — nếu một lần gen quên ghi `_seams.tsv`, cơ chế chống mồ côi thủng. Đáng có validate.
- **`--phase=ui` với mock từ System BDD (không contract)** → shape có thể lệch → rework ở integration. Chi phí này giảm nếu BE publish tech-doc §4 trước.
- **Build verify ≤3 retry** — nếu 3 lần fail thì sao? Đáng làm rõ hành vi khi vượt retry.
- **Refresh Panel Mirror** chỉ local umbrella — trạng thái Living Docs phụ thuộc `/sync` để lan.

---

## Kết nối

**Trước:** [`/review-tech-docs`](08-review-tech-docs.md) · **Sau:** lần đầu → [`/review-code`](10-review-code.md); gen lại → [`/dev-gen-test`](12-dev-gen-test.md).
