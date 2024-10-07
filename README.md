#include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>
#include <Keypad.h>
#include <LiquidCrystal_I2C.h>

//Wi-­Fi credentials
const char* ssid = "SLT FIBRE";     // replace with your surname and SSID
const char* password_wifi = "00000000";  // replace with your Wi-­Fi password

ESP8266WebServer server(80);  // Web server object

// Define the keypad rows and columns
const int ROW_NUM = 4;  // Four rows
const int COLUMN_NUM = 3; // Three columns

char key_layout[ROW_NUM][COLUMN_NUM] = {
  {'1', '2', '3'},
  {'4', '5', '6'},
  {'7', '8', '9'},
  {'*', '0', '#'}
};

byte pin_rows[ROW_NUM] = {D0, D3, D4, D5};  // ESP8266 pins connected to row pins
byte pin_column[COLUMN_NUM] = {D6, D7, D8}; // ESP8266 pins connected to column pins

Keypad keypad = Keypad(makeKeymap(key_layout), pin_rows, pin_column, ROW_NUM, COLUMN_NUM);
LiquidCrystal_I2C lcd(0x27, 16, 2); // Initialize the LCD (address 0x27)

String password = "1212"; // Default password
String enteredPassword = ""; // Store entered password
int column_cursor = 0; // LCD cursor position
int max_length = 4; // Password length
const int D1_PIN = D1;     // Correct definition of D1 pin

void setup() {
  Serial.begin(115200); // Initialize  Serial Monitor
  pinMode(D1_PIN, OUTPUT);  
  digitalWrite(D1_PIN, LOW); // Initially set the D1 pin low

  lcd.init();               // initialize the LCD
lcd.backlight();         // Turn on the LCD backlight
  lcd.setCursor(0, 0);
  lcd.print("Enter Password:");

  // Connect to Wi-Fi
  WiFi.begin(ssid, password_wifi);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi.");
  }
  Serial.println("Connected to WiFi");
  Serial.print("ESP8266 IP address: ");
  Serial.println(WiFi.localIP());

  // HTTP handle password update
server.on("/update_password", []() {
    if (server.hasArg("password")) {
      password = server.arg("password"); // Update the password
      server.send(200, "text/plain", "Password updated successfully");
    } else {
      server.send(400, "text/plain", "Password argument missing");
    }
  });

  server.begin(); // Start the web server
}

void loop() {
server.handleClient();     // Handle incoming HTTP requests

  char key = keypad.getKey();  // Get keypress from keypad

  if (key) {
    if (key == '*') {  // Clear the screen if '*' is pressed
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("Enter Password:");
enteredPassword = "";  // Reset entered password
      column_cursor = 0;
    } else {
      enteredPassword += key;  // Add key to entered password
      lcd.setCursor(column_cursor, 1);  // Display on second row of the LCD
      lcd.print(key);
column_cursor++;
  // When password reaches max length, check it
  if (enteredPassword.length() == max_length) {
    checkPassword();
  }
}
}
}

// Function to check the password
void checkPassword() {
  delay(500);  // Small delay for readability

if (enteredPassword == password) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Correct Password");
    digitalWrite(D1_PIN, HIGH);  // Turn D1 pin high
    delay(5000);              // Keep high for 5 seconds
digitalWrite(D1_PIN, LOW);   // Turn D1 pin low
    lcd.clear();
    lcd.print("Enter Password:");
  } else {
    countdownRetry();  // Start retry countdown
  }

  // Reset password input after checking
  enteredPassword = "";
  column_cursor = 0;
}

// Function for retry countdown
void countdownRetry() {
for(int i=5; i > 0; i--) {
   lcd.clear();
   lcd.setCursor(0, 0);
   lcd.print("Try Again " + String(i)); // Countdown display
   delay(1000); // Delay for 1 second
 }
 lcd.clear();
 lcd.setCursor(0, 0);
 lcd.print("Enter Password:");
}
