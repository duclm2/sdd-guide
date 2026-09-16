# 🔬 Explain — Giải phẫu từng command (Command Deep-Dive)

> Tài liệu **đi sâu vào bên trong** từng command của pipeline, theo đúng thứ tự thực thi. Mục tiêu: làm **cơ sở review, phân tích và tối ưu** các bước trong pipeline.
>
> Khác với [Pipeline Steps](../02-concepts/pipeline-steps/) (tầng khái niệm, gom theo phase), thư mục này bám **logic thực tế trong command file** (`commands/*.md`) — từng bước command làm gì, giải quyết vấn đề gì, và **điểm nào đáng cân nhắc tối ưu**.

---

## Cách đọc (How to read)

Mỗi trang command theo cùng một khuôn:

| Mục | Nội dung |
|-----|----------|
| **Một câu** | Command làm gì |
| **Vấn đề giải quyết** | Tại sao command này tồn tại |
| **Vị trí & tiền đề** | Chạy sau gì, cần gì mở khoá |
| **Input / Output** | Đầu vào & sản phẩm cụ thể |
| **Các bước xử lý** | ⭐ Đi từng bước bên trong, dễ hiểu |
| **Checkpoint & Gate** | Điểm dừng con người |
| **Cơ chế đặc biệt** | Phần logic riêng đáng chú ý |
| **👓 Góc nhìn tối ưu** | Điểm review/optimize: chi phí, rủi ro, phụ thuộc |
| **Kết nối** | Bước trước ← → bước sau |

---

## ⭐ Bộ khung chung mọi command (Shared Skeleton)

**Đọc phần này trước.** Mọi command file được build từ `.tmpl` + `{{include:steps/*.md}}`, nên đều có **cùng một bộ khung** bao quanh logic riêng. Hiểu bộ khung một lần → các trang command chỉ cần nói phần **riêng**.

Cấu trúc một command file:

```
┌─ Gate (steps/gate) ──────────── chung, giống hệt mọi lệnh
├─ Context Loader (steps/context-loader) ── chung, 7 bước
├─ Business Language Guard (steps/business-language) ── chung, chỉ lệnh viết doc nghiệp vụ
├─ ★ LOGIC RIÊNG CỦA LỆNH ★ ──── phần mỗi trang explain tập trung
└─ Report Footer (steps/report-footer) ── chung
```

### 1 · Gate — Cổng vào chuẩn (5 bước con)

Chạy **trước** mọi logic riêng:

| Bước | Tên | Việc | Ý nghĩa tối ưu |
|------|-----|------|----------------|
| 0 | **Sub-agent mode** | Nếu `$ARGUMENTS` là JSON có `_agent_mode` → bỏ Gate 1/2/3, chạy đúng phạm vi orchestrator giao (target_file, uc_id, uc_section, dimension) | Cơ chế fan-out per-UC dùng chính lệnh này làm "worker" |
| 0-B | **Model check** | Khuyến nghị Opus. `Y`=tiếp · `S`=bỏ qua (⚠️ report) · khác=DỪNG | Checkpoint mềm; sub-agent bỏ qua (orchestrator đã check) |
| 1 | **Target file** | Phân giải file mục tiêu từ path / UC-ID / ticket bằng glob theo bố cục feature-package; nhiều kết quả → hỏi | Điểm hay tốn 1 vòng hỏi khi `$ARGUMENTS` rỗng |
| 2 | **Context loader** | Chạy 7 bước nạp context (mục 2 dưới) | Nơi quyết định "đúng-đủ-gọn" — trọng tâm tối ưu |
| 3 | **CHECKPOINT** | Trình target + scope → chờ `Y` | Read-only command bỏ qua |

### 2 · Context Loader — "Thủ thư" (7 bước)

Nạp context theo thứ tự chống Lost-in-the-Middle (đầu = "build gì", giữa = ràng buộc, cuối = "follow style này"):

| Bước | Nạp gì | Ghi chú |
|------|--------|---------|
| 1 | **project-context.yaml** | tech_stack, conventions, domains, paths; trích `domain`/`prd_slug` từ path target |
| 1.5 | **Service routing** (umbrella) | Khớp domain → service; dạng phẳng (2a) hay map-theo-platform (2b); override paths sang `spec_source` |
| 1.6 | **Service conventions** (umbrella) | Nạp `build_command`/`test_command` riêng của service; set `service_root` |
| 2 | **Module stack-profile** | `.agent/modules/{module}/stack-profile.yaml` — layer/test pattern |
| 3 | **CLAUDE.md phân tầng** | root (BASE) + service overlay (stack) — **overlay thắng**; §2 layer/package, §3 naming, §5 error |
| 4 | **data-protection** | Pattern file nhạy cảm — cấm truy cập cả phiên |
| 5 | **Business dictionary** | Canonical + **banned terms** (thực thi chủ động) + enum registry |
| 6 | **Core entities** | Entity catalog + field registry + relationship map |
| 6.5 | **platform_type** | Suy `backend`/`web-frontend`/`mobile` từ module |
| 6.7 | **Project lessons** | Guardrail từ `/learn` — ràng buộc cứng ngang coding standards |
| 7 | **Recap** | In khối `[CTX LOADED]` — đẩy sự thật quan trọng lên cuối bộ nhớ |

