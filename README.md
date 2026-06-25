#define BLYNK_TEMPLATE_ID "YOUR_TEMPLATE_ID"
#define BLYNK_TEMPLATE_NAME "Smart Irrigation"
#define BLYNK_AUTH_TOKEN "YOUR_AUTH_TOKEN"

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <DHT.h>

char ssid[] = "YOUR_WIFI_NAME";
char pass[] = "YOUR_WIFI_PASSWORD";

#define SOIL_PIN 34
#define WATER_LEVEL_PIN 35
#define RELAY_PIN 26
#define DHTPIN 4
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);
BlynkTimer timer;

void sendSensorData() {
  int soil = analogRead(SOIL_PIN);
  int waterLevel = analogRead(WATER_LEVEL_PIN);
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  Blynk.virtualWrite(V0, soil);
  Blynk.virtualWrite(V1, waterLevel);
  Blynk.virtualWrite(V2, temperature);
  Blynk.virtualWrite(V3, humidity);

  Serial.print("Soil: ");
  Serial.println(soil);

  if (soil > 2500) {
    digitalWrite(RELAY_PIN, LOW);   // Pump ON
  } else {
    digitalWrite(RELAY_PIN, HIGH);  // Pump OFF
  }
}

void setup() {
  Serial.begin(115200);

  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, HIGH);

  dht.begin();

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  timer.setInterval(2000L, sendSensorData);
}

void loop() {
  Blynk.run();
  timer.run();
}
// ---------- Blynk Manual Pump Control ----------
BLYNK_WRITE(V4)
{
  int pump = param.asInt();

  if (pump == 1)
  {
    digitalWrite(RELAY_PIN, LOW);
    Serial.println("Pump ON (Manual)");
  }
  else
  {
    digitalWrite(RELAY_PIN, HIGH);
    Serial.println("Pump OFF (Manual)");
  }
}

// ---------- Automatic Irrigation ----------
void automaticIrrigation()
{
  int soil = analogRead(SOIL_PIN);
  int water = analogRead(WATER_LEVEL_PIN);

  float temp = dht.readTemperature();
  float hum = dht.readHumidity();

  Serial.println("-------------------------");
  Serial.print("Soil Moisture : ");
  Serial.println(soil);

  Serial.print("Water Level : ");
  Serial.println(water);

  Serial.print("Temperature : ");
  Serial.println(temp);

  Serial.print("Humidity : ");
  Serial.println(hum);

  Blynk.virtualWrite(V0, soil);
  Blynk.virtualWrite(V1, water);
  Blynk.virtualWrite(V2, temp);
  Blynk.virtualWrite(V3, hum);

  // Dry soil
  if (soil > 2500)
  {
    digitalWrite(RELAY_PIN, LOW);
    Blynk.virtualWrite(V5, "Pump ON");
  }
  else
  {
    digitalWrite(RELAY_PIN, HIGH);
    Blynk.virtualWrite(V5, "Pump OFF");
  }

  // Water tank low alert
  if (water < 1000)
  {
    Blynk.logEvent("water_low", "Water Tank Level is LOW!");
  }
}
timer.setInterval(2000L, sendSensorData);
timer.setInterval(2000L, automaticIrrigation);
