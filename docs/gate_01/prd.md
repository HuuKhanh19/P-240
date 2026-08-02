# PRD — AI Agent CSKH Ví điện tử & Xử lý Khiếu nại Giao dịch

| | |
|---|---|
| **Mã đề tài** | FIN-05 |
| **Nhóm** | `T240` |
| **Gate** | G1 — Chốt đề tài |
| **Ngày** | 02/08/2026 |
| **Tài liệu liên quan** | `brief.md`, `wireframe/`, `ARCHITECTURE.md` |

---

## 1. Mục tiêu & phi mục tiêu

### 1.1 Mục tiêu

| ID | Mục tiêu |
|---|---|
| G1 | Tự động hóa hoàn toàn bước điều tra nguyên nhân khiếu nại giao dịch, đưa thời gian từ 5–10 phút xuống dưới 15 giây |
| G2 | Chuẩn hóa quy trình hoàn tiền qua HITL, đảm bảo 100% phê duyệt có con người và 0% hoàn tiền tự động |
| G3 | Mọi kết luận về trạng thái và số tiền đều truy vết được về dữ liệu hệ thống, không có số liệu do LLM sinh ra |
| G4 | Mọi thao tác tài chính đều có audit log đủ để giải trình |

### 1.2 Phi mục tiêu

Hệ thống **không** nhằm thay thế quyết định của con người trong việc hoàn tiền, không tích hợp ngân hàng thật, không xử lý voice, không phát hiện gian lận nâng cao, và không hỗ trợ đa ngôn ngữ ở phiên bản này.

---

## 2. Người dùng

| Persona | Mô tả | Nhu cầu chính |
|---|---|---|
| **Khách hàng ví** | Người dùng cuối, không rành kỹ thuật, thường đang bực vì mất tiền | Biết tiền của mình đang ở đâu, biết bao giờ được giải quyết |
| **CSKH L1** | Nhân viên tuyến đầu, thẩm quyền duyệt < 200.000đ | Nhận case đã có sẵn kết luận, chỉ cần phán quyết nhanh |
| **CSKH L2** | Trưởng nhóm, thẩm quyền 200.000đ – 5.000.000đ | Xử lý case phức tạp, xem được toàn bộ căn cứ |
| **Risk team** | Thẩm quyền ≥ 5.000.000đ hoặc case có cờ nghi ngờ gian lận | Đánh giá rủi ro, xem lịch sử khách |
| **Admin / Dev** | Vận hành hệ thống | Cấu hình chính sách, xem trace và audit log |

---

## 3. User stories

| ID | Vai trò | Story | Tiêu chí chấp nhận |
|---|---|---|---|
| US-01 | Khách | Mô tả vấn đề bằng ngôn ngữ tự nhiên mà không cần biết mã giao dịch | Agent tự trích xuất được thông tin, hoặc hỏi tối đa 3 lượt rồi chuyển người |
| US-02 | Khách | Biết nguyên nhân giao dịch của mình gặp vấn đề | Nhận được giải thích kèm trạng thái thực tế trong dưới 15 giây |
| US-03 | Khách | Tra lại tình trạng khiếu nại vào hôm sau | Có mã ticket, tra được trạng thái hiện tại |
| US-04 | Khách | Được bảo vệ thông tin | Không bao giờ thấy dữ liệu giao dịch của người khác |
| US-05 | CSKH L1 | Mở hàng đợi và thấy case đã điều tra xong | Mỗi case hiển thị nguyên nhân, căn cứ chính sách, đề xuất, độ tin cậy |
| US-06 | CSKH L1 | Phê duyệt hoặc từ chối hoàn tiền trong vài giây | Có 4 lựa chọn: duyệt, từ chối, sửa số tiền, yêu cầu điều tra thêm |
| US-07 | CSKH L1 | Không vô tình duyệt vượt thẩm quyền | Hệ thống chặn và định tuyến lên cấp cao hơn |
| US-08 | CSKH L2 | Xem đầy đủ bằng chứng và cả lý do có thể từ chối | Hồ sơ đề xuất có mục phản chứng |
| US-09 | Admin | Truy vết một quyết định hoàn tiền bất kỳ | Audit log dựng lại đủ dòng thời gian: ai, lúc nào, căn cứ gì |
| US-10 | Admin | Cập nhật chính sách mà không sửa code | Chính sách nằm trong KB, đánh index lại là có hiệu lực |

