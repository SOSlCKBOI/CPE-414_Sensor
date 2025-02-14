1. การตั้งค่าพื้นฐานและการเชื่อมต่อ Wi-Fi🤞💕:

WiFi.begin(ssid, password);  // เชื่อมต่อกับ Wi-Fi
while (WiFi.status() != WL_CONNECTED) {  // รอจนกว่า Wi-Fi จะเชื่อมต่อสำเร็จ
  delay(500);
  Serial.print(".");
}
Serial.println("\nWiFi connected");  // แสดงข้อความเมื่อเชื่อมต่อ Wi-Fi สำเร็จ

- โค้ดนี้เริ่มการเชื่อมต่อกับเครือข่าย Wi-Fi โดยใช้ข้อมูล ssid (ชื่อเครือข่าย Wi-Fi) และ password (รหัสผ่าน Wi-Fi) ที่ได้ถูกตั้งค่าไว้ในโค้ด
- หลังจากนั้นจะตรวจสอบสถานะการเชื่อมต่อ Wi-Fi โดยใช้ WiFi.status(). ถ้ายังไม่เชื่อมต่อจะพิมพ์จุด (.) ไปเรื่อยๆ จนกว่าจะเชื่อมต่อสำเร็จ จากนั้นจะแสดงข้อความใน Serial Monitor ว่าเชื่อมต่อ Wi-Fi สำเร็จแล้ว

2. การตั้งค่ากล้อง (Camera) ด้วย ESP32-CAM😘:
camera_config_t config;  // กำหนดค่ากล้อง
config.ledc_channel = LEDC_CHANNEL_0;  // ช่อง LEDC
config.ledc_timer = LEDC_TIMER_0;  // ตั้งเวลา LEDC
...
if (esp_camera_init(&config) != ESP_OK) {  // ถ้าการเริ่มต้นกล้องล้มเหลว
  Serial.println("Camera initialization failed!");  // แสดงข้อความข้อผิดพลาด
  return;
}

-การตั้งค่ากล้องใน ESP32 เริ่มจากการกำหนด camera_config_t config ซึ่งจะมีการตั้งค่าหลายๆ ค่า เช่น การเลือกช่องสัญญาณ LEDC, การกำหนดพอร์ตต่างๆ สำหรับการเชื่อมต่อกับพินของกล้อง (เช่น pin_d0, pin_d1, pin_d2, ฯลฯ)
-ถ้าการตั้งค่ากล้องสำเร็จ esp_camera_init(&config) จะส่งค่าผลลัพธ์เป็น ESP_OK, หากไม่สำเร็จจะแสดงข้อความข้อผิดพลาดใน Serial Monitor

3. การตั้งค่าเซิร์ฟเวอร์ HTTP😎: 
httpd_uri_t index_uri = {
  .uri       = "/",
  .method    = HTTP_GET,
  .handler   = index_handler,
  .user_ctx  = NULL
};
...
if (httpd_start(&camera_httpd, &config) == ESP_OK) {
  httpd_register_uri_handler(camera_httpd, &index_uri);
  httpd_register_uri_handler(camera_httpd, &cmd_uri);
}

- ส่วนนี้จะทำการตั้งค่าเซิร์ฟเวอร์ HTTP เพื่อให้ ESP32 สามารถให้บริการเว็บเพจที่สามารถเข้าถึงได้ผ่าน IP ของ ESP32
- httpd_uri_t index_uri เป็นการตั้งค่าการเชื่อมต่อเว็บในรูปแบบ HTTP GET ที่พาไปที่ URI (/) และจะทำการเรียกฟังก์ชัน index_handler สำหรับแสดงหน้าแรก
- นอกจากนี้ยังมีการลงทะเบียน URI อื่นๆ เช่น /cmd และ /stream สำหรับคำสั่งและสตรีมภาพจากกล้อง

4. การตั้งค่าเซ็นเซอร์ AMG8833 🎁:
Wire.begin();  // เริ่มต้นการเชื่อมต่อ I2C
if (!amg.begin()) {  // ถ้าไม่สามารถเชื่อมต่อกับเซนเซอร์ AMG8833
  Serial.println("Failed to initialize AMG8833!");  // แสดงข้อความข้อผิดพลาด
  while (1);  // หยุดโปรแกรม
}
Serial.println("AMG8833 Initialized!");  // แสดงข้อความว่าเซนเซอร์ AMG8833 ได้รับการตั้งค่าแล้ว

