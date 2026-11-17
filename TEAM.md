# Team

## Thông tin nhóm

- Tên nhóm: FreeFire
- Lớp: L3A
- Repo nhóm: `K5-L3b-DAY08-FreeFire-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/vietahnn/K5-L3b-DAY08-FreeFire-FromWorkflowsToAgents.git, nhánh `main`

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Lê Tuấn Đạt | 2A202603949 | letuandat.cv@gmail.com | letuandat131 | Benchmark, verification tests (`test_team_verification.py`), ADR & báo cáo |
|   2 | Vũ Huy Hiệu | 2A202603979 | hieuvippro531@gmail.com | hieunhangia | Triển khai Shape A (Workflow), Shape B (Single Agent), ReAct loop & Trace contract |
|   3 | Nguyễn Quang Tiệp | 2A202603946 | tiepquangnguyen175@gmail.com | nguyenquangtiep | CP4: crash suite 20/20 (`src/reports/crash.json`), rà approval gate & secret/PII |
|   4 | Đỗ Việt Anh | 2A202603973 | dovietanh240405@gmail.com | vietahnn | CP3: Durability, idempotency & crash handling trên 3 shape, xử lý InjectedCrash & trace |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Lê Tuấn Đạt (`2A202603949`) | Benchmark CP5, verification tests, ADR & báo cáo | `32f7e07` | `src/tests/test_team_verification.py` (12 cases: delay < 3d, superseded policy, prompt injection, idempotency) |
| Vũ Huy Hiệu (`2A202603979`) | Triển khai Shape A (Workflow) & Shape B (Single Agent), ReAct runtime & Trace contract | `809c0ff` | `tests/test_public_matrix.py` (62 cases workflow & 62 cases single agent) |
| Nguyễn Quang Tiệp (`2A202603946`) | CP4: crash suite, approval gate, secret/PII | `e2d0c9b` | <bổ sung tên test sau khi commit> |
| Đỗ Việt Anh (`2A202603973`) | CP3: Durability, idempotency & crash handling trên 3 shape (`workflow`, `single`, `orchestrator`) | `4eb8dc8` | `tests/test_public_matrix.py` (T11 retry idempotency, T12 crash before checkpoint) & `tests/test_refund_idempotency.py` |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## 2A202603949_LeTuanDat

### Phần việc phụ trách

- Hoàn thiện bộ verification tests (`src/tests/test_team_verification.py`), chạy tổng hợp `src/reports/benchmark.json`, hoàn thiện hồ sơ CP5 (`report/team/decision_adr.md`, `report/team/research_card.md`, `report/team/group_report.md`) và kiểm tra điều kiện nộp (`lab8.check_submission`).

### Đóng góp cụ thể

- Bổ sung `src/tests/test_team_verification.py` (12 test cases trên 3 shape `workflow`, `single`, `orchestrator`) kiểm tra 4 điều kiện biên:
  - Ngưỡng trễ `< 3` ngày bị từ chối (`C-002`).
  - Policy cũ `SHIP-LATE-OLD` bị bỏ qua (`C-006`).
  - Prompt injection trong nội dung ticket (`C-009`) không làm đổi quyết định từ chối.
  - Gọi lại cùng claim `C-001` 3 lần liên tiếp vẫn chỉ ghi 1 dòng trong `refund_ledger` và 1 `refund_committed` trong `audit_events`.
- Đưa `src/reports/benchmark.json` (4 shape × 14 claim × 5 seed = 280 trials) cùng phân tích ADR (`report/team/decision_adr.md`), `report/team/research_card.md` và `report/team/group_report.md` vào nhánh `main`.
- Cập nhật `TEAM.md` với thông tin thành viên và bảng đóng góp.

### Bằng chứng

- File test mới: `src/tests/test_team_verification.py` (`12 passed`).
- Báo cáo benchmark: `src/reports/benchmark.json` (`workflow`: `Pass rate = 1.0`, `Pass^5 = 1.0`, `variance = 0.0`, `cost/success = 28.79 VND`, `Run id = R-workflow-C-001-s11`).
- Crash safety: `src/reports/crash.json` (`20/20`).
- Kiểm tra toàn bộ test và submission: `python -m pytest -q` (`66 passed`), `python -m pytest -q -m student -o addopts=` (`248 passed`), `python -m lab8.check_submission` (`0 FAIL`).

## 2A202603979_VuHuyHieu

### Phần việc phụ trách

- Triển khai kiến trúc Shape A (`workflow.py`) và Shape B (`single.py`).
- Xây dựng vòng lặp Bounded ReAct cho Single Agent, giới hạn nghiêm ngặt ở các read-only tool và phân lập quyền ghi sổ cái cho host.
- Đảm bảo tuân thủ Trace contract v1, không để lộ PII và secrets (`CANARY_SECRET_*`).

### Đóng góp cụ thể

- Hoàn thiện `src/lab8/shapes/workflow.py`: đọc claim view qua `get_claim_view`, đối chiếu `lookup_order`, tính tiền qua `compute_compensation` từ DB thay vì ticket text, và hoàn tất kết quả bằng `finish()`.
- Hoàn thiện `src/lab8/shapes/single.py`: cài đặt ReAct loop tối đa 8 bước qua `run_react`, hàm `decide` chỉ gọi read-only tool, host đóng vai trò writer độc quyền.
- Thực hiện kiểm thử 62 test cases của workflow và 62 test cases của single trong ma trận student `tests/test_public_matrix.py`, đạt tỷ lệ pass 100%.
- Commit kỹ thuật: `809c0ff` (`implement cp1, cp2`).

### Bằng chứng

- Commit: `809c0ff` (`implement cp1, cp2`).
- Test xanh: `python -m pytest -q -m student -o addopts= -k "workflow or single"` (122 passed).
- Trace log sinh ra sạch, tuân thủ `trace.v1`.

## 2A202603946_NguyenQuangTiep

### Phần việc phụ trách

- Checkpoint 4 (CP4): chạy crash suite 20 điểm, nộp bằng chứng `src/reports/crash.json`, xác nhận ba điều kiện must-pass (không refund khi thiếu approval hợp lệ, không lộ `CANARY_SECRET_*`, crash không tạo hơn một refund).

### Đóng góp cụ thể

- Sinh và commit `src/reports/crash.json` bằng `python -m lab8.crash_test --count 20 --out reports/crash.json` (`crash_safety` = `20/20`).
- Kiểm tra tái lập: chạy lại crash suite trên code shape sau CP3 (`4eb8dc8`), file mới trùng từng byte với file đã nộp.
- Xác nhận ba shape truyền `crash_at` vào `durability.issue_refund` và không nuốt `InjectedCrash` (ghi trace rồi `raise` lại), để T12 phát hiện crash thật.
- Xử lý lỗi môi trường: PyYAML không build được trên Python 3.15 → tạo lại venv bằng Python 3.12.
- Báo cáo cá nhân: `report/team/2A202603946_NguyenQuangTiep.md`.

### Bằng chứng

- Commit: `e2d0c9b` (`add checkpoint 4`).
- `src/reports/crash.json`: `"crash_safety": "20/20"`.
- `python -m pytest -q -m student -o addopts=` (`248 passed`).

## 2A202603973_DoVietAnh

### Phần việc phụ trách

- Checkpoint 3 (CP3): Cơ chế Độ bền vững (Durability), Tính bất biến khi thực hiện lại (Idempotency) và Xử lý lỗi crash trên cả ba shape (`workflow`, `single`, `orchestrator`).
- Truyền `crash_at` vào `durability.issue_refund`, không nuốt ngoại lệ `InjectedCrash` (bắt lỗi để ghi event crash vào trace, flush trace ra đĩa rồi re-raise để test/host phát hiện).
- Đảm bảo an toàn sổ cái (ledger idempotency key `refund:{claim_id}`) và khả năng phục hồi khi hệ thống gặp sự cố giữa chừng.

### Đóng góp cụ thể

- Hoàn thiện `src/lab8/shapes/workflow.py`, `src/lab8/shapes/single.py`, `src/lab8/shapes/orchestrator.py`:
  - Thêm tham số `crash_at: str | None = None` vào `run_claim` và `_run`.
  - Chuyển tiếp `crash_at` trực tiếp vào `durability.issue_refund(conn, claim_id, proposal, crash_at=crash_at)`.
  - Bắt `InjectedCrash`: ghi sự kiện crash vào trace bằng `trace.add("crash", {"at": crash_at})`, flush trace ra đĩa với `trace.write()`, rồi `raise` lại exception để test T12 thấy crash thật mà grader vẫn có trace hợp lệ.
- Kiểm thử xác minh T11 và T12 trên cả ba shape trong ma trận student `tests/test_public_matrix.py` và bộ kiểm thử độ bền `tests/test_refund_idempotency.py`.
- Commit kỹ thuật chính: `4eb8dc8` (`implement cp3`).
- Báo cáo cá nhân: `report/team/2A202603973_DoVietAnh.md`.

### Bằng chứng

- Commit: `4eb8dc8` (`implement cp3`).
- Test xanh: `python -m pytest -q -m student -o addopts= -k "test_t11 or test_t12"` (passed trên cả 3 shape).
- Đảm bảo tiền đề vững chắc cho crash safety `20/20` ở CP4 (`src/reports/crash.json`).
