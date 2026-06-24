# OpenFlighController
A Pixhawk-like flight controller made from scratch for fun, using an STM32H743 main flight MCU, a dedicated IO MCU, redundant motion sensors, barometers, CAN, Ethernet, SD logging, FRAM storage, and protected aircraft I/O.

EASYEDA LINK for macondo reviewers- https://u.easyeda.com/join?type=project&key=2c0e2fa679abcafb98220dea7d7db287&inviter=3d0033e0d0a04fe88f26aec1c4ce7725

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
<img width="805" height="608" alt="image" src="https://github.com/user-attachments/assets/7c25f677-9630-4b07-95b4-21a566dbc41d" />

<img width="889" height="605" alt="image" src="https://github.com/user-attachments/assets/acfcac7c-ded0-4a32-9a07-a6cecfe3c508" />

<img width="928" height="580" alt="image" src="https://github.com/user-attachments/assets/a7863a98-84f0-484e-aaeb-a12c7dc4d13c" />

<img width="537" height="552" alt="image" src="https://github.com/user-attachments/assets/704d93fa-e0c6-49a1-a837-5993b070b2ef" />

<img width="960" height="579" alt="image" src="https://github.com/user-attachments/assets/aa62459e-80cb-4e1a-8283-3f1a8746e98a" />

<img width="998" height="629" alt="image" src="https://github.com/user-attachments/assets/c1f85100-f8c3-4d7f-b6fe-7297fa09f6f9" />

<img width="1012" height="673" alt="image" src="https://github.com/user-attachments/assets/b6e5abae-3f40-4f5a-8caa-8eecd4e80216" />

<img width="901" height="656" alt="image" src="https://github.com/user-attachments/assets/41cce102-f5e9-4ac7-bfbf-6b1e1751ecd7" />

<img width="922" height="444" alt="image" src="https://github.com/user-attachments/assets/0f6d52d4-6540-440d-887a-577c9d7ce642" />

<img width="944" height="517" alt="image" src="https://github.com/user-attachments/assets/c1f42cf3-0c4d-4a50-a648-627585e88128" />

<img width="962" height="651" alt="image" src="https://github.com/user-attachments/assets/9eebc211-ee98-4ca4-bbd9-7929d6c601e6" />

<img width="805" height="608" alt="image" src="https://github.com/user-attachments/assets/fe218486-25cc-4165-8b6a-9dbc2568ec24" />

