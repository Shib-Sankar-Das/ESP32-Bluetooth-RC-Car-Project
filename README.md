# 🚗 ESP32 Bluetooth RC Car Project

A comprehensive remote-controlled car project using ESP32 microcontroller with Bluetooth connectivity and L298N Dual H-Bridge motor driver. This project allows you to control a robotic car wirelessly using a smartphone app via Bluetooth communication with four DC gear motors for enhanced traction and control.

## � Project Overview

This RC car is built using an **ESP32 development board** and can be controlled remotely through Bluetooth using the **Dabble app**. The car features a **four-wheel drive system** with **L298N motor driver** controlling four DC geared motors. It supports basic movements including forward, backward, left turn, and right turn operations with tank-style steering (differential drive) for precise maneuvering.

## 📁 Files Included

* **`ESP32_Bluetooth_Car.ino`**: Arduino code for the ESP32 to handle Bluetooth commands and drive motors accordingly
* **`ESP32 Bluetooth Car.png`**: Detailed Fritzing circuit diagram illustrating all hardware connections
* **`README.md`**: Comprehensive project documentation with assembly and safety instructions

## 📋 Components Required

### Electronic Components:
| Component                          | Quantity | Description |
| ---------------------------------- | -------- | ----------- |
| **ESP32 Development Board**        | 1        | Main microcontroller with built-in Bluetooth |
| **L298N Dual H-Bridge Motor Driver** | 1      | Controls up to 4 DC motors (2 channels) |
| **DC Geared Motors (TT Motors)**    | 4        | For four-wheel drive system |
| **Wheels**                         | 4        | Compatible with TT motor shaft |
| **Jumper Wires**                   | Set      | Male-to-Male, Male-to-Female connections |
| **Breadboard or PCB**              | 1        | Optional for prototyping connections |
| **Motor Power Supply**             | 1        | 7-12V DC (Li-Po battery or 6xAA battery pack) |
| **Car Chassis**                    | 1        | Frame to mount all components |

### Tools Required:
- **Soldering iron and solder** - For permanent connections
- **Wire strippers** - For preparing wire connections
- **Multimeter** - Essential for voltage/continuity testing
- **Screwdrivers** - For mechanical assembly
- **Hot glue gun** - Optional for securing components
- **Reverse polarity protection diode** - Safety component for battery connections

## 🔌 Circuit Diagram Analysis

**Refer to `ESP32 Bluetooth Car.png` for detailed visual connections.**

### Pin Configuration:

#### Motor Driver Control Pins:
Based on the circuit diagram, the ESP32 controls the L298N motor driver through the following GPIO connections:

| ESP32 GPIO | L298N Pin | Function | Motor Control |
| ---------- | --------- | -------- | ------------- |
| GPIO 22    | ENA       | Enable A | Right motors speed control (PWM) |
| GPIO 16    | IN1       | Input 1  | Right motors direction control |
| GPIO 17    | IN2       | Input 2  | Right motors direction control |
| GPIO 23    | ENB       | Enable B | Left motors speed control (PWM) |
| GPIO 18    | IN3       | Input 3  | Left motors direction control |
| GPIO 19    | IN4       | Input 4  | Left motors direction control |
| GND        | GND       | Ground   | Common ground reference |

### Circuit Connection Details:

#### L298N Motor Driver to Motors:
```
L298N Output  →   Motor Connection        →   Description
OUT1          →   Right Motor1 (+)        →   Right side motors in parallel
OUT2          →   Right Motor1 (-)        →   Right side motors in parallel
OUT1          →   Right Motor2 (+)        →   Connected parallel to Motor1
OUT2          →   Right Motor2 (-)        →   Connected parallel to Motor1
OUT3          →   Left Motor1 (+)         →   Left side motors in parallel
OUT4          →   Left Motor1 (-)         →   Left side motors in parallel
OUT3          →   Left Motor2 (+)         →   Connected parallel to Motor1
OUT4          →   Left Motor2 (-)         →   Connected parallel to Motor1
```

#### Power Distribution System:
```
Power Source     →    Connection Point           →    Voltage Level
7-12V Battery +  →    L298N VCC (+12V)          →    7-12V DC Motor Power
7-12V Battery -  →    L298N GND & ESP32 GND     →    0V (Common Ground)
L298N +5V Out    →    ESP32 VIN                 →    5V Regulated (ESP32 Power)
```

