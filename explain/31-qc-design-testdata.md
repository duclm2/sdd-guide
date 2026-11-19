[← /qc-automation-assess](30-qc-automation-assess.md) · [Explain Home](README.md) · [Next: /qc-design-script · /qc-run-script →](19-qc-run-test.md)

> *Utility, **không mang số trạm** — đứng giữa trạm 5 (`/qc-automation-assess`) và trạm 6 (`/qc-design-script`). Số file `31` chỉ là thứ tự trang.*

# 31 · `/qc-design-testdata` — Kế hoạch dữ liệu test cho automation

> **Một câu.** Trước khi viết script, lập **một** bảng `TEST_DATA_PLAN.md` cho cả PRD: mỗi dữ liệu test (tài khoản, giá trị biên, seed…) được **ai dùng**, **lấy từ đâu**, **dùng riêng hay chung** — rồi chạy hai phép kiểm cơ học để mâu thuẫn lộ ra ngay lúc lập bảng.

---

## Vấn đề giải quyết

Ví dụ trong skill: hai bài test dùng chung tài khoản `qa_student_01`. Bài A đổi mật khẩu, bài B đăng nhập bằng mật khẩu cũ. Chạy tuần tự thì B đỏ; chạy song song thì tuỳ thứ tự. Người điều tra thấy B *"chập chờn"*, gắn `flaky`, quarantine — tuần sau nó nổ lại.

Lỗi này **không sửa được bằng cách sửa test**: nguyên nhân là hai bài cùng sở hữu một tài khoản mà không ai khai ra. Phải sửa **quyết định về dữ liệu**, và quyết định đó cần một chỗ để nằm — chính là bảng này.

---

## Vị trí & tiền đề

- **Vị trí:** sau `/qc-automation-assess` (cần biết TC nào `Automatable: Y`), trước `/qc-design-script`.
- **Vì sao không đánh số trạm:** chen vào giữa trạm 5 và 6 nghĩa là đánh số lại toàn bộ phần sau (đo 2026-09-23: 72 chỗ nhắc *"trạm &lt;số&gt;"* trong 28 file). Ba lý do hoãn: dây chuyền 9 trạm chưa ai chạy hết lần nào · đội QC để ngỏ khả năng **gộp** lệnh này vào `/qc-automation-assess` sau 2–3 PRD chạy thật · bộ đề xuất của đội QC dùng một sơ đồ 8 phase khác.
- **Thứ giữ thứ tự là cái chốt, không phải con số:** `/qc-design-script` **DỪNG** khi thiếu `TEST_DATA_PLAN.md` (`⚠️ Chưa có TEST_DATA_PLAN.md cho {TICKET-ID}. Chạy /qc-design-testdata {TICKET-ID} trước.`).
- **Tiền điều kiện:** không có `AUTOMATION_ASSESSMENT.md` ⇒ **DỪNG**, báo chạy `/qc-automation-assess` trước. Có file nhưng không TC nào `Automatable: Y` ⇒ **DỪNG** với `CHƯA CÓ ≠ XONG. Chưa có gì để thiết kế dữ liệu cho.`

---

## Input / Output

**Input:** `{qc_artifact_dir}AUTOMATION_ASSESSMENT.md` (chỉ TC `Automatable: Y`) · skill `qa-testdata/test-data-plan.md` (khuôn 9 cột + hai phép kiểm).

**Chỉ phục vụ automation.** Dữ liệu cho test chạy tay vẫn ở field `Test Data` của `.Test.md`, độc lập, không tham chiếu qua lại — người chạy tay tự xoay được dữ liệu, script thì cần địa chỉ chính xác.

**Output:** `{qc_artifact_dir}TEST_DATA_PLAN.md` — một file cho cả PRD, có bảng metadata + stamp phiên bản nguồn (`steps/qc-stamp.md`) ở đầu, bảng 9 cột, và mục **Mâu thuẫn phát hiện được**. Kèm một dòng KPI `"phase":"qc-design-testdata"` vào sổ `quality-metrics.jsonl`.

| Cột | Giá trị | Chặn chuyện gì |
|---|---|---|
| `TD-xxx` | ID | trace về TC/SC dùng nó |
| Type | `Account` · `Boundary Value` · `Negative Value` · `Seed/Reference Data` · `Env Config` | biết cách chuẩn bị |
| **Dùng bởi** | `{UC-ID}-SC{n}` · `TC_xxx` | cột quan trọng nhất — nhìn là biết dùng ở mấy chỗ |
| Nguồn | `Static fixture` · `Factory` · `DB seed` · `Env var` | script biết lấy ở đâu |
| **Cách ly** | `Dedicated` · `Pool` · `Stateless` | chặn đúng chuyện hai bài chung tài khoản |
| Vòng đời | `Persistent` · `Ephemeral` | có được sửa không |
| Cleanup | có/không + cách dọn | DB test phình tới lúc trùng khoá |
| Nhạy cảm | có PII/credential không | credential lọt vào git |
| Owner & refresh | ai bảo trì, khi nào làm mới | tài khoản test hết hạn thì ai biết |

