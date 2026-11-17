# Báo cáo cá nhân — 2A202603946_NguyenQuangTiep

## 1. Thông tin thành viên

- Họ tên: Nguyễn Quang Tiệp
- MSSV: 2A202603946
- Email: tiepquangnguyen175@gmail.com
- GitHub: nguyenquangtiep

## 2. Phần việc phụ trách

Checkpoint 4 (CP4): chạy crash suite 20 điểm và nộp bằng chứng `src/reports/crash.json`, xác nhận ba điều kiện must-pass của host (không refund khi thiếu approval hợp lệ, không lộ `CANARY_SECRET_*`, crash không tạo hơn một refund) vẫn giữ trên code shape hiện tại của nhóm.

## 3. Đóng góp cụ thể

- Commit chính: `e2d0c9b` — *add checkpoint 4*: thêm `src/reports/crash.json` do `python -m lab8.crash_test --count 20 --out reports/crash.json` sinh ra, kết quả `crash_safety` = `20/20`.
- Kiểm tra tính tái lập của report: chạy lại crash suite trên commit `4eb8dc8` (code shape sau CP3) và so sánh với file đã nộp — hai file giống hệt nhau từng byte. Report không chứa timestamp hay run id thay đổi, nên giảng viên chạy lại sẽ ra cùng nội dung (tiêu chí C7).
- Rà cổng host và ma trận student trước khi nộp: `python -m pytest -q` → 54 passed; `python -m pytest -q -m student -o addopts=` → 248 passed, 0 failed (gồm T11 gọi lại cùng claim, T12 crash rồi resume, các test approval và per-order cap).
- Xác nhận cả ba shape truyền `crash_at` vào `durability.issue_refund` và không nuốt `InjectedCrash`: shape ghi event `crash` vào trace, ghi trace ra file rồi `raise` lại, để test T12 thấy crash thật và grader vẫn có trace hợp lệ.
- Hỗ trợ môi trường: xử lý lỗi cài PyYAML trên Python 3.15 (không có wheel, đòi Visual C++ Build Tools) bằng cách tạo lại venv với Python 3.12 (`py -3.12 -m venv .venv`), đúng phạm vi 3.11–3.13 lab hỗ trợ.
- Test đã thêm hoặc sửa: <bổ sung tên test sau khi commit>

## 4. Bằng chứng

- Commit: `e2d0c9b` trên nhánh `main` của repo nhóm (https://github.com/vietahnn/K5-L3b-DAY08-FreeFire-FromWorkflowsToAgents/commit/e2d0c9b).
- Trích `src/reports/crash.json`:

  ```json
  {
    "count": 20,
    "passed": 20,
    "crash_safety": "20/20",
    ...
  }
  ```

- Lệnh tái lập (chạy trong `src/`):

  ```text
  python -m lab8.crash_test --count 20 --out reports/crash.json
  -> crash_safety 20/20
  ```

- Ma trận student: `248 passed, 54 deselected`.

## 5. Công cụ AI đã dùng

- **Claude Code (VS Code extension):** đọc và tóm tắt đề (README, CHECKPOINTS, RUBRIC, RULES, contract), chẩn đoán lỗi cài PyYAML, so sánh folder được gửi với repo nhóm, chạy test và crash suite.
- **Cách tự kiểm tra:** không chép số liệu từ AI mà chạy lại lệnh trên máy mình; so khớp `crash.json` với một lần chạy mới bằng `git diff --no-index`; đối chiếu yêu cầu với chính văn bản `RUBRIC.md` và `SUBMISSION.md`. AI không dùng để tạo hay sửa nội dung JSON trong `src/reports/`.

## 6. Tự đánh giá

- **Làm được:** nộp đúng bằng chứng CP4 do công cụ sinh ra, có kiểm tra tái lập; hiểu vì sao ledger với idempotency key `refund:{claim_id}` mới bảo đảm một refund khi crash, còn checkpoint đơn lẻ thì không.
- **Còn thiếu:** <test cá nhân — cập nhật khi đã thêm>. Crash suite chỉ kiểm host và swarm; độ an toàn khi crash của ba shape nhóm viết được chứng minh qua T12 trong ma trận student chứ không qua crash suite.
