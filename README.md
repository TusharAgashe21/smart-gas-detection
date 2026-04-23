//# smart-gas-detection
//4th sem micro projet IIOT SVPCET
#include <WiFi.h>
#include <HTTPClient.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// 📶 WiFi
const char* ssid = "OnePlus Nord 4";
const char* password = "123451234";

// 🤖 Telegram
String botToken = "8747945819:AAH_VZwVTt5di1EB_IZ8OxZGqueZiE9G6WQ";
String chatID   = "1868788761";

// 📟 LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// 📌 Pins
#define MQ2_PIN 34
#define MQ135_PIN 35
#define BUZZER_PIN 25
#define LED_PIN 26

// ⚙️ Separate thresholds
int mq2Threshold   = 5000;   // adjust after testing
int mq135Threshold = 50;   // adjust after testing

unsigned long lastSend = 0;
unsigned long interval = 1000; // 1 sec

// 📤 Telegram function
void sendTelegram(String msg) {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;

    String url = "https://api.telegram.org/bot" + botToken +
                 "/sendMessage?chat_id=" + chatID +
                 "&text=" + msg;

    http.begin(url);
    http.GET();
    http.end();
  }
}

void setup() {
  Serial.begin(115200);

  lcd.init();
  lcd.backlight();

  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED_PIN, OUTPUT);

  // WiFi connect
  lcd.print("Connecting WiFi");
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  lcd.clear();
  lcd.print("WiFi Connected");
  delay(2000);
  lcd.clear();

  sendTelegram("System Started ✅");
}

void loop() {
  int mq2 = analogRead(MQ2_PIN);
  int mq135 = analogRead(MQ135_PIN);

  Serial.print("MQ2: ");
  Serial.print(mq2);
  Serial.print(" | MQ135: ");
  Serial.println(mq135);

  // LCD display
  lcd.setCursor(0,0);
  lcd.print("MQ2:");
  lcd.print(mq2);
  lcd.print("   ");

  lcd.setCursor(0,1);
  lcd.print("MQ135:");
  lcd.print(mq135);
  lcd.print(" ");

  String status = "SAFE ✅";

  // 🔍 Individual detection
  if (mq2 > mq2Threshold) {
    status = "MQ2 ALERT 🔥";
  }

  if (mq135 > mq135Threshold) {
    status = "MQ135 ALERT 🌫";
  }

  // Combined alert
  if (mq2 > mq2Threshold || mq135 > mq135Threshold) {
    lcd.setCursor(10,1);
    lcd.print("ALERT");

    digitalWrite(BUZZER_PIN, HIGH);
    digitalWrite(LED_PIN, HIGH);
  } else {
    lcd.setCursor(10,1);
    lcd.print("SAFE ");

    digitalWrite(BUZZER_PIN, LOW);
    digitalWrite(LED_PIN, LOW);
  }

  // 📤 Send Telegram update
  if (millis() - lastSend > interval) {
    String msg = "Gas Update:\n";
    msg += "MQ2: " + String(mq2) + "\n";
    msg += "MQ135: " + String(mq135) + "\n";
    msg += "Status: " + status;

    sendTelegram(msg);
    lastSend = millis();
  }

  delay(1000);
}
