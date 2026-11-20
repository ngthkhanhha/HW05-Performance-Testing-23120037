# Báo cáo kiểm thử hiệu năng EShop

**Mã số sinh viên:** 23120037  
**Ngày kiểm thử:** 20 tháng 11 năm 2026  
**Hệ thống được kiểm thử (SUT):** REST API EShop, Node.js/Express với SQLite  
**Công cụ tạo tải:** Apache JMeter 5.6.3

## 1. Tóm tắt điều hành

Đã phân tích bốn lần chạy kiểm thử hiệu năng cục bộ: Load, Stress, Spike và Endurance 10 phút. Các tệp JTL cuối cùng có tổng cộng 4.724 mẫu; tất cả mẫu được ghi nhận thành công. Mức đồng thời cao nhất được cấu hình là 15 người dùng ảo trong kịch bản Stress và Spike. Độ trễ cao nhất của một yêu cầu là 29 ms trong Load; p95 tổng thể là 8 ms ở cả bốn lần chạy.

Kết quả cho thấy cấu hình cục bộ đã xử lý các tải đã chọn mà không ghi nhận yêu cầu thất bại. Tuy nhiên, các kết quả không xác lập giới hạn năng lực của môi trường vận hành thực tế: bài Stress kết thúc ở 15 người dùng, bộ tạo tải chạy trên cùng máy với phần phụ trợ, và dữ liệu kiểm thử chỉ có một tài khoản. Bài Endurance chạy đủ 10 phút với các yêu cầu thành công và độ trễ tổng hợp thấp. Ảnh Task Manager được chụp gần thời điểm bắt đầu, nên không đo được mức sử dụng tài nguyên trong toàn bộ lần chạy.

## 2. Phạm vi và luồng nghiệp vụ

Kế hoạch JMeter thực hiện luồng mua hàng có xác thực theo thứ tự:

1. `POST /api/login` - đăng nhập tài khoản kiểm thử và trích xuất JWT.
2. `GET /api/products` - lấy danh sách sản phẩm.
3. `POST /api/cart` - thêm sản phẩm lấy từ CSV vào giỏ hàng bằng JWT.
4. `POST /api/checkout` - gửi yêu cầu thanh toán bằng JWT.

Tệp dữ liệu là `data/purchase_data.csv`, gồm một dòng tiêu đề và một dòng dữ liệu có email, mật khẩu, mã/tên sản phẩm, giá, số lượng và địa chỉ giao hàng. JSON Extractor lưu `$.token` vào biến JMeter `token`; các yêu cầu cần xác thực gửi `Authorization: Bearer ${token}`. Bật chế độ đọc dữ liệu CSV có dấu ngoặc kép để xử lý dấu phẩy trong địa chỉ giao hàng.

## 3. Môi trường

| Thành phần | Cấu hình quan sát được |
|---|---|
| CPU | Intel Core i7-12700H thế hệ 12; 20 bộ xử lý logic |
| Bộ nhớ | 24.576 MB RAM vật lý (24 GB) |
| Máy tính | ASUS TUF Gaming F15 FX507ZC4; hostname `DESKTOP-LIFKQ8N` |
| Hệ điều hành | Windows 11 Home Single Language 64-bit (10.0, Build 26200), theo ảnh DxDiag người học gửi |
| JMeter | Apache JMeter 5.6.3 |
| Java | OpenJDK Temurin 17.0.21 |
| Node.js | v24.11.1 |
| Backend | EShop API cục bộ tại `localhost:3000`; cơ sở dữ liệu SQLite |

API và bộ tạo tải chạy trên cùng một máy tính. Vì vậy, các con số là kết quả đo trong môi trường phát triển cục bộ, không phải ước lượng hiệu năng trên Internet hoặc môi trường vận hành thực tế.

## 4. Cấu hình tải

Mỗi sampler dùng Uniform Random Timer với độ trễ cố định 1.000 ms và độ trễ ngẫu nhiên tối đa 500 ms. Mỗi kịch bản ghi mẫu thô của JMeter vào một tệp JTL riêng. Các kế hoạch Load, Stress và Spike dùng các loại listener kết quả khác nhau theo yêu cầu.

