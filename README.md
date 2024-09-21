#define BLYNK_TEMPLATE_ID           "TMPL6meyoSuST"
#define BLYNK_TEMPLATE_NAME         "Quickstart Template"
#define BLYNK_AUTH_TOKEN  "dMRKmaj05yuzcZl7NqijOzIx4KfkIU7z"

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

// Define pins
#define MOTOR_PIN D1
#define PUMP_PIN D2
#define FAN_PIN D3
#define DHT_PIN D4  // Use a digital pin for DHT22

// Line Token
String LINE_TOKEN = "XVbUAzKkSWUxRLGEeY5JZgp1angeV89jNB0mMyG0OFB";

char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "minmin";
char pass[] = "0617924890";

DHT dht(DHT_PIN, DHT22);  // Initialize DHT22 sensor
BlynkTimer timer;

void sendToLine(String message) {
  WiFiClient client;
  if (!client.connect("notify-api.line.me", 443)) {
    return;
  }

  String query = "message=" + message;
  String header = "POST /api/notify HTTP/1.1\r\n";
  header += "Host: notify-api.line.me\r\n";
  header += "Authorization: Bearer " + LINE_TOKEN + "\r\n";
  header += "Content-Type: application/x-www-form-urlencoded\r\n";
  header += "Content-Length: " + String(query.length()) + "\r\n\r\n";

  client.print(header + query);
  delay(500); // Ensure the message is sent
}

void setup() {
  pinMode(MOTOR_PIN, OUTPUT);
  pinMode(PUMP_PIN, OUTPUT);
  pinMode(FAN_PIN, OUTPUT);

  digitalWrite(MOTOR_PIN, LOW);
  digitalWrite(PUMP_PIN, LOW);
  digitalWrite(FAN_PIN, LOW);

  Serial.begin(9600);
  Blynk.begin(auth, ssid, pass);
  dht.begin();  // Initialize DHT sensor

  timer.setInterval(10000L, checkSoilMoisture);  // Check every 10 seconds
}

void loop() {
  Blynk.run();
  timer.run();
}

void checkSoilMoisture() {
  float humidity = dht.readHumidity();  // Read humidity from DHT22

  if (isnan(humidity)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  // Start operations
  digitalWrite(MOTOR_PIN, HIGH); // Turn on motor
  timer.setTimeout(60000L, []() { digitalWrite(MOTOR_PIN, LOW); }); // Turn off motor after 1 minute

  digitalWrite(PUMP_PIN, HIGH);  // Turn on pump
  timer.setTimeout(30000L, []() { digitalWrite(PUMP_PIN, LOW); }); // Turn off pump after 30 seconds

  // Check humidity level
  if (humidity > 10) {
    digitalWrite(FAN_PIN, HIGH); // Turn on fan
  } else {
    digitalWrite(FAN_PIN, LOW);  // Turn off fan
    sendToLine("ทำงานเสร็จสิ้น");
    timer.setTimeout(2000L, []() { ESP.deepSleep(0); }); // Sleep after 2 seconds delay
  }
}
