# Tekuteru Servo

TekuteruServo is an advanced servo motor that significantly outperforms standard hobby servos like the SG90, while remaining just as easy to control. 
It supports ±1° angular accuracy, multi-turn positioning (±524,287 rotations), adjustable rotation speeds (up to 600 deg/s), and real-time angle feedback—all while maintaining the same programming control logic, physical dimensions, and wiring compatibility as the SG90.

**Note:** This library is specifically designed for TekuteruServo hardware and is **not compatible** with standard hobby servos (e.g., SG90, MG996R).
The Tekuteru-Servo hardware can be purchased here: [Buy TekuteruServo](https://tekuteru-en.square.site/product/TekuteruServo-full-angle-servo-motor-TekuteruServo-/6YOHLJ6JAV6S326MC53HLNRV?cp=true&sa=true&sbp=false&q=false)


## Features
* **High-Precision Multi-turn Positioning:** Supports ±524,287 full rotations (±188,743,320°) with ±1° accuracy.
* **Intuitive Library Interface:** Dedicated `attach()` and `write()` methods, maintaining compatibility with standard Arduino Servo library programming logic.
* **Adjustable Dynamics:** Controlled rotation speeds (10–600 deg/s) and real-time angle feedback.
* **Drop-in Replacement:** Same wiring, physical dimensions, and logic voltage (3.3V–5V) as the SG90.
* **Volatile Rotation Count:** While the absolute position (0–359°) is preserved, the multi-turn rotation count resets to zero upon power-up.


## Mechanical Specifications
* **Supply Voltage:** 5V
* **Max Speed:** 600 deg/s (0.1s/60deg, approx. 100rpm)
* **Angular Acceleration:** 6,000 deg/s²
* **Stall Torque:** 1.8 kgf·cm
* **Gear Material:** Plastic
* **Weight:** 15 g


## Usage Notes & Limitations

### ⚠ Control & Connectivity
* **Pin Control Limits:** Each I/O pin is designed to control one motor. However, you can broadcast to multiple motors on a single pin using methods that do not require feedback, such as `write()` with `wait=false`, `stop()`, `setZero()`, and `setHold()`.
* **Hardware Interferences:** Using hardware interrupts in your sketch may cause jitter. Additionally, avoid strong magnetic fields and unstable power sources.
* **Serial Pins:** Using pins dedicated to hardware serial communication (such as D0 and D1 on the Arduino Uno) is not recommended for motor control.

### ⚠ Operational Risks
* **Stall Torque & Impacts:** Physically forcing the motor to stop may cause **erratic behavior**. Sustained stall torque or strong impacts beyond specifications will **damage the plastic gears**.
* **49.7-Day Limit:** Malfunctions will occur if the power remains on for more than 49.7 consecutive days due to internal timer overflow.

### ⚠ Physical Handling
* **Cable Care:** Do not pull on the wiring; internal connections are delicate and may break.


## Wiring Guide
This motor follows the standard SG90 wiring convention:

| Wire Color | Function | Connection      |
|------------|----------|-----------------|
| Brown      | GND      | Arduino GND     |
| Red        | VCC      | 5V Power Supply |
| Yellow     | Signal   | Arduino I/O pin |

![Wiring Diagram](wiring.png)


## Class Methods

### `attach(pin)`
Attaches the servo to the specified pin.  

**Arguments:**
- `pin`: `uint8_t`

---

### `write(angle)`
Rotates to the target angle at maximum speed.  
Upon power-up, the initial angle is initialized within the range of 0° to 359°.

**Arguments:**
- `angle`: `int32_t` (Range: `-188,743,320` to `188,743,320`)

---

### `write(angle, speed)`
Rotates to the target angle at a specified speed.
The `speed` argument is an integer between 0 and 255:
- **0**: Stop
- **1**: Minimum speed (10 deg/s)
- **255**: Maximum speed (600 deg/s)

To specify a particular angular velocity (between 10 and 600 deg/s), use the following mapping in your code:
`speed = map(angularVelocity, 10, 600, 1, 255);`

| Speed Value (0–255) | Angular Velocity [deg/s] |
|---------------------|--------------------------|
| 0                   | 0 deg/s (Stop)           |
| 1                   | 10 deg/s                 |
| 127                 | 303 deg/s                |
| 255                 | 600 deg/s (Max speed)    |

**Note on Motion Control:**
This motor utilizes trapezoidal motion profiling, which includes automatic acceleration and deceleration phases for smoother starts and stops.

**Arguments:**
- `angle`: `int32_t`
- `speed`: `uint8_t`

---

### `write(angle, speed, wait)`
Rotates with an option to block execution until the movement is complete. If `true`, the function waits for the move to finish.

**Arguments:**
- `angle`: `int32_t`
- `speed`: `uint8_t`
- `wait`: `bool` (Default: `false`)

---

### `stop()`
Immediately stops the servo at its current position.

---

### `read()`
Returns the current angle in degrees.  
The initial angle at power-on is between 0° and 359°. Values are rounded to the nearest integer.  
Returns `-2,147,483,648` if a communication error occurs.

**Returns:** `int32_t`

---

### `wait()`
Blocks execution until the current movement is finished.

---

### `isMoving()`
Returns `true` if the servo is currently rotating.

**Returns:** `bool`

---

### `setZero()`
Sets the current position as the 0° reference point.  
This zero-point is stored in non-volatile memory (EEPROM/Flash) and persists across power cycles.  
**Note:** Only the absolute position (0-359°) is saved; the rotation count is not preserved.

---

### `setHold(hold)`
Configures the motor's behavior when external force is applied. If set to `true`, the servo will actively maintain its current position (default). If set to `false`, the motor enters a "free move" state, allowing it to be rotated manually by external force; however, please note that angular accuracy may decrease while in this state.

**Arguments:**
- `hold`: `bool`(Default: `true`)


## Code Examples

### 1. Basic Rotation
```
#include <TekuteruServo.h>

TekuteruServo myservo;

void setup() {
  myservo.attach(2);
}

void loop() {
  myservo.write(180);  // Move to 180 degrees
  delay(3000);

  myservo.write(-180); // Move to -180 degrees
  delay(3000);

  myservo.write(540);  // Move to 540 degrees
  delay(3000);
}
```

### 2. Speed control
```
#include <TekuteruServo.h>

TekuteruServo myservo;

int angularVelocity;

void setup() {
  myservo.attach(2);
}

void loop() {
  myservo.write(180, 0);  // No rotation
  delay(3000);

  myservo.write(-180, 100);  // Move to -180 degrees with speed 100
  delay(3000);

  myservo.write(540, 255);  // Move to 540 degrees with speed 255 (max speed)
  delay(3000);


  angularVelocity = 300;
  myservo.write(-180, map(angularVelocity, 10, 600, 0, 255));  // Move to -180 degrees at approximately 300 [deg/s]
  delay(3000);

  angularVelocity = 600;
  myservo.write(540, map(angularVelocity, 10, 600, 0, 255));  // Move to 540 degrees at 600 [deg/s] (max speed)
  delay(3000);
}
```

### 3. Wait for movement to complete
```
#include <TekuteruServo.h>

TekuteruServo myservo;

void setup() {
  myservo.attach(2);
}

void loop() {
  myservo.write(180, 255, true);   // Move to 180 degrees, Wait for completion

  myservo.write(-180, 255, true);  // Move to -180 degrees, Wait for completion

  myservo.write(540, 255);          // Move to 540 degrees
  myservo.wait();                   // Wait until myservo finishes rotating
}
```

### 4. Read the current angle
```
#include <TekuteruServo.h>

TekuteruServo myservo;

long currentAngle;  // Declare it as a long type.

void setup() {
  Serial.begin(9600);  // Start serial communication (Set the serial monitor to 9600 baud.)

  myservo.attach(2);

  currentAngle = myservo.read();  // Read the current angle (0 ≤ angle < 360)
  Serial.println(currentAngle);   // Display on serial monitor
}

void loop() {
  myservo.write(360, 255, true);  // Move to 360 degrees, Wait for completion
  currentAngle = myservo.read();  // Read the current angle (360±1)
  Serial.println(currentAngle);

  myservo.write(0, 255, true);    // Move to 0 degrees, Wait for completion
  currentAngle = myservo.read();  // Read the current angle (0±1)
  Serial.println(currentAngle);

  myservo.write(3600);            // Move to 3600 degrees
  delay(1000);
  currentAngle = myservo.read();  // Read the current angle
  Serial.println(currentAngle);
}
```

### 5. Multiple servos
```
#include <TekuteruServo.h>

TekuteruServo myservo1, myservo2; // There is no software limit on the number of servos

void setup() {
  myservo1.attach(2);
  myservo2.attach(3);
}

void loop() {
  myservo1.write(180);
  myservo2.write(-90);
  myservo1.wait();    // Wait until myservo1 finishes rotating
  myservo2.wait();    // Wait until myservo2 finishes rotating

  myservo1.write(-90, 255, true);

  myservo2.write(180, 255, true);

  myservo1.write(-180, 255);
  myservo2.write(-180, 255, true);
  myservo1.wait();
}
```

### 6. Set setZero
```
#include <TekuteruServo.h>

TekuteruServo myservo;

void setup() {
  Serial.begin(9600);

  myservo.attach(2);

  myservo.setHold(false);  // the servo will not hold its position

  myservo.setZero();      // Set the current angle to 0 degrees (Multiple rotations are not saved)

  Serial.println("setZero successful");
}

void loop() {
}
```


## Support, Disclaimer, and Feedback
* Library Design: The design of this library is inspired by the [VarSpeedServo](https://github.com/netlabtoolkit/VarSpeedServo) library.
* Brand Affiliation: TekuteruServo is not a product of Tower Pro.
* Feedback: I have used translation tools to prepare this documentation. If you find any technical errors or awkward English expressions, we would greatly appreciate your feedback. Please contact us at: abc@gmail.com
