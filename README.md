# TekuteruServo (Serial Servo)

TekuteruServo is a high-performance serial servo motor designed as a drop-in replacement for standard hobby servos like the SG90. 

While maintaining the same physical dimensions, wiring, and core programming logic as the SG90, it adds advanced features including multi-turn positioning (±5.96 million rotations), ±1° angular accuracy, adjustable speeds up to 600 deg/s, and real-time position feedback.

**Note:** This library is specifically designed for TekuteruServo hardware and is **not compatible** with standard PWM hobby servos (e.g., SG90, MG996R).  
The TekuteruServo hardware can be purchased here: [Buy TekuteruServo](https://tekuteru.handcrafted.jp/items/121327019)


## Features
* **High-Precision Multi-turn Positioning:** Supports ±5,965,232 full rotations (-2,147,483,647° to +2,147,483,648°) with ±1° accuracy.
* **Familiar Interface:** Includes `attach()` and `write()` methods, maintaining compatibility with the standard Arduino Servo library's logic.
* **Adjustable Dynamics:** Controlled rotation speeds (10–600 deg/s) and real-time angle feedback.
* **Seamless Integration:** Uses the same wiring, form factor, and logic voltage (3.3V–5V) as the SG90.
* **Volatile Rotation Count:** While the absolute position (0–359°) is preserved, the multi-turn rotation count resets to zero upon power-up.


## Mechanical Specifications
* **Supply Voltage:** 5V
* **Logic Voltage:** 3.3V - 5V
* **Max Speed:** 600 deg/s (approx. 0.1s/60° or 100 rpm)
* **Angular Acceleration:** 5,000 deg/s²
* **Stall Torque:** 1.8 kgf·cm
* **Gear Material:** Plastic
* **Weight:** 15 g
* **Cable Length:** 20 cm


## Usage Notes & Limitations

### ⚠ Control & Connectivity
* **Pin Assignment:** Each I/O pin is designed to control one motor. However, multiple motors can be controlled via a single pin for "broadcast" commands that do not require feedback, such as `write()` (with `wait=false`), `stop()`, `setZero()`, and `setHold()`.
* **Hardware Interference:** Using hardware interrupts in your sketch may cause jitter. Avoid using the motor near strong magnetic fields or with unstable power sources.

### ⚠ Operational Risks
* **Stall & Impact:** Physically forcing the motor to stop may lead to **erratic behavior**. Exceeding stall torque or applying strong impacts will **damage the plastic gears**.
* **Power Stability:** Ensure a stable power supply to prevent unstable or erratic motor behavior.

### ⚠ Physical Handling
* **Cable Care:** Do not pull on the wiring; internal connections are delicate and prone to breaking if mishandled.


## Wiring Guide
TekuteruServo follows the standard SG90 wiring convention:

| Wire Color | Function | Connection      |
|------------|----------|-----------------|
| Brown      | GND      | Arduino GND     |
| Red        | VCC      | 5V Power Supply |
| Yellow     | Signal   | Arduino I/O pin |

![Wiring Diagram](wiring.png)


## Class Methods

### `attach(pin)`
Attaches the servo to the specified pin.
- **`pin`**: `uint8_t`

---

### `write(angle)`
Rotates to the target angle at maximum speed (600 deg/s).  
Upon power-up, the initial position is initialized within the 0° to 359° range.
- **`angle`**: `int32_t` (Range: `-2,147,483,648` to `2,147,483,647`)

---

### `write(angle, speed)`
Rotates to the target angle at a specified speed (0–255).
- **0**: Stop
- **1**: Minimum speed (10 deg/s)
- **255**: Maximum speed (600 deg/s)

To map a specific angular velocity (10–600 deg/s) to the speed argument:  
`uint8_t speed = map(angularVelocity, 10, 600, 1, 255);`

| Speed Value (0–255) | Angular Velocity [deg/s] |
|---------------------|--------------------------|
| 0                   | 0 (Stop)                 |
| 1                   | 10                       |
| 127                 | 303                      |
| 255                 | 600 (Max)                |

**Note on Motion Control:** Uses trapezoidal motion profiling with automatic acceleration/deceleration for smooth movement.

---

### `write(angle, speed, wait)`
Rotates to the target angle. If `wait` is `true`, execution blocks until the motor reaches within ±1° of the target.
- **`angle`**: `int32_t`
- **`speed`**: `uint8_t`
- **`wait`**: `bool`

---

### `stop()`
Immediately stops the servo at its current position.

---

### `read()`
Returns the current angle in degrees (rounded to the nearest integer).  
Returns `-2,147,483,648` in case of a communication error.
- **Returns**: `int32_t`

---

### `wait()`
Blocks execution until the current movement is completed.

---

### `isMoving()`
Returns `true` if the servo is currently rotating. Returns `false` if stopped or if a communication error occurs.
- **Returns**: `bool`

---

### `setZero()`
Sets the current absolute position (0–359°) as the 0° reference point. This is saved to non-volatile memory (EEPROM/Flash) and persists after power cycles. Ongoing rotations will stop when this is called.  
**Note:** Only the absolute angle is saved; the rotation count is reset.

---

### `setHold(hold)`
Configures holding behavior.  
- **`true` (Default)**: Actively maintains its position against external force.
- **`false`**: "Free move" state; allows manual rotation, though angular accuracy may decrease.


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
**Note:** When operating multiple servos simultaneously, using an external power supply is highly recommended to ensure stable operation and prevent voltage drops.
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

### 6. Set Zero
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


## Support & Feedback
* **Library Design:** Inspired by the [VarSpeedServo](https://github.com/netlabtoolkit/VarSpeedServo) library.
* **Disclaimer:** TekuteruServo is an independent product and is not affiliated with Tower Pro.
* **Feedback:** This documentation was prepared with the help of translation tools. If you encounter any technical errors or have suggestions for improving the English expressions, please contact us at: abc@gmail.com