---

## 4. Yêu cầu chức năng

### 4.A — Xác thực & phân quyền

| ID | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-A1 | Đăng nhập theo hai vai trò: khách hàng và nhân viên CSKH | Must |
| FR-A2 | Nhân viên có ba cấp thẩm quyền L1 / L2 / Risk, gán khi tạo tài khoản | Must |
| FR-A3 | Mọi truy vấn dữ liệu giao dịch bị kẹp bởi `customer_id` đã xác thực ở tầng service, không phụ thuộc prompt | Must |
| FR-A4 | Thao tác nhạy cảm (xem chi tiết giao dịch, khởi tạo yêu cầu hoàn tiền) yêu cầu step-up authentication | Should |

### 4.B — Giao diện chat khách hàng

| ID | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-B1 | Chat realtime, hiển thị trạng thái agent đang xử lý theo từng bước | Must |
| FR-B2 | Hiển thị báo cáo điều tra ở dạng dễ đọc, che thông tin nội bộ và tên đối tác | Must |
| FR-B3 | Hiển thị mã ticket khi tạo, cho phép tra cứu lại bằng mã | Must |
| FR-B4 | Ghi nhớ ngữ cảnh hội thoại trong phiên và lịch sử khiếu nại của khách qua các phiên | Should |
| FR-B5 | Thông báo cho khách khi yêu cầu hoàn tiền được duyệt hoặc từ chối | Must |

### 4.C — Option 1: Transaction Investigation Workflow

| ID | Node | Yêu cầu | Ưu tiên |
|---|---|---|---|
| FR-C1 | `authenticate` | Code thuần, xác thực và gán scope dữ liệu | Must |
| FR-C2 | `intake_and_classify` | LLM trả structured output gồm `complaint_category` và `extracted`, mỗi trường kèm confidence | Must |
| FR-C3 | `check_sufficiency` | Code tất định, mỗi category khai báo tập trường tối thiểu để tra cứu được | Must |
| FR-C4 | `clarify` | Sinh một câu hỏi nhắm vào một trường thiếu; trần 3 lượt, chạm trần thì escalate | Must |
| FR-C5 | `lookup_transactions` | Tool được bind theo category, LLM không tự chọn tool ngoài whitelist | Must |
| FR-C6 | `handle_not_found` | Phân biệt ba trường hợp: không tồn tại / ngoài phạm vi lưu trữ / thuộc khách khác. Trường hợp thứ ba trả lời trung tính, không xác nhận giao dịch tồn tại | Must |
| FR-C7 | `retrieve_policy` | RAG với metadata filter theo loại giao dịch và **hiệu lực chính sách tại thời điểm giao dịch** | Must |
| FR-C8 | `analyze_root_cause` | Sinh danh sách giả thuyết, mỗi giả thuyết bắt buộc có `evidence_refs` trỏ tới ledger event cụ thể | Must |
| FR-C9 | `verify_grounding` | Đối chiếu mọi số tiền và trạng thái trong kết luận với `txn_records`; lệch thì retry 1 lần rồi escalate | Must |
| FR-C10 | `generate_report` | Sinh hai bản: bản cho khách và bản cho nhân viên có đầy đủ trace | Must |
| FR-C11 | `route_next_action` | Định tuyến: đóng case / chờ đối tác / escalate / kích hoạt Option 2 | Must |

Phân loại khiếu nại được hỗ trợ: `MONEY_NOT_RECEIVED`, `DOUBLE_CHARGE`, `WRONG_RECIPIENT`, `PENDING_TOO_LONG`, `BALANCE_MISMATCH`, `REFUND_STATUS`, `NOT_A_COMPLAINT`.

### 4.D — Option 2: Refund Approval Workflow (HITL)

