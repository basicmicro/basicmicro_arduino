# Basicmicro Arduino Library

[![Arduino Library Manager](https://img.shields.io/badge/Arduino%20Library%20Manager-Available-green.svg)](https://www.arduino.cc/reference/en/libraries/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

An Arduino library for controlling Basicmicro motor controllers with comprehensive support for all controller features including duty cycle control, speed control, position control, encoder feedback, PID parameter configuration, and more.

## Overview

This library provides a complete interface for communicating with Basicmicro motor controllers using serial communication. It supports the full command set for controlling motors, reading sensors, and configuring controller parameters.

## Supported Hardware

This library is compatible with:

- All Basicmicro motor controllers with serial interface
- Arduino boards with hardware or software serial capabilities
- ESP8266, ESP32, and other Arduino-compatible boards

## Installation

### Using the Arduino Library Manager (Recommended)

1. In the Arduino IDE, go to **Sketch > Include Library > Manage Libraries**
2. Search for "Basicmicro"
3. Click "Install"

### Manual Installation

1. Download the latest release from the [GitHub repository](https://github.com/basicmicro/BasicmicroArduino)
2. In the Arduino IDE, go to **Sketch > Include Library > Add .ZIP Library**
3. Select the downloaded zip file

## Hardware Connection

Connect your Arduino to the Basicmicro controller using the following connections:

- Arduino TX pin → Controller RX pin
- Arduino RX pin → Controller TX pin
- Common ground between Arduino and controller

For hardware serial:
```cpp
#include <Basicmicro.h>
Basicmicro roboclaw(&Serial1, 10000); // Using Serial1, 10ms timeout
```

For software serial (AVR-based boards only):
```cpp
#include <Basicmicro.h>
#include <SoftwareSerial.h>
SoftwareSerial serial(10, 11); // RX, TX
Basicmicro roboclaw(&serial, 10000); // Using SoftwareSerial, 10ms timeout
```

## Basic Usage

### Initialize the Controller

```cpp
#include <Basicmicro.h>

#define ADDRESS 0x80 // Default address of the controller

Basicmicro roboclaw(&Serial1, 10000);

void setup() {
  Serial.begin(115200);
  roboclaw.begin(38400); // Set the baud rate for communication with the controller
}
```

### Motor Control Methods

```cpp
// Basic duty cycle control (-32768 to +32767)
roboclaw.DutyM1(ADDRESS, 16384);      // 50% duty cycle forward
roboclaw.DutyM2(ADDRESS, -16384);     // 50% duty cycle backward
roboclaw.DutyM1M2(ADDRESS, 8192, 8192); // Both motors at 25% forward

// Speed control using encoders
roboclaw.SpeedM1(ADDRESS, 1000);      // Run at 1000 encoder counts per second
roboclaw.SpeedAccelM1(ADDRESS, 500, 1000); // Accelerate to 1000 counts/sec at 500 counts/sec²

// Position control
roboclaw.SpeedAccelDeccelPositionM1(ADDRESS, 500, 1000, 500, 10000, 0); // Move to position 10000

// Stop motors
roboclaw.DutyM1(ADDRESS, 0);
roboclaw.DutyM2(ADDRESS, 0);
```

### Reading Encoder and Status Information

```cpp
// Read encoder values
uint32_t enc1 = roboclaw.ReadEncM1(ADDRESS);
uint32_t enc2 = roboclaw.ReadEncM2(ADDRESS);

// Read speed values
uint8_t status;
bool valid;
uint32_t speed1 = roboclaw.ReadSpeedM1(ADDRESS, &status, &valid);
if (valid) {
  Serial.print("Motor 1 Speed: ");
  Serial.println(speed1);
}

// Read current values
int16_t current1, current2;
if (roboclaw.ReadCurrents(ADDRESS, current1, current2)) {
  Serial.print("Motor 1 Current: ");
  Serial.print(current1);
  Serial.print("mA, Motor 2 Current: ");
  Serial.print(current2);
  Serial.println("mA");
}

// Read battery voltage
uint16_t voltage = roboclaw.ReadMainBatteryVoltage(ADDRESS, &valid);
if (valid) {
  float volts = voltage / 10.0; // Convert to volts
  Serial.print("Battery Voltage: ");
  Serial.print(volts);
  Serial.println("V");
}
```

### Configuration Settings

```cpp
// Set PID parameters for velocity control
roboclaw.SetM1VelocityPID(ADDRESS, 1.0, 0.5, 0.25, 12000);

// Set current limits
roboclaw.SetM1MaxCurrent(ADDRESS, 5000, 2500); // 5A max, 2.5A min

// Save settings to non-volatile memory
roboclaw.WriteNVM(ADDRESS);
```

## Advanced Example

Here's a more complete example showing movement with encoder feedback:

```cpp
#include <Basicmicro.h>

#define ADDRESS 0x80
Basicmicro roboclaw(&Serial1, 10000);

void setup() {
  Serial.begin(115200);
  Serial1.begin(38400);
  
  // Set PID parameters for velocity control
  roboclaw.SetM1VelocityPID(ADDRESS, 1.0, 0.5, 0.25, 12000);
  
  // Reset encoder counters
  roboclaw.ResetEncoders(ADDRESS);
}

void loop() {
  // Move forward for 5 seconds
  roboclaw.SpeedAccelM1(ADDRESS, 500, 2000);
  delay(5000);
  
  // Read encoder position
  uint8_t status;
  bool valid;
  uint32_t position = roboclaw.ReadEncM1(ADDRESS, &status, &valid);
  if (valid) {
    Serial.print("Encoder position: ");
    Serial.println(position);
  }
  
  // Stop motor
  roboclaw.SpeedAccelM1(ADDRESS, 500, 0);
  delay(1000);
  
  // Return to start position
  roboclaw.SpeedAccelDeccelPositionM1(ADDRESS, 500, 2000, 500, 0, 0);
  delay(5000);
}
```

## Complete Documentation

The library provides many more functions for advanced control and configuration. See the header file or refer to the examples folder for more details on specific functions.

For complete documentation of all commands and protocols, please refer to the [official Basicmicro documentation](https://www.basicmicro.com/).

## Troubleshooting

- **No communication with controller**: Verify wiring, check baudrate settings, and ensure power is connected
- **Motors don't move**: Check error codes with `ReadError()`, verify voltage with `ReadMainBatteryVoltage()`
- **Erratic movement**: Adjust PID parameters for smoother control

## Contributing

Contributions to improve the library are welcome. Please submit issues and pull requests on the GitHub repository.

## License

This library is released under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## Supported Hardware

This library works with all current Basicmicro motor controllers:

- **[RoboClaw Motor Controllers](https://www.basicmicro.com/motor-controller)** — Dual-channel brushed DC, 2×7A through 2×300A, up to 60VDC
- **[MCP Industrial Motor Controllers](https://www.basicmicro.com/motor-controller-industrial)** — 30A to 60A continuous at 60VDC, CAN bus, RS-232, MCL scripting

## Resources

- **[Motion Studio](https://www.basicmicro.com/motor-controller-downloads)** — Free configuration and PID tuning software (Windows)
- **[Datasheets, User Manuals & CAD](https://www.basicmicro.com/motor-controller-downloads)** — Datasheets, user manuals, CAD models and drawings
- **[Tutorials & Guides](https://resources.basicmicro.com/)** — Articles, guides, and news
- **[Contact Support](https://www.basicmicro.com/Contact_ep_7.html)** — Technical support and sales inquiries
