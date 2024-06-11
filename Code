#define BLYNK_TEMPLATE_ID "TMPL69zR3FJSa"
#define BLYNK_TEMPLATE_NAME "Smart door notification"
#define BLYNK_AUTH_TOKEN "oBzChq1jYbu1mabyxasfhfvwKC5HiyIE"

#include <SPI.h>
#include <MFRC522.h>
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <ESP32Servo.h>  // Use the ESP32Servo library

// RFID reader pins
#define SS_PIN 21  // SDA
#define RST_PIN 22
#define MOSI_PIN 23
#define MISO_PIN 19
#define SCK_PIN 18

// Buzzer pin
const int buzzerPin = 15;

// Servo pin
const int servoPin = 14;
Servo myServo;

// Predefined RFID tags (replace with your own tag UIDs)
byte authorizedTags[][4] = {
  {0x33, 0xB7, 0x4D, 0xA8},
  {0x85, 0xAD, 0xD2, 0xCF}
};

MFRC522 mfrc522(SS_PIN, RST_PIN); // Create MFRC522 instance

// Your WiFi credentials
char ssid[] = "WIFI_SSID";
char pass[] = "WIFI_PASS";

// Blynk authorization token
char auth[] = BLYNK_AUTH_TOKEN;

void setup() {
  Serial.begin(115200);   // Initialize serial communications with 115200 baud rate
  SPI.begin(SCK_PIN, MISO_PIN, MOSI_PIN, SS_PIN); // Init SPI bus with custom pins
  mfrc522.PCD_Init();   // Init MFRC522
  pinMode(buzzerPin, OUTPUT); // Initialize the buzzer pin as an output
  myServo.attach(servoPin); // Initialize the servo motor
  myServo.write(0); // Ensure the door is closed initially

  // Connect to Wi-Fi
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");

  // Initialize Blynk
  Blynk.begin(auth, ssid, pass);
}

void loop() {
  Blynk.run(); // Run Blynk
  // Look for new cards
  if (!mfrc522.PICC_IsNewCardPresent() || !mfrc522.PICC_ReadCardSerial()) {
    delay(50);
    return;
  }

  // Get the UID of the scanned tag
  byte readTag[4];
  for (byte i = 0; i < 4; i++) {
    readTag[i] = mfrc522.uid.uidByte[i];
  }

  // Check if the tag is authorized
  if (isAuthorized(readTag)) {
    accessApproved();
  } else {
    accessDenied();
  }

  // Halt PICC
  mfrc522.PICC_HaltA();
}

bool isAuthorized(byte tag[]) {
  for (byte i = 0; i < sizeof(authorizedTags) / 4; i++) {
    if (memcmp(tag, authorizedTags[i], 4) == 0) {
      return true;
    }
  }
  return false;
}

void accessApproved() {
  // Buzzer beep for access approved
  digitalWrite(buzzerPin, HIGH);
  delay(200);
  digitalWrite(buzzerPin, LOW);
  delay(200);
  digitalWrite(buzzerPin, HIGH);
  delay(800);
  digitalWrite(buzzerPin, LOW);
  delay(800);
  
  // Open the door (rotate servo to 120 degrees to the right)
  myServo.write(60); // Rotate to 120 degrees to open the door
  Blynk.virtualWrite(V0, 1); // Set the Blynk button to "open"
  
  // Send notification to Blynk immediately
  Blynk.logEvent("security_alert", "Access Approved: Door is opened");
  
  delay(4000); // Keep the door open for 4 seconds
  
  myServo.write(0); // Rotate back to 0 degrees to close the door
  Blynk.virtualWrite(V0, 0); // Set the Blynk button to "close"
}

void accessDenied() {
  // Buzzer beep for access denied
  for (int i = 0; i < 3; i++) {
    digitalWrite(buzzerPin, HIGH);
    delay(100);
    digitalWrite(buzzerPin, LOW);
    delay(100);
  }
  
  // Send notification to Blynk immediately
  Blynk.logEvent("security_alert", "Access Denied: Unauthorized tag");
}

// Function to handle the button press in the Blynk app
BLYNK_WRITE(V0) {
  int pinValue = param.asInt(); // Get the state of the button
  if (pinValue == 1) {
    accessApproved(); // Simulate access approved when the button is pressed
  } else {
    accessDenied(); // Simulate access denied when the button is released
  }
}