![image](https://cdn.hackclub.com/019eeb4b-ec25-7856-8c96-5c8f7ff2fd07/image.png)
![image](https://cdn.hackclub.com/019eeb4c-0bdd-7dd3-bc2c-53d47e2663f9/image.png)
![image](https://cdn.hackclub.com/019eeb4c-2d6b-740d-8e94-8f83c89d2a06/image.png)
![image](https://cdn.hackclub.com/019eeb4c-505f-77ce-a51f-e622e0fa12c4/image.png)
![image](https://cdn.hackclub.com/019eeb4c-7044-77d5-a884-05762f22d162/image.png)
![image](https://cdn.hackclub.com/019eeb4c-920e-7328-abe3-947cd452e62d/image.png)


## BOM
| No. | Quantity | Comment               | Designator                                                                 | Footprint                                    | Value     | Manufacturer Part         | Manufacturer        | Supplier Part | Supplier |
|-----|----------|-----------------------|----------------------------------------------------------------------------|----------------------------------------------|-----------|---------------------------|---------------------|---------------|----------|
| 1   | 6        | 100nF                 | C12,C99,C138,C139,C141,C244                                                | C0402                                        | 100nF     | GRM155R71H104KE14D        | muRata(村田)          | C1523         | LCSC     |
| 2   | 1        | 1uF                   | C14                                                                        | C0402                                        | 1uF       | CL05A105KP5NNNC           | SAMSUNG(三星)         | C15195        | LCSC     |
| 3   | 13       | 100nF                 | C15,C122,C123,C124,C125,C126,C127,C128,C129,C130,C131,C132,C133            | C0402                                        | 100nF     | 0402B104K160NT            | FH(风华)              | C15195        | LCSC     |
| 4   | 2        | 2.2uF                 | C16,C17                                                                    | C0402                                        | 2.2uF     | CL05A225KP5NSNC           | SAMSUNG(三星)         | C15195        | LCSC     |
| 5   | 1        | 10nF                  | C32                                                                        | C0402                                        | 10nF      | CC0402KRX7R9BB103         | YAGEO(国巨)           | C1523         | LCSC     |
| 6   | 1        | 470nF                 | C40                                                                        | C0402                                        | 470nF     | GRM155R61H474KE11D        | muRata(村田)          | C1523         | LCSC     |
| 7   | 2        | 9pF                   | C84,C85                                                                    | C0402                                        | 9pF       | 0402CG9R0C500NT           | FH(风华)              | C1523         | LCSC     |
| 8   | 5        | 10uF                  | C91,C97,C147,C153,C175                                                     | C0603                                        | 10uF      | CL10A106MA8NRNC           | SAMSUNG(三星)         | C19702        | LCSC     |
| 9   | 10       | 100nF                 | C92,C93,C95,C96,C148,C149,C151,C152,C230,C231                              | C0402                                        | 100nF     | GRM155R71H104KE14D        | muRata(村田)          | C15195        | LCSC     |
| 10  | 2        | 10uF                  | C94,C150                                                                   | C0603                                        | 10uF      | GRM188R60J476ME15D        | muRata(村田)          | C19702        | LCSC     |
| 11  | 3        | 2.2uF                 | C100,C115,C140                                                             | C0402                                        | 2.2uF     | CL05A225MQ5NSNC           | SAMSUNG(三星)         | C1523         | LCSC     |
| 12  | 1        | 100nF                 | C101                                                                       | C0402                                        | 100nF     | CL05A225MQ5NSNC           | SAMSUNG(三星)         | C1523         | LCSC     |
| 13  | 1        | 4.7uF                 | C102                                                                       | C0402                                        | 4.7uF     | CL05A475MP5NRNC           | SAMSUNG(三星)         | C1523         | LCSC     |
| 14  | 2        | 2.2uF                 | C106,C242                                                                  | C0402                                        | 2.2uF     | CL05A225MQ5NSNC           | SAMSUNG(三星)         | C12530        | LCSC     |
| 15  | 2        | 100nF                 | C107,C243                                                                  | C0402                                        | 100nF     | CL05B104KB54PNC           | SAMSUNG(三星)         | C307331       | LCSC     |
| 16  | 5        | 100nF                 | C114,C171,C172,C173,C174                                                   | C0402                                        | 100nF     | CC0402KRX7R7BB104         | YAGEO(国巨)           | C1523         | LCSC     |
| 17  | 2        | 12pF                  | C145,C146                                                                  | C0402                                        | 12pF      | GCM1555C1H120FA16D        | muRata(村田)          | C1523         | LCSC     |
| 18  | 2        | 10pF                  | C161,C162                                                                  | C0402                                        | 10pF      | CL05C100JB5NNNC           | SAMSUNG(三星)         | C1523         | LCSC     |
| 19  | 1        | 1uF                   | C163                                                                       | C0402                                        | 1uF       | CL05C100JB5NNNC           | SAMSUNG(三星)         | C1523         | LCSC     |
| 20  | 1        | 470pF                 | C164                                                                       | C0402                                        | 470pF     | CL05C100JB5NNNC           | SAMSUNG(三星)         | C1523         | LCSC     |
| 21  | 5        | 100nF                 | C165,C166,C167,C168,C169                                                   | C0402                                        | 100nF     | CL05C100JB5NNNC           | SAMSUNG(三星)         | C1523         | LCSC     |
| 22  | 1        | 1nF/2KV               | C170                                                                       | C1206                                        | 1nF/2KV   | CC1206KKX7RDBB102         | YAGEO(国巨)           | C9196         | LCSC     |
| 23  | 2        | 22uF                  | C176,C177                                                                  | C0603                                        | 22uF      | CL10A226MO7JZNC           | SAMSUNG(三星)         | C19702        | LCSC     |
| 24  | 1        | 10uF                  | C178                                                                       | C0603                                        | 10uF      | CL10A226MO7JZNC           | SAMSUNG(三星)         | C19702        | LCSC     |
| 25  | 10       | 100nF                 | C179,C180,C183,C184,C189,C192,C238,C239,C240,C241                          | C0402                                        | 100nF     | GRM155R71H104KE14D        | muRata(村田)          | C77020        | LCSC     |
| 26  | 4        | 10nF                  | C185,C186,C187,C188                                                        | C0402                                        | 10nF      | GRM155R71H104KE14D        | muRata(村田)          |               | LCSC     |
| 27  | 2        | 12pF                  | C190,C191                                                                  | C0402                                        | 12pF      | GCM1555C1H120FA16D        | muRata(村田)          | C393682       | LCSC     |
| 28  | 8        | 100nF                 | C199,C200,C201,C202,C203,C204,C245,C246                                    | C0402                                        | 100nF     | GCM1555C1H120FA16D        | muRata(村田)          |               | LCSC     |
| 29  | 1        | 1nF                   | C205                                                                       | C0402                                        | 1nF       | GRM155R71H104KE14D        | muRata(村田)          |               | LCSC     |
| 30  | 5        | 47uF                  | C206,C226,C229,C232,C234                                                   | CAP-SMD_L3.5-W2.8-R-RD                       | 47uF      | CA45-B-16V-47uF-K         | CEC(振华新云)           | C136751       | LCSC     |
| 31  | 5        | 47uF                  | C208,C227,C228,C233,C235                                                   | CASE-A_3216                                  | 47uF      | TLJA476M010R0600          | Kyocera AVX         | C131173       | LCSC     |
| 32  | 15       | NFM18PC104R1C3D       | C211,C212,C213,C214,C215,C216,C217,C218,C219,C220,C221,C222,C223,C224,C225 | FILTER-SMD_3P-L1.6-W0.8-L_NFM18CC223R1C3D    |           | NFM18PC104R1C3D           | muRata(村田)          | C111493       | LCSC     |
| 33  | 2        | 100nF                 | C236,C237                                                                  | C0402                                        | 100nF     | CC0402KRX7R7BB104         | YAGEO(国巨)           | C60474        | LCSC     |
| 34  | 1        | 5033981892            | CARD4                                                                      | TF-SMD_503398-1892                           |           | 5033981892                | MOLEX               | C428492       | LCSC     |
| 35  | 1        | DEBUG                 | CN3                                                                        | conn-smd_8p-p1.25_wafer-gh1.25-8plb          |           | DEBUG                     | XUNPU(讯普)           | C3029374      | LCSC     |
| 36  | 1        | SPI外设                 | CN4                                                                        | conn-smd_8p-p1.25_wafer-gh1.25-8plb          |           | SPI外设                     | XUNPU(讯普)           | C3029374      | LCSC     |
| 37  | 1        | WAFER-GH1.25-7PLB     | CN5                                                                        | CONN-SMD_7P-P1.25_XUNPU_WAFER-GH1.25-7PLB    |           | WAFER-GH1.25-7PLB         | XUNPU(讯普)           | C3029373      | LCSC     |
| 38  | 1        | 1N4148WS              | D1                                                                         | SOD-323_L1.8-W1.3-LS2.5-RD                   |           | 1N4148WS                  | CJ(江苏长电/长晶)         | C2128         | LCSC     |
| 39  | 1        | TISP4350H3BJR-S       | D2                                                                         | SMB_L4.6-W3.6-LS5.3-BI                       |           | TISP4350H3BJR-S           | BOURNS              | C96055        | LCSC     |
| 40  | 1        | DSK34                 | D4                                                                         | SOD-123_L2.8-W1.8-LS3.7-RD                   |           | DSK34                     | FUXINSEMI(富芯森美)     | C908230       | LCSC     |
| 41  | 17       | PZ254V-11-03P         | H1,H2,H3,H4,H5,H6,H7,H8,H9,H10,H11,H12,H13,H14,H15,H16,H17                 | HDR-TH_3P-P2.54-V-M                          |           | PZ254V-11-03P             | XFCN(兴飞)            | C2937625      | LCSC     |
| 42  | 1        | B1603S                | L6                                                                         | SOIC-16_L12.7-W7.1-P1.27-LS9.5-BL            |           | B1603S                    | CND-tek(磁联达)        | C3003215      | LCSC     |
| 43  | 1        | BLM15PD121SN1D        | L7                                                                         | L0402                                        |           | BLM15PD121SN1D            | muRata(村田)          | C76891        | LCSC     |
| 44  | 2        | TC3838RGBE07-3CJH-E19 | LED1,LED2                                                                  | LED-SMD_6P-L3.5-W3.5-BL                      |           | TC3838RGBE07-3CJH-E19     | TCWIN(天成)           | C784547       | LCSC     |
| 45  | 10       | 10kΩ                  | R3,R86,R87,R142,R143,R229,R230,R240,R242,R247                              | R0402                                        | 10kΩ      | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 46  | 1        | ICM-20689             | R4                                                                         | QFN-24_L4.0-W4.0-P0.50-TL-EP2.6              |           | ICM-20689                 | TDK InvenSense(应美盛) | C2649494      | LCSC     |
| 47  | 1        | 220Ω                  | R8                                                                         | R0402                                        | 220Ω      | 0402WGF2200TCE            | UNI-ROYAL(厚声)       | C25091        | LCSC     |
| 48  | 5        | 10kΩ                  | R22,R23,R24,R25,R26                                                        | R0402                                        | 10kΩ      | 0402WGF1002TCE            | UNI-ROYAL(厚声)       | C25744        | LCSC     |
| 49  | 2        | 120Ω                  | R56,R157                                                                   | R0402                                        | 120Ω      | 0402WGF1200TCE            | UNI-ROYAL(厚声)       | C25079        | LCSC     |
| 50  | 2        | 10kΩ                  | R66,R231                                                                   | R0402                                        | 10kΩ      | RC0402FR-0710KL           | YAGEO(国巨)           | C60490        | LCSC     |
| 51  | 2        | 1.5K                  | R76,R119                                                                   | R0402                                        | 1.5K      | 0402WGF4701TCE            | UNI-ROYAL(厚声)       | C25091        | LCSC     |
| 52  | 1        | 1kΩ                   | R89                                                                        | R0402                                        | 1kΩ       | 0402WGF1001TCE            | UNI-ROYAL(厚声)       | C11702        | LCSC     |
| 53  | 1        | 1kΩ                   | R120                                                                       | R0402                                        | 1kΩ       | 0402WGF1001TCE            | UNI-ROYAL(厚声)       | C25091        | LCSC     |
| 54  | 2        | 5.1kΩ                 | R127,R128                                                                  | R0402                                        | 5.1kΩ     | 0402WGF5101TCE            | UNI-ROYAL(厚声)       | C25905        | LCSC     |
| 55  | 2        | 22Ω                   | R129,R130                                                                  | R0402                                        | 22Ω       | 0402WGF220JTCE            | UNI-ROYAL(厚声)       | C25092        | LCSC     |
| 56  | 1        | 0Ω                    | R135                                                                       | R0402                                        | 0Ω        | 0402WGF0000TCE            | UNI-ROYAL(厚声)       | C25091        | LCSC     |
| 57  | 4        | 1kΩ                   | R138,R153,R154,R155                                                        | R0402                                        | 1kΩ       | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 58  | 1        | 12.1K                 | R140                                                                       | R0402                                        | 12.1K     | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 59  | 1        | 1M                    | R141                                                                       | R0402                                        | 1M        | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 60  | 4        | 49.9R                 | R144,R145,R146,R147                                                        | R0402                                        | 49.9R     | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 61  | 4        | 2R                    | R148,R149,R150,R151                                                        | R0402                                        | 2R        | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 62  | 1        | 75R                   | R152                                                                       | R0402                                        | 75R       | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 63  | 2        | 10K                   | R156,R158                                                                  | R0402                                        | 10K       | 0402WGF1200TCE            | UNI-ROYAL(厚声)       |               | LCSC     |
| 64  | 2        | 1kΩ                   | R163,R164                                                                  | R0402                                        | 1kΩ       | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 65  | 2        | 2M                    | R165,R166                                                                  | R0402                                        | 2M        | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 66  | 11       | 10K                   | R167,R168,R169,R170,R171,R172,R174,R176,R179,R236,R237                     | R0402                                        | 10K       | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 67  | 1        | 20K                   | R175                                                                       | R0402                                        | 20K       | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 68  | 3        | 1K                    | R181,R182,R183                                                             | R0402                                        | 1K        | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 69  | 8        | 1.5K                  | R184,R185,R188,R189,R192,R193,R196,R197                                    | R0402                                        | 1.5K      | 0402WGF4701TCE            | UNI-ROYAL(厚声)       |               | LCSC     |
| 70  | 8        | 100R                  | R186,R187,R190,R191,R194,R195,R198,R199                                    | R0402                                        | 100R      | 0402WGF4701TCE            | UNI-ROYAL(厚声)       |               | LCSC     |
| 71  | 1        | 10K                   | R202                                                                       | R0402                                        | 10K       | 0402WGF4701TCE            | UNI-ROYAL(厚声)       |               | LCSC     |
| 72  | 1        | 1K                    | R203                                                                       | R0402                                        | 1K        | 0402WGF4701TCE            | UNI-ROYAL(厚声)       |               | LCSC     |
| 73  | 10       | 220R                  | R204,R207,R208,R211,R212,R213,R214,R215,R216,R219                          | R0402                                        | 220R      | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 74  | 2        | 100R                  | R205,R206                                                                  | R0402                                        | 100R      | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 75  | 1        | 1.5K                  | R209                                                                       | R0402                                        | 1.5K      | RC0402FR-0710KL           | YAGEO(国巨)           |               | LCSC     |
| 76  | 1        | 10K                   | R210                                                                       | R0402                                        | 10K       | 0402WGF0000TCE            | UNI-ROYAL(厚声)       | C25091        | LCSC     |
| 77  | 9        | 100Ω                  | R220,R221,R222,R223,R224,R225,R226,R234,R235                               | R0402                                        | 100Ω      | 0402WGF1000TCE            | UNI-ROYAL(厚声)       | C25076        | LCSC     |
| 78  | 2        | 4.7kΩ                 | R227,R228                                                                  | R0402                                        | 4.7kΩ     | 0402WGF4701TCE            | UNI-ROYAL(厚声)       | C25091        | LCSC     |
| 79  | 2        | 100kΩ                 | R232,R233                                                                  | R0402                                        | 100kΩ     | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 80  | 1        | 100kΩ                 | R238                                                                       | R0402                                        | 100kΩ     | 0402WGF1002TCE            | UNI-ROYAL(厚声)       |               | LCSC     |
| 81  | 1        | 10K                   | R239                                                                       | R0402                                        | 10K       | 0402WGF4701TCE            | UNI-ROYAL(厚声)       | C25091        | LCSC     |
| 82  | 1        | 220R                  | R241                                                                       | R0402                                        | 220R      | RC0402FR-0710KL           | YAGEO(国巨)           | C25091        | LCSC     |
| 83  | 6        | 100Ω                  | RN1,RN2,RN3,RN4,RN5,RN6                                                    | RES-ARRAY-SMD_0603-8P-L3.2-W1.6-BL           | 100Ω      | 4D03WGJ0101T5E            | UNI-ROYAL(厚声)       | C25506        | LCSC     |
| 84  | 1        | SKRPACE010            | SW4                                                                        | KEY-SMD_4P-L4.2-W3.2-P2.20-LS4.6             |           | SKRPACE010                | ALPSALPINE(阿尔卑斯阿尔派) | C139797       | LCSC     |
| 85  | 1        | ML414H-IV01E          | U4                                                                         | BAT-SMD_ML414H-IV01E                         |           | ML414H-IV01E              | Seiko(精工)           | C167511       | LCSC     |
| 86  | 1        | STM32H743IIK6         | U6                                                                         | UFBGA-201_L10.0-W10.0-P0.65-TL_STM32H743IIK6 |           | STM32H743IIK6             | ST(意法半导体)           | C129442       | LCSC     |
| 87  | 1        | ICM-42688-P           | U30                                                                        | LGA-14_L3.0-W2.5-P0.50-TL                    |           | ICM-42688-P               | TDK InvenSense(应美盛) | C1850418      | LCSC     |
| 88  | 1        | IST8310               | U37                                                                        | LGA-16_L3.0-W3.0-P0.5-BL                     |           | IST8310                   |                     | C9900047266   | LCSC     |
| 89  | 1        | BMI088                | U56                                                                        | LGA-16_L4.5-W3.0-P0.50-BL                    |           | BMI088                    | Bosch(博世)           | C194919       | LCSC     |
| 90  | 2        | MIC5330-SSYML-TR      | U58,U91                                                                    | MLF-8_L2.0-W2.0-P0.50-BL-EP                  |           | MIC5330-SSYML-TR          | MICROCHIP(美国微芯)     | C144168       | LCSC     |
| 91  | 1        | FM25V05/FM25V02       | U61                                                                        | SOIC-8_L4.9-W3.9-P1.27-LS6.0-BL              |           | FM25V05/FM25V02           | CYPRESS(赛普拉斯)       | C2953554      | LCSC     |
| 92  | 1        | 供电1                   | U65                                                                        | CONN-SMD_GH1.25-6PLTPZ                       |           | 供电1                       | BOOMELE(博穆精密)       | C2829264      | LCSC     |
| 93  | 1        | 供电2                   | U66                                                                        | CONN-SMD_GH1.25-6PLTPZ                       |           | 供电2                       | BOOMELE(博穆精密)       | C2829264      | LCSC     |
| 94  | 1        | CAN供电1                | U67                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | CAN供电1                    | XUNPU(讯普)           | C3029370      | LCSC     |
| 95  | 1        | CAN1                  | U68                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | CAN1                      | XUNPU(讯普)           | C3029370      | LCSC     |
| 96  | 1        | GPS1                  | U69                                                                        | CONN-SMD_GH1.25-6PLTPZ                       |           | GPS1                      | BOOMELE(博穆精密)       | C2829264      | LCSC     |
| 97  | 1        | GPS2                  | U70                                                                        | CONN-SMD_GH1.25-6PLTPZ                       |           | GPS2                      | BOOMELE(博穆精密)       | C2829264      | LCSC     |
| 98  | 1        | 电台1                   | U71                                                                        | CONN-SMD_GH1.25-6PLTPZ                       |           | 电台1                       | BOOMELE(博穆精密)       | C2829264      | LCSC     |
| 99  | 1        | 电台2                   | U72                                                                        | CONN-SMD_GH1.25-6PLTPZ                       |           | 电台2                       | BOOMELE(博穆精密)       | C2829264      | LCSC     |
| 100 | 1        | I2C1                  | U73                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | I2C1                      | XUNPU(讯普)           | C3029370      | LCSC     |
| 101 | 1        | I2C2                  | U74                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | I2C2                      | XUNPU(讯普)           | C3029370      | LCSC     |
| 102 | 1        | I2C3                  | U75                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | I2C3                      | XUNPU(讯普)           | C3029370      | LCSC     |
| 103 | 1        | I2C4                  | U76                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | I2C4                      | XUNPU(讯普)           | C3029370      | LCSC     |
| 104 | 1        | 以太网                   | U77                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | 以太网                       | XUNPU(讯普)           | C3029370      | LCSC     |
| 105 | 1        | CAN供电                 | U79                                                                        | CONN-SMD_4P-P1.25_XUNPU_WAFER-GH1.25-4PLB    |           | CAN供电                     | XUNPU(讯普)           | C3029370      | LCSC     |
| 106 | 1        | BMP581                | U87                                                                        | LGA-10_L2.0-W2.0-P0.50-TL_BMP581             |           | BMP581                    | Bosch(博世)           | C5362283      | LCSC     |
| 107 | 1        | 16MHz                 | U90                                                                        | CRYSTAL-SMD_4P-L2.0-W1.6-BL                  | 16MHz     | X201616MOB4SI             | YXC(扬兴晶振)           | C19711739     | LCSC     |
| 108 | 1        | LAN8742A              | U93                                                                        | QFN24/0.5/4X4                                |           |                           |                     | C621424       |          |
| 109 | 1        | 25MHz                 | U94                                                                        | CRYSTAL-SMD_4P-L2.0-W1.6-BL                  | 25MHz     | XL7EL89CKI-111YLC-25M     | YXC(扬兴晶振)           | C19711739     | LCSC     |
| 110 | 2        | CDSOD323-T15SC        | U95,U96                                                                    | SOD-323_L1.8-W1.3-LS2.6-BI                   |           | CDSOD323-T15SC            | BOURNS              | C3704064      | LCSC     |
| 111 | 1        | ICP-20100             | U97                                                                        | LGA-10_L2.0-W2.0-P0.50-TL_ICP-20100          |           | ICP-20100                 | TDK InvenSense(应美盛) | C5343527      | LCSC     |
| 112 | 1        | LP5912-3.3DRVT        | U204                                                                       | WSON-6_L2.0-W2.0-P0.65-TL-EP                 |           | LP5912-3.3DRVT            | TI(德州仪器)            | C134121       | LCSC     |
| 113 | 2        | TJA1051TK/3/1J        | U205,U206                                                                  | HVSON-8_L3.0-W3.0-P0.65-BL-EP                |           | TJA1051TK/3/1J            | NXP(恩智浦)            | C2875699      | LCSC     |
| 114 | 3        | TXS0108ERGYR          | U214,U215,U235                                                             | VQFN-20_L4.6-W3.6-P0.50-BL-EP_WU             |           | TXS0108ERGYR              | Texas Instruments   | C90706        | LCSC     |
| 115 | 1        | SN74LVC8T245RHLR      | U217                                                                       | VQFN-24_L5.6-W3.6-P0.50-BL-EP                |           | SN74LVC8T245RHLR          | TI(德州仪器)            | C350563       | LCSC     |
| 116 | 2        | SN74LVC1G240DBVR      | U219,U220                                                                  | SOT-23-5_L3.0-W1.7-P0.95-LS2.8-BR            |           | SN74LVC1G240DBVR          | TI(德州仪器)            | C7837         | LCSC     |
| 117 | 1        | 42pF                  | U221                                                                       | SOT-563_L1.6-W1.2-P0.50-LS1.6-TL             | 42pF      | NUF2042XV6T1G             | onsemi(安森美)         | C90707        | LCSC     |
| 118 | 1        | DSM/RSSI/SBUS_OUTPUT  | U223                                                                       | CONN-SMD_GH1.25-6PLTPZ                       |           | DSM/RSSI/SBUS_OUTPUT      | BOOMELE(博穆精密)       | C2829264      | LCSC     |
| 119 | 1        | MF-PSMF050X-2         | U224                                                                       | F0805                                        |           | MF-PSMF050X-2             | JK(金科)              | C5307781      | LCSC     |
| 120 | 2        | AP2311FGEG-7          | U226,U227                                                                  | U-DFN3030-8_L3.0-W3.0-P0.65-BL-EP            |           | AP2311FGEG-7              | DIODES(美台)          | C526365       | LCSC     |
| 121 | 1        | NX3DV42GU10X          | U228                                                                       | XQFN-10_L1.6-W1.3-P0.40-BL                   |           | NX3DV42GU10X              | NXP(恩智浦)            | C2151046      | LCSC     |
| 122 | 1        | NX3DV42GU,115         | U229                                                                       | XQFN-10_L1.6-W1.3-P0.40-BL                   |           | NX3DV42GU,115             | NXP(恩智浦)            | C2151046      | LCSC     |
| 123 | 1        | STM32F103CBU6         | U231                                                                       | VFQFPN48_N                                   |           |                           |                     |               |          |
| 124 | 1        | TYPE-C 16PIN 2MD(073) | USB2                                                                       | USB-C-SMD_TYPE-C-6PIN-2MD-073                |           | TYPE-C 16PIN 2MD(073)     | SHOU HAN(首韩)        | C2765186      | LCSC     |
| 125 | 1        | 32.768kHz             | X3                                                                         | CRYSTAL-SMD_2P-L2.0-W1.2                     | 32.768kHz | XKHEL89CGI-112YLC-32.768K | YXC(扬兴晶振)           | C7500573      | LCSC     |
| 126 | 1        | 8MHz                  | X4                                                                         | CRYSTAL-SMD_4P-L2.0-W1.6-BL-B                | 8MHz      | XRCGB24M000F3A00R0        | muRata(村田)          |               | LCSC     |
|     |          |                       |                                                                            |                                              |           |                           |                     |               |          |


#### Final cost-
<img width="1541" height="815" alt="image" src="https://github.com/user-attachments/assets/609ad079-e54d-4570-99b9-c5141db9ad2c" />
<img width="1491" height="684" alt="image" src="https://github.com/user-attachments/assets/198a9bbf-b6b9-4b43-9eb2-21bf3ad30d20" />

= ~170$
