/* 
 * ======================================================================
 *  hardware:
 *  1x ESP8266
 * ======================================================================
 *  about this file:
 *  digital garage door: when powered up, the device sets a MQTT topic
 *  to TRUE (= garage door open). When the ESP dies off power-off, the
 *  MQTT last will sets the topic back to FALSE (= garage door closed).
 *  This way the ESP can be powered with a simple reed contact, powering
 *  the ESP only while the door is open.
 *  Smarthome clients can react to the open garage door, e.g. by raising
 *  an alarm in the evening if the door is still open.
 * ======================================================================
 */

#include <ESP8266WiFi.h>
#include <WiFiManager.h>
#include <WiFiClientSecure.h>
#include <PubSubClient.h>
#include <ESP8266HTTPClient.h>
#include <ESP8266httpUpdate.h>
#include <ESP8266mDNS.h>

// ======================================================================
// OTA CONFIGURATION
// ======================================================================
const char* baseUrl  = "YOUR_OTA_SERVER_URL_GOES_HERE";
#define CURRENT_VERSION "1.0"

// ======================================================================
// CONFIGURATION
// ======================================================================

const char* mqttServer   = "YOUR_MQTT_BROKER_URL_GOES_HERE";
const char* mqttUsername = "MQTT_USERNAME";
const char* mqttPassword = "MQTT_PASSWORD";

// ======================================================================
// OBJECTS & VARIABLES
// ======================================================================

WiFiManager wifiManager;


WiFiClientSecure secureClient;
WiFiClient mqttClient;
PubSubClient client(mqttClient);


// ======================================================================
// FUNCTIONS
// ======================================================================

void checkForOTA() {
  String chipID = String(ESP.getChipId(), HEX);
  chipID.toUpperCase();

  String versionUrl  = String(baseUrl) + "fw_" + chipID + ".txt";
  String firmwareUrl = String(baseUrl) + "fw_" + chipID + ".bin";

  Serial.println("Version-URL: " + versionUrl);
  Serial.println("Firmware-URL: " + firmwareUrl);

  HTTPClient http;
  WiFiClient httpClient;
  http.begin(httpClient, versionUrl.c_str());
  int httpCode = http.GET();

  if (httpCode == HTTP_CODE_OK) {
    String newVersion = http.getString();
    newVersion.trim();
    Serial.printf("ESP-Version: %s / Server-Version: %s\n",
                  CURRENT_VERSION, newVersion.c_str());

    if (!newVersion.equals(CURRENT_VERSION)) {
      Serial.println("Neue Version gefunden – starte OTA-Update ...");
      WiFiClient client;
      t_httpUpdate_return ret = 
        ESPhttpUpdate.update(client, firmwareUrl, CURRENT_VERSION);

      switch (ret) {
        case HTTP_UPDATE_FAILED:
          Serial.printf("Update fehlgeschlagen. Fehler (%d): %s\n",
                        ESPhttpUpdate.getLastError(),
                        ESPhttpUpdate.getLastErrorString().c_str());
          break;

        case HTTP_UPDATE_NO_UPDATES:
          Serial.println("Keine neue Version gefunden.");
          break;

        case HTTP_UPDATE_OK:
          Serial.println("Update erfolgreich! Neustart ...");
          break;
      }
    } else {
      Serial.println("Firmware ist aktuell.");
    }
  } else {
    Serial.printf("Fehler beim Abrufen der Version: HTTP-Code %d\n",
      httpCode);
  }
  http.end();
}

void connectToMQTT() {
  while (!client.connected()) {
    if (client.connect("garagentor", mqttUsername, mqttPassword,
               "monitor/garagentor", 1, true, "false")) {
    } else {
      delay(5000);
    }
  }
}



// ======================================================================
// SETUP
// ======================================================================

void setup() {
  Serial.begin(115200);

  // WiFi with WiFiManager
  String ssid = "garagentor_" + String(ESP.getChipId());
  wifiManager.autoConnect(ssid.c_str());
  Serial.println("WiFi connected.");

  if (MDNS.begin("garagentor")) {
    Serial.println("mDNS responder gestartet: garagentor.local");
  } else {
    Serial.println("Fehler beim Starten des mDNS responders");
  }

  // check OTA for updates
  checkForOTA();

  // start MQTT
  client.setServer(mqttServer, 1883);
  connectToMQTT();

  delay(2000);

  // ESP powered up --> garage door open
  if (client.connected()) {
    client.publish("monitor/garagentor", "true");
    Serial.printf("MQTT: true (Garagentor AUF)");
  }
}

// ======================================================================
// LOOP
// ======================================================================
void loop() {
  if (!client.connected()) {
    connectToMQTT();
  }
  client.loop();

  delay(1000);
}

