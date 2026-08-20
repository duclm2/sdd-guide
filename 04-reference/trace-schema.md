[← Reference](./) · [Concepts › Traceability](../02-concepts/traceability.md)

# Reference · Trace Schema

> Field metadata `@trace.*` và cột file `.tsv`. Giải thích khái niệm → [Traceability](../02-concepts/traceability.md).

> ⚙️ **Bản máy đọc: `bin/trace-schema.json`.** File đó là nguồn-sự-thật mà `bin/self-check.js`
> đối chiếu với `commands/*.tmpl` + `steps/*.md` mỗi lần `npm run build` — build **fail** nếu
> lệnh lệch schema. Đổi contract thì sửa file JSON **trước**, rồi sửa lệnh, rồi cập nhật trang này.

---

## Trace tags theo artifact

### BDD — header file (`.feature`)

| Tag | Ý nghĩa | Bắt buộc |
|-----|---------|:--------:|
| `@trace.id` | UC-ID — `{TICKET-ID}-UC{N}` (vd `SEG01-UC1`) | ✅ |
| `@trace.platform` | `web` / `app` / `system` — **mọi mode, kể cả umbrella** | ✅ |
| `@trace.domain` | Domain nghiệp vụ | ✅ |
| `@trace.prd` | TICKET-ID của PRD nguồn | ✅ |
| `@trace.prd_version` | Version PRD lúc sinh BDD | ✅ |
| `@trace.bdd_version` | Version **cả file** — tăng 0.1 mỗi lần gen lại / `--fix` | ✅ |
| `@trace.status` | `draft` / `in-review` / `approved` — cổng duyệt BDD | ✅ |
| `@trace.title` · `@trace.revision` · `@trace.author` · `@trace.created_at` · `@trace.business_rules` · `@trace.dataset` | thông tin | ⚪ |
| `@trace.service` · `@trace.module` | chỉ **umbrella mode**; vắng ở spec repo mode là đúng | ⚪ có điều kiện |
| `@trace.api_source` | `existing` — chỉ khi `platform = system` và PRD brownfield | ⚪ có điều kiện |

> **`@trace.platform` là field load-bearing nhất.** Thiếu nó: `/generate-code` không quyết được BE/FE (và **cấm** fallback sang `platform_type`), không định vị được sổ trace `{UC-ID}-{platform}.tsv`, không tìm được design-spec. Nó phải **khớp** segment `{platform}` của đường dẫn file.

### BDD — mỗi scenario

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.scenario` | SC-ID — `{UC-ID}-SC{N}` (vd `SEG01-UC1-SC3`) |
| **`@trace.sc_version`** | Version **của riêng scenario này**. Tăng 0.1 khi **thân SC** đổi (tên · step · data table · side-effect). |
| `@trace.business_rules` | BR liên quan — `{TICKET-ID}-UC{N}-BR{m}` |

> **`sc_version` vs `bdd_version`:** `sc_version` là tín hiệu **duy nhất** cho `DRIFT` (`spec_ver != gen_ver`). `bdd_version` bắt thay đổi cấp file mà `sc_version` không thấy (Background, dataset, Business Definition, Coverage Matrix). Quên bump `sc_version` = code sinh từ scenario cũ **vĩnh viễn** hiện `OK`.

### Code — entry-point (boundary only)

```java
// @trace.implements=SEG01-UC1-SC3
// @trace.prd_version=1.2
// @trace.bdd_version=1.4
// @trace.tech_doc_revision=3
// @trace.design_spec_version=1.5      ← CHỈ FE/App; bỏ hẳn dòng này với system/backend
// @trace.source=specs/segment/scoring/bdd/system/SEG01-UC1-scoring.feature
public ScoreDto calculate(...) { }
```

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.implements` | SC mà method này hiện thực |
| `@trace.prd_version` · `@trace.bdd_version` · `@trace.tech_doc_revision` | version của từng artifact upstream **tại thời điểm codegen** — nguồn của `PRD_DRIFT` / `BDD_DRIFT` / `TECHDOC_DRIFT` |
| `@trace.design_spec_version` | *(chỉ FE/App)* version design-spec lúc codegen — nguồn của `DESIGNSPEC_DRIFT`. Design-spec điều khiển cả BDD FE/App lẫn code FE, nhưng từng là artifact upstream **duy nhất** không có cột, không có tag, không có cờ |
| `@trace.source` | `.feature` nguồn — **phải gồm segment `{platform}`** |

