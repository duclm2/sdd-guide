[← /dev-smoke-test](14-dev-smoke-test.md) · [Explain Home](README.md) · [Next: /qc-plan →](16-qc-plan.md)

# 15 · `/qc-analyze` — Trạm 1: Phân rã yêu cầu + gap tài liệu

> **Một câu.** Trạm đầu của dây chuyền QC: phân rã yêu cầu từ spec, map trace, và ghi **DOC_GAP** (chỗ tài liệu thiếu để test được). Markdown-first, chưa có script.

> 🏭 **Dây chuyền QC 9 trạm** (1 → 9, kèm 8b) dùng module `qc-playwright-ts` (web + API/`system`) hoặc `qc-wdio-appium` (mobile) và **QC skill** nạp từ `paths.qc_skills_dir` — mặc định bundled `.agent/skills/qc`, **override sang repo riêng của team QC** để nâng cấp framework không ghi đè (QC skill tiến hoá độc lập).

---

## Vấn đề giải quyết

QC chính thức cần hiểu yêu cầu **testable** trước khi viết test. Trạm này phân rã spec thành phân tích yêu cầu + phát hiện **gap tài liệu** (thiếu gì để test được) → tránh viết test trên hiểu lầm.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 1), **sau `/review-tech-docs` APPROVED** — chạy **song song** với `/generate-code`, chưa cần code (BDD + tech-doc đã đóng băng là đủ).
- **Gate vào:** `steps/qc-scope.md` — chỉ UC có BDD `approved` vào phạm vi; UC nháp ghi `⏸ Chưa xét`; **0 UC approved → DỪNG**. `--force` để xét cả bản nháp (artifact tự khai dựa trên BDD nháp).
- **Đặc biệt:** **Platform Resolution** ở đây khoá platform cho **cả QC pass** — mọi trạm sau kế thừa.

---

## Input / Output

**Input:** `{TICKET-ID} [platform]` (UC-ID / file `.feature` / file PRD cũng được — đều quy về TICKET-ID) + spec (PRD/BDD/design-spec từ spec repo) + **tech-doc gộp** `{TICKET-ID}-tech-design.md` làm nguồn thứ hai + skill `qa-analyst`.

**Output (per PRD × nền):** `{qc_dir}/{TICKET-ID}-{prd-slug}/{platform}/REQUIREMENT_ANALYSIS.md` + `DOC_GAP.md` (11 cột, cột 2 là `UC`) + `{refinement_dir}/{TICKET-ID}-qa-findings.yaml`. Một lần chạy phủ **mọi UC có BDD `approved`** của PRD; UC còn nháp vào bảng *Phạm vi phân tích* với dấu `⏸ Chưa xét` (`--force` để xét luôn).

---

## Các bước xử lý (chi tiết)

1. **Platform Resolution** — chốt 1 platform cho toàn QC pass (sổ trace ghi `qc_status` sẽ là `{UC-ID}-{platform}.tsv`).
2. **Phạm vi — BDD approved?** — `qc-scope` chia UC thành *trong phạm vi* / `⏸ Chưa xét`; 0 UC approved → DỪNG (trừ `--force`).
3. **Kiểm kê tài liệu đầu vào** — in bảng PRD · BDD · design-spec · tech-doc kèm **phiên bản + ngày + dựng trên PRD nào**; lệch phiên bản → ghi gap loại SYNC, **không chặn**.
4. **Đối chiếu tech-doc** — so §3/§4/§4.5/§5/§6/§8 với PRD/BDD, mỗi chỗ vênh là gap `CONTRADICTORY` (vd PRD "lớp 1–6", contract enum `1..9`). Thiếu tech-doc → cảnh báo mềm. **§12 GAP Register chỉ đọc, không ghi** — ẩn số `open` ghi vào mục *Đang chờ chốt*, không mở gap trùng (trừ loại `spec-defect`).
5. **Role qa-analyst** — nạp skill `{qc_skills_dir}/qa-analyst/` (spec-breakdown → business-rules / data-flow → acceptance-criteria).
6. **Trace mapping (bắt buộc)** — map yêu cầu ↔ scenario ↔ SC, cho **từng UC** trong phạm vi (`BR`/`AC` mang rõ UC).
7. **Quét gap theo lăng kính** — `steps/review-fanout.md` với 4 lăng kính `D2 Xử lý lỗi` · `D3 Giao diện` · `D4 Dữ liệu & cấu hình` · `D5 Đối chiếu chéo` (thu hẹp: chỉ PRD↔design-spec, bdd↔design-spec); `GRANULARITY=auto`. Gộp gap của cả hai nguồn rồi mới thẩm định.
8. **DOC_GAP (bắt buộc)** — ghi chỗ tài liệu thiếu/mơ hồ chặn test. `🔴 Blocker` còn `Open`/`Stale` **chặn theo từng UC, không chặn cả PRD** — và chặn *chạy* test, không chặn *viết* test.
9. **Thẩm định gap (`steps/gap-verify.md`, bắt buộc)** — lọc gap ảo (đã trả lời ở tài liệu khác, trích sai, trùng); gap rớt được đóng kèm lý do, không xoá. In khối `[GAP VERIFY]`.
10. **Guard — BR-tag** *(mới)* — phép **đếm cơ học** chống bỏ sót business rule:

   ```
   A = mọi business rule mà .feature ĐÃ GẮN TAG (@trace.business_rules)
   B = mọi rule bản phân tích VỪA SINH RA (REQUIREMENT_ANALYSIS.md)
   A ∖ B  → rule BDD đã nhắc mà phân tích BỎ SÓT
   ```

   `A ∖ B` rỗng → in `Guard BR-tag: khớp {n}/{n}`. Không rỗng → **tự bổ sung từ PRD** rồi in danh sách đã bù. **In dòng này kể cả khi sạch** — guard im lặng khi sạch là guard không ai biết nó tồn tại.
