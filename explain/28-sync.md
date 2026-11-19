[← /learn](27-learn.md) · [Explain Home](README.md) · [Next: /update-framework →](29-update-framework.md)

# 28 · `/sync` — Đồng bộ & làm mới umbrella

> **Một câu.** Lệnh vận hành hằng ngày cho umbrella: pull + init submodule + advance spec repo + **nổi feedback tester** + bootstrap config service + làm mới Living Docs & spec manifest. An toàn chạy lặp lại.

---

## Vấn đề giải quyết

Umbrella nhiều submodule + một spec repo dùng chung dễ lệch nhau. `/sync` là "một lệnh lo mọi thứ": đồng bộ code/spec, nổi feedback, cập nhật dashboard — tự phát hiện cần làm gì.

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (vận hành), chủ yếu **umbrella mode**.
- **Đặc biệt:** idempotent — chạy lặp an toàn.

---

## Input / Output

**Cú pháp:** `/sync [spec-branch]` — branch của spec submodule để pull; bỏ trống thì tự phân giải (Step 0-D).

**Input:** umbrella repo + submodule + `spec_source`.

**Output:** repo đồng bộ + service config bootstrap + Living Docs + spec-manifest làm mới + feedback nổi lên.

---

## Các bước xử lý (chi tiết)

1. **Step 0 · Pre-flight Checks** — kiểm trạng thái submodule (entry `-` chưa clone).
2. **Step 1 · Umbrella Pull** — pull umbrella; sync `.gitmodules`; init **chỉ** submodule chưa clone (KHÔNG blanket `submodule update --recursive` để không detach submodule đang làm việc).
3. **Step 1b · Phân loại & Sync từng Submodule** — xử lý per-submodule (an toàn với cái đang active).
4. **Step 1c · Advance Spec Submodule** (nếu `spec_source`) — cập nhật spec repo dùng chung.
5. **Step 1d · Surface Feedback tester/QC** — nổi bug report / scenario proposal / PRD change request lên.
   **Step 1e · Spec delta** — tài liệu nào vừa đổi, theo hai mốc (xem mục riêng cuối trang).
6. **Step 2 · Post-sync State Check.**
7. **Step 3 · Bootstrap Service Configs** — tự sinh `project-context.yaml` cho mỗi service (phát hiện module từ pom.xml/go.mod/package.json/pubspec.yaml).
8. **Step 4 · Check luật git cho sổ trace** — `{paths.trace_dir}` **phải commit** (bị gitignore là 🔴 báo động), `.trace-mirror/` và `.living-docs/` phải gitignore; kiểm luật merge khi hai người cùng ghi sổ.
9. **Step 5 · Refresh Living Docs** (umbrella).
10. **Step 6 · Refresh Spec Manifest** (nếu `spec_source`).

---

## Checkpoint & Gate

- **Không chạy gate** (lệnh bảo trì, không có target để phân giải) — idempotent, tự dò việc cần làm.

---

## Cơ chế đặc biệt

- **Không blanket submodule update** — bảo vệ submodule đang chỉnh khỏi bị detach.
- **Bootstrap service config tự động** — phát hiện stack từ manifest file → giảm setup tay.
- **Nổi feedback** — khép vòng: feedback tester đi vào tầm nhìn PO/Dev.

---

## 👓 Góc nhìn tối ưu

- **Lệnh "lo mọi thứ"** — mạnh nhưng nhiều bước; khó chẩn khi một step lỗi. Đáng có chế độ verbose/dry-run.
- **Chỉ umbrella** — single-service không hưởng bootstrap/living-docs auto; các bước tương đương phải làm tay.
- **Chạy hằng ngày phụ thuộc kỷ luật** — feedback/Living Docs chỉ tươi nếu sync đều.
- **Phát hiện module từ manifest file** — stack lạ (không pom/go.mod/package.json/pubspec) không auto-detect được.

---

## Kết nối

**Trước:** bất kỳ (vận hành) · **Sau:** [`/validate-traces`](21-validate-traces.md); xử lý `📥 tester feedback` nổi lên → [`/fix-bug`](23-fix-bug.md)/[`/generate-bdd`](06-generate-bdd.md).

---

