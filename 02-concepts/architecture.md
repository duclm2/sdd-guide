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
.tmpl (source) + steps/*.md  --[node bin/build.js, {{include:...}}]-->  commands/*.md + core/
                                                                              │
                                                                              ▼
                                                              .agent/commands/*.md (runtime)
```

- Cơ chế `{{include:steps/...}}` ghép phẳng → single source of truth ở `.tmpl` + `steps/`.
- **Không sửa tay** `commands/*.md` / `.agent/` — sửa `.tmpl`/`steps` rồi `node bin/build.js`. *(Quy ước + memory bảo vệ, không phải hook.)*

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
