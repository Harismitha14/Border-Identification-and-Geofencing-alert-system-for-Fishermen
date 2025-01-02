# Border-Identification-and-Geofencing-alert-system-for-Fishermen
The system navigates autonomously and alerts fishermen when approaching maritime border &amp; when border is crossed through Buzzer &amp; Telegram notification.

//Code for obtaining GPS co-ordinates and HC-05 Bluetooth module communication
#include <TinyGPS++.h>
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <ArduinoJson.h>
#include <HardwareSerial.h>

#define RXD2 16
#define TXD2 17
#define GPS_BAUD 9600


const char* ssid = "realmeC12";
const char* password = "0123456789";

#define BOTtoken "7814301009:AAGHtx_zzyo_9j1dcbJMSoNM_ziDIA8t27o"  // your Bot Token (Get from Botfather)

#define CHAT_ID "2014387236"


WiFiClientSecure client;
UniversalTelegramBot bot(BOTtoken, client);

TinyGPSPlus gps;
// Create an instance of the HardwareSerial class for Serial 2
HardwareSerial gpsSerial(2);

HardwareSerial Serialbt(1);

void setup() {
  Serial.begin(115200); // Debugging on Serial Monitor
  Serial.print("Connecting Wifi: ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  client.setCACert(TELEGRAM_CERTIFICATE_ROOT); // Add root certificate for api.telegram.org
  
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(500);
  }

    Serial.println("");
  Serial.println("WiFi connected");

  
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
  // Start Serial 2 for GPS with defined RX and TX pins
  gpsSerial.begin(GPS_BAUD, SERIAL_8N1, RXD2, TXD2);
  Serial.println("Serial 2 (GPS) started at 9600 baud rate");
  Serialbt.begin(115200, SERIAL_8N1,3,1); // RX: GPI03, TX: GPIO1
  Serial.println("ESP32 ready to send data.");

}

void loop() {
  
  while (gpsSerial.available() > 0) {
    char gpsData = gpsSerial.read();
    gps.encode(gpsData); // Feed data into the TinyGPS++ library

    // When a new location is available, print latitude and longitude
    if (gps.location.lat()>12.00&&gps.location.lng()>74.00) {
      Serial.print("Latitude: ");
      Serial.println(gps.location.lat(), 6);  // 6 decimal places for latitude
      Serial.print("Longitude: ");
      Serial.println(gps.location.lng(), 6);  // 6 decimal places for longitude
      Serialbt.println("0");
      delay(1000);
      Serial.println("-------------------------------");
      bot.sendMessage(CHAT_ID, "Approching border", "");
      delay(1000);
    }
    if (gps.location.lat()>12.80&&gps.location.lng()>74.80) {
      Serial.print("Latitude: ");
      Serial.println(gps.location.lat(), 6);  // 6 decimal places for latitude
      Serial.print("Longitude: ");
      Serial.println(gps.location.lng(), 6);  // 6 decimal places for longitude
      Serialbt.println("1");
      Serial.println("-------------------------------");
      bot.sendMessage(CHAT_ID, "border", "");
      delay(1000);
    }
    
  }
  delay(1000); // Wait for 1 second before sending the next message
}
