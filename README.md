# esp32_ble
ESP32 Wi-Fi Provisioning via BLE (Bluetooth Low Energy) – Arduino IDE
#include "WiFiProv.h"
#include "WiFi.h"

String pop = "abcd1234"; // Proof of possession - PIN for provisioning
String service_name = "PROV_123"; // Device name for provisioning
String service_key = ""; // Empty string for SofAP method (not needed)
bool reset_provisioned = true; // Automatically delete previously provisioned data

// Wi-Fi Provisioning Event Handler
void SysProvEvent(arduino_event_t *sys_event) {
    switch (sys_event->event_id) {
    case ARDUINO_EVENT_WIFI_STA_GOT_IP:
        Serial.print("\nConnected IP address : ");
        Serial.println(IPAddress(sys_event->event_info.got_ip.ip_info.ip.addr));
        break;
    case ARDUINO_EVENT_WIFI_STA_DISCONNECTED:
        Serial.println("\nDisconnected. Reconnecting...");
        break;
    case ARDUINO_EVENT_PROV_START:
        Serial.println("\nProvisioning started\nProvide Wi-Fi credentials via the app.");
        break;
    case ARDUINO_EVENT_PROV_CRED_RECV: {
        Serial.println("\nWi-Fi credentials received");
        Serial.print("\tSSID: ");
        Serial.println((const char *) sys_event->event_info.prov_cred_recv.ssid);
        Serial.print("\tPassword: ");
        Serial.println((const char *) sys_event->event_info.prov_cred_recv.password);
        break;
    }
    case ARDUINO_EVENT_PROV_CRED_FAIL: {
        Serial.println("\nProvisioning failed!\nPlease reset and try again.");
        if (sys_event->event_info.prov_fail_reason == WIFI_PROV_STA_AUTH_ERROR)
            Serial.println("\nIncorrect Wi-Fi password");
        else
            Serial.println("\nWi-Fi AP not found");
        break;
    }
    case ARDUINO_EVENT_PROV_CRED_SUCCESS:
        Serial.println("\nProvisioning successful!");
        break;
    case ARDUINO_EVENT_PROV_END:
        Serial.println("\nProvisioning completed.");
        break;
    default:
        break;
    }
}

void setup() {
  Serial.begin(115200);
  WiFi.onEvent(SysProvEvent);

  Serial.println("Starting BLE Provisioning...");

  // Dynamic UUID allocation
  uint8_t *uuid = new uint8_t[16];
  uuid[0] = 0xb4; uuid[1] = 0xdf; uuid[2] = 0x5a; uuid[3] = 0x1c;
  uuid[4] = 0x3f; uuid[5] = 0x6b; uuid[6] = 0xf4; uuid[7] = 0xbf;
  uuid[8] = 0xea; uuid[9] = 0x4a; uuid[10] = 0x82; uuid[11] = 0x03;
  uuid[12] = 0x04; uuid[13] = 0x90; uuid[14] = 0x1a; uuid[15] = 0x02;

  // Start provisioning using BLE
  WiFiProv.beginProvision(WIFI_PROV_SCHEME_BLE, WIFI_PROV_SCHEME_HANDLER_FREE_BTDM, WIFI_PROV_SECURITY_1, pop.c_str(), service_name.c_str(), service_key.c_str(), uuid, reset_provisioned);

  delete[] uuid; // Free the dynamically allocated memory
}

void loop() {
  // Event-driven provisioning; nothing needed in loop
}