| ID | Node | Yêu cầu | Ưu tiên |
|---|---|---|---|
| FR-D1 | `precheck_hard_rules` | Code thuần, không có LLM. Kiểm: giao dịch tồn tại và thuộc khách, chưa từng hoàn, còn thời hiệu, số tiền hợp lệ, tài khoản không bị khóa, không có case refund đang mở cho cùng giao dịch | Must |
| FR-D2 | `evaluate_eligibility` | LLM + RAG, chỉ chạy sau khi qua hard rules, output kèm trích dẫn điều khoản | Must |
| FR-D3 | `compute_amount` | LLM chỉ chọn `refund_type` trong enum; **code tính ra con số** | Must |
| FR-D4 | `determine_authority` | Định tuyến theo ngưỡng số tiền và cờ rủi ro; cảnh báo khi vượt thẩm quyền | Must |
| FR-D5 | `build_proposal` | Hồ sơ gồm tóm tắt, nguyên nhân, bằng chứng, trích dẫn chính sách, số tiền, độ tin cậy, và **phản chứng** | Must |
| FR-D6 | `interrupt` | Dừng bền vững, state persist qua checkpointer; không tồn tại edge nào đi vòng qua node này | Must |
| FR-D7 | `human_decision` | Bốn lối ra: `APPROVE`, `REJECT`, `MODIFY_AMOUNT`, `REQUEST_MORE_INFO`. Sửa số tiền vượt thẩm quyền thì re-route | Must |
| FR-D8 | `verify_before_execute` | Chụp lại trạng thái giao dịch, so với `txn_snapshot`; lệch thì hủy thực thi và trả về hàng đợi kèm cảnh báo | Must |
| FR-D9 | `execute_refund` | Gọi API với idempotency key cố định; thất bại N lần thì chuyển `FAILED` tường minh và escalate | Must |
| FR-D10 | `notify` + `audit` | Thông báo khách, đóng ticket, ghi audit log | Must |
| FR-D11 | SLA timer | Quá hạn chờ duyệt thì nhắc, rồi tự escalate lên cấp cao hơn | Should |

Ngưỡng thẩm quyền:

```
amount < 200.000                      → L1
200.000 ≤ amount < 5.000.000          → L2
amount ≥ 5.000.000 hoặc fraud_flag    → RISK_TEAM
```

### 4.E — Dashboard nhân viên CSKH

| ID | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-E1 | Hàng đợi case, lọc theo trạng thái, cấp thẩm quyền, độ ưu tiên, hạn SLA | Must |
| FR-E2 | Màn chi tiết case hiển thị hồ sơ đề xuất đầy đủ kèm bằng chứng | Must |
| FR-E3 | Bốn nút hành động tương ứng FR-D7, có ô ghi chú bắt buộc khi từ chối hoặc sửa | Must |
| FR-E4 | Chỉ hiển thị case trong thẩm quyền của người đăng nhập | Must |
| FR-E5 | Xem timeline điều tra: các bước agent đã chạy, hệ thống đã kiểm tra, kết quả từng bước | Should |
| FR-E6 | Bảng chỉ số: số case, thời gian duyệt trung bình, tỷ lệ duyệt/từ chối | Could |

### 4.F — Ticketing

| ID | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-F1 | Tạo ticket tự động khi phân loại được khiếu nại | Must |
| FR-F2 | Trạng thái ticket: `OPEN` → `INVESTIGATING` → `PENDING_APPROVAL` → `RESOLVED` / `REJECTED` / `ESCALATED` | Must |
| FR-F3 | Ticket tồn tại độc lập với phiên chat, tra cứu được bằng mã | Must |
| FR-F4 | Gán người phụ trách và hạn SLA theo loại và độ ưu tiên | Should |

### 4.G — Mock Wallet API & dữ liệu

| ID | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-G1 | API mô phỏng: `get_transaction`, `search_transactions`, `get_ledger_events`, `get_partner_status`, `get_balance_snapshots`, `execute_refund` | Must |
| FR-G2 | Dữ liệu phủ đủ các kịch bản ở mục 6 | Must |
| FR-G3 | Ledger event có đủ các loại: `authorize`, `capture`, `settle`, `reverse`, `hold`, `hold_release` | Must |
| FR-G4 | `execute_refund` hỗ trợ idempotency key và mô phỏng được lỗi mạng / timeout | Must |
| FR-G5 | Có seed script tái tạo dữ liệu từ đầu | Must |

### 4.H — RAG chính sách

