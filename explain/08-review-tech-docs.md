[← /map-testids](11-map-testids.md) · [Explain Home](README.md) · [Next: /generate-code →](09-generate-code.md)

# 08 · `/review-tech-docs` — Review Technical Design (8 dimension + ký T7)

> **Một câu.** Review tech-design qua **8 dimension T1–T7 + T3b** (kiến trúc, entity, BDD trace, cross-PRD conflict, nội bộ, cấu trúc, và cổng ký liên team), sinh findings + `--resume`.

---

## Vấn đề giải quyết

Chốt sai contract kỹ thuật = rework tốn kém cho nhiều team. `/review-tech-docs` soát tech-design đa chiều **trước khi đốt budget code**, và đảm bảo FE/App/BE **đồng thuận contract** qua sign-off T7.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Design, sau `/generate-tech-docs`.
- **Đặc biệt:** read-only (chỉ báo findings); có **Resume Mode** áp finding accepted.

---

## Input / Output

**Input:** tech-design (một doc full-stack) + CLAUDE.md §2 + core-entities + mọi BDD của PRD (system/web/app).

**Output:** findings file + cập nhật `@trace.sign_off` khi đủ.

---

## Các bước xử lý (chi tiết)

Chạy 8 dimension (mỗi cái phân loại severity + auto-fixable):

| Dim | Tên | Soi gì | Auto-fix |
|-----|-----|--------|----------|
| **T1** | Architecture Alignment | Vi phạm CLAUDE.md §2 (controller gọi repo, logic trong controller, pattern cấm) — **luôn critical** | ❌ người quyết |
| **T2** | Entity Consistency | Đối chiếu core-entities (entity thiếu, tên field lệch, quan hệ khác) | một phần (field → canonical) |
| **T3** | BDD Traceability | 2 chiều design ↔ scenario, **theo đúng lane platform** (system/web/app SC không so chéo) | một phần |
| **T3b** | **BDD Freshness** | Doc dựng từ BDD **version nào**, BDD giờ ở version nào — so từng entry của map `@trace.bdd_versions` với `.feature` tương ứng | 2 ca (thêm/xoá entry) |
| **T4** | Cross-PRD Endpoint Conflict | grep endpoint/entity ở doc PRD khác, **load-on-hit**; va chạm shape/behavior → critical | ❌ |
| **T5** | Internal Consistency | Sequence vs mô tả, API spec vs code sketch, ref không định nghĩa | một phần |
| **T6** | Structural Completeness | Section chuẩn có mặt & không rỗng · **+ hợp đồng test-id** (dưới) | ✅ thêm skeleton — **trừ** hai ca test-id |
| **T7** | Cross-Team API Contract | **Cổng ký liên team** — chỉ khi doc có backend (system) + không phải `api_source: existing` | sign-off block auto-fix |

**T6 · hợp đồng test-id** *(mới)* — hai ca **Major** và **không tự sửa được**:

| Điều kiện | Severity | Auto-fix |
|---|---|---|
| **§4.5.6 Test Selectors rỗng** (doc có platform client) | **Major** | ❌ — phải chạy `/map-testids` |
| Header **thiếu** `@trace.testid_attr` | **Major** | ❌ — phải chạy `/map-testids` |

> **Vì sao KHÔNG auto-fix.** Tự điền một bảng test-id là **bịa hợp đồng** — mà hợp đồng này có hai bên tiêu thụ (FE gắn attribute, QC viết script). Thêm skeleton rỗng thì lần review sau nó "có mặt & không rỗng" và cổng tự tắt, trong khi bảng vẫn vô nghĩa.
>
> **Vì sao cổng nằm ở đây.** Đây là chỗ **cuối cùng còn chặn được trước khi có code**. Trôi qua đây thì `/generate-code` phải tự xoay xở với một bảng rỗng, và QC thì không có gì để bám.

**T3b chi tiết** — T3 kiểm *nội dung* khớp, T3b kiểm *độ tươi*:

| Điều kiện | Severity | Auto-fix |
|---|---|---|
| `.feature` **mới hơn** map | **Major** | ❌ cần người review lại §4/§4.5 rồi bump `@trace.revision` |
| Platform có `.feature` nhưng **vắng** trong map | Major | ✅ thêm entry sau khi xác nhận §4 đã phủ |
| Map có platform mà không còn `.feature` | Minor | ✅ xoá entry |

