[← What is SDD](what-is-sdd.md) · [Getting Started](../README.md) · [Next: Quick Start →](quickstart.md)

# Installation — Cài đặt & khởi tạo

> Yêu cầu: [Node.js](https://nodejs.org) (kiểm tra `node -v`) và [Claude Code](https://claude.com/claude-code).

---

## 1. Khởi tạo dự án (Init)

**Single-service** (một repo, một stack):

```bash
npx @educa-corp/sdd-framework --init --module java-spring
```

**Multi-service monorepo** (nhiều service, nhiều stack):

```bash
npx @educa-corp/sdd-framework --init \
  --services backend:java-spring,web-admin:react,app-mobile:flutter
```

**Umbrella** (nhiều service submodule + một spec repo dùng chung):

```bash
npx @educa-corp/sdd-framework --init --umbrella
```

Lệnh init sẽ:
1. Copy framework vào `.agent/` (`commands|steps|rules|hooks|templates|skills|modules`).
2. Deploy cấu hình Claude Code vào `.claude/`.
3. Sinh `.agent/project-context.yaml` từ template (single/umbrella).
4. Cài các **module stack** đã chọn.
5. Tạo `CLAUDE.md` (root; thêm overlay theo service khi umbrella).

---

## 2. Setup trong Claude Code

Mở dự án trong Claude Code rồi chạy:

```
/setup-ai-first
```

Lệnh này dẫn qua setup một-lần: tạo thư mục, cài `CLAUDE.md`, verify môi trường, chọn mode & module. Xem chi tiết cơ chế ở [Pipeline › Setup](../02-concepts/pipeline-steps/00-setup.md).

---

## 3. Điền config (bắt buộc trước feature đầu tiên)

AI *follow* các file này — để trống thì nó *đoán*:

| File | Điền gì |
|------|---------|
| `CLAUDE.md` (root) | Luật chung dự án |
| `{service}/CLAUDE.md` (overlay) | §2 kiến trúc & thứ tự layer · §3 coding standards · §5 error handling |
| `.agent/project-context.yaml` | paths, mode, services, `conventions.build_command`, tech_stack |
| `specs/domain-knowledge/` | Business dictionary, entity catalog, tri thức miền |

→ Chi tiết: [Reference › Configuration](../04-reference/configuration.md).

---

## 4. Bắt đầu feature đầu tiên

```
/define-product
```

→ Theo [Quick Start](quickstart.md).

---

## Nâng cấp & đồng bộ (Upgrade & Sync)

| Lệnh | Việc | Ghi đè | KHÔNG đụng |
|------|------|--------|-----------|
| `/update-framework` | Sync bản npm mới | `.agent/commands\|steps\|hooks\|rules\|templates\|skills\|modules` | `CLAUDE.md`, `project-context.yaml`, `domain-knowledge/`, `.trace/` |
| `/sync` (umbrella) | Pull + init submodule + nổi feedback + Living Docs | — | — |

Kiểm tra version mới nhất: `npm view @educa-corp/sdd-framework version`.

> **Quy tắc edit (critical):** `.tmpl` + `steps/` là source of truth. Sửa framework artifact phải sửa `.tmpl`/`steps` rồi `node bin/build.js`. Sửa thẳng `.agent/commands/*.md` sẽ **mất** khi rebuild/update.

---

## Đọc tiếp (Next)

➡️ [Quick Start — feature đầu tiên end-to-end](quickstart.md)
