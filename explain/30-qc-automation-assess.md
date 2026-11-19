[← /qc-review-testcase · /qc-review-script](18-qc-review.md) · [Explain Home](README.md) · [Next: /qc-design-testdata →](31-qc-design-testdata.md)

> *Số file `30` không theo thứ tự trạm — trang viết sau khi dãy 00–29 đã chốt. Theo dây chuyền, đây là **trạm 5**, đứng giữa `/qc-review-testcase` (trạm 4) và `/qc-design-script` (trạm 6).*

# 30 · `/qc-automation-assess` — Trạm 5: cái nào máy chạy, cái nào phải tay

> **Một câu.** Với mọi test case đã `APPROVED` của cả PRD, quyết `Automatable: Y/N` kèm **nhãn lý do chuẩn hoá**, ghi vào `AUTOMATION_ASSESSMENT.md` — **nguồn duy nhất** mà ba trạm sau đọc để biết sinh script cho TC nào, chạy file nào, và hỏi chạy tay TC nào.

---

## Vấn đề giải quyết

Trước trạm này, framework **ngầm định mọi TC đều automate được**: nhận `.Test.md` rồi sinh script cho tất cả. TC không automate được (OTP qua SMS, captcha, in ra máy in thật, màn hình redesign hàng tuần) vẫn bị sinh script rồi `skip("lý do")`. Ba hệ quả:

| # | Hệ quả | Ví dụ |
|:-:|---|---|
| 1 | Quyết định không được ghi lại | Sáu tháng sau không ai biết TC skip vì *không automate được* hay vì *quên* |
| 2 | Không có số liệu | *"Automation phủ bao nhiêu %?"* không trả lời được — mẫu số lẫn cả TC không bao giờ định automate |
| 3 | TC không automate được **biến mất** | Nằm trong file TC, có dòng `skip`, và **không ai chạy tay nó** |

Điểm 3 là nửa còn lại của `/qc-run-manualtest`: lệnh đó biết *cách* ghi kết quả chạy tay, trạm này cho nó biết **chạy tay những gì**.

---

## Vị trí & tiền đề

- **Vị trí:** trạm 5/9 — sau `/qc-review-testcase`, trước `/qc-design-script`.
- **Target là `{TICKET-ID}` (cả PRD)**, khác `/qc-review-testcase` chạy theo UC-ID. `active_platform` + `qc_artifact_dir` do `steps/qc-scope.md` phân giải.
- **Tiền điều kiện — chỉ nhận TC đã duyệt.** Đọc **mọi** `{qc_artifact_dir}test-cases/REVIEW_<FEATURE>.md` (quét cả thư mục, không đoán tên file — một PRD có thể có `REVIEW_LOGIN.md` + `REVIEW_LOGIN_API.md`):

| Verdict đọc được | Xử lý |
|---|---|
| `APPROVED` · `APPROVED_WITH_SUGGESTIONS` | đưa vào lượt đánh giá |
| `REVISION_REQUIRED` · `REJECTED` · không có file | **loại khỏi lượt**, và liệt kê trong report kèm trạng thái thật |

> Biên bản soát khoá theo `<FEATURE>`, **không có cột UC** — nên lặp *"với mỗi UC"* là đọc cùng một file N lần.

---

## Input / Output

**Input:** các `REVIEW_<FEATURE>.md` (lọc TC) · `test-cases/*.Test.md` · sổ trace `{UC-ID}-{active_platform}.tsv` (cột `status`, để bắt `DRIFT`) · bản `AUTOMATION_ASSESSMENT.md` cũ nếu có · skill `qa-automation-assess/matrix.md`.

**Output:** `{qc_artifact_dir}AUTOMATION_ASSESSMENT.md` — cấp PRD, cùng chỗ `TEST_PLAN.md`:

| TC ID | UC | Automatable | Lý do (nếu N) | Trace SC | Script file | Ghi chú |
|---|---|:---:|---|---|---|---|
| `TC_LOGIN_001` | UC1 | Y | — | UC1-SC1 | `—` *(chưa qua qc-design-script)* | |
| `TC_LOGIN_004` | UC1 | N | `Cần OTP/captcha thủ công` | UC1-SC4 | `—` *(không automate)* | Giữ trong bộ manual |

Cột `Script file` **luôn khởi tạo `—`** — script chưa tồn tại. Kèm một dòng KPI append vào `{paths.trace_dir}/quality-metrics.jsonl` (`"phase":"qc-automation-assess"`).

---

## Các bước xử lý (chi tiết)

| Step | Việc |
|---|---|
| Gate + Phạm vi QC | `steps/gate.md` → `steps/qc-scope.md` chốt `TICKET-ID`, nền, `qc_artifact_dir`, `uc_list` |
| Stamp | `steps/qc-stamp.md` — so dấu phiên bản của `.Test.md`; lệch ⇒ **⚠️ cảnh báo, đi tiếp** |
| Lọc TC | đọc mọi `REVIEW_<FEATURE>.md`, giữ TC đã duyệt, ghi lại phần bị loại |
| Guard `DRIFT` | TC đã có dòng từ lượt trước mà scenario nó verify đang `DRIFT` ⇒ gắn `🔄 Re-assess`, đánh giá lại **ngay lượt này** |
| Đánh giá | `matrix.md`: 6 câu hỏi **tuần tự, dừng ở câu đầu tiên trả lời "Có"** — (1) cần input người/bên thứ ba · (2) phụ thuộc hệ thống ngoài · (3) UI đang đổi liên tục · (4) thiếu test-id và không có cam kết · (5) ROI âm · (6) còn lại ⇒ `Y` |
| Ghi file | đọc file cũ **trước**, merge theo bảng ở *Cơ chế đặc biệt* |
| Self-Review | 3 nhóm: *bịa* (mỗi `N` có nhãn chuẩn, không phải *"lý do kỹ thuật"*) · *nhảy bước* (đủ 6 tiêu chí theo thứ tự) · *số liệu* (`%Automated` là phép chia thật, mẫu số không gồm TC bị loại) |
| Report + KPI | in report, rồi append **đúng một** dòng KPI; lệnh dừng giữa chừng thì **không ghi dòng nào** |

