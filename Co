// กำหนดค่าคอนฟิก Blynk
#define BLYNK_TEMPLATE_ID "TMPL6nnxMqks-"
#define BLYNK_TEMPLATE_NAME "ESP8266CODHT22"
#define BLYNK_AUTH_TOKEN "Bc55-FJ9TKsJC4wTxwTrAA9s4_MO_UY5"

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>  // เพิ่มไลบรารี ArduinoJson สำหรับการจัดการ JSON

// Pin Configuration
#define DHTPIN D4         // Pin เชื่อมต่อ DHT22
#define DHTTYPE DHT22     // ระบุเซ็นเซอร์ DHT22
#define MQ7PIN A0         // Pin เชื่อมต่อ MQ-7

DHT dht(DHTPIN, DHTTYPE);

// WiFi Credentials
char ssid[] = "OPPO";       // ชื่อ WiFi
char pass[] = "12345678";   // รหัสผ่าน WiFi

// HTTP Server Base URL
const char* serverURL = "http://npmh.moph.go.th/api/itwork/api/iot/temporature";

WiFiClient client;  // สร้าง WiFiClient

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, pass);
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);
  dht.begin();

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi Connected!");
}

void loop() {
  Blynk.run();

  static unsigned long lastSendTime = 0;
  if (millis() - lastSendTime > 60000) { // ทุก 1 นาที (60,000 ms)
    lastSendTime = millis();

    float temperature = dht.readTemperature();
    float humidity = dht.readHumidity();
    int coConcentration = analogRead(MQ7PIN);

    if (isnan(temperature) || isnan(humidity)) {
      Serial.println("Failed to read from DHT sensor!");
      return;
    }

    // ส่งข้อมูลผ่าน Blynk
    Blynk.virtualWrite(V1, temperature);
    Blynk.virtualWrite(V2, humidity);
    Blynk.virtualWrite(V3, coConcentration);

    // สร้าง JSON object
    StaticJsonDocument<200> doc;
    doc["coConcentration"] = coConcentration;
    doc["humidity"] = humidity;
    doc["temperature"] = temperature;

    // แปลง JSON object เป็น string
    String postData;
    serializeJson(doc, postData);

    // สร้าง URL สำหรับการส่งข้อมูล
    String fullURL = String(serverURL);

    // ส่งข้อมูลผ่าน HTTPClient ด้วย POST Method
    HTTPClient http;
    http.begin(client, fullURL); // ใช้ WiFiClient พร้อม URL
    http.addHeader("Content-Type", "application/json"); // ระบุประเภทของข้อมูลที่ส่งเป็น JSON

    int httpResponseCode = http.POST(postData); // ส่งข้อมูลแบบ POST

    if (httpResponseCode > 0) {
      Serial.println("POST Response: " + String(httpResponseCode));
      Serial.println("Response Body: " + http.getString());
    } else {
      Serial.println("Error on sending POST: " + http.errorToString(httpResponseCode));
    }
    http.end();
  }
}
