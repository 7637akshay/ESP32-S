#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <HTTPClient.h>
#include <Update.h>

// ----------- WiFi credentials -----------
const char* ssid = "Knest-R&D-2G";
const char* password = "Knest@007";

// ----------- GitHub repo links -----------
const char* version_url = "https://raw.githubusercontent.com/7637akshay/ESP32-S4/main/version.txt";
const char* base_firmware_url = "https://raw.githubusercontent.com/7637akshay/ESP32-S4/main/firmware_V";

// ----------- Firmware version (bump this before compiling new firmware) -----------
#define FW_VERSION "1.0"

// ----------- Timer variables -----------
unsigned long lastUpdateCheck = 0;
const unsigned long updateInterval = 10 * 60 * 1000; // 5 minutes

// ----------- OTA Update function -----------
void checkForUpdates() {
  WiFiClientSecure client;
  client.setInsecure();
  HTTPClient http;

  Serial.println("\n[OTA] Checking for update...");
  Serial.printf("[OTA] Current device version: %s\n", FW_VERSION);

  if (http.begin(client, version_url)) {
    int httpCode = http.GET();

    if (httpCode == 200) {
      String newVersion = http.getString();
      newVersion.trim();
      Serial.printf("[OTA] Latest version on server: %s\n", newVersion.c_str());

      if (newVersion != FW_VERSION) {
        Serial.println("[OTA] New version available! Starting update...");

        // Build firmware filename dynamically (firmware_VX.X.bin)
        String firmwareURL = String(base_firmware_url) + newVersion + ".bin";
        Serial.printf("[OTA] Downloading firmware: %s\n", firmwareURL.c_str());

        HTTPClient httpUpdate;
        if (httpUpdate.begin(client, firmwareURL)) {
          int updateCode = httpUpdate.GET();

          if (updateCode == 200) {
            int contentLength = httpUpdate.getSize();
            if (contentLength > 0 && Update.begin(contentLength)) {
              WiFiClient* stream = httpUpdate.getStreamPtr();
              size_t written = Update.writeStream(*stream);
              if (written == contentLength) {
                Serial.println("[OTA] Firmware written successfully");
              } else {
                Serial.printf("[OTA] Written only : %d/%d bytes\n", written, contentLength);
              }

              if (Update.end() && Update.isFinished()) {
                Serial.println("[OTA] ✅ Update successful! Rebooting...");
                delay(3000); // Give time for Serial message to print
                ESP.restart();
              } else {
                Serial.printf("[OTA] Update error: %s\n", Update.errorString());
              }
            }
          } else {
            Serial.printf("[OTA] Firmware download failed. HTTP code: %d\n", updateCode);
          }
          httpUpdate.end();
        }
      } else {
        Serial.println("[OTA] Device is already on the latest version.");
      }
    } else {
      Serial.printf("[OTA] Version check failed. HTTP code: %d\n", httpCode);
    }
    http.end();
  }
}

// ----------- Setup & Loop -----------
void setup() {
  Serial.begin(115200);
  pinMode(2, OUTPUT);

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n[WiFi] Connected!");

  Serial.printf("[OTA] Running firmware version: %s\n", FW_VERSION);

  // First check on boot
  checkForUpdates();
}

void loop() {
  // Blink LED (example normal operation)
  static unsigned long lastBlink = 0;
  if (millis() - lastBlink >= 500) {
    digitalWrite(2, !digitalRead(2));
    lastBlink = millis();
  }

  // Check for updates every 5 minutes
  if (millis() - lastUpdateCheck >= updateInterval) {
    checkForUpdates();
    lastUpdateCheck = millis();
  }
}
