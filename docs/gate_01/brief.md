# Brief — AI Agent CSKH Ví điện tử & Xử lý Khiếu nại Giao dịch

| | |
|---|---|
| **Mã đề tài** | FIN-05 |
| **Nhóm** | `T240` |
| **Gate** | G1 — Chốt đề tài · 02/08/2026 |
| **Repo** | [github.com/HuuKhanh19/P-240](https://github.com/HuuKhanh19/P-240) |
| **Tài liệu** | [PRD](prd.md) · [UI Flow](wireframe/ui_flow.md) · [Wireframe](wireframe/wireframe.html) |

---

## 1. Bối cảnh & vấn đề

Tổng đài ví điện tử quá tải vì khiếu nại giao dịch: chuyển nhầm, tiền chưa về, trừ tiền hai lần, yêu cầu hoàn tiền.

| # | Pain point | Bản chất |
|---|---|---|
| P1 | Khách chờ lâu, tổng đài nghẽn | Nghẽn năng lực xử lý |
| P2 | Nhân viên tra cứu nhiều hệ thống, 5–10 phút mỗi ca | Chi phí thu thập thông tin |
| P3 | Quyết định hoàn tiền chậm, thiếu chuẩn hóa, khó giải trình | Chi phí & rủi ro ra quyết định |
| P4 | Rủi ro AI bịa trạng thái / số tiền giao dịch | Rủi ro tính đúng đắn dữ liệu |

P1–P2 là bài toán **biết thông tin**, P3 là bài toán **ra quyết định**. Hai loại có mức rủi ro khác nhau nên được tách thành hai luồng riêng.

## 2. Giải pháp

Hệ thống agentic workflow gồm hai luồng, ngăn cách bởi một ranh giới tin cậy rõ ràng.

**Option 1 — Transaction Investigation Workflow** *(chẩn đoán, chỉ đọc).* Trích xuất thông tin giao dịch từ mô tả tự nhiên → tra cứu qua tool → phân tích nguyên nhân dựa trên log và chính sách → sinh báo cáo kiểm chứng được. Không side effect, chạy lại được, tự động hóa 100%.

**Option 2 — Refund Approval Workflow (HITL)** *(điều trị, có ghi).* Đánh giá điều kiện hoàn tiền → tính số tiền bằng code → dựng hồ sơ đề xuất kèm căn cứ và phản chứng → **dừng chờ nhân viên phê duyệt** → tái kiểm trạng thái → thực thi và ghi audit log.

Vai trò nhân viên CSKH chuyển từ *người thực thi* sang *người kiểm soát*: hệ thống chuẩn bị quyết định, con người phán quyết.

## 3. Giá trị & chỉ số thành công

| Chỉ số | Hiện trạng | Mục tiêu |
|---|---|---|
| Thời gian xác định nguyên nhân | 5–10 phút | < 15 giây |
| Bước tra cứu giao dịch tự động hóa | 0% | 100% |
| Thời gian xử lý 1 yêu cầu hoàn tiền | vài phút | ~5 giây |
| Yêu cầu hoàn tiền qua HITL | — | 100% |
| Hoàn tiền tự động không được phê duyệt | — | 0% |

## 4. Ràng buộc bắt buộc

- **HITL** trước mọi phê duyệt hoàn tiền hoặc điều chỉnh số dư — bảo đảm bằng cấu trúc graph, không bằng prompt.
- **Cấm LLM sinh trạng thái / số tiền** — mọi số liệu đến từ tool, có node kiểm chứng trước khi trả kết quả.
- **PDPA** — xác thực trước thao tác nhạy cảm; truy vấn bị kẹp bởi `customer_id` đã xác thực; không xác nhận sự tồn tại của giao dịch không thuộc về khách.
- **Audit log** append-only, ghi ở mọi node.

## 5. Phạm vi

**Trong phạm vi:** web deploy, đăng nhập hai vai trò, chat agent, mock wallet API và transaction DB, hai workflow trên, dashboard phê duyệt, tạo ticket, RAG chính sách, audit log.

**Ngoài phạm vi:** ngân hàng thật, thanh toán thật, đa ngôn ngữ, voice, phát hiện gian lận nâng cao, mobile app native.

## 6. Tech stack

LLM đối thoại + reasoning · LangGraph (state machine, checkpointer, interrupt) · RAG vector DB cho KB chính sách · Mock Wallet API · FastAPI · PostgreSQL · React · Docker · deploy cloud.

## 7. Rủi ro chính

Hallucination số liệu → node kiểm chứng grounding, escalate sau 1 lần retry. Tool calling sai tham số → whitelist tool theo nhóm khiếu nại, structured output. Mock data quá nghèo → sinh bộ kịch bản phủ đủ trạng thái lỗi ngay từ đầu. Trạng thái đổi giữa lúc đề xuất và lúc duyệt → node tái kiểm trước thực thi. Vòng lặp hội thoại không kết thúc → trần số lượt hỏi làm rõ.

## 8. Kế hoạch

Hai track song song, mỗi track ≈ 9 ngày. Mốc: mock API + tool sẵn sàng → Option 1 chạy end-to-end → Option 2 + dashboard → tích hợp, eval, deploy. Đường găng là schema `transactions` và Mock Wallet API, vì Option 2 không khởi động được trước khi hai thứ này ổn định.