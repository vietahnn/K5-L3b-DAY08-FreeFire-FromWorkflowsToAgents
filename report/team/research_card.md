# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | MAST, NeurIPS 2025, https://arxiv.org/abs/2411.08442, benchmark & empirical failure taxonomy |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Hệ thống Multi-Agent thường thất bại do lỗi kiểm chứng thông tin (verification failure) khi tiếp nhận dữ liệu không tin cậy từ ngữ cảnh bên ngoài |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Môi trường xử lý khiếu nại đối soát tự động có dữ liệu ticket người dùng mâu thuẫn với cơ sở dữ liệu hệ thống |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Đổi Swarm (D) sang Workflow (A) sẽ triệt tiêu lỗi kiểm chứng, nâng pass rate từ 0.986 lên 1.000, đưa variance từ 0.0008 về 0 và giảm 42% cost/success |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Giữ nguyên MockLLM, bộ tool, 5 seed (11,22,33,44,55), 14 claim, pricing.yaml; Biến thay đổi duy nhất là Topology kiểm soát: Workflow (code quyết định) vs Swarm (cho phép ghi đè) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi kiểm chứng (Verification Failure) tại C-002 seed 55; Đo bằng Pass rate, Pass^5, Seed variance và Cost per success |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `src/reports/benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Đúng hoàn toàn. Tại seed 55 với claim C-002, Swarm gặp lỗi kiểm chứng thông tin (tin ticket 5 ngày thay vì dữ liệu DB 2 ngày), khiến pass rate giảm xuống 0.9857 (pass^5 = 0.9286), seed variance tăng lên 0.000816 và chi phí đẩy lên 49.57 VND/success. Ngược lại, Workflow loại bỏ hoàn toàn lỗi kiểm chứng nhờ code validation, đạt pass rate 1.000, pass^5 1.000, seed variance 0.000000 và chi phí chỉ 28.79 VND/success (tiết kiệm 41.9% chi phí so với Swarm).
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.