| ID | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-H1 | KB chính sách và FAQ ở dạng tài liệu, chunk kèm metadata: loại giao dịch, khoảng hiệu lực, mã điều khoản | Must |
| FR-H2 | Retrieval có filter metadata, không chỉ tìm theo ngữ nghĩa | Must |
| FR-H3 | Kết quả trả về kèm mã điều khoản để trích dẫn trong báo cáo | Must |
| FR-H4 | Cập nhật KB và đánh index lại không cần sửa code | Must |

### 4.I — Audit & observability

| ID | Yêu cầu | Ưu tiên |
|---|---|---|
| FR-I1 | Audit log append-only, ghi ở mọi node, không chỉ ở cuối luồng | Must |
| FR-I2 | Mỗi entry ghi: node, thời điểm, input hash, output tóm tắt, actor (agent hoặc user id) | Must |
| FR-I3 | Trace được toàn bộ vòng đời một case từ tin nhắn đầu tiên tới lúc tiền về | Must |
| FR-I4 | Log mọi tool call kèm tham số và kết quả | Must |

---

## 5. Mô hình dữ liệu

```
customers        (customer_id, name, phone, kyc_level, status)
wallets          (wallet_id, customer_id, balance, currency, status)
transactions     (txn_id, wallet_id, counterparty_id, amount, type,
                  status, partner_ref, created_at, settled_at)
ledger_events    (event_id, txn_id, event_type, amount, occurred_at, meta)
tickets          (ticket_id, customer_id, txn_id, category, status,
                  priority, assignee_id, sla_deadline, created_at)
complaint_cases  (case_id, ticket_id, state_json, root_cause,
                  confidence, report_customer, report_agent)
refund_cases     (refund_id, case_id, txn_snapshot, refund_type, amount,
                  authority_level, status, reviewer_id, reviewer_note,
                  idempotency_key, decided_at, executed_at)
policies         (policy_id, clause_code, content, txn_types,
                  effective_from, effective_to)
staff_users      (user_id, name, role, authority_level)
audit_logs       (log_id, case_id, node, actor, payload_summary, created_at)
```

Ràng buộc: `transactions`, `ledger_events` là nguồn sự thật, chỉ tool node được đọc và chỉ `execute_refund` được ghi. Không node LLM nào có quyền ghi vào hai bảng này.

---

## 6. Bộ kịch bản dữ liệu mock

Đây là yếu tố then chốt để hệ thống bộc lộ được năng lực suy luận. Tối thiểu phải phủ:

| # | Kịch bản | Mục đích kiểm thử |
|---|---|---|
| S1 | Giao dịch thành công bình thường | Đường cơ sở, không phải khiếu nại |
| S2 | Đối tác chậm settle, đang `PENDING` | Phân biệt lỗi thật với chờ hợp lệ |
| S3 | Double capture thật | Xác định đúng nguyên nhân, đủ điều kiện hoàn |
| S4 | Một giao dịch + một khoản hold chưa release | Trông giống S3 nhưng **không** phải, kiểm tra khả năng phân biệt |
| S5 | Giao dịch fail nhưng chưa rollback | Đây là reversal, không phải refund |
| S6 | Chuyển đúng kỹ thuật nhưng sai người nhận | Ví không chịu trách nhiệm, đi luồng khác |
| S7 | Đã được hoàn tiền trước đó | Hard rule chặn double refund |
| S8 | Giao dịch không tồn tại | Nhánh `handle_not_found` |
| S9 | Giao dịch tồn tại nhưng thuộc khách khác | Kiểm thử PDPA, không được lộ |
| S10 | Giao dịch quá cũ, ngoài phạm vi lưu trữ | Escalate đúng cách |
| S11 | Nhiều giao dịch cùng số tiền trong cửa sổ 10 phút | Nhánh disambiguate |
| S12 | Giao dịch số tiền lớn, vượt thẩm quyền L1 | Kiểm thử định tuyến thẩm quyền |
| S13 | Trạng thái thay đổi giữa lúc đề xuất và lúc duyệt | Kiểm thử `verify_before_execute` |

---

## 7. Yêu cầu phi chức năng

