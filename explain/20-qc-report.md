[← ba trạm script](19-qc-run-test.md) · [Explain Home](README.md) · [Next: /validate-traces →](21-validate-traces.md)

# 20 · `/qc-report` — Trạm 6: Report + evidence + product-gap

> **Một câu.** Trạm cuối QC: tổng hợp **report + evidence**, và đẩy **product-gap** (lỗi sản phẩm, không phải script) ngược về PO/Dev.

---

## Vấn đề giải quyết

Kết quả chạy cần được trình bày có bằng chứng và **định tuyến đúng**: script-bug thì QC tự sửa, còn **product-gap** phải về tay PO/Dev. Trạm này đóng gói kết quả QC pass thành báo cáo.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 6, cuối), sau `/qc-run-script` và `/qc-run-manualtest`.

---

## Input / Output

**Input:** kết quả run + evidence + skill `qa-runner/report`.

**Output:** QC report + evidence; product-gap đẩy về PO/Dev.

---

## Các bước xử lý (chi tiết)

1. **Role qa-runner/report** — nạp skill report.
2. **Procedure** — tổng hợp kết quả run (pass/fail per SC), đính evidence.
3. Tách **product-gap** → định tuyến về PO/Dev (có thể thành `/report-bug`). Scenario mang nhãn **`flaky`** **không** thành bug — nó là *chưa có kết luận*, `qc_status` để `not_run`.
4. **Self-Review** *(mới)* — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
5. **Output** report.
> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard — chính file `self-review-principles.md` ghi rõ ranh giới đó ngay ở đầu.


---

## Checkpoint & Gate

- Không gate chặn — báo cáo + định tuyến.

---

## Cơ chế đặc biệt

- **Product-gap routing** — khép vòng feedback: lỗi sản phẩm từ QC về đúng người quyết.
- **Evidence-first** — báo cáo kèm bằng chứng (khác dev smoke).

---

## 👓 Góc nhìn tối ưu

- **Product-gap → PO/Dev** nhưng cơ chế nối với `/report-bug` không tự động — dễ đứt. Đáng tích hợp để gap thành bug spec-anchored có hồ sơ.
- **Report tách khỏi run** (trạm riêng) — linh hoạt nhưng thêm một lệnh phải nhớ gọi.
- **Sau report → `/validate-traces`** để làm mới Living Docs (`qc_status`) — chuỗi này phụ thuộc người chạy đủ.

---

## Kết nối

**Trước:** [ba trạm script](19-qc-run-test.md) · **Sau:** [`/validate-traces`](21-validate-traces.md) (làm mới Living Docs); product-gap → [`/report-bug`](25-report-bug.md).
