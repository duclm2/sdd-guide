[← Reference](./) · [Concepts › Architecture](../02-concepts/architecture.md)

# Reference · Modules — Stack overlay

> Module = overlay tuỳ chọn mở rộng base. Skill **đọc `stack-profile.yaml`** thay vì hardcode → đổi stack chỉ cần thêm module.

---

## Module có sẵn (17)

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
| `phaser-game` | Game HTML5 — Phaser 3 + TypeScript + Vite, đóng gói cho Android WebView |

### Capability

| Module | Vai trò |
|--------|---------|
| `qc-playwright-ts` | QC automation web + API — Playwright Test + TypeScript (`layout.web` · `layout.system`) |
| `qc-wdio-appium` | QC automation app — WebdriverIO v9 + Appium v2 + UiAutomator2, TypeScript |
| `context-engineering` | Thiết kế context cho AI (system prompt, tối ưu context, tiêu chí đánh giá) — `stack_type: ai-first`; dùng làm module của service backend AI (làn DEV) |

---

## Stack profile contract

Mỗi stack module khai báo qua `stack-profile.yaml`:

| Khoá | Ý nghĩa |
|------|---------|
| `build` | Lệnh `compile` · `test` · `run` của stack (build verify ≤3 retry) |
| `architecture` | `style` + `key_rules` của kiến trúc |
| `coding_standards` | Quy ước đặt tên + pattern |
| `testing` | Framework/loại test hỗ trợ |
| `trace_tags` | Mẫu tag `@trace.*` gắn vào code/test của stack |

Module QC (`qc-*`) khai khác: `versions` · `build` · `layout` (cây thư mục dưới `{paths.qc_automation_dir}`) · `naming` · `locator_priority` · `reporting` · `trace_tags`; `qc-wdio-appium` thêm `driver` · `api_helper` · `gesture_helper` · `environment_validation`. `context-engineering` không phải stack code nên dùng bộ khoá riêng (`context_types` · `context_optimization` · `evaluation_criteria` · `common_patterns`).

Cộng `module.yaml` khai báo metadata module (`name` · `description` · `language` · `framework` · `stack_type` · `default_layer_order` — thứ tự layer khi sinh code · `test_framework`).

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
