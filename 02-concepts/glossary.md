[← Overview](overview.md) · [Concepts](./) · [Roles & HITL →](roles-and-hitl.md)

# Glossary — Thuật ngữ (Terminology)

> Định nghĩa ngắn gọn các thuật ngữ dùng xuyên suốt framework. Sắp theo nhóm.

---

## Artifact (Sản phẩm của pipeline)

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **Product Definition** | Khung intent 8 chặng từ `/define-product` — *chưa phải PRD* |
| **PRD** (Product Requirements Document) | Đặc tả nghiệp vụ: Metadata · AC · UC · BR · Wireframe · Change Log |
| **Design-Spec** | Đặc tả visual bám Figma (chỉ FE/App), 2 tầng ngôn ngữ |
| **BDD** (`.feature`) | Kịch bản hành vi viết bằng Gherkin, mang `@trace.*` |
| **Tech-Docs / Tech-Design** | Thiết kế kỹ thuật full-stack: API contract, entity, data, dependency |
| **Living Docs** | Tài liệu tự cập nhật qua `/sync` (umbrella) |

---

## Đơn vị nghiệp vụ (Business units)

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **UC** (Use Case) | Một tính năng/luồng nghiệp vụ; đơn vị phân rã chính |
| **SC** (Scenario) | Một kịch bản hành vi kiểm chứng được, thuộc một UC |
| **BR** (Business Rule) | Ràng buộc nghiệp vụ (vd: link hết hạn 15 phút) |
| **BL** (Business Logic) | Cơ chế nghiệp vụ, cùng ngăn altitude với BR |
| **AC** (Acceptance Criteria) | Tiêu chí nghiệm thu — "thế nào là làm xong đúng" |
| **Altitude** | 4 ngăn không lộn: AC (nghiệm thu) · BR/BL (cơ chế) · Scope (ranh giới) · Dictionary (định nghĩa) |

---

## Traceability

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **@trace.\*** | Tag metadata gắn artifact về nguồn (id/scenario/source/business_rules…) |
| **Boundary-only tagging** | Chỉ tag `@trace` ở boundary (controller/handler); shared code dò qua import chain |
| **`.tsv` (trace state)** | Sổ trạng thái per-UC×platform: `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` |
| **Drift** | Spec đổi sau khi sinh code (`spec_ver != gen_ver`) → cần regen |
| **Coverage status** | UNTRACKED · GAP · DRIFT · OK (xem [Traceability](traceability.md)) |
| **`dev_selftest`** | Kết quả smoke của **dev** (cột `.tsv`) |
| **`qc_status`** | Kết quả QC **chính thức** (Playwright, có evidence) — độc lập `dev_selftest` |

---

## Kiểm soát & quy trình (Control)

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **HITL** (Human-in-the-Loop) | Điểm dừng để con người duyệt — dày ở thượng nguồn |
| **Checkpoint** (🛑) | Điểm AI dừng, trình output, chờ `Y` |
| **Gate** (🔒) | Trạng thái (`Status`/`@trace.status`) do người đặt, chặn downstream tới khi `approved` |
| **Findings** | Danh sách lỗi có mã: PRD **P0–P5**, BDD **B1–B6**; sạch *critical* mới qua |
| **Comprehension checkpoint** | AI báo "{X} new, {Y} drifted — Proceed?" trước khi sinh code |
| **Model check** | Gate mềm khuyến nghị model Opus (Y/S/N) |
| **Business Language Guard** | Chặn thuật ngữ kỹ thuật lọt vào PRD/BDD |
| **Scope Lock** | Cấm implement/xoá UC khác trong file dùng chung |
| **Completeness-critic** | Vòng critic "đã đủ chưa / có lộn tầng không" sau fan-out |
| **Fan-out** | Spawn mỗi UC/dimension một sub-agent chạy song song |

---

## Kiến trúc & vận hành (Architecture & Ops)

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **Command / Skill / Step** | 3 lớp: orchestrator / domain logic / infra |
| **context-loader** | "Thủ thư" chọn đúng-đủ-gọn context cho mỗi workflow |
| **Module** | Overlay theo stack (java-spring, react, qc-playwright…), khai báo `stack-profile.yaml` |
| **`project-context.yaml`** | Config runtime: paths, mode, services, conventions, tech_stack |
| **`CLAUDE.md`** | Instruction cho agent: root (chung) + service overlay (stack) |
| **Domain** | Nhóm nghiệp vụ; segment đầu trong `specs/{domain}/…` |
| **prd-slug** | Tên folder feature-package (segment sau `{domain}`) |
| **Single vs Umbrella** | Một repo · vs · nhiều service submodule + spec repo dùng chung (`spec_source`) |
| **T7 sign-off** | Cổng ký liên team cho contract cross-service |
| **`/learn` lesson** | Guardrail ghi vào `project-lessons.md`, nạp lại vào context |
| **data-guard** | Hook chặn đọc/ghi file nhạy cảm (secret/.env) |

---

## Đọc tiếp (Next)

- [Roles & HITL](roles-and-hitl.md) · [Traceability](traceability.md) · [Architecture](architecture.md)
- [Pipeline Steps](pipeline-steps/) — thuật ngữ trong ngữ cảnh
