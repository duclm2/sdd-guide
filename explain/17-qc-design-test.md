[← /qc-plan](16-qc-plan.md) · [Explain Home](README.md) · [Next: hai cổng review →](18-qc-review.md)

# 17 · `/qc-design-test` — Trạm 3: Thiết kế test case (Markdown)

> **Một câu.** Thiết kế **test case dạng Markdown** (`.Test.md`) từ plan — chưa sinh script; script đến sau ở `/qc-design-script`.

---

## Vấn đề giải quyết

Tách "thiết kế test case" (con người đọc/review được) khỏi "code test" (máy chạy). `.Test.md` là bản thiết kế mà `/qc-review-testcase` duyệt và `/qc-design-script` biến thành script.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 3), sau `/qc-plan`.
- **Không cần code.** Trạm này chỉ cần spec + **hợp đồng test-id §4.5.6** (đã chốt ở `/map-testids`, trước `/generate-code`). Nên nó chạy **song song với FE**.

---

## Input / Output

**Cú pháp:** `/qc-design-test {UC-ID} [--api | --all] [--atomic-max=explode|keep-completeness] [--shape=api|job|queue]` — target là **UC-ID** (một trong ba lệnh QC nhận UC-ID).

**Input:** `REQUIREMENT_ANALYSIS.md` + `DOC_GAP.md` + `TEST_PLAN.md` (cả ba cấp PRD, **lọc theo cột `UC`**) + skill `qa-designer` (luật `shared/*` luôn nạp, rồi chọn layer, nạp MỘT file) + `.feature` (nguồn scenario cho Guard) + **§4.5.6 Test Selectors** và `@trace.testid_attr` từ tech-doc gộp.

**Output:** `{qc_dir}/{TICKET-ID}-{prd-slug}/{platform}/test-cases/TC_<FEATURE>.Test.md` (không cờ — nhóm GUI · Validation · Functional · Integration qua UI · NFR · E2E) và/hoặc `TC_<FEATURE>_API.Test.md` (`--api`; `--all` = cả hai file). Trace mapping bắt buộc. Gọi theo **từng UC**, ghi vào thư mục test-case dùng chung cấp PRD. Đuôi bắt buộc `.Test.md` — sai đuôi thì không trạm nào tìm thấy mà không có gì báo lỗi.

---

## Các bước xử lý (chi tiết)

0. **Guard — tiền đề** — thiếu bất kỳ file nào trong `REQUIREMENT_ANALYSIS.md` / `DOC_GAP.md` / `TEST_PLAN.md` → **DỪNG** (không hỏi Y/N, `--yes` không qua được). Đủ → in một dòng bằng chứng `✅ Đầu vào trạm 3: 3/3 — …` (cổng đạt mà im lặng thì không phân biệt được với cổng chưa chạy).
1. **Role qa-designer** — chọn layer, nạp một file skill (không nạp cả bộ → tiết kiệm context). Nền `system` mà **0 dấu hiệu endpoint** (cả BDD lẫn tech-doc) → in bằng chứng rồi hỏi hình dạng: API / Job chạy ngầm / Consumer hàng đợi; ghi câu trả lời vào `Tags` (`<lane>` = `api`·`job`·`queue`) của mọi TC. `--yes` mà không có `--shape` → DỪNG.
2. **Conventions** — theo quy ước test-case của QC team (`shared/tc-metadata-format.md`); step GUI trích **test-id** từ §4.5.6; ghi dòng `Test-ID attribute: {attr}` ở metadata (thiếu `@trace.testid_attr` → ghi rõ là thiếu, không tự đoán).
3. **Trace mapping (bắt buộc)** — mỗi test case ↔ SC.
4. **Guard — SC coverage** *(mới)* — phép **đếm cơ học**, không phải lượt đọc lại:

   ```
   1. Thu SC — mọi @trace.scenario trong .feature của UC ĐANG CHẠY (đúng nền)
               — KHÔNG phải uc_list (trạm này per-UC; mẫu số phải khớp phạm vi lệnh).
   2. Thu V  — mọi @trace.verifies trong MỌI *.Test.md dưới test-cases/ (không chỉ
               file vừa ghi), lọc theo tiền tố "{UC-ID}-SC" (có "-SC" để UC1 không khớp UC11).
   3. Đếm    — mỗi SC ∈ tập bước 1 phải có ≥ 1 TC trỏ tới.
   ```

   | Kết quả | Hành vi |
   |---|---|
   | Mọi SC đều có ≥1 TC | in `Guard SC coverage: khớp {K}/{K}` |
   | Có SC chưa phủ | **viết bù TC ngay** — vào đúng file theo câu hỏi *"verify được mà không cần UI không?"* — rồi in `⚠️ Guard SC coverage: {K}/{total} — đã bù {m} SC: {…}`. Đang chạy `--api` mà SC cần UI → **không** bù vào file API, SC vẫn tính chưa phủ, in dòng chỉ đường chạy lại không cờ |

   **In dòng này kể cả khi sạch** — guard im lặng khi sạch là guard không ai biết nó tồn tại, nên cũng không ai phát hiện khi nó hỏng.