| Kịch bản | Người dùng | Ramp-up | Thời lượng cấu hình | Listener JMeter | Mẫu | Lỗi | Thông lượng quan sát |
|---|---:|---:|---:|---|---:|---:|---:|
| Load | 5 | 30 giây | 120 giây | Summary Report | 424 | 0 (0%) | 3,58 yêu cầu/giây |
| Stress | 15 | 30 giây | 120 giây | Aggregate Report | 1.258 | 0 (0%) | 10,60 yêu cầu/giây |
| Spike | 15 | 1 giây | 60 giây | Graph Results | 706 | 0 (0%) | 12,00 yêu cầu/giây |
| Endurance | 5 | 30 giây | 600 giây | Summary Report | 2.336 | 0 (0%) | 3,90 yêu cầu/giây |

Thời lượng quan sát từ lúc mẫu đầu tiên bắt đầu đến khi mẫu cuối cùng hoàn tất lần lượt là 118,3 giây; 118,7 giây; 58,8 giây; và 598,5 giây. Chênh lệch so với thời lượng cấu hình là bình thường do JMeter khởi chạy/dừng luồng và thực hiện sampler trong cửa sổ thời gian đã đặt.

## 5. Kết quả tổng hợp

Độ trễ bên dưới được tính từ trường `elapsed` trong từng JTL. Phân vị dùng phương pháp nearest-rank. Thông lượng bằng số mẫu đã ghi nhận chia cho cửa sổ thời gian quan sát trong JTL.

| Kịch bản | Mẫu | Tỷ lệ lỗi | Độ trễ trung bình | p90 | p95 | p99 | Tối đa | Thông lượng |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Load | 424 | 0% | 3,00 ms | 7 ms | 8 ms | 9 ms | 29 ms | 3,58 yêu cầu/giây |
| Stress | 1.258 | 0% | 2,67 ms | 7 ms | 8 ms | 9 ms | 11 ms | 10,60 yêu cầu/giây |
| Spike | 706 | 0% | 2,47 ms | 6 ms | 8 ms | 9 ms | 11 ms | 12,00 yêu cầu/giây |
| Endurance | 2.336 | 0% | 2,72 ms | 7 ms | 8 ms | 9 ms | 21 ms | 3,90 yêu cầu/giây |

### Kết quả theo từng yêu cầu

| Kịch bản | Sampler | Mẫu | Trung bình | p95 | Tối đa | Lỗi |
|---|---|---:|---:|---:|---:|---:|
| Load | Login | 107 | 2,03 ms | 3 ms | 4 ms | 0 |
| Load | Product List | 107 | 1,62 ms | 2 ms | 3 ms | 0 |
| Load | Add to Cart | 105 | 1,43 ms | 2 ms | 3 ms | 0 |
| Load | Checkout | 105 | 6,96 ms | 9 ms | 29 ms | 0 |
| Stress | Login | 319 | 1,70 ms | 3 ms | 6 ms | 0 |
| Stress | Product List | 316 | 1,28 ms | 2 ms | 7 ms | 0 |
| Stress | Add to Cart | 313 | 1,04 ms | 2 ms | 3 ms | 0 |
| Stress | Checkout | 310 | 6,75 ms | 9 ms | 11 ms | 0 |
| Spike | Login | 181 | 1,68 ms | 3 ms | 9 ms | 0 |
| Spike | Product List | 181 | 1,31 ms | 2 ms | 7 ms | 0 |
| Spike | Add to Cart | 175 | 0,95 ms | 2 ms | 3 ms | 0 |
| Spike | Checkout | 169 | 6,14 ms | 9 ms | 11 ms | 0 |
| Endurance | Login | 586 | 1,64 ms | 2 ms | 5 ms | 0 |
| Endurance | Product List | 585 | 1,26 ms | 2 ms | 8 ms | 0 |
| Endurance | Add to Cart | 583 | 0,99 ms | 2 ms | 3 ms | 0 |
| Endurance | Checkout | 582 | 7,01 ms | 9 ms | 21 ms | 0 |

Tất cả phản hồi được ghi nhận thành công và có mã HTTP 200. Checkout luôn có độ trễ trung bình cao hơn ba yêu cầu còn lại, với mức trung bình 6,14–7,01 ms giữa các lần chạy. Phản hồi đơn lẻ cao nhất là mẫu Checkout 29 ms trong Load; mức cao này không lặp lại ở p95 của các kịch bản khác.

## 6. Quan sát từ công cụ giám sát tài nguyên

Ảnh Task Manager đã lưu cho thấy các số liệu toàn hệ thống tại từng thời điểm:

