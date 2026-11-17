# Báo cáo nhóm — Lab 8 LogiX-VN

## 1. Thông tin nhóm

- Tên nhóm: FreeFire
- Lớp: L3A
- Repo: `K5-L3b-DAY08-FreeFire-FromWorkflowsToAgents`
- URL repo: `https://github.com/vietahnn/K5-L3b-DAY08-FreeFire-FromWorkflowsToAgents.git`, nhánh `main`
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): `e2d0c9b`

## 2. Mục tiêu và cách tiếp cận

Bài toán Lab 8 đặt ra tình huống xử lý khiếu nại (claims processing) bồi thường phí vận chuyển và hư hỏng hàng hóa tại công ty giả lập LogiX-VN. Trọng tâm của bài lab không phải là xây dựng một hệ thống multi-agent phức tạp nhất có thể, mà là thực nghiệm khảo sát trên **bốn kiến trúc khác nhau** (Workflow, Single Agent, Orchestrator, Swarm) để tìm ra kiến trúc **ít tự chủ nhất mà vẫn hoàn thành tốt công việc**.

Cách tiếp cận của nhóm:
1. **Thiết lập ranh giới bất biến (Hard Security & Business Boundary)**: Mọi thao tác tính tiền, kiểm tra approval, trần hoàn tiền theo đơn (order cap) và ghi sổ cái (ledger) phải do code quản lý trong giao dịch SQLite nguyên tử; Model chỉ được phép đọc và soạn thảo văn bản phản hồi.
2. **Triển khai tuần tự và kiểm thử chặt chẽ theo từng Checkpoint**: Đi từ việc xây dựng 3 shape (CP2), củng cố khả năng chịu lỗi và an toàn ranh giới handoff (CP3), kiểm thử crash và secret leak (CP4), đến đo lường định lượng trên tập benchmark đa seed để đưa ra quyết định kiến trúc dựa trên dữ liệu (CP5).

## 3. Giải pháp

### 3.1. Ranh giới Code / Model
- **Code (Host)**: Thực hiện kiểm tra quyền sở hữu đơn (`principal_id`), hiệu lực điều khoản (`is_current` và `effective_date <= 2026-11-15`), kiểm tra approval hợp lệ gắn đúng với claim, trần tích lũy bồi thường trên cùng một đơn hàng (`order_already_refunded`), và đảm bảo tính bất biến Idempotency (`refund:{claim_id}`).
- **Model**: Đóng vai trò soạn thảo câu trả lời cho khách hàng hoặc đề xuất bước đọc dữ liệu trong ReAct loop; tuyệt đối không cấp công cụ ghi (`WRITE_TOOLS`) cho model.

### 3.2. Chi tiết ba shape do nhóm triển khai
- **Shape A (Workflow)**: Thực hiện luồng cố định bằng Python. Đọc claim an toàn qua `get_claim_view`, tra cứu đơn hàng qua `lookup_order`, tính tiền qua `compute_compensation` từ thông tin đơn thật, và gọi `durability.issue_refund` với quote nguyên vẹn.
- **Shape B (Single ReAct Agent)**: Vận hành vòng lặp ReAct tối đa 8 bước qua `run_react`. Hàm `decide` chỉ được gọi các công cụ đọc (`lookup_order`, `compute_compensation`). Host đứng ngoài vòng lặp là thực thể duy nhất gọi `issue_refund`.
- **Shape C (Orchestrator)**: Khởi tạo gói handoff chuẩn hóa qua `lab8.handoff.make_handoff` chỉ chứa facts trong allowlist và các tool đọc. Các worker investigator hoạt động độc lập và chỉ đọc; Orchestrator thu thập dữ liệu và gọi ghi refund.

### 3.3. Handoff an toàn & Trace contract v1
- **Sanitized Handoff**: Loại bỏ hoàn toàn `parent_secret`, authorization headers và nội dung ticket thô trước khi chuyển giao cho worker.
- **Trace v1**: Mỗi lượt chạy sinh ra file JSON tương ứng trong `src/traces/` chứa đúng `run_id` và danh sách sự kiện có cấu trúc (`claim`, `order_view`, `quote`, `decision`, `crash`). Cố ý không ghi các trường PII (số điện thoại, địa chỉ, tên hàng hóa) và secret.

## 4. Kết quả thực nghiệm

### 4.1. Bảng số liệu đối chứng (Trích xuất từ `src/reports/benchmark.json` - 280 trials)

