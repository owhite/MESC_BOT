# MESC_brain_board.md


## Overview


The **MESC Brain Board** is a Teensy 4.0-based controller board that coordinates BLDC motor control via CAN bus and receives telemetry from distributed motor controllers. It is part of a modular robotics system leveraging **MESC firmware** for precise and responsive control.


This board serves as the high-level processor, aggregating IMU data, user commands, and telemetry from motor controllers to execute balancing, motion, and behavior strategies.


---


## Hardware Architecture


### Microcontroller
- **Teensy 4.0**
  - 600 MHz ARM Cortex-M7
  - High-speed CAN interface
  - SPI, I2C, UART, PWM capable
  - No native Wi-Fi


### Communication
- **CAN Bus**
  - Used for motor controller communication
  - Sends: torque/current/velocity commands
  - Receives: torque estimate, phase current, position, velocity


- **Wi-Fi Module (ESP32)**
  - Used for UI control and telemetry
  - Can receive control inputs (e.g., from sliders)
  - Can receive PWM input (e.g., RC controller)


### Sensor Suite
- **IMU: MPU6050**
  - Connected via I2C
  - Supplies orientation data for balancing


- **Encoders (MT6701)**
  - Magnetic absolute encoders
  - Connected to motor controllers for closed-loop control


---


## Functional Description


### Control Loop
- The Teensy receives real-time IMU data
- It fuses sensor data and applies a control law (e.g., PID, LQR, or hybrid rule-based)
- Based on orientation/command input, it sends torque/velocity commands over CAN


### Telemetry
- Every 100ms, motor controllers send the following via CAN:
  - Angular position
  - Mechanical velocity
  - Phase current
  - Torque estimate
  - Supply voltage
  - Temperature (if supported)


### UI Control
- Wi-Fi (via ESP32) enables:
  - Real-time tuning using sliders
  - Graphing telemetry in a browser
  - Command input from browser or RC PWM input


---


## Design Considerations


- **High-frequency loop support**: Teensy 4.0 allows fast control cycles.
- **Isolation/Testpoints**: CAN and encoder signals routed to testpoints for probing.
- **Power Regulation**: 15V supply stepped down to 3.3V for logic via buck converter.


---


## Expansion


The MESC Brain Board is designed for extensibility:
- Additional sensors via I2C/SPI
- Logging to SD card (future)
- External kill switch or emergency stop
- Torque estimation refinement


---


## Future Enhancements


A list of advanced features to consider adding to the motor controller firmware:
- Feedforward torque
- Gain scheduling
- Motion profiling
- Sensor fusion filtering
- Onboard diagnostics and fault logging


---


## Summary


This brain board complements the MESC motor control system by offloading sensor fusion and motion logic to a dedicated controller while maintaining real-time telemetry and control over CAN. With a focus on modularity, observability, and hybrid control strategies, this system is ideal for balancing robots and quadrupedal walking robots alike.
