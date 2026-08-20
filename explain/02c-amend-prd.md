[← /extend-prd](02b-extend-prd.md) · [Explain Home](README.md) · [Next: /refine-prd →](03-refine-prd.md)

# 02c · `/amend-prd` — Đổi một yêu cầu đã duyệt

> **Một câu.** Sửa **tại chỗ** nội dung của một AC/BR/UC đã duyệt — PO khai tường minh ID, lệnh kiểm va chạm, ghi bằng `Edit` với guard **hai chiều**, bump version + `Status → draft`, và nói rõ UC nào phải làm lại.

---

## Vấn đề giải quyết

Có bốn tình huống PO chạm PRD. Trước GAPS-v4 G54 chỉ ba cái có lệnh.

| Tình huống | Lệnh | Có từ |
|---|---|---|
| PRD chưa có | `/generate-prd` | đầu |
| **Thêm** UC/AC/BR mới | `/extend-prd` | v0.4.3 |
| Sửa vấn đề **review chỉ ra** | `/refine-prd` | đầu |
| **Đổi một yêu cầu đang đúng cú pháp** | *(không có)* | — |

Nhánh thứ tư là *"BR8 nói tối đa 5 file, giờ đổi thành 20"*. Không thêm gì mới. Không phải AI phát hiện lỗi. Chỉ đổi một con số đã duyệt.

Ba lệnh có sẵn đều **từ chối đúng việc đó**:

| Lệnh | Vì sao không dùng được |
|---|---|
| `/generate-prd` | **DỪNG HẲN** — §Guard *"Tồn tại → DỪNG. KHÔNG ghi, KHÔNG hỏi Y/N"*. Đúng vậy: ghi đè mất changelog và **đánh số lại BR** ⇒ phá `@trace.business_rules` trong mọi `.feature` đã sinh, cả ba đều không hoàn tác được từ trong lệnh |
| `/extend-prd` | **Add-only.** Bước 5 §3 đòi output là *"superset chặt"* của bản cũ. Có **một** cửa sửa nội dung cũ (Bước 3.2 case 1, "mâu thuẫn rule") nhưng nó **phái sinh** — chỉ mở khi phần THÊM làm BR cũ sai |
| `/refine-prd` | Resume Mode Phase 2 tự cấm đụng section nào không được một finding trỏ tới, và findings sinh từ việc soi PRD hiện có ⇒ **không có đường nào để một ý định MỚI của PO đi vào** |

Nên hành vi hợp lý duy nhất còn lại là **mở file `.md` ra gõ**.

---

## Vì sao nhánh thiếu này nặng hơn nó trông (GAPS-v4 G54)

Toàn bộ lưới an toàn của framework so **nhãn version**, không so **nội dung**. Kiểm bằng máy: `grep "content_hash|checksum|sha256|md5"` trên toàn bộ lệnh + schema + step → **0 kết quả**.

Nên khi PO sửa tay: bìa không đổi, trang changelog trắng, con dấu `approved` vẫn còn.

| Tầng canh | Nó hỏi gì | Trả lời | Kết quả |
|---|---|---|---|
| `/validate-traces` Step 4 | *"PRD Version == cột `prd_version`?"* | `1.3 == 1.3` | ✅ sạch |
| `gate-trace` G2 | *"report khớp sổ TSV?"* | khớp | ✅ PASS |
| `require-fresh-audit` | *"PR có chạm file mang tag trace?"* | PO chỉ sửa `.md` | ✅ không đòi audit |

**Ba tầng xanh, và ba tầng đều đúng theo định nghĩa của chính chúng.** Code vẫn chặn ở 5, test vẫn assert 5 và vẫn PASS — nó đang test đúng code, chỉ là code sai spec. Dashboard hiện `OK · ✅ 10 tests · qc pass`.

So với hai họ hàng gần:

