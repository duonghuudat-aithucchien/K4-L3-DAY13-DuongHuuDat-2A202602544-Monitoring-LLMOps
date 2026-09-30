# Báo cáo cá nhân — K4-L3B Day 13 Monitoring & LLMOps

> Mỗi học viên hoàn thiện một file duy nhất này. Khi dẫn evidence, dùng đường dẫn tương đối, ví dụ `evidence/07-trace-waterfall.png`.

## 1. Thông tin học viên

- **Họ và tên: Dương Hữu Đạt**
- **MSSV: 2A202602544**
- **Lớp: K4-L3B**
- **Repository URL: https://github.com/duonghuudat-aithucchien/K4-L3-DAY13-DuongHuuDat-2A202602544-Monitoring-LLMOps**
- **Commit SHA cuối: fb39a929617077f0337b21e23264e0ed460f2802**
- **Challenge ID: day13-k4-l3b-monitoring-llmops-v1**
- **Tên project Langfuse cá nhân:** `day13-k4-l3b-2A202602544`

## 2. Evidence index

Điền đúng đường dẫn tới evidence thực tế. Có thể đổi tên hoặc dùng nhiều ảnh nếu cần.

| Evidence | Đường dẫn |
|---|---|
| Pytest cuối | `evidence/01-pytest.png` |
| Log validator | `evidence/02-log-validator.png` |
| Dashboard validator | `evidence/03-dashboard-validator.png` |
| Structured log | `evidence/04-structured-log.png` |
| PII redaction | `evidence/05-pii-redaction.png` |
| Trace list | `evidence/06-trace-list.png` |
| Trace waterfall | `evidence/07-trace-waterfall.png` |
| Trace metadata | `evidence/08-trace-metadata.png` |
| Prompt versions | `evidence/09-prompt-versions.png` |
| Prompt rollback | `evidence/10-prompt-rollback.png` |
| Dashboard runtime | `evidence/11-dashboard-overview.png` |
| Incident metric | `evidence/12-incident-metric.png` |
| Incident log | `evidence/13-incident-log.png` |
| Incident trace | `evidence/14-incident-trace.png` |

## 3. Kết quả kỹ thuật

| Nội dung | Baseline | Kết quả cuối | Nhận xét |
|---|---|---|---|
| `validate_logs.py` | 55/100 | 100/100 | Đã thêm correlation_id và PII redaction |
| `validate_dashboard.py` | 0/6 panel | 6/6 panel | Đã tạo đủ 6 panel trên Langfuse |
| `pytest` | Lỗi | Pass 100% | Đã cập nhật đúng cú pháp của SDK v4 |
| Số traces hợp lệ | 0 | >10 | Đã log đầy đủ bằng decorator @observe |
| Số PII leak | >0 | 0 | Đã che giấu thành công email, SĐT, thẻ tín dụng |
| Latency P95 / TTFT P95 | N/A | Đã có data | Dashboard đã ghi nhận được dữ liệu |
| Retrieval success rate | N/A | 100% | Dashboard đã thống kê lỗi truy xuất |

- **Cách tạo/nhận và truyền correlation ID:** Dùng `uuid.uuid4().hex[:8]` để tạo (nếu request chưa có `x-request-id`) và lưu vào `contextvars` để dùng chung cho toàn bộ request.
- **Các metadata được ghi vào structured log:** `user_id_hash`, `session_id`, `feature`, `model`, `env`, `correlation_id`.
- **Cách bảo đảm PII được scrub trước khi ghi:** Dùng hàm `scrub_pii` (chứa các Regex) chèn vào trước bước ghi file (file handler formatter).
- **Cách kiểm chứng kết quả:** Chạy script `validate_logs.py` đạt 100/100 điểm.
## 5. Tracing và prompt versioning

- **Cách xác nhận traces do chính tôi tạo trong project cá nhân:** Các trace xuất hiện trên Dashboard của project Langfuse tự tạo.
- **Cấu trúc root/retrieval/generation observations:** Root (`lab-agent-run`) bọc bên ngoài, bên trong gọi hàm `retrieve()` (Span) và `generate()` (Generation).
- **Cách nối trace với log:** Tiêm `correlation_id` vào thuộc tính của Trace thông qua tính năng `update_current_trace` / `update_current_observation` của SDK.
- **Prompt name:** `day13-chat`
- **Version/label baseline:** `v1` (production)
- **Version/label candidate:** `v2` (production)
- **Trace ID của mỗi version:** (Ghi nhận trực tiếp trên giao diện Langfuse)
- **Cách promote và rollback `production`:** Vào giao diện Prompts, chọn version muốn dùng, rồi bấm Promote to Production để đổi nhãn.

## 6. Dashboard, SLO và alerts