> **5 tag bắt buộc — 6 với FE/App.** `/review-code` lăng kính Traceability gác: thiếu tag nào = `major`; riêng `@trace.design_spec_version` bất đối xứng — thiếu ở FE = `major`, **có** ở backend = `minor` (tag thừa, không có design-spec để so).

> **File phủ nhiều UC → lặp CẢ BLOCK theo từng method.** Không gộp về một header file, không trỏ thư mục. 4 tag version là scalar **theo từng UC**; gộp lại thì không diễn đạt được "UC1 ở bdd v1.4, UC3 ở v2.1" → drift báo oan hoặc mù. Và các lệnh tra tag bằng **khớp chuỗi chính xác**, nên `@trace.source` trỏ thư mục sẽ ra 0 kết quả → UC rơi về `UNTRACKED` dù code đã có.

### Code — chỗ chưa implement (sổ `_seams.tsv`)

| Tag | Đặt ở đâu | Ý nghĩa |
|-----|---|---------|
| `@trace.stub` · `@trace.stub_owner` · `@trace.stub_for` | method trắng | logic thuộc BDD **khác của cùng feature** — ai để trắng / ai sẽ lấp / trách nhiệm gì |
| `@trace.seam_pending` · `@trace.seam_port` | class stub | port **cross-UC** do UC khác sở hữu, hàng thật chưa có |

### Test

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.verifies` | SC mà test kiểm chứng — `{UC-ID}-SC{N}` |

### Bug fix

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.fixes` | `{BUG-ID}` nếu fix từ bug report đã file, else TICKET_ID |
| `@trace.root_cause` | Nguyên nhân gốc |
| `@trace.regression` | Test regression thêm vào |

