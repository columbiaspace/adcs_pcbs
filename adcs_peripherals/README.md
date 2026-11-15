# ADCS Peripherals

Sensor interfaces and supporting electronics for the Columbia Space
Initiative CubeSat’s Attitude Determination and Control System (ADCS).

This directory contains the KiCad schematics and PCB design files
for the ADCS peripheral board.


## Hardware Overview

| Function | Component | Role |
| :--- | :--- | :--- |
| Microcontroller | Raspberry Pi RP2350B | Sensor configuration, data acquisition, and communication with the spacecraft |
| Sun sensor | BPW34 photodiodes | Light measurements used to estimate the Sun’s direction |
| IMU | Bosch BMI088 | Three-axis acceleration and angular velocity measurements |
| GNSS RF front end | MAX2769 — model to be confirmed | GNSS signal reception for subsequent processing by a positioning backend |
| Magnetometer | PNI RM3100 | Three-axis measurements of the local magnetic field |

## Sensor Functions

### Sun Sensor — BPW34

The sun-sensing assembly uses BPW34 photodiodes to estimate the
Sun’s direction from light measurements taken in different orientations.

These measurements support coarse attitude control, solar panel
pointing, and Sun avoidance for sensitive instruments.

The sun sensors are mounted externally and connect to the peripheral
board through connectors.

### Inertial Measurement Unit — BMI088

The BMI088 combines a three-axis accelerometer and a three-axis
gyroscope to measure specific force and angular velocity.

Gyroscope measurements support attitude propagation between updates
from other attitude sensors. Placement near the spacecraft’s center
of mass helps reduce rotational contributions to accelerometer readings.

### GPS / GNSS

The GNSS subsystem is intended to estimate the CubeSat’s position,
velocity, and time using satellite navigation signals.

The selected RF front end requires additional signal processing
to produce a navigation solution. The processing platform, data
interface, and software implementation remain to be confirmed.

### Magnetometer — PNI RM3100

The RM3100 measures the local magnetic field. These measurements
support attitude estimation and magnetorquer control.

The sensor should be placed as far as practical from magnetorquers,
high-current wiring, and other magnetic interference sources.
Calibration and interference testing are required to assess
measurement reliability.

## Design Tools

- **EDA software:** KiCad 10.0
- **Microcontroller:** Raspberry Pi RP2350B
- **Version control:** Git / GitHub
