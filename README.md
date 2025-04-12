#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

#define BLYNK_TEMPLATE_ID "TMPL3pjeJJhOA"
#define BLYNK_TEMPLATE_NAME "Smart Plant"
#define BLYNK_AUTH_TOKEN "yIISxEFGkN_Tke5qzdxgB81Ehn_fYFjH"

char ssid[] = "JIOFIBER-5G";
char pass[] = "123456789";

#define DHTPIN D4          // DHT Sensor Pin
#define DHTTYPE DHT11      // DHT Type (DHT11 or DHT22)
DHT dht(DHTPIN, DHTTYPE);

#define SOIL_MOISTURE_PIN A0  // Soil Moisture Pin
#define RELAY_PIN D3           // Relay (Pump) Pin

LiquidCrystal_I2C lcd(0x27, 16, 2);  // Set I2C LCD Address

BlynkTimer timer;

// Function to read sensor data and send it to Blynk
void sendSensorData() {
    float temperature = dht.readTemperature();
    float humidity = dht.readHumidity();
    int soilMoisture = analogRead(SOIL_MOISTURE_PIN);

    // Send data to Blynk App
    Blynk.virtualWrite(V0, temperature);
    Blynk.virtualWrite(V1, humidity);
    Blynk.virtualWrite(V2, soilMoisture);

    // Display on LCD
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temp: ");
    lcd.print(temperature);
    lcd.print("C");
    
    lcd.setCursor(0, 1);
    lcd.print("Soil: ");
    lcd.print(soilMoisture);
}

// Function to control water pump automatically
void controlWaterPump() {
    int soilMoisture = analogRead(SOIL_MOISTURE_PIN);
    if (soilMoisture < 400) {  // Adjust threshold as needed
        digitalWrite(RELAY_PIN, LOW); // Turn ON Pump
        Blynk.virtualWrite(V3, "Pump ON");
    } else {
        digitalWrite(RELAY_PIN, HIGH); // Turn OFF Pump
        Blynk.virtualWrite(V3, "Pump OFF");
    }
}

void setup() {
    Serial.begin(115200);
    Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);
    
    pinMode(RELAY_PIN, OUTPUT);
    digitalWrite(RELAY_PIN, HIGH); // Turn OFF pump initially

    dht.begin();
    lcd.begin(16, 2);
    lcd.backlight();

    // Run functions every few seconds
    timer.setInterval(2000L, sendSensorData);
    timer.setInterval(5000L, controlWaterPump);
}

void loop() {
    Blynk.run();
    timer.run();
}
