#include <Servo.h>
#include <SoftwareSerial.h>

#define TEMP_PIN A0
#define LDR_PIN A1
#define SWITCH_PIN 2
#define LED_PIN 3
#define SERVO_PIN 5
#define WIFI_TX 6
#define WIFI_RX 7

Servo servo;
SoftwareSerial wifi(WIFI_RX, WIFI_TX);

float temperature;
int lightValue;
bool autoMode = true;
int servoAngle = 0;

void setup() {
  Serial.begin(9600);
  wifi.begin(9600);
  servo.attach(SERVO_PIN);
  pinMode(LED_PIN, OUTPUT);
  pinMode(SWITCH_PIN, INPUT);
  servo.write(0);
}

void loop() {
  // 센서값 읽기
  temperature = analogRead(TEMP_PIN) * (5.0 / 1023.0) * 100; // LM35 변환식
  lightValue = analogRead(LDR_PIN);
  autoMode = digitalRead(SWITCH_PIN) == LOW;

  if (autoMode) {
    // 자동 모드
    if (temperature >= 30) {
      servoAngle = 90; // 가림막 닫힘
      analogWrite(LED_PIN, 255); // 과열: LED 빠르게 점멸
      blinkLED(100);
    } 
    else if (lightValue <= 350) {
      servoAngle = 0;
      blinkLED(400); // 조도 낮음: 천천히 점멸
    } 
    else {
      servoAngle = 0;
      digitalWrite(LED_PIN, LOW); // 정상: LED 꺼짐
    }
    servo.write(servoAngle);
  } 
  else {
    // 수동 모드
    manualControl();
  }

  // 시리얼 및 WiFi 전송
  Serial.print("T=");
  Serial.print(temperature);
  Serial.print(",L=");
  Serial.print(lightValue);
  Serial.print(",mode=");
  Serial.print(autoMode ? "auto" : "manual");
  Serial.print(",servo=");
  Serial.println(servoAngle);

  wifi.print("GET /log?T=");
  wifi.print(temperature);
  wifi.print("&L=");
  wifi.print(lightValue);
  wifi.print("&mode=");
  wifi.print(autoMode ? "auto" : "manual");
  wifi.print("&servo=");
  wifi.println(servoAngle);

  delay(1000);
}

// LED 점멸 함수
void blinkLED(int interval) {
  digitalWrite(LED_PIN, HIGH);
  delay(interval);
  digitalWrite(LED_PIN, LOW);
  delay(interval);
}

// 수동 제어 함수 (예시)
void manualControl() {
  static int angle = 0;
  if (Serial.available()) {
    char cmd = Serial.read();
    if (cmd == 'o') angle = 0;
    if (cmd == 'c') angle = 90;
    servo.write(angle);
  }
  digitalWrite(LED_PIN, HIGH); // 수동 모드 표시
}