- การเริ่มต้นการเชื่อมต่อกับเซ็นเซอร์ AMG8833 ผ่าน I2C (Wire.begin())
- หากการเชื่อมต่อเซ็นเซอร์ไม่สำเร็จจะแสดงข้อความผิดพลาดและหยุดโปรแกรม

5. การอ่านค่าอุณหภูมิจากเซ็นเซอร์ AMG8833💎:
amg.readPixels(pixels);  // อ่านค่าจากเซนเซอร์
float maxTemp = -100, minTemp = 100;  // กำหนดอุณหภูมิสูงสุดและต่ำสุดเริ่มต้น
for (int i = 0; i < 64; i++) {  // ลูปอ่านค่าจากแต่ละพิกเซล
  if (pixels[i] > maxTemp) maxTemp = pixels[i];  // อัปเดตอุณหภูมิสูงสุด
  if (pixels[i] < minTemp) minTemp = pixels[i];  // อัปเดตอุณหภูมิต่ำสุด
}

- ฟังก์ชัน amg.readPixels(pixels) จะอ่านค่าจากเซ็นเซอร์ AMG8833 และเก็บค่าอุณหภูมิจากพิกเซลทั้ง 64 พิกเซลไว้ในอาร์เรย์ pixels
- จากนั้นจะทำการค้นหาอุณหภูมิสูงสุดและต่ำสุดจากค่าที่ได้

6. การตรวจสอบอุณหภูมิและการแจ้งเตือน 🎄:
if (maxTemp > TEMP_HIGH_THRESHOLD || minTemp < TEMP_LOW_THRESHOLD) {  // ถ้าอุณหภูมิสูงหรือต่ำกว่าค่าที่กำหนด
  digitalWrite(BUZZER_PIN, HIGH);  // เปิด Buzzer
  Serial.println("Temperature Alert! Buzzer Activated!");  // แสดงข้อความแจ้งเตือน
} else {
  digitalWrite(BUZZER_PIN, LOW);  // ปิด Buzzer
}

- หากอุณหภูมิสูงสุดหรืออุณหภูมิต่ำสุดที่อ่านได้เกินขีดจำกัดที่กำหนดไว้ (TEMP_HIGH_THRESHOLD และ TEMP_LOW_THRESHOLD) จะทำการเปิด Buzzer เพื่อแจ้งเตือน
- ถ้าอุณหภูมิอยู่ในช่วงที่ปลอดภัยก็จะปิด Buzzer

7. การควบคุมมอเตอร์ 🍭:
pinMode(MOTOR_1_PIN_1, OUTPUT);  // ตั้งค่าพินมอเตอร์ 1 ขั้ว 1
pinMode(MOTOR_1_PIN_2, OUTPUT);  // ตั้งค่าพินมอเตอร์ 1 ขั้ว 2
pinMode(MOTOR_2_PIN_1, OUTPUT);  // ตั้งค่าพินมอเตอร์ 2 ขั้ว 1
pinMode(MOTOR_2_PIN_2, OUTPUT);  // ตั้งค่าพินมอเตอร์ 2 ขั้ว 2

- กำหนดพินที่เชื่อมต่อกับมอเตอร์ให้เป็น OUTPUT เพื่อให้สามารถควบคุมการหมุนของมอเตอร์ได้

8. การหน่วงเวลาและการทำงานซ้ำ🥨:
delay(1000);  // หน่วงเวลา 1 วินาที

- หน่วงเวลา 1 วินาทีเพื่อให้ระบบไม่ทำงานเร็วเกินไป ทำให้สามารถตรวจสอบอุณหภูมิและทำการแจ้งเตือนได้อย่างมีประสิทธิภาพ

สรุป:
**ใช้ ESP32 ในการเชื่อมต่อ Wi-Fi เพื่อให้สามารถเข้าถึงจากระยะไกล
**เชื่อมต่อกับเซ็นเซอร์ AMG8833 เพื่ออ่านอุณหภูมิและตรวจสอบค่าพิกเซลอุณหภูมิ
**ส่งข้อมูลกล้องและการควบคุมมอเตอร์ผ่านเซิร์ฟเวอร์ HTTP
**แจ้งเตือนเมื่ออุณหภูมิสูงหรือต่ำกว่าขีดจำกัดด้วย Buzzer