| | Hỏng kiểu gì | Có gì để phát hiện? |
|---|---|---|
| G52 | **Ồn** — báo oan cho mọi UC | Có, quá nhiều |
| G53 | **Im lặng** — nhưng version vẫn lệch, vẫn còn cờ ⓘ để người tinh ý thấy | Có, một chút |
| **G54** | **Im lặng tuyệt đối** | **Không có gì** |

Và đây là nhánh **dùng nhiều nhất**: trên một sản phẩm đang sống, *"đổi một yêu cầu đã có"* xảy ra thường hơn *"thêm một UC hoàn toàn mới"* rất nhiều.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Specification — nhánh *"PRD đã tồn tại, đổi nội dung"*.
- **Tiền đề:** có PRD. Không có → `/generate-prd`.
- **Mức chặn:** **CỨNG** (`--yes` không bỏ qua được). Đây là thao tác ghi **duy nhất** trong framework được phép làm output **không phải superset** của bản cũ.

---

## Nó làm gì, theo thứ tự

| Bước | Làm gì | Điểm đáng chú ý |
|---|---|---|
| **1** | Nạp PRD + **PO khai tường minh `amend_targets`** | Trình danh sách UC/BR/AC để PO chọn. **Không tự suy** target từ mô tả mơ hồ — đoán sai là sửa sai một yêu cầu đã duyệt. Mỗi ID được phân giải về **UC sở hữu** ngay |
| **2** | **Kiểm va chạm** (3 câu) | Tái dùng `/extend-prd` Bước 3.2, **đảo hướng**: ở đó là *"phần THÊM có làm cái cũ sai không"*, ở đây là *"cái SỬA có làm phần còn lại sai không"*. Mỗi "có" **mở rộng `amend_targets`** |
| **3** | Altitude | Cơ chế xuống BR/BL, AC chỉ giữ outcome + ref. Đây là chỗ dễ trôi nhất khi sửa tại chỗ |
| **4** | Ghi + **guard sau-ghi HAI CHIỀU** | Xem dưới — đây là phần cốt lõi |
| **5** | Bump version + `{changelog_scope}` + `Status → draft` | Đổi một giới hạn nghiệp vụ là **major**: code hiện tại đang sai so với spec mới |
| **6** | Report | Nêu **UC PHẢI làm lại** kèm lệnh, và **cấm tường minh** `--realign` cho chúng |

### Guard hai chiều — chỗ lệnh này khác mọi thao tác ghi khác

`/extend-prd` guard bằng *"output là **superset chặt**"*. Ở đây output **cố ý không** phải superset, nên guard phải đảo:

| Chiều | Kiểm gì | Fail nghĩa là |
|---|---|---|
| **Bảo toàn** | Mọi UC/BR/AC-ID + row changelog cũ **vẫn còn** | Đã xoá thứ không được xoá |
| **Giới hạn** | **Mọi** nội dung đã đổi đều thuộc một ID trong `amend_targets` | Đã sửa **lan ra ngoài** phạm vi PO chốt |

Chiều **Giới hạn** quan trọng bằng chiều Bảo toàn, và vì một lý do cụ thể: `{changelog_scope}` dựng từ `amend_targets`. Nếu bản ghi lỡ sửa một UC không có trong danh sách đó thì changelog **không nêu** UC ấy ⇒ `/validate-traces` xếp nó vào ⓘ `PRD_STALE_REF` ⇒ `--realign-prd-version` **mở cửa** và dán nhãn version lại lên một thay đổi chưa ai implement. Đúng hình dạng **G53**, chỉ đến từ một hướng khác.

Fail chiều nào → **khôi phục file**, dừng, **không** sang Bước 5.

---

## Hai chế độ, và những gì nó **không** làm

| Chế độ | Cờ | Làm gì |
|---|---|---|
| Sửa nội dung | *(mặc định)* | Đổi nội dung của ID đã có. ID giữ nguyên |
| **Khai tử tại chỗ** | `--retire {ID}` | Đánh dấu ID không còn hiệu lực **nhưng GIỮ NGUYÊN row + ID** |

