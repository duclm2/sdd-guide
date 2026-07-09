[← Docs Home](../README.md) · [Reference](./)

# Reference · Commands — Bảng đầy đủ (Command Catalog)

> 30 slash command, gom theo phase pipeline. Chi tiết cơ chế từng bước → [Pipeline Steps](../02-concepts/pipeline-steps/).

Mọi lệnh chạy chung một **Gate** (model check → target → context-loader → checkpoint). Xem [Khung chung](../02-concepts/pipeline-steps/README.md#khung-chung-mọi-command-common-gate).

---

## 0 · Setup

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/setup-ai-first` | Dự án trống/có code | `.agent/`, `.claude/`, `project-context.yaml`, `CLAUDE.md` | Admin/Lead |

## 1 · Discovery

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/define-product` | Ý tưởng (+Figma) | `product-definition/*.md` (8 chặng) | PO |

## 2 · Specification (PRD)

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-prd` | product-definition | PRD draft | PO |
| `/refine-prd` | PRD | Findings 3 lăng kính DEV/SA/PO | PO+SA+Dev |
| `/review-context` (PRD) | PRD | Findings P0–P5 → `Status: approved` | PO |

## 3 · Design-Spec (chỉ FE/App)

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-design-spec` | PRD approved + Figma | `design-spec/*.md` | PO/PM |

## 4 · BDD

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-bdd` | PRD approved (+design-spec) | `bdd/**/*.feature` | PO |
| `/review-context` (BDD) | `.feature` | Findings B1–B6 → `@trace.status: approved` | PO/Dev |

## 5 · Tech-Docs

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-tech-docs` | BDD approved + entity catalog | `tech-docs/{TICKET}-tech-design.md` | SA |
| `/review-tech-docs` | tech-design | Findings đa chiều + ký T7 | SA/Lead |

## 6 · Code

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/generate-code` | `.feature` approved + tech-design | Code + `.trace/*.tsv` | Dev |
| `/review-code` | Code | Findings (read-only) | Dev/Lead |
| `/fix-bug` | Bug report | Fix + regression test | Dev |
| `/map-testids` | UI code | testid map (FE) | Dev |
| `/debug` | Mô tả lỗi | Phân tích (read-only) | Dev |

## 7 · Dev self-test

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/dev-gen-test` | Code + `.feature` | Bộ self-test | Dev |
| `/dev-run-test` | Self-test | Kết quả + cột `dev_selftest` | Dev |
| `/dev-smoke-test` | Service/app đang chạy | Kết quả smoke tại chỗ | Dev |

## 8 · QC Automation

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/qc-analyze` | UC + spec | `REQUIREMENT_ANALYSIS.md`, `DOC_GAPS.md` | QA |
| `/qc-plan` | Analysis | `TEST_PLAN.md` (rủi ro) | QA |
| `/qc-design-test` | Plan | `test-cases/*.Test.md` | QA |
| `/qc-review` | Test case/script | 🛑 Cổng review | QA |
| `/qc-run-test` | `.Test.md` reviewed | Script Playwright + `qc_status` | QA |
| `/qc-report` | Kết quả run | Report + evidence + product-gap | QA |

## 9 · Quality & Trace

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/validate-traces` | `.trace/*.tsv` + spec/code/test | Ma trận coverage (read-only) | Dev/QA/Lead |
| `/generate-spec-manifest` | Specs | Mục lục spec cho agent ngoài | SA/Lead |

## 10 · Feedback & Ops

| Lệnh | Input | Output | Owner |
|------|-------|--------|-------|
| `/report-bug` | Lỗi phát hiện | `feedback/bug-reports/` (spec-anchored) | Tester/QC |
| `/propose-scenario` | Scenario thiếu | `feedback/bdd-proposals/` | Tester/QC |
| `/learn` | Định hướng lặp lại | `project-lessons.md` | Tất cả |
| `/sync` | (umbrella) | Pull + submodule + nổi feedback + Living Docs | Lead |
| `/update-framework` | — | Sync bản npm mới | Lead |

---

## Ký hiệu trạng thái trong report

- 🛑 checkpoint (dừng, chờ `Y`) · 🔒 gate trạng thái (chặn downstream) · ⚪ read-only
- ⚠️ cảnh báo mềm (không chặn) · ✅ pass · ❌ fail

→ [Trace Schema](trace-schema.md) · [Modules](modules.md) · [Configuration](configuration.md) · [Model Selection](model-selection.md)
