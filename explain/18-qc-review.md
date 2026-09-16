[← /qc-design-test](17-qc-design-test.md) · [Explain Home](README.md) · [Next: ba trạm script →](19-qc-run-test.md)

# 18 · `/qc-review-testcase` + `/qc-review-script` — Trạm 4: Hai cổng review

> **Một câu.** Gate chất lượng của QC, **hai lệnh riêng**: `/qc-review-testcase` soát test case sau design-test; `/qc-review-script` soát code test sau khi script được sinh.

> **Trước Đợt 2 đây là MỘT lệnh `/qc-review` làm cả hai vai và tự đoán vai nào bằng cách so ngày sửa file.** Chính trang này từng ghi ở mục *Góc nhìn tối ưu*: *"Chạy 2 lần cùng một lệnh — người dùng phải nhớ gọi đúng thời điểm… **Cân nhắc tách rõ**"*. Đó là việc đã làm.

---

## Vấn đề giải quyết

Không chạy test kém. Hai cổng chặn hai thứ khác nhau:

1. **`/qc-review-testcase`** — test case chưa đủ tốt trước khi biến thành script.
2. **`/qc-review-script`** — script chưa đúng trước khi coi kết quả chạy là chính thức.

Và **cái mà một-lệnh-hai-vai không làm được:** đặt điều kiện tiên quyết cho trạm sau. `REVIEW_<FEATURE>.md` của lệnh cũ không nói nó là kết quả soát vai nào, nên trạm sau không hỏi được *"cái tôi cần đã APPROVED chưa?"*.

---

## Vị trí & tiền đề

| Lệnh | Chạy sau | Đọc gì | Ghi gì |
|---|---|---|---|
| `/qc-review-testcase` | `/qc-design-test` | `.Test.md` | `REVIEW_<FEATURE>.md` |
| `/qc-review-script` | trạm sinh script | code test + Page Object | `REVIEW_SCRIPT_<FEATURE>.md` |

**File riêng, không chung.** Hai lượt soát cách nhau vài trạm; chung file thì lượt sau đè bảng chi tiết của lượt trước, và trạm tiêu thụ verdict lại phải đoán — tức mang nguyên vấn đề cũ sang chỗ mới.

---

## Input / Output

**Input:** `.Test.md` *(vai test case)* hoặc code test + Page Object *(vai script)*, cộng skill `qa-reviewer`.

**Output:** file biên bản riêng của mỗi vai, chứa điểm `XX/100` và **một dòng verdict máy đọc được**:

```
**Verdict:** APPROVED
**Verdict:** NEEDS_FIX
```

`APPROVED` khi điểm `≥80` **và** không còn `FAIL` chặn. Đây là **contract**, không phải định dạng cho đẹp — `/qc-automation-assess` loại TC chưa `APPROVED`, `/qc-run-script` dừng khi script chưa `APPROVED`.

---

## Các bước xử lý

1. **Role qa-reviewer** — nạp `{qc_skills_dir}/qa-reviewer/`, **đúng bộ của vai mình**.
2. **Review focus** — test case: đủ phủ SC, expected cụ thể, trace, `🚫 Block` còn hiệu lực. Script: khớp `.Test.md` 1-1, Page Object gọn, `expect()` thật, **selector bám §4.5.6 chứ không dò DOM**, không hard-code.
3. **Self-Review** — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
4. **Verdict** — `APPROVED` → đi tiếp; `NEEDS_FIX` → sửa rồi review lại.

> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard.

---

## Checkpoint & Gate

- Cả hai lệnh ở mức **không chặn** — chúng chỉ ghi **biên bản**, và sinh lại = soát lại.
- Cổng thật nằm ở **verdict chúng phát ra**, được trạm sau đọc.

---

## Cơ chế đặc biệt

- **Chân thứ tư của hợp đồng test-id** — phép so §4.5.6 ↔ `*.Test.md`, mức `warn`, một chiều. Vai này thuộc **`/qc-review-testcase`** *(`bin/trace-schema.json` → `testid_fourth_leg.checked_by`)*, vì phép so chạy trên `.Test.md`. Ở trạm script, locator dựng **lúc chạy** từ bảng tươi nên nó tự cứu.
- **Đọc stamp phiên bản nguồn** — cũng thuộc vai test case *(`qc_artifact_stamp.checked_by`)*.
- **Điểm có luật chấm** — trừ 5đ mỗi `FAIL`, 2đ mỗi `WARN`; bảng Tổng quan **thêm một hàng mỗi vòng**, không ghi đè. Đó là cách duy nhất thấy được sửa xong có tốt lên không.
- **Reviewer KHÔNG tự sửa** — chỉ nhận xét và chấm. *Tự sửa rồi tự duyệt là bỏ mất cái cổng.*

---

## 👓 Góc nhìn tối ưu

- **Verdict vẫn do LLM phát ra** — có luật chấm rõ, nhưng không có máy nào kiểm lại. Lớp CI cho nó là `T19` *(chân thứ tư)*; phần điểm số thì chưa.
- **Bộ tiêu chí `script/*` còn phẳng theo tầng.** Khi Đợt 2 bước 2 chốt stack, nó sẽ tách theo nền (`web`/`mobile`) vì tiêu chí khác nhau. Cố ý chưa làm ở bước 1 — viết tiêu chí cho stack chưa chốt là viết hai lần.

---

## Kết nối

**Trước:** [`/qc-design-test`](17-qc-design-test.md) *(vai test case)* · [ba trạm script](19-qc-run-test.md) *(vai script)*
**Sau:** test case `APPROVED` → [ba trạm script](19-qc-run-test.md) · script `APPROVED` → [`/qc-report`](20-qc-report.md) rồi tạo PR
