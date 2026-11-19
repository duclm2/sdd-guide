[← Validate Traces](09-validate-traces.md) · [Pipeline Steps](README.md) · [Overview →](../overview.md)

# Bước 10 · Feedback Loop — Vòng phản hồi & tự học (Feedback & Learning)

> **Tóm tắt.** Bug, đề xuất scenario và bài học được đưa vào các **kênh có hồ sơ**, nổi lên qua `/sync`, và nạp lại vào context — **không tạo loop ngầm** mà cải tiến spec & tri thức dự án.
> **Commands:** `/report-bug` · `/propose-scenario` · `/learn` · `/sync` · `/fix-bug`

| | |
|---|---|
| **Giai đoạn** | Cross-cutting (xuyên suốt) |
| **Owner** | 👤 Tester / QA (và tất cả) |
| **Đầu vào** | Bug, gap, định hướng lặp lại |
| **Đầu ra** | `feedback/*`, `project-lessons.md`, Living Docs |
| **HITL** | ⚪ Kênh có hồ sơ |

---

## Mục đích (Purpose)

Framework là pipeline **một chiều** — nhưng vẫn cần đường **phản hồi ngược có kiểm soát** để hệ thống *học* qua thời gian. Nguyên tắc: định hướng/guardrail lặp lại **phải được ghi**, đừng để nằm trong đầu vài senior. Bước này:

- Cho tester/QC một **kênh feedback có hồ sơ spec** (bug, scenario proposal).
- Biến bài học thành **lesson dán tường** (`project-lessons.md`) — nạp lại để không sai lại.
- **Nổi** feedback lên PO/Dev qua `/sync`.

> Đây **không phải loop** để chạy vòng vòng — mỗi phản hồi đi vào spec hoặc tri thức rồi pipeline lại chảy một chiều.

---

## Các kênh (Channels)

| Lệnh | Ai dùng | Kết quả | Đi về đâu |
|------|---------|---------|-----------|
| `/report-bug` | Tester / QC | Bug **spec-anchored** (gồm product-gap từ `/qc-run-script` · `/qc-report`) | `feedback/bug-reports/` |
| `/propose-scenario` **Case A** | Tester / QC | Thiếu scenario cho **AC đã có** | `feedback/bdd-proposals/` |
| `/propose-scenario` **Case B** | Tester / QC | **Requirement MỚI** — không AC nào phủ | `feedback/prd-change-requests/` |
| `/learn` | Tất cả | Guardrail lesson | `project-lessons.md` (qua step `capture-lesson`) |
| `/fix-bug` | Dev | Sửa lỗi có root-cause + regression test | Code + `@trace.fixes/root_cause/regression` |
| `/extend-prd` | PO | **Drain** PRD change request → UC/AC/BR mới trong PRD | PRD v+1 · request → `archived/` |
| `/sync` | Lead (umbrella) | Pull + submodule + **nổi feedback** + **nổi spec delta** (Step 1e) + làm mới Living Docs | Chạy hằng ngày |

### Ba hàng đợi — mỗi cái phải có người lấy ra

| Hàng đợi | Ai bỏ vào | Ai **lấy ra** | Ai **nhắc lại** |
|---|---|---|---|
| `bug-reports/` | `/report-bug` | `/fix-bug` (quét thư mục mỗi lần chạy) | cột `qc_blocked_by` của TSV |
| `bdd-proposals/` | `/propose-scenario` A | `/generate-bdd` (quét mỗi lần chạy, 9 bước: chèn → normalize → append row TSV → archive → commit) | — |
| `prd-change-requests/` | `/propose-scenario` B | **`/extend-prd`** | `/validate-traces` Step 7b — đếm `Status: Open` kèm **số ngày chờ** |

> **Vì sao cột "ai nhắc lại" quan trọng.** `/sync` chỉ hiện những gì về **trong đúng lần pull đó** (`git diff old..new`) — nó là **chuông cửa, không phải tồn kho**. Bỏ lỡ một lần là mất khỏi màn hình vĩnh viễn. Hai hàng đợi đầu không sao vì có lệnh **quét lại thư mục mỗi lần chạy**; riêng `prd-change-requests/` thì không — nên `/validate-traces` phải nhắc thay.