11. **Self-Review** — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
12. **Output** REQUIREMENT_ANALYSIS.md + DOC_GAP.md + `qa-findings.yaml`; append một dòng `"phase":"qc-analyze"` vào `quality-metrics.jsonl` (dừng giữa chừng thì không ghi).
> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard — chính file `self-review-principles.md` ghi rõ ranh giới đó ngay ở đầu.


---

## Checkpoint & Gate

- **CHECKPOINT: chặn CỨNG khi `DOC_GAP.md` đã tồn tại** — ghi đè là mất cột `Trạng thái`/`Câu trả lời` PO điền tay và đánh số lại GAP-ID (phá `🚫 Block` trong `.Test.md`). `--yes` **không** bỏ qua được. Lần chạy đầu không ghi đè gì → đi thẳng.
- **Chạy lại = HOÀ (thêm, không thay):** giữ mã gap + câu trả lời; gap không còn thấy → `Stale` (không xoá hàng); mã chỉ cấp mới, không tái dùng. Muốn lập lại từ trắng phải thêm `--force` (report khai đã xoá bao nhiêu câu trả lời).
- **Phạm vi (`qc-scope`)** — UC nháp không bị chặn im lặng mà ghi `⏸ Chưa xét`; 0/n approved → DỪNG. `--yes` không thay được `--force`.
- **Platform Resolution chạy TRƯỚC Guard** (sửa 2026-08-25): UC đa nền có nhiều `.feature`, mỗi bản
  mang `@trace.status` riêng — kiểm duyệt trước khi biết platform là kiểm nhầm bản.

---

## Cơ chế đặc biệt

- **QC skill từ repo riêng** (`qc_skills_dir`) — QC team sở hữu, không bị framework upgrade ghi đè.
- **Platform khoá một lần** cho cả pass → nhất quán sổ trace.
- **DOC_GAP** — kênh QC phản hồi ngược chất lượng tài liệu (song song `/report-bug`).
- **`Stale` có hạn 14 ngày** (`review-by`) — quá hạn thì máy **không tự chuyển**: in khối `🟠 Cần người quyết` cuối report (kể cả khi `--yes`), đổi dòng `Trạng thái chung` đầu `DOC_GAP.md`, đẩy chỉ số `stale_overdue` lên `/qc-metrics`, và `Go/No-Go: 🔴 no-go`.
- **`qa-findings.yaml`** — cùng tập gap render cho Review Board; `auto_fixable` luôn `false`, không có `--resume` (gap phát hiện sau khi BDD + tech-doc đã đóng băng, tự sửa PRD là sửa sau lưng dây chuyền).

---

## 👓 Góc nhìn tối ưu

- **DOC_GAP trùng vai với review-context B-check?** — cả hai bắt gap tài liệu, nhưng QC nhìn từ góc "test được không". Đáng xem có nối được feedback này về spec.
- **Platform một pass** — feature multi-platform phải chạy QC pass nhiều lần. Chi phí lặp phân tích.
- **Phụ thuộc QC skill ngoài** — nếu `qc_skills_dir` trỏ sai/thiếu, trạm hụt logic.

---

## Kết nối

**Trước:** [`/review-tech-docs`](08-review-tech-docs.md) APPROVED (nhánh song song với `/generate-code`) · **Sau:** xử gap blocker 🔴 → [`/qc-plan`](16-qc-plan.md).
