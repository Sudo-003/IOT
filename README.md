# IOT

***Practical 2 — Python Word & Character Count***<br>
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
***Practical 3 — Raspberry Pi Pico + PIR Sensor***<br>
**I recommend using Serial so the output is visible in Wokwi's Serial Monitor**
```
void setup() {
  pinMode(15, INPUT);

  Serial.begin(115200);
  Serial.println("Hello, Raspberry Pi Pico!");
}

void loop() {
  int pir = digitalRead(15);

  if (pir == HIGH) {
    Serial.println("MOVEMENT DETECTED");
    delay(500);
  } 
  else {
    Serial.println("NO MOVEMENT DETECTED");
    delay(500);
  }
}

```
