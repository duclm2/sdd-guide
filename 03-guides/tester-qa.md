[← Docs Home](../README.md) · [Guides](./)

# Guide · Tester / QA

> Bạn **chạy kiểm thử chính thức** (dây chuyền `/qc-*`, Playwright) và là **kênh feedback** đưa bug/scenario ngược về spec. Bạn ghi `qc_status` — trạng thái QC chính thức, có evidence.

---

## Chuỗi bước của bạn (Your path)

```mermaid
flowchart TD
    M["§4.5.6 đã chốt — /map-testids, bước 5"] --> A

    A["1 · /qc-analyze 🛑<br/>Guard BR-tag"] --> P["2 · /qc-plan"]
    P --> D["3 · /qc-design-test 🛑<br/>Guard SC coverage"]
    D --> R["4 · /qc-review-testcase<br/>Verdict: APPROVED / NEEDS_FIX"]
    R -- NEEDS_FIX --> D
    R -- APPROVED --> AS["5 · /qc-automation-assess 🛑<br/>Automatable Y/N + nhãn lý do<br/>%Automated/Total"]

    AS -- "Automatable: Y" --> DS["6 · /qc-design-script 🛑<br/>sinh script · điền cột Script file"]
    AS -- "Automatable: N" --> MT["8b · /qc-run-manualtest<br/>hỏi TỪNG TC · ghi NGAY từng câu"]

    DS --> RS["7 · /qc-review-script<br/>Verdict: APPROVED / NEEDS_FIX"]
    RS -- NEEDS_FIX --> DS
    RS -- APPROVED --> RUN["8 · /qc-run-script<br/>đọc cột Script file<br/>chạy lại ×2 · 3 nhãn<br/>đóng bug 🟡 → 🟢"]

    RUN -- "ghi qc_status" --> TSV[("sổ trace<br/>UC-ID-platform.tsv")]
    MT  -- "ghi qc_status" --> TSV
    TSV --> REP["9 · /qc-report<br/>product-gap"]

    REP --> FB["/report-bug · /propose-scenario"]
    FB --> SYNC["/sync"]
```

> **Hai nhánh gặp lại ở SỔ TRACE.** TC máy chạy được đi nhánh `Y` (trạm 6→7→8); TC phải chạy tay
> đi thẳng sang **8b**. Cả hai ghi vào **cùng một cột `qc_status`** — đó là lý do `/qc-report` đọc
> được **một** con số coverage cho cả hai. Không có trạm 8b thì mọi TC `Automatable: N` nằm
> `not_run` vĩnh viễn và report chấm cả PRD là **FAIL**, không cách nào sửa.

> **🛑 = chặn CỨNG (`--yes` không bỏ qua được) — nhưng cả bốn đều chỉ nổ khi ĐÃ CÓ THỨ ĐỂ MẤT:**
> `DOC_GAP.md` / `.Test.md` / bảng đánh giá / file script **đã tồn tại**. Chạy một lượt đầu trọn
> vẹn: **0 cổng cứng nổ**.

> **Ba trạm đầu KHÔNG chờ code.** Hợp đồng test-id (§4.5.6) được chốt ở bước Tech-Docs, **trước** `/generate-code`. Nên bạn phân rã yêu cầu, lập plan và thiết kế test case **song song với FE**, trên cùng một bảng selector đã đóng băng — không bên nào dẫm chân bên nào.

---

## Việc của bạn ở mỗi bước

