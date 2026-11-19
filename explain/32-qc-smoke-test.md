[← /qc-report](20-qc-report.md) · [Explain Home](README.md) · [Next: /qc-metrics →](33-qc-metrics.md)

> *Utility **ngoài luồng tuần tự** — không phải trạm nào của dây chuyền QC, chạy bất cứ lúc nào. Số file `32` chỉ là thứ tự trang.*

# 32 · `/qc-smoke-test` — Cổng nhanh Go/No-Go trên tập test lõi

> **Một câu.** Chạy **chỉ** các test mang tag `@smoke` (= TC `Priority: P0`) đã được duyệt, trên một phạm vi co giãn (1 PRD · 1 domain · toàn app), retry 1 lần, rồi in một tín hiệu đi/không-đi — **không sinh script, không ghi `qc_status`**.

---

## Vấn đề giải quyết

Có build mới, câu cần trả lời nhanh là *"tính năng lõi có đúng không"* — chưa phải *"PRD này đã kiểm xong chưa"*. Regression đầy đủ (`/qc-run-script`) trả lời câu sau, nhưng chậm và gắn với từng PRD. Lệnh này trả lời câu trước bằng tập P0.

| | `/dev-smoke-test` | `/qc-smoke-test` |
|---|---|---|
| Của ai | dev | QC |
| Hỏi gì | *"service đang chạy có sống không"* (health check, vài endpoint) | *"tính năng lõi có đúng không"* |

---

## Vị trí & tiền đề

- **Vị trí:** ngoài dây chuyền `qc-analyze → … → qc-run-script ∥ qc-run-manualtest → qc-report`.
- **Tiền đề:** script đã qua `/qc-design-script` → `/qc-review-script` với verdict `APPROVED` hoặc `APPROVED_WITH_SUGGESTIONS`. Tag `@smoke` do `/qc-design-script` gắn tự động theo `Priority` (`P0 → @smoke + @regression`).
- **Phạm vi tự phân giải — KHÔNG dùng `steps/qc-scope.md`.** Đây là khác biệt có chủ ý: 11 lệnh QC kia cần `TICKET-ID` + `uc_list` khoá chặt; lệnh này hỏi câu cấp app, và cổng `4a`/`4b` của `qc-scope` sẽ DỪNG khi target không phải một PRD/UC cụ thể.

| Gõ | Phân giải thành | Thư mục test |
|---|---|---|
| `{TICKET-ID}` · `{domain}/{prd-slug}` | 1 PRD | `tests/{domain}/{prd-slug}/` |
| `{domain}` *(không khớp PRD nào)* | mọi PRD của domain | `tests/{domain}/` |
| rỗng · `all` | toàn app | `tests/` |

Không phân giải được ⇒ liệt kê domain dưới `{paths.specs_dir}/`, hỏi người chọn — **đừng đoán**. Nền: `--platform=` nếu có; không có ⇒ chạy **mọi nền** có test `@smoke` trong phạm vi.

---

## Input / Output

**Input:** phạm vi + cờ · script `@smoke` đã duyệt · skill `qa-script-runner/smoke.md` (lệnh chạy từng nền, cách loại `@quarantine`, khuôn đọc kết quả).

**Output:** một **Smoke Test Summary** in ra màn hình, gắn với đúng lần chạy này:

```
🔥 Smoke Test — phạm vi: {scope} ({nền})
Kết quả: ✅ {pass} pass · ❌ {fail} fail · 🔄 {flaky} flaky      (tổng smoke: {N})
         🚧 {q} quarantined ({qo} quá hạn) · 🚫 {d} disabled ({du} không có mã)
❌ {UC-ID}-SC{n} ({TC_id}) — {script-bug | product-gap → BUG-{id} ĐÃ mở | flaky → quarantine}
Self-review: …
Go/No-Go: 🟢 SẠCH — đi tiếp được, chạy regression đầy đủ
        | 🔴 KHÔNG NÊN BÀN GIAO — {n} tính năng lõi đang fail
Lưu ý: lệnh này KHÔNG ghi qc_status. Kết quả chính thức vẫn phải qua /qc-run-script.
```

File duy nhất được ghi: **một dòng KPI** `"phase":"qc-smoke-test"` vào `quality-metrics.jsonl` (append).

---

## Các bước xử lý (chi tiết)

