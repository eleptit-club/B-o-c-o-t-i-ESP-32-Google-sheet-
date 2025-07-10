# ELE-D23-DO-HUY-HOANG- BÁO CÁO ĐỀ TÀI ESP32 Google Sheet

#I. CÔNG VIỆC ĐÃ LÀM

## 1. Tìm hiểu RFID(RFIC RC522)
		- Tìm hiểu các chân RFID 522
				SDA : Dùng để truyền và nhận dữ liệu giữa module RFID RC522 Arduino.
				SCK : Dùng để đồng bộ hóa truyền thông SPI giữa module RFID RC522 và Arduino.
				MOSI: Dùng để truyền dữ liệu từ Arduino tới module RFID RC522.
				MISO: Chân đầu vào của Arduino nối với chân MISO trên module RFID RC522. Dùng để nhận dữ liệu từ module RFID RC522 về Arduino qua giao thức SPI.
				IRQ : Chân ngắt không được sử dụng trong một số ứng dụng của module RFID RC522. Nếu sử dụng, chân này được sử dụng để xác định xem module RFID RC522 có sự kiện cần xử lý hay không.
				GND : Chân đất kết nối với chân GND trên Arduino.
				RST : Chân đặt lại (reset) module RFID RC522. Khi chân này được kích hoạt, module sẽ được đặt lại về trạng thái ban đầu.
				3.3V: Nguồn cấp 3.3V cho module.
				
		- Kết nối các chân với RFID		
			| **RC522 Pin** | **Chức năng**            | **ESP32 Pin** (trong code)   |
			| ------------- | ------------------------ | ---------------------------- |
			| **SDA (SS)**  | Chân chọn thiết bị       | **D5** (`#define SS_PIN 5`)  |
			| **SCK**       | Clock SPI                | **D18** (mặc định của ESP32) |
			| **MOSI**      | SPI dữ liệu (Master Out) | **D23** (mặc định)           |
			| **MISO**      | SPI dữ liệu (Master In)  | **D19** (mặc định)           |
			| **RST**       | Reset                    | **D0** (`#define RST_PIN 0`) |
			| **3.3V**      | Nguồn                    | **3.3V**                     |
			| **GND**       | Mass                     | **GND**                      |

## 2. Cách kết nối lên google sheet
		Tạo gg sheet trên gg drive:https://docs.google.com/spreadsheets/d/1e1ABqSi2Xge97wklbMty1quTHQfUuu934hzB8mKNHuU/edit?gid=0#gid=0 (link bài tạo)
		Thêm tiện ích apps script 
			+ Dùng cope kết nối với link script 
				function doGet(e) {
			  Logger.log(JSON.stringify(e));
			  var result = 'OK';

			  if (!e.parameter.ten || !e.parameter.mssv || !e.parameter.malop || !e.parameter.ngaysinh) {
				return ContentService.createTextOutput('Thiếu tham số!');
			  }

			  // Lấy ID thực tế từ liên kết
			  var sheet_id = 'https://docs.google.com/spreadsheets/d/1e1ABqSi2Xge97wklbMty1quTHQfUuu934hzB8mKNHuU/edit?usp=sharing'; ( gắn liên kết với link gg sheet )
			  var sheet = SpreadsheetApp.openById(sheet_id).getSheets()[0]; // dùng sheet đầu tiên

			  var newRow = sheet.getLastRow() + 1;
			  var rowData = [
				stripQuotes(e.parameter.ten),
				stripQuotes(e.parameter.mssv),
				stripQuotes(e.parameter.malop),
				stripQuotes(e.parameter.ngaysinh)
			  ];

			  sheet.getRange(newRow, 1, 1, rowData.length).setValues([rowData]);

			  return ContentService.createTextOutput(result);
			}

			function stripQuotes(value) {
			  return value.replace(/^["']|['"]$/g, "");
			}
## 3. Nạp code kết nối với scipt
			#include <WiFi.h>
			#include <HTTPClient.h>
			#include <SPI.h>
			#include <MFRC522.h>

			// WiFi credentials
			const char* ssid = "Huy Hoang";
			const char* password = "hoimehuong12";

			// Google Apps Script URL
			String GOOGLE_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbz6aaPwKjSjlV69xXbRZmtlX1XyETJSx8tYT0JLmbKfLnUvYqyCn0cypfQ892-Fm1sl/exec"; (kết nối cập nhật dữ k=liệu esp lên link scripts)

			#define SS_PIN 5
			#define RST_PIN 0
			MFRC522 mfrc522(SS_PIN, RST_PIN);

			unsigned long lastReadTime = 0;
			const unsigned long readDelay = 5000;

			// Giả lập cơ sở dữ liệu: ánh xạ UID với thông tin sinh viên
			String getStudentInfo(String uid) {
			  if (uid == "C2A7B316") {
				return "ten=Nguyen Van A&mssv=SV001&malop=CTK42&ngaysinh=2003-01-01";
			  } else if (uid == "E4F9B312") {
				return "ten=Le Thi B&mssv=SV002&malop=CTK42&ngaysinh=2002-12-15";
			  }
			  return "";
			}

			void setup() {
			  Serial.begin(115200);
			  SPI.begin();
			  mfrc522.PCD_Init();

			  WiFi.begin(ssid, password);
			  Serial.print("Đang kết nối WiFi");
			  while (WiFi.status() != WL_CONNECTED) {
				delay(500);
				Serial.print(".");
			  }
			  Serial.println("\nWiFi connected!");
			  Serial.println("Sẵn sàng quét thẻ...");
			}

			void loop() {
			  if (!mfrc522.PICC_IsNewCardPresent() || !mfrc522.PICC_ReadCardSerial())
				return;

			  if (millis() - lastReadTime < readDelay) return;
			  lastReadTime = millis();

			  // Lấy UID
			  String uid = "";
			  for (byte i = 0; i < mfrc522.uid.size; i++) {
				if (mfrc522.uid.uidByte[i] < 0x10)
				  uid += "0";  // Thêm số 0 nếu byte < 0x10
				uid += String(mfrc522.uid.uidByte[i], HEX);
			  }
			  uid.toUpperCase();

			  Serial.println("Đã quét UID: " + uid);

			  String params = getStudentInfo(uid);

			  if (params != "") {
				sendToGoogleSheet(params);
			  } else {
				Serial.println("UID chưa đăng ký!");
			  }

			  mfrc522.PICC_HaltA();
			  mfrc522.PCD_StopCrypto1();
			}

			void sendToGoogleSheet(String params) {
			  HTTPClient http;
			  String url = GOOGLE_SCRIPT_URL + "?" + params;
			  Serial.println("Gửi tới URL: " + url);

			  http.begin(url.c_str());
			  int httpCode = http.GET();
			  if (httpCode > 0) {
				String payload = http.getString();
				Serial.println("Phản hồi từ Google Sheet: " + payload);
			  } else {
				Serial.println("Lỗi gửi dữ liệu! Mã lỗi: " + String(httpCode));
			  }
			  http.end();
			}
			
Link Hoàng Tham khảo: https://www.electronicwings.com/esp32/rfid-rc522-interfacing-with-esp32
