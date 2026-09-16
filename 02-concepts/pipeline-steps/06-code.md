[← Tech-Docs](05-tech-docs.md) · [Pipeline Steps](README.md) · [Next: Dev self-test →](07-dev-selftest.md)

# Bước 6 · Code — Sinh mã nguồn (Code Generation)

> **Tóm tắt.** Sinh code từ `.feature approved` + tech-design, gắn `@trace` ở boundary, verify build, cập nhật trace state — với checkpoint "AI đã hiểu đúng chưa" trước khi chạy.
> **Commands:** `/generate-code` · `/review-code` · `/fix-bug`

| | |
|---|---|
| **Giai đoạn** | Implementation |
| **Owner** | 👤 Developer |
| **Đầu vào** | `.feature approved` + tech-design + CLAUDE.md |
| **Đầu ra** | Code có `@trace` + trace row trong `.tsv` |
| **HITL** | 🟡 Mỏng — 🛑 comprehension checkpoint |

---

## Mục đích (Purpose)

Code là **hệ quả của spec, không phải nguồn**. Bước này biến scenario thành code sao cho:

- Mỗi boundary mang **`@trace`** → đọc code biết ngay phục vụ scenario nào.
- **Drift per-UC** → không đập code refactor mù; chỉ chạm phần lệch.
- **Build verify** sẵn (≤3 retry) → không nhận code không build được.
- Không sinh code cho file **không** có `.feature` backing (trừ `/fix-bug`, `/debug`).

---

## Input (Đầu vào)

- **`.feature approved`** (hoặc UC-ID) — target.
- **Tech-design** `approved` (§4 làm nguồn contract cho shape DTO/endpoint/error).
- **Hợp đồng test-id** — `@trace.testid_attr` ở header (**tên** thuộc tính) + **§4.5.6 Test Selectors** (**giá trị** test-id). Do [`/map-testids`](05-tech-docs.md) chốt ở bước 5. Code **đọc** hợp đồng này, **không** tự đặt test-id.
- `CLAUDE.md` §2 (thứ tự layer, package strategy) + §3 (coding standards) + §5 (error handling) — **service overlay thắng**.
- `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` — để so drift.

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| File code | Theo thứ tự layer của stack, **block 5 tag** ở boundary (dưới) |
| `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` | Trace row cập nhật: `status`, `implemented_by`, `gen_ver`, `fe_phase`… |
| `_seams.tsv` | Sổ nợ seam/stub — chỗ chưa implement, để không đẻ mồ côi |

### Block 5 tag trên mỗi entry-point

```java
// @trace.implements=AUTH-UC1-SC3
// @trace.prd_version=1.2
// @trace.bdd_version=1.4
// @trace.tech_doc_revision=3
// @trace.source=specs/auth/login/bdd/system/AUTH-UC1-login.feature
public TokenDto login(...) { }
```

3 tag version không phải trang trí — chúng là nguồn của `PRD_DRIFT` / `BDD_DRIFT` / `TECHDOC_DRIFT` ở [`/validate-traces`](09-validate-traces.md). Thiếu một tag = drift detection **mù ở file đó**, im lặng.

> **File phủ nhiều UC → lặp CẢ BLOCK theo từng method.** Không gộp về một header file, không trỏ `@trace.source` vào **thư mục**.
>
> Vì sao: 3 tag version là scalar **theo từng UC** — gộp lại thì không diễn đạt được "UC1 ở bdd v1.4, UC3 ở v2.1" → drift báo oan hoặc mù. Và các lệnh tra tag bằng **khớp chuỗi chính xác** (`/dev-gen-test`, `/review-code`, `/validate-traces` đều tìm `@trace.implements={UC-ID}`), nên tag trỏ thư mục ra **0 kết quả** → UC rơi về `UNTRACKED` dù code đã có.
>
> Quy tắc EXTEND vốn đã yêu cầu giữ **nguyên si** mọi `@trace.implements` cũ *kể cả của UC khác* — tức thiết kế vốn là **tích luỹ nhiều block**.

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Dev** | Xác nhận comprehension checkpoint; xem report; gọi `/review-code` nếu cần |
| 🤖 **AI** | Phát hiện drift, sinh code theo layer, tag trace, build verify, ghi `.tsv` |
| 👤 **Lead** | `/review-code` (read-only) khi cần soát kỹ |

---

## Câu hỏi cần trả lời (Questions this step answers)

- So với lần sinh trước: SC nào **mới**, **drifted**, hay **synced** (bỏ qua)?
- Code phải theo **layer/package/convention** nào của stack này?
- Contract (DTO/endpoint/error) lấy từ đâu — tech-design §4 hay System BDD?
- Build có **pass** không?

---

## Framework xử lý thế nào (Mechanics)