### Tech-doc gộp (header, cấp PRD)

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.id` · `@trace.domain` · `@trace.prd` | định danh |
| `@trace.ucs` | **danh sách** UC mà doc này phủ |
| `@trace.platforms` | platform có mặt |
| `@trace.bdd_versions` | **map theo platform** (`system=1.5, web=1.9`). Tên **số nhiều** để phân biệt với `@trace.bdd_version` (scalar) của `.feature` — cùng tên cho hai kiểu dữ liệu sẽ làm vỡ parser generic. |
| `@trace.revision` | integer, bump mỗi lần sửa — nguồn của `TECHDOC_DRIFT` |
| `@trace.status` | `draft` / `in-review` / `approved` — cổng của `/generate-code` DS3 |
| `@trace.api_source` | `existing` → chế độ reverse-document, bỏ cổng T7 |

---

## Boundary-only tagging

| ✅ Tag | ❌ Không tag |
|--------|-------------|
| Controller · Handler · Middleware · Consumer · Steps file | Entity · Repository · DTO · Interface · Base class |

Shared code dò qua **import chain** từ boundary → tránh tag explosion.

---

## Trace state — `.tsv`

Đường dẫn: `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` — **một sổ cho mỗi UC × platform** (`sc_id` chỉ độc nhất trong phạm vi đó; `web-SC1` và `system-SC1` là hai scenario khác nhau). Ở umbrella + `spec_source`, sổ nằm trong spec repo.

**24 cột, tab-separated:**

| # | Cột | Ý nghĩa | Chủ ghi |
|---|-----|---------|---------|
| 1 | `sc_id` | `{UC-ID}-SC{N}` | generate-bdd |
| 2 | `sc_title` | tiêu đề scenario | generate-bdd |
| 3 | `spec_ver` | gương của `@trace.sc_version` hiện tại | generate-bdd · validate-traces |
| 4 | `gen_ver` | `spec_ver` **tại thời điểm codegen** | generate-code |
| 5 | `implemented_by` | `{Class}.{method}` (`—` nếu chưa) | generate-code |
| 6 | `test_count` | số test phủ SC | dev-gen-test · fix-bug |
| 7 | `test_classes` | tên test class / describe | dev-gen-test · fix-bug |
| 8 | `dev_selftest` | `pass`/`fail`/`not_run` — **dev tự chạy** | **chủ:** dev-run-test · *hạ hiệu lực:* generate-bdd · generate-code · fix-bug |
| 9 | `dev_selftest_at` | ngày | như trên |
| 10 | `qc_status` | `pass`/`fail`/`skip`/`not_run` — **QC chính thức** | **chủ:** qc-run-test · *hạ hiệu lực:* generate-bdd · generate-code |
| 11 | `qc_run_at` | ngày | như trên |
| 12 | `qc_owner` | SC đang chờ ai: `dev` / `po` | qc-run-test · report-bug |
| 13 | `qc_blocked_by` | `BUG-{id}` / `GAP-{id}` | qc-run-test · report-bug |
| 14 | `prd_version` | version PRD lúc sinh BDD | generate-bdd |
| 15 | `bdd_version` | version `.feature` | generate-bdd · review-context |
| 16 | `tech_doc_revision` | `@trace.revision` của tech-doc | generate-code · review-tech-docs |
| 17 | `fe_tech_doc_revision` | revision lúc FE wire adapter thật (§4.5.4) | generate-code |
| 18 | `prd_status` | gương của PRD Metadata `Status` | generate-bdd · validate-traces |
| 19 | `uc_status` | gương của `@trace.status` (`.feature`) | generate-bdd · validate-traces |
| 20 | `fe_phase` | `ui` / `integrated` / `—` | generate-code `--phase` |
| 21 | `status` | tổng hợp — xem bảng dưới | validate-traces |
| 22 | `last_updated` | `YYYY-MM-DD` | mọi lệnh ghi row |
| 23 | `service` | đội/submodule sở hữu — `multi`/`unresolved`/`—` | generate-bdd *(từ `@trace.service`)* |
| 24 | `design_spec_version` | version design-spec lúc sinh BDD — `—` cho backend | generate-bdd |

> `dev_selftest` (dev smoke) và `qc_status` (QC chính thức) là **hai tín hiệu riêng**, không bao giờ gộp. Cả hai **trực giao** với `status` — `status` đo *coverage*, chúng đo *kết quả chạy*.

> **Làm mất hiệu lực ≠ ghi đè.** Chủ sở hữu là người **duy nhất** ghi giá trị **khẳng định** (`pass`/`fail`/số lượng). Nhưng lệnh nào làm giá trị đó **hết đúng** (spec đổi, code đổi) **bắt buộc** hạ nó về `not_run`/`—`. Giữ một `pass` sinh ra từ spec đã bị sửa là **báo cáo sai**. Ngoại lệ có chủ ý: `qc_owner`/`qc_blocked_by` (con trỏ bug vẫn còn giá trị) và `test_count`/`test_classes` (test vẫn trên đĩa — cảnh báo, không hạ số).

> Giá trị rỗng trong TSV là `—`. Khi xuất JSON: `implemented_by`→`null`, `test_count`→`0`, `test_classes`→`[]`, `tech_doc_revision`/`fe_tech_doc_revision`→`0`, `dev_selftest`/`qc_status`→`"not_run"`, `service`/`design_spec_version`→`null`.

> **Backward-compat:** TSV cũ thiếu cột mới → đọc thành giá trị rỗng, **không báo lỗi**. Đọc theo **tên cột ở header row**, không theo vị trí. Header tự nâng lên 24 cột ở lần `/generate-bdd` gen lại kế tiếp — không cần script migration.

---

## Phân loại `status` (thứ tự ưu tiên, first-match-wins)

| # | Trạng thái | Điều kiện | Hành động |
|---|-----------|-----------|-----------|
| 0 | **ORPHANED** | SC **không còn trong `.feature`** nhưng `implemented_by != —` | Người quyết định: xoá code+test, hoặc đưa scenario trở lại |
| 1 | **UNTRACKED** | `implemented_by == —` | `/generate-code` |
| 2 | **DRIFT** | có code **và** `spec_ver != gen_ver` | regen **trước khi** test |
| 3 | **GAP** | có code **và** `test_count == —`/`0` | `/dev-gen-test` |
| 4 | **OK** | version khớp, có code, có test | đủ phủ |

`code_coverage = (rows where implemented_by != —) / total_scs` — **`total_scs` loại row `ORPHANED`** (không còn là scope; tính vào sẽ bóp méo coverage vì thứ không ai cần implement).

> **ORPHANED là Rule 0** vì 4 rule kia đều giả định scenario **còn tồn tại**. Để rule khác thắng thì mỗi giá trị route người dùng sang một lệnh vô nghĩa: `GAP`→sinh test cho SC không tồn tại · `DRIFT`→regen từ SC đã xoá · `OK`→cho tạo PR.
>
> **DRIFT xét trước GAP** vì `/generate-code` xử `GAP` = "skip codegen" còn `DRIFT` = "regenerate". Nếu GAP thắng, code lỗi thời bị bỏ qua và test lại sinh trên code cũ.

---

## Cờ audit (không phải cột — do `/validate-traces` tính)

| Cờ | Nguồn | Nghĩa |
|---|---|---|
| `PRD_UNTRACKED_EDIT` 🔴 | Step 3.9 | **nội dung PRD đổi mà nhãn `Version` KHÔNG đổi** — có người sửa ngoài `/generate-prd` · `/extend-prd` · `/amend-prd` · `/refine-prd` · `/review-context`. So bằng `git diff` **và** `git status` với mốc `spec_baseline`. Cố ý **không** vào `gate.blocking` — xem dưới |
| `PRD_DRIFT` | Step 4 | version PRD lệch **và** changelog **có** nêu UC này → nội dung đổi thật |
| `PRD_STALE_REF` ⓘ | Step 4 | version lệch nhưng changelog **không** nêu UC này → chỉ con trỏ cũ. `--realign-prd-version` |
| `BDD_DRIFT` | Step 5c | code mang `@trace.bdd_version` cũ hơn `.feature` |
| `TECHDOC_DRIFT` · `FE_TECHDOC_DRIFT` | Step 5 | code sinh từ revision tech-doc cũ hơn, **và** changelog nêu UC này |
| `TECHDOC_STALE_REF` ⓘ | Step 5 | đối xứng `PRD_STALE_REF`. `--realign-techdoc-revision` |
| `TECHDOC_STALE_VS_BDD` | Step 5c | tech-doc dựng từ BDD cũ hơn `.feature` hiện tại |
| `DESIGNSPEC_DRIFT` · `DESIGNSPEC_STALE_VS_BDD` | Step 5d | *(chỉ FE/App)* code / BDD dựng từ design-spec cũ hơn bản hiện tại |
| `TRACE_ORPHAN` 🔴 | Step 2b | tag `@trace.implements`/`@trace.verifies` trỏ SC không tồn tại **và** không có row TSV |
| `SEAM_UNWIRED` 🔴 | Step 5b | hàng thật đã có nhưng consumer còn wire vào stub |
| `STUB_UNRESOLVED` 🔴 | Step 5b | method còn trắng dù owner đã gen / có hàm song song |
| `SEAM_PENDING` · `STUB_PENDING` | Step 5b | owner UC chưa gen — **bình thường**, chỉ nhắc |

> 🔴 = **chặn PR**. Build xanh, test từng-UC xanh, coverage đẹp — nhưng luồng ghép chạy vào no-op hoặc code trỏ vào scenario đã bị xoá.
>
> 🔴 **`PRD_UNTRACKED_EDIT` là ngoại lệ có chủ ý — 🔴 nhưng KHÔNG chặn PR.** `gate.blocking` nghĩa hẹp là *code đang hỏng*; cờ này nói về **spec**, và code có thể đang hoàn toàn đúng. Thêm nữa: mọi project đang chạy đều đã có PRD sửa tay ⇒ một cờ chặn mới sẽ đỏ khắp nơi ở lần đầu ⇒ người ta **tắt cổng** ⇒ mất luôn 4 cờ chặn thật. Đó đúng là thất bại `self-check` R9(e) được viết ra để chống, chỉ đến bằng một cửa khác. Team đã dọn sạch nợ tồn thì tự thêm `prd_untracked_edit_count` vào `gate.blocking` (kèm `why`; R13(e) canh).
>
> **Đường ra tự lành, cố ý không có `--accept-edit`:** bump `Version` + ghi một row changelog nêu UC (tức đúng việc `/amend-prd` làm hộ) ⇒ `version != version_at_audit` ⇒ cờ tự tắt, và `PRD_DRIFT` bình thường tiếp quản. Một cờ escape sẽ là một đường **dán nhãn lên thay đổi chưa ai xem** — đúng cái ba rào của `--realign-*` tồn tại để chặn.
>
> ⓘ = **không phải lỗi.** Hai cờ `*_STALE_REF` tồn tại vì version PRD/tech-doc là **MỘT số cho cả tài liệu nhiều UC** — thêm một UC làm mọi UC cũ lệch số dù không đổi một chữ. Không lọc thì cả loạt UC ăn cờ đỏ oan, và làm theo hướng dẫn cũng không tắt được (`/generate-code` skip row đang `OK`). Sạch bằng `--realign-*`: chỉ sửa dòng `@trace.*`, **không đụng logic**, và **từ chối chạy** nếu UC đó đang thật sự `DRIFT`/`ORPHANED`.
>
> **Mọi cờ đều PHẢI có counter `{flag}_count`** trong Step 7 + `summary` của `trace-report.json` — `bin/self-check.js` R7 ép, không có ngoại lệ. Thiếu counter = cờ vô hình với dashboard.

---

## Dòng changelog — contract máy đọc

Hai cờ ⓘ ở trên hoạt động được là nhờ **một dòng văn bản**: row `# Change Log` của PRD (và `## Changelog` của tech-doc). `/validate-traces` Step 4/5 đọc nó để biết lần bump version đó **đụng UC nào**. Nên nó không phải ghi chú cho người đọc — nó là **đầu vào của một phép phân loại**.