> **Cùng nguyên tắc đó áp cho SPEC — và đó là Step 1e (GAPS-v4 G56).** Trước đó `/sync` diff đúng ba đường dẫn `feedback/*` và **bỏ qua `specs/`**, tức hỏi *"có góp ý gì mới"* rồi bỏ qua chính tài liệu mà mọi lệnh downstream đọc. Giờ nó trả lời **hai** câu bằng **hai** mốc:
>
> | | Mốc | Trả lời | Vấn đề nếu chỉ có nó |
> |---|---|---|---|
> | **1e-A** | `{old_sha}..{new_sha}` | *"đổi gì kể từ lần **PULL**"* | **chuông cửa** — reset mỗi lần pull; pull 4 ngày liền không audit thì ngày thứ 5 chỉ thấy delta của **một** ngày |
> | **1e-B** | `spec_baseline.sha_at_audit` | *"đổi gì kể từ lần **AUDIT**"* | **tồn kho** — tích luỹ đúng |
>
> 1e-B đọc mốc mà `/validate-traces` Step 6b ghi (cùng khối dùng cho cờ `PRD_UNTRACKED_EDIT`). `/sync` **chỉ đọc, không bao giờ ghi** — nếu nó cũng ghi thì mốc audit trượt theo mỗi lần pull, tức 1e-B thoái hoá thành 1e-A. Và dòng `Next` giờ **rẽ nhánh theo dữ liệu**, không còn in một hằng số.
>
> Trước v0.4.3, hàng đợi thứ ba **không có người lấy ra**: có producer, có storage, có commit, có mặt trong `/sync` — nhưng 0 consumer, và **không gì báo**. Yêu cầu nghiệp vụ thật do tester phát hiện từ sản phẩm chạy thật rơi vào im lặng hoàn toàn. Từ v0.4.3, cả ba hàng đợi được khai vào `bin/trace-schema.json` §`queues` nên **self-check chặn build** nếu một hàng đợi mất consumer.

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Tester/QC** | `/report-bug`, `/propose-scenario` — feedback có hồ sơ |
| 👤 **Tất cả** | `/learn` khi thấy định hướng lặp lại |
| 👤 **Dev** | `/fix-bug` — truy gốc → sửa → regression test |
| 👤 **Lead** | `/sync` — đồng bộ umbrella, nổi feedback |
| 🤖 **AI** | Ghi feedback đúng schema, incorporate proposal `accepted` vào `/generate-bdd`, nạp lại lessons |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Bug này gắn với **scenario/spec** nào (spec-anchored)?
- Scenario còn thiếu nào cần đề xuất bổ sung?
- Định hướng nào **lặp lại** đủ để thành lesson dán tường?
- (Umbrella) feedback nào cần **nổi lên** cho PO/Dev?

---

## Framework xử lý thế nào (Mechanics)

1. **`/report-bug` / `/propose-scenario`** — ghi feedback kèm tham chiếu spec vào `feedback/` (spec repo nếu umbrella).
2. **`/learn`** — qua step `capture-lesson`, ghi guardrail vào `project-lessons.md`; các workflow sau **nạp lại** vào context → hệ thống *nhớ*.
3. **`/generate-bdd`** — incorporate scenario proposal đã `accepted`, kèm **normalize** (xem dưới).
3b. **`/extend-prd`** — nhặt PRD change request `accepted` làm **nguyên liệu** (không chèn thẳng: yêu cầu nghiệp vụ phải qua PO chốt AC/BR đúng tầng) → discovery delta + **kiểm va chạm** với UC/BR đã có → đánh số **nối tiếp** → ghi **add-only** + guard sau-ghi → bump version + changelog nêu rõ scope → đóng dấu `incorporated` + `archived/` + commit.
4. **`/fix-bug`** — đọc bug spec-anchored → branch `fix/{TICKET}-<slug>` → root cause → sửa (tag `@trace.fixes/root_cause/regression`) → regression test + build verify → **cập nhật sổ trace** → commit sau khi user duyệt.
5. **`/sync`** — pull, init submodule, **nổi feedback** lên PO/Dev, **nổi spec delta** (Step 1e), bootstrap config service, kiểm luật git cho sổ trace, làm mới Living Docs + spec manifest. An toàn chạy lặp lại. Không thấy thư mục spec (`paths.specs_dir`) trong repo → Step 1e in một dòng `ⓘ Spec: bỏ qua — không thấy …` thay vì im lặng (im lặng đọc như "tài liệu không đổi").

