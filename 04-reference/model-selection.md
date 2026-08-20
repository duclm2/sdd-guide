[← Reference](./) · [Concepts › Roles & HITL](../02-concepts/roles-and-hitl.md)

# Reference · Model Selection

> Framework **không ràng buộc model cứng** — nó **khai báo** model đang chạy ở report cuối. Giả định LLM đủ năng lực reasoning.

---

## Model được KHAI BÁO, không bị hỏi *(từ v0.5.1)*

Gate Bước 0-B ghi nhận model agent đang chạy; `report-footer` in ra ở cuối mỗi lệnh:

```
Model: claude-opus-5
```

Nếu agent biết mình không phải một model Opus:

```
Model: claude-haiku-4-5   ⚠️ lệnh này khuyến nghị Opus — model nhỏ hơn dễ bỏ sót
                             edge case, phân tích spec thiếu sót, vi phạm kiến trúc.
                             Cân nhắc chạy lại với /model → Opus trước khi dùng kết quả.
```

**Không có prompt nào. Không chờ. Không dừng.**

### Vì sao bỏ prompt cũ (GAPS-v3 G41)

Bản trước hiện một khối `⚙️ MODEL CHECK` rồi chờ `Y/S/N` ở **mọi** lệnh. Ba vấn đề:

| | |
|---|---|
| Hỏi sai người | Nó hỏi **người dùng** thứ mà **agent đã biết chính xác** |
| Không kiểm chứng được | Gõ `Y` xong vẫn đang chạy Haiku thì không gì phát hiện |
| Không chặn được ai | **Cả `Y` lẫn `S` đều đi tiếp** — cách duy nhất để nó dừng là tự nguyện gõ `N` |

Cái giá: một lần chặn ở mọi lệnh. Một feature đi hết pipeline dùng **20 lệnh** — 20 lần bấm
cho một tín hiệu tự-khai không kiểm chứng được. Và chính cái giá đó **làm mòn CHECKPOINT**,
cổng có giá trị thật ngay sau nó: bấm `Y` 40 lần một feature thì `Y` thành phản xạ.

Khai báo ở footer **mạnh hơn** hỏi: đúng nguồn (agent tự khai), và nằm **cạnh kết quả** để
người đọc cân nhắc có nên tin, thay vì nằm trước khi có kết quả để bấm cho xong.

> Sub-agent (`_agent_mode`) bỏ qua bước này — orchestrator đã ghi nhận rồi.

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
| Opus 5 | `claude-opus-5` |
| Sonnet 5 | `claude-sonnet-5` |
| Haiku 4.5 | `claude-haiku-4-5-20251001` |
| Fable 5 | `claude-fable-5` |

> Dùng model mạnh nhất (Opus) cho các bước generate/review; các bước cơ học có thể dùng model nhẹ hơn.

---

→ [Roles & HITL](../02-concepts/roles-and-hitl.md) · [Commands](commands.md)