- **Dashboard và sáu panel:** Gồm Latency, Traffic, Errors, Cost, Tokens, Quality.
- **SLO và lý do chọn:** Latency P95 < 3000ms. Vì với hệ thống AI/RAG, người dùng sẽ mất kiên nhẫn nếu phản hồi quá 3s.
- **Cách tính error budget:** SLO 99% trong 28 ngày nghĩa là error budget 1%. Nếu workload có 10,000 request thì tối đa 100 request được phép lỗi hoặc vượt quá 3s.
- **Ba alert và runbook tương ứng:**
  1. `latency_p95_high`: Báo động khi P95 > 3s. Runbook: Kiểm tra VectorDB và API của LLM.
  2. `error_rate_high`: Báo động khi Error > 5%. Runbook: Kiểm tra rate limit hoặc timeout.
  3. `quality_drop`: Báo động khi Quality < 0.8. Runbook: Xem xét rollback prompt về bản v1.

> Ví dụ cách viết error budget: "SLO 99.5% trong 28 ngày nghĩa là error budget 0.5%. Nếu workload có 10,000 request thì tối đa 50 request được phép lỗi hoặc chậm hơn ngưỡng SLO."

## 7. Điều tra challenge

- **Challenge ID:** day13-k4-l3b-monitoring-llmops-v1
- **Khoảng thời gian điều tra:** 12:20 ngày 30/09/2026
- **Triệu chứng từ metrics:** Trên bảng Dashboard, biểu đồ Latency P95 và Mean đều tăng vọt đột biến, vượt ngưỡng SLO 3000ms lên đến 20000ms.
- **Log line và correlation ID liên quan:** Lọc trong `data/logs.jsonl` thấy dòng log sự kiện `response_sent` có `correlation_id="req-886c8fbd"` với `latency_ms` cao bất thường (2652).
- **Trace ID và span gây ảnh hưởng:** Tra cứu `req-886c8fbd` trên Langfuse, phân tích Trace waterfall cho thấy span `retrieval` tốn đến 2.5s.
- **Root cause:** Hệ thống tìm kiếm tài liệu (Vector Database / RAG) bị thắt nút cổ chai hoặc phản hồi quá chậm (sự kiện `rag_slow`).
- **Fix action:** Scale up tài nguyên cho VectorDB, hoặc tạm thời kích hoạt hệ thống Cache cho retrieval để giảm tải.
- **Preventive measure:** Thiết lập cảnh báo (Alert) riêng cho Latency của tác vụ `retrieval`. Thêm cơ chế Timeout cứng (ví dụ max 1.5s) cho hàm `retrieve()`, nếu quá thời gian thì dùng fallback answer.

> Gợi ý cách viết ngắn, không thay cho evidence thực tế: "Metric cho thấy `[latency/error/cost/quality]` bất thường trong `[khoảng thời gian]`. Log line `[event]` có `correlation_id=[...]` đại diện cho request bị ảnh hưởng. Trace cùng `correlation_id` cho thấy span `[retrieval/generation/prompt/tool]` có dấu hiệu `[chậm/lỗi/token tăng]`. Root cause là `[nguyên nhân suy ra từ evidence]`. Fix action là `[hành động khôi phục]`; preventive measure là `[alert/runbook/test/guardrail để ngăn tái diễn]`."

- **Một quyết định kỹ thuật quan trọng và lý do:** Quyết định sử dụng `contextvars` để quản lý `correlation_id`. Lý do: Nó an toàn trong môi trường bất đồng bộ (async) của FastAPI, đảm bảo các concurrent request không bị lẫn lộn ID.
- **Một lỗi/blocker đã gặp:** Gặp lỗi khi dùng SDK v4 cũ với phương thức `.span()` và truyền biến `usage`.
- **Cách tìm nguyên nhân và xử lý:** Đọc tài liệu SDK v4, chuyển sang dùng decorator `@observe()` và đổi tham số thành `usage_details`.
- **Cách hiểu luồng Metrics → Logs → Traces:** Metrics báo động có cái gì đó sai (What) -> Logs chỉ ra cụ thể request nào sai (Who/Where) -> Traces đi sâu vào trong request để chỉ ra tại sao lại sai (Why).
- **Vai trò của prompt version, token/cost, SLO hoặc rollback trong vận hành LLM:** LLM thay đổi thường xuyên, quản lý version và rollback giúp an toàn khi deploy. SLO giúp giữ cam kết chất lượng.
- **Điều quan trọng nhất đã học:** Cách tư duy điều tra sự cố từ tổng quát (Dashboard) đến chi tiết (Waterfall).
- **Hạn chế hoặc phần chưa hoàn thành, nếu có:** Không có.

## 9. Checklist trước khi nộp

- [x] Kết quả và evidence thuộc commit SHA cuối.
- [x] Tất cả ảnh/output mở được bằng đường dẫn tương đối.
- [x] Incident evidence nối đúng metric → log → trace.
- [x] Trace/prompt evidence thuộc project Langfuse cá nhân và ảnh không lộ key/secret.
- [x] Repository chạy lại được theo README.
- [x] Không có secret, API key, PII thô hoặc evidence của người khác/lớp khác.
- [x] URL repo và commit SHA cuối đã được nộp trên LMS/Codelabs.
