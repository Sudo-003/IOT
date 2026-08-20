# IOT

## Practical 2 — Run python program on Pi having problem statement: word and character count of a given string.
**File: `word_char_count.py`**<br>
**This is a normal Python program, not a Wokwi hardware simulation.**
```
# Word and Character Count Program

text = input("Enter a string: ")

word_count = len(text.split())
char_count = len(text)

print("Number of Words:", word_count)
print("Number of Characters:", char_count)

```

**Note:- Example Image**<br>
![image alt](https://github.com/Sudo-003/IOT/blob/4f6ddbe8085da1e020e5336c024210da9596a2a4/Practical-2.png)<br>

## Practical 3 — Exercise on working principle of Raspberry Pi.with 40 PIN Interface component
![image alt](https://github.com/Sudo-003/IOT/blob/4d68ce4f098020dd1ee059ce7d923dba107cd161/Practical-3.png)

**Sketch.ino**
```
void setup() {
  pinMode(15,INPUT);
  Serial1.begin(115200);
  Serial1.println("Hello, Raspberry Pi Pico!");
}

void loop(){
  int pir = digitalRead(15);
  if(pir == HIGH)
  {
    Serial1.println("MOVEMENT DECTECTED");
    delay(500);
  } 
  else
  {
    Serial1.println("NO MOVEMENT DECTECTED");
    delay(500);
  }
}

```
**diagram.json**<br>
```
{
  "version": 1,
  "editor": "wokwi",
  "parts": [
    { "type": "wokwi-pi-pico", "id": "pico", "top": -12.75, "left": -130.8, "attrs": {} },
    { "type": "wokwi-pir-motion-sensor", "id": "pir1", "top": -92, "left": 40.62, "attrs": {} }
  ],
  "connections": [
    [ "pico:GP0", "$serialMonitor:RX", "", [] ],
    [ "pico:GP1", "$serialMonitor:TX", "", [] ],
    [ "pir1:VCC", "pico:VBUS", "red", [ "v0", "h-124.8" ] ],
    [ "pir1:GND", "pico:GND.8", "black", [ "v0" ] ],
    [ "pir1:OUT", "pico:GP15", "green", [ "v220.8", "h-259.34", "v-38.4" ] ]
  ],
  "dependencies": {}
}

```

## Practical 4 — Bluetooth + LED ON/OFF
**Original MicroPython code**<br>
```
from machine import Pin, UART

uart = UART(0, 9600)
led = Pin(19, Pin.OUT)

while True:
    if uart.any():
        command = uart.readline()

        if command == b'ON':
            led.high()
            print("LED ON")

        elif command == b'OFF':
            led.low()
            print("LED OFF")

```

**Wokwi alternative**<br>
```
from machine import Pin

led = Pin(19, Pin.OUT)

while True:
    command = input("Enter ON or OFF: ")

    if command == "ON":
        led.on()
        print("LED ON")

    elif command == "OFF":
        led.off()
        print("LED OFF")

```
## Practical 5 — Connect with the Available Wi-Fi Using Arduino
**Board :- Use an ESP32 in Wokwi.** <br>
**sketch.ino**
```

#include <WiFi.h>

// Replace with your network credentials (STATION)
#define ssid "Wokwi-GUEST"
#define password ""


void initWiFi() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  // WiFi.begin("Wokwi-GUEST", "");
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.println(WiFi.status());
    Serial.print('.');
    delay(1000);
  }
  Serial.println("Connected");
  Serial.println(WiFi.status());
  Serial.println(WiFi.localIP());
  Serial.print("RRSI: ");
  Serial.println(WiFi.RSSI());
}

void setup() {
  Serial.begin(115200);
  initWiFi();
}

void loop() {
  // put your main code here, to run repeatedly:
}

```
## Practical 6 — Write a program on Raspberry Pi to retrieve temperature and humidity data from thing speak cloud 
**Board :- Raspberry Pi Pico W / suitable Wi-Fi MicroPython board** <br>
**sketch.ino**

```
import network, urequests, time

# ---------- WiFi ----------
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect("Wokwi-GUEST", "")
while not wlan.isconnected():
    time.sleep(0.5)
print("WiFi connected\n")

# ---------- ThingSpeak settings ----------
# Public demo channel (works out of the box, no key needed)
CHANNEL_ID = "12397"
READ_KEY   = ""
TEMP_FIELD = "field4"    # temperature on demo channel
HUM_FIELD  = "field3"    # humidity on demo channel

# ---------- Build the READ url (do not change) ----------
URL = "https://api.thingspeak.com/channels/" + CHANNEL_ID + "/feeds/last.json"
if READ_KEY:
    URL += "?api_key=" + READ_KEY

# ---------- Main loop ----------
while True:
    try:
        r = urequests.get(URL)
        data = r.json()
        r.close()

        if isinstance(data, dict):
            print("Time    :", data.get("created_at"))
            print("Temp    :", data.get(TEMP_FIELD))
            print("Humidity:", data.get(HUM_FIELD))
            print("-" * 30)
        else:
            print("Bad channel ID / key, server said:", data)
    except Exception as e:
        print("Network error:", e)

    time.sleep(20)

```
## Practical 7 — Write a program on Raspberry Pi to publish temperature data to MQTT broker (Message Queuing Telemetry Transport) 
![image alt](https://github.com/Sudo-003/IOT/blob/49f338683cda7932bbb9e4f310aa0480921d3193/Practical-7.png) <br>

**main.py**
```
import network
import time
from machine import Pin
import dht
import ujson
from umqtt.simple import MQTTClient

# MQTT Server Parameters
MQTT_CLIENT_ID = "micropython-weather-demo"
MQTT_BROKER    = "broker.mqttdashboard.com"
MQTT_USER      = ""
MQTT_PASSWORD  = ""
MQTT_TOPIC     = "wokwi-weather"

sensor = dht.DHT22(Pin(15))

print("Connecting to WiFi", end="")
sta_if = network.WLAN(network.STA_IF)
sta_if.active(True)
sta_if.connect('Wokwi-GUEST', '')
while not sta_if.isconnected():
  print(".", end="")
  time.sleep(0.1)
print(" Connected!")

print("Connecting to MQTT server... ", end="")
client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER, user=MQTT_USER, password=MQTT_PASSWORD)
client.connect()

print("Connected!")

prev_weather = ""
while True:
  print("Measuring weather conditions... ", end="")
  sensor.measure() 
  message = ujson.dumps({
    "temp": sensor.temperature(),
    "humidity": sensor.humidity(),
  })
  if message != prev_weather:
    print("Updated!")
    print("Reporting to MQTT topic {}: {}".format(MQTT_TOPIC, message))
    client.publish(MQTT_TOPIC, message)
    prev_weather = message
  else:
    print("No change")
  time.sleep(1)

```
**diagram.json**
```
{
  "version": 1,
  "editor": "wokwi",
  "parts": [
    { "type": "board-esp32-devkit-c-v4", "id": "esp", "top": -37.58, "left": -96.37, "attrs": {} },
    { "type": "wokwi-dht22", "id": "dht1", "top": -32.2, "left": 40.16, "attrs": {} }
  ],
  "connections": [
    [ "esp:TX", "$serialMonitor:RX", "", [] ],
    [ "esp:RX", "$serialMonitor:TX", "", [] ],
    [ "dht1:VCC", "esp:3V3", "red", [ "v109.3", "h-170.36", "v-200.78" ] ],
    [ "dht1:SDA", "esp:15", "green", [ "v0" ] ],
    [ "dht1:GND", "esp:GND.1", "black", [ "v99.7", "h-189.56", "v-66.38" ] ]
  ],
  "dependencies": {}
}

```
## Practical 8 — Connect Raspberry Pi with your existing system components
