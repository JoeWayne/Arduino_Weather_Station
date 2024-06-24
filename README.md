
# Arduino Weather Station with REST API Integration

This Arduino project uses a DHT22 sensor to measure local temperature and humidity, and an ESP8266 WiFi module to fetch current weather data from the OpenWeatherMap REST API. The data is output to the Serial Monitor, providing a comparison between local sensor readings and online weather data.

## Features
- Measure and display local temperature and humidity using a DHT22 sensor.
- Connect to WiFi and fetch current weather data from the OpenWeatherMap API.
- Output local and API weather data to the Serial Monitor.
- Modular code structure with functions for better readability and maintainability.

## Components Needed
- Arduino board (e.g., Arduino Uno)
- DHT22 temperature and humidity sensor
- ESP8266 WiFi module
- Breadboard and jumper wires

## Setup
1. Connect the DHT22 sensor to the Arduino.
2. Connect the ESP8266 WiFi module to the Arduino.
3. Install necessary libraries: `DHT`, `ESP8266WiFi`, `ESP8266HTTPClient`.
4. Replace the placeholders (`YOUR_SSID`, `YOUR_PASSWORD`, `YOUR_API_KEY`, `YOUR_CITY`, `YOUR_COUNTRY_CODE`) with your actual WiFi credentials and OpenWeatherMap API information.
5. Upload the code to your Arduino and open the Serial Monitor to view the data.

## Libraries Used
- `DHT`: for reading temperature and humidity data.
- `ESP8266WiFi`: for WiFi connectivity.
- `ESP8266HTTPClient`: for HTTP requests to the weather API.

## Code

```cpp
#include <DHT.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

// DHT Sensor setup
#define DHT_SENSOR_PIN 2
#define DHT_SENSOR_TYPE DHT22
DHT dht(DHT_SENSOR_PIN, DHT_SENSOR_TYPE);

// WiFi credentials
const char* wifiSSID = "YOUR_SSID";
const char* wifiPassword = "YOUR_PASSWORD";

// OpenWeatherMap API configuration
const String weatherAPIKey = "YOUR_API_KEY";
const String cityName = "YOUR_CITY";
const String countryCode = "YOUR_COUNTRY_CODE";

// Timing variables
unsigned long lastUpdateTime = 0;
const long updateInterval = 60000; // 1 minute interval

void setup() {
  Serial.begin(115200);
  dht.begin();

  // Connect to WiFi
  connectToWiFi();
}

void loop() {
  if (millis() - lastUpdateTime >= updateInterval) {
    lastUpdateTime = millis();

    // Read and print sensor data
    readAndPrintSensorData();

    // Fetch and print weather data from API
    fetchAndPrintWeatherData();
  }
}

void connectToWiFi() {
  WiFi.begin(wifiSSID, wifiPassword);
  Serial.print("Connecting to WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" Connected!");
}

void readAndPrintSensorData() {
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Error reading from DHT sensor");
    return;
  }

  Serial.print("Local Temp: ");
  Serial.print(temperature);
  Serial.print(" °C, Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");
}

void fetchAndPrintWeatherData() {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String apiUrl = "http://api.openweathermap.org/data/2.5/weather?q=" + cityName + "," + countryCode + "&appid=" + weatherAPIKey + "&units=metric";
    http.begin(apiUrl);
    int httpResponseCode = http.GET();

    if (httpResponseCode > 0) {
      String response = http.getString();
      Serial.println(response);

      // Basic JSON parsing
      int tempIndex = response.indexOf(""temp":") + 7;
      float apiTemperature = response.substring(tempIndex, response.indexOf(",", tempIndex)).toFloat();

      int humIndex = response.indexOf(""humidity":") + 11;
      int apiHumidity = response.substring(humIndex, response.indexOf("}", humIndex)).toInt();

      Serial.print("API Temp: ");
      Serial.print(apiTemperature);
      Serial.print(" °C, Humidity: ");
      Serial.print(apiHumidity);
      Serial.println(" %");
    } else {
      Serial.println("HTTP request failed");
    }
    http.end();
  } else {
    Serial.println("WiFi Disconnected");
  }
}
```