| Step | Việc |
|---|---|
| Phạm vi | tách cờ khỏi `$ARGUMENTS`, phân giải theo bảng trên |
| Tiền điều kiện | phạm vi **0 test `@smoke`** ⇒ **DỪNG**, in khối `CHƯA CÓ ≠ SẠCH` kèm hướng dẫn `/qc-automation-assess → /qc-design-script (P0) → /qc-review-script APPROVED`. **Không tự tạo test tạm** |
| Chạy | theo `smoke.md`: web · system dùng `npx playwright test … --grep @smoke --grep-invert @quarantine`; app dùng WDIO với **một** regex lookahead gộp cả hai điều kiện |
| FAIL | chạy lại **1 lần** (không phải 2), rồi phân loại bằng đúng ba nhãn của `/qc-run-script`: `script-bug` · `product-gap` · `flaky`. Phân loại **luôn chờ người xác nhận, kể cả có `--yes`** |
| `product-gap` | người xác nhận ⇒ gọi `/report-bug {UC-ID} {expected-vs-actual}` **ngay**, không đợi chạy hết phạm vi. Severity đề xuất `Critical`, nhãn `🔥 Smoke-fail` |
| Self-review | *phạm vi* đúng cái người gõ · *số liệu* lấy thẳng từ runner lần này · mọi FAIL đã retry · *tập rỗng* đã in khối "chưa có gì để kiểm". In **trước** dòng Go/No-Go |
| Summary + KPI | in Summary, rồi append một dòng KPI; dừng giữa chừng thì không ghi |

---

## Checkpoint & Gate

- **CHECKPOINT: không chặn** — lệnh chỉ chạy test đã có và in tóm tắt; không sinh file, không sửa script, không ghi cột nào của sổ trace.
- **Ngoại lệ duy nhất `--yes` không bỏ qua:** bước phân loại FAIL — đó là cổng của thao tác, không phải CHECKPOINT của lệnh.
- **Ai quyết khi 🔴:** QC đưa tín hiệu, **Lead/PM quyết**. QC không tự chốt *"vẫn bàn giao"* hay *"hoãn"*.

---

## Cơ chế đặc biệt

- **`0 test chạy · 0 fail` KHÔNG phải Go.** Luật *"Untested ≠ Pass"* áp ở tầng lệnh: tín hiệu xanh dựng trên tập rỗng là lỗi ngầm — nhìn y hệt một lượt chạy sạch, và xuất hiện đúng lúc người ta gấp nhất.
- **Vì sao không ghi `qc_status`:** retry ×1 trên tập P0 là tốc độ mua bằng độ chắc. Nếu ghi, `/qc-report` sẽ tính execution coverage trên dữ liệu rút gọn — báo *"đã kiểm 85%"* trong khi chỉ tập P0 được chạy. Bản đề xuất gốc của đội QC có bước cập nhật `qc_status`; framework **cố ý bỏ** và ghi rõ lý do để người sau không *"thêm lại cho tiện"*.
- **Loại `@quarantine` khỏi smoke:** test đang cách ly là test chưa kết luận — để nó quyết Go/No-Go thì hoặc no-go oan, hoặc go giả.
- **Hai nền loại `@quarantine` hai cách, không hoán đổi được:** Playwright có `--grep` / `--grep-invert` độc lập; Mocha chỉ nhận **một** pattern, nên viết kiểu web cho mobile sẽ hoặc chạy cả test quarantine, hoặc bỏ chính test `@smoke`.
- **Bố cục cũ vẫn phải tìm.** Dự án chưa chuyển sang `tests/{domain}/{prd-slug}/` thì phạm vi domain phân giải **qua spec** (liệt kê PRD, truyền nhiều đường dẫn) và in cảnh báo chuyển — chỉ dò bố cục mới là chạy 0 test ở dự án cũ rồi báo xanh.

---

## 👓 Góc nhìn tối ưu

- **Điểm mạnh:** kỷ luật *"không ghi sổ"* giữ execution coverage của trạm 9 sạch, và luật tập rỗng chặn đúng ca xanh giả nguy hiểm nhất.
- **Skill chạy và lệnh đang nói hai điều về thư mục.** `.tmpl` (cập nhật theo cây 2026-09-24) nói `{domain}` **là** một thư mục test; `qa-script-runner/smoke.md` §1 vẫn ghi *"Không có tầng `tests/{domain}/`"* và bảo phân giải qua spec. Agent đọc skill theo lời dặn của lệnh sẽ gặp chỉ dẫn ngược nhau.
- **Phân loại luôn chờ người** — đúng về chất lượng, nhưng làm lệnh không chạy trọn được trong CI headless khi có FAIL; tín hiệu *"nhanh"* phụ thuộc có người trực.
- **Tất cả dựa trên `Priority` = P0.** Nếu nhãn `Priority` cũ (PRD đổi mà TC chưa chấm lại), tập smoke cũng cũ — lệnh không tự so stamp như các trạm khác vì không đi qua `qc-scope`/`qc-stamp`.

---

## Kết nối

**Trước:** script `@smoke` đã qua [`/qc-review-script`](18-qc-review.md) `APPROVED` · **Sau:** 🟢 → `/qc-run-script {UC-ID}` cho từng UC (regression đầy đủ, xem [ba trạm script](19-qc-run-test.md)) · 🔴 → chờ dev fix bug (đã mở qua [`/report-bug`](25-report-bug.md)), hoặc `/qc-design-script` lại nếu `script-bug` · số liệu flaky của smoke đi vào [`/qc-metrics`](33-qc-metrics.md).
