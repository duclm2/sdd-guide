[← /propose-scenario](26-propose-scenario.md) · [Explain Home](README.md) · [Next: /sync →](28-sync.md)

# 27 · `/learn` — Ghi một Project Lesson (Guardrail)

> **Một câu.** Biến một định hướng/lỗi lặp lại thành **guardrail** ghi vào `project-lessons.md` — nạp lại vào context mọi lệnh sau để không sai lại.

---

## Vấn đề giải quyết

Định hướng lặp lại nằm trong đầu vài senior → AI sai lại. `/learn` biến nó thành **lesson dán tường** máy-đọc, được context-loader (Bước 6.7) nạp làm ràng buộc cứng ngang coding standards → hệ thống *nhớ* qua thời gian.

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (cơ chế tự học). Cũng được gọi từ `/review-code`, `/fix-bug`, `/debug`.

---

## Input / Output

**Cú pháp:** `/learn {free-text}` (lý tưởng dạng "AI làm X, nên làm Y") · `/learn --review`.

**Input:** mô tả lesson (Rule + ngữ cảnh) + `category` + `scope`.

**Output:** entry `L-NNN` trong `project-lessons.md` (`{lessons_path}`); chế độ `--review` đổi `Status: active` → `retired` kèm lý do, không xoá dòng nào.

---

## Các bước xử lý (chi tiết)

0. **Step 0 · `--review`** *(nếu có cờ)* — phân loại lesson `active` thành 🔴 đề nghị retire / ⚠️ rà lại / ✅ còn sống, hỏi id cần retire rồi DỪNG (không đi tiếp Step 1/2).
1. **Step 1 · Parse Lesson** — tách *mistake* + *rule*, suy `category` (code-gen · bdd · tech-docs · tests · prd · general) và `scope` (domain / file glob / all); input mơ hồ thì hỏi 2 câu. Hiện lesson đã parse và hỏi `Lưu lesson này? (Y/N)`.
2. **Step 2 · Capture** (step `capture-lesson`):
   - **L1** phân giải file lessons (theo `paths.lessons_file` / service override).
   - **L2** dựng lesson (Rule + category + scope + ví dụ).
   - **L3** khử trùng lặp (không ghi lesson đã có).
   - **L4** ghi vào `project-lessons.md`.
   - **L5** in xác nhận đã ghi + nhắc commit file lessons.

---

## Checkpoint & Gate

- **Không chặn** — lệnh chỉ ghi file lessons **sau khi người trả lời `Y`** ở Step 1 (câu hỏi đó đã là cổng; thêm CHECKPOINT là hỏi hai lần cho một hành động). L5 là xác nhận **sau** khi ghi.

---

## Cơ chế đặc biệt

- **Lesson là ràng buộc cứng** — context-loader Bước 6.7 nạp lesson `Status: active` có `category` khớp lệnh (+ `general`); output vi phạm → AI sửa trước khi trình + ghi `L-NNN` đã áp.
- **Dùng chung `capture-lesson`** với `/review-code`, `/fix-bug`, `/debug` — lesson sinh tự nhiên từ chỗ phát hiện.
- **`/learn --review` — đường RA** *(từ v0.5.1, GAPS-v3 G46)*. Retire = đổi `Status` + ghi lý do, **không xoá** (lesson retired ở lại làm lịch sử). Tiêu chí 🔴 **kiểm được bằng máy**: `Scope` là file glob mà glob không còn khớp file nào ⇒ code lesson canh đã không tồn tại. `Scope` là `all`/domain thì không tự kiểm được — chỉ liệt kê theo tuổi để người quyết.

---

## 👓 Góc nhìn tối ưu

- **Bộ nhớ dự án cốt lõi** — nhưng phụ thuộc con người chủ động `/learn`. Lesson không ghi = không nhớ.
- **Dedup L3 phụ thuộc AI so khớp** — lesson gần giống có thể lọt thành trùng.
- **Scope/category filtering** quyết định lesson nào nạp — nếu gắn sai, guardrail không kích hoạt đúng lúc.
- ~~**Không có cơ chế "retire" lesson lỗi thời** — file chỉ lớn dần.~~ ✅ **Đã có từ v0.5.1** (`/learn --review`). *Ghi chú này từng đứng ở đây trước khi GAPS-v3 rà tới — nó đã đúng, chỉ chưa ai làm.*
- **Vẫn phụ thuộc con người chạy `--review`** — không gì tự retire. Cố ý: tự bỏ một guardrail trong im lặng là đúng thứ framework này tồn tại để chống. Recap cảnh báo khi ≥40 lesson active.

---

## Kết nối

**Trước:** [`/review-code`](10-review-code.md) / [`/fix-bug`](23-fix-bug.md) / phát hiện định hướng lặp · **Sau:** lesson áp dụng ở mọi lệnh kế tiếp.
