# Project-Week-3-Task-4
Use bluetooth
#include <LiquidCrystal.h>
#include <SoftwareSerial.h>

// === 1. Hardware Pins ===
// Motor A
#define MOTOR_A_EN 3   
#define MOTOR_A_IN1 13 
#define MOTOR_A_IN2 2  

// Motor B
#define MOTOR_B_EN 11  
#define MOTOR_B_IN1 12 
#define MOTOR_B_IN2 1  

// === 2. Bluetooth Pins (Using A4/A5 as SoftwareSerial) ===
SoftwareSerial btSerial(A4, A5); // RX, TX

// LCD Settings
LiquidCrystal lcd(8, 9, 4, 5, 6, 7);

// === 3. Custom Arrow Icons (5x8 Pixels) ===
// 0: Forward Arrow (Up)
byte arrowUp[8] = {
  0b00100,
  0b01110,
  0b10101,
  0b00100,
  0b00100,
  0b00100,
  0b00000,
  0b00000
};

// 1: Backward Arrow (Down)
byte arrowDown[8] = {
  0b00100,
  0b00100,
  0b00100,
  0b10101,
  0b01110,
  0b00100,
  0b00000,
  0b00000
};

// 2: Left Arrow
byte arrowLeft[8] = {
  0b00000,
  0b00100,
  0b01000,
  0b11111,
  0b01000,
  0b00100,
  0b00000,
  0b00000
};

// 3: Right Arrow
byte arrowRight[8] = {
  0b00000,
  0b00100,
  0b00010,
  0b11111,
  0b00010,
  0b00100,
  0b00000,
  0b00000
};

// 4: Stop Icon (Solid Block)
byte iconStop[8] = {
  0b00000,
  0b01110,
  0b11111,
  0b11111,
  0b11111,
  0b01110,
  0b00000,
  0b00000
};

// === Runtime Parameters ===
int moveSpeed = 200;   // 前进速度
int turnSpeed = 220;   // 转向速度 (加大一点防止转不动)
char command = 'S';    // Current command state

void setup() {
  // Initialize LCD
  lcd.begin(16, 2); 
  
  // Register Custom Characters
  lcd.createChar(0, arrowUp);
  lcd.createChar(1, arrowDown);
  lcd.createChar(2, arrowLeft);
  lcd.createChar(3, arrowRight);
  lcd.createChar(4, iconStop);

  lcd.clear();
  lcd.print("BT Mode Ready"); 
  
  // Initialize Bluetooth ONLY
  btSerial.begin(9600); 

  // Setup Motor Pins
  pinMode(MOTOR_A_EN, OUTPUT); pinMode(MOTOR_A_IN1, OUTPUT); pinMode(MOTOR_A_IN2, OUTPUT);
  pinMode(MOTOR_B_EN, OUTPUT); pinMode(MOTOR_B_IN1, OUTPUT); pinMode(MOTOR_B_IN2, OUTPUT);

  // Initial Stop
  stopCar();
}

void loop() {
  // Check if data is available from Bluetooth
  if (btSerial.available() > 0) {
    command = btSerial.read(); 
    
    // Update LCD based on command
    // Format: 
    // Line 1: "  Status Name   "
    // Line 2: "     <<<      " (Arrows)
    
    switch (command) {
      case 'F': // Forward
        moveForward(moveSpeed);
        lcd.clear();
        lcd.setCursor(4, 0); lcd.print("FORWARD");
        lcd.setCursor(6, 1); 
        lcd.write(byte(0)); lcd.write(byte(0)); lcd.write(byte(0)); 
        break;
        
      case 'B': // Backward
        moveBackward(moveSpeed);
        lcd.clear();
        lcd.setCursor(4, 0); lcd.print("REVERSE");
        lcd.setCursor(6, 1); 
        lcd.write(byte(1)); lcd.write(byte(1)); lcd.write(byte(1)); 
        break;
        
      case 'L': // Left
        turnLeft(turnSpeed);
        lcd.clear();
        lcd.setCursor(5, 0); lcd.print("LEFT");
        lcd.setCursor(6, 1); 
        lcd.write(byte(2)); lcd.write(byte(2)); lcd.write(byte(2)); 
        break;
        
      case 'R': // Right
        turnRight(turnSpeed);
        lcd.clear();
        lcd.setCursor(5, 0); lcd.print("RIGHT");
        lcd.setCursor(6, 1); 
        lcd.write(byte(3)); lcd.write(byte(3)); lcd.write(byte(3)); 
        break;
        
      case 'S': // Stop
        stopCar();
        lcd.clear();
        lcd.setCursor(5, 0); lcd.print("STOP");
        lcd.setCursor(6, 1); 
        lcd.write(byte(4)); lcd.write(byte(4)); lcd.write(byte(4)); 
        break;
        
      default:
        break;
    }
  }
}

// === Motor Helper Functions ===

// Forward: Both Motors HIGH/LOW
void moveForward(int speed) {
  digitalWrite(MOTOR_A_IN1, HIGH); digitalWrite(MOTOR_A_IN2, LOW);
  analogWrite(MOTOR_A_EN, speed);
  digitalWrite(MOTOR_B_IN1, HIGH); digitalWrite(MOTOR_B_IN2, LOW);
  analogWrite(MOTOR_B_EN, speed);
}

// Backward: Both Motors LOW/HIGH
void moveBackward(int speed) {
  digitalWrite(MOTOR_A_IN1, LOW); digitalWrite(MOTOR_A_IN2, HIGH);
  analogWrite(MOTOR_A_EN, speed);
  digitalWrite(MOTOR_B_IN1, LOW); digitalWrite(MOTOR_B_IN2, HIGH);
  analogWrite(MOTOR_B_EN, speed);
}

// Turn Left: Motor A Backward, Motor B Forward
void turnLeft(int speed) {
  digitalWrite(MOTOR_A_IN1, LOW); digitalWrite(MOTOR_A_IN2, HIGH);
  analogWrite(MOTOR_A_EN, speed);
  digitalWrite(MOTOR_B_IN1, HIGH); digitalWrite(MOTOR_B_IN2, LOW);
  analogWrite(MOTOR_B_EN, speed);
}

// Turn Right: Motor A Forward, Motor B Backward
void turnRight(int speed) {
  digitalWrite(MOTOR_A_IN1, HIGH); digitalWrite(MOTOR_A_IN2, LOW);
  analogWrite(MOTOR_A_EN, speed);
  digitalWrite(MOTOR_B_IN1, LOW); digitalWrite(MOTOR_B_IN2, HIGH);
  analogWrite(MOTOR_B_EN, speed);
}

// Stop: Coast to stop
void stopCar() {
  digitalWrite(MOTOR_A_IN1, LOW); digitalWrite(MOTOR_A_IN2, LOW);
  analogWrite(MOTOR_A_EN, 0);
  digitalWrite(MOTOR_B_IN1, LOW); digitalWrite(MOTOR_B_IN2, LOW);
  analogWrite(MOTOR_B_EN, 0);
}