---

## Checkpoint & Gate

- **CHECKPOINT: chặn CỨNG** — ghi đè `AUTOMATION_ASSESSMENT.md` đã có là mất quyết định `Y/N` + nhãn lý do đã chốt, và **xoá cột `Script file`** do `/qc-design-script` điền. **`--yes` KHÔNG bỏ qua được.**
- Mức cứng **chỉ áp khi file đã tồn tại**. Lần đánh giá đầu không ghi đè gì, đi thẳng.
- **Vì sao cứng mà `/qc-review-testcase` thì không:** biên bản soát là kết quả của một lượt đọc — soát lại là đúng bản chất. Bảng này là **quyết định tích luỹ**, và một nửa của nó do lệnh khác điền.

---

## Cơ chế đặc biệt

- **Chạy lại = merge, không phải sinh lại.** Chỉ hàng TC + cột `Trace SC` là sinh lại được:

| Tình huống | Xử lý |
|---|---|
| TC cũ, scenario không `DRIFT` | **giữ nguyên** `Automatable` · nhãn · `Script file` · ghi chú |
| TC cũ, scenario `DRIFT` | `🔄 Re-assess`, đánh giá lại từ đầu 6 tiêu chí |
| `Y → N` sau re-assess | **giữ** path cũ + ghi chú `⚠️ Script đã lỗi thời — cân nhắc gỡ ở lần /qc-design-script kế tiếp`. **Không tự xoá file** |
| `N → Y` sau re-assess | cột vẫn `—`, chờ `/qc-design-script` |
| TC mới | đánh giá mới, `Script file` = `—` |
| TC cũ không còn trong `.Test.md` | **không xoá hàng** — ghi chú `⚠️ TC không còn trong .Test.md — kiểm xem đã bị gỡ có chủ ý chưa` |

- **`Automatable` sống ở đúng một chỗ.** Không chép vào `.Test.md`; `tc-metadata-format.md` chỉ **trỏ sang** file này. Hai nơi ghi cùng một quyết định thì một nơi sẽ lệch.
- **Ba trạm tiêu thụ:** `/qc-design-script` lọc `Y` rồi điền `Script file` · `/qc-run-script` **đọc** `Script file` để biết chạy file nào, không tự suy path · `/qc-run-manualtest` lọc `N`. Thiếu file này thì cả ba đứt; mắt xích nguy nhất là `/qc-run-script` — suy sai đường dẫn thì **chạy 0 test mà vẫn báo xanh** (lỗi ngầm), rồi con số đó đi thẳng vào `/qc-report`.
- **Cảnh báo `P0` + `N`** — tính năng lõi không có smoke test tự động; report gợi ý mở `IMPROVE-xxx` để gỡ rào cản thay vì chấp nhận `N` vĩnh viễn.
- **Nhãn chuẩn để đếm được.** *"3 TC N vì thiếu test-id"* là việc đi đàm phán với dev; *"3 TC vì lý do kỹ thuật"* thì không. Nhãn `Thiếu test-id contract` còn là phép đo cho hợp đồng test-id §4.5.6 — làm đúng thì nó phải giảm dần. Không khớp 5 nhãn chuẩn ⇒ tạo nhãn mới và liệt kê trong report.

---

## 👓 Góc nhìn tối ưu

- **Điểm mạnh:** tách *"không automate"* khỏi *"quên automate"* và đưa phần manual lên bề mặt — đây là thứ biến `%Automated` thành số tin được, chứ không chỉ thêm một bảng.
- **`Next` chọn theo `TEST_DATA_PLAN.md`** *(sửa 2026-09-25 — trước đó trỏ thẳng trạm 6, nên người làm theo gặp một lần dừng)*: chưa có file, hoặc còn TC `Y` chưa có dòng dữ liệu → `/qc-design-testdata` rồi mới `/qc-design-script`; đủ dòng → thẳng trạm 6. Ca "còn TC thiếu dòng" hay gặp khi chạy lại trạm này làm thêm TC `Y`.
- **Quyết định `Y/N` là của người, nhưng lệnh đề xuất trước.** Tiêu chí 3 (*"UI đổi trong < 2 tuần"*) và 5 (ROI) là phán đoán; nếu không ai chốt lại thì `Y/N` của máy thành quyết định mặc nhiên. `reassess_count` trong sổ KPI là chỗ phát hiện tiêu chí đang không ổn định.
- **Chặn cứng không có đường lùi tự động.** Merge giữ được phần người chốt, nhưng muốn đánh giá lại TC mới thì vẫn phải qua cổng cứng mỗi lần — chạy headless (`--yes`) sẽ không đi qua được.

---

## Kết nối

**Trước:** [`/qc-review-testcase`](18-qc-review.md) (TC `APPROVED`) · **Sau:** TC `Y` → [`/qc-design-testdata`](31-qc-design-testdata.md) rồi [`/qc-design-script`](19-qc-run-test.md) · TC `N` → `/qc-run-manualtest {UC-ID}` · cuối cùng [`/qc-report`](20-qc-report.md) đọc bảng này làm một trong bốn nguồn.