Grammar: `bin/trace-schema.json` → `changelog_row_contract`. `bin/self-check.js` **R12** fail build nếu lệch.

```
| {version} | {YYYY-MM-DD} | {changelog_scope} |
```

**`{changelog_scope}`** — các mệnh đề ngăn bằng `;`, mỗi mệnh đề **mở đầu bằng đơn vị sở hữu**:

| Dạng | Nghĩa |
|---|---|
| `{UC-ID}: {mô tả}` | thay đổi thuộc UC này → UC này ăn 🟠 |
| `PRD-global:` / `doc-global:` | không thuộc UC nào → **không** UC nào ăn cờ |
| `… [no-behavior]` | producer **chứng minh được** mệnh đề này không đổi hành vi → UC nêu trong đó ở lại ⓘ. Danh sách được phép: `changelog_row_contract.neutral_checks` |

**Ví dụ đúng**

```
| 1.4 | 2026-08-19 | Auto-fix — UC5: banned-term (khách hàng→người mua); PRD-global: skeleton §4b [no-behavior] |
| 2.0 | 2026-08-22 | thêm UC7: AC12-AC14, BR21-BR23; UC3: sửa BR8 (giới hạn 5→20) |
| 3   | 2026-08-25 | UC1: thêm block client web §4.5.1.2, từ BDD web v1.2 |
```

