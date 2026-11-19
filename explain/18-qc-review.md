[← /qc-design-test](17-qc-design-test.md) · [Explain Home](README.md) · [Next: /qc-automation-assess →](30-qc-automation-assess.md)

# 18 · `/qc-review-testcase` + `/qc-review-script` — Trạm 4 và 7: hai cổng review

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
| `/qc-review-testcase {TICKET-ID}` (trạm 4) | `/qc-design-test` | `.Test.md` | `REVIEW_<FEATURE>.md` (và `REVIEW_<FEATURE>_API.md` cho file API) |
| `/qc-review-script {TICKET-ID}` (trạm 7) | `/qc-design-script` (trạm 6) | code test + Page Object / API Object (đọc `.Test.md` chỉ để đối chiếu 1-1) | `REVIEW_SCRIPT_<FEATURE>.md` |

Cả hai nhận **TICKET-ID** (UC-ID / file feature cũng quy về TICKET-ID) — chạy cấp PRD, lọc TC theo UC trong phạm vi (`qc-scope` §4c).

**File riêng, không chung.** Hai lượt soát cách nhau vài trạm; chung file thì lượt sau đè bảng chi tiết của lượt trước, và trạm tiêu thụ verdict lại phải đoán — tức mang nguyên vấn đề cũ sang chỗ mới.

---

## Input / Output

**Input:** `.Test.md` *(vai test case)* hoặc code test + Page Object *(vai script)*, cộng skill `qa-reviewer`.

**Output:** file biên bản riêng của mỗi vai, chứa điểm `XX/100` và **một dòng verdict máy đọc được**:

```
**Verdict:** APPROVED
**Verdict:** APPROVED_WITH_SUGGESTIONS
**Verdict:** REVISION_REQUIRED
**Verdict:** REJECTED
```

**Bốn mức, và chúng suy từ SỐ ĐẾM SEVERITY — không từ điểm** *(quyết định `S-VERDICT`, lấy từ `AGT-006` §3.2–3.3)*:

| Đếm được | Verdict |
|---|---|
| ≥1 `BLOCKER` | `REJECTED` |
| ≥1 `MAJOR` | `REVISION_REQUIRED` |
| chỉ `MINOR` / `SUGGESTION` | `APPROVED_WITH_SUGGESTIONS` |
| sạch | `APPROVED` |

Điểm `XX/100` **vẫn ghi** — nhưng để **so hai vòng soát với nhau**, không để quyết verdict. Một con điểm gộp mọi thứ lại thành một số thì **`80` của hai lần soát khác nhau không cùng nghĩa**; số đếm severity thì có.

Đây là **contract**, không phải định dạng cho đẹp — trạm sau đi tiếp khi **≥ `APPROVED_WITH_SUGGESTIONS`** *(`AGT-005:132`)*: `/qc-automation-assess` loại TC chưa đạt ngưỡng, `/qc-run-script` dừng khi script chưa đạt.

> **Lane DEV giữ 2 mức** *(`/review-code` · `/review-context` · `/review-tech-docs` — `APPROVED`/`NEEDS_FIX`)*. Bất đối xứng **có lý do**: lane QC có bảng severity 4 ô đếm được nên 4 verdict chỉ là **tên của bốn ô đó**; lane DEV chưa có bảng đó, ở đó 4 tên sẽ là bốn nhãn phải tự đoán.

---

## Các bước xử lý

