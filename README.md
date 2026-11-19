# 📚 SDD Framework — Tài liệu (Documentation)

> **SDD Framework** cho Claude Code — đường ray một chiều đưa đội ngũ đi từ
> **ý tưởng → PRD → Design-Spec → BDD → Tech-Docs → Code → Dev self-test → QC**,
> với **gate review của con người** ở mỗi khúc cua và **truy vết (traceability)** đầu-cuối.

Tài liệu chia theo **mục tiêu người đọc**. Bắt đầu ở đúng mục dành cho bạn:

| Bạn muốn… | Đọc ở |
|-----------|-------|
| Cài đặt & chạy feature đầu tiên | [01 · Getting Started](01-getting-started/) |
| Hiểu framework: mô hình toàn trình + từng bước | [02 · Concepts](02-concepts/) |
| Biết **vai trò của tôi** làm gì ở mỗi bước | [03 · Guides theo vai trò](03-guides/) |
| Tra cứu lệnh, schema, module, config | [04 · Reference](04-reference/) |

---

## Mục lục (Table of Contents)

### 01 · [Getting Started](01-getting-started/) — *Cài & chạy được*
- [What is SDD](01-getting-started/what-is-sdd.md) — framework là gì, giải quyết vấn đề gì
- [Installation](01-getting-started/installation.md) — cài đặt, `npx --init`, chọn module
- [Quick Start](01-getting-started/quickstart.md) — feature đầu tiên end-to-end

### 02 · [Concepts](02-concepts/) — *Hiểu framework*
- [Overview](02-concepts/overview.md) — mô hình toàn trình + triết lý
- [Glossary](02-concepts/glossary.md) — thuật ngữ (PRD, BDD, UC, trace, drift, gate, HITL…)
- [Roles & HITL](02-concepts/roles-and-hitl.md) — ma trận vai trò × pipeline, mật độ checkpoint
- [Traceability](02-concepts/traceability.md) — trace tags, `.tsv`, drift, `dev_selftest` vs `qc_status`
- [Architecture](02-concepts/architecture.md) — 3 lớp, context-loader, module
- ⭐ [**Pipeline Steps**](02-concepts/pipeline-steps/) — **chi tiết từng bước**: input · output · ai làm gì · câu hỏi · cách xử lý

### 03 · [Guides theo vai trò](03-guides/) — *Tôi làm gì ở mỗi bước*
- [Product Owner](03-guides/product-owner.md) · [Developer](03-guides/developer.md) · [Architect / SA](03-guides/architect.md) · [Tester / QA](03-guides/tester-qa.md) · [Dựng máy QC](03-guides/qc-machine-setup.md)

### 04 · [Reference](04-reference/) — *Tra cứu*
- [Commands](04-reference/commands.md) — bảng đầy đủ mọi slash command
- [Trace Schema](04-reference/trace-schema.md) · [Modules](04-reference/modules.md) · [Configuration](04-reference/configuration.md) · [Model Selection](04-reference/model-selection.md)

---

*Phiên bản framework mới nhất: xem trên [npm](https://www.npmjs.com/package/@educa-corp/sdd-framework) — `npm view @educa-corp/sdd-framework version`.*
