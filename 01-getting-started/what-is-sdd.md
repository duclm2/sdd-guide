[← Docs Home](../README.md) · [Next: Installation →](installation.md)

# What is SDD — Framework là gì? (5 phút)

> **SDD Framework** (`@educa-corp/sdd-framework`) là một **spec-driven development framework** đóng vai trò **đường ray** điều phối AI agent (Claude Code) đi từ **ý tưởng sản phẩm → PRD → Design-Spec → BDD → Tech-Docs → Code → Dev self-test → QC**, theo một **pipeline một chiều, có gate kiểm soát chất lượng và truy vết đầy đủ**.

---

## Khác gì với "dùng AI để code"?

Dùng AI tự do (mở chat, gõ prompt, copy-paste) → AI suy diễn theo trực giác, output không bám requirement, sửa code làm spec lệch, lần sau AI hiểu sai. Framework này **áp đặt cấu trúc**:

| | Dùng AI tự do | SDD Framework |
|---|---|---|
| **Nguồn chân lý** | Code (và trí nhớ vài senior) | **Spec** (PRD + BDD) — code là *hệ quả* |
| **Truy vết** | Không | Mọi dòng code link `@trace` về scenario |
| **Kiểm soát chất lượng** | Tuỳ người | **Gate bằng findings + approval** trước downstream |
| **Điểm dừng con người** | Ngẫu nhiên | **HITL có chủ đích** — dày ở spec, mỏng ở code |
| **Đổi requirement** | Đập đi làm lại | **Drift per-UC** — chỉ regen phần lệch |
| **Trí nhớ dự án** | Trong đầu người | `/learn` + `domain-knowledge/` |

---

## Vấn đề framework giải quyết

| Vấn đề | Cách giải |
|--------|-----------|
| AI viết lung tung, không bám requirement | Spec là **anchor cứng**; code link `@trace.source` về scenario |
| Sửa code → spec lệch → AI hiểu sai lần sau | **Spec là SSOT của "bản hiện tại"**; code mang version của "bản tôi được sinh theo" — **lệch nhau chính là tín hiệu drift** |
| Sửa spec → không biết code nào cần regen | **Trace state `.tsv`** một sổ / UC × platform → `OK`/`GAP`/`DRIFT`/`UNTRACKED`/`ORPHANED` |
| Xoá scenario → code mồ côi nằm im | **Quét ngược code → spec**; code trỏ vào SC không còn tồn tại bị gắn cờ 🔴, không tự hết |
| Build xanh nhưng ghép luồng chạy vào no-op | **Sổ seam/stub** — bắt lớp lỗi mà build + test từng-UC không thấy |
| PRD/BDD kém → code rác hàng loạt | **Quality gate** (`/review-context`) phải sạch critical + PO approve |
| Lỗi/định hướng sai lặp lại | **`/learn`** ghi lesson, nạp lại vào context |
| Đổi tech stack → viết lại workflow | **Module overlay** — skill đọc `stack-profile.yaml`, không hardcode |
| Nhiều team, một nguồn spec | **Umbrella** — spec repo dùng chung + `/sync` |

---

## Triết lý cốt lõi (Core Philosophy)

> **Viết spec trước. Sinh code từ spec. Truy vết mọi thứ.**
> *(Write the spec first. Generate the code from the spec. Trace everything.)*

- **Con người định nghĩa *cái gì*** — acceptance criteria, business rule, yêu cầu nền tảng.
- **AI sinh ra *như thế nào*** — BDD, tech design, code, test — thích ứng theo platform.
- **Mỗi artifact được review** trước khi sang phase sau.
- **Mỗi dòng code truy vết** về một scenario trong `.feature`.
- **Đa service**: mỗi service tiến hoá độc lập nhưng chung một workflow.

---

## Ba "trái tim" của hệ thống

Bỏ một trong ba là hỏng:

1. **HITL (điểm dừng con người)** — sai ở thượng nguồn nhân lên cấp số nhân. Bỏ = "tàu lao xuống vực".
2. **Traceability (truy vết)** — biết mỗi dòng code thuộc scenario nào. Bỏ = "lạc rừng".
3. **Context-loader (nạp đúng context)** — AI nhận đúng-đủ-gọn thông tin. Bỏ = "lái tàu trong sương mù".

---

## Framework KHÔNG giải quyết (Scope boundary)

- Không thay **product judgment** (PO quyết cái gì đáng làm).
- Không thay **architecture judgment** (SA quyết trade-off).
- Không **debug runtime production**, không **CI/CD deploy**.
- Không **ràng buộc AI model cứng** — chỉ khuyến nghị (model check mềm).

---

## Đọc tiếp (Next)

- [Installation](installation.md) — cài đặt & khởi tạo dự án
- [Quick Start](quickstart.md) — feature đầu tiên end-to-end
- [Concepts › Overview](../02-concepts/overview.md) — mô hình toàn trình chi tiết
- [Glossary](../02-concepts/glossary.md) — thuật ngữ
