[← /qc-metrics](33-qc-metrics.md) · [Explain Home](README.md) · [Next: /generate-spec-manifest →](22-generate-spec-manifest.md)

# 21 · `/validate-traces` — Ma trận độ phủ spec ↔ code ↔ test

> **Một câu.** Audit độ phủ giữa spec, code, test (gồm PRD version drift); phân loại mỗi SC và làm mới Living Docs. **Không sửa code/spec** — nhưng **có ghi**: `status`/`spec_ver` về TSV, `trace-report.json`, `.trace-mirror/`, append `trace-history.jsonl` (và với `--reconcile-code` / `--realign-*` thì thêm row TSV, `_seams.tsv`, dòng `@trace.*`).

---

## Vấn đề giải quyết

Traceability chỉ có giá trị khi kiểm được. Command cho bức tranh toàn cục: SC nào có code, có test, hay còn hở — để không "tưởng xong mà chưa xong".

---

## Vị trí & tiền đề

- **Vị trí:** Phase Trace Audit (xuyên suốt).
- **Đặc biệt:** an toàn chạy bất kỳ lúc nào — mọi thao tác ghi vô điều kiện đều dựng lại được hoặc chỉ append; hai thao tác nguy hơn nằm sau cờ.
- **Target:** không có target file đơn — phạm vi do Step 0-A phân giải từ cờ. Token trần: UC-ID → `--uc`, TICKET-ID → `--prd`; không phân giải được → **DỪNG**, không rơi về `all`.

---

## Input / Output

**Input:** `.trace/…/{UC-ID}-{platform}.tsv` + spec + code + test.

**Output:** ma trận coverage + `code_coverage`; `trace-report.json` (ghi đè) + **`trace-history.jsonl`** (append 1 dòng delta — **phải commit**, không regenerate được); làm mới Living Docs dashboard.

**Flag — phạm vi:** `--domain {d}` · `--prd {TICKET-ID}` · `--uc {UC-ID}`; không cờ nào = **toàn bộ**. Report mang field `scope`, và `gate-trace` **G2 fail** nếu `scope.kind !== "all"` — **không ngoại lệ**, không đếm số domain trên đĩa. Biên bản có scope **không bao giờ** là biên bản đầy đủ; muốn tạo PR thì phải có một lần audit toàn bộ đã commit. *(`/sync` Step 1e nói cho bạn biết scope là gì.)*

**Flag — realign:** `--realign-prd-version {UC-ID}` · `--realign-techdoc-revision {UC-ID}` — ngoại lệ có kiểm soát của "không sửa code": chỉ sửa **dòng `@trace.*`** trong code, **từ chối chạy** nếu UC đang `DRIFT`/`ORPHANED` hoặc bị Step 4/5 xếp 🟠.

**Flag — reconcile:** `--reconcile-code` *(Step 5f)* — đường ra cho cờ `LEDGER_BEHIND_CODE` 🟠 (code đã có tag `@trace.implements` mà sổ ghi `implemented_by = —`): điền `implemented_by`/`gen_ver`/`fe_phase` từ tag, `test_count` **chỉ** khi có `@trace.verifies`, **không** set `status`. Chỉ điền row trống; không tham số = mọi row lệch trong scope.

**Cờ mới `PRD_UNTRACKED_EDIT`** 🔴 *(Step 3.9, chạy **trước** Step 4)* — nội dung PRD đổi mà nhãn `Version` **không** đổi ⇒ có người sửa ngoài `/generate-prd` · `/extend-prd` · `/amend-prd` · `/refine-prd` · `/review-context`. So bằng `git diff` **và** `git status` *(nguồn thứ hai bắt ca sửa **chưa** commit — ca thường gặp nhất)* với mốc `spec_baseline` mà Step 6b ghi vào `{trace_dir}/trace-report.json` (file được commit; chạy có phạm vi vẫn giữ mốc của PRD ngoài phạm vi). **Không chặn PR** có chủ ý, và **không có `--accept-edit`**: đường ra là bump `Version` + row changelog ⇒ cờ tự tắt.

---

## Các bước xử lý (chi tiết)

0. **Step 1.0 — lint sổ trace trước khi đọc** (bắt buộc): `--lint-trace`; rule sổ (T1–T14) luôn chạy toàn bộ, rule tech-doc thu hẹp theo `--scope-prd` khi có scope.
1. Quét trace `.tsv` + đối chiếu spec/code/test. **Step 2b — reverse audit** (chiều code → spec), một lượt quét hai câu: tag trỏ vào SC đã bị xoá → `ORPHANED` / `TRACE_ORPHAN` 🔴; code có tag mà sổ chưa ghi → `LEDGER_BEHIND_CODE` 🟠.
2. Phân loại mỗi SC theo **thứ tự ưu tiên** (rule sớm thắng):
   | # | Trạng thái | Điều kiện |
   |---|-----------|-----------|
   | 0 | ORPHANED | SC không còn trong `.feature` mà vẫn có code |
   | 1 | UNTRACKED | `implemented_by == —` (chưa sinh code) |
   | 2 | DRIFT | có code + `spec_ver != gen_ver` |
   | 3 | GAP | có code + `test_count == —/0` |
   | 4 | OK | version khớp + có code + có test |
