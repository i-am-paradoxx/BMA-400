# Persistent Tilt Angle Monitoring System Using ESP32-C6 and BMA400

<div align="center">

**Roll & Pitch Measurement with Power-Off Data Retention and External Recalibration Functionality**

**Real-time tilt monitoring with EEPROM persistence and 5-second recalibration button**

</div>

---

## Table of Contents

- [Overview](#Overview)
- [Hardware Requirements](#hardware-requirements)
- [Wiring Diagram](#wiring-diagram)
- [Software Setup](#software-setup)
- [Installation Guide](#installation-guide)
- [Usage](#usage)
- [Calibration](#calibration)
- [Troubleshooting](#troubleshooting)
- [Features](#features)

---

## Overview

This project develops a persistent tilt angle monitoring system using ESP32-C6 and BMA400 accelerometer over I2C protocol. The system calculates and displays roll and pitch angles in real-time, with data persistence across power cycles through EEPROM storage and external recalibration functionality.

### Project Objective

The system aims to:
- Monitor tilt angles (roll & pitch) continuously using BMA400 sensor
- Save last known angles in EEPROM during power-off for data retention
- Restore saved angles on power-on for continuity across power cycles
- Provide external recalibration via button (5-second hold) to set new zero reference
- Ensure reliable data acquisition with filtering and power-efficient design
- Activate LED indicator when tilt exceeds 5° threshold

### Key Capabilities

- **Real-time Monitoring**: Continuous roll and pitch angle calculation
- **Data Persistence**: EEPROM storage of angles during power-off
- **Power-On Restoration**: Automatic recovery of last known angles
- **External Recalibration**: 5-second button hold for new zero reference
- **Threshold Detection**: LED activation beyond ±5° tilt
- **Low-pass Filtering**: Noise reduction for stable readings
- **Serial Monitoring**: Real-time angle display via Serial Monitor

---

## Hardware Requirements

### Components List

| Component | Quantity | Description |
|-----------|----------|-------------|
| **ESP32-C6 Development Board** | 1 | RISC-V + Wi-Fi 6 microcontroller (up to 160 MHz) |
| **BMA400 Accelerometer** | 2 | Ultra-low power 3-axis MEMS accelerometer |
| **LED** | 1 | Visual indicator for tilt threshold |
| **Resistor** | 1 | 220Ω - 330Ω for LED current limiting |
| **Push Button** | 1 | External recalibration button |
| **Resistor** | 1 | 3.3kΩ pull-up resistor for button |
| **Power Supply** | 1 | 3.3V power source |
| **Connectors** | Several | For secure connections |
| **Breadboard/PCB** | 1 | For prototyping connections |

### ESP32-C6 Specifications

- **Processor**: Single-core 32-bit RISC-V CPU (up to 160 MHz)
- **Co-processor**: LP-Core for ultra-low power operations
- **RAM**: ~512 KB SRAM
- **Flash Memory**: External, typically 4-8 MB
- **Wi-Fi**: 802.11 b/g/n/ax (Wi-Fi 6) built-in
- **Bluetooth**: Bluetooth 5.0 LE and Zigbee/Thread support
- **GPIO Pins**: 30 configurable pins
- **Analog Input**: 12-bit ADC (up to 7 channels)
- **Operating Voltage**: 3.3V
- **Security**: Hardware security features with secure boot

### BMA400 Specifications

- **Sensor Type**: 3-axis MEMS accelerometer
- **Operating Voltage**: 1.7V to 3.6V
- **Interfaces**: I²C and SPI
- **Measurement Ranges**: ±2g, ±4g, ±8g, ±16g
- **Power Consumption**: As low as 14 µA in low-power mode
- **Output Data Rate**: 12.5 Hz to 800 Hz
- **FIFO Buffer**: 1 KB for efficient data handling
- **Embedded Functions**: Step counting, tap detection, motion recognition

---

## Wiring Diagram
| ESP32-C6   | BMA400 #1         | BMA400 #2         | LED                | Button                        |
|------------|-------------------|-------------------|--------------------|-------------------------------|
| GPIO 22    | SDA               | SDA               |                    |                               |
| GPIO 21    | SCL               | SCL               |                    |                               |
| 3.3V       | VDD               | VDD               |                    |                               |
| GND        | GND               | GND               |                    |                               |
| GPIO 2     |                   |                   | [220Ω] -> LED -> GND |                               |
| GPIO 25    |                   |                   |                    | Button -> [3.3kΩ] -> 3.3V     |
| GND        |                   |                   |                    | Button (other terminal)        |

### Detailed Pin Connections

#### ESP32-C6 to BMA400 Sensors (I2C Bus)
- **ESP32-C6 GPIO 22** → **BMA400 SDA** (I2C Data Line)
- **ESP32-C6 GPIO 21** → **BMA400 SCL** (I2C Clock Line)
- **ESP32-C6 3.3V** → **BMA400 VDD** (Power Supply)
- **ESP32-C6 GND** → **BMA400 GND** (Ground)

#### ESP32-C6 to LED Indicator
- **ESP32-C6 GPIO 2** → **220Ω Resistor** → **LED Anode (+)**
- **LED Cathode (-)** → **ESP32-C6 GND**

#### ESP32-C6 to Recalibration Button
- **ESP32-C6 GPIO 25** → **Button Terminal 1**
- **Button Terminal 2** → **3.3kΩ Resistor** → **ESP32-C6 3.3V**
- **ESP32-C6 GND** → **Button Terminal 2**

### BMA400 Pin Configuration

| Pin Name | Function | Connection |
|----------|----------|------------|
| **VDD** | Power supply input (1.7V - 3.6V) | ESP32-C6 3.3V |
| **GND** | Ground | ESP32-C6 GND |
| **SDA** | I2C data line | ESP32-C6 GPIO 22 |
| **SCL** | I2C clock line | ESP32-C6 GPIO 21 |
| **CS** | Chip select for SPI (tie HIGH for I²C) | Not Connected |
| **SDO** | I2C address selector (0x14 or 0x15) | Not Connected |
| **INT1** | Interrupt pin for motion detection | Not Connected |
| **INT2** | Optional second interrupt pin | Not Connected |

---

## Software Setup

### Arduino IDE Configuration

#### 1. Install ESP32-C6 Board Support

1. Open **Arduino IDE**
2. Go to **File** → **Preferences**
3. Add this URL to **Additional Board Manager URLs**:
   ```
   https://dl.espressif.com/dl/package_esp32_index.json
   ```
4. Go to **Tools** → **Board** → **Boards Manager**
5. Search for **"ESP32"** and install **"ESP32 by Espressif Systems"** (version 3.0.0 or later for C6 support)

#### 2. Install Required Libraries

Go to **Tools** → **Manage Libraries** and install:

| Library | Author | Purpose |
|---------|--------|---------|
| **SparkFun BMA400 Arduino Library** | SparkFun Electronics | BMA400 sensor communication |
| **Preferences** | Espressif | EEPROM data persistence |
| **Wire** | Arduino | I2C communication protocol |

#### 3. Board Configuration

1. Go to **Tools** → **Board** → **ESP32 Arduino**
2. Select your ESP32 variant (e.g., **"ESP32C6 Dev Module"**)
3. Configure settings:
   - **Upload Speed**: 921600
   - **CPU Frequency**: 160MHz
   - **Flash Frequency**: 80MHz
   - **Flash Mode**: QIO
   - **Flash Size**: 4MB (32Mb)
   - **Partition Scheme**: Default 4MB with spiffs
   - **Core Debug Level**: None
   - **Port**: Select your ESP32-C6 COM port

---

## Installation Guide

### Step 1: Hardware Assembly

1. **Connect both BMA400 sensors** to ESP32-C6 using shared I2C bus (GPIO 21, 22)
2. **Connect LED** to GPIO 2 with current limiting resistor
3. **Connect Recalibration Button** to GPIO 25 with 3.3kΩ pull-up resistor
4. **Verify power connections** - all components at 3.3V
5. **Double-check all connections** before powering on

### Step 2: Upload Code

1. **Download** the `bma.cpp` file or copy the code
2. **Open** Arduino IDE and paste the code
3. **Verify** all libraries are installed
4. **Select** correct board and port
5. **Upload** the code to ESP32-C6

### Step 3: Initial Setup and Data Persistence

1. **Open Serial Monitor** (Tools → Serial Monitor)
2. **Set baud rate** to **115200**
3. **Reset ESP32-C6** - you should see:
   ```
   BMA400 sensors connected.
   Restoring last known angles from EEPROM...
   Roll: [saved_value]°, Pitch: [saved_value]°
   System ready for tilt monitoring.
   ```
4. **Verify Data Persistence**: Power off and on - angles should be restored

---

## Usage

### Normal Operation with Data Persistence

1. **Power on** the ESP32-C6
2. **System automatically restores** last known angles from EEPROM
3. **Wait 5 seconds** for sensor initialization
4. **Monitor** serial output for real-time tilt values:
   ```
   Real-time Tilt Monitoring:
   Roll: 2.34°, Pitch: -1.87°
   Last saved: Roll: 2.30°, Pitch: -1.85°
   Status: LEVEL (within ±5°)
   ------------------------
   ```
5. **LED behavior**:
   - **OFF**: Device level (within ±5°)
   - **ON**: Device tilted (beyond ±5°)

### Data Persistence Features

- **Automatic Saving**: Angles saved to EEPROM every update cycle
- **Power-Off Retention**: Data persists through power cycles
- **Instant Restoration**: Last known angles restored on boot
- **Continuous Tracking**: Seamless angle monitoring across resets

### External Recalibration Process

1. **Position device** to desired zero reference orientation
2. **Press and hold** recalibration button on GPIO 25
3. **Hold for 5 seconds** until confirmation message appears:
   ```
   Recalibration initiated...
   New zero reference set.
   Current orientation saved as level position.
   Calibration complete.
   ```
4. **Release button** - new reference angles saved to EEPROM

### Serial Output Format

```
Real-time Tilt Monitoring:
Roll: [current_value]°, Pitch: [current_value]°
Last saved: Roll: [eeprom_value]°, Pitch: [eeprom_value]°
Status: [LEVEL/TILTED] (threshold: ±5°)
EEPROM: [SAVING/RESTORED]
------------------------
```

**Output Parameters:**
- **Roll**: Left/Right tilt (positive = right tilt)
- **Pitch**: Forward/Backward tilt (positive = forward tilt)  
- **Last saved**: Previous angles stored in EEPROM
- **Status**: Current tilt state relative to 5° threshold
- **EEPROM**: Data persistence operation status
- **Update Rate**: 50Hz (20ms intervals)

---

## Working Principle & System Architecture

### Tilt Detection Method

The system utilizes trigonometric calculations on 3-axis accelerometer data to determine spatial orientation:

```cpp
// Roll calculation (rotation around X-axis)
roll = atan2(accelY, sqrt(accelX² + accelZ²)) × (180/π)

// Pitch calculation (rotation around Y-axis)  
pitch = atan2(-accelX, sqrt(accelY² + accelZ²)) × (180/π)
```

**Note**: Yaw cannot be accurately determined using accelerometer alone. Yaw involves rotation around the gravity vector, which doesn't change gravity readings. Yaw sensing requires gyroscope integration.

### Data Persistence Architecture

1. **Real-time Processing**: Continuous angle calculation from dual BMA400 sensors
2. **EEPROM Storage**: Automatic saving of current angles using Preferences library
3. **Power-Off Retention**: Data persists in non-volatile memory during power cycles
4. **Boot Restoration**: Automatic recovery of last known angles on startup
5. **Threshold Monitoring**: LED activation when tilt exceeds ±5° in any direction

### Low-Pass Filtering

Applied to reduce noise and vibration effects:
```cpp
filtered_angle = α × new_reading + (1-α) × previous_filtered
// Where α = 0.05 (configurable smoothing factor)
```

### Recalibration Functionality (GPIO 25)

**Purpose**: Manual sensor recalibration for new reference positions

**Process**:
1. **Button Detection**: 5-second hold on GPIO 25 triggers recalibration
2. **Reference Reset**: Current orientation becomes new zero position  
3. **Offset Calculation**: New calibration offsets computed and stored
4. **EEPROM Update**: Calibration data saved to non-volatile memory
5. **System Reset**: Immediate application of new reference values

### Calibration Best Practices

- Use a **stable, level surface** (spirit level recommended)
- **Avoid vibrations** during recalibration process
- **Room temperature** calibration for optimal results
- **Recalibrate** when moving to different altitude/temperature
- **Never calibrate** on soft surfaces or moving platforms
- **Verify calibration** using serial monitor angle readings

---

## Configuration Options

### Adjustable Parameters

Edit these values in the code to customize system behavior:

```cpp
// Tilt threshold (degrees)
if (abs(filteredRoll) > 5 || abs(filteredPitch) > 5) {
    // Change '5' to desired threshold (1-45 degrees)
}

// Low-pass filter smoothing factor
float alpha = 0.05;  // Lower = smoother, Higher = more responsive

// EEPROM save interval
int saveInterval = 1000;   // milliseconds between EEPROM writes

// Recalibration button hold time
const int RECALIBRATION_HOLD_TIME = 5000;  // 5 seconds

// Dual sensor configuration
bool useDualSensors = true;  // Enable/disable second BMA400
```

### Pin Configuration Customization

```cpp
// GPIO pin assignments
const int LED_PIN = 2;               // LED output pin
const int RECALIBRATION_PIN = 25;    // Button input pin (3.3kΩ pull-up)
const int SDA_PIN = 22;              // I2C Data pin
const int SCL_PIN = 21;              // I2C Clock pin

// BMA400 I2C addresses (if using different addresses)
#define BMA400_ADDR_1 0x14           // First sensor address
#define BMA400_ADDR_2 0x15           // Second sensor address (if SDO tied high)
```

### EEPROM Storage Configuration

```cpp
// Preferences namespace and keys
const char* PREF_NAMESPACE = "tiltSystem";
const char* ROLL_KEY = "lastRoll";
const char* PITCH_KEY = "lastPitch";
const char* CALIB_KEY = "calibrated";
```

---

## Troubleshooting

### Common Issues

#### "Error: BMA400 sensor(s) not connected!"

**Causes & Solutions:**
- Check I2C wiring (SDA to GPIO 22, SCL to GPIO 21)
- Verify 3.3V power connections to both BMA400 sensors
- Ensure GND connections are secure for all components
- Try different I2C pull-up resistors (4.7kΩ recommended)
- Verify BMA400 I2C addresses (0x14 and 0x15 if SDO configured)
- Use I2C scanner code to detect sensor addresses

#### LED Always On/Off

**Causes & Solutions:**
- Check LED wiring and current limiting resistor (220Ω-330Ω)
- Verify GPIO 2 connection and functionality
- Perform recalibration on perfectly level surface
- Check threshold value in code (default: ±5°)
- Monitor serial output for actual tilt angle values
- Ensure EEPROM data is not corrupted

#### Recalibration Button Not Working

**Causes & Solutions:**
- Verify button wiring to GPIO 25 with 3.3kΩ pull-up resistor
- Check button connection to 3.3V and GND
- Test button continuity with multimeter
- Hold button for full 5 seconds (not less)
- Monitor serial output for "Recalibration initiated" message
- Ensure button is momentary type, not latching

#### EEPROM Data Loss or Corruption

**Causes & Solutions:**
- Check for power supply instability during write operations
- Verify Preferences library initialization
- Monitor serial output for EEPROM save/restore messages
- Clear EEPROM and perform fresh recalibration
- Check for excessive write cycles (EEPROM wear)
- Implement data validation and checksum verification

#### Unstable or Noisy Readings

**Causes & Solutions:**
- Increase low-pass filter strength (higher alpha value)
- Check for loose I2C connections
- Shield system from electromagnetic interference
- Use shorter, shielded I2C cables
- Verify stable power supply (±5% regulation)
- Check sensor mounting for mechanical vibrations

### Diagnostic Commands

Add these debug functions to troubleshoot system issues:

```cpp
// In setup(), add after BMA400 initialization:
Serial.print("BMA400 Sensor 1 Device ID: 0x");
Serial.println(accelerometer1.getDeviceID(), HEX);
Serial.print("BMA400 Sensor 2 Device ID: 0x");
Serial.println(accelerometer2.getDeviceID(), HEX);

// EEPROM diagnostic
preferences.begin("tiltSystem", false);
Serial.print("Stored Roll: ");
Serial.println(preferences.getFloat("lastRoll", 0.0));
Serial.print("Stored Pitch: ");
Serial.println(preferences.getFloat("lastPitch", 0.0));
preferences.end();

// Real-time sensor data monitoring
Serial.print("Sensor 1 Raw - X: ");
Serial.print(accelerometer1.data.accelX);
Serial.print(", Y: ");
Serial.print(accelerometer1.data.accelY);
Serial.print(", Z: ");
Serial.println(accelerometer1.data.accelZ);

// I2C bus scanner function
void scanI2C() {
    for (byte address = 1; address < 127; address++) {
        Wire.beginTransmission(address);
        if (Wire.endTransmission() == 0) {
            Serial.print("I2C device found at address 0x");
            Serial.println(address, HEX);
        }
    }
}
```

---

## Features

### Current Features

-  **Real-time tilt detection** with ±5° threshold
-  **Automatic calibration** on first boot
-  **Manual recalibration** via button press
-  **Persistent storage** of calibration data
-  **Low-pass filtering** for stable readings
-  **Serial monitoring** with formatted output
-  **LED indication** for tilt status
-  **Single sensor operation** for simplicity

### Potential Enhancements

-  **WiFi connectivity** for remote monitoring
-  **Multiple threshold levels** with different indicators
-  **Data logging** to SD card or cloud
-  **Battery power** with sleep modes
-  **OLED display** for local readouts
-  **Buzzer alerts** for audio indication
-  **Smartphone app** integration

---

## Technical Details

### Sensor Specifications

| Parameter | Value |
|-----------|-------|
| **Resolution** | 12-bit |
| **Range** | ±2g, ±4g, ±8g, ±16g |
| **Sensitivity** | 1024 LSB/g (@±2g) |
| **Noise** | 150 μg/√Hz |
| **Temperature Range** | -40°C to +85°C |
| **Interface** | I2C, SPI |

### Mathematical Formulas

```cpp
// Roll calculation (rotation around X-axis)
roll = atan2(y, sqrt(x² + z²)) × (180/π)

// Pitch calculation (rotation around Y-axis)  
pitch = atan2(-x, sqrt(y² + z²)) × (180/π)

// Low-pass filter
filtered = α × new_value + (1-α) × filtered_previous
```

### Memory Usage

- **Flash Memory**: ~50KB (program + libraries)
- **SRAM**: ~8KB (variables + stack)
- **NVS Storage**: ~200 bytes (calibration data)

---

## Contributing

Feel free to contribute improvements:

1. **Fork** the repository
2. **Create** a feature branch
3. **Test** your changes thoroughly
4. **Submit** a pull request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Support

For issues or questions:

1. **Check** the troubleshooting section
2. **Monitor** serial output for error messages
3. **Verify** all hardware connections
4. **Test** with minimal setup first

---

