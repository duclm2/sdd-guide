[← /qc-smoke-test](32-qc-smoke-test.md) · [Explain Home](README.md) · [Next: /validate-traces →](21-validate-traces.md)

> *Utility **ngoài luồng tuần tự** — không phải trạm nào của dây chuyền QC. Số file `33` chỉ là thứ tự trang.*

# 33 · `/qc-metrics` — Số liệu chất lượng, đọc từ sổ KPI

> **Một câu.** Lead/PM chạy cuối sprint hoặc trước release để xem **số đếm cơ học** về hoạt động QC (coverage automation, workload, cycle time, flaky, test không chạy…) — **chỉ đọc**, không ghi file nào; phần **suy diễn** quy luật lỗi chỉ chạy khi gõ `--defects`.

---

## Vấn đề giải quyết

Mỗi lệnh QC khi chạy xong append một dòng vào sổ KPI `{paths.trace_dir}/quality-metrics.jsonl`. Sổ đó chỉ có giá trị nếu có chỗ **đọc gộp** nó thành câu trả lời cho người quyết: *"automation phủ bao nhiêu"*, *"việc có dồn vào một người không"*, *"smoke có đáng tin không"*. Lệnh này là chỗ đó — và nó được dựng để **không biến "chưa đo" thành "bằng 0"**, cũng không để một phán đoán đi ra ngoài mang dáng một số đo.

---

## Vị trí & tiền đề

- **Vị trí:** utility, chạy bất cứ lúc nào; đầu ra là **số cho người quyết**, không có lệnh kế tiếp.
- **Phạm vi tự phân giải — KHÔNG dùng `steps/qc-scope.md`** (cùng lý do `/qc-smoke-test`: câu hỏi ở đây là câu cả đợt, cổng `4a`/`4b` sẽ DỪNG khi target không phải một PRD/UC):

| Gõ | Phạm vi |
|---|---|
| `{TICKET-ID}` · `{domain}/{prd-slug}` | 1 PRD |
| `{domain}` | cả domain |
| rỗng · `all` | toàn dự án |

- **Khung thời gian:** `--from YYYY-MM-DD` · `--to YYYY-MM-DD`. Không nêu ⇒ **toàn bộ sổ**, và in rõ khoảng thời gian thật đã dùng (dòng cũ nhất → mới nhất) để không ai tưởng đó là số của sprint này.
- **Tiền điều kiện:** sổ KPI chưa có, hoặc 0 dòng khớp phạm vi/khung ⇒ **DỪNG**, in `CHƯA ĐỦ DỮ LIỆU ≠ KẾT QUẢ BẰNG 0` kèm số dòng tổng / số dòng khớp. **Không in bảng toàn số `0`.**

---

## Input / Output

**Input:** sổ KPI `quality-metrics.jsonl` · sổ trace · `{paths.bug_reports_dir}/` · skill `qa-metrics/aggregate.md` (công thức + nguồn từng chỉ số, khuôn output).

**Output:** một báo cáo in ra màn hình theo khuôn `aggregate.md §Khuôn output` (Coverage & Workload · Cycle time · Độ tin cậy · Flaky rate · Automation ROI). `Output Artifacts: none` — kể cả sổ KPI cũng không được ghi thêm (đọc số liệu không phải một hoạt động QC; ghi vào thì workload của Lead/PM lẫn vào workload của QC).

---

## Các bước xử lý (chi tiết)

| Step | Việc |
|---|---|
| Gate | `steps/gate.md` — target không phải UC-ID |
| Phạm vi + khung | theo bảng trên; in khoảng thời gian thật |
| Tiền điều kiện | không có dữ liệu ⇒ DỪNG, không in bảng số 0 |
| Phần ĐẾM (mặc định) | tính các chỉ số theo `aggregate.md` — ví dụ: *automation coverage* (lấy dòng `qc-automation-assess` **mới nhất** mỗi `uc_id`, kèm `reassess_count`) · *workload* theo `qc_owner_name` (một người >60% ⇒ cảnh báo nghẽn cổ chai) · *cycle time* báo **trung vị** · *flaky rate* tách riêng regression và smoke · *bốn con số test không chạy* · đối chiếu chéo tổng trạm 8 ↔ trạm 9 |
| Phần `--defects` (chỉ khi gõ cờ) | 5 Whys từ triệu chứng xuống nguyên nhân, gom theo nhóm nguyên nhân và module; nêu quy luật khi **≥ 2 bug** cùng nhóm/module |
| Self-review | *cỡ mẫu* có in chưa · *không lấp bảng* (ô `0` nào thật ra là "chưa đo") · *`--defects`* người có gõ cờ không · *giả thuyết* có kèm mã bug · *ROI* đã dán nhãn ƯỚC LƯỢNG |
| Report | footer **bỏ dòng Pipeline** (utility xuyên suốt) |