`--retire` tồn tại vì **xoá hẳn một BR là bẫy**: `@trace.business_rules` trong mọi `.feature` đã sinh đang trỏ vào ID đó, nên xoá row biến một liên kết hợp lệ thành `TRACE_ORPHAN` 🔴. Khai tử tại chỗ đạt cùng mục đích nghiệp vụ mà không phá liên kết.

| PO muốn | Lệnh đúng |
|---|---|
| Thêm UC/AC/BR mới | `/extend-prd` — lệnh này **không đánh số mới** bao giờ |
| **Xoá hẳn** một dòng | *(không có, có chủ ý)* → `--retire` |
| Sửa lỗi `/refine-prd` vừa chỉ ra | `/refine-prd --resume` |

---

## Cửa sau vẫn có chuông

Mở cửa chính không có nghĩa không ai đi cửa sau. `/validate-traces` **Step 3.9** canh:

- Đọc mốc `spec_baseline` (`prd_path` · `sha_at_audit` · `version_at_audit`) mà lần audit trước đã ghi.
- So bằng **hai nguồn**: `git diff` (sửa đã commit) **và** `git status` (sửa **chưa** commit — ca thường gặp nhất, vì PO đang gõ).
- Nội dung đổi **mà** `Version` không đổi → 🔴 **`PRD_UNTRACKED_EDIT`**.

**Không báo oan:** sửa **và** bump version ⇒ `version != version_at_audit` ⇒ im.

**Đường ra tự lành, cố ý không có `--accept-edit`:** cách sửa đúng là bump version + ghi row changelog nêu UC — tức đúng việc lệnh này làm hộ. Làm xong thì cờ tự tắt và logic `PRD_DRIFT` bình thường tiếp quản. Thêm một cờ escape sẽ là thêm một đường **dán nhãn lên thay đổi chưa ai xem**, đúng cái ba rào của `--realign` tồn tại để chặn.

**Vì sao cờ này không chặn PR:** `gate.blocking` nghĩa hẹp là *code đang hỏng*; cờ này nói về *spec*. Thêm nữa, mọi project đang chạy đều đã có PRD sửa tay ⇒ một cờ chặn mới sẽ đỏ khắp nơi ở lần đầu ⇒ người ta **tắt cổng** ⇒ mất luôn 4 cờ 🔴 thật. Đó đúng là thất bại mà `self-check` R9(e) được viết ra để chặn, chỉ đến bằng một cửa khác. Team đã dọn sạch nợ tồn thì tự thêm counter vào `gate.blocking`; R13(e) canh việc đó có kèm `why`.

---

## Sau đó chạy gì

```
/amend-prd {prd-file} UC3-BR8
  → /review-context {prd-file}      ← kiểm chất lượng phần vừa sửa
  → PO đặt Status: approved
  → /generate-bdd {prd-file}        ← CHỈ cho UC bị sửa
  → /generate-code {UC-ID}
  → /dev-gen-test → /dev-run-test   ← test cũ đang assert giá trị cũ và vẫn PASS
```

❌ **Không** `--realign-prd-version` cho UC bị sửa — nội dung đổi thật.

---

## Máy canh gì

| Rule | Canh gì |
|---|---|
| `self-check` **R11** | Lệnh phải tự khai mức `chặn CỨNG`, khớp `gate.checkpoint_levels.hard` |
| `self-check` **R12** | Là producer dòng changelog → phải có **dòng template** mang `{changelog_scope}` |
| `self-check` **R13** | `spec_baseline` phải được **GHI** như một key trong `trace-report.json` — nếu không, cờ `PRD_UNTRACKED_EDIT` không bao giờ bật mà R6/R7 vẫn ✅ |

---

[← /extend-prd](02b-extend-prd.md) · [Explain Home](README.md) · [Next: /refine-prd →](03-refine-prd.md)
