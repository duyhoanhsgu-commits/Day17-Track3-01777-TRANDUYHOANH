# Báo cáo Thu hoạch Lab 17 - Multi-Memory Agent với Zep

- **Họ và tên**: TRẦN DUY HOÀNH
- **Mã học viên / MSSV**: 01777
- **Mã Repository**: Day17-Track3-01777-TRANDUYHOANH

---

## 1. Phân tích Benchmark

1. **Layer có hit rate thấp nhất**: Trong bài lab này cả 4 layer đều đạt 100% (11/11 PASS). Tuy nhiên ở baseline `no-memory`, các layer Long-term, Episodic và Semantic đều rớt 0% vì thông tin nằm cross-session/external graph; chỉ Short-term đạt 100% nhờ context còn nằm trong phiên hiện tại.
2. **Case retrieve nhiều token nhất**: Case E07 (mixed) retrieve nhiều token nhất vì phải truy xuất kết hợp cả Long-term (user preference `Python`) và Semantic (`Idempotency-Key` payment rule).
3. **Case mixed (E07)**: Cần kết hợp Long-term Memory (user preference) + Semantic Memory (business rule). Evidence bắt buộc bao gồm cả `Python` và `Idempotency-Key`.
4. **Token reduction & No-memory**: No-memory đạt token reduction cao (81.8%) chỉ vì nó không trả về bộ nhớ nào, khiến Hit rate rớt xuống 18.2%. Token reduction chỉ có ý nghĩa khi đi kèm Evidence hit rate cao.

---

## 2. Thảo luận Thực hành

1. **Layer quan trọng nhất**: Long-term Memory (Context Block) và Episodic Memory là quan trọng nhất trong bộ test này (giải quyết 6/11 case từ E02-E05, E08-E09), giúp Agent giữ được profile người dùng và kinh nghiệm xử lý sự cố cross-session.
2. **Trade-off Zep Cloud vs Redis+Qdrant**:
   - *Zep Cloud V3 (Managed)*: Tự động trích xuất graph, facts, validity range, hỗ trợ Context Block tốt nhưng phụ thuộc API cloud và độ trễ cao hơn (~0.5s - 1.5s).
   - *Redis + Qdrant (Local)*: Độ trễ siêu thấp, toàn quyền kiểm soát dữ liệu nhưng đòi hỏi tự quản lý embedding, chunking, TTL và tự ghép nối context thủ công.
3. **Guardrail chống Memory Poisoning**:
   - Áp dụng PII minimization & Consent check (`privacy_guard.py`) trước khi ghi.
   - Giới hạn quyền ghi durable memory (Heartbeat dry-run, de-duplication, chỉ cho phép ghi thông qua xác thực consent).
   - Phân tách biệt lập giữa User Memory (`user_id`) và System/Semantic Graph (`graph_id`).
