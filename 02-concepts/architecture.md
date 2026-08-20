[← Traceability](traceability.md) · [Concepts](./) · [Pipeline Steps →](pipeline-steps/)

# Architecture — Kiến trúc kỹ thuật (Internals)

> Cách framework được tổ chức bên trong: 3 lớp, context-loader, template pipeline, module, hook. Đủ để hiểu *tại sao* framework hành xử như vậy.

---

## Mô hình 3 lớp (3-layer)

```
Command (orchestrator)   commands/*.md  ← build từ .tmpl
    ├── Skill            skills/{discovery,prd,spec,design-spec,code,test,qc,debug,setup-ai-first}
    └── Step             steps/*.md
```

| Lớp | Là gì | Ví dụ |
|-----|-------|-------|
| **Command** | Điều phối các phase của một workflow | `/generate-prd`, `/generate-code` |
| **Skill** | Domain logic, **stateless**, chỉ consume context | write-prd, write-bdd |
| **Step** | Infra tái dùng | `context-loader`, `gate`, `spawn-agent`, `review-fanout`, `business-language`, `capture-lesson`, `trace-mirror`, `report-footer` |

**Tại sao tách:** đổi pipeline không đụng skill; tái dùng skill không kéo infra; đổi infra không đụng domain. **Skill không gọi Step** — orchestrator ghép cả hai.

---

## Phase chuẩn của một Command

| Phase | Tên | Trách nhiệm |
|-------|-----|-------------|
| 0 | **Gate** | Validate input, model check mềm, service routing, nạp CLAUDE.md phân tầng |
| 1 | **Context** | `context-loader` nạp file theo thứ tự ưu tiên |
| 2 | **Execute** | Gọi skill — sinh artifact |
| 3 | **Reflect** | (review workflow) fan-out + completeness-critic |
| 4 | **Finalize** | Doc-sync, cập nhật trace, version bump |
| 5 | **Verify & Report** | Build verify (≤3 retry), report, gợi ý workflow kế |
| 6 | **Commit** | Commit theo convention (sau khi user duyệt) |

> Workflow read-only (`/review-*`, `/validate-traces`, `/debug`) bỏ qua Execute-write / Verify / Commit.

---

## Context-loader — "trái tim" của framework

Spec-driven thành/bại phụ thuộc **~80%** vào việc context được nạp **đúng — đủ — gọn — theo thứ tự**. Skill **không tự đọc file**; nó nhận một bundle đã sắp xếp từ `context-loader`.

**Lost-in-the-Middle** — LLM recall **đầu** và **cuối** tốt hơn **giữa**:

| Vị trí | Nội dung | Ý |
|--------|----------|---|
| **ĐẦU** | Input file (`.feature`/PRD), core entities (filtered), tech-doc section | "Build cái gì" |
| **GIỮA** | project-context, rules, business dictionary, domain knowledge, lessons | Constraints |
| **CUỐI** | CLAUDE.md (kiến trúc + coding standards), design-spec/UI | "Follow style này" |

**Nguyên tắc quản lý size:** Required vs Optional rõ ràng (thiếu Required → halt) · filter theo domain/scope · trim section không load file thô · service overlay (umbrella) · budget có ngưỡng · lazy-load qua subagent.

