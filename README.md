#include <Servo.h>
#include <Stepper.h>

// Stepper motor setup
#define STEPS 2048
Stepper stepper(STEPS, 8, 10, 9, 11);

// Servo for lid
Servo lidServo;

// Pin definitions
const int lidServoPin = 7;
const int proximityPin = 6;
const int moisturePin = A0;
const int IR_SENSOR = 5;

// Variables
int moistureThreshold = 500;
int currentBin = 0;

void setup() {
  Serial.begin(9600);
  lidServo.attach(lidServoPin);

  pinMode(proximityPin, INPUT);
  pinMode(IR_SENSOR, INPUT);

  stepper.setSpeed(10);
  lidServo.write(0);

  resetToHome(); // Reset at startup
}

void loop() {

    if (digitalRead(proximityPin) == HIGH) {
      Serial.println("Metal Detected");
      rotateToBin(3);
    } else {
      int moisture = analogRead(moisturePin);
      Serial.println("Moisture Value: " + String(moisture));
      if (moisture < moistureThreshold) {
        Serial.println("Wet Waste");
        rotateToBin(2);
      } else {
        Serial.println("Dry Waste");
        rotateToBin(1);
      }
  if (digitalRead(IR_SENSOR) == LOW) {
    Serial.println("Object Detected");

    openLid();
    delay(2000);
    closeLid();
    delay(500);


    }

    delay(1000);  // Short delay after sorting
    resetToHome();  // Always return to home (bin 0)
    delay(4000);    // Wait before next object
  }
}

void openLid() {
  lidServo.write(90);
  Serial.println("Lid Opened");
}

void closeLid() {
  lidServo.write(0);
  Serial.println("Lid Closed");
}

void rotateToBin(int targetBin) {
  int stepsPerBin = STEPS / 3;
  int stepDifference = targetBin - currentBin;

  if (stepDifference != 0) {
    stepper.step(stepDifference * stepsPerBin);
    currentBin = targetBin;
    Serial.println("Rotated to Bin " + String(targetBin));
  } else {
    Serial.println("Already at correct bin.");
  }
}

// ✅ Reset bin position to Bin 0 (Home) after each waste drop
void resetToHome() {
  int stepsPerBin = STEPS / 3;
  int stepDifference = 0 - currentBin;

  if (stepDifference != 0) {
    stepper.step(stepDifference * stepsPerBin);
    currentBin = 0;
    Serial.println("Returned to Home (Bin 0)");
  } else {
    Serial.println("Already at Home (Bin 0)");
  }
}