#### Critical Power Supply Notes:
- **⚠️ L298N 5V Regulator Jumper**: Ensure the onboard 5V regulator jumper on the L298N is present to supply regulated 5V to ESP32
- **⚠️ Current Capacity**: L298N supports up to **2A per channel** - verify your motors don't exceed this limit
- **⚠️ Heat Dissipation**: L298N may require heat sink for continuous operation at high currents

## 📱 Software Requirements

### Arduino IDE Setup:
1. **Install ESP32 Board Package**:
   - Open Arduino IDE
   - Go to File → Preferences
   - Add this URL to Additional Board Manager URLs:
     ```
     https://dl.espressif.com/dl/package_esp32_index.json
     ```
   - Go to Tools → Board → Board Manager
   - Search for "ESP32" and install
   - **⚠️ IMPORTANT**: Use **ESP32 Board Package version 2.0.16** for compatibility

2. **ESP32 Library Version Compatibility**:
   ```
   ⚠️ CRITICAL COMPATIBILITY NOTE:
   The PWM setup functions in the code may throw compilation errors 
   with newer ESP32 board package versions:
   
   ledcSetup(rightMotorPWMSpeedChannel, PWMFreq, PWMResolution);
   ledcSetup(leftMotorPWMSpeedChannel, PWMFreq, PWMResolution);  
   ledcAttachPin(enableRightMotor, rightMotorPWMSpeedChannel);
   ledcAttachPin(enableLeftMotor, leftMotorPWMSpeedChannel);
   
   ✅ SOLUTION: Install ESP32 Board Package version 2.0.16 specifically
   - In Board Manager, click on ESP32 package
   - Select version 2.0.16 from dropdown
   - This ensures PWM functions work correctly
   ```

3. **Install Dabble Library**:
   - Go to Sketch → Include Library → Manage Libraries
   - Search for "Dabble ESP32" 
   - Install the library by STEMpedia

### Mobile App:
- Download **Dabble** app from Google Play Store or Apple App Store
- The app provides a gamepad interface for controlling the car
- Alternative: Use any **Bluetooth Terminal app** (like *Serial Bluetooth Terminal*) for direct command control

### Alternative Control Methods:
| Command | Action        | Dabble GamePad | Terminal Command |
| ------- | ------------- | -------------- | ---------------- |
| Forward | Move Forward  | Up Arrow ↑     | Send `F`        |
| Backward| Move Backward | Down Arrow ↓   | Send `B`        |
| Left    | Turn Left     | Left Arrow ←   | Send `L`        |
| Right   | Turn Right    | Right Arrow →  | Send `R`        |
| Stop    | Stop Motors   | Release buttons| Send `S`        |

## 🔧 Code Explanation

### Key Features:
- **PWM Speed Control**: Uses 8-bit PWM (0-255) for smooth speed control via enable pins
- **Bluetooth Communication**: Uses Dabble library for wireless gamepad control
- **Four-Wheel Drive**: Controls 4 motors (2 per side) for enhanced traction
- **Tank-Style Steering**: Implements differential drive for precise turning
- **Gamepad Interface**: Responds to directional commands from smartphone

### Motor Control Logic:
The code uses a differential drive system where motors on each side work together:

```cpp
// Forward: Both motor groups rotate forward
if (GamePad.isUpPressed()) {
    rightMotorSpeed = MAX_MOTOR_SPEED;  // 255 - Right motors (Motor1 & Motor2)
    leftMotorSpeed = MAX_MOTOR_SPEED;   // 255 - Left motors (Motor1 & Motor2)
}

// Backward: Both motor groups rotate backward  
if (GamePad.isDownPressed()) {
    rightMotorSpeed = -MAX_MOTOR_SPEED; // -255 - Right motors reverse
    leftMotorSpeed = -MAX_MOTOR_SPEED;  // -255 - Left motors reverse
}

// Left Turn: Right motors forward, left motors backward (tank steering)
if (GamePad.isLeftPressed()) {
    rightMotorSpeed = MAX_MOTOR_SPEED;  // 255 - Right side forward
    leftMotorSpeed = -MAX_MOTOR_SPEED;  // -255 - Left side backward
}

// Right Turn: Left motors forward, right motors backward (tank steering)
if (GamePad.isRightPressed()) {
    rightMotorSpeed = -MAX_MOTOR_SPEED; // -255 - Right side backward
    leftMotorSpeed = MAX_MOTOR_SPEED;   // 255 - Left side forward
}
```

