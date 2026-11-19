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
  specs_dir: "specs"                 # gốc mọi spec artifact (PRD · BDD · tech-docs · design-spec)
  src_dir: "src"                     # gốc mã nguồn — nơi /generate-code đặt & quét code (nguồn chính cho FE, phạm vi quét reuse)
  refinement_dir: ".agent/review"    # findings review
  qc_dir: "qc"                       # working docs của QC — {qc_dir}/{TICKET-ID}-{prd-slug}/{platform}/ (trước 2026-09-24: "docs")
  qc_input_docs_dir: "qc/input-docs" # tài liệu THAM KHẢO của QC — đọc để hiểu, KHÔNG làm bằng chứng gap
  qc_automation_dir: "qc-automation" # gốc script: web/ · mobile/ · system/ (bố cục trong: modules/qc-*/stack-profile.yaml §layout)
  qc_skills_dir: ".agent/skills/qc"  # nơi qc-* nạp skill (override sang repo QC riêng)
  product_definitions_dir: "specs/product-definition"
  domain_knowledge_dir: "specs/domain-knowledge"
  business_dictionary: "specs/domain-knowledge/business-dictionary.md"
  core_entities: "specs/domain-knowledge/core-entities.md"
  lessons_file: "specs/domain-knowledge/lessons-learned.md"   # /learn ghi; umbrella/service: ".agent/project-lessons.md"
  tech_docs_dir: "specs"             # tech-docs nằm trong feature-package: specs/{domain}/{prd-slug}/tech-docs/
  trace_dir: ".trace"                # trace state .tsv
  bug_reports_dir: "feedback/bug-reports"                # /report-bug
  bdd_proposals_dir: "feedback/bdd-proposals"            # /propose-scenario Case A
  prd_change_requests_dir: "feedback/prd-change-requests" # /propose-scenario Case B

tech_stack:
  language: "..."                    # vd Java 17 / TypeScript
  framework: "..."                   # vd Spring Boot 3.2
  build_tool: "..."
  test_framework: "..."
  database: "..."                    # vd PostgreSQL
  module: "..."                      # module stack, vd java-spring / nextjs / context-engineering

conventions:
  build_command: "..."               # lệnh build verify (≤3 retry)
  test_command: "..."
  service_run: "..."                 # lệnh chạy service
  ticket_prefix: "..."

domains:
  - "..."

setup:                               # (chỉ umbrella — để comment ở single-service)
  mode: single | umbrella
  spec_source: "..."                 # spec repo dùng chung

services:                            # (multi/umbrella) map domain → service
  # dạng phẳng: {domain}: { path, module }
  # dạng map-theo-platform: {domain}: { web: {path,module}, app: {...}, system: {...}, webview: {...} }
  # dạng theo prd-slug: {domain hoặc platform}: { by_prd_slug: { {prd-slug}: {path,module} } }

architecture:
  style: "..."                       # vd Layered / Clean / Hexagonal
  layers: "..."                      # vd Controller → Facade → Service → Repository
  key_rules: ["..."]
