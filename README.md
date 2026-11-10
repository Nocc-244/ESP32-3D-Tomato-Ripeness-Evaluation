# SMART TOMATO RIPENESS & QUALITY EVALUATION SYSTEM
> **Hệ Thống Phân Loại Độ Chín Và Đánh Giá Chất Lượng Cà Chua Không Phá Hủy Trong Buồng Đo 3D Chuẩn Hóa**

---

## 1. Đặt Bài Toán (Problem Statement)

### Nhu Cầu Thực Tế Trong Nông Nghiệp & Công Nghiệp Thực Phẩm
Trong chuỗi cung ứng, xuất khẩu và công nghiệp chế biến nông sản (đặc biệt là các loại quả có màng mỏng như cà chua), việc xác định chính xác độ chín và phát hiện sớm các hiện tượng hư hỏng nội tạng đóng vai trò quyết định đến hiệu quả lưu kho và giá trị thương phẩm. Currently, hai phương pháp phân loại truyền thống tồn tại những nhược điểm lớn:
- **Phân loại thủ công bằng mắt và cảm quan:** Phụ thuộc hoàn toàn vào kinh nghiệm chủ quan của công nhân, năng suất thấp, sai số cao và thao tác cầm nắm liên tục dễ gây dập nát cơ học.
- **Phân loại bằng hóa nghiệm destructive (Đo độ Brix, hàm lượng Axit organic):** Cho kết quả chính xác cao nhưng buộc phải phá hỏng quả mẫu. Phương pháp này chỉ áp dụng xác suất cho một lượng mẫu rất nhỏ, không thể áp dụng cho 100% sản phẩm trên dây chuyền commercial.

### Nguồn Gốc Ý Tưởng & Đóng Góp Sáng Tạo (Inspiration & Innovation)
Được lấy cảm hứng từ các nghiên cứu tiên tiến về **Kiểm tra không phá hủy (Non-Destructive Testing - NDT)** trên thế giới (tham khảo từ các dự án nông nghiệp thông minh trên *Hackaday*), đề tài này đề xuất giải pháp ứng dụng công nghệ IoT và **Hợp nhất dữ liệu cảm biến (Data Fusion)** để đánh giá chất lượng quả ngay trong một **Buồng kín 3D chuẩn hóa thể tích (126 × 106 × 78 mm)**. 

Thay vì chỉ dùng camera quan sát bề mặt bên ngoài, hệ thống kết hợp 3 lớp dữ liệu quan trọng:
1. **Dữ liệu Quang học quang phổ (TCS34725):** Đo chính xác thông số màu sắc RGB/HSV của vỏ quả dưới nguồn sáng LED chuẩn hóa trong buồng kín, loại bỏ hoàn toàn nhiễu từ ánh sáng môi trường bên ngoài.
2. **Dữ liệu Khí sinh học (SGP30):** Gom tụ và phân tích nồng độ các hợp chất hữu cơ dễ bay hơi (TVOC) và nồng độ khí sinh học tương đương ($eCO_2$) thoát ra từ sự hô hấp của quả. Điều này giúp phát hiện sớm các hiện tượng **chín ẩn sâu bên trong** hoặc **úng thối nội tạng** khi vỏ ngoài chưa biến màu.
3. **Dữ liệu Môi trường & Bù nhiệt ẩm (BME280):** Giám sát liên tục nhiệt độ, độ ẩm tuyệt đối và áp suất trong buồng kín để chạy thuật toán tự động bù ẩm cho cảm biến khí SGP30, đảm bảo kết quả đo đạc đạt độ chính xác tối đa.

---

## 2. Nội Dung Chính Của Dự Án & Kiến Trúc Kỹ Thuật

Dự án được thiết kế và triển khai hoàn chỉnh theo mô hình IoT End-to-End với các khối chức năng cụ thể:

### A. Vi Điều Khiển & Nền Tảng Lập Trình
- **Vi điều khiển trung tâm:** ESP32 DevKit V1 (Dual-core 240MHz, tích hợp WiFi/BLE).
- **Môi trường phát triển:** VS Code tích hợp PlatformIO / Arduino IDE, viết mã bằng ngôn ngữ C/C++ tối ưu hiệu năng.

### B. Khối Thu Thập Dữ Liệu Đa Cảm Biến (Giao tiếp Bus I2C)
Tất cả các cảm biến cao cấp được kết nối chung trên một đường Bus I2C của ESP32 để tối ưu hóa số chân GPIO:
- **Cảm biến màu sắc TCS34725 (I2C Address: `0x29`):** Có bộ lọc cắt tia hồng ngoại (IR filter), tích hợp LED trợ sáng điều khiển được.
- **Cảm biến khí SGP30 (I2C Address: `0x58`):** Công nghệ MOXsens® đo TVOC ($0 - 60000 \text{ ppb}$) và $eCO_2$ ($400 - 60000 \text{ ppm}$) với tần số lấy mẫu 1Hz.
- **Cảm biến môi trường BME280 (I2C Address: `0x76`):** Đo chính xác nhiệt độ ($\pm 0.5^\circ\text{C}$), độ ẩm ($\pm 3\%$) và áp suất không khí.

