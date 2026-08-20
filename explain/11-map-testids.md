[← /review-code](10-review-code.md) · [Explain Home](README.md) · [Next: /dev-gen-test →](12-dev-gen-test.md)

# 11 · `/map-testids` — Dán test-id ổn định cho UI (FE)

> **Một câu.** Gắn **test-id ổn định** vào các element có hành động trên UI FE và ghi bản đồ selector vào tech-doc §4.5.6 — làm cầu nối để QC Playwright bám selector không vỡ.

---

## Vấn đề giải quyết

Test tự động (Playwright) vỡ khi selector đổi (class/text thay đổi). `/map-testids` chuẩn hoá **test-id ổn định** cho mọi element tương tác, đảm bảo component tái dùng forward được test-id, và ghi map để QC dùng — tách concern "làm UI test được" khỏi "viết test".

---

## Vị trí & tiền đề

- **Vị trí:** Phase Tech Design / Implementation (FE), giữa code FE và QC.
- **Chặn cứng:** chỉ FE/App (platform guard) — BE không có UI.

---

## Input / Output

**Input:** UI code FE (element có action) + catalog component tái dùng + tech-doc §4.5.

**Output:** code FE được patch test-id + map §4.5.6 Test Selectors trong tech-doc.

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

- **Lệnh cầu nối FE→QC** — vị trí pipeline hơi mờ (giữa Tech Design & Code). Chạy sớm quá thì UI chưa xong, muộn quá thì QC phải chờ. Đáng làm rõ thời điểm tối ưu.
- **Phụ thuộc catalog component tái dùng** — nếu component không forward được prop test-id, Step 3 phát sinh sửa lan rộng.
- **EXTEND-only** an toàn nhưng nếu test-id cũ sai thì không tự sửa.
- **Không bắt buộc trong golden path** — dễ bị bỏ qua, khiến QC selector giòn. Cân nhắc tích hợp vào `/generate-code --phase=ui`.

---

## Kết nối

**Trước:** [`/generate-code`](09-generate-code.md) (UI FE) · **Sau:** [`/qc-*`](15-qc-analyze.md) dùng selector đã map.
