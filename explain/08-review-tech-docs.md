[← /map-testids](11-map-testids.md) · [Explain Home](README.md) · [Next: /generate-code →](09-generate-code.md)

# 08 · `/review-tech-docs` — Review Technical Design (10 dimension + ký T7)

> **Một câu.** Review tech-design qua **10 dimension T1–T9 + T3b** (kiến trúc, entity, BDD trace, cross-PRD conflict, nội bộ, cấu trúc, cổng ký liên team, đối chiếu PRD-BR, trung thực GAP), sinh findings + `--resume`.

---

## Vấn đề giải quyết

Chốt sai contract kỹ thuật = rework tốn kém cho nhiều team. `/review-tech-docs` soát tech-design đa chiều **trước khi đốt budget code**, và đảm bảo FE/App/BE **đồng thuận contract** qua sign-off T7.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Design, sau `/generate-tech-docs`.
- **Đặc biệt:** hai chế độ — *phân tích* chỉ đọc, ghi file findings; *`--resume`* **ghi vào thân tech-doc, header (`@trace.bdd_versions`, `@trace.sign_off`, `@trace.status`, `@trace.revision`) và sổ TSV của mọi UC trong `@trace.ucs`**.

---

## Input / Output

**Input:** tech-design (một doc full-stack) + CLAUDE.md §2 + core-entities + mọi BDD của PRD (system/web/app) + bảng Business Rule của PRD (cho T8).

**Output:** `{refinement_dir}/{TICKET-ID}-tech-review-findings.yaml`; với `--resume`: tech-doc đã sửa + `@trace.revision` +1 + `tech_doc_revision` mới trong TSV của mọi UC doc phủ.

---

## Các bước xử lý (chi tiết)

Chạy 10 dimension (mỗi cái phân loại severity + auto-fixable):

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
| **T8** | Reconciliation & Completeness | Đối chiếu PRD-BR, core-entities, seam UC anh em: enum không producer, field không writer, BR bị bỏ sót, leak boundary, thiếu dedup/ordering/ack path | chỉ invariant (minor) |
| **T9** | Gap Honesty & Constants | Bịa không nguồn (lẽ ra `[GAP]`), constant inline, happy-only, marker GAP mồ côi với §12, **§12 còn 🔴 blocker `open` → critical** | constant/citation/đồng bộ register |

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

## Duyệt theo phần — `/review-tech-docs {doc} web` / `app`

Khi FE tự viết phần client, lead FE duyệt riêng phần của platform mình, không phải chờ duyệt cả doc.

| | Duyệt cả doc (mặc định) | Duyệt theo phần (`web` / `app`) |
|---|---|---|
| Phạm vi | Toàn doc | §4.5 của platform + §5.B/§5.C + row §8–§10, §12 của platform đó |
| Dimension | T1–T9 + T3b | T3, T3b, T5, T6, T8, T9 trong phần; T1/T4/T7 chỉ khi duyệt phần system |
| Kiểm thêm | — | Chỗ nối §4.5.4 ↔ §4.1 (gọi API không tồn tại / sai field) → **Critical** |
| File findings | `{TICKET-ID}-tech-review-findings.yaml` | `{TICKET-ID}-{platform}-tech-review-findings.yaml` |
| `--resume` | Đặt `@trace.status`, bump revision | **Không** đụng `@trace.status`; duyệt xong thì row §10 của platform `🔵` → `✅`; chỉ bump revision khi có fix, kèm row Changelog `UC{n}[{platform}]: …`; chỉ cập nhật sổ TSV của platform đó |

---

## Checkpoint & Gate

- 🔒 **T7 sign-off** — contract liên team chưa ký đủ (be/fe/app/sa) → chưa mở khoá code phía tiêu thụ.
- 🟠 **T6 test-id** — §4.5.6 rỗng hoặc thiếu `@trace.testid_attr` → Major, không tự sửa. Đường ra là chạy [`/map-testids`](11-map-testids.md), không phải bấm qua.
- 🟡 **T3b (chặn mềm)** — còn finding T3b Major `open` → CHECKPOINT `Y/N` trước khi đặt `approved`.
- 🔒 **Set `approved` chỉ khi** sign-off đủ **và** §12 GAP Register còn 0 🔴 blocker `open`; thiếu một trong hai → `in-review` (chặn `/generate-code`).
- Nhãn khai **chặn CỨNG** — chỉ nổ khi findings đã có quyết định reviewer, hoặc chạy `--full` / `--resume`; `--yes` **không** bỏ qua được. Lần phân tích đầu → đi thẳng.
- Finding cần người qua Board → `--resume`.

---

## Cơ chế đặc biệt

- **T4 load-on-hit** — không nạp full doc PRD khác, chỉ grep path/entity rồi đọc đoạn khớp → rẻ.
- **SC scope theo platform** — `system UC1-SC1` ≠ `web UC1-SC1`; match trong đúng lane.
- **T7 skip khi brownfield** (`api_source: existing`) — contract đã do PO chốt, không có design mới để đồng thuận.
- **T1 luôn critical** — vi phạm kiến trúc chặn cứng.

---

## 👓 Góc nhìn tối ưu

- **10 dimension trong một lệnh** — nặng; T4 (cross-PRD) và T7 (sign-off) là hai phần đắt nhất. T4 dùng grep khéo để rẻ; T7 phụ thuộc con người ký.
- **T7 sign-off là quy trình đa người** — dễ nghẽn nếu một team chậm ký. Đáng có cơ chế nhắc/timeout.
- **Chồng lấn với conflict resolution ở generate-bdd (system)** — cả hai lo contract cross-platform. Ranh giới: BDD-system chốt *hành vi contract*, T7 chốt *shape API + đồng thuận team*.
- **Không dùng review-fanout** (khác `/review-context`/`/refine-prd`) — 10 dimension chạy tuần tự trong một session. Với doc lớn có thể lost-in-the-middle; cân nhắc fan-out.

---

## Kết nối

**Trước:** [`/generate-tech-docs`](07-generate-tech-docs.md) → [`/map-testids`](11-map-testids.md) · **Sau:** đủ ký T7 + §4.5.6 đã chốt → **hai nhánh song song**: [`/generate-code {feature}`](09-generate-code.md) (FE gắn attribute) ∥ [`/qc-analyze {TICKET-ID} {platform}`](15-qc-analyze.md) (cửa vào làn QC — trạm 1→3 chạy được ngay, chưa cần code; đọc cùng §4.5.6). NEEDS_FIX → sửa doc.