---

## Các bước xử lý (chi tiết)

| Step | Việc |
|---|---|
| Gate + Phạm vi QC | `steps/gate.md` → `steps/qc-scope.md` |
| Tiền điều kiện | có `AUTOMATION_ASSESSMENT.md`? có TC `Y`? — thiếu một trong hai ⇒ DỪNG |
| Lập bảng | theo khuôn 9 cột của skill, chỉ cho TC `Y` |
| Kiểm 1 | `Cách ly = Dedicated` mà **Dùng bởi** có **>1** TC ⇒ **DỪNG**, in mâu thuẫn, hỏi người: đổi sang `Pool` hay tách nhiều bản? *Không im lặng ghi rồi đi tiếp* |
| Kiểm 2 | `Nhạy cảm = có` mà `Nguồn ≠ Env var` ⇒ **LỖI** — credential chỉ đến từ biến môi trường |
| Ghi file | kèm stamp; mục **Mâu thuẫn phát hiện được** in **cả khi rỗng** (`(none)`) |
| Self-Review | *bịa* (mỗi `TD-xxx` trỏ về một TC `Y` có thật) · *nhảy bước* (đã chạy cả hai phép kiểm) · *số liệu* (không gộp hai loại dữ liệu vào một dòng) · *secret* (đọc lại file, không lọt mật khẩu/token) |
| KPI | append **đúng một** dòng; dừng giữa chừng thì không ghi |

---

## Checkpoint & Gate

- **CHECKPOINT: chặn CỨNG** — ghi đè `TEST_DATA_PLAN.md` đã có làm mất cột `Owner & refresh` và các dòng người đã sửa tay. Theo `steps/gate.md`, mức cứng **không** bị `--yes` bỏ qua.
- Mức cứng **chỉ áp khi file đã tồn tại**; lần chạy đầu không ghi đè gì, đi thẳng.
- Ngoài CHECKPOINT, phép kiểm 1 còn là một điểm **DỪNG hỏi người** — nó chạy trước khi ghi file, nên áp cả ở lần chạy đầu.

---

## Cơ chế đặc biệt

- **Hai cặp cột tự tố cáo mâu thuẫn** — thứ làm bảng này khác một bảng ghi chép. Mâu thuẫn `Dedicated` × nhiều người dùng lộ ra **trước khi ai viết dòng script nào**; không có bảng thì nó chỉ lộ sau vài tuần dưới dạng một test *"chập chờn"*.
- **Chỉ ghi TÊN BIẾN, không ghi GIÁ TRỊ** — `process.env.QA_STUDENT_PW`, không phải mật khẩu. Cùng luật `rules/data-protection.md`.
- **Mục rỗng vẫn in.** Bỏ mục *Mâu thuẫn* lúc rỗng thì lần có mâu thuẫn, người đọc không biết mục đó vốn tồn tại.
- **Chưa tách registry dùng chung.** Chỉ cân nhắc khi thấy một dữ liệu bị dùng lại ở ≥2 PRD — không dựng hạ tầng cho vấn đề chưa xuất hiện.

---

## 👓 Góc nhìn tối ưu

- **Điểm mạnh:** chuyển một loại lỗi vốn hiện ra dưới dạng *flaky* (khó quy trách nhiệm) thành một mâu thuẫn **đọc được trên bảng**, ở thời điểm rẻ nhất để sửa.
- **Chạy lại là ghi đè, không phải merge.** Khác `/qc-automation-assess` (giữ phần người chốt), lệnh này không mô tả cách giữ dòng cũ khi PRD thêm TC `Y` mới. Muốn bổ sung dữ liệu sau khi người đã điền `Owner & refresh` thì phải qua cổng cứng và chấp nhận mất phần đó, hoặc sửa tay.
- **Stamp được ghi nhưng chưa ai đọc.** Lệnh include `steps/qc-stamp.md`, nhưng khối `qc_artifact_stamp` trong `bin/trace-schema.json` không có hàng nào cho `TEST_DATA_PLAN.md` — đúng dạng *"điền mỗi lần chạy, 0 consumer"* mà chính `qc-stamp.md` lấy làm bài học.
- **Tương lai chưa chốt:** đội QC có thể gộp lệnh vào trạm 5. Người dùng nên coi *"utility bắt buộc trước trạm 6"* là trạng thái tạm.

---

## Kết nối

**Trước:** [`/qc-automation-assess`](30-qc-automation-assess.md) (cần bảng `Automatable`) · **Sau:** `/qc-design-script {TICKET-ID}` — viết script theo bảng dữ liệu vừa lập, xem [ba trạm script](19-qc-run-test.md).