| Shape | Pass rate | Pass^5 | Seed min-max | Seed variance | Cost/Success (VND) | Mean steps | p95 steps |
|---|---:|---:|---:|---:|---:|---:|---:|
| **Workflow (A)** | **1.000** | **1.000** | **1.000 – 1.000** | **0.000000** | **28.79** | 2.86 | 3.0 |
| **Single Agent (B)** | **1.000** | **1.000** | **1.000 – 1.000** | **0.000000** | 29.52 | 2.00 | 2.0 |
| **Orchestrator (C)** | **1.000** | **1.000** | **1.000 – 1.000** | **0.000000** | 47.36 | 3.79 | 4.0 |
| **Swarm (D - Ref)** | 0.986 | 0.929 | 0.929 – 1.000 | 0.000816 | 49.57 | 2.80 | 3.0 |

### 4.2. Khả năng chịu lỗi và an toàn bảo mật
- **Crash Safety**: Đạt **20/20** theo kết quả kiểm thử ngẫu nhiên trong `src/reports/crash.json`.
- **Approval Bypass**: **0** vi phạm (100% trường hợp thiếu approval hoặc sai claim bị từ chối và ghi audit `policy_denied`).
- **Secret Leak**: **0** vi phạm (không rò rỉ canary secret trong trace hay kết quả).

### 4.3. Quyết định kiến trúc & Nghiên cứu khoa học
- **Quyết định**: Chọn **Shape A (Workflow)**. Chi tiết lập luận và phân tích đánh đổi nằm trong [`report/team/decision_adr.md`](decision_adr.md).
- **Ablation R2 (Failure Taxonomy - MAST NeurIPS 2025)**: Phân tích nguyên nhân Swarm thất bại tại seed 55 claim C-002 do lỗi kiểm chứng đầu vào; xem chi tiết trong [`report/team/research_card.md`](research_card.md).

## 5. Khó khăn và cách xử lý

1. **Rủi ro Double-pay khi gặp sự cố crash trước checkpoint**:
   - *Khó khăn*: Nếu chỉ dựa vào checkpoint lưu sau khi thực hiện hành động, khi crash xảy ra ngay sau khi chuyển tiền nhưng trước khi checkpoint kịp ghi, lượt chạy lại sẽ thực hiện lại việc chuyển tiền, dẫn đến chi trả hai lần.
   - *Cách xử lý*: Đưa việc ghi sổ cái `refund_ledger` và audit log `refund_committed` vào chung một SQLite transaction có khóa duy nhất `refund:{claim_id}`, biến việc chạy lại thành thao tác idempotent an toàn tuyệt đối.
2. **Trần hoàn tiền tích lũy theo đơn hàng (Per-order Cap)**:
   - *Khó khăn*: Các đơn hàng có thể phát sinh nhiều claim qua thời gian (ví dụ claim hoàn phí ship trước, claim hư hỏng hàng sau).
   - *Cách xử lý*: Shape luôn truyền **toàn bộ quote** vào host `durability.issue_refund`, để host tự truy vấn tổng tiền đã hoàn của đơn hàng và chỉ chi trả phần chênh lệch entitlement, tránh việc shape tự trừ khiến tiền bị trừ hai lần.

## 6. Tổng hợp đóng góp

Chi tiết phân công, mã commit và các test kiểm thử của từng thành viên được khai báo minh bạch tại `TEAM.md` (mục `## Bảng đóng góp`) và các file báo cáo cá nhân `report/team/<MSSV>_<HoTen>.md`. Nhóm đã bổ sung bộ kiểm thử `src/tests/test_team_verification.py` bao phủ các trường hợp biên quan trọng (ngưỡng trễ < 3 ngày, điều khoản cũ hết hạn, chống prompt injection, và tính bất biến khi retry nhiều lần).

## 7. Hạn chế và hướng cải thiện

- **Giới hạn của MockLLM**: Thử nghiệm sử dụng mô hình MockLLM giả lập để cố định seed và phục vụ kiểm thử determinism trong phòng lab. Trong thực tế triển khai sản xuất với LLM thật, chi phí và độ trễ sẽ biến thiên đáng kể.
- **Hướng cải thiện**: Khi mở rộng quy trình tiếp nhận khiếu nại phức tạp hơn (như thẩm định hình ảnh hàng vỡ thật qua Multimodal Model hoặc trao đổi trực tiếp với khách hàng), có thể kết hợp mô hình Hybrid: sử dụng Agent ở tầng giao tiếp/phân loại phi cấu trúc ở đầu vào, nhưng bắt buộc ủy quyền tính toán tiền và ghi nhận giao dịch cho Workflow cố định ở tầng thực thi.

## 8. Tài liệu tham khảo

1. Google Research (2026), *Towards a Science of Scaling Agent Systems*.
2. MAST (NeurIPS 2025), *Why Do Multi-Agent LLM Systems Fail?*.
3. Tài liệu kiến trúc và hợp đồng bài lab: `docs/business_rules.md`, `docs/contracts.md`, `docs/architecture.md`.
