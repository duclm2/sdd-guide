[← /dev-smoke-test](14-dev-smoke-test.md) · [Explain Home](README.md) · [Next: /qc-plan →](16-qc-plan.md)

# 15 · `/qc-analyze` — Trạm 1: Phân rã yêu cầu + gap tài liệu

> **Một câu.** Trạm đầu của dây chuyền QC: phân rã yêu cầu từ spec, map trace, và ghi **DOC_GAP** (chỗ tài liệu thiếu để test được). Markdown-first, chưa có script.

> 🏭 **Dây chuyền QC 6 trạm** dùng module `qc-playwright` và **QC skill** nạp từ `paths.qc_skills_dir` — mặc định bundled `.agent/skills/qc`, **override sang repo riêng của team QC** để nâng cấp framework không ghi đè (QC skill tiến hoá độc lập).

---

## Vấn đề giải quyết

QC chính thức cần hiểu yêu cầu **testable** trước khi viết test. Trạm này phân rã spec thành phân tích yêu cầu + phát hiện **gap tài liệu** (thiếu gì để test được) → tránh viết test trên hiểu lầm.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 1), sau code chạy được.
- **Gate vào:** Guard — cảnh báo nếu BDD chưa approved.
- **Đặc biệt:** **Platform Resolution** ở đây khoá platform cho **cả QC pass** — mọi trạm sau kế thừa.

---

## Input / Output

**Input:** UC-ID + spec (PRD/BDD từ spec repo) + skill `qa-analyst`.

**Output (per PRD × nền):** `{qc_dir}/{TICKET-ID}/{platform}/REQUIREMENT_ANALYSIS.md` + `DOC_GAP.md` (11 cột, cột 2 là `UC`) + `{refinement_dir}/{TICKET-ID}-qa-findings.yaml`. Một lần chạy phủ **mọi UC có BDD `approved`** của PRD; UC còn nháp vào bảng *Phạm vi phân tích* với dấu `⏸ Chưa xét` (`--force` để xét luôn).

---

## Các bước xử lý (chi tiết)

1. **Guard — BDD approved?** — cảnh báo mềm nếu chưa.
2. **Platform Resolution** — chốt 1 platform cho toàn QC pass (sổ trace ghi `qc_status` sẽ là `{UC-ID}-{platform}.tsv`).
3. **Role qa-analyst** — nạp skill `{qc_skills_dir}/qa-analyst/`.
4. **Trace mapping (bắt buộc)** — map yêu cầu ↔ scenario ↔ SC.
5. **DOC_GAP (bắt buộc)** — ghi chỗ tài liệu thiếu/mơ hồ chặn test (blocker 🔴 xử trước ở `/qc-plan`).
6. **Guard — BR-tag** *(mới)* — phép **đếm cơ học** chống bỏ sót business rule:

   ```
   A = mọi business rule mà .feature ĐÃ GẮN TAG (@trace.business_rules)
   B = mọi rule bản phân tích VỪA SINH RA (REQUIREMENT_ANALYSIS.md)
   A ∖ B  → rule BDD đã nhắc mà phân tích BỎ SÓT
   ```

   `A ∖ B` rỗng → in `Guard BR-tag: khớp {n}/{n}`. Không rỗng → **tự bổ sung từ PRD** rồi in danh sách đã bù. **In dòng này kể cả khi sạch** — guard im lặng khi sạch là guard không ai biết nó tồn tại.
7. **Self-Review** — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
8. **Output** REQUIREMENT_ANALYSIS.md + DOC_GAP.md.
> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard — chính file `self-review-principles.md` ghi rõ ranh giới đó ngay ở đầu.


---

## Checkpoint & Gate

- **CHECKPOINT: chặn thường** — lệnh ghi 2 file artifact. `--yes` bỏ qua được.
- Guard mềm (BDD approved) — cảnh báo, không chặn cứng; `--yes` coi như Y nhưng **vẫn in cảnh báo**.
- **Platform Resolution chạy TRƯỚC Guard** (sửa 2026-08-25): UC đa nền có nhiều `.feature`, mỗi bản
  mang `@trace.status` riêng — kiểm duyệt trước khi biết platform là kiểm nhầm bản.

---

## Cơ chế đặc biệt

- **QC skill từ repo riêng** (`qc_skills_dir`) — QC team sở hữu, không bị framework upgrade ghi đè.
- **Platform khoá một lần** cho cả pass → nhất quán sổ trace.
- **DOC_GAP** — kênh QC phản hồi ngược chất lượng tài liệu (song song `/report-bug`).

---

## 👓 Góc nhìn tối ưu

- **DOC_GAP trùng vai với review-context B-check?** — cả hai bắt gap tài liệu, nhưng QC nhìn từ góc "test được không". Đáng xem có nối được feedback này về spec.
- **Platform một pass** — feature multi-platform phải chạy QC pass nhiều lần. Chi phí lặp phân tích.
- **Phụ thuộc QC skill ngoài** — nếu `qc_skills_dir` trỏ sai/thiếu, trạm hụt logic.

---

## Kết nối

**Trước:** code + [`/validate-traces`](21-validate-traces.md) · **Sau:** xử gap blocker 🔴 → [`/qc-plan`](16-qc-plan.md).
