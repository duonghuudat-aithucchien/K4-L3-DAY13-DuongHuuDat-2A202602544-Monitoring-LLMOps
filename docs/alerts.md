# Template Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## Alert mẫu để tham khảo

Ví dụ dưới đây minh họa mức độ cụ thể cần có. Học viên không cần copy nguyên, nhưng ba alert trong bài nộp nên rõ ràng tương tự: điều kiện là gì, kéo dài bao lâu, ảnh hưởng tới user ra sao và người trực cần kiểm tra gì trước.

- Tên: `HighLatencyP95`
- Severity: `warning`
- Duration: `5m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: latency P95 của `response_sent.latency_ms`
- Điều kiện và thời gian duy trì: `p95(latency_ms) > 3000ms` trong 5 phút
- Ảnh hưởng tới người dùng: người dùng phải chờ lâu hơn trước khi nhận câu trả lời
- Ba bước kiểm tra đầu tiên:
  1. Mở dashboard latency để xác nhận P95/P99 và khoảng thời gian tăng.
  2. Lọc `data/logs.jsonl` trong khoảng đó, lấy một `correlation_id` có `latency_ms` cao.
  3. Mở trace cùng `correlation_id` trên Langfuse, so sánh các span chính để xác định bước nào bất thường.
- Mitigation tạm thời: dựa trên evidence thực tế để rollback prompt, khôi phục cấu hình liên quan, tắt practice scenario hoặc giảm tải khi demo.
- Owner: `student-<MSSV>`

## Alert 1

- Tên: HighLatencyP95
- Severity: warning
- Duration: 5m
- Kênh thông báo: Slack
- SLI/SLO liên quan: latency P95 của response_sent.latency_ms
- Điều kiện và thời gian duy trì: p95(latency_ms) > 3000ms trong 5 phút
- Ảnh hưởng tới người dùng: Người dùng phải chờ quá lâu để nhận được câu trả lời.
- Ba bước kiểm tra đầu tiên:
  1. Kiểm tra dashboard Latency xem mức tăng đột biến có trùng với tăng Traffic không.
  2. Truy vết các request chậm trên Langfuse bằng correlation_id để xem Retrieval hay Generation đang chậm.
  3. Kiểm tra logs để xem có warning timeout từ VectorDB hoặc LLM provider không.
- Mitigation tạm thời: Rollback lại version prompt ngắn hơn hoặc thêm cache cho retrieval.
- Owner: student-123456

## Alert 2

- Tên: HighErrorRate
- Severity: critical
- Duration: 5m
- Kênh thông báo: Slack
- SLI/SLO liên quan: error_rate_pct_max <= 2%
- Điều kiện và thời gian duy trì: error_rate_pct > 2% trong 5 phút
- Ảnh hưởng tới người dùng: Hệ thống trả về lỗi liên tục khiến người dùng không thể sử dụng dịch vụ.
- Ba bước kiểm tra đầu tiên:
  1. Xem bảng Errors trên Dashboard để lấy mã lỗi phổ biến nhất.
  2. Kiểm tra Logs để lấy thông báo lỗi chi tiết (ví dụ: API Key expired, Rate limit).
  3. Kiểm tra Langfuse xem LLM API có bị down không.
- Mitigation tạm thời: Chuyển sang model dự phòng (fallback model) nếu LLM chính lỗi, hoặc khởi động lại container RAG.
- Owner: student-123456

## Alert 3

- Tên: LowQualityScore
- Severity: warning
- Duration: 10m
- Kênh thông báo: Slack
- SLI/SLO liên quan: quality_score_avg_min >= 0.75
- Điều kiện và thời gian duy trì: mean(quality_score) < 0.75 trong 10 phút
- Ảnh hưởng tới người dùng: Câu trả lời của Agent bị sai lệch, thiếu thông tin hoặc vi phạm PII.
- Ba bước kiểm tra đầu tiên:
  1. Mở dashboard kiểm tra xem Retrieval success rate có bị giảm không (nếu RAG không tìm được tài liệu, chất lượng sẽ giảm).
  2. Đọc các trace trên Langfuse để xem prompt version hiện tại có sinh ra câu trả lời quá ngắn không.
  3. Kiểm tra xem có bản release prompt mới nào vừa được push lên không.
- Mitigation tạm thời: Rollback prompt về version `production` trước đó hoặc đổi sang model mạnh hơn.
- Owner: student-123456