**Hai kiểu sai — và chúng hỏng theo hai hướng ngược nhau**

| Viết sai | Consumer làm gì | Hướng hỏng |
|---|---|---|
| `cập nhật theo yêu cầu mới` — **mơ hồ**, không nêu đơn vị nào | 🟠 cho **MỌI** UC (lưới an toàn) | **ỒN.** Cờ sáng thường trực → người đọc học cách bỏ qua → lần lệch thật cũng bị bỏ qua |
| `…; sửa BR8` — **nêu BR mà bỏ UC sở hữu** | Row nêu đủ ID nên **không** bị coi là mơ hồ; nhưng phép thử là *"**UC** này có trong tập?"* nên UC3 → **ⓘ** | **IM LẶNG.** Và ⓘ **mở cửa** cho `--realign-*` (nó chỉ từ chối khi UC là 🟠) ⇒ nhãn version bị dán lại trên thay đổi chưa ai implement |

Ca thứ hai là lý do consumer **phải** chuẩn hoá tập bị ảnh hưởng **về UC** trước khi so — phép phân giải **`BR/AC → UC sở hữu`**: `BR{n}` → UC có BR đó trong bảng Business Rule (PRD §3) · `AC{n}` → UC có AC đó ở dòng `**AC liên quan:**`. Không phân giải được → coi **cả row** là mơ hồ, **không** bỏ qua im lặng.

> **Lưới an toàn không phải cái cớ để producer ghi bừa.** Nó đúng khi **thiếu** thông tin, và sai khi producer **có** thông tin mà không ghi. R12 canh đúng chỗ đó: producer phải có một **dòng template** mang `{changelog_scope}` (ví dụ đã điền cụ thể thì không bị canh), và consumer phải nhắc cả `[no-behavior]` lẫn phép phân giải — vì một marker được **ghi** mà không ai **hiểu** thì tệ hơn không có marker.

---

## Xuất JSON cho panel

`trace-report.json` giữ enum `status` **đúng 4 giá trị** `OK`/`DRIFT`/`GAP`/`UNTRACKED` — VS Code extension "Spec Driven Docs Tools" sống ngoài repo framework và switch trên field này. Row `ORPHANED` xuất ra là `"status": "DRIFT"` + `"orphaned": true`; panel cũ hiện nó như DRIFT (đúng nghĩa, không im lặng), panel mới đọc `orphaned` để hiện nhãn riêng. **TSV giữ nguyên chữ `ORPHANED`** — TSV là nguồn-sự-thật.

---

→ [Traceability](../02-concepts/traceability.md) · [Validate Traces](../02-concepts/pipeline-steps/09-validate-traces.md)