1. **Drift detection** — so `bdd_version`/spec với `.tsv`: phân loại **new (UNTRACKED)** / **DRIFT** / **OK (synced-skip)**.
2. 🛑 **Comprehension checkpoint** (mềm): *"{X} new, {Y} drifted, {Z} synced-skip — Proceed?"* → tránh AI hiểu sai mà vẫn chạy.
3. **Scope Lock** — chỉ implement UC target; code của UC khác trong file dùng chung là **bất khả xâm phạm** (đọc `@trace.implements` để bảo toàn, không xoá).
4. **Generate** theo **thứ tự layer** (vd Controller → Facade → Service → Repository) từ CLAUDE.md §2; tag `@trace` chỉ ở **boundary** (controller/handler), shared code dò qua import chain.
5. **Gắn test-id theo hợp đồng** — đọc `@trace.testid_attr` ở header tech-doc để biết gắn **thuộc tính nào**, đọc §4.5.6 để biết gắn **giá trị nào** lên element nào:

   | Tình huống | Hành vi |
   |---|---|
   | §4.5.6 **rỗng** | ⚠️ **Cảnh báo mạnh rồi hỏi Y/N** — không tự bịa test-id. Đường đúng là dừng lại chạy `/map-testids`; tiếp tục là quyết định của dev, có ghi nhận |
   | Header **không có** `@trace.testid_attr` | Cảnh báo mềm nêu rõ rủi ro → fallback theo platform. Không im lặng hardcode |
   | `.feature` cũng khai attr và **lệch** header | In **cả hai** giá trị, để người quyết |

6. **Build verify** — chạy `{conventions.build_command}`, ≤3 retry.
7. **Ghi trace row** vào `.tsv` (trong spec repo nếu umbrella — thao tác ghi liên-repo).

**Mode theo phase × platform:**
| Mode | Ý nghĩa |
|-------|---------|
| `--phase=ui` | FE Phase 1 — sinh UI + layer **mock API** từ System BDD contract |
| `--phase=integration` | FE Phase 2 — **thay** mock bằng **BE contract thật** (đọc tech-design từ spec repo); UI đã có |
| *(default, `web`/`app`)* | **`fe_full`** — sinh UI **+** wire API thật một lần, bỏ lớp mock |
| *(default, `system`)* | Backend — sinh từ tech-design §4 |

> Nhánh wire API thật (`integration` / `fe_full`) đi qua **DS4** (kiểm §4.5.4 đủ, vét nguồn rồi gộp-hỏi) và **DS5** (phát hiện component/service FE đang chạy → hỏi reuse/new, không dựng song song). Bảng hành vi đầy đủ từng bước → [Explain · generate-code](../../explain/09-generate-code.md#hành-vi-theo-mode-phase--platform).

**`/review-code`** — read-only, chỉ báo findings (không tự sửa: "AI tự fix tự review" = lặp lỗi). **5 lăng kính:**

| # | Lăng kính | Điểm đáng chú ý |
|---|---|---|
| 1 | Traceability | đủ **4 tag version** kèm mỗi `@trace.implements` · `@trace.source` trỏ file **có thật, đúng platform** · file đa-UC có block riêng theo method · **tag mồ côi** (`TRACE_ORPHAN`) = critical |
| 2 | Layer Architecture | đúng layer, chiều phụ thuộc, không bypass |
| 3 | Coding Standards | naming, wrapper, exception, transaction, không magic number |
| 4 | Spec Compliance | mỗi scenario có implementation, không endpoint không-spec |
| 5 | **Seam & Stub** | sổ `_seams.tsv` 0 dòng `READY` · stub không còn là binding khi hàng thật đã có · **method thật mồ côi** do đẻ song song |

> Critical ở lăng kính 5 → `NEEDS_FIX` **kể cả khi build xanh và test từng-UC xanh** — đó chính là lớp lỗi hai thứ đó không bắt được.

**`/fix-bug`** — sửa lỗi có root-cause + regression test, **và cập nhật sổ trace**; xem [Feedback Loop](10-feedback-loop.md).

---

## HITL / Gate

- 🛑 **Comprehension checkpoint** — điểm dừng chính: Dev xác nhận phân loại drift đúng trước khi sinh.
- Build phải **pass** trước khi commit.
- `/review-code` là gate read-only tuỳ chọn — không auto-apply fix.

---

## Ví dụ (Example)

```
/generate-code UC-02
  Drift: 1 new (SC-02.3), 1 drifted (SC-02.1 bdd_version 1.0→1.1), 2 synced-skip
  🛑 Proceed? (Y/N) → Y
  → sinh/patch code, @trace.implements ở PasswordResetController
  → build verify ✅ (attempt 1)
  → cập nhật .trace/auth/quen-mat-khau/UC-02-web.tsv
```

---

## Anti-pattern

- ❌ Sinh code cho file không có `.feature` backing (ngoài fix-bug/debug).
- ❌ Tái tạo file dùng chung "chỉ gồm scenario UC này" → xoá nhầm nghiệp vụ UC khác.
- ❌ Tag `@trace` mọi file → tag explosion; chỉ tag boundary.
- ❌ Lưu version trong code — version chỉ ở spec; code dùng `.tsv`.
- ❌ **Tự đặt test-id** khi §4.5.6 rỗng — QC đang viết script theo hợp đồng đó song song; code bịa một bộ id khác là làm vỡ script mà không ai báo.

---

## Bước tiếp theo (Next step)

Code build được → dev tự kiểm nhanh:

➡️ [Bước 7 · Dev self-test — `/dev-gen-test` · `/dev-run-test` · `/dev-smoke-test`](07-dev-selftest.md)