## Step 1e — "tài liệu nào vừa đổi" *(mới, GAPS-v4 G56)*

Câu hỏi số **một** của dev sau mỗi lần sync. Trước đó `/sync` **không** trả lời: nó diff đúng ba đường dẫn `feedback/*` và **bỏ qua `specs/`** — tức hỏi *"có góp ý gì mới"* rồi bỏ qua chính tài liệu mà mọi lệnh downstream đọc (PRD · BDD · tech-doc · design-spec). Range `{old_sha}..{new_sha}` đã có sẵn từ Step 1c, nên thêm nó là thêm **một tham số đường dẫn**.

Step này trả lời **hai câu khác nhau**, bằng **hai mốc khác nhau**:

| | Mốc | Trả lời | Vấn đề nếu chỉ có cái này |
|---|---|---|---|
| **1e-A** | `{old_sha}..{new_sha}` | *"đổi gì kể từ lần **PULL** trước"* | Mốc **reset mỗi lần pull**. Pull 4 ngày liền không audit → ngày thứ 5 chỉ thấy delta của **một ngày** |
| **1e-B** | `spec_baseline.sha_at_audit` | *"đổi gì kể từ lần **AUDIT** gần nhất"* | — con số này **tích luỹ đúng** |

1e-B đọc khối `spec_baseline` mà `/validate-traces` Step 6b ghi (cùng khối dùng cho cờ `PRD_UNTRACKED_EDIT` của G54). `/sync` **chỉ đọc, không bao giờ ghi** — nếu nó cũng ghi thì mốc audit trượt theo mỗi lần pull, tức phá đúng thứ 1e-B tồn tại để cung cấp. `self-check` R13 canh cả lời khai này.

Mốc nằm ở `{trace_dir}/trace-report.json`, cùng file `/validate-traces` ghi, **không** phải bản cùng tên ở `.living-docs/` (bản đó dựng lại từ sổ, không có mốc). Trước 2026-09-25 cả `/sync` lẫn `/validate-traces` đọc nhầm bản `.living-docs/`, nên phần này luôn báo *"chưa có mốc"*. PRD chưa có mốc giờ được đếm và in ra, không bị bỏ qua.

**Không thấy thư mục spec thì vẫn in một dòng** `ⓘ Spec: bỏ qua — không thấy …`, không để cả mục biến mất. Mục vắng hẳn đọc như *"tài liệu không đổi"*, trong khi thật ra lệnh chưa kiểm gì. Ca hay gặp là khai sai `paths.specs_dir`. `feedback/` cũng vậy: `📥 Feedback: chưa có thư mục feedback/`.

**Đường dẫn trong lệnh diff tính từ bên trong spec repo** (`{SPEC_REL}`), không dùng thẳng `paths.specs_dir`. Ở repo tổng, `paths.specs_dir` đã mang tiền tố `{spec_source}/`. Đưa nó vào `git -C {spec_source} diff` thì tiền tố bị nhân đôi, diff ra rỗng và report báo *"không đổi"*, tức báo sai.

### Và dòng `Next` không còn là hằng số

Bản cũ in cứng `/validate-traces (full coverage check) | /generate-code {UC-ID}` — **y hệt nhau** dù 0 file đổi hay 12 file đổi. Một lời nhắc không bao giờ thay đổi thì không mang thông tin nên bị lướt; cộng thêm việc lệnh duy nhất nó gợi ý là lệnh **đắt nhất** (quét cả repo), nên con đường duy nhất được chỉ là con đường người ta sẽ không đi.

Giờ nó rẽ nhánh: có nợ audit → `/validate-traces {các PRD đó}` · chỉ 1e-A có đổi → `/generate-code` cho phần đó · cả hai sạch → `✅ Spec khớp audit — không cần audit lại`.

> Đây là nguyên tắc `gate.md` Bước 3b đã áp cho CHECKPOINT (cổng luôn in một bảng giống hệt nhau thì `Y` thành phản xạ, và cổng mất tác dụng mà không ai hay — lỗi ngầm) — G56 là chỗ nó còn thiếu.
>
> **Lưu ý về bản chất:** G56 **không** phải một detector bị hỏng. Mọi detector đều đúng. Đây là một **công tắc bị thiếu** — không ai biết là cần bật.