- **Major chứ không Minor:** `/generate-code` DS3 thấy doc `approved` + 0 blocker-GAP sẽ lấy shape §4 **nguyên văn** làm contract "đã chốt" → contract dựng từ BDD cũ lan **thẳng** vào code. Tệ hơn drift-về-code vì sai từ nguồn.
- **Chặn `approved` — nhưng MỀM** (CHECKPOINT `Y/N`), khác GATE §12 blocker-GAP chặn cứng: BDD hay bump vì lý do **không chạm contract** (sửa từ ngữ step, thêm side-effect assertion), người review là người biết.
- Khi `--resume` áp fix: **cấm chỉ sửa số trong map** cho ca "`.feature` mới hơn" — làm thế là dán nhãn "đã đồng bộ" lên contract chưa ai review. Platform còn finding `open` → **giữ số cũ** để cờ `TECHDOC_STALE_VS_BDD` của `/validate-traces` còn sáng.
- Tên key là `@trace.bdd_versions` (**số nhiều**, map theo platform) — cố ý khác `@trace.bdd_version` (scalar) của `.feature`, vì cùng một tên cho hai kiểu dữ liệu sẽ làm vỡ parser generic.

**T7 chi tiết:**
- Đọc block `@trace.sign_off` (be_team / fe_team / app_team / sa); vắng → thêm skeleton.
- Cross-check contract §4 vs web & app BDD → đảm bảo mọi team đồng thuận trước khi implement.

Sau phân tích → ghi findings; **Resume Mode** áp finding `accepted`/`modified`.

---

## Checkpoint & Gate

- 🔒 **T7 sign-off** — contract liên team chưa ký đủ (be/fe/app/sa) → chưa mở khoá code phía tiêu thụ.
- 🟠 **T6 test-id** — §4.5.6 rỗng hoặc thiếu `@trace.testid_attr` → Major, không tự sửa. Đường ra là chạy [`/map-testids`](11-map-testids.md), không phải bấm qua.
- 🟡 **T3b (chặn mềm)** — còn finding T3b Major `open` → CHECKPOINT `Y/N` trước khi đặt `approved`.
- Read-only — không tự sửa; findings qua Board → `--resume`.

---

## Cơ chế đặc biệt

- **T4 load-on-hit** — không nạp full doc PRD khác, chỉ grep path/entity rồi đọc đoạn khớp → rẻ.
- **SC scope theo platform** — `system UC1-SC1` ≠ `web UC1-SC1`; match trong đúng lane.
- **T7 skip khi brownfield** (`api_source: existing`) — contract đã do PO chốt, không có design mới để đồng thuận.
- **T1 luôn critical** — vi phạm kiến trúc chặn cứng.

---

## 👓 Góc nhìn tối ưu

- **8 dimension trong một lệnh** — nặng; T4 (cross-PRD) và T7 (sign-off) là hai phần đắt nhất. T4 dùng grep khéo để rẻ; T7 phụ thuộc con người ký.
- **T7 sign-off là quy trình đa người** — dễ nghẽn nếu một team chậm ký. Đáng có cơ chế nhắc/timeout.
- **Chồng lấn với conflict resolution ở generate-bdd (system)** — cả hai lo contract cross-platform. Ranh giới: BDD-system chốt *hành vi contract*, T7 chốt *shape API + đồng thuận team*.
- **Không dùng review-fanout** (khác `/review-context`/`/refine-prd`) — 8 dimension chạy tuần tự trong một session. Với doc lớn có thể lost-in-the-middle; cân nhắc fan-out.

---

## Kết nối

**Trước:** [`/generate-tech-docs`](07-generate-tech-docs.md) → [`/map-testids`](11-map-testids.md) · **Sau:** đủ ký T7 + §4.5.6 đã chốt → **hai nhánh song song**: [`/generate-code {feature}`](09-generate-code.md) (FE gắn attribute) ∥ [`/qc-design-test`](17-qc-design-test.md) (QC dựng test theo cùng bảng).
