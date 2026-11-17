# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 32f7e07
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.0 / 1.0 / 1.0-1.0 / 0.0
- Cost per success (mô phỏng, VND): 28.79
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

## Vì sao shape này

Nhóm FreeFire chọn shape `workflow` theo nguyên tắc kiến trúc "Ít tự chủ nhất mà vẫn hoàn thành tốt công việc". Dựa trên số đo đối chứng 280 trials (4 shape x 14 claim x 5 seed) trong `src/reports/benchmark.json`, `workflow` đạt kết quả lý tưởng tuyệt đối: Pass rate = 1.0 (70/70 trial), Pass^5 = 1.0 (14/14 claim pass cả 5 seed), Seed variance = 0.000000, và Chi phí trên mỗi ca thành công là thấp nhất (28.79 VND so với 29.52 VND của single, 47.36 VND của orchestrator, 49.57 VND của swarm). Quy trình đối soát bồi thường tại LogiX-VN có luật nghiệp vụ xác định và ranh giới rõ ràng; việc dùng code trực tiếp kiểm tra và tính toán entitlement từ SQLite loại bỏ triệt để rủi ro hallucination hay bị thao túng bởi văn bản ticket từ phía khách hàng (điểm yếu đã khiến Swarm fail ở seed 55 tại T02 khi rớt pass rate xuống 0.929).

## Có đáng đổi lấy multi-agent không

Hoàn toàn không đáng. So sánh trên cùng corpus và cùng seed, các kiến trúc tự chủ cao hơn (Orchestrator, Swarm) không nâng cao tỷ lệ thành công mà ngược lại làm chi phí mô phỏng tăng vọt từ 64% đến 72% và đưa vào phương sai (variance > 0). Nhóm chỉ cân nhắc chuyển đổi sang multi-agent khi bài toán xuất hiện các yếu tố phi cấu trúc đòi hỏi năng lực reasoning và phân vai động: ví dụ như đối thoại đa lượt đàm phán phương án đền bù với khách hàng, hoặc phối hợp giữa agent thị giác (đọc ảnh hư hỏng hàng hóa) và agent pháp lý (diễn giải điều khoản bảo hiểm phức tạp) - những ngữ cảnh mà code tuần tự cố định không thể lập trình trước mọi phân nhánh.

## Trace và bằng chứng

Trace của shape `workflow` ghi lại các event có cấu trúc: `claim` (chỉ ghi claim_id và order_id), `order_view` (trạng thái hiển thị và lý do phạm vi), `quote` (chi tiết điều khoản và số tiền tính bằng code), `decision` và `crash` (phục vụ kiểm thử lỗi). Shape cố ý **KHÔNG** ghi các field: `parent_secret`, raw authentication headers, số điện thoại, địa chỉ nhà, tên hàng hóa chi tiết và ghi chú nội bộ. Cơ chế phòng thủ được đặt ngay tại cổng đọc: `get_claim_view` và `lookup_order` chỉ trả về danh sách thuộc tính cho phép (`ORDER_FIELDS`). Nếu file trace nằm ngoài phạm vi quét tự động, đội bảo mật có thể chứng minh tính sạch sẽ thông qua code audit: mọi thao tác tạo payload cho `trace.add()` đều chỉ nhận dữ liệu đã qua allowlist filter, ngăn chặn rò rỉ secret ngay từ thiết kế.

## Giới hạn

Thử nghiệm được thực hiện trên MockLLM cố định theo seed để đảm bảo tính tái lập (reproducibility) trong khuôn khổ lab. Chi phí token là giá mô phỏng theo `config/pricing.yaml`, không đại diện cho hóa đơn API thực tế khi triển khai mô hình thương mại. Độ trễ (latency) của MockLLM được báo cáo là `unavailable`. Nhóm cam kết không sửa đổi bất kỳ file JSON bằng chứng hay file thước đo nào.