> Kim chỉ nam khi AI ra output lạ (sai pattern, lẫn UC khác, quên rule): **95% nguyên nhân ở context** — xem [Quick Start › Troubleshooting](../01-getting-started/quickstart.md#nếu-ai-ra-output-sai-troubleshooting).

---

## Template Pipeline

```
.tmpl + steps/*.md ──build──► commands/*.md ──► core/* ──► .agent/    904 KB
                                    │                        ▲
                     bin/self-check.js (fail build)      --init cài vào đây
```

**Vì sao slim (G45):** build inline `{{include:}}` vào **từng** file lệnh. Đo lúc đó (32 lệnh), kết quả là
2069 KB mà chỉ 580 KB là nội dung riêng của chúng — **72% là vài step giống hệt nhau, chép 30 lần**.
`/generate-code` từng nặng 108 KB (≈27k token đọc **trước** khi làm gì), gần một nửa không nói gì về
việc sinh code. Cái giá thật không phải tiền: trên PRD nhiều UC nó làm tăng rủi ro **cạn context
giữa lúc ghi sổ trace**.

| Step | Xử lý trong `core/` | Vì sao |
|---|---|---|
| `context-loader.md` (33 K × 29) | **đọc lúc chạy** | 64% lãng phí. Bỏ sót ⇒ lệnh **dừng ngay** vì thiếu path/config — hỏng ồn ào |
| `report-footer.md` (7 K × 32) | **đọc lúc chạy** | 16%. Bỏ sót ⇒ report kém cấu trúc, không hỏng gì |
| `gate.md` (7 K × 30) | **giữ inline** | Chỉ 15%, nhưng là **lưới an toàn** (model check · resolve target · CHECKPOINT). Bỏ sót ⇒ lệnh **vẫn chạy** mà không còn cổng nào — hỏng **âm thầm** |

Kết quả: `/generate-code` 108 KB → **69 KB**, `/refine-prd` 85 KB → **45 KB**.

> **Một biến thể duy nhất.** Bản đầu của G45 phải build **hai** bản: `commands/*.md` inline đầy đủ
> cho legacy mode (`--project` / không cờ) — vì nó copy thẳng file đó vào `.claude/commands/` mà
> **không** cài `.agent/`, nên không có `.agent/steps/` để đọc — và bản slim cho `--init`.
> **G50 gỡ hẳn legacy mode**, nên giờ mọi bản cài đều có `.agent/steps/` và nhánh build thứ hai
> biến mất. Hai nhánh build gần giống nhau là nợ chờ lệch.

### Tập publish — chỉ ship MỘT bản *(GAPS-v4 G59)*

`npm pack` chỉ mang **`bin/` · `core/` · `scripts/` · `docs/`**.

Trước G59, `files` có **10 mục**, và **7 trong 10** là bản sao của thứ đã có trong `core/` —
`commands/` (33/33 file `.md` **byte-identical** với `core/commands/`), cộng
`hooks/ modules/ rules/ skills/ steps/ templates/` (`diff -rq` không khác gì). Installer đọc
**chỉ `core/`** (`installCore(coreDir, agentDir, …)`), nên bản thứ hai không bao giờ được dùng.

| | Trước | Sau |
|---|---:|---:|
| Tarball nén | 1.2 MB | **710 kB** |
| Giải nén | 4.5 MB | **2.3 MB** |
| Số file | 395 | **215** |

**Nhưng cái đáng sửa hơn là một phép phân biệt bị vô hiệu.** `bin/index.js` dùng
`hasSources = exists(commands/generate-code.tmpl)` để biết *"đây là dev checkout hay bản cài từ
npm"*, và chú thích của nó viết thẳng: *"Chỉ nói trong DEV CHECKOUT. **Consumer không cần biết bước
này tồn tại**"*. Nhưng `commands/` được ship ⇒ `.tmpl` có mặt ⇒ phép thử **luôn đúng** ⇒ **mọi**
người dùng `npx` thấy `"Vừa sửa commands/*.tmpl ? Chạy npm run build trước"` — một câu họ không thể
làm gì với nó. *(Kiểm bằng cách `npm pack` rồi chạy tarball thật, không phải suy đoán.)*

Bốn nhánh sau khi sửa, `hasSources` đặt tên **một lần**:

| `corePrebuilt` | `hasSources` | Nghĩa | Làm gì |
|:---:|:---:|---|---|
| ✗ | ✓ | dev checkout, `core/` vắng/lệch | build từ nguồn |
| ✗ | ✗ | **bản cài npm bị thiếu/hỏng** | **lỗi rõ ràng + `exit 1`** — không cố build (G43: build ghi vào npx cache / global `node_modules`, có thể read-only, và hai `--init` song song sẽ đua nhau) |
| ✓ | ✓ | dev checkout, đã khớp version | in lời nhắc *"sửa `.tmpl` thì build lại"* |
| ✓ | ✗ | **bản cài npm, mọi thứ đúng** | **im lặng** ← đường của consumer |

Bất biến được `test/run.js` canh, viết theo **hình dạng** chứ không theo danh sách tên nên tự khớp
với dir thêm sau này: *không mục nào trong `files` được có bản mirror dưới `core/`*.

`scripts/` **phải giữ** — `bin/index.js` `require('../scripts/migrate-specs.js')` cho `--migrate-*`.

- Cơ chế `{{include:steps/...}}` → single source of truth ở `.tmpl` + `steps/`.
- **Không sửa tay** `commands/*.md` / `.agent/` — sửa `.tmpl`/`steps` rồi `node bin/build.js`. *(Quy ước + memory bảo vệ, không phải hook.)*
- **Sửa `steps/context-loader.md` hay `report-footer.md` giờ có hiệu lực NGAY** ở project đã cài — chúng được đọc lúc chạy, không còn phải build + publish + `/update-framework`.
- **Template artifact cũng bị inline lúc build.** `templates/feature.template` và `prd.template.md` được `{{include}}` **nướng cứng** vào file lệnh, nên lệnh không đọc path template lúc chạy — sửa `.agent/templates/` **không có tác dụng**. Đổi cấu trúc `.feature`/PRD = sửa `templates/*` trong repo framework rồi build lại.

### Self-check — contract trace không được lệch

```
bin/trace-schema.json  ──[bin/self-check.js]──>  đối chiếu commands/*.tmpl + steps/*.md
   (SoT máy đọc)                                 → exit 1 nếu lệch → build FAIL
```

**Vì sao cần:** phần lớn lỗi lịch sử của framework là **cùng một dạng** — contract (field `@trace.*`, cột `.tsv`, path pattern, giá trị enum) được phát biểu lại bằng **prose** ở nhiều lệnh rồi lệch nhau qua từng lần sửa, và không có gì phát hiện. Ca thuần khiết nhất: `@trace.sc_version` từng có **3 consumer, 0 producer** — nghĩa là `DRIFT` chết hoàn toàn — mà không lệnh nào, cổng nào, test nào bắt được.

| Rule | Bắt gì | Mức |
|:---:|---|:---:|
| R1 | field có consumer nhưng **không producer** | ERROR |
| R2 | field có producer nhưng **không ai đọc** (field chết) | WARN |
| R3 | actor khai trong schema mà file của nó **không nhắc** field | ERROR |
| R4 | `{paths.X}` dùng mà không khai / khai mà không ai dùng | ERROR / WARN |
| R5 | pattern bị cấm **quay lại** (vd `bdd/` thiếu `{platform}`) | ERROR |
| R6 | giá trị enum khai mà **không xuất hiện ở đâu** | WARN |

**Quy tắc vận hành:** đổi contract → sửa `bin/trace-schema.json` **TRƯỚC**, rồi mới sửa lệnh, rồi cập nhật [Trace Schema (người đọc)](../04-reference/trace-schema.md). Chạy riêng: `npm run self-check`.

*(Schema là JSON chứ không YAML vì `package.json` có **zero dependency** — npx chạy không cần install; thêm `js-yaml` chỉ để đọc một file build-time là không đáng. Nó nằm ở `bin/` chứ không `templates/` để không lọt vào `.agent/templates/` của mọi project.)*

---

## Configuration (2 file)

| File | Vai trò |
|------|---------|
| `.agent/project-context.yaml` | Runtime config — paths, mode (single/umbrella), services, conventions, tech_stack |
| `CLAUDE.md` | root (luật chung) + `{service}/CLAUDE.md` overlay (kiến trúc + coding standards theo stack) |

→ [Reference › Configuration](../04-reference/configuration.md).

---

## Module System

Module = overlay tuỳ chọn mở rộng base. Mỗi module khai báo `module.yaml` + **`stack-profile.yaml`** (`artifact_types[]`, `generation_layers[]`, `build_verify`, `test_types{}`). Skill **đọc profile** thay vì hardcode → đổi stack chỉ cần thêm module.

→ Danh sách 15 module: [Reference › Modules](../04-reference/modules.md).

---

## Hook System

Bản hiện tại có **một hook**:

| Event | Hook | Mục đích |
|-------|------|----------|
| PreToolUse (Read/Write/Edit/Bash) | `data-guard.js` | Chặn đọc/ghi/execute file nhạy cảm (secret, private key, `.env`) — exit 2 = hard block |

Bổ trợ bằng **rules** nạp vào context (`rules/data-protection.md`, `rules/workflow.md`), không phải hook.

---

## Đọc tiếp (Next)

- [Pipeline Steps](pipeline-steps/) — từng bước dùng các cơ chế này
- [Reference](../04-reference/) — schema, module, config chi tiết
