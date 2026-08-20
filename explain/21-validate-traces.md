[← /qc-report](20-qc-report.md) · [Explain Home](README.md) · [Next: /generate-spec-manifest →](22-generate-spec-manifest.md)

# 21 · `/validate-traces` — Ma trận độ phủ spec ↔ code ↔ test

> **Một câu.** Check **read-only** độ phủ giữa spec, code, test (gồm PRD version drift); phân loại mỗi SC và làm mới Living Docs. Không sửa gì.

---

## Vấn đề giải quyết

Traceability chỉ có giá trị khi kiểm được. Command cho bức tranh toàn cục: SC nào có code, có test, hay còn hở — để không "tưởng xong mà chưa xong".

---

## Vị trí & tiền đề

- **Vị trí:** Phase Trace Audit (xuyên suốt).
- **Đặc biệt:** read-only — an toàn chạy bất kỳ lúc nào.

---

## Input / Output

**Input:** `.trace/…/{UC-ID}-{platform}.tsv` + spec + code + test.

**Output:** ma trận coverage + `code_coverage`; `trace-report.json` (ghi đè) + **`trace-history.jsonl`** (append 1 dòng delta — **phải commit**, không regenerate được); làm mới Living Docs dashboard.

**Flag — phạm vi:** `--domain {d}` · `--prd {TICKET-ID}` · `--uc {UC-ID}`; không cờ nào = **toàn bộ**. Report mang field `scope`, và `gate-trace` **G2 fail** nếu `scope.kind !== "all"` — **không ngoại lệ**, không đếm số domain trên đĩa. Biên bản có scope **không bao giờ** là biên bản đầy đủ; muốn tạo PR thì phải có một lần audit toàn bộ đã commit. *(`/sync` Step 1e nói cho bạn biết scope là gì.)*

**Flag — realign:** `--realign-prd-version {UC-ID}` · `--realign-techdoc-revision {UC-ID}` — ngoại lệ có kiểm soát của "read-only": chỉ sửa **dòng `@trace.*`** trong code, **từ chối chạy** nếu UC đang `DRIFT`/`ORPHANED` hoặc bị Step 4/5 xếp 🟠.

**Cờ mới `PRD_UNTRACKED_EDIT`** 🔴 *(Step 3.9, chạy **trước** Step 4)* — nội dung PRD đổi mà nhãn `Version` **không** đổi ⇒ có người sửa ngoài `/generate-prd` · `/extend-prd` · `/amend-prd` · `/refine-prd` · `/review-context`. So bằng `git diff` **và** `git status` *(nguồn thứ hai bắt ca sửa **chưa** commit — ca thường gặp nhất)* với mốc `spec_baseline` mà Step 6b ghi. **Không chặn PR** có chủ ý, và **không có `--accept-edit`**: đường ra là bump `Version` + row changelog ⇒ cờ tự tắt.

---

## Các bước xử lý (chi tiết)

1. Quét trace `.tsv` + đối chiếu spec/code/test.
2. Phân loại mỗi SC theo **thứ tự ưu tiên** (rule sớm thắng):
   | # | Trạng thái | Điều kiện |
   |---|-----------|-----------|
   | 1 | UNTRACKED | `gen_ver == —` |
   | 2 | DRIFT | có code + `spec_ver != gen_ver` |
   | 3 | GAP | có code + `test_count == —/0` |
   | 4 | OK | version khớp + có code + có test |
3. Dựng dashboard: `dev_selftest` (DEV smoke) **và** `qc_status` (QC chính thức) hiển thị cạnh nhau — **không merge**; cột `qc_owner` + `qc_blocked_by` ("Waiting on"); và **hai trục chia nhóm**: **`by_service`** (coverage theo từng đội — cột `service`) · **`by_platform`** (coverage theo `web`/`app`/`system`).
   > `by_platform` trả lời *"web xong bao nhiêu %, system xong bao nhiêu %"* — câu thường ngày khi làm FE và BE song song. Trước v0.5.1 không trả lời được **từ `summary`**: `by_service` là bảng chia nhóm duy nhất, mà cột `service` là `—` ở mọi row của dự án single-service ⇒ nó gộp tất cả vào một ô.
4. **Lọc báo động oan (Step 4/5).** PRD và tech-doc là tài liệu **gộp** nhiều UC nhưng chỉ **một** số version — thêm UC7 làm mọi UC cũ lệch số dù không đổi một chữ. Đọc **scope của row changelog**: UC có trong danh sách → `PRD_DRIFT` 🟠 · không có → `PRD_STALE_REF` ⓘ (sạch bằng `--realign-*`) · row **mơ hồ** → 🟠 cho mọi UC (lưới an toàn).
5. **Step 5d — design-spec drift** *(chỉ FE/App)*: 2 chiều, design-spec→BDD và design-spec→code.
6. **Step 7b — hàng đợi**: đếm PRD change request còn `Open` kèm **số ngày chờ** (hàng đợi duy nhất không có lệnh nào quét lại mỗi lần chạy).
7. **Step 8c — nhật ký**: append delta vào `trace-history.jsonl` → in khối `📈 So lần chạy trước` (đo **tốc độ**, không chỉ trạng thái).

---

## Checkpoint & Gate

- ⚪ Read-only, không gate.

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

**Trước:** bất kỳ (đặc biệt sau [`/qc-report`](20-qc-report.md)) · **Sau:** DRIFT/UNTRACKED → [`/generate-code`](09-generate-code.md); GAP → [`/dev-gen-test`](12-dev-gen-test.md); OK → PR.