### Vòng đời bug — mỗi bước có đúng một chủ

| State | Ai đặt | Khi nào |
|---|---|---|
| `🟢 Open` | `/report-bug` | tester/QC file bug |
| `🟡 Fixed` | `/fix-bug` Phase 5.5 | fix đã commit + push |
| `🟢 Closed` | **`/qc-run-script`** | QC chạy lại và `qc_status` của SC liên kết flip `pass` |

> **Dev không tự đóng bug của mình** — QC sở hữu verification. `/qc-run-script` đọc `qc_blocked_by` **trước** khi clear nó (cột đó chính là con trỏ tới bug; clear xong là mất đường về).
>
> Ngoại lệ có chủ đích: SC pass mà bug còn `🟢 Open` (chưa ai fix) → **không đóng**, giữ `Open` + cảnh báo kiểm tra lại test. Test pass trên bug chưa fix là dấu hiệu **test sai**, không phải bug hết — tự đóng ở đây sẽ chôn một defect thật.

### `/fix-bug` ghi gì vào sổ trace

Regression test không phải test "ngoài luồng" — nó phải hiện lên coverage:

| Cột | Ghi gì |
|---|---|
| `test_count` | **+=** số test regression (cộng dồn, không ghi đè) |
| `test_classes` | **append** tên class mới |
| `dev_selftest` → `not_run` · `dev_selftest_at` → `—` | code vừa đổi nên tín hiệu self-test cũ hết hiệu lực |

**Không** đụng `qc_*` (QC sở hữu) và **không** đụng `spec_ver`/`gen_ver` — fix bug không đổi spec, đụng vào là tạo `DRIFT` giả. Next của `/fix-bug` là `/dev-run-test` để lấy lại tín hiệu xanh, rồi mới tạo PR.

### Proposal của tester — normalize khi chèn

`/propose-scenario` viết scenario theo **đúng** bộ tag canonical (`@trace.scenario` với placeholder `SC?` · `@trace.sc_version: 1.0` · `@trace.business_rules`; AC ghi thành comment `# Covers:` chứ **không** phải trace key). Khi `/generate-bdd` chèn proposal `accepted`, nó phải:

1. gán `sc_id` = số SC **kế tiếp** trong file (thay `SC?`)
2. bổ sung `@trace.business_rules` nếu proposal để `—`
3. **strip** `@proposed` / `@from-test` — nhãn vòng đời proposal, không thuộc BDD canonical
4. đặt scenario vào **đúng NHÓM** theo business theme
5. **append row `.tsv`** (`spec_ver = 1.0`, `status = UNTRACKED`)

> Thiếu bước 1/5 thì scenario vào file mà **không có row trace** → vô hình với toàn bộ coverage và drift.

---

## HITL / Gate

- ⚪ Không gate chặn — nhưng là nơi **con người quyết** điều gì đáng thành spec/lesson.
- `/fix-bug` commit **sau khi user duyệt**.

---

## Ví dụ (Example)

```
QC phát hiện: link reset vẫn dùng được sau khi đổi mật khẩu
  /report-bug  → feedback/bug-reports/BUG-217.md  (@trace tới UC-02/SC-02.1)
  /sync        → nổi lên cho Dev
  /fix-bug BUG-217 → branch fix/BUG-217-invalidate-link
                     → root cause: thiếu invalidate token
                     → fix + regression test + build ✅
                     → trace: test_count +2, dev_selftest → not_run
                     → BUG-217 State: 🟡 Fixed
  /dev-run-test AUTH-UC2 → dev_selftest → pass
  /qc-run-script  AUTH-UC2 → qc_status SC1 → pass
                     → BUG-217 State: 🟢 Closed (verified)
  /learn "luôn invalidate one-time token sau khi dùng"
                     → project-lessons.md (nạp lại lần sau)
```

---

## Anti-pattern

- ❌ Sửa định hướng bằng miệng rồi để AI "quên" — phải `/learn`.
- ❌ Bug không gắn spec → khó truy vết, khó regression.
- ❌ Coi feedback loop là "chạy vòng lặp" — nó cải tiến spec/tri thức, pipeline vẫn một chiều.

---

## Hết pipeline (End of pipeline)

Bạn đã đi hết 11 bước (0→10). Xem lại bức tranh lớn ở [Overview](../overview.md), hoặc quay về [danh sách các bước](README.md).
