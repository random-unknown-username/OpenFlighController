# OpenFlighController
A Pixhawk-like flight controller made from scratch for fun, using an STM32H743 main flight MCU, a dedicated IO MCU, redundant motion sensors, barometers, CAN, Ethernet, SD logging, FRAM storage, and protected aircraft I/O.

<img width="1013" height="638" alt="image" src="https://github.com/user-attachments/assets/1019e9f5-a65e-44e4-9d5f-c5e1656152b9" />
<img width="726" height="548" alt="image" src="https://github.com/user-attachments/assets/9d7cb7ee-081a-48d6-a15b-4174e44a4c4a" />

**OpenFlightController** is my attempt at designing a complete drone flight controller from the ground up instead of using an existing board. **The goal of this project was to understand how real flight-controller hardware is built:** power rails, IMU redundancy, MCU pin planning, sensor buses, protected I/O, CAN, USB, Ethernet, storage, and dense PCB routing.

### Project Status
This project currently includes:

- STM32H743 main flight controller section
- STM32F103 IO controller section
- Multiple IMUs and barometers
- microSD and FRAM storage
- USB-C, CAN, Ethernet, UART, I2C, SPI, PWM, ADC, and RC input sections
- Power regulation and protected 5V peripheral outputs

The board is still a hardware learning project and **should be treated as experimental** until manufactured, assembled, tested, and validated properly.

### Why I Made This

I wanted to build a flight controller from scratch to understand what actually goes into boards like Pixhawk.
(Also Pixhawk boards are really costly so i had to be creative, i wanted something powerful and versatile enough i can use it in all my RC projects)

A flight controller has to combine a lot of different electronics topics in one board:

- Fast digital buses for sensors
- Protected power for external modules
- Reliable storage for logs and parameters
- PWM outputs for ESCs and servos
- RC input handling
- USB for programming/debugging
- CAN for drone peripherals
- ADC monitoring for battery and rail voltages


## Main Hardware Architecture

The design is split into two main controller sections:

#### FMU MCU

The main flight controller MCU is an STM32H743. It is responsible for the core flight-controller tasks such as sensor reading, control loops, logging, communication, and managing onboard peripherals.

The STM32H743 was chosen because it has a large number of peripherals, including:

- Multiple SPI buses
- Multiple UART/USART ports
- I2C buses
- CAN interfaces
- SDMMC
- Ethernet RMII
- ADC inputs
- Timers for PWM
- USB FS
- High processing performance for real-time control

#### IO MCU

The board also includes a separate STM32F103 IO MCU.

The IO MCU is used for output and receiver-related work, similar to the split architecture used in some Pixhawk-style flight controllers. It can handle PWM outputs, RC input, SBUS/PPM-related signals, safety switch logic, and communication with the main FMU.

This separation makes the design more organized because the main MCU can focus on flight computation and sensors, while the IO MCU handles aircraft I/O.

## Sensors

The board includes multiple sensors for redundancy and comparison.

#### IMUs

The design includes multiple IMU options:

- BMI088
- ICM-42688-P
- ICM-20689

The IMUs are connected mostly through SPI because SPI is faster and more deterministic than I2C. For flight control, gyro and accelerometer timing matters a lot, so SPI is a better choice for the main motion sensors.

Multiple IMUs allow the firmware to compare readings and potentially reject a noisy or failed sensor.

#### Barometers

The board includes barometer options such as:

- BMP581
- ICP-20100

Barometers are used for pressure-based altitude estimation. 

#### Magnetometer

The board includes an **IST8310** magnetometer section.

A magnetometer can be used for heading estimation, although in real aircraft designs it is often better to place the compass away from high-current traces and magnetic noise. This board also includes external I2C/GPS-style ports where an external compass could be used.

## Storage

The board includes two types of storage.

#### microSD

The microSD card is connected through SDMMC. It is intended for flight logs, debug logs, blackbox-style data, and general storage.

SDMMC was used instead of basic SPI mode because it can provide better performance for logging.

#### FRAM

The board also includes SPI FRAM.

FRAM is useful for smaller but important data, such as:

