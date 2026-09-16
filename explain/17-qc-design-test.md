[← /qc-plan](16-qc-plan.md) · [Explain Home](README.md) · [Next: hai cổng review →](18-qc-review.md)

# 17 · `/qc-design-test` — Trạm 3: Thiết kế test case (Markdown)

> **Một câu.** Thiết kế **test case dạng Markdown** (`.Test.md`) từ plan — chưa sinh Python; script đến sau ở `/qc-design-script`.

---

## Vấn đề giải quyết

Tách "thiết kế test case" (con người đọc/review được) khỏi "code test" (máy chạy). `.Test.md` là bản thiết kế mà `/qc-review` duyệt và `/qc-design-script` biến thành script.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 3), sau `/qc-plan`.
- **Không cần code.** Trạm này chỉ cần spec + **hợp đồng test-id §4.5.6** (đã chốt ở `/map-testids`, trước `/generate-code`). Nên nó chạy **song song với FE**.

---

## Input / Output

**Input:** TEST_PLAN.md + skill `qa-designer` (chọn layer, nạp MỘT file) + `.feature` (nguồn scenario cho Guard) + **§4.5.6 Test Selectors**.

**Output:** `{qc_dir}/{TICKET-ID}/{platform}/test-cases/*.Test.md` (trace mapping bắt buộc). Gọi theo **từng UC**, ghi vào thư mục test-case dùng chung cấp PRD.

---

## Các bước xử lý (chi tiết)

1. **Role qa-designer** — chọn layer, nạp một file skill (không nạp cả bộ → tiết kiệm context).
2. **Conventions** — theo quy ước test-case của QC team.
3. **Trace mapping (bắt buộc)** — mỗi test case ↔ SC.
4. **Guard — SC coverage** *(mới)* — phép **đếm cơ học**, không phải lượt đọc lại:

   ```
   1. Thu SC — mọi @trace.scenario trong .feature của uc_list (đúng nền).
               LOẠI SC của UC mà qc-scope đã lọc ra (⏸ BDD chưa approved).
   2. Thu V  — mọi @trace.verifies trong các file *.Test.md VỪA GHI.
   3. Đếm    — mỗi SC ∈ tập bước 1 phải có ≥ 1 TC trỏ tới.
   ```

   | Kết quả | Hành vi |
   |---|---|
   | Mọi SC đều có ≥1 TC | in `Guard SC coverage: khớp {K}/{K}` |
   | Có SC chưa phủ | **viết bù TC ngay**, rồi in `⚠️ Guard SC coverage: {K}/{total} — đã bù {m} SC: {…}` |

   **In dòng này kể cả khi sạch** — guard im lặng khi sạch là guard không ai biết nó tồn tại, nên cũng không ai phát hiện khi nó hỏng.

5. **Self-Review** — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
6. **Output** `.Test.md`.

---

## Checkpoint & Gate

- Không gate chặn người ở đây; gate là `/qc-review-testcase` kế tiếp.
- Nhưng Guard SC coverage **có hệ quả bắt buộc**: thiếu thì lệnh **viết bù** rồi mới báo, không phải chỉ cảnh báo.

---

## Cơ chế đặc biệt

- **Markdown-first** — test case là tài liệu review được trước khi thành code.
- **Nạp MỘT file skill theo layer** — tối ưu context (chỉ layer cần).
- **Trace mapping** — giữ test case gắn SC để về sau ghi `qc_status` đúng.
- **Guard SC coverage KHÔNG có đường thoát.** SC bị gap chặn **vẫn** phải có TC — viết đủ, mang dấu `🚫 Block: [GAP-UC{N}-{nnn}] — lý do`. Gap là thứ được **ghi vào** test case, không phải cái cớ để không viết. Cho phép "bỏ trống có lý do" là dạy hai luật trái nhau trong cùng một lệnh, và cho agent một câu-lý-do-cho-qua.
- **Guard ≠ Self-Review.** Guard là phép **đếm**, có hệ quả bắt buộc. Self-review là lượt đọc lại rộng hơn nhưng mềm hơn. Không bao giờ dùng self-review làm lý do gỡ Guard.

---

## 👓 Góc nhìn tối ưu

- **Tách design ↔ script** thêm một trạm nhưng cho phép review sớm — đánh đổi giữa số bước và chất lượng gate.
- **Phụ thuộc convention QC team** — cần đồng bộ khi convention đổi.
- **Guard đếm sớm, ở chỗ còn sửa được.** `/qc-report` cũng tính design coverage bằng **đúng phép đếm này**, nhưng ở **cuối** — lúc đó chỉ còn kịp báo, không còn kịp viết bù. Cùng một phép đếm, giá trị khác hẳn theo thời điểm.
- Con số `{K}/{total}` vốn **đã in** ở report từ trước, nhưng **không có phép tính nào đứng sau nó**. Một con số trong báo cáo không chứng minh có phép đo.

---

## Kết nối

**Trước:** [`/qc-plan`](16-qc-plan.md) · **Sau:** [`/qc-review-testcase`](18-qc-review.md) (review test case).