5. **Guard — vòng đời `🚫 Block`** — so `GAP-ID` trong các dòng `🚫 Block:` với `Trạng thái` ở `DOC_GAP.md`: `Answered` (có câu trả lời) / `Resolved` · `Out of Scope` (có trích nguồn) → **gỡ dấu + sửa `Expected Result`**; `Re-scoped → Covered` → chuyển dấu sang gap gốc; `Stale` hoặc thiếu câu trả lời/trích nguồn → **không gỡ**, in ⚠️. In dòng `Guard 🚫 Block:` kể cả khi sạch.
6. **UC đang `Blocked`** (TEST_PLAN §5) — **không chặn**, chỉ in thêm dòng ⚠️ ở report.
7. **Self-Review** — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
8. **Output** `.Test.md`; append một dòng `"phase":"qc-design-test"` vào `quality-metrics.jsonl`.

`--atomic-max` *(tuỳ chọn)* — nổ danh sách completeness thành từng TC; phải nêu ranh giới `explode` hay `keep-completeness` (không nêu → hỏi; `--yes` → DỪNG), dựng bản đồ tách trước khi ghi, verify bằng diff.

---

## Checkpoint & Gate

- **CHECKPOINT: chặn CỨNG khi `.Test.md` đã tồn tại** — ghi đè là mất `Status` đã duyệt, `Expected Result` đã sửa theo finding, dấu `🚫 Block` vừa gỡ, và đánh số lại `TC_<FEATURE>_NNN` mà `REVIEW_<FEATURE>.md` đang trỏ vào. `--yes` **không** bỏ qua được. Lần thiết kế đầu → đi thẳng.
- **Chạy lại = GIỮ:** TC cũ giữ số + `Status`; spec đổi thì cập nhật nội dung kèm ⚠️; TC mới cấp số tiếp theo; TC không còn ứng SC → `Status: Obsolete` (không xoá). Thiết kế lại từ trắng phải thêm `--force`.
- Cổng HITL duyệt nội dung TC là `/qc-review-testcase` kế tiếp.
- Nhưng Guard SC coverage **có hệ quả bắt buộc**: thiếu thì lệnh **viết bù** rồi mới báo, không phải chỉ cảnh báo.

---

## Cơ chế đặc biệt

- **Markdown-first** — test case là tài liệu review được trước khi thành code.
- **Nạp MỘT file skill theo layer** — tối ưu context (chỉ layer cần).
- **Trace mapping** — giữ test case gắn SC để về sau ghi `qc_status` đúng.
- **Guard SC coverage KHÔNG có đường thoát.** SC bị gap chặn **vẫn** phải có TC — viết đủ, mang dấu `🚫 Block: [GAP-UC{N}-{nnn}] — lý do`. Gap là thứ được **ghi vào** test case, không phải cái cớ để không viết. Cho phép "bỏ trống có lý do" là dạy hai luật trái nhau trong cùng một lệnh, và cho agent một câu-lý-do-cho-qua.
- **Guard ≠ Self-Review.** Guard là phép **đếm**, có hệ quả bắt buộc. Self-review là lượt đọc lại rộng hơn nhưng mềm hơn. Không bao giờ dùng self-review làm lý do gỡ Guard.

---

## 👓 Góc nhìn tối ưu

- **Tách design ↔ script** thêm một trạm nhưng cho phép review sớm — đánh đổi giữa số bước và chất lượng gate.
- **Phụ thuộc convention QC team** — cần đồng bộ khi convention đổi.
- **Guard đếm sớm, ở chỗ còn sửa được.** `/qc-report` cũng tính design coverage bằng **đúng phép đếm này**, nhưng ở **cuối** — lúc đó chỉ còn kịp báo, không còn kịp viết bù. Cùng một phép đếm, giá trị khác hẳn theo thời điểm.
- Con số `{K}/{total}` vốn **đã in** ở report từ trước, nhưng **không có phép tính nào đứng sau nó**. Một con số trong báo cáo không chứng minh có phép đo.

---

## Kết nối

**Trước:** [`/qc-plan`](16-qc-plan.md) · **Sau:** [`/qc-review-testcase`](18-qc-review.md) (review test case).
