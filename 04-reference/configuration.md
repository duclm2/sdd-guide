[← Reference](./) · [Concepts › Architecture](../02-concepts/architecture.md)

# Reference · Configuration

> Hai file cấu hình chính. `/update-framework` **không** đụng chúng — đây là nội dung của bạn.

---

## 1. `project-context.yaml`

**Single source of truth** cho paths/mode/routing. Nằm ở `.agent/project-context.yaml`, sinh từ `templates/project-context.yaml` qua token substitution khi init. Mọi workflow đọc file này để biết **WHERE** tìm mọi thứ.

### Cấu trúc chính

```yaml
project:
  name: "..."
  description: "..."

paths:
  specs_dir: "specs"                 # gốc mọi spec artifact
  trace_dir: ".trace"                # trace state .tsv
  qc_dir: "docs"                     # working docs của QC
  qc_skills_dir: ".agent/skills/qc"  # nơi qc-* nạp skill (override sang repo QC riêng)
  refinement_dir: ".agent/review"    # findings review

setup:
  mode: single | umbrella
  spec_source: "..."                 # (umbrella) spec repo dùng chung

services:                            # (multi/umbrella) map domain → service
  # dạng phẳng: {domain}: { path, module }
  # dạng map-theo-platform: {domain}: { web: {path,module}, app: {...}, system: {...} }

conventions:
  build_command: "..."               # lệnh build verify (≤3 retry)
  test_command: "..."

tech_stack:
  database: "..."                    # vd PostgreSQL
```

### Bố cục feature-package (path resolution)

Mọi artifact của một feature nằm chung `specs/{domain}/{prd-slug}/`:

```
specs/{domain}/{prd-slug}/
├── {TICKET-ID}-{prd-slug}.md          # PRD (file .md duy nhất ở gốc)
├── bdd/{web|app|system}/*.feature     # BDD
├── tech-docs/{TICKET-ID}-tech-design.md
└── design-spec/{TICKET-ID}*.md        # FE/App
.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv
```

- `{domain}` = segment đầu sau `specs_dir`.
- `{prd-slug}` = segment kế tiếp (folder feature-package) — **không** phải folder cha trực tiếp của file lồng sâu.

---

## 2. `CLAUDE.md` (phân tầng)

Instruction cho AI agent — AI *follow* file này thay vì *invent*.

| Tầng | Nội dung |
|------|----------|
| **root** `CLAUDE.md` | Luật chung dự án/umbrella |
| **overlay** `{service}/CLAUDE.md` | §2 kiến trúc & thứ tự layer & package strategy · §3 coding standards · §5 error handling — **theo stack** |

> Service overlay **thắng** root khi mâu thuẫn. context-loader nạp cả hai, đặt ở **cuối** context (Lost-in-the-Middle: "follow style này").

---

## 3. Single vs Umbrella

| | Single | Umbrella |
|---|--------|----------|
| `setup.mode` | `single` | `umbrella` |
| Spec ở đâu | Cùng repo | **Spec repo dùng chung** (`spec_source`) |
| Code ở đâu | Cùng repo | Service submodule (chỉ code + tooling) |
| Trace/feedback/findings | Cùng repo | Trong spec repo (một nơi authoritative) |
| Đồng bộ | — | `/sync` (pull + submodule + nổi feedback + Living Docs) |

Khi `spec_source` được đặt, **mọi** PRD/BDD/tech-doc/design-spec/`.trace`/`.agent/review`/`feedback` route về spec repo — vì đều là **artifact liên team**.

---

## Không bị ghi đè khi update

`/update-framework` ghi đè `.agent/commands|steps|hooks|rules|templates|skills|modules` nhưng **giữ nguyên**: `CLAUDE.md`, `project-context.yaml`, `domain-knowledge/`, `.trace/`.

---

→ [Architecture › Configuration](../02-concepts/architecture.md#configuration-2-file) · [Modules](modules.md)
