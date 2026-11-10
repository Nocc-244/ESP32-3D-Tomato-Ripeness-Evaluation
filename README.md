# ĐỀ TÀI KẾT THÚC MÔN HỌC: HỆ THỐNG ĐÁNH GIÁ ĐỘ TƯƠI VÀ ĐỘ CHÍN CỦA CÀ CHUA TRONG BUỒNG KÍN 3D (SMART TOMATO RIPENESS & QUALITY EVALUATION SYSTEM)

## 1. Đặt bài toán (Nhu cầu thực tế & Tính cấp thiết)

Trong ngành công nghiệp chế biến và xuất khẩu nông sản, phân loại độ chín của cà chua là khâu quyết định đến thời gian lưu kho và chất lượng thành phẩm. Hiện nay, hai phương pháp truyền thống tồn tại nhiều bất cập:
- **Phân loại thủ công bằng mắt/tay:** Mang tính chủ quan, sai số cao và dễ gây dập nát nông sản (phá hủy cơ học).
- **Phân loại bằng hóa nghiệm (Đo độ Brix/Acid):** Cho kết quả chính xác nhưng phải phá hỏng quả mẫu, không thể áp dụng trên quy mô lớn.

**Giải pháp đề xuất:**
Hệ thống hướng tới phân khúc trạm kiểm định chất lượng vừa và nhỏ, sử dụng **Buồng đo 3D cố định thể tích (126×106×78 mm)** tích hợp vi điều khiển ESP32 và công nghệ IoT để phân loại **Không phá hủy (Non-destructive testing)** dựa trên kỹ thuật hợp nhất dữ liệu (Data Fusion):
1. **Chuẩn hóa quang học:** Buồng kín cách ly ánh sáng môi trường bên ngoài, sử dụng LED chuẩn chiếu sáng giúp cảm biến màu sắc TCS34725 nhận diện chính xác mã màu vỏ (RGB/HSV).
2. **Gom tụ khí sinh học:** Thể tích buồng cố định giúp tích tụ nồng độ hợp chất hữu cơ dễ bay hơi (TVOC/eCO2) phát ra từ quả, giúp cảm biến khí SGP30 phát hiện sớm hiện tượng chín ẩn hoặc úng thối nội tạng mà mắt thường chưa nhìn thấy.
3. **Bù nhiệt độ & độ ẩm:** Cảm biến BME280 đo môi trường buồng kín để tự động bù ẩm cho thuật toán đọc SGP30, đảm bảo độ chính xác cao.
4. **Giám sát & Tự động hóa IoT:** Truyền dữ liệu thời gian thực qua giao thức MQTT lên Node-RED Dashboard, giúp trung tâm kiểm định theo dõi và điều khiển chu kỳ hút/xả khí buồng đo từ xa.

## 2. Kiến trúc Hệ thống & Sơ đồ khối (Block Diagram)

### 2.1. Sơ đồ luồng dữ liệu:
[Cảm biến TCS34725, SGP30, BME280] ---> [ESP32 DevKit] <--- (Giao thức MQTT) ---> [MQTT Broker Online] <--- (Giao thức MQTT) ---> [Node-RED Dashboard & Báo cáo Quality]

### 2.2. Sơ đồ khối chi tiết:
- **Khối Vi điều khiển Trung tâm:** ESP32 Dev Board (Lập trình trên VS Code + PlatformIO / C++ Arduino Framework).
- **Khối Cảm biến (Input - Chuẩn bus I2C):**
  - Cảm biến màu sắc TCS34725 (Địa chỉ I2C: `0x29`): Nhận diện độ chín qua sắc tố vỏ RGB/HSV.
  - Cảm biến khí SGP30 (Địa chỉ I2C: `0x58`): Đo nồng độ hợp chất hữu cơ dễ bay hơi (TVOC) và eCO2.
  - Cảm biến nhiệt độ - độ ẩm - áp suất BME280 (Địa chỉ I2C: `0x76` / `0x77`): Giám sát môi trường buồng kín và hỗ trợ hiệu chỉnh độ ẩm tuyệt đối cho SGP30.
- **Khối Thực thi (Output):**
  - Đèn LED chiếu sáng chủ động (nối nguồn 3.3V / GPIO điều khiển): Cung cấp môi trường ánh sáng cố định trong buồng 3D khi đo màu.
  - Quạt hút/thông gió mini 5V (qua Module Relay/Transistor): Bật hút xả sạch khí tồn dư trong buồng kín sau mỗi chu kỳ lấy mẫu (Purge phase).
  - Còi Buzzer & Đèn LED báo trạng thái phân loại tại chỗ (Xanh: Tươi/Đạt - Đỏ: Quá chín/Úng hỏng).
- **Khối Bus I2C & Nguồn/Lọc nhiễu:**
  - Điện trở Pull-up 4.7kΩ kéo SDA, SCL lên +3.3V.
  - Tụ lọc nhiễu 100nF nối song song sát chân nguồn từng cảm biến.
- **Khối Giám sát & Điều khiển (IoT):**
  - Giao thức truyền thông: MQTT qua Public Broker (`broker.hivemq.com` / `broker.emqx.io`).
  - Giao diện người dùng: Node-RED Dashboard hiển thị màu sắc quả, nồng độ khí TVOC/eCO2, đồ thị theo thời gian thực và nút bấm kích hoạt chu kỳ đo từ xa.

## 3. Chức năng chính của dự án
1. **Thu thập chỉ số môi trường kín:** Thu thập dữ liệu màu sắc vỏ (RGB) và nồng độ khí VOCs sinh ra từ quả cà chua đặt trong buồng kín 3D (kích thước 126 × 106 × 78 mm).
2. **Thuật toán Phân loại hợp nhất (Data Fusion):** Xử lý dữ liệu từ cả màu sắc và nồng độ khí để đưa ra đánh giá chính xác trạng thái quả: *Xanh (Unripe) -> Ướm chín (Breaker) -> Chín mọng (Ripe) -> Úng thối (Spoiled)*.
3. **Giám sát & Điều khiển IoT từ xa:** 
   - Trực quan hóa dữ liệu đo đạc trên Node-RED Dashboard.
   - Cho phép gửi lệnh MQTT từ Node-RED để kích hoạt chu kỳ đo mới (Bật LED -> Đọc cảm biến -> Bật quạt xả khí).

## 4. Dự kiến Phần cứng & Thiết kế PCB (KiCad)
- **Thiết kế bo mạch:** Sơ đồ nguyên lý (Schematic) và PCB Layout thực hiện trên KiCad.
- **Phần cứng thật:** ESP32 DevKit V1, Cảm biến TCS34725, Cảm biến SGP30, Cảm biến BME280, Quạt mini 5V, Module Relay 1 kênh, Buzzer 5V, LED, Điện trở 4.7kΩ, Tụ 100nF, Buồng kín 3D.
- **Mô phỏng & Đột phá:** Lập trình PlatformIO (VS Code), mô phỏng debug giao thức trên Wokwi.
- **Server:** Node-RED Dashboard, MQTT Broker public.

## 5. Tài liệu tham khảo
1. Adafruit TCS34725 Color Sensor Guide: https://learn.adafruit.com/adafruit-color-sensors
2. Sensirion SGP30 Gas Sensor Datasheet: https://sensirion.com/products/catalog/SGP30/
3. Bosch BME280 Datasheet: https://www.bosch-sensortec.com/products/environmental-sensors/humidity-sensors-bme280/
4. ESP32 MQTT with Node-RED Tutorial: https://randomnerdtutorials.com/esp32-mqtt-publish-subscribe-arduino-ide/