---

## Checkpoint & Gate

- **CHECKPOINT: không chặn** — read-only hoàn toàn: đọc ba nguồn, in báo cáo, không ghi file nào. Không có gì để mất nên không có gì để hỏi.
- **Không ghi dòng KPI cho chính nó** — khác mọi lệnh QC còn lại; vì vậy `qc-metrics` không nằm trong danh sách phase của sổ.

---

## Cơ chế đặc biệt

- **Hai luật trùm:** thiếu dữ liệu ⇒ ghi *"chưa đủ dữ liệu"* kèm cỡ mẫu, không lấp bảng · `Automation ROI` **luôn dán nhãn ƯỚC LƯỢNG** và in giả định đã dùng (tám chỉ số kia đo trực tiếp, cái này thì không; không có căn cứ cho giả định nào ⇒ bỏ hẳn chỉ số).
- **Vế thứ hai của mỗi cặp mới đáng xem:** `quarantined_count` / `quarantine_overdue` · `disabled_count` / `disabled_untracked`, cả hai nền. *"8 test quarantine"* là trạng thái; *"3 cái quá hạn"* và *"5 test skip không ai nhận"* là nợ.
- **`--defects` bị rào ba lớp:** (1) in là **giả thuyết** kèm **danh sách mã bug** để người tự kiểm · (2) **ngưỡng ≥ 5 bug**, dưới ngưỡng in `chưa đủ dữ liệu ({n}/5 bug)` · (3) **không tự sửa gì** — mỗi đề nghị mang một mã `IMPROVE-XX-nn` để lần sau đối chiếu đã làm chưa.
- **Không đẻ từ vựng thứ ba cho nguyên nhân:** dùng sáu nhánh `BUG_FLOW` của `/report-bug` (Code · BDD bug · PRD ambiguity · BDD coverage gap · Design Spec · Environment); hai nhóm test-data và automation/regression ánh xạ vào nhãn `script-bug` của `/qc-run-script`.
- **Đối chiếu chéo là phép kiểm việc đếm, không phải chỉ số chất lượng:** tổng ở `qc-report` phải bằng tổng cộng dồn từ `qc-run-script` + `qc-run-manualtest`. Lệch thì in cả hai số và danh sách SC chỉ có ở một bên — không tự chọn bên nào đúng.

---

## 👓 Góc nhìn tối ưu

- **Điểm mạnh:** tách rõ phần **cơ học** (máy ra cùng một số mỗi lần) khỏi phần **phán đoán** (`--defects`), và mặc định chỉ in phần cơ học. Sai ở phần phán đoán không phải một con số lệch mà là *sửa nhầm quy trình* — nên rào cờ là đúng chỗ.
- **Chỉ tốt bằng sổ KPI.** Dòng chỉ được ghi khi lệnh chạy trọn; lệnh bị bỏ dở, chạy ngoài framework, hoặc `qc_owner_name` không phân giải được (`unknown`) đều làm số thiếu. Dự án mới bật cơ chế sẽ không có số một thời gian — lệnh nói thẳng điều này thay vì in 0.
- **Số chỉ số và số nguồn chưa khớp giữa lệnh và skill.** `.tmpl` nói *"chín chỉ số"* và *"đọc ba nguồn"*; `aggregate.md` có thêm §7b (nợ nhìn thấy được), §7c (đối chiếu chéo) và §7d (`stale_overdue`, đọc thẳng `DOC_GAP.md` — nguồn thứ tư). Người đọc lệnh sẽ đánh giá thấp phạm vi báo cáo.
- **"Cycle time/UC" thực chất là theo PRD.** Công thức lấy hai đầu mút là dòng `qc-analyze` và `qc-report` của cùng một `uc_id`, mà cả hai phase này đều cấp PRD — theo `bin/trace-schema.json` (`uc_id_note`) chúng ghi `{TICKET-ID}` vào ô `uc_id`. Khuôn output vẫn in *"ngày/UC"*, dễ đọc nhầm đơn vị.

---

## Kết nối

**Trước:** các lệnh QC đã ghi sổ KPI — từ [`/qc-analyze`](15-qc-analyze.md) tới [`/qc-report`](20-qc-report.md), và [`/qc-smoke-test`](32-qc-smoke-test.md) · **Sau:** không có lệnh kế tiếp — mỗi đề nghị `IMPROVE-XX-nn` là việc của người. Muốn làm mới Living Docs thì [`/validate-traces`](21-validate-traces.md).