| Ảnh chụp | Trạng thái lần chạy nhìn thấy trong ảnh | CPU | Bộ nhớ |
|---|---|---:|---:|
| `evidence/load_run_01.png` | Load result view | 7% | 56% |
| `evidence/stress_run_01.png` | Stress run in progress, 7/15 users active | 2% | 55% |
| `evidence/spike_run_01.png` | Spike run in progress, 15/15 users active | 4% | 56% |
| `evidence/endurance_monitor_start.png` | Endurance startup, 2 s elapsed, 1/5 users active | 7% | 57% |
| `evidence/endurance_run_01.png` | Summary Report Endurance sau khi hoàn tất | Không hiển thị | Không hiển thị |

Đây là ảnh chụp tại một thời điểm, không phải số trung bình hoặc mức đỉnh. Ảnh Stress và Spike được chụp khi bài kiểm thử còn chạy, không phải sau khi đã có kết quả cuối cùng. Ảnh tài nguyên Endurance được chụp chỉ 2 giây sau khi bắt đầu ramp-up, khi mới có 1 trong 5 người dùng hoạt động. Ảnh chứng minh Task Manager hiển thị lúc bắt đầu, nhưng không cho biết mức sử dụng tài nguyên dưới tải ổn định 5 người dùng hoặc trong toàn bộ 10 phút.

## 7. Diễn giải

- Cả bốn lần chạy có 0 mẫu JTL thất bại và chỉ có phản hồi HTTP 200. Điều này cho thấy các yêu cầu HTTP của tải đã đo hoàn tất; các kế hoạch chưa kiểm tra nội dung phản hồi hoặc xác minh dữ liệu đơn hàng được tạo.
- Thông lượng tăng từ 3,58 yêu cầu/giây ở 5 người dùng lên 10,60 yêu cầu/giây ở bài Stress 15 người dùng và 12,00 yêu cầu/giây ở bài Spike 15 người dùng. p95 tổng thể vẫn là 8 ms trong mọi kịch bản.
- Bài Stress không tìm ra điểm phá vỡ vì kết thúc ở 15 người dùng mà không có lỗi. Năng lực vượt quá 15 người dùng chưa được đo.
- Bài Endurance hoàn thành thời lượng cấu hình 600 giây với 2.336 mẫu thành công. Độ trễ tổng hợp thấp, nhưng ảnh Task Manager đầu bài và Summary Report cuối bài không thể loại trừ tăng tài nguyên nhất thời hoặc xu hướng độ trễ tăng dần.
- Tất cả người dùng ảo dùng chung một tài khoản trong CSV. Cách này thuận tiện cho luồng kiểm soát, nhưng có thể gây tranh chấp giỏ hàng chung và không đại diện cho nhiều khách hàng độc lập.
- JMeter, API và cơ sở dữ liệu dùng chung một máy cục bộ. Số liệu CPU và bộ nhớ có tính cả các ứng dụng khác và không thể tách riêng mức tiêu thụ tài nguyên của API.

## 8. Kết luận và bước tiếp theo

Trong phạm vi cục bộ đã đo, luồng mua hàng EShop hoàn thành các bài Load, Stress, Spike và Endurance 10 phút mà không ghi nhận yêu cầu thất bại. p95 tổng thể quan sát được không quá 8 ms, và yêu cầu đơn lẻ cao nhất là 29 ms. Endurance ghi nhận một điểm tải duy trì 5 người dùng, 3,90 yêu cầu/giây trong khoảng 598,5 giây. Mức cao nhất đã kiểm thử thành công là 15 người dùng ở Stress, với 10,60 yêu cầu/giây, p95 8 ms và 0% lỗi. Tôi chọn không chạy thêm bài tăng tải để tiết kiệm thời gian; vì vậy báo cáo kết luận đã xác nhận ổn định đến 15 người dùng, nhưng chưa xác định ngưỡng tối đa hay điểm bão hòa.

