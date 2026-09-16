[← /generate-tech-docs](07-generate-tech-docs.md) · [Explain Home](README.md) · [Next: /review-tech-docs →](08-review-tech-docs.md)

> **Số `11` là số file lịch sử, không phải thứ tự chạy.** Lệnh này chạy ở **Phase Design**, giữa `/generate-tech-docs` và `/review-tech-docs` — tức **trước** `/generate-code`. Giữ nguyên tên file để không gãy link cũ.

# 11 · `/map-testids` — Dán test-id ổn định cho UI (FE)

> **Một câu.** Gắn **test-id ổn định** vào các element có hành động trên UI FE và ghi bản đồ selector vào tech-doc §4.5.6 — làm cầu nối để QC Playwright bám selector không vỡ.

---

## Vấn đề giải quyết

Test tự động (Playwright) vỡ khi selector đổi (class/text thay đổi). `/map-testids` chuẩn hoá **test-id ổn định** cho mọi element tương tác, đảm bảo component tái dùng forward được test-id, và ghi map để QC dùng — tách concern "làm UI test được" khỏi "viết test".

---

## Vị trí & tiền đề

- **Vị trí:** Phase **Tech Design**, sau `/generate-tech-docs` và **TRƯỚC** `/review-tech-docs` — tức trước cả `/generate-code`. Chốt hợp đồng test-id ở đây để FE và QC đọc cùng một bản đã đóng băng rồi **chạy song song**.
- **Hai chế độ:** mặc định (feature mới — nguồn là design-spec + BDD, **không đụng code**) · `--from-code` (brownfield — đọc/patch code đã có, chạy một lần mỗi UC cũ).
- **Chặn cứng:** chỉ FE/App (platform guard) — BE không có UI.

---

## Input / Output

**Input:** design-spec (Component Inventory, màn hình) + step `When` của `.feature` FE + tech-doc §4.5.1 (cây component) + catalog component tái dùng. *Chế độ `--from-code` đọc thêm UI code FE.*

**Output:** map §4.5.6 Test Selectors + `@trace.testid_attr` ở header tech-doc. *Chế độ `--from-code` còn patch test-id vào code FE.*

---

## Các bước xử lý (chi tiết)

| Step | Việc |
|------|------|
| **0 · Platform guard** | Chỉ FE/App; else STOP |
| **1 · Thu thập element có action** | Quét UI tìm element người dùng tương tác (nút, ô nhập, link…) |
| **2 · Phân giải test-id ổn định** | Đặt test-id ổn định (không phụ thuộc text/class dễ đổi) cho mỗi element |
| **3 · Đảm bảo component tái dùng forward test-id** | Component dùng lại (catalog) phải cho phép truyền test-id xuống — sửa component nếu chưa |
| **4 · Patch usage site** (chỉ EXTEND) | Gắn test-id vào nơi dùng, chỉ thêm (không viết đè) |
| **5 · Ghi/làm mới map §4.5.6** | Ghi bảng Test Selectors vào tech-doc để QC bám |
| **6 · Handoff** | Bàn giao cho QC (`/qc-*`) |

---

## Checkpoint & Gate

- Không gate chặn; EXTEND-only khi patch (an toàn).

---

## Cơ chế đặc biệt

- **Test-id ổn định** — chống test vỡ do đổi visual; nguyên tắc "selector là contract QC↔FE".
- **Forward test-id qua component tái dùng** — sửa gốc component để test-id lan xuống, không hardcode từng chỗ.
- **Map ở §4.5.6 tech-doc** — QC đọc từ một nguồn, không tự dò.

---

## 👓 Góc nhìn tối ưu

- **Lệnh chốt hợp đồng FE↔QC** — vị trí **đã được chốt**: phase Tech Design, trước `/generate-code`. Trước đây vị trí mờ (tài liệu này từng nói 'giữa Tech Design & Code') nên lệnh hay bị bỏ qua; giờ `/generate-tech-docs` trỏ thẳng sang đây, và `/review-tech-docs` T6 không cho APPROVED nếu §4.5.6 rỗng.
- **Phụ thuộc catalog component tái dùng** — nếu component không forward được prop test-id, Step 3 phát sinh sửa lan rộng.
- **EXTEND-only** an toàn nhưng nếu test-id cũ sai thì không tự sửa.
- **Đã vào golden path** — `/generate-tech-docs` → **`/map-testids`** → `/review-tech-docs` (T6 chặn nếu §4.5.6 rỗng). `lint-trace` T15/T16 canh bảng, T17/T18 canh bảng-vs-code.

---

## Kết nối

**Trước:** [`/generate-tech-docs`](07-generate-tech-docs.md) · **Sau:** [`/review-tech-docs`](08-review-tech-docs.md), rồi rẽ hai nhánh song song — [`/generate-code`](09-generate-code.md) (FE gắn attribute) ∥ [`/qc-design-test`](17-qc-design-test.md) (QC dựng test theo cùng hợp đồng).
