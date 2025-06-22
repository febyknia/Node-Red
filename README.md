
>👩🏻‍💻Feby 🗓 22 June 2025

# TEMPERATURE AND HUMIDITY MONITORING SIMULATION BASED ON IOT USING ESP32 AND NODERED

---
## **A. Introduction**

This practicum is a design and simulation practicum of an Internet of Things (IoT)-based temperature and humidity monitoring system using an ESP32 microcontroller and a DHT22 sensor, developed virtually through the Wokwi platform and Visual Studio Code.

## **B. Diagram**
<img width="1020" alt="Screenshot 2025-06-22 233212" src="https://github.com/user-attachments/assets/3cc46f40-8c36-4a82-baab-1ef5be8764ce" />



```json
{
  "version": 1,
  "author": "Anonymous maker",
  "editor": "wokwi",
  "parts": [
    { "type": "wokwi-esp32-devkit-v1", "id": "esp", "top": -278.9, "left": 52.76, "attrs": {} },
    { "type": "wokwi-led", "id": "led1", "top": -262.8, "left": 3.8, "attrs": { "color": "red" } },
    {
      "type": "wokwi-resistor",
      "id": "r5",
      "top": -196.8,
      "left": -0.55,
      "rotate": 90,
      "attrs": { "value": "1000" }
    },
    {
      "type": "wokwi-dht22",
      "id": "dht1",
      "top": -258.9,
      "left": 177,
      "attrs": { "temperature": "58.7", "humidity": "77" }
    }
  ],
  "connections": [
    [ "esp:TX0", "$serialMonitor:RX", "", [] ],
    [ "esp:RX0", "$serialMonitor:TX", "", [] ],
    [ "led1:A", "r5:1", "red", [ "v0" ] ],
    [ "led1:C", "esp:GND.2", "black", [ "v0" ] ],
    [ "esp:D15", "dht1:SDA", "black", [ "h0" ] ],
    [ "esp:GND.1", "dht1:GND", "black", [ "h0" ] ],
    [ "esp:D13", "r5:2", "black", [ "h0" ] ],
    [
      "dht1:VCC",
      "esp:VIN",
      "black",
      [
        "v0",
        "h-28.8",
        "v-134.4",
        "h-115.2",
        "v134.4",
        "h-9.6",
        "v0",
        "h0",
        "v-115.2",
        "h-28.8",
        "v134.4",
        "h0",
        "v9.6"
      ]
    ]
  ],
  "dependencies": {}
}
```

## **C. Program Code**
This program code is for setting the temperature and humidity, as well as the connection to NodeRed.

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHTesp.h>

const int LED_RED = 2;
const int DHT_PIN = 15;
DHTesp dht; 



const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "broker.emqx.io"; //server

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
float temp = 0;
float hum = 0;

void setup_wifi() { 
  delay(10);
  
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA); 
  WiFi.begin(ssid, password); 

  while (WiFi.status() != WL_CONNECTED) { 
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) { 
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) { 
    Serial.print((char)payload[i]);
  }
  Serial.println();

  
  if ((char)payload[0] == '1') {
    digitalWrite(LED_RED, HIGH);  
  } else {
    digitalWrite(LED_RED, LOW);  
  }
}

void reconnect() { 
  
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    
    String clientId = "ESP32Client-";
    clientId += String(random(0xffff), HEX);
    
    if (client.connect(clientId.c_str())) {
      Serial.println("Connected");
      
      client.publish("IOT/Test1/mqtt", "Test IOT"); 
      
      client.subscribe("IOT/Test1/mqtt"); 
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      
      delay(5000);
    }
  }
}

void setup() {
  pinMode(LED_RED, OUTPUT);    
  Serial.begin(115200);
  setup_wifi(); 
  client.setServer(mqtt_server, 1883); 
  client.setCallback(callback); 
  dht.setup(DHT_PIN, DHTesp::DHT22);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) { 
    lastMsg = now;
    TempAndHumidity  data = dht.getTempAndHumidity();

    String temp = String(data.temperature, 2); 
    client.publish("IOT/Test1/temp", temp.c_str()); //Topic Temperature
    
    String hum = String(data.humidity, 1); 
    client.publish("IOT/Test1/hum", hum.c_str()); //Topic Humidity

    Serial.print("Temperature: ");
    Serial.println(temp);
    Serial.print("Humidity: ");
    Serial.println(hum);
  }
}
```
## **C. Result**
The results of the practicum of monitoring temperature and humidity show that the dashboard can send data in realtime from the project folder from both Wokwi and Visual Studio Code to the dashboard display that has been set up the diagram flow and configuration of each part. The dashboard is divided into two pages namely the page for IOT / Temperature and the Humidity page, both have the same flow of work but what distinguishes is the topic chosen different and for other elements customized according to the
needs of each respectively.

Temperature 23.0 °C and 40.7 °C 
<img width="1020" alt="Screenshot 2025-06-22 205816" src="https://github.com/user-attachments/assets/47cfacd0-54ee-4460-8241-b48638675545" />
<img width="1020" alt="Screenshot 2025-06-22 205959" src="https://github.com/user-attachments/assets/1af32a68-57bd-4a63-a4aa-f37d30e83637" />


---
Humidity 52.0 °C dan 66.0 °C 
<img width="1020" alt="Screenshot 2025-06-22 205847" src="https://github.com/user-attachments/assets/6e7f5a98-3f59-4ac6-b7f4-e97152a61f6f" />
<img width="1020" alt="Screenshot 2025-06-22 210045" src="https://github.com/user-attachments/assets/fb06b3b9-3eab-49da-b2b3-739c781fa153" />





For more details, please follow the implementation steps in the NodeRed check report (pdf).
