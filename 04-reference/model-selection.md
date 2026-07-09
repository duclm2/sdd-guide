[← Reference](./) · [Concepts › Roles & HITL](../02-concepts/roles-and-hitl.md)

# Reference · Model Selection

> Framework **không ràng buộc model cứng** — chỉ khuyến nghị qua **model check mềm** ở Gate. Giả định LLM đủ năng lực reasoning.

---

## Model check (Gate Bước 0-B)

Các lệnh sinh nội dung & review phức tạp đòi hỏi suy luận mạnh. Ở Gate, framework hiển thị:

```
⚙️  MODEL CHECK
  Recommended : claude-opus (model Opus mới nhất)
  Why needed  : Phân tích spec, review kiến trúc, sinh code đòi hỏi
                suy luận sâu. Model nhỏ hơn dễ bỏ sót edge case.

  Đang chạy claude-opus?
    Y — đúng → tiếp tục
    S — bỏ qua (chấp nhận rủi ro chất lượng thấp hơn)
    N — dừng, đổi model rồi chạy lại
```

| Trả lời | Kết quả |
|---------|---------|
| **Y** | Tiếp tục |
| **S** | Tiếp tục, thêm ⚠️ vào report cuối (người dùng chấp nhận rủi ro) |
| **N** / khác | **Dừng** — đổi sang Opus rồi chạy lại |

> Sub-agent (`_agent_mode`) **bỏ qua** model check — orchestrator đã kiểm tra rồi.

---

## Tại sao khuyến nghị Opus

Rủi ro khi dùng model nhỏ hơn cho các lệnh nặng:
- Bỏ sót edge case khi phân tích spec.
- Phân tích spec thiếu sót → findings không đầy đủ.
- Vi phạm kiến trúc khi sinh code.

Các lệnh **read-only cơ học** (áp findings, resume) có thể bỏ qua model check vì không cần suy luận sâu.

---

## Đổi model trong Claude Code

- Settings → Model → chọn "claude-opus", **hoặc**
- `/model` → chọn `claude-opus`

---

## Model IDs (tham khảo)

Model Claude mới nhất tại thời điểm viết:

| Model | ID |
|-------|-----|
| Opus 4.8 | `claude-opus-4-8` |
| Sonnet 5 | `claude-sonnet-5` |
| Haiku 4.5 | `claude-haiku-4-5-20251001` |
| Fable 5 | `claude-fable-5` |

> Dùng model mạnh nhất (Opus) cho các bước generate/review; các bước cơ học có thể dùng model nhẹ hơn.

---

→ [Roles & HITL](../02-concepts/roles-and-hitl.md) · [Commands](commands.md)