3. Dựng dashboard: `dev_selftest` (DEV smoke) **và** `qc_status` (QC chính thức) hiển thị cạnh nhau — **không merge**; cột `qc_owner` + `qc_blocked_by` ("Waiting on"); và **hai trục chia nhóm**: **`by_service`** (coverage theo từng đội — cột `service`) · **`by_platform`** (coverage theo `web`/`app`/`system`).
   > `by_platform` trả lời *"web xong bao nhiêu %, system xong bao nhiêu %"* — câu thường ngày khi làm FE và BE song song. Trước v0.5.1 không trả lời được **từ `summary`**: `by_service` là bảng chia nhóm duy nhất, mà cột `service` là `—` ở mọi row của dự án single-service ⇒ nó gộp tất cả vào một ô.
4. **Lọc báo động oan (Step 4/5).** PRD và tech-doc là tài liệu **gộp** nhiều UC nhưng chỉ **một** số version — thêm UC7 làm mọi UC cũ lệch số dù không đổi một chữ. Đọc **scope của row changelog**: UC có trong danh sách → `PRD_DRIFT` 🟠 · không có → `PRD_STALE_REF` ⓘ (sạch bằng `--realign-*`) · row **mơ hồ** → 🟠 cho mọi UC (lưới an toàn). Riêng tech-doc: mệnh đề `UC3[web]: …` (FE ghi phần client song song với BE) chỉ bật `FE_TECHDOC_DRIFT` cho row web của UC3 — phần BE không bị báo oan.
5. **Step 5c — BDD drift**: `@trace.bdd_version` của `.feature` vs code (`BDD_DRIFT`) và vs map `@trace.bdd_versions` của tech-doc (`TECHDOC_STALE_VS_BDD`). **Step 5d — design-spec drift** *(chỉ FE/App)*: 2 chiều, design-spec→BDD và design-spec→code. **Step 5b — Seam & Stub audit**: port cross-UC còn nối vào stub trong khi hàng thật đã có (`SEAM_UNWIRED` 🔴), method `@trace.stub` còn rỗng khi owner đã gen (`STUB_UNRESOLVED` 🔴); cập nhật `_seams.tsv`.
   **Step 6 — ghi lại TSV**: `spec_ver` · `status` · `last_updated`, đồng bộ `uc_status` ← `@trace.status` của `.feature` và `prd_status` ← `Status` của PRD; không đụng cột `dev_selftest*` / `qc_*`. Step 6b ghi mốc `spec_baseline` cho Step 3.9 lần sau.
6. **Step 7b — hàng đợi**: đếm PRD change request còn `Open` kèm **số ngày chờ** (hàng đợi duy nhất không có lệnh nào quét lại mỗi lần chạy).
7. **Step 8c — nhật ký**: append delta vào `trace-history.jsonl` → in khối `📈 So lần chạy trước` (đo **tốc độ**, không chỉ trạng thái).

---

## Checkpoint & Gate

- ⚪ **Không chặn** — *không phải vì read-only* (lệnh có ghi file), mà vì mọi thao tác ghi vô điều kiện đều dựng lại được hoặc là append; thao tác nguy hơn nằm sau `--reconcile-code` / `--realign-*`.
- Report không phải cổng, nhưng các cờ 🔴 (`SEAM_UNWIRED` · `STUB_UNRESOLVED` · `ORPHANED` · `TRACE_ORPHAN`) **chặn PR** qua `gate-trace`.

---

## Cơ chế đặc biệt

- **DRIFT xét trước GAP** — code lỗi thời chưa test phải hiện DRIFT (regen) không phải GAP.
- **`dev_selftest` ≠ `qc_status`** — hai cột riêng, không trộn.
- **"Waiting on" column** — `qc_owner`/`qc_blocked_by` trả lời "case nào chờ ai".

---

## 👓 Góc nhìn tối ưu

- **Chỉ báo cáo, không hành động** — hành động ở `/generate-code` (regen DRIFT) & QC (bù GAP). Chuỗi phụ thuộc người chạy tiếp.
- **Nguồn sự thật của Living Docs** — chất lượng dashboard phụ thuộc `.tsv` được các lệnh code/test ghi đúng.
- **Là "single pane" để PM/PO nhìn trạng thái** — ứng viên tốt cho UI viewer (blueprint có gợi ý).

---

## Kết nối

**Trước:** bất kỳ (đặc biệt sau [`/qc-report`](20-qc-report.md)) · **Sau:** cờ 🔴 trước — SEAM_UNWIRED → nối binding sang class thật · STUB_UNRESOLVED → `/generate-code {owner_uc}` · ORPHANED/TRACE_ORPHAN → quyết định thủ công; `LEDGER_BEHIND_CODE` → `--reconcile-code`; rồi DRIFT/UNTRACKED → [`/generate-code`](09-generate-code.md); GAP → [`/dev-gen-test`](12-dev-gen-test.md). **Chỉ tạo PR khi mọi cờ 🔴 = 0.**