### PWM Speed Control Implementation:
```cpp
// PWM configuration for smooth speed control
const int PWMFreq = 1000; /* 1 KHz frequency */
const int PWMResolution = 8; /* 8-bit resolution (0-255) */
const int rightMotorPWMSpeedChannel = 4; /* Channel 4 for right motors */
const int leftMotorPWMSpeedChannel = 5;  /* Channel 5 for left motors */

// Apply speed control via PWM
ledcWrite(rightMotorPWMSpeedChannel, abs(rightMotorSpeed));
ledcWrite(leftMotorPWMSpeedChannel, abs(leftMotorSpeed));
```

## ⚡ Assembly Instructions

### Step 1: Component Testing (MANDATORY)
**⚠️ CRITICAL: Test each component individually before assembly to prevent damage!**

#### 🔧 Individual Component Testing Protocol:

1. **Test Each DC Motor Individually**:
   ```
   ✅ Connect each motor directly to power supply (within voltage limits 6-12V)
   ✅ Verify smooth rotation in both directions
   ✅ Check for any mechanical binding or unusual noises
   ✅ Test current consumption (should be <2A per motor)
   ✅ Ensure all 4 motors have similar performance characteristics
   ```

2. **Test ESP32 Development Board**:
   ```
   ✅ Upload a simple blink program to verify programming capability
   ✅ Test all required GPIO pins (22, 16, 17, 23, 18, 19) with LED connections
   ✅ Verify Bluetooth functionality with serial monitor
   ✅ Check 3.3V and GND pin outputs with multimeter
   ```

3. **Test L298N Motor Driver Module**:
   ```
   ✅ Apply 7-12V power and verify 5V regulated output
   ✅ Test each motor output channel (OUT1/OUT2 and OUT3/OUT4) separately
   ✅ Verify enable pins (ENA, ENB) control speed correctly with PWM
   ✅ Check for overheating during continuous operation
   ✅ Ensure onboard 5V regulator jumper is properly positioned
   ```

4. **Test Power Supply System**:
   ```
   ✅ Measure output voltage with multimeter (should be 7-12V stable)
   ✅ Test under load conditions with motors connected
   ✅ Check for voltage stability during motor operation
   ✅ Verify current capacity meets system requirements (>4A recommended)
   ✅ Test reverse polarity protection if implemented
   ```

### Step 2: Circuit Assembly

#### 🏗️ Mechanical Assembly:
1. **Prepare the Chassis**:
   - Mount all 4 motors securely to the chassis frame
   - Attach wheels to motor shafts ensuring proper alignment
   - Create secure mounting points for ESP32 and L298N modules
   - Ensure adequate spacing for wire routing and component access

#### ⚡ Electrical Assembly (Follow pin configuration table above):

2. **Power Connection Sequence** (CRITICAL ORDER):
   ```
   Step 1: Connect all GND lines first (ESP32 GND ↔ L298N GND ↔ Battery -)
   Step 2: Connect motor power (Battery + → L298N VCC, 7-12V)
   Step 3: Connect ESP32 power (L298N +5V → ESP32 VIN)
   Step 4: VERIFY all power connections with multimeter before proceeding
   ```

3. **Motor Wiring Configuration**:
   ```
   Right Side Motors (Parallel Connection):
   - Motor1: OUT1 (+), OUT2 (-)
   - Motor2: OUT1 (+), OUT2 (-) [Connected parallel to Motor1]
   
   Left Side Motors (Parallel Connection):
   - Motor1: OUT3 (+), OUT4 (-)
   - Motor2: OUT3 (+), OUT4 (-) [Connected parallel to Motor1]
   ```

4. **Control Signal Wiring**:
   ```
   ESP32 → L298N Signal Connections:
   GPIO 22 → ENA (Right motors speed)
   GPIO 16 → IN1 (Right motors direction)
   GPIO 17 → IN2 (Right motors direction)  
   GPIO 23 → ENB (Left motors speed)
   GPIO 18 → IN3 (Left motors direction)
   GPIO 19 → IN4 (Left motors direction)
   ```

5. **Wire Management & Safety**:
   - Use different colored wires for easy identification (Red: +V, Black: GND, Others: Signals)
   - Secure all connections with proper connectors or soldering
   - Route wires away from moving parts (wheels, motors)
   - Use cable ties or clips to prevent wire disconnection during operation

## ⚠️ Important Precautions & Safety Guidelines

### 🔋 Critical Power Supply Precautions:
1. **Voltage Verification Protocol**:
   ```
   ⚠️ NEVER connect power without multimeter verification
   ✅ Verify 7-12V at L298N VCC terminal before connection
   ✅ Verify 5V regulated output from L298N before connecting ESP32
   ✅ Check 3.3V logic levels at ESP32 GPIO pins
   ✅ Ensure voltage stability under load conditions
   ```

2. **Polarity Protection Measures**:
   ```
   ❌ NEVER reverse polarity - can destroy components instantly
   ✅ Add reverse polarity protection diode to battery connections
   ✅ Mark positive and negative terminals clearly
   ✅ Use polarized connectors where possible
   ✅ Double-check with multimeter before each power-on
   ```

3. **Current & Thermal Management**:
   ```
   ⚠️ L298N maximum: 2A per channel (4A total for 4 motors)
   ✅ Monitor L298N temperature during operation
   ✅ Add heat sink if continuous high-current operation expected
   ✅ Ensure adequate ventilation around motor driver
   ✅ Use appropriate wire gauge for current requirements (18-20 AWG recommended)
   ```

### 🔌 Connection Testing Protocol:
1. **Multi-Stage Verification Process**:
   ```
   Stage 1: Visual inspection of all connections
   Stage 2: Continuity testing with multimeter (power OFF)
   Stage 3: Voltage verification at each connection point
   Stage 4: Individual subsystem testing before integration
   Stage 5: Complete system test with gradual power increase
   ```

2. **Short Circuit Prevention**:
   ```
   ✅ Check for shorts between VCC and GND at every connection point
   ✅ Verify no exposed wire strands touching adjacent connections
   ✅ Test each GPIO pin for proper voltage levels (0V/3.3V)
   ✅ Ensure proper insulation between power and signal lines
   ```

### 🛡️ Component Protection Guidelines:
1. **ESP32 Protection**:
   ```
   ✅ Never exceed 3.3V on GPIO pins (5V tolerant but not recommended)
   ✅ Use current-limiting resistors for LED connections
   ✅ Protect against electrostatic discharge during handling
   ✅ Ensure stable power supply (avoid voltage spikes)
   ```

2. **Motor Driver Protection**:
   ```
   ✅ Never connect motors while power is applied
   ✅ Add flyback diodes for inductive load protection (usually built-in)
   ✅ Ensure proper ground connection between all components
   ✅ Monitor for overheating during extended operation
   ```

### 🔧 Assembly Best Practices:
1. **Progressive Testing Protocol**:
   ```
   Step 1: Test individual components separately
   Step 2: Test power distribution system
   Step 3: Test control signals without motors
   Step 4: Test one motor at a time
   Step 5: Test complete system with reduced power
   Step 6: Full system operation test
   ```

2. **Emergency Preparedness**:
   ```
   ✅ Keep spare fuses and components available
   ✅ Have fire extinguisher rated for electrical fires nearby
   ✅ Install emergency power disconnect switch
   ✅ Keep multimeter accessible for troubleshooting
   ```

### ✅ Pre-Operation Checklist:
- [ ] All connections verified with multimeter
- [ ] No short circuits between power rails
- [ ] Correct polarity on all power connections  
- [ ] GPIO pins correctly mapped and tested
- [ ] Motor driver properly configured and cooled
- [ ] Bluetooth pairing successful and responsive
- [ ] Motors rotate in correct directions
- [ ] Emergency stop procedure established
- [ ] Spare components available for quick replacement

## 🚀 Usage Instructions

### 📱 Control Setup Options:

#### Option 1: Dabble App (Recommended)
1. **Initial Setup**:
   - Power on the ESP32 car
   - Open Dabble app on your smartphone
   - Connect to "MyBluetoothCar" via Bluetooth
   - Select GamePad module in the app

2. **GamePad Controls**:
   - **Up Arrow ↑**: Move forward (all 4 motors forward)
   - **Down Arrow ↓**: Move backward (all 4 motors reverse)
   - **Left Arrow ←**: Turn left (right motors forward, left motors reverse)
   - **Right Arrow →**: Turn right (left motors forward, right motors reverse)
   - **Release**: Stop all motors

#### Option 2: Bluetooth Terminal App
1. **Setup**:
   - Use any Bluetooth terminal app (like *Serial Bluetooth Terminal*)
   - Pair with ESP32 via Bluetooth (name: "ESP32_BT" or similar)
   - Open serial connection

2. **Terminal Commands**:
   | Command | Action | Motor Behavior |
   | ------- | ------ | -------------- |
   | `F` | Forward | All motors rotate forward |
   | `B` | Backward | All motors rotate backward |
   | `L` | Left Turn | Right motors forward, left reverse |
   | `R` | Right Turn | Left motors forward, right reverse |
   | `S` | Stop | All motors stop |

### 🔧 Troubleshooting Guide:

#### Connection Issues:
```
Problem: Car doesn't respond to commands
Solutions:
✅ Check Bluetooth connection status in app
✅ Verify ESP32 is powered and LED indicators active
✅ Restart both app and ESP32
✅ Check if paired device list shows the car

Problem: Bluetooth won't pair
Solutions:
✅ Clear Bluetooth cache on smartphone
✅ Reset ESP32 and try pairing again
✅ Ensure no other devices connected to ESP32
✅ Check ESP32 code for correct device name
```

#### Movement Issues:
```
Problem: Car moves in wrong direction
Solutions:
✅ Check motor wiring polarity (swap motor wires if needed)
✅ Verify GPIO pin assignments match code
✅ Test individual motors to confirm rotation direction

Problem: Car moves slowly or inconsistently
Solutions:
✅ Check battery voltage (should be >7V under load)
✅ Verify all motor connections are secure
✅ Check for mechanical binding in wheels/chassis
✅ Monitor L298N temperature (may be overheating)

Problem: Only some motors work
Solutions:
✅ Test each motor individually with direct power
✅ Check L298N output channels with multimeter
✅ Verify enable pin connections (ENA, ENB)
✅ Check for loose connections on non-working motors
```

#### Power Issues:
```
Problem: ESP32 resets randomly
Solutions:
✅ Check power supply stability under motor load
✅ Verify adequate current capacity (>4A total)
✅ Check for voltage drops during motor operation
✅ Add capacitors across power supply if needed

Problem: L298N gets very hot
Solutions:
✅ Reduce motor current (check motor specifications)
✅ Add heat sink to L298N chip
✅ Improve ventilation around motor driver
✅ Check for motor stall conditions
```

#### Compilation & Upload Issues:
```
Problem: PWM setup functions throw compilation errors
Error messages like: "ledcSetup() not declared" or "ledcAttachPin() not found"
Solutions:
✅ Install ESP32 Board Package version 2.0.16 specifically
✅ Avoid newer ESP32 board package versions (3.x) that have API changes
✅ In Board Manager: ESP32 → Select version 2.0.16 from dropdown
✅ Restart Arduino IDE after version change

Problem: Code compiles but PWM doesn't work
Solutions:
✅ Verify ESP32 board package version is 2.0.16
✅ Check PWM channel assignments (channels 4 and 5 used in code)
✅ Ensure PWM frequency (1000 Hz) is supported
✅ Test with simple PWM example first

Problem: Upload fails or "Brownout detector" errors
Solutions:
✅ Hold BOOT button on ESP32 during upload
✅ Check USB cable quality and connection
✅ Reduce baud rate in Arduino IDE (115200 → 9600)
✅ Ensure adequate power supply during programming
```

## 🔧 Customization & Enhancement Options

### Basic Customizations:
- **Speed Control**: Modify `MAX_MOTOR_SPEED` value (0-255) for different speed levels
- **Bluetooth Name**: Change device name in `Dabble.begin("YourCustomCarName")`
- **PWM Frequency**: Adjust `PWMFreq` for different motor response characteristics
- **Turn Sensitivity**: Implement variable speed turning instead of full-speed tank steering

### Advanced Enhancements:
#### 🚀 Performance Upgrades:
- **Variable Speed Control**: Add joystick control for analog speed adjustment
- **Acceleration Curves**: Implement gradual speed ramping for smoother operation
- **PID Control**: Add feedback control for straight-line driving accuracy
- **Battery Monitoring**: Implement voltage sensing for low-battery warnings

#### 🔧 Hardware Additions:
- **Obstacle Avoidance**: Add ultrasonic sensors (HC-SR04) for autonomous navigation
- **Camera Module**: Integrate ESP32-CAM for FPV (First Person View) control
- **LED Lighting**: Add headlights, brake lights, and turn signal indicators
- **Speaker/Buzzer**: Add audio feedback for commands and warnings
- **IMU Sensor**: Add gyroscope/accelerometer for stability control

#### 📱 Software Enhancements:
- **Custom Mobile App**: Develop using MIT App Inventor or Flutter for personalized control
- **Web Interface**: Create web-based control panel accessible via ESP32 WiFi
- **Voice Control**: Integrate voice commands through smartphone apps
- **Automation Modes**: Add pre-programmed movement patterns or routes