1. **Role qa-reviewer** — nạp `{qc_skills_dir}/qa-reviewer/`, **đúng bộ của vai mình**. Vai test case: `shared/review-check-groups.md` (7 nhóm kiểm tra) + một file `test-case/{functional,api,e2e,integration,non-functional,exploratory}.md`. Vai script: `script/_shared/review-rules.md` (R01–R07) + một file **theo nền** `script/web/*` · `script/mobile/*` · `script/system/*` (nền phân giải ở `qc-scope` §2b). Không nạp chéo bộ của vai kia.
2. **Review focus** — test case: đủ phủ SC, expected cụ thể, trace, `🚫 Block` còn hiệu lực (gap đã đóng mà TC vẫn mang dấu ⇒ finding), hợp đồng test-id chân thứ tư. Script: khớp `.Test.md` 1-1, Page Object chỉ locator + thao tác, **assertion ở spec** (helper `expect…` trong PO chỉ khi lặp ≥2 test), `expect()` thật, **locator theo thứ tự của nền** (web `getByTestId` từ §4.5.6 trước · app `~accessibilityId` → `id` → `xpath` · api không có locator), không hard-code, không chờ cứng (`waitForTimeout` · `browser.pause`).
3. **Self-Review** — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
4. **Verdict** — **≥ `APPROVED_WITH_SUGGESTIONS`** → đi tiếp; `REVISION_REQUIRED` hoặc `REJECTED` → sửa rồi review lại. Append một dòng vào `quality-metrics.jsonl` (`"phase":"qc-review-testcase"` / `"qc-review-script"`).

> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard.

---

## Checkpoint & Gate

- Cả hai lệnh ở mức **không chặn** — chúng chỉ ghi **biên bản**, và sinh lại = soát lại.
- Cổng thật nằm ở **verdict chúng phát ra**, được trạm sau đọc.

---

## Cơ chế đặc biệt

- **Chân thứ tư của hợp đồng test-id** — phép so §4.5.6 ↔ `*.Test.md`, mức `warn`, một chiều. Vai này thuộc **`/qc-review-testcase`** *(`bin/trace-schema.json` → `testid_fourth_leg.checked_by`)*, vì phép so chạy trên `.Test.md`. Ở trạm script, locator dựng **lúc chạy** từ bảng tươi nên nó tự cứu.
- **Đọc stamp phiên bản nguồn** — cũng thuộc vai test case *(`qc_artifact_stamp.checked_by`)*.
- **Điểm có luật chấm** — trừ 5đ mỗi `FAIL`, 2đ mỗi `WARN`, mốc chất lượng 80; bảng Tổng quan **thêm một hàng mỗi vòng**, không ghi đè, ô `Điểm · Mức` ghi cả số đếm severity (vd `84 · 0·0·0·8`). So hai vòng thì đọc **`Mức`**, không đọc điểm.
- **Cảnh báo `⚠️ ĐỤNG ĐỘ`** — điểm ≥ 80 mà vẫn có `BLOCKER`/`MAJOR` → in ngay trên dòng `Verdict`: verdict thắng, điểm cao không mở cửa.
- **Reviewer KHÔNG tự sửa** — chỉ nhận xét và chấm. *Tự sửa rồi tự duyệt là bỏ mất cái cổng.*

---

## 👓 Góc nhìn tối ưu

- **Verdict vẫn do LLM phát ra** — có luật chấm rõ, nhưng không có máy nào kiểm lại. Lớp CI cho nó là `T19` *(chân thứ tư)*; phần điểm số thì chưa.
- **Bộ tiêu chí `script/*` đã tách theo nền** (`web` · `mobile` · `system`) — ba nền soát ba thứ khác nhau (thứ tự locator web không tồn tại ở Appium; lane `api` không có locator lẫn wait). Nạp nhầm lane là đi tìm thứ không tồn tại rồi kết luận "không có lỗi" — chi phí rủi ro nằm ở bước phân giải nền.

---

## Kết nối

**Trước:** [`/qc-design-test`](17-qc-design-test.md) *(vai test case)* · [`/qc-design-script`](19-qc-run-test.md) *(vai script)*
**Sau:** test case ≥ `APPROVED_WITH_SUGGESTIONS` → [`/qc-automation-assess {TICKET-ID}`](30-qc-automation-assess.md) rồi [`/qc-design-script`](19-qc-run-test.md) · script ≥ `APPROVED_WITH_SUGGESTIONS` → [`/qc-run-script {UC-ID}`](19-qc-run-test.md) (chạy thật) → [`/qc-report {TICKET-ID}`](20-qc-report.md)