### C. Khối Bộ Thực Thi & Phụ Kiện Phần Cứng (Actuators)
- **Đèn LED chiếu sáng chuẩn hóa:** Kích hoạt tự động khi bắt đầu chu kỳ lấy mẫu màu sắc.
- **Module Relay 5V + Quạt thông gió mini 5V:** Thực hiện chu kỳ **Xả khí (Purge Phase)** để hút sạch các khí tồn dư trong buồng kín ra ngoài trước và sau mỗi lần đo, đảm bảo kết quả đo độc lập giữa các quả.
- **Còi Buzzer 5V + Đèn LED RGB báo trạng thái tại chỗ:** Phát âm thanh cảnh báo và hiển thị màu sắc trực tiếp theo trạng thái quả (*Xanh / Ướm chín / Chín mọng / Úng thối*).

### D. Giao Thức Truyền Thông IoT & Hệ Thống Cloud
- **Giao thức:** Truyền nhận dữ liệu qua **MQTT** nhẹ, tốc độ cao với các chuỗi JSON được đóng gói tối ưu.
- **MQTT Broker Online:** Sử dụng EMQX Public Broker (`broker.emqx.io:1883`) hoặc HiveMQ Cloud (`broker.hivemq.com`).

### E. Giao Diện Giám Sát & Điều Kiểm Node-RED Dashboard
Xây dựng giao diện **Node-RED Dashboard 2.0** tương thích trên cả PC và Smartphone với 5 nhóm chức năng:
- **Hệ thống & Kết nối:** Trạng thái WiFi, chất lượng kết nối MQTT Broker và Nhật ký hệ thống (System Logs).
- **Phân tích màu sắc (Color Analytics):** Đồ thị RGB, thông số mã màu Hex và ô màu sắc hiển thị mô phỏng trực quan vỏ quả.
- **Giám sát Khí hậu & Khí thải:** Đồ thị diễn biến nồng độ TVOC, $eCO_2$, Nhiệt độ và Độ ẩm thời gian thực.
- **Kết quả Đánh giá Chất lượng:** Hiển thị kết luận độ chín và cảnh báo nguy cơ úng thối nội tạng.
- **Điều khiển từ xa (Remote Control):** Nút bấm bắt đầu chu kỳ đo thủ công, công tắc cưỡng bức bật/tắt quạt hút xả khí.

---

## 3. Sơ Đồ Khối Dự Kiến (Block Diagram)

Sơ đồ khối kiến trúc luồng dữ liệu và điều khiển của hệ thống:

```mermaid
graph TD
    subgraph SENSORS ["KHỐI CẢM BIẾN (Input - Bus I2C)"]
        A1["TCS34725<br/>(Đo Màu Sắc RGB/HSV)"]
        A2["SGP30<br/>(Đo Khí TVOC & eCO2)"]
        A3["BME280<br/>(Đo Nhiệt Độ / Độ Ẩm)"]
    end

    subgraph MCU ["KHỐI XỬ LÝ TRUNG TÂM"]
        B["ESP32 DevKit V1<br/>(PlatformIO / C++)"]
    end

    subgraph ACTUATORS ["KHỐI BỘ THỰC THI (Output)"]
        C1["Đèn LED chiếu sáng buồng đo"]
        C2["Module Relay 5V + Quạt hút xả 5V"]
        C3["Buzzer 5V + LED Báo Trạng Thái"]
    end

    subgraph IOT ["HỆ THỐNG TRUYỀN THÔNG & DASHBOARD"]
        D["MQTT Broker Online<br/>(EMQX / HiveMQ)"]
        E["Node-RED Dashboard 2.0<br/>(PC / Smartphone Control)"]
    end

    %% Kết nối cảm biến đến ESP32
    A1 -->|Bus I2C - SDA/SCL| B
    A2 -->|Bus I2C - SDA/SCL| B
    A3 -->|Bus I2C - SDA/SCL| B

    %% Kết nối ESP32 đến Bộ thực thi
    B -->|GPIO 5| C1
    B -->|GPIO 4| C2
    B -->|GPIO 18 / 19| C3

    %% Kết nối IoT
    B <===>|WiFi - MQTT Pub/Sub| D
    D <===>|MQTT Protocol| E