> 👓 **Đây là component quyết định 80% chất lượng.** Khi review tối ưu: chú ý `required` vs `optional`, filter theo domain, và budget context (~50% window).

### 3 · Business Language Guard (chỉ lệnh viết doc nghiệp vụ)

Chặn thuật ngữ kỹ thuật rò vào PRD/BDD/product-definition. 4 nhóm xử lý: (1) tương tác/UI → diễn đạt lại nghiệp vụ · (2) visual thuần → chuyển Design Spec · (3) backend/contract → bỏ về Tech Docs · (4) ẩn dụ dữ liệu → xét ngữ cảnh (không thay máy móc). Áp cho `/define-product`, `/generate-prd`, `/refine-prd`, `/review-context`, `/generate-bdd`.

### 4 · Report Footer (mọi lệnh)

Kết thúc bằng: **Status badge** (✅/❌/⚠️) · **Output Artifacts** (file tạo/sửa) · **Pipeline Position** (`◀ bạn ở đây`) · **Next command** (gợi ý lệnh kế + tham số).

---

## Danh sách command theo thứ tự pipeline (Pipeline Order)

### Phase Setup & Discovery
- [00 · `/setup-ai-first`](00-setup-ai-first.md)
- [00b · `/generate-architecture`](00b-generate-architecture.md)
- [01 · `/define-product`](01-define-product.md)

### Phase Specification (PRD)
- [02 · `/generate-prd`](02-generate-prd.md)
- [02b · `/extend-prd`](02b-extend-prd.md) — thêm yêu cầu vào PRD **đã duyệt**
- [02c · `/amend-prd`](02c-amend-prd.md) — **đổi** một yêu cầu đã duyệt (sửa tại chỗ)
- [03 · `/refine-prd`](03-refine-prd.md)
- [04 · `/review-context`](04-review-context.md) *(dùng cho cả PRD & BDD)*

### Phase Design
- [05 · `/generate-design-spec`](05-generate-design-spec.md)
- [06 · `/generate-bdd`](06-generate-bdd.md)
- [07 · `/generate-tech-docs`](07-generate-tech-docs.md)
- [**11** · `/map-testids`](11-map-testids.md) ← **chốt hợp đồng test-id, TRƯỚC code**
- [08 · `/review-tech-docs`](08-review-tech-docs.md)

> Danh sách này xếp theo **thứ tự chạy**, không theo số file. `/map-testids` giữ số `11` vì lý do lịch sử — giữ tên file để không gãy link cũ.
> §4.5.6 chốt ở đây là lý do **Phase Implementation** và **Phase QC Automation** chạy được **song song**.

### Phase Implementation
- [09 · `/generate-code`](09-generate-code.md)
- [10 · `/review-code`](10-review-code.md)

### Phase Dev Self-Test
- [12 · `/dev-gen-test`](12-dev-gen-test.md)
- [13 · `/dev-run-test`](13-dev-run-test.md)
- [14 · `/dev-smoke-test`](14-dev-smoke-test.md)

### Phase QC Automation
- [15 · `/qc-analyze`](15-qc-analyze.md)
- [16 · `/qc-plan`](16-qc-plan.md)
- [17 · `/qc-design-test`](17-qc-design-test.md)
- [18 · `/qc-review-testcase` + `/qc-review-script`](18-qc-review.md)
- [19 · `/qc-design-script` · `/qc-run-script` · `/qc-run-manualtest`](19-qc-run-test.md)
- [20 · `/qc-report`](20-qc-report.md)

### Phase Trace & Quality
- [21 · `/validate-traces`](21-validate-traces.md)
- [22 · `/generate-spec-manifest`](22-generate-spec-manifest.md)

### Lệnh xuyên suốt (Cross-cutting)
- [23 · `/fix-bug`](23-fix-bug.md)
- [24 · `/debug`](24-debug.md)
- [25 · `/report-bug`](25-report-bug.md)
- [26 · `/propose-scenario`](26-propose-scenario.md)
- [27 · `/learn`](27-learn.md)
- [28 · `/sync`](28-sync.md)
- [29 · `/update-framework`](29-update-framework.md)

---

*Nguồn: `commands/*.md` (build từ `.tmpl` + `steps/`). Khi command đổi, cập nhật trang tương ứng ở đây.*