| Loại | Yêu cầu |
|---|---|
| **Hiệu năng** | P95 thời gian điều tra end-to-end < 15 giây. Mock API phản hồi < 500ms. |
| **Chi phí** | Trần 8 lần gọi LLM cho một lượt điều tra; vượt trần thì escalate thay vì lặp tiếp. |
| **Bảo mật** | JWT + RBAC. Scope dữ liệu kẹp ở tầng service. Không log dữ liệu nhạy cảm dạng thô. |
| **Tính đúng đắn** | Tỷ lệ vi phạm grounding phải bằng 0 trên bộ test. Tỷ lệ hoàn tiền không qua HITL phải bằng 0. |
| **Độ tin cậy** | Idempotency cho mọi thao tác ghi tài chính. State persist, resume được sau restart. |
| **Quan sát được** | Trace theo `case_id` xuyên suốt mọi node. |
| **Triển khai** | Docker Compose chạy được local; deploy cloud có domain công khai. |

---

## 8. Kế hoạch đánh giá

Bộ golden dataset gồm tối thiểu 40 case trải đều 13 kịch bản ở mục 6, mỗi case có nhãn kỳ vọng về category, root cause, và quyết định hoàn tiền.

| Chỉ số | Ngưỡng đạt |
|---|---|
| Độ chính xác phân loại khiếu nại | ≥ 90% |
| F1 trích xuất thông tin giao dịch | ≥ 85% |
| Độ chính xác xác định nguyên nhân gốc | ≥ 85% |
| Tỷ lệ vi phạm grounding | 0% |
| Tỷ lệ gọi tool sai tham số | ≤ 2% |
| Tỷ lệ hoàn tiền không qua HITL | 0% |
| Tỷ lệ phân biệt đúng S3 với S4 | ≥ 80% |

Ba chỉ số bắt buộc bằng 0 hoặc 100% là ràng buộc tuân thủ, không phải chỉ tiêu chất lượng — không đạt thì coi như fail bất kể các chỉ số khác.

---

## 9. Tiêu chí hoàn thành

**Mức cơ bản (theo đề bài)**

- [ ] Web deploy công khai, đăng nhập được hai vai trò
- [ ] Agent chat trả lời, tra cứu giao dịch mô phỏng
- [ ] Phân loại khiếu nại và tạo ticket

**Mức nâng cao (theo đề bài)**

- [ ] Điều tra đa bước tự động, có giải thích và căn cứ
- [ ] Đề xuất hoàn tiền với HITL phê duyệt hoạt động đầy đủ
- [ ] Memory hội thoại và lịch sử khách hàng
- [ ] Escalation thông minh theo confidence và thẩm quyền
- [ ] Xử lý đúng khi không tìm thấy giao dịch (đủ ba trường hợp)
- [ ] Cảnh báo khi vượt thẩm quyền

---

## 10. Lộ trình

| Track | Module | Effort |
|---|---|---|
| **Option 1** | Information Extraction | 1 ngày |
| | Mock Wallet API & Transaction DB | 1,5 ngày |
| | Transaction Lookup Tool | 2 ngày |
| | Agent Reasoning Engine | 2,5 ngày |
| | Policy Retrieval Integration | 1 ngày |
| | Investigation Report Generator | 1 ngày |
| | **Tổng** | **≈ 9 ngày** |
| **Option 2** | Refund Eligibility Engine | 2 ngày |
| | Refund Proposal Generator | 1 ngày |
| | HITL Queue & State Management | 2 ngày |
| | Approval Dashboard | 1,5 ngày |
| | Refund Execution API | 1,5 ngày |
| | Notification & Audit Log | 1 ngày |
| | **Tổng** | **≈ 9 ngày** |

Mốc chung: Mock API + seed data sẵn sàng → Option 1 chạy end-to-end → Option 2 + dashboard → tích hợp, eval trên golden set, deploy.

Phụ thuộc: Option 2 không khởi động được trước khi Mock Wallet API và schema `transactions` ổn định. Đây là đường găng của dự án.

---

## 11. Câu hỏi còn mở

1. Chính sách hoàn tiền lấy từ đâu — nhóm tự soạn giả lập hay dựa trên chính sách công khai của một ví thật?
2. Ngưỡng thẩm quyền ở mục 4.D là giả định của nhóm, cần chốt lại có phù hợp với kỳ vọng chấm điểm không.
3. Step-up authentication (FR-A4) có bắt buộc ở phiên bản demo không, hay chỉ mô tả trong tài liệu?
4. Memory lịch sử khách hàng lưu ở mức nào — chỉ ticket cũ, hay tóm tắt hội thoại các phiên trước?