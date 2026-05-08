#include <Servo.h>

const int thrustPin = 9;
const int steerPin = 10;

bool isMovingBackward = false;

// BASE CALIBRATION - Kept your working values
const int NEUTRAL_SPEED = 90;
const int MAX_F_VAL = 135; // Your "Backward" logic speed
const int MAX_R_VAL = 20;  // Your "Forward" logic speed

const int STEER_CENTER = 90;
const int STEER_LEFT   = 135;
const int STEER_RIGHT  = 45;

Servo thrustESC;
Servo steeringServo;

void setup() {
  Serial.begin(9600);
  thrustESC.attach(thrustPin);
  steeringServo.attach(steerPin);

  // Arming Sequence
  thrustESC.write(NEUTRAL_SPEED);
  steeringServo.write(STEER_CENTER);
  delay(3000);
  Serial.println("READY: Use 1-5 with f, b, l, r (e.g., 3fl or 5b)");
}

void loop() {
  if (Serial.available() > 0) {
    String command = Serial.readStringUntil('\n');
    command.trim();
    command.toLowerCase();

    if (command.length() == 0) return;

    // --- STEP 1: DETECT SPEED (1 to 5) ---
    int speedLevel = 5; // Default to max if no number is sent
    if (command.indexOf('1') >= 0) speedLevel = 1;
    else if (command.indexOf('2') >= 0) speedLevel = 2;
    else if (command.indexOf('3') >= 0) speedLevel = 3;
    else if (command.indexOf('4') >= 0) speedLevel = 4;
    else if (command.indexOf('5') >= 0) speedLevel = 5;

    // Calculate actual power based on the 1-5 level
    // map(value, low_in, high_in, low_out, high_out)
    int calcF = map(speedLevel, 1, 5, 100, MAX_F_VAL); // 100 is a slow crawl
    int calcR = map(speedLevel, 1, 5, 63, MAX_R_VAL);  // 80 is a slow crawl

    // --- STEP 2: STEERING ---
    if (command.indexOf('l') >= 0) {
      steeringServo.write(STEER_LEFT);
    } else if (command.indexOf('r') >= 0) {
      steeringServo.write(STEER_RIGHT);
    } else {
      steeringServo.write(STEER_CENTER);
    }

// ... inside void loop() ...

    // --- STEP 3: THRUST ---
    if (command == "s") {
      thrustESC.write(NEUTRAL_SPEED);
      steeringServo.write(STEER_CENTER);
      isMovingBackward = false; // Reset when stopping
    }
    else if (command.indexOf('f') >= 0) {
      thrustESC.write(calcF);
      isMovingBackward = false; // Reset when switching to forward
    }
    else if (command.indexOf('b') >= 0) {
      if (!isMovingBackward) {
        // ONLY Double Tap if we aren't already going backward
        thrustESC.write(calcR);
        delay(400);
        thrustESC.write(NEUTRAL_SPEED);
        delay(400);
        thrustESC.write(calcR);
        isMovingBackward = true; // Mark that we are now in reverse mode
      } else {
        // Just update the speed without the delays if already in reverse
        thrustESC.write(calcR);
      }
    }
  }
