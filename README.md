#  VictoriaLogs Learning Repo

> Tài liệu dành cho người mới bắt đầu tìm hiểu **VictoriaLogs (VicLogs)** — từ cơ bản đến thực hành. Tài liệu sẽ được chia nhỏ thành các file hướng dẫn từ cài đặt như thế nào tới sử dụng ra sao

---

## VictoriaLogs là gì?

**VictoriaLogs** là một hệ thống **log storage & query hiệu năng cao**, thuộc hệ sinh thái của **VictoriaMetrics**.

Nói đơn giản:
> Nó giúp bạn gom toàn bộ log từ nhiều service / server về một chỗ, rồi tìm kiếm và phân tích chúng một cách nhanh chóng.

## Nó khác gì so với hệ thống khác
VictoriaLogs tập trung vào:

- Hiệu năng cao (ingest & query nhanh)
- Dùng ít tài nguyên hơn (đặc biệt so với ELK)
- Triển khai đơn giản Chỉ cần chạy **một binary duy nhất** là có thể bắt đầu (không cần cluster phức tạp để bắt đầu) 


Bạn có thể bắt đầu test thử với single node với tài nguyên cơ bản
- CPU: **1 core**
- RAM: **512MB – 1GB**
- Disk: **tùy theo lượng log (SSD khuyến nghị)**

## Nó hoạt động như thế nào?

VictoriaLogs hoạt động theo flow:

Application / Service → Log Agent → VictoriaLogs → Query (LogsQL) → Grafana / UI

### 1. Application / Service
- Các ứng dụng (backend, frontend, database, container...) sinh ra log. Nó cũng có thể là log hệ thống như vcenter chẳng hạn
- Log có thể ở dạng text, JSON hoặc structured log

### 2. Log Agent
- victorialogs hỗ trợ nhận rất nhiều các log agent như **fluent-bit, vector, filebeat...** hoặc cũng có thể nhận trực tiếp các log từ syslog
- Khi đi qua các log agent có thể :
  - Đọc log từ file hoặc stdout
  - **Xử lý / biến đổi log trước khi gửi** ( giảm tải khi phải xử lý query). Trước khi log vào VictoriaLogs, agent có thể:
    - **Loại bỏ field không cần thiết** → giảm dung lượng lưu trữ và tăng tốc query
    - **Transform log** (parse JSON, rename field, format lại dữ liệu)
    - **Thêm metadata** (service name, environment, host, region…) 
    - **Filter log** (bỏ debug log, chỉ giữ error/warning nếu cần)
  - Gửi log đã được xử lý về VictoriaLogs qua HTTP

Điều này rất quan trọng vì:

> VictoriaLogs càng nhận ít dữ liệu “rác” thì càng query nhanh và tiết kiệm tài nguyên.

### 3. VictoriaLogs
VictoriaLogs đóng vai trò là **log storage + query engine** trong toàn bộ hệ thống.

- Nhận log từ các agent qua HTTP
- Ghi trực tiếp xuống disk (không phụ thuộc nhiều vào RAM)
- Dữ liệu được lưu dưới dạng **tối ưu hóa cho việc đọc/ghi nhanh**

VictoriaLogs được thiết kế để:

- Ghi log với tốc độ cao (high throughput)
- Xử lý tốt khi có nhiều service gửi log cùng lúc
- Không cần queue phức tạp như Kafka để bắt đầu

Tối ưu cho query

- Hỗ trợ truy vấn nhanh với **LogsQL**
- Có thể filter theo:
  - text
  - field (structured log)
  - thời gian
- Truy vấn trực tiếp trên dữ liệu đã lưu (không cần reprocess)

VictoriaLogs có thể được triển khai theo hai mô hình chính:

- **Single-node**: phù hợp cho môi trường dev, test hoặc hệ thống nhỏ
- **Cluster**: phù hợp cho production khi cần scale, tăng tính sẵn sàng và chịu tải lớn hơn

### 4. Query (LogsQL)

VictoriaLogs sử dụng **LogsQL** để truy vấn log.

Bạn có thể:

- Tìm kiếm theo text  
- Filter theo field 
- Kết hợp điều kiện  
  
LogsQL đơn giản, dễ học và được tối ưu để search log nhanh.

### 5. Visualization (Grafana / UI)

VictoriaLogs thường được sử dụng cùng **Grafana** để hiển thị log.

- Kết nối VictoriaLogs với Grafana qua datasource
- Query log bằng LogsQL trực tiếp trên dashboard
- Hỗ trợ:
- Xem log realtime  
- Search & filter  
- Alerting (khi kết hợp thêm)

---

👉 VictoriaLogs tập trung vào **lưu trữ & query**,  
còn Grafana dùng để **hiển thị & phân tích log**.