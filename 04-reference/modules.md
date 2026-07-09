[← Reference](./) · [Concepts › Architecture](../02-concepts/architecture.md)

# Reference · Modules — Stack overlay

> Module = overlay tuỳ chọn mở rộng base. Skill **đọc `stack-profile.yaml`** thay vì hardcode → đổi stack chỉ cần thêm module.

---

## Module có sẵn (15)

### Tech stack — Backend

| Module | Stack |
|--------|-------|
| `java-spring` | Java + Spring Boot |
| `dotnet` | .NET / C# |
| `golang` | Go |
| `php-laravel` | PHP + Laravel |

### Tech stack — Frontend / Mobile

| Module | Stack |
|--------|-------|
| `react` | React |
| `vue` | Vue |
| `angular` | Angular |
| `nextjs` | Next.js |
| `nuxt` | Nuxt |
| `react-native` | React Native |
| `flutter` | Flutter |
| `android-compose` | Android (Jetpack Compose) |
| `ios-swiftui` | iOS (SwiftUI) |

### Capability

| Module | Vai trò |
|--------|---------|
| `qc-playwright` | QC automation — Python + pytest-playwright + Page Object |
| `context-engineering` | Hỗ trợ context-loader |

---

## Stack profile contract

Mỗi stack module khai báo qua `stack-profile.yaml`:

| Khoá | Ý nghĩa |
|------|---------|
| `artifact_types[]` | Loại artifact stack này sinh |
| `generation_layers[]` | Thứ tự layer + dependency (vd Controller → Service → Repository) |
| `build_verify` | Lệnh build để verify (≤3 retry) |
| `test_types{}` | Các loại test hỗ trợ |

Cộng `module.yaml` khai báo metadata module.

---

## Cài & dùng

```bash
# Khi init
npx @educa-corp/sdd-framework --init --module java-spring
# Multi-service
npx @educa-corp/sdd-framework --init \
  --services backend:java-spring,web:react,app:flutter
```

Module nằm ở `.agent/modules/{name}/`. QC skill có thể trỏ ra repo riêng của team QC qua `paths.qc_skills_dir` để không bị ghi đè khi `/update-framework`.

> Ghi chú: bản hiện tại **không** có module jira/figma/locale tích hợp tự động; framework mặc định **ngôn ngữ Việt**; tham chiếu Jira chỉ là field Ticket trong Metadata PRD.

---

→ [Architecture › Module System](../02-concepts/architecture.md#module-system) · [Configuration](configuration.md)
