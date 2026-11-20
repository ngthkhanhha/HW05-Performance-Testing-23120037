# HW05 Kiểm thử hiệu năng EShop

**Mã số sinh viên:** 23120037  
**SUT:** EShop REST API, Node.js/Express và SQLite  
**Công cụ:** Apache JMeter 5.6.3, OpenJDK 17.0.21, Task Manager  
**Bản báo cáo:** Markdown; người học sẽ tự chuyển sang PDF và đóng gói ZIP sau khi chốt video demo.

## Tóm tắt kết quả

| Kịch bản | Tải cấu hình | Thời lượng | Mẫu JTL | Lỗi JTL | Thông lượng ghi nhận | Bằng chứng |
|---|---:|---:|---:|---:|---:|---|
| Load | 5 người dùng, ramp-up 30 giây | 120 giây | 424 | 0 | 3,58 yêu cầu/giây | `plans/23120037_Load_20261120.jmx`, `results/load/`, `evidence/load_run_01.png` |
| Stress | 15 người dùng, ramp-up 30 giây | 120 giây | 1.258 | 0 | 10,60 yêu cầu/giây | `plans/23120037_Stress_20261120.jmx`, `results/stress/`, `evidence/stress_run_01.png` |
| Spike | 15 người dùng, ramp-up 1 giây | 60 giây | 706 | 0 | 12,00 yêu cầu/giây | `plans/23120037_Spike_20261120.jmx`, `results/spike/`, `evidence/spike_run_01.png` |
| Endurance | 5 người dùng, ramp-up 30 giây | 600 giây | 2.336 | 0 | 3,90 yêu cầu/giây | `plans/23120037_Endurance_20261120.jmx`, `results/endurance/`, `evidence/endurance_run_01.png` |

Cả bốn kế hoạch đo cùng luồng Login → Product List → Add to Cart → Checkout, tương ứng các nhóm API xác thực, đọc và giao dịch. Dữ liệu nằm trong `data/purchase_data.csv`. Ba kịch bản chính dùng ba loại listener riêng: Summary Report, Aggregate Report và Graph Results.

## Mốc Endurance quan sát được

Ở cấu hình hiện có, bài Endurance chạy khoảng 598,5 giây với 5 người dùng, 2.336 mẫu thành công, p95 tổng thể 8 ms và thông lượng 3,90 yêu cầu/giây. Bài Stress cao nhất đạt 15 người dùng, 10,60 yêu cầu/giây, p95 8 ms và 0% lỗi. Báo cáo ghi nhận đã xác nhận ổn định đến 15 người dùng; **chưa xác định ngưỡng tối đa của phần cứng**. Theo lựa chọn của người học, không chạy thêm bài tăng tải để tiết kiệm thời gian. Mức RAM 57% chỉ thấy tại ảnh khởi động sau 2 giây; chưa có số đo liên tục hoặc mức trần bộ nhớ.

## Lỗi và vấn đề

Không có lỗi SUT nào được xác nhận trong bốn JTL cuối. Lỗi 403 ban đầu ở Add to Cart do biến JWT `${token}` chưa được trích xuất đúng trong kế hoạch JMeter; đã sửa cấu hình extractor và các JTL cuối không còn mẫu thất bại. Chưa tạo GitHub Issue vì chưa có lỗi sản phẩm được xác nhận.

## Đường dẫn và bằng chứng

- Báo cáo chính: [`reports/23120037_Performance_Test_Report_20261120.md`](reports/23120037_Performance_Test_Report_20261120.md)
- Phân tích Task 2: [`reports/23120037_AI_Analysis_Task2_20261120.md`](reports/23120037_AI_Analysis_Task2_20261120.md)
- Đề xuất Task 3: [`reports/23120037_Task3_Continuous_Performance_20261120.md`](reports/23120037_Task3_Continuous_Performance_20261120.md)
- Skill dùng lại: [gói Agent Skill](23120037_eshop-performance-testing.skill.zip); mã nguồn skill và script phân tích JTL nằm trong gói ZIP.
- Kiểm toán AI: [`audit/23120037_AI_Audit_Report_20261120.md`](audit/23120037_AI_Audit_Report_20261120.md)
- Phê bình AI 200–300 từ (người học đã sửa; AI đã rà độ dài và định dạng): [`audit/23120037_AI_Critique_20261120.md`](audit/23120037_AI_Critique_20261120.md)
- Thông số phần cứng: [`evidence/hardware_specs.md`](evidence/hardware_specs.md)
- Rà soát kế hoạch Task 1: [`reports/23120037_Task1_Review_Draft_20261120.md`](reports/23120037_Task1_Review_Draft_20261120.md)

Kho SUT công khai gốc: https://github.com/ttbhanh/eshop-sut . Kho cá nhân chứa bài nộp: https://github.com/ngthkhanhha/HW05-Performance-Testing-23120037 . Nhánh `main` đã được cập nhật; nhật ký commit nằm trong [`audit/23120037_Git_Log_20261120.md`](audit/23120037_Git_Log_20261120.md).

**Liên kết video demo YouTube không công khai:** Chưa có; để lại theo yêu cầu của người học.

## Bảng tự đánh giá

| Hạng mục | Điểm tối đa | Điểm tự đánh giá |
|---|---:|---|
| Task 1 — Load testing | 30 | 30 |
| Task 1 — Stress testing | 20 | 20 |
| Task 1 — Spike testing | 20 | 20 |
| Task 2 — Phân tích AI và tìm diễn giải sai | 10 | 10 |
| Task 3 — Đề xuất kiểm thử liên tục | 10 | 10 |
| Agent Skill | 10 | 10 |
| **Tổng** | **100** | **100** |

Bạn đã tự điền điểm cho từng mục; tổng điểm cộng lại là 100. Tên ZIP cuối cần dùng điểm ba chữ số `100` theo đề bài.

## Trạng thái bàn giao

1. Nội dung Task 1–3, báo cáo chính, bản rà soát của người học, kiểm toán AI và phê bình AI đã có ở định dạng Markdown. Bài phê bình AI do người học sửa; AI đã kiểm tra độ dài/định dạng.
2. Ảnh DxDiag và PowerShell đã lưu trong `evidence/`; hostname `DESKTOP-LIFKQ8N` khớp JMeter log. Bằng chứng hiện có xác nhận máy chạy kiểm thử; không khẳng định thay cho môi trường triển khai nếu môi trường đó khác.
3. Kết quả đã xác nhận ổn định đến 15 người dùng; **chưa xác định ngưỡng tối đa**. Không chạy thêm bài tăng tải theo lựa chọn của người học.
4. Các tệp bài nộp đã commit và push lên repo cá nhân; xem Git Log để biết hash commit.
5. Còn video demo YouTube không công khai dài tối thiểu 6 phút. Sau khi có video, người học sẽ tự cập nhật liên kết, xuất PDF từ Markdown và tạo ZIP theo mẫu `{MSSV}_HW05_AI_Performance_{điểm 3 chữ số}.zip`.