Để xác định điểm bão hòa trong một đợt kiểm thử tiếp theo, có thể tăng người dùng đồng thời theo từng bước, dùng tài khoản riêng hoặc cô lập trạng thái giỏ hàng, và ghi liên tục CPU/bộ nhớ. Đợt này dừng ở mức đã đo nhằm tiết kiệm thời gian; không diễn giải 15 người dùng thành giới hạn phần cứng tối đa. Sau khi assertion nghiệp vụ và baseline CI được hiệu chuẩn, đề xuất chạy smoke test trong pull request, Load ngắn khi commit lên nhánh chính, và Load/Stress/Spike/Endurance theo lịch nightly hoặc trước phát hành. Pipeline so sánh p95 cùng cấu hình và xác nhận lại trước khi gắn cờ hồi quy; sơ đồ, chính sách và chi phí/báo động giả nằm ở [đề xuất Task 3](23120037_Task3_Continuous_Performance_20261120.md). Đây là mô hình đề xuất, chưa triển khai trong repository. Trong lượt rà soát cuối, tên hiển thị Thread Group của JMX Endurance được sửa từ “Load” thành “Endurance”; thay đổi chỉ là nhãn nên không làm thay đổi kết quả tải đã đo. Giữ video demo thành hạng mục chờ riêng theo yêu cầu của bạn.

## 9. Tệp bàn giao

| Loại | Đường dẫn |
|---|---|
| Plans | `plans/23120037_Load_20261120.jmx`, `plans/23120037_Stress_20261120.jmx`, `plans/23120037_Spike_20261120.jmx`, `plans/23120037_Endurance_20261120.jmx` |
| Test data | `data/purchase_data.csv` |
| Kết quả thô | `results/load/load_run_01.jtl`, `results/stress/stress_run_01.jtl`, `results/spike/spike_run_01.jtl`, `results/endurance/endurance_run_01.jtl` |
| Bảng điều khiển HTML | `results/load/load_run_01_html/index.html`, `results/stress/stress_run_01_html/index.html`, `results/spike/spike_run_01_html/index.html`, `results/endurance/endurance_run_01_html/index.html` |
| Ảnh chụp | `evidence/load_run_01.png`, `evidence/stress_run_01.png`, `evidence/spike_run_01.png`, `evidence/endurance_monitor_start.png`, `evidence/endurance_run_01.png` |
| Thông số phần cứng | `evidence/hardware_specs.md`; `evidence/hardware_dxdiag_system_20261120.png`; `evidence/hardware_hostname_20261120.png` |
| Rà soát thiết kế Task 1 | `reports/23120037_Task1_Review_Draft_20261120.md` (người học đã đọc và xác nhận; AI hỗ trợ đối chiếu/diễn đạt) |
| Gói Agent Skill | `submission/23120037_eshop-performance-testing.skill.zip` |

**Giới hạn bằng chứng:** Mức sử dụng tài nguyên trong Endurance chỉ được ghi nhận bằng ảnh chụp lúc bắt đầu, ở giây thứ 2 và khi 1/5 người dùng hoạt động. Không có xu hướng hoặc mức đỉnh tài nguyên trong toàn bộ thời gian chạy; báo cáo không ước đoán các giá trị còn thiếu.

## 10. Nhiệm vụ 2 — Phân tích AI và rà soát của người học

Phân tích AI sử dụng cả bốn JTL thô cuối cùng và đề xuất các ngưỡng hồi quy tạm thời: tỷ lệ lỗi ≤1%, p95 tổng thể ≤16 ms và p95 Checkout ≤18 ms. Đây là ngưỡng so sánh các lần chạy lặp lại trên cùng máy, rút ra từ mốc tham chiếu này; không phải mục tiêu mức dịch vụ cho môi trường vận hành thực tế hay ngưỡng tối đa phần cứng. AI cũng đề xuất cân nhắc chỉ mục SQLite và thử nghiệm WAL khi có điều kiện; đồng thời bác bỏ bể kết nối chung chung và chỉ mục B-tree thường cho truy vấn tìm kiếm có ký tự đại diện ở đầu hiện tại.

Bản đối chiếu AI kiểm tra số mẫu, cờ thành công, mã HTTP, p95 và giá trị tối đa trong JTL, không thấy sai số tính toán. Tôi đã đọc và xác nhận các nhận xét này phù hợp với bằng chứng. Tôi giới hạn cách hiểu từ “thành công”: JTL cho thấy HTTP 200 và không có lỗi JMeter được ghi nhận, nhưng kế hoạch chưa xác nhận nội dung phản hồi hoặc dữ liệu đơn hàng đúng về nghiệp vụ. Bài kiểm thử 15 người dùng chưa tìm được điểm bão hòa; kết quả Spike 12 yêu cầu/giây không phải giới hạn năng lực. Chưa thể đặt ngưỡng tài nguyên vì ảnh Endurance chỉ chụp lúc bắt đầu. AI hỗ trợ phân tích và diễn đạt bản rà soát.

