# ADCS Peripherals

Sensor interfaces and supporting electronics for the Columbia Space
Initiative (CSI) CubeSat's Attitude Determination and Control System (ADCS).

This directory contains the KiCad schematics and PCB design files for the
ADCS peripheral board and the sun sensor boards.

## System Overview

The ADCS peripheral board is built around an RP2350B microcontroller that
collects data from the attitude and navigation sensors and communicates with
the rest of the spacecraft.

- Six sun sensors, one on the center of each face of the CubeSat, measure the
  Sun direction. Each sensor is a small PCB with one BPW34 photodiode, wired
  to the peripheral board with a twisted pair.
- A BMI088 IMU (I2C) and an RM3100 magnetometer (SPI) are on the peripheral
  board.
- An Orion B16 GNSS receiver (UART) provides position, velocity, and time,
  using an external TW4721 antenna.
- The peripheral board sits at the top of the Z axis, close to the GNSS
  antenna, so the 50 ohm RF path stays as short as possible.

## Hardware Overview

| Function | Component | Qty | Interface | Role |
| :--- | :--- | :---: | :--- | :--- |
| Microcontroller | Raspberry Pi RP2350B | 1 | - | Sensor configuration, data acquisition, and communication with the spacecraft |
| Sun sensor | BPW34 | 6 | ADC (6 channels) | Light measurements used to estimate the Sun's direction |
| IMU | BMI088 | 1 | I2C | Three-axis acceleration and angular velocity measurements |
| Magnetometer | PNI RM3100 | 1 | SPI | Three-axis measurements of the local magnetic field |
| GNSS receiver | Orion B16 | 1 | UART | Position, velocity, and time estimation |
| GNSS antenna | TW4721 | 1 | RF (50 ohm) | GNSS signal reception, mounted outside the board |

All components except the sun sensors are single units.

## Sensor Functions

### Sun Sensor - BPW34

The sun-sensing assembly uses six BPW34 photodiodes to estimate the Sun's
direction. The photocurrent of each diode depends on the angle between the
Sun and the face it is mounted on, so comparing the six measurements gives
the Sun direction. These measurements support coarse attitude control, solar
panel pointing, and Sun avoidance for sensitive instruments.

**Placement (current plan, not final).** The CubeSat is a 2 x 3 form factor.
One sensor is placed at the center of each of the six faces. Each sensor has
its own small PCB, connected to the peripheral board with a twisted pair.
Because the distance to the peripheral board and the exact position differ
for each sensor, the cable length and routing of each channel must be taken
into account in the calculations (wire resistance, noise pickup, and
calibration).

**Wiring and layout guidelines.**
- Twist the two wires from each BPW34 together to minimize loop area; avoid
  routing them separately.
- Keep sensor wiring away from magnetorquer coils, driver circuits, DC/DC
  inductors, and switching nodes.
- Place the analog front end close to the sensor connector to minimize trace
  length between the connector and sensitive input nodes.
- Maintain a continuous ground reference and use component placement and
  routing to control high-current return paths.

**ADC design notes.**

1. *Sampling rate allocation.* The RP2350 internal ADC runs at 500 kS/s
   (48 MHz ADC clock, 96 cycles per conversion, 2 us per sample). The ADC is
   a single shared unit, so the six channels are sampled in round-robin mode.
   Each channel gets about 83.3 kS/s (one full round takes 12 us), and the
   per-channel Nyquist frequency is about 41.7 kHz.
2. *Low-pass filter.* Each channel uses a 200 ohm sense resistor followed by
   an RC low-pass filter (1 kohm + 1 uF). The time constant is about 1.2 ms
   (including the 200 ohm sense resistor), giving a cutoff frequency of about
   133 Hz (159 Hz if only the 1 kohm is counted). Attenuation at the Nyquist
   frequency is about 50 dB.
3. *Estimated maximum current and voltage.* At perihelion (solar irradiance
   about 1408 W/m^2), the estimated BPW34 photocurrent is 4 to 9 mA
   (extrapolated from the datasheet, not a datasheet value). The design limit
   is 12 mA, which gives a maximum sense voltage of about 2.4 V with the
   200 ohm resistor, below the 3.3 V supply and ADC input limit. This should
   be verified with real sunlight or a solar simulator.
4. *ADC choice.* The design uses the RP2350 internal ADC to keep the design
   simple, with no external ADC. If the internal ADC turns out to be
   unusable, one of the following external ADCs can be added:
   - MCP3208 (SPI, breadboard-friendly package, external voltage reference):
     https://ww1.microchip.com/downloads/aemDocuments/documents/APID/ProductDocuments/DataSheets/21298e.pdf
   - ADS7828 (I2C, surface-mount, internal voltage reference):
     https://www.ti.com/lit/ds/symlink/ads7828.pdf

### Inertial Measurement Unit - BMI088

The BMI088 combines a three-axis accelerometer and a three-axis gyroscope to
measure specific force and angular velocity. It communicates with the RP2350B
over I2C.

Gyroscope measurements support attitude propagation between updates from
other attitude sensors. Placement near the spacecraft's center of mass helps
reduce rotational contributions to accelerometer readings.

### Magnetometer - PNI RM3100

The RM3100 measures the local magnetic field. These measurements support
attitude estimation and magnetorquer control. It communicates with the
RP2350B over SPI.

**Placement and layout.**
- Place the sensor as far as practical from the magnetorquers, high-current
  wiring, and other magnetic interference sources.
- Follow the RM3100 Breakout Board User Manual (r08) for PCB layout. The X-
  and Y-axis sensors must be perpendicular to each other, with each sensor
  aligned parallel to its corresponding PCB edge.
- Calibration and interference testing are required to assess measurement
  reliability.

### GPS / GNSS - Orion B16 and TW4721 Antenna

The GNSS subsystem estimates the CubeSat's position, velocity, and time. The
Orion B16 receiver communicates with the RP2350B over UART and is connected
to an external TW4721 antenna. $ 1020.

**Placement.** The antenna is mounted outside the board, preferably on the
top of the Z axis. The peripheral board is placed at the top of the stack,
close to the antenna, so the RF trace is short.

**RF layout guidelines (RF input impedance 50 ohm).**
- Maintain 50 ohm impedance throughout the entire RF signal path, and keep
  the path as short as possible.
- Separate RF and digital circuits into different PCB regions.
- Do not route the RF signal near noisy sources such as digital signals,
  oscillators, switching power supplies, or other RF transmitting circuits.
- Do not route the RF signal under or over any other components (including the
  Orion B16) or other signal traces.
- Do not route the RF signal path on an inner layer of a multilayer PCB, to
  minimize signal loss.
- Avoid sharp bends. Use two 45-degree bends or a circular bend instead of a
  single 90-degree bend.
- Avoid vias on the RF signal path whenever possible, since every via adds
  inductance. Vias are acceptable for connecting RF grounds between layers.
  Each ground pin of the module should have a short trace tied immediately to
  the ground plane below through a via.
- Bypass capacitors should be low-ESR ceramic types, located directly adjacent
  to the pin they serve.

## Board Layout Summary

- The peripheral board is at the top of the Z axis, near the GNSS antenna.
- The magnetometer is kept far from the magnetorquers.
- Sun sensor wiring is kept away from magnetorquer coils, driver circuits, and
  switching nodes.
- RF and digital circuits are in separate regions.

## Repository Contents

- `component_datasheet/` - datasheets of the components used
- `library/` - symbols, footprints, and 3D models of the components
- KiCad project files for the peripheral board and the sun sensor boards

## Status and Open Items

- The Orion B16 is an export-controlled item and cannot be sold to
  individuals. Procurement under the CSI name is being discussed with the
  seller.
- The sun sensor placement (one per face) is the current plan and may change.
- The processing and software side of the GNSS and sensor interfaces is still
  to be implemented.

## Design Tools

- **EDA software:** KiCad 10.0
- **Microcontroller:** Raspberry Pi RP2350B
- **Version control:** Git / GitHub