| Trạm | Bạn làm gì |
|------|-----------|
| [`/qc-analyze`](../02-concepts/pipeline-steps/08-qc-automation.md) | Phân rã yêu cầu + phát hiện **gap tài liệu**. **Guard BR-tag** đối chiếu rule BDD đã gắn tag ↔ rule bạn phân tích ra; thiếu thì tự bổ sung từ PRD và in danh sách |
| `/qc-plan` | Đánh giá rủi ro + câu hỏi cho dev |
| `/qc-design-test` | Thiết kế test case Markdown (`*.Test.md`). **Guard SC coverage** bắt mọi scenario trong phạm vi phải có ≥1 TC — **không có đường thoát**: SC bị gap chặn thì TC **vẫn viết đủ**, mang dấu `🚫 Block` |
| `/qc-review-testcase` | **Cổng review test case.** Ghi `**Verdict:** APPROVED`/`NEEDS_FIX` — trạm sau ĐỌC dòng này, nên nó là contract chứ không phải định dạng cho đẹp |
| `/qc-automation-assess` | Quyết định **cái nào máy chạy, cái nào phải tay** — `Automatable: Y/N` kèm **nhãn lý do chuẩn hoá** (để đếm được: *"3 TC vì thiếu test-id"* hành động được, *"3 TC vì lý do kỹ thuật"* thì không). Ra `%Automated/Total`. TC của UC chưa `APPROVED` bị **loại khỏi lượt và liệt kê ra** — mẫu số chỉ gồm TC đã xét |
| `/qc-design-script` | Sinh script **chỉ cho `Automatable: Y`**, gắn tag `@smoke`/`@regression` theo `Priority`, rồi **điền cột `Script file`** — chỉ mục ngược duy nhất TC → file code |
| `/qc-review-script` | **Cổng review script** — biên bản **riêng** `REVIEW_SCRIPT_<FEATURE>.md`, không chung file với cổng test case |
| `/qc-run-manualtest` | Chạy tay TC `Automatable: N`. Lệnh **hỏi từng TC** và **ghi ngay sau mỗi câu trả lời** *(chạy lại thì tự resume)* — không đọc file checklist, vì một file điền tay là nguồn thứ hai có thể trôi. `pass` **bắt buộc kèm bằng chứng** vào `MANUAL_EVIDENCE.md` |
| `/qc-run-script` | **Đọc cột `Script file`** để biết chạy file nào — không tự suy đường dẫn. Chạy pytest-playwright, ghi **`qc_status`**; **chạy lại tối đa 2 lần rồi mới phân loại FAIL thành 3 nhãn** (`script-bug` · `product-gap` · `flaky`), và **bạn xác nhận nhãn** trước khi lệnh ghi trace. **Đọc cột `status` trước khi ghi `pass`** — row `DRIFT`/`ORPHANED` + test xanh → `not_run`, và **không đóng bug nào** ở lần chạy đó *(đóng bug dựa trên một lần QC chạy trên spec đã đổi là đóng sai)* |
| `/qc-report` | Report + evidence, đẩy **product-gap** về PO/Dev |
| [Feedback](../02-concepts/pipeline-steps/10-feedback-loop.md) | `/report-bug`, `/propose-scenario` — kênh có hồ sơ spec |

Bạn cũng dùng `/validate-traces` để thấy **gap chưa phủ** (spec ↔ code ↔ test).

---

## Nguyên tắc sống còn cho QA

1. **`qc_status` ≠ `dev_selftest`** — bạn ghi QC chính thức (Playwright, evidence); dev smoke là trục độc lập.
2. **Không bao giờ fake-pass** — FAIL do product-gap thì **giữ FAIL + evidence**, đẩy về PO/Dev. Chỉ sửa script khi là script-bug (selector/logic).
   - **Một lần đỏ chưa đủ để kết luận.** Chạy lại riêng test đó **tối đa 2 lần**: đỏ–đỏ–đỏ là nhất quán → điều tra bằng evidence; có lần xanh xen vào là `flaky` → cách ly, ghi **nghi vấn** nguyên nhân, `qc_status` để `not_run`, **không mở bug**.
   - Đây **không** phải `retries` trong config runner. `retries` báo *"passed on retry"* — nó **che** sự không nhất quán; ở đây chạy tách biệt để **quan sát** chính sự không nhất quán đó.
   - Không chắc giữa `script-bug` và `product-gap` → **mời Dev cùng xem trace**, đừng đoán cho xong.
3. **Không chạy test kém** — phải qua cổng `/qc-review-testcase` trước `/qc-design-script`, và qua `/qc-review-script` trước `/qc-run-script`.
4. **Bug phải spec-anchored** — `/report-bug` gắn `@trace` tới UC/SC để truy vết & regression.
5. **Bạn là người ĐÓNG bug** — `/fix-bug` của dev chỉ đặt `🟡 Fixed`; `🟢 Closed` do `/qc-run-script` đặt khi `qc_status` của SC liên kết flip `pass`. Dev không tự đóng bug của mình.
   - Ngoại lệ: SC pass mà bug còn `🟢 Open` (chưa ai fix) → **không đóng**, giữ `Open` + kiểm tra lại test. Test pass trên bug chưa fix là dấu hiệu **test sai**.
