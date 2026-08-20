[← Glossary](glossary.md) · [Concepts](./) · [Traceability →](traceability.md)

# Roles & HITL — Vai trò và điểm dừng con người

> Ai làm gì ở mỗi giai đoạn, và tại sao điểm dừng của con người (HITL) **không cào bằng**.

---

## Ma trận Role × Pipeline

| Role | Discovery | Specification | Design | Implementation | Dev-test | QC |
|------|:---------:|:-------------:|:------:|:--------------:|:--------:|:--:|
| **PO / Business** | 🟢 Lead | 🟢 Lead | 🟡 Review | ⚪ | ⚪ | 🟡 Review gap |
| **SA / Architect** | 🟡 Consult | 🟡 Review (SA lens) | 🟢 Lead (tech-docs) | 🟡 Review | ⚪ | ⚪ |
| **QA / Tester** | 🟡 Consult | 🟡 Feedback | 🟡 Review | 🟡 Review | 🟡 | 🟢 Lead (qc-*) |
| **Dev** | ⚪ | 🟡 Review (DEV lens) | 🟡 Review | 🟢 Lead (code) | 🟢 Lead | 🟡 Triage |
| **AI Agent** | 🤖 Co-pilot | 🤖 Draft PRD/BDD | 🤖 Draft design/tech | 🤖 Generate code | 🤖 Run smoke | 🤖 Run Playwright |

🟢 Lead · 🟡 Consult/Review · ⚪ Không tham gia · 🤖 AI thực thi (luôn dưới HITL)

---

## Mật độ HITL (HITL Density)

**Rule vàng:** sai ở thượng nguồn nhân lên cấp số nhân → checkpoint **dày** ở spec/design, **mỏng** ở code/test.

```
Discovery ████████████  🔴 cao nhất — chốt từng chặng
PRD       ██████████    🔴 review-context sạch critical + approve
BDD       █████████     🔴 UC decomposition + findings B1–B6
Tech-Docs ██████        🟠 review đa chiều + ký T7
Code      ███           🟡 comprehension checkpoint
Dev-test  ██            🟡 dev tự chạy
QC        █████         🟠 cổng review test & script
```

---

## Hai loại điểm dừng (Checkpoint vs Gate)

| | Checkpoint (🛑) | Gate (🔒) |
|---|---|---|
| **Cơ chế** | AI trình output, hỏi `Y/N` | Field trạng thái do người đặt |
| **Ví dụ** | UC decomposition, comprehension | `Status: approved`, `@trace.status: approved` |
| **Chặn downstream?** | Không — chỉ dừng tại chỗ | Có — chặn tới khi `approved` |

> **Gate = trạng thái, không phải lệnh.** Không có `/approve-prd` — "duyệt" là người đặt `| Status | approved |` trong Metadata PRD (hoặc `@trace.status: approved` trong `.feature`).

---

## Nguyên tắc thiết kế checkpoint

1. **Tường minh** — marker thống nhất (🛑/CHECKPOINT) để AI không "trượt qua".
2. **Hỏi đúng câu** — không "OK chưa?" mà *"UC list đủ chưa? thiếu/thừa UC nào?"*.
3. **Short-circuit** — AI tự detect không có gap thì được skip, nhưng phải nêu lý do.
4. **Save state** — workflow dài (Discovery 8 chặng) lưu state để resume.
5. **Reject = học** — con người sửa → cân nhắc `/learn` ghi lesson.

---

## Anti-pattern (cần tránh)

- ❌ AI đi thẳng từ "ý tưởng" tới code, bỏ qua PRD/BDD.
- ❌ Sinh code khi BDD còn `draft`.
- ❌ Checkpoint chỉ "confirm Y/N" mà không trình output cụ thể.
- ❌ Đưa chi tiết kỹ thuật vào PRD/BDD (sai altitude).
- ❌ AI tự fix tự review — lặp lỗi; review là **read-only**.

---

## Đọc theo vai trò của bạn (Role Guides)

→ [Product Owner](../03-guides/product-owner.md) · [Developer](../03-guides/developer.md) · [Architect](../03-guides/architect.md) · [Tester/QA](../03-guides/tester-qa.md)
