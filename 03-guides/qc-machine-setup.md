[← Docs Home](../README.md) · [Guides](./) · [Tester / QA](./tester-qa.md)

# Guide · Dựng máy QC *(làm một lần, khi có máy mới hoặc CI runner mới)*

> File này trả lời **"máy mới cần cài gì, theo thứ tự nào, vì sao"**. Cách **dùng** dây chuyền `/qc-*`
> ở [`tester-qa.md`](./tester-qa.md) — hai câu hỏi khác nhau.

---

## Luật đầu tiên — phiên bản KHÔNG nằm trong tài liệu này

Mọi con số phiên bản nằm trong **file khoá** của repo automation, không nằm trong văn xuôi:

| Công cụ | Khoá ở đâu |
|---|---|
| Node.js | `.nvmrc` ở gốc repo automation |
| Playwright · TypeScript | `package.json` + `package-lock.json` của nhánh `qc-automation/web/` và `qc-automation/system/` |
| WebdriverIO · Appium + driver (nền mobile) | `package.json` + `package-lock.json` của nhánh `qc-automation/mobile/` |

*`qc-automation/` là giá trị mặc định của `paths.qc_automation_dir` — dự án đổi key này thì đọc theo giá trị đã khai.*

**Vì sao** *(chị QC chốt 2026-09-24)*: số phiên bản đổi mỗi tháng, còn người viết tài liệu thì không ngồi
trên máy thật. Viết số vào đây thì tài liệu cũ đi mà không ai biết; để ở file khoá thì `npm ci` dùng đúng
nó, và đổi phiên bản là một commit có người review. *(Có test canh file này không chứa số phiên bản.)*

---

## Thứ tự cài — và vì sao theo thứ tự đó

1. **Trình quản lý phiên bản Node** (`nvm`, hoặc `fnm`/`volta`), rồi `nvm use` ở gốc repo — nó đọc `.nvmrc`.
   *Vì sao trước tiên:* mọi bước sau chạy trên Node; cài Node "bản mới nhất" rồi mới biết lệch là mất một buổi.
2. **`npm ci`** — không phải `npm install`. Chạy **trong từng nhánh** cần dùng (`web/` · `system/` · `mobile/` dưới `qc-automation/`) — mỗi nhánh có `package.json` riêng.
   *Vì sao:* `npm ci` cài **đúng** những gì `package-lock.json` ghi và dừng nếu lock lệch `package.json`;
   `npm install` có thể nâng phiên bản và sửa lock trên máy bạn — máy bạn chạy khác CI mà không ai thấy.
3. **Trình duyệt cho Playwright** (nền `web` · `system`): `npx playwright install --with-deps`.
   *Vì sao sau `npm ci`:* lệnh này tải đúng bản trình duyệt khớp Playwright **đã khoá** — chạy trước thì
   tải bản khớp một Playwright khác.
4. **Nền mobile:** cài Appium và driver theo lệnh mà nhánh `qc-automation/mobile/` khai *(script trong `package.json`)* —
   driver chọn theo module app: Flutter/React Native một driver, native Android/iOS một driver khác
   *(`qa-script-designer/_shared/mobile-conventions.md`)*.
5. **Kiểm cài đặt** — chạy một lần, phải qua cả hai:
   - web · system: `npx playwright test --list` (chạy trong nhánh tương ứng) liệt kê được test, không lỗi biên dịch
   - mobile: Appium server khởi động được, và `npx wdio run wdio.conf.ts --mochaOpts.dryRun` không lỗi (WDIO CLI không có cờ `--dry-run`)

---

## Phần do đội QC giữ — OS-level

Xcode *(nền iOS, bắt buộc macOS)* · Android SDK + emulator · thiết bị thật · device farm · CI runner ·
license công cụ. Phần này phụ thuộc máy và hợp đồng của đội, đổi theo hạ tầng — nên **đội QC giữ**, ở
tài liệu dựng máy của đội: `guide/CAI-DAT-MOI-TRUONG.md` trong bộ tài liệu đội QC.

Framework không chép lại phần đó: hai bản cùng tả một máy thì một bản sẽ cũ trước.