Xem [phân tích AI](23120037_AI_Analysis_Task2_20261120.md), [rà soát Task 2 của tôi](23120037_Task2_Human_Review_20261120.md) và [rà soát kế hoạch Task 1 của tôi](23120037_Task1_Review_Draft_20261120.md) để xem số liệu, giới hạn và đánh giá tính khả thi. Tôi đã đọc và xác nhận nội dung hai bản rà soát; AI hỗ trợ đối chiếu và diễn đạt.

## 11. Nhiệm vụ 3 — Mô hình kiểm thử hiệu năng liên tục

Đề xuất pipeline theo dõi commit có thay đổi backend, kế hoạch JMeter, dữ liệu kiểm thử hoặc cấu hình CI. Pull request chạy health check và smoke test ngắn; nhánh chính chạy Load ngắn; nightly và trước phát hành chạy bộ Load/Stress/Spike cùng Endurance. Pipeline so sánh p95 với mốc riêng của cùng runner, cấu hình và dữ liệu; chạy xác nhận tối đa hai lần khi vượt ngưỡng và chỉ báo hồi quy khi ít nhất 2/3 lượt vượt cùng ngưỡng. Báo cáo JTL, HTML và SHA commit được lưu làm bằng chứng.

Đây là đề xuất, chưa có workflow CI được triển khai. Repository hiện thiếu bộ test tự động dùng được trong pipeline; script khởi động server có đường dẫn tuyệt đối và lệnh dừng tiến trình rộng; cơ sở dữ liệu bị tạo lại khi khởi động. Trước khi bật cổng chặn merge cần runner và DB dùng một lần, health check/assertion nghiệp vụ, baseline riêng cho CI và kiểm soát trạng thái tài khoản/giỏ hàng. Các ngưỡng Task 2 chỉ là điểm khởi đầu cục bộ, không áp trực tiếp lên runner khác. Chi phí runner, nhiễu tài nguyên, baseline sai và tải thử chưa đại diện được phân tích trong tài liệu riêng.

Xem [đề xuất Task 3](23120037_Task3_Continuous_Performance_20261120.md) để xem sơ đồ pipeline, chính sách kích hoạt, ngưỡng p95, quy tắc xác nhận và phân tích chi phí/báo động giả.

## 12. Phê bình AI

Phần dưới đây là bài phê bình do tôi chỉnh sửa sau khi xem lại quá trình làm bài. AI hỗ trợ kiểm tra số từ và định dạng; nội dung phản ánh đánh giá của tôi.

Tôi thấy AI hữu ích khi hướng dẫn cấu hình JMeter từng bước, giải thích luồng mua hàng và tính số liệu JTL. Đối chiếu log sau khi chạy giúp tránh kết luận vội từ ảnh chụp giữa bài Stress và Spike. Tuy nhiên, tôi không nên tin xác nhận cấu hình nếu chưa kiểm tra JMX. Lần Add to Cart đầu gửi nguyên `${token}` và nhận 403 vì JSON Extractor chưa tạo đúng biến. AI đã bỏ sót trường này khi hướng dẫn header Authorization. Bản Endurance cũ cũng chưa đáp ứng mục tiêu 5 người dùng trong 600 giây; sai lệch chỉ lộ ra khi xem lại màn hình và tệp.

AI còn đánh giá tiến độ quá sớm trước khi kiểm kê đủ các tệp bàn giao. Ảnh Task Manager hữu ích để biết trạng thái tại một thời điểm, nhưng ảnh Endurance chụp ở giây thứ hai không chứng minh tài nguyên ổn định suốt mười phút. JTL ghi HTTP 200 và `success=true`, song kế hoạch chưa có Response Assertion xác minh token, giỏ hàng hay đơn hàng. Vì vậy, tôi dùng AI để gợi ý và tính toán, rồi đối chiếu với JMX, JTL và giới hạn của bằng chứng. Tôi không kết luận lỗi SUT, mức tải tối đa hoặc xu hướng tài nguyên nếu thiếu phép đo. Bài học của tôi là tự kiểm chứng dữ liệu, sửa hướng dẫn sai và chịu trách nhiệm về nhận định cuối cùng.
