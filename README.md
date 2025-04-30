# Rain_detection_and_automation_system
For the capstone project.
<br>
Author - Sumit Raj
<br>

#include <WiFi.h>
#include <WebServer.h>
#include <ESP32Servo.h>

const char* ssid = "vivo";       // Your Wi-Fi SSID
const char* password = "12345678"; // Your Wi-Fi Password

// Pins
const int rainSensorPin = 34;  // Pin for the rain sensor (Analog)
const int lightPin = 15;       // Pin for controlling LED
const int servoPin = 18;       // Pin for controlling the servo motor

WebServer server(80);  // Web server object
Servo myServo;  // Servo object

int rainValue = 0;  // Variable to store the sensor value

void setup() {
  Serial.begin(115200);  // Start serial communication for debugging

  WiFi.begin(ssid, password);  // Connect to Wi-Fi
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected to Wi-Fi! IP: " + WiFi.localIP().toString());

  pinMode(lightPin, OUTPUT);  // Set light pin as output
  pinMode(rainSensorPin, INPUT);  // Set rain sensor pin as input

  myServo.attach(servoPin);  // Attach the servo motor to the defined pin

  // Web server endpoint for light control
  server.on("/", HTTP_GET, []() {
    server.send(200, "text/html", "<h1>Rain Detection</h1><p>Status: <span id='status'>Checking...</span></p>");
  });

  // Start the server
  server.begin();
}

void loop() {
  // Read rain sensor value
  rainValue = analogRead(rainSensorPin);
  Serial.println("Rain Sensor Value: " + String(rainValue));

  // Control the LED based on the rain sensor
  if (rainValue < 2000) {  // When rain is detected
    digitalWrite(lightPin, HIGH);  // Turn on light
    myServo.write(90);  // Move servo to 90 degrees (adjust this as needed)
    Serial.println("Rain Detected: Servo moved to 90°");
  } else {
    digitalWrite(lightPin, LOW);   // Turn off light
    myServo.write(0);  // Move servo to 0 degrees (adjust this as needed)
    Serial.println("No Rain: Servo moved to 0°");
  }

  // Handle web server requests
  server.handleClient();
}
