[← Reference](./) · [Concepts › Traceability](../02-concepts/traceability.md)

# Reference · Trace Schema

> Field metadata `@trace.*` và cột file `.tsv`. Giải thích khái niệm → [Traceability](../02-concepts/traceability.md).

---

## Trace tags theo artifact

### BDD scenario (`.feature`)

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.id` | UC-ID (vd `UC-02`) |
| `@trace.scenario` | SC-ID (vd `SC-02.1`) |
| `@trace.business_rules` | BR liên quan (vd `BR-03`) |
| `@trace.bdd_version` | Version của `.feature` (phát hiện drift) |
| `@trace.prd_version` | Version PRD nguồn |
| `@trace.status` | `draft` / `in-review` / `approved` (gate) |
| `@trace.platform` | `web` / `app` / `system` |
| `@trace.domain` | Domain nghiệp vụ |

### Code (boundary only)

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.implements` | SC mà method này hiện thực |
| `@trace.source` | `.feature` nguồn |

> **KHÔNG** lưu version trong code — tránh dual SSOT. Version dò qua `.tsv`.

### Test

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.verifies` | SC mà test kiểm chứng (vd `UC-02-SC1`) |
| `@trace.covers` | Phạm vi phủ |

### Bug fix

| Tag | Ý nghĩa |
|-----|---------|
| `@trace.fixes` | Bug/ticket được sửa |
| `@trace.root_cause` | Nguyên nhân gốc |
| `@trace.regression` | Test regression thêm vào |

---

## Boundary-only tagging

| ✅ Tag | ❌ Không tag |
|--------|-------------|
| Controller · Handler · Middleware · Steps file | Entity · Repository · DTO · Interface · Base class |

Shared code dò qua **import chain** từ boundary → tránh tag explosion.

---

## Trace state — `.tsv`

Đường dẫn: `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` (mỗi UC × platform một sổ; trong umbrella nằm ở spec repo).

| Cột | Chủ sở hữu | Ý nghĩa |
|-----|-----------|---------|
| `status` | `/generate-code`, `/validate-traces` | OK / GAP / DRIFT / UNTRACKED |
| `implemented_by` | `/generate-code` | File code hiện thực SC (`—` nếu chưa) |
| `spec_ver` | spec | Version spec hiện tại |
| `gen_ver` | `/generate-code` | Version lúc sinh code (`—` nếu chưa) |
| `test_count` | test | Số test phủ SC |
| `dev_selftest` | `/dev-run-test` | Smoke của dev (pass/fail/`—`) |
| `qc_status` | `/qc-run-test`, `/report-bug` | QC chính thức (pass/fail/gap) |
| `bdd_version` | spec | Version BDD |
| `last_updated` | nhiều | Mốc cập nhật |

---

## Phân loại coverage (thứ tự ưu tiên)

| # | Trạng thái | Điều kiện |
|---|-----------|-----------|
| 1 | **UNTRACKED** | `gen_ver == —` |
| 2 | **DRIFT** | `implemented_by != —` AND `spec_ver != gen_ver` |
| 3 | **GAP** | `implemented_by != —` AND (`test_count == —` OR `0`) |
| 4 | **OK** | `spec_ver == gen_ver` AND `implemented_by != —` AND `test_count > 0` |

`code_coverage = (rows where implemented_by != —) / total_scs`

> DRIFT xét **trước** GAP: code lỗi thời + chưa test + spec drift phải hiện DRIFT (regen) không phải GAP (skip).

---

→ [Traceability](../02-concepts/traceability.md) · [Validate Traces](../02-concepts/pipeline-steps/09-validate-traces.md)