- Parameters
- Calibration values
- Counters
- Boot state
- Small persistent logs

FRAM has much better write endurance than normal flash, which makes it useful for data that may be written frequently.

## Communication Interfaces

The board has several communication options.

#### USB-C

USB-C is included for programming, debugging, and communication with the board. The design uses USB 2.0 D+ and D- lines with CC resistors and ESD protection.

#### CAN

The board includes two CAN bus transceiver sections.

CAN is useful in drones because it is robust over cables and allows multiple devices to share one bus. It can be used for smart batteries, **GPS modules, ESCs, companion nodes, and UAVCAN/Cyphal-style peripherals.**

#### Ethernet (optionals kinda)

This was just add because like I could add it, this can be useful in a few ways on land RC vehicles

#### UART / Telemetry / GPS

The board includes multiple UART ports for:

- GPS
- Telemetry
- Debug
- IO MCU communication
- External serial peripherals

Some telemetry-style ports include RTS/CTS flow control signals.

#### I2C

The board includes multiple I2C buses for sensors, EEPROMs, GPS/magnetometer ports, and external peripherals.

I2C is useful for slower devices because many chips can share the same two wires, but it requires pull-up resistors and careful handling when routed to external connectors.

## Power System

The power system is separated into multiple rails instead of using one shared 3.3V rail for everything.

The design includes rails such as:

- Main 5V input
- MCU 3.3V
- IO 3.3V
- IMU 3.3V
- SD card 3.3V
- Protected 5V output rails for peripherals

This separation helps reduce noise, allows some rails to be enabled or disabled by firmware, and protects the main flight controller from external peripheral faults.

## ADC Monitoring

The board includes ADC inputs for monitoring things like:

- Battery voltage
- Battery current
- 5V rail
- 3.3V rail
- Spare analog inputs
- RSSI

Voltage dividers are used to scale higher voltages down to safe ADC levels. Small capacitors are used for filtering so the ADC readings are less noisy.

## RC Input and PWM Output

The design includes support for RC input signals such as:

- PPM
- SBUS
- DSM-style input
- RSSI

SBUS needs special handling because it is commonly inverted compared to normal UART logic, so the board includes inverter/buffer logic.

**The board also has multiple PWM output channels for ESCs or servos.**

## Screenshots

<img width="805" height="608" alt="image" src="https://github.com/user-attachments/assets/fe218486-25cc-4165-8b6a-9dbc2568ec24" />

<img width="962" height="651" alt="image" src="https://github.com/user-attachments/assets/9eebc211-ee98-4ca4-bbd9-7929d6c601e6" />

<img width="944" height="517" alt="image" src="https://github.com/user-attachments/assets/c1f42cf3-0c4d-4a50-a648-627585e88128" />

<img width="922" height="444" alt="image" src="https://github.com/user-attachments/assets/0f6d52d4-6540-440d-887a-577c9d7ce642" />

<img width="901" height="656" alt="image" src="https://github.com/user-attachments/assets/41cce102-f5e9-4ac7-bfbf-6b1e1751ecd7" />

<img width="1012" height="673" alt="image" src="https://github.com/user-attachments/assets/b6e5abae-3f40-4f5a-8caa-8eecd4e80216" />

<img width="998" height="629" alt="image" src="https://github.com/user-attachments/assets/c1f85100-f8c3-4d7f-b6fe-7297fa09f6f9" />

<img width="960" height="579" alt="image" src="https://github.com/user-attachments/assets/aa62459e-80cb-4e1a-8283-3f1a8746e98a" />

<img width="537" height="552" alt="image" src="https://github.com/user-attachments/assets/704d93fa-e0c6-49a1-a837-5993b070b2ef" />

<img width="928" height="580" alt="image" src="https://github.com/user-attachments/assets/a7863a98-84f0-484e-aaeb-a12c7dc4d13c" />

<img width="889" height="605" alt="image" src="https://github.com/user-attachments/assets/acfcac7c-ded0-4a32-9a07-a6cecfe3c508" />

<img width="805" height="608" alt="image" src="https://github.com/user-attachments/assets/7c25f677-9630-4b07-95b4-21a566dbc41d" />
