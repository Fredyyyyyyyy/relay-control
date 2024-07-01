#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "VASE_WIFI_SIT";
const char* password = "VASE_WIFI_HESLO";
const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 1883;
const char* mqtt_topic = "vasetema/rele1";

const int relayPin = 5;  // GPIO pin pro relé
bool relayState = false;

WiFiClient espClient;
PubSubClient client(espClient);

unsigned long lastMsg = 0;
const long interval = 300000;  // 5 minut v milisekundách

void setup_wifi() {
  delay(10);
  Serial.println("Připojování k WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi připojeno");
}

void callback(char* topic, byte* payload, unsigned int length) {
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println("Přijata zpráva: " + message);
  
  if (message == "ON" && !relayState) {
    digitalWrite(relayPin, HIGH);
    relayState = true;
    publishRelayState();
  } else if (message == "OFF" && relayState) {
    digitalWrite(relayPin, LOW);
    relayState = false;
    publishRelayState();
  }
}

void reconnect() {
  while (!client.connected()) {
    Serial.println("Připojování k MQTT...");
    if (client.connect("ESP32Client")) {
      Serial.println("MQTT připojeno");
      client.subscribe(mqtt_topic);
      publishRelayState();  // Publikuj stav po připojení
    } else {
      Serial.print("Chyba, rc=");
      Serial.print(client.state());
      Serial.println(" zkouším znovu za 5 sekund");
      delay(5000);
    }
  }
}

void publishRelayState() {
  client.publish(mqtt_topic, relayState ? "ON" : "OFF", true);
}

void setup() {
  Serial.begin(115200);
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, LOW);  // Inicializace relé jako vypnuté
  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > interval) {
    lastMsg = now;
    publishRelayState();  // Pravidelná aktualizace stavu
  }
}
