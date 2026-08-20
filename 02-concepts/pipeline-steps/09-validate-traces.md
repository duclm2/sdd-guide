[← QC Automation](08-qc-automation.md) · [Pipeline Steps](README.md) · [Next: Feedback Loop →](10-feedback-loop.md)

# Bước 9 · Validate Traces — Ma trận độ phủ (Coverage Matrix)

> **Tóm tắt.** Check độ phủ giữa **spec ↔ code ↔ test** — 2 chiều quét, **6 tầng drift**, 4 cờ 🔴 chặn PR, 1 cờ 🔴 không-chặn (`PRD_UNTRACKED_EDIT`), 2 cờ ⓘ. Chỉ ra chỗ chưa phủ — mặc định **không sửa gì**.
> **Có phạm vi:** `--domain {d}` · `--prd {TICKET-ID}` · `--uc {UC-ID}`; không cờ nào = toàn bộ.
> **Command:** `/validate-traces` · `--realign-prd-version {UC-ID}` · `--realign-techdoc-revision {UC-ID}`
>
> *Lệnh **read-only** ở chế độ thường. Hai flag `--realign-*` là ngoại lệ có kiểm soát: chúng sửa **đúng dòng `@trace.*`** trong code, không đụng logic — xem [Realign](#realign--đường-ra-cho-cờ-ⓘ).*

| | |
|---|---|
| **Giai đoạn** | Quality (xuyên suốt) |
| **Owner** | 👤 Dev / QA / Lead |
| **Đầu vào** | `.trace/*.tsv` + spec + code + test |
| **Đầu ra** | Ma trận coverage + phân loại từng SC |
| **HITL** | ⚪ Read-only |

---

## Mục đích (Purpose)

Traceability chỉ có giá trị khi **kiểm được**. Bước này cho một **bức tranh toàn cục**: scenario nào đã có code, có test, hay còn hở — để không "tưởng xong mà chưa xong". Vì là **read-only**, chạy lúc nào cũng an toàn.

- Đối chiếu spec ↔ code ↔ test cho từng SC (chiều **spec → code**).
- Quét **chiều ngược** (code → spec): tag trỏ vào scenario **không còn tồn tại**.
- Phát hiện drift ở **4 tầng**: PRD · BDD · tech-doc → code, và tech-doc lỗi thời so với BDD.
- Bắt **mồ côi khi ghép luồng** (seam/stub) — lớp lỗi mà build xanh + test từng-UC xanh không thấy.
- Chỉ ra **gap** chưa phủ để lên kế hoạch bù.

---

## Input (Đầu vào)

- `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` (mọi UC × platform).
- Spec (`.feature`), code (`@trace.implements`), test (`@trace.verifies`).

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| Ma trận coverage spec ↔ code ↔ test | Trạng thái từng SC + `code_coverage` tổng |
| `{trace_dir}/trace-report.json` | Bản máy đọc cho **panel VS Code** ("Spec Driven Docs Tools") — bị **ghi đè** mỗi lần chạy |
| `{trace_dir}/trace-history.jsonl` | **Nhật ký append-only** — mỗi lần chạy ghi thêm 1 dòng *delta*. Đây là **dữ liệu**, không phải mirror: **phải commit**, mất là mất vĩnh viễn |
| Cờ audit | 6 cờ drift + 4 cờ 🔴 chặn PR + `PRD_UNTRACKED_EDIT` 🔴 (không chặn) + 2 cờ ⓘ (bảng dưới) |
| Hàng đợi | Đếm PRD change request còn `Open` kèm **số ngày chờ** (Step 7b) |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Dev/QA/Lead** | Đọc ma trận, quyết bù gap / regen drift |
| 🤖 **AI** | Quét trace, phân loại SC, dựng ma trận (không sửa file) |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Scenario nào **chưa có code** (UNTRACKED)? Chưa có test (GAP)?
- Code nào **lỗi thời** so với spec (DRIFT)?
- Có code nào đang trỏ vào **scenario đã bị xoá** (ORPHANED / TRACE_ORPHAN)?
- Màn FE nào **demo được nhưng chưa nối backend** (`fe_phase = ui`)?
- Luồng ghép có chỗ nào chạy vào **no-op** (SEAM_UNWIRED / STUB_UNRESOLVED)?
- Độ phủ tổng thể (`code_coverage`) bao nhiêu?

---

## Framework xử lý thế nào (Mechanics)

### Phạm vi audit — `--domain` / `--prd` / `--uc`

Đây là **lệnh đắt nhất** trong framework: ~24k token chỉ dẫn + ~9k `context-loader`, rồi đọc **mọi** PRD · `.feature` · tech-doc · design-spec · file source có tag · `.tsv`. Chi phí tăng **tuyến tính theo cả repo**, không theo phần việc đang làm.

| Cờ | `scope.kind` | Phạm vi |
|---|---|---|
| *(không có)* | `all` | Toàn bộ |
| `--domain {d}` | `domain` | Một domain |
| `--prd {TICKET-ID}` | `prd` | Một feature-package |
| `--uc {UC-ID}` | `uc` | Một UC — **mọi platform của nó** |

**`/sync` Step 1e nói cho bạn biết scope là gì**: nó liệt kê PRD nào vừa đổi *(so với lần pull)* và PRD nào đã đổi *kể từ lần audit gần nhất* — con số thứ hai tích luỹ đúng qua nhiều lần pull. Hai lệnh khớp nhau thành một vòng: `/sync` chỉ chỗ → audit scoped rẻ → sửa → audit **toàn bộ** một lần trước khi tạo PR.

⚠️ **Biên bản có scope KHÔNG BAO GIỜ được coi là đầy đủ.** Report mang field `scope`, và `gate-trace` **G2 fail** nếu `scope.kind !== "all"` — **không ngoại lệ**, không đếm xem trên đĩa có bao nhiêu domain. Muốn tạo PR thì phải có một lần audit **toàn bộ** đã commit.

> **Vì sao điều kiện phải tuyệt đối (GAPS-v4 G57):** bản cũ hỏi *"còn domain **nào khác** không"*, nên trong repo **một domain** thì không còn domain nào khác ⇒ **không fail** ⇒ một biên bản hẹp-theo-PRD được nhận là *"toàn bộ"*. Thêm cờ scope mà không siết G2 là **tự tay mở** đúng cái *"cấp giấy xanh cho thứ chưa ai xem"* mà chú thích của gate cảnh báo.

**Ba chỗ cố ý KHÔNG âm thầm:** `--prd`/`--uc` không phân giải được → **DỪNG** *(không rơi về `all` — chạy toàn bộ khi người ta xin một phần là đốt 30 phút; và không audit rỗng rồi báo "sạch" trên 0 row)* · nhiều cờ scope cùng lúc → **DỪNG** *(không tự ưu tiên)* · **lint vẫn chạy toàn bộ** dù audit có scope *(sổ hỏng ở domain khác vẫn là sổ hỏng, và lint rẻ vì không cần LLM)*.

### Phân loại `status` từng SC (thứ tự ưu tiên, rule sớm thắng)

| # | Trạng thái | Điều kiện |
|---|-----------|-----------|
| 0 | **ORPHANED** | SC **không còn trong `.feature`** nhưng `implemented_by != —` — code trỏ vào scenario đã bị xoá |
| 1 | **UNTRACKED** | `implemented_by == —` — scenario chưa từng sinh code |
| 2 | **DRIFT** | có `implemented_by` **và** `spec_ver != gen_ver` — spec đổi sau codegen → **regen trước khi test** |
| 3 | **GAP** | có `implemented_by` **và** (`test_count == — / 0`) — có code, chưa test |
| 4 | **OK** | `spec_ver == gen_ver`, có `implemented_by`, `test_count > 0` |

> **Vì sao ORPHANED là Rule 0:** 4 rule kia đều giả định scenario **còn tồn tại** — chúng trả lời *"spec này implement tới đâu"*. `ORPHANED` trả lời câu ngược: *"code này còn spec nào bảo lãnh không"*. Để rule khác thắng thì mỗi giá trị **route người dùng sang một lệnh vô nghĩa**: `GAP` → sinh test cho SC không tồn tại · `DRIFT` → regen từ SC đã xoá · `OK` → cho tạo PR.
>
> **Vì sao DRIFT xét trước GAP:** một SC đã có code, chưa test, **và** spec vừa drift phải hiện `DRIFT` (không phải `GAP`) — vì `/generate-code` xử `GAP` = "skip codegen" còn `DRIFT` = "regenerate". Nếu GAP thắng, code lỗi thời bị bỏ qua và test sinh trên code cũ.

`code_coverage = (rows where implemented_by != —) / total_scs` — và **`total_scs` loại row `ORPHANED`**: nó không còn là scope, tính vào mẫu số sẽ bóp méo coverage vì một thứ không ai cần implement.

### Hai chiều quét

| Chiều | Bước | Bắt gì |
|---|---|---|
| **spec → code** | Step 2 | mỗi row `.tsv` — SC đó implement/test tới đâu |
| **code → spec** | Step 2b | tag `@trace.implements`/`@trace.verifies` trỏ SC **không tồn tại** |

Chiều ngược là cần thiết vì gen lại BDD có thể làm một SC biến mất (gộp / đổi số / xoá) trong khi code implement nó vẫn nằm đó, vẫn được caller gọi. Không có Step 2b thì method đó **vô hình**: không status nào, không report nào — và coverage còn *đẹp hơn* thực tế vì mẫu số nhỏ đi.

### Cờ drift — 6 tầng

| Cờ | So cái gì | Step |
|---|---|:---:|
| `PRD_UNTRACKED_EDIT` 🔴 | **Nội dung PRD đổi mà nhãn `Version` KHÔNG đổi** — `git diff` **và** `git status` so với mốc `spec_baseline` của lần audit trước. Có người sửa ngoài đường chính thức | **3.9** |
| `PRD_DRIFT` | Version PRD vs cột `prd_version` vs `@trace.prd_version` trong code — **và** changelog **có** nêu UC này | 4 |
| `TECHDOC_DRIFT` · `FE_TECHDOC_DRIFT` | `@trace.revision` tech-doc vs cột đã lưu — **và** changelog nêu UC này | 5 |
| `BDD_DRIFT` | `@trace.bdd_version` trong code vs `.feature` hiện tại | 5c |
| `TECHDOC_STALE_VS_BDD` | map `@trace.bdd_versions` của tech-doc vs `.feature` hiện tại | 5c |
| `DESIGNSPEC_DRIFT` | `@trace.design_spec_version` trong code FE vs Version design-spec *(chỉ FE/App)* | 5d |
| `DESIGNSPEC_STALE_VS_BDD` | cột `design_spec_version` vs Version design-spec — BDD dựng từ bản cũ, **có thể thiếu Screen State / AC-UI vừa thêm** | 5d |

**Design-spec vào trace từ v0.4.3.** Trước đó nó là artifact upstream **duy nhất** không có cột TSV, không có tag trong code, không có cờ — dù nó điều khiển **cả** BDD FE/App (Screen States + AC-UI) **lẫn** code FE (màn hình, component inventory, Figma frame). Nó tự bảo vệ **một chiều** (reset `draft` khi PRD đổi); chiều *"designer sửa design-spec **sau khi** BDD/code đã sinh"* thì không gì bắt được.

`BDD_DRIFT` **bổ trợ** cho `sc_version`, không thay thế: `sc_version` bắt thay đổi trong **thân scenario**, `bdd_version` bắt thay đổi **cấp file** mà `sc_version` không thấy (Background, `@trace.dataset`, Business Definition, Coverage Matrix).

`TECHDOC_STALE_VS_BDD` là ca nguy hiểm nhất trong bảng: `/generate-code` DS3 thấy tech-doc `approved` sẽ lấy shape §4 **nguyên văn** làm contract "đã chốt" — contract dựng từ BDD cũ lan **thẳng** vào code. Cổng chặn nằm ở [`/review-tech-docs` **T3b**](05-tech-docs.md).

### 4 cờ 🔴 — chặn PR

| Cờ | Nghĩa |
|---|---|
| `ORPHANED` | code còn, scenario đã bị xoá khỏi `.feature` (row `.tsv` được giữ lại **chủ động**) |
| `TRACE_ORPHAN` | tag trỏ SC không tồn tại **và không có row `.tsv`** — nợ cũ, **không chỗ nào khác bắt được** |
| `SEAM_UNWIRED` | hàng thật đã có nhưng consumer còn wire vào stub → luồng chạy vào no-op |
| `STUB_UNRESOLVED` | method còn trắng dù owner UC đã gen / đã đẻ hàm song song |

> `SEAM_PENDING` / `STUB_PENDING` (owner chưa gen) là **bình thường** — chỉ nhắc.
>

### 2 cờ ⓘ — báo động oan đã được lọc ra

| Cờ | Nghĩa | Đường ra |
|---|---|---|
| `PRD_STALE_REF` | Version PRD lệch **nhưng changelog KHÔNG nêu UC này** → nội dung không đổi, chỉ con trỏ cũ | `--realign-prd-version {UC-ID}` |
| `TECHDOC_STALE_REF` | Đối xứng, cho tech-doc | `--realign-techdoc-revision {UC-ID}` |

**Vì sao cần tách hai cờ này ra.** PRD là tài liệu **cấp feature** phủ nhiều UC, nhưng Version của nó là **một scalar**. Thêm UC7 — không đụng một chữ nào của UC1–UC6 — vẫn làm **cả 6 UC cũ lệch version**. So version thuần thì cả 6 ăn cờ đỏ **oan**.

Tệ hơn: **làm theo hướng dẫn cũng không tắt được.** `/generate-bdd` sạch được cột TSV, nhưng tag trong code chỉ `/generate-code` ghi — mà nó thấy row đang `OK` là **skip**. Vòng lặp đóng, và lối ra duy nhất là ép sinh lại code cho hàng loạt UC không hề thay đổi.

Bộ lọc đọc **`{changelog_scope}` của row changelog** — một **contract máy đọc** (`bin/trace-schema.json` → `changelog_row_contract`) với **bốn** producer: `/refine-prd` Phase 3 · `/extend-prd` Bước 6 · `/amend-prd` Bước 5 · `/review-context` Fix/Resume Phase 3. `self-check` **R12** fail build nếu producer nào không có dòng template mang token.

Ba bước dựng tập bị ảnh hưởng:

1. **Tách mệnh đề** — ngăn bằng `;`, mỗi mệnh đề mở đầu bằng đơn vị sở hữu (`{UC-ID}:` hoặc `PRD-global:`).
2. **Chuẩn hoá về UC** — phép phân giải **`BR/AC → UC sở hữu`**: `BR{n}` → UC có BR đó trong bảng Business Rule (PRD §3) · `AC{n}` → UC có AC đó ở dòng `**AC liên quan:**`. **Không bỏ bước này**: phép thử là *"**UC** này có trong tập?"*, nên một row chỉ nêu `sửa BR8` làm UC sở hữu BR8 rơi vào ⓘ trong khi nội dung của nó vừa đổi — và ⓘ **mở cửa** cho `--realign-prd-version` dán nhãn lại (GAPS-v4 G53).
3. **Phân loại** *(first-match-wins)* — **row mơ hồ xét TRƯỚC**: nó là điều kiện **cấp row**, nên xét sau thì một UC có thể được xếp ⓘ trước khi ta biết là không suy đoán được gì.

**Row nào mơ hồ → 🟠 cho MỌI UC** — lưới an toàn: mất tính năng *lọc*, không mất tính năng *cảnh báo*. ⚠️ Nhưng nó **không phải cái cớ để producer ghi bừa**: đúng khi **thiếu** thông tin, sai khi producer **có** thông tin mà không ghi.

Mệnh đề mang hậu tố **`[no-behavior]`** → UC nêu trong đó ở lại ⓘ. Chỉ producer **chứng minh được** tính trung tính mới được dùng (`changelog_row_contract.neutral_checks`) — hiện chỉ `/review-context --fix` cho fix thuần cấu trúc (P4).

> Đây là bài mà framework **đã giải đúng ở cấp scenario**: `sc_version` chỉ bump khi thân scenario thực sự đổi, vì *"bump vô cớ sẽ tạo DRIFT giả, làm cờ mất giá trị"*. Hai cờ ⓘ là bản tương ứng ở cấp tài liệu.

### Realign — đường ra cho cờ ⓘ

```bash
/validate-traces --realign-prd-version {UC-ID}
/validate-traces --realign-techdoc-revision {UC-ID}
```

Cập nhật cột TSV **và** tag trong code lên version hiện tại. **Ba rào an toàn:**

1. **Từ chối chạy** nếu UC đang `DRIFT` / `ORPHANED` / bị gắn 🟠 — khi đó nội dung **đổi thật**, dán nhãn lại là **che lỗi**
2. **Chỉ sửa dòng `@trace.*`** — guard sau-ghi diff lại, lệch là khôi phục file
3. **In chính xác** file + dòng đã sửa — realign im lặng là realign không kiểm chứng được

> Không đụng `dev_selftest`/`qc_status`: tiền đề của realign là **không có logic nào đổi**, nên luật *"làm mất hiệu lực"* không áp.

### Nhật ký lịch sử — đo *tốc độ*, không chỉ *trạng thái*

`trace-report.json` bị **ghi đè** mỗi lần chạy, và cột `last_updated` chỉ là một ngày bị 8 lệnh cùng ghi đè. Nên framework đo **trạng thái** rất tốt nhưng không đo được **tốc độ**.

Step 8c append 1 dòng *delta* vào `{trace_dir}/trace-history.jsonl` mỗi lần chạy — vài trăm byte, rotate theo tháng — rồi in:

```
📈 So lần chạy trước (12/08): code +2% · test −1% · DRIFT +3 · UNTRACKED −5
   Đèn mới bật: SEAM_UNWIRED × 1   ·   Đèn đã tắt: STUB_UNRESOLVED × 2
```

Trả lời được: *lệch từ bao giờ · đang lên hay xuống · case này hỏng đi hỏng lại mấy lần · sprint vừa rồi đóng được bao nhiêu.*

| Ràng buộc | Vì sao |
|---|---|
| Chỉ ghi vào `{trace_dir}`, **không** mirror | Nhân bản dữ liệu tích luỹ ra chỗ sinh-ra = hai lịch sử lệch nhau |
| **Phải commit** cùng `.tsv` | Là **dữ liệu**, không regenerate được |
| Không đụng TSV / `trace-report.json` | TSV là bảng **trạng thái** (giữ phẳng); JSON là contract với panel |
| **Không lệnh nào gác cổng dựa trên nó** | Để **nhìn**, không phải để chặn. Cổng PR vẫn chỉ là 4 cờ 🔴 |
> **Build xanh, test từng-UC xanh, coverage đẹp — vẫn có thể còn 🔴.** Đó chính là lớp lỗi mà hai thứ kia không bắt được. `ORPHANED`/`TRACE_ORPHAN` **không tự hết**: phải có người quyết định xoá code+test, hay đưa scenario trở lại `.feature`.

### Tín hiệu FE còn dùng mock

`fe_on_mock` = số SC có `fe_phase = ui` — FE đã có UI nhưng **chưa wire API thật**. Các SC này có thể đang hiện `OK` (có code, có test) nhưng **test chạy trên mock** → đừng coi là xong tính năng. Route: `/generate-code {feature-file} --phase=integration`.

---

## HITL / Gate

- ⚪ **Read-only** — bỏ qua checkpoint ghi-file. Đây là công cụ *quan sát*, không phải *hành động*.

---

## Ví dụ (Example)

```
/validate-traces auth

🔴 GATE — có MỒ CÔI: 1 SEAM_UNWIRED · 0 STUB_UNRESOLVED · 1 ORPHANED · 0 TRACE_ORPHAN
   Build xanh, test từng-UC xanh, coverage đẹp — nhưng luồng ghép chạy vào no-op,
   hoặc code đang trỏ vào scenario đã bị xoá. KHÔNG coi là pass tới khi CẢ BỐN = 0.

  AUTH-UC1  SC1  OK
  AUTH-UC1  SC2  GAP        (có code, chưa test)
  AUTH-UC2  SC1  DRIFT      (spec_ver 1.1 ≠ gen_ver 1.0 → regen)
  AUTH-UC2  SC3  UNTRACKED  (chưa sinh code)
  AUTH-UC2  SC7  ORPHANED   ⚠ đã xoá khỏi spec nhưng code còn
  code_coverage = 7/9 (78%)        ← mẫu số KHÔNG tính row ORPHANED

BDD Version Drift:
  AUTH-UC1 (web) — code sinh từ BDD v1.4, .feature giờ v1.6   [SC đang DRIFT: SC2]

FE còn dùng mock (fe_phase = ui):
  AUTH-UC1 (web) — 3 SC — có test nhưng chạy trên mock, chưa nối API thật
```

---

## Anti-pattern

- ❌ Coi validate-traces là "chạy xong là fix xong" — nó chỉ báo cáo; hành động ở `/generate-code` (regen DRIFT) và QC (bù GAP).
- ❌ Bỏ qua DRIFT rồi test trên code cũ.
- ❌ **Tạo PR khi còn cờ 🔴.** Build xanh không chứng minh luồng ghép chạy đúng.
- ❌ Coi `fe_phase = ui` là xong vì status đã `OK` — test đang chạy trên mock.
- ❌ Xoá row `ORPHANED` khỏi `.tsv` cho "sạch bảng" — làm thế là đưa code mồ côi về trạng thái **vô hình**, đúng cái bug mà Rule 0 sinh ra để chống.

---

## Bước tiếp theo (Next step)

**Thứ tự xử lý:** cờ 🔴 trước (chặn PR) → rồi drift → rồi GAP. Route đầy đủ nằm ở bảng "Gợi ý lệnh tiếp theo" cuối report.

Gap/drift phát hiện → quay lại `/generate-code` (regen) hoặc bù test; lỗi/thiếu từ QC → vòng phản hồi:

➡️ [Bước 10 · Feedback Loop — `/report-bug` · `/propose-scenario` · `/learn` · `/sync`](10-feedback-loop.md)
