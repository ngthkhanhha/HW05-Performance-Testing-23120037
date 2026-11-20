# Thông số phần cứng và môi trường kiểm thử

**Mã số sinh viên:** 23120037  
**Thời điểm chạy JMeter:** 20 tháng 11 năm 2026  
**Nguồn:** log khởi động JMeter tại `C:\HW5\jmeter.log` và thông tin đã ghi trong báo cáo hiệu năng.

| Thuộc tính | Giá trị đã ghi nhận | Nguồn/giới hạn |
|---|---|---|
| Hostname | `DESKTOP-LIFKQ8N` | Đã đối chiếu: ảnh DxDiag, ảnh PowerShell chạy `hostname` và JMeter log đều khớp |
| Hệ điều hành | Windows 11 Home Single Language 64-bit (10.0, Build 26200) | Ảnh `hardware_dxdiag_system_20261120.png` trong thư mục này |
| Nhà sản xuất/mẫu máy | ASUSTeK COMPUTER INC.; ASUS TUF Gaming F15 FX507ZC4 | Ảnh DxDiag tab System |
| CPU | Intel Core i7-12700H thế hệ 12 | Ảnh DxDiag tab System |
| Bộ xử lý logic | 20 | Ảnh DxDiag tab System và JMeter log |
| RAM vật lý | 24.576 MB (24 GB) | Ảnh DxDiag tab System |
| JMeter | Apache JMeter 5.6.3 | JMeter log; hostname `DESKTOP-LIFKQ8N` khớp với ảnh DxDiag |
| Java | OpenJDK 64-bit, phiên bản 17.0.21 | JMeter log |
| Backend | Node.js/Express, SQLite, `localhost:3000` | Cấu hình dự án/kế hoạch JMeter |

## Giới hạn bằng chứng

Ảnh `hardware_dxdiag_system_20261120.png` và `hardware_hostname_20261120.png` đã được lưu trong thư mục này. Ba nguồn (DxDiag, PowerShell và JMeter log) cùng cho hostname `DESKTOP-LIFKQ8N`. Ảnh PowerShell xác nhận hostname hiện tại; nếu lần triển khai trước chạy trên máy/VM khác thì cần bằng chứng hostname từ môi trường đó.

Các ảnh Task Manager hiện có ghi CPU/bộ nhớ toàn hệ thống tại một thời điểm và danh sách tiến trình Node.js; chúng không xác định chắc PID backend nào và không ghi chuỗi thời gian tài nguyên.

Mốc Endurance hiện có là 5 người dùng trong khoảng 598,5 giây với 2.336 mẫu, 0 lỗi JTL và 3,90 yêu cầu/giây. Bài Stress đạt 15 người dùng, 10,60 yêu cầu/giây, p95 8 ms và 0% lỗi. Người học chọn không chạy thêm tải tăng dần; mức đã xác nhận ổn định cao nhất là 15 người dùng, ngưỡng tối đa chưa xác định. Ảnh Task Manager Endurance được chụp sau 2 giây, khi 1/5 người dùng hoạt động; CPU 7%, bộ nhớ 57%. Không suy ra mức trần CPU/RAM từ ảnh này.