#### 🏁 Racing & Competition Features:
- **Lap Timer**: Implement checkpoint-based timing system
- **Speed Telemetry**: Real-time speed and distance tracking
- **Remote Monitoring**: Live status updates via Bluetooth or WiFi
- **Multiple Car Control**: Support for controlling multiple cars simultaneously

## 💡 Future Project Ideas:
1. **Autonomous Mode**: GPS navigation with waypoint following
2. **Swarm Robotics**: Multiple cars working together
3. **Soccer Robot**: Implement ball-following and kicking mechanisms
4. **Line Following**: Add line-following sensors for track-based navigation
5. **Gesture Control**: Control via smartphone accelerometer/gyroscope

## 📚 Additional Resources & References

### Official Documentation:
- [ESP32 Official Documentation](https://docs.espressif.com/projects/esp32/en/latest/) - Complete ESP32 reference
- [Dabble Library Documentation](https://thestempedia.com/docs/dabble/) - Dabble app and library guide
- [L298N Motor Driver Guide](https://lastminuteengineers.com/l298n-dc-stepper-driver-arduino-tutorial/) - Detailed L298N tutorial

### Circuit Analysis Resources:
- **Fritzing Software**: For creating and modifying circuit diagrams
- **KiCad**: Professional PCB design software for permanent circuits
- **Tinkercad Circuits**: Online circuit simulation before building

### Programming Resources:
- **Arduino IDE**: Official ESP32 development environment
- **PlatformIO**: Advanced development platform for ESP32
- **ESP-IDF**: Native ESP32 development framework

### Hardware Suppliers:
- **Adafruit**: Quality components and educational resources
- **SparkFun**: Electronics components with excellent tutorials
- **Arduino Official Store**: Genuine Arduino-compatible boards
- **Local Electronics Stores**: For basic components and quick purchases

### Community & Support:
- **Arduino Forum**: Community support for programming questions
- **ESP32 Reddit Community**: Active discussion and project sharing
- **YouTube Tutorials**: Visual learning resources for assembly and programming
- **GitHub Projects**: Open-source RC car projects for inspiration

## 📷 Circuit Diagram Reference

**Important**: Always refer to the included `ESP32 Bluetooth Car.png` diagram for accurate visual representation of all connections. The diagram shows:

- Complete wiring layout with color-coded connections
- Component placement and orientation
- Power distribution system
- Signal routing between ESP32 and L298N
- Motor connection configuration (parallel wiring)
- Safety considerations and component spacing

## 🤝 Contributing to This Project

We welcome contributions to improve this RC car project! Here's how you can help:

### Ways to Contribute:
- **Bug Reports**: Report any issues with code, documentation, or circuit design
- **Feature Suggestions**: Propose new capabilities or improvements
- **Documentation**: Improve clarity, add translations, or expand explanations  
- **Code Enhancements**: Optimize performance, add features, or fix bugs
- **Circuit Improvements**: Suggest better component choices or layout optimization

### Contribution Guidelines:
1. **Fork the repository** and create a feature branch
2. **Test your changes** thoroughly before submitting
3. **Document your modifications** clearly in commit messages
4. **Update README** if your changes affect usage or assembly
5. **Submit pull request** with detailed description of improvements

### Share Your Build:
- Post photos of your completed car
- Share performance videos or modifications
- Document any creative enhancements you've made
- Help other builders with troubleshooting advice

## 📄 License & Legal

This project is released under the **MIT License**, which means:

- ✅ **Free to use** for personal and educational purposes
- ✅ **Modify and distribute** with proper attribution  
- ✅ **Commercial use allowed** with license inclusion
- ❌ **No warranty provided** - use at your own risk

### Attribution:
When sharing or modifying this project, please include:
- Link to original repository
- Credit to original authors
- Copy of MIT license text

---

## ⚠️ Final Safety Reminder

**CRITICAL**: This project involves electricity, moving parts, and potentially dangerous voltages. Always prioritize safety:

- 🔋 **Double-check all electrical connections** before applying power
- 🧯 **Keep fire safety equipment** nearby when working with electronics  
- 👥 **Adult supervision required** for younger builders
- 🔧 **Test components individually** before complete assembly
- 📞 **Know emergency procedures** in case of electrical accidents

**Remember**: It's better to spend extra time on safety verification than to risk component damage or personal injury. When in doubt, consult with experienced electronics enthusiasts or professionals.

---

*Happy building, and enjoy your ESP32 Bluetooth RC Car! 🚗💨*
