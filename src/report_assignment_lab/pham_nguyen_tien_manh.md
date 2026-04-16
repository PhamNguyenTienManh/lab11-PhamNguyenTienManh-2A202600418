# Báo Cáo Cá Nhân

**Sinh viên:** Phạm Nguyễn Tiến Mạnh
**MSSV:** 2A202600418

---

## Question 1 — Layer Analysis

_Đối với mỗi prompt tấn công trong Test 2, layer nào phát hiện đầu tiên?_

Tất cả 7 prompt tấn công đều bị chặn ngay tại **Input Guardrails (Layer 2)**.

- Lý do: Các regex đã bao phủ hầu hết pattern phổ biến như:
  - “ignore previous instructions”
  - “DAN jailbreak”
  - giả mạo quyền (authority spoofing)
  - dịch system prompt
- Các layer khác như:
  - Topic filter (Input Guardrails)
  - LLM-as-Judge  
    cũng có thể bắt được, nhưng không cần vì đã bị chặn từ sớm.

Kết luận: Input Guardrails hoạt động rất hiệu quả cho các attack đã biết.

---

## Question 2 — False Positive Analysis

_Có query an toàn nào bị chặn nhầm không? Trade-off là gì?_

### Ban đầu:

- Tất cả query hợp lệ đều bị block do lỗi:
  - Model judge bị deprecated
  - Fail-closed (lỗi parse → block luôn)

### Sau khi fix:

- Không còn false positive ở cấu hình bình thường

### Khi tăng độ nghiêm ngặt:

- Thêm keyword → block nhầm
- Tăng threshold → reject câu trả lời hợp lệ
- Regex quá rộng → “password” cũng bị block

### Trade-off:

- **Bảo mật cao → dễ block nhầm**
- **Dễ dùng → dễ lọt attack**

Cần cân bằng:

- Regex: strict
- Keyword: vừa phải
- Judge: fail-open

---

## Question 3 — Gap Analysis

_3 attack chưa bị bắt_

### Attack 1 — Social Engineering

- Dạng kể chuyện, cảm xúc
- Không match regex
- Không có keyword nguy hiểm rõ ràng

Cách fix:

- Thêm **intent classifier (LLM)**

---

### Attack 2 — Multi-turn attack

- Chia nhỏ qua nhiều bước
- Mỗi câu riêng lẻ đều hợp lệ

Cách fix:

- Lưu lịch sử hội thoại
- Phân tích context nhiều turn

---

### Attack 3 — Unicode attack

- Dùng ký tự giống nhau (Cyrillic)
- Regex không match

Cách fix:

- Normalize Unicode trước khi check

---

## Question 4 — Production Readiness

_Nếu deploy cho 10k user_

### Vấn đề:

- 2 LLM call / request → tốn tiền + chậm
- Logging chưa scale
- Rule hardcode

### Giải pháp:

1. **Giảm latency**
   - Chạy judge async

2. **Giảm cost**
   - Chỉ dùng judge khi cần

3. **Monitoring**
   - Dùng DB + dashboard

4. **Config động**
   - Lưu rule ngoài code (YAML/JSON)

5. **Rate limit**
   - Dùng Redis thay vì memory

---

## Question 5 — Ethical Reflection

_Có thể tạo AI an toàn tuyệt đối không?_

### Lý do:

- Regex không hiểu ngữ nghĩa
- Keyword không hiểu context
- LLM không hoàn hảo

Không thể đạt 100% an toàn

---

### Khi nào nên từ chối?

- Khi hỏi:
  - password
  - API key
  - bypass system

→ **Refuse**

---

### Khi nào nên trả lời có cảnh báo?

Ví dụ:

- “Quên mật khẩu ngân hàng”

Trả lời:

- hướng dẫn reset
- không tiết lộ thông tin

---