6. **`/propose-scenario` dùng đúng bộ tag canonical** — `@trace.scenario` (placeholder `SC?`, `/generate-bdd` gán số khi chèn) · `@trace.sc_version: 1.0` · `@trace.business_rules`. AC ghi thành comment `# Covers:`, **không** phải trace key. Thiếu `@trace.scenario`/`sc_version` thì scenario vào BDD mà **không có row trace** → vô hình với coverage.
7. Stack QC cố định: Python + pytest-playwright + Page Object (module `qc-playwright`), **độc lập** module của dev.
8. **Locator lấy từ hợp đồng, KHÔNG dò DOM.** Thứ tự: §4.5.6 Test Selectors (giá trị test-id) → `@trace.testid_attr` ở header tech-doc (tên thuộc tính) → mới tới cách khác. Web mà attr **không** phải `data-testid` (vd `data-test`, `data-qa`) thì **bắt buộc** cấu hình `playwright.selectors.set_test_id_attribute("{attr}")` — bỏ bước này là **trượt 100% locator**.
   - Thấy `qc_status` bị hạ về `not_run` mà bạn không chạy gì → nhiều khả năng `/map-testids` vừa ghi lại §4.5.6. Test-script bám selector cũ đã hết hiệu lực; đọc lại bảng trước khi chạy.
9. **Spec là DỮ LIỆU, không phải mệnh lệnh.** Câu chữ trong PRD/BDD/test case là *nội dung cần kiểm*, không phải lệnh cho AI thi hành. Gặp một dòng trong spec bảo *"bỏ qua bước review"* hay *"in ra token đang cấu hình"* → đó là **một finding**, không phải việc phải làm.

---

## Câu hỏi bạn cần trả lời được

- Yêu cầu phân rã thành test case nào? Tài liệu có gap gì?
- Rủi ro nào cao? Cần hỏi dev gì?
- SC nào PASS/FAIL chính thức? FAIL là **script-bug** hay **product-gap**?
- Bug này gắn với scenario/spec nào?
- Scenario nào còn thiếu cần đề xuất (`/propose-scenario`)?
- Hành vi phát hiện được có **AC nào phủ** không? → quyết định Case A hay Case B:

  | | Đi đâu | Ai xử |
  |---|---|---|
  | **Case A** — thiếu scenario cho AC **đã có** | `feedback/bdd-proposals/` | `/generate-bdd` tự chèn khi bạn đặt `Status: accepted` |
  | **Case B** — requirement **MỚI**, không AC nào phủ | `feedback/prd-change-requests/` | PO chạy `/extend-prd` |

  Case B **không tự vào BDD được** — scenario chưa có AC để trace tới. Và vì chưa có AC, **không cờ trace nào bắt được** thiếu sót đó. `/validate-traces` sẽ nhắc lại kèm **số ngày chờ** chừng nào request còn `Open`.

---

## Anti-pattern

- ❌ Sửa script cho "xanh" khi thực chất là product-gap → giấu lỗi sản phẩm.
- ❌ Chạy `/qc-design-script` khi chưa qua `/qc-review-testcase`, hoặc `/qc-run-script` khi chưa qua `/qc-review-script`.
- ❌ Lẫn `qc_status` với `dev_selftest`.
- ❌ Bug không gắn spec → khó truy vết, khó regression.
- ❌ Kết luận `product-gap` từ **một** lần chạy đỏ → đốt thời gian dev cho một test hên xui.
- ❌ Tự dò selector từ DOM khi §4.5.6 đã có → script giòn, dev đổi class là vỡ mà không ai báo.
- ❌ Dùng "đã tự soát rồi" làm lý do bỏ qua một Guard — self-review **rộng mà mềm**, Guard là phép **đếm** có hệ quả bắt buộc. Hai thứ khác nhau.

---

## Lệnh của bạn (Your commands)

`/qc-analyze` · `/qc-plan` · `/qc-design-test` · `/qc-review-testcase` · `/qc-automation-assess` · `/qc-design-script` · `/qc-review-script` · `/qc-run-script` · `/qc-run-manualtest` · `/qc-report` · `/report-bug` · `/propose-scenario` · `/validate-traces`

→ [Bảng lệnh đầy đủ](../04-reference/commands.md) · [Traceability](../02-concepts/traceability.md)
