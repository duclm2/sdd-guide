[← /dev-gen-test](12-dev-gen-test.md) · [Explain Home](README.md) · [Next: /dev-smoke-test →](14-dev-smoke-test.md)

# 13 · `/dev-run-test` — Chạy self-test + chẩn lỗi, ghi `dev_selftest`

> **Một câu.** Chạy bộ self-test theo lệnh test của platform, chẩn đoán lỗi, và ghi kết quả vào cột **`dev_selftest`** của trace.

---

## Vấn đề giải quyết

Sinh test chưa đủ — phải chạy và biết pass/fail. Command chạy test đúng service/platform, phân tích lỗi, và ghi trạng thái smoke của dev vào trace (độc lập với QC).

---

## Vị trí & tiền đề

- **Vị trí:** Phase Dev Self-Test, sau `/dev-gen-test`.
- **Đặc biệt:** chạy lệnh shell **bên trong** `service_root` (umbrella).

---

## Input / Output

**Input:** bộ self-test + `conventions.test_command` (theo service) + `platform_type`.

**Output:** cột `dev_selftest` trong `.tsv` + Panel Mirror; nếu fail → chẩn đoán.

---

## Các bước xử lý (chi tiết)

1. **Service Detection** + **Submodule Working Directory** — xác định thư mục chạy.
2. **Run** — chạy test theo platform (scoped để feedback nhanh):
   | platform_type | Lệnh (ví dụ) |
   |---------------|--------------|
   | backend | java-spring (mvn), golang (go test), dotnet, php-laravel, pytest |
   | web-frontend | Vitest / Jest; E2E Playwright / Cypress |
   | mobile | Flutter test, React Native, iOS (xcodebuild), Android |
3. **Analyze Failures** — chẩn lỗi theo platform (điều gì fail, tại sao).
4. **Write Trace State** — **ĐỌC cột `status` của row TRƯỚC KHI GHI**, rồi set `dev_selftest`:

   | `status` của row | Ghi gì |
   |---|---|
   | `OK` · `GAP` · `UNTRACKED` | `pass` / `fail` như thường |
   | **`DRIFT`** | test **pass** → **`not_run`** *(KHÔNG ghi `pass`)* · test **fail** → **`fail`** như thường |
   | **`ORPHANED`** | **`not_run`** — scenario đã bị xoá khỏi `.feature` |

   > **Vì sao (GAPS-v4 G55).** `pass` **không** mang nghĩa *"test đã chạy và xanh"* — nó mang nghĩa *"scenario này đã được nghiệm thu theo spec **hiện tại**"*. Trên row `DRIFT` nghĩa thứ nhất đúng và nghĩa thứ hai **sai**.
   >
   > Bản cũ ghi `pass` chỉ dựa vào *test có xanh không*, nên chuỗi này báo xanh sai: PO đổi AC → `/generate-bdd` đặt `status = DRIFT` và **hạ** `dev_selftest → not_run` → sáng sau dev chạy lệnh này theo thói quen (chưa `/generate-code`, chưa `/dev-gen-test`) → test cũ + code cũ xanh hết → ghi `pass` **kèm ngày hôm nay**. Tức lệnh kế tiếp trong vòng lặp dev bình thường **dựng lại** đúng tín hiệu vừa bị hạ.
   >
   > **Tin xấu luôn hợp lệ:** đây là guard cho lời khẳng định **DƯƠNG**, không phải lệnh *"bỏ qua kết quả khi DRIFT"*. `fail` vẫn được ghi — chặn cả `fail` là biến guard chống-báo-cáo-sai thành guard **che tin xấu**.
   >
   > **Tầng thứ hai độc lập:** `lint-trace` **T12** bắt trạng thái này ở sổ thật (`status ∈ {DRIFT, ORPHANED}` mà `dev_selftest`/`qc_status = pass`), bất kể lệnh nào ghi ra — kể cả sổ sửa tay hoặc sổ sinh bởi version framework cũ.
5. **Refresh Panel Mirror** — Living Docs local (umbrella).

---

## Checkpoint & Gate

- Không gate chặn. Fail → gợi ý `/fix-bug` hoặc `/debug`.

---

## Cơ chế đặc biệt

- **Scoped run** — chạy hẹp (theo class) cho feedback nhanh thay vì toàn bộ suite.
- **`dev_selftest` là trục riêng** — không đụng `qc_status`.
- **Chẩn lỗi theo platform** — dùng `platform_type` để đọc lỗi đúng ngữ cảnh stack.

---

## 👓 Góc nhìn tối ưu

- **Bảng lệnh test theo stack lớn** — mỗi stack một dòng; thêm stack mới phải sửa ở đây (dù stack-profile đã có `test_command`). Có thể đọc hoàn toàn từ profile.
- **Kết quả `dev_selftest`** không chặn pipeline — dev có thể bỏ qua fail và đi tiếp. Đáng cân nhắc có nên cảnh báo mạnh hơn trước QC.
- **Chẩn lỗi phụ thuộc AI đọc log** — với lỗi mơ hồ có thể cần `/debug`.

---

## Kết nối

**Trước:** [`/dev-gen-test`](12-dev-gen-test.md) · **Sau:** pass → [`/review-code`](10-review-code.md) / [`/dev-smoke-test`](14-dev-smoke-test.md); fail → [`/fix-bug`](23-fix-bug.md) / [`/debug`](24-debug.md).