```

### Bố cục feature-package (path resolution)

Mọi artifact của một feature nằm chung `specs/{domain}/{prd-slug}/`:

```
specs/{domain}/{prd-slug}/
├── {TICKET-ID}-{prd-slug}.md          # PRD (file .md duy nhất ở gốc)
├── bdd/{web|app|system|webview}/*.feature   # BDD — subfolder platform LUÔN có
├── tech-docs/{TICKET-ID}-tech-design.md   # MỘT doc gộp cho cả PRD
├── design-spec/{TICKET-ID}*.md        # FE/App
└── changelog/{TICKET-ID}-{prd-slug}.changelog.md   # chỉ tạo khi changelog vượt 5 version
.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv   # MỘT sổ / UC × platform (24 cột)
.trace/{domain}/{prd-slug}/_seams.tsv               # sổ seam/stub — nguồn 2 cờ 🔴 chặn PR
.trace/trace-report.json                            # snapshot cho panel — GHI ĐÈ mỗi lần chạy
.trace/trace-history.jsonl                          # nhật ký append-only — PHẢI commit
```

- `{domain}` = segment đầu sau `specs_dir`.
- `{prd-slug}` = segment kế tiếp (folder feature-package) — **không** phải folder cha trực tiếp của file lồng sâu.
- **Segment `{platform}` không optional** — mọi mode, kể cả umbrella. `web` và `system` của cùng một UC là hai file khác nhau; bỏ segment thì chúng va tên và **ghi đè nhau**. Trace tách theo platform nên bố cục spec phải khớp.
  *Project còn ở bố cục phẳng: `npx @educa-corp/sdd-framework --migrate-bdd-platform` (dry-run mặc định).*

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

## Vùng nào bị ghi đè khi update

`.agent/` là **mirror sinh ra**: `/update-framework` chạy `npx … --init`, và `--init` copy `core/` → `.agent/` **vô điều kiện**.

| Đường dẫn | Sửa được? | Vì sao |
|---|:---:|---|
| `CLAUDE.md` | ✅ | ngoài `.agent/` |
| `.agent/project-context.yaml` | ✅ | file **duy nhất trong `.agent/`** có guard: chỉ tạo nếu chưa tồn tại |
| `.agent/project-lessons.md` · `.agent/review/` | ✅ | không nằm trong `core/` nên nâng cấp không đụng |
| `specs/domain-knowledge/` · `.trace/` · `feedback/` | ✅ | ngoài `.agent/` |
| `.agent/commands` `steps` `rules` `skills` `hooks` `templates` `modules` | ❌ | copy từ repo framework, **ghi đè mỗi lần nâng cấp** |

**Nếu bạn đã sửa gì trong vùng ❌** — từ v0.4.2 `--init` tự cứu:

- bản cũ copy sang `.agent/.overwritten-{version}-{YYYYMMDD}/` (giữ nguyên cây thư mục)
- danh sách file bị ghi đè in ra ngay sau bước cài
- `.agent/.install-manifest.json` ghi hash của **đúng những gì lần cài trước đã viết** — nhờ đó lệnh phân biệt được *bạn sửa file* với *framework tự đổi file giữa hai version* (một phép so nội dung thuần sẽ flag cả hai và báo oan hàng chục file mỗi lần nâng cấp). **Manifest phải được commit cùng `.agent/`** — ignore nó thì máy đồng đội không gỡ được lệnh framework đã bỏ; `--init` cảnh báo nếu thấy nó bị ignore

Thêm vào `.gitignore` của project:

```gitignore
.agent/.overwritten-*/
.trace-mirror/          # bản sao cho panel VS Code — sinh lại được
.living-docs/           # report gộp — sinh lại được  (đặt trong .gitignore của spec repo)
```

> ### ⚠️ `.trace/` **KHÔNG** được gitignore
>
> Đây là chỗ dễ mất dữ liệu nhất trong toàn framework.
>
> | Đường dẫn | Vai trò | Git |
> |---|---|---|
> | `{paths.trace_dir}` (`.trace/`) | **AUTHORITATIVE** — `.tsv` + `trace-history.jsonl`. **Không regenerate được** | **PHẢI commit** |
> | `.trace-mirror/` | bản sao cho panel VS Code | gitignore |
> | `.living-docs/` | report gộp | gitignore |
>
> **Trước v0.4.3 cả hai cùng tên `.trace`.** Khi dev mở thẳng spec repo làm workspace thì hai đường dẫn **bằng nhau** — và `/sync` (bản cũ) gợi ý gitignore theo **tên**, không theo vai trò. Làm theo là **mất sổ gốc — lỗi ngầm**: máy vẫn chạy, dashboard vẫn có số; chỉ người thứ hai clone về mới phát hiện, và lúc đó lịch sử đã mất vĩnh viễn.
>
> **Kiểm dự án đang chạy:**
> ```bash
> git -C {spec_source} check-ignore .trace    # trả về kết quả = ĐANG DÍNH
> ```
> Dính thì gỡ dòng khớp `.trace` khỏi `.gitignore` rồi `git add -f .trace && git commit`.
> Từ v0.4.3 `/sync` Step 4b tự kiểm và báo động 🔴 nếu phát hiện.

> Muốn thay đổi **bền vững** thì sửa trong repo framework rồi phát hành. Tuỳ biến riêng của một project thuộc về `CLAUDE.md`, `.agent/project-context.yaml`, hoặc `.agent/project-lessons.md` — cả ba đều không bị ghi đè. Ranh giới đầy đủ: `.agent/README.md`.
>
> **Template `.feature`/PRD không cấu hình được.** Skeleton được `{{include}}` nướng cứng vào file lệnh lúc build, nên sửa `.agent/templates/` **không có tác dụng** (và sẽ bị ghi đè). Đổi cấu trúc artifact = sửa `templates/*` trong repo framework rồi `npm run build`. Chi tiết: `.agent/templates/README.md`.

---

→ [Architecture › Configuration](../02-concepts/architecture.md#configuration-2-file) · [Modules](modules.md)
