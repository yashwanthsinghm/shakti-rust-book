**SHAKTI**, a groundbreaking RISC-V-based processor, introduces the E and C-classes—representing the initial set of indigenous designs tailored for the Internet of Things (IoT), Embedded Systems, and Desktop markets. Emphasizing an open and collaborative approach, the processor design is devoid of royalties and released under the BSD3 license. SHAKTI's E and C-classes provide a foundation for innovative processor architectures with diverse applications.

## E-Class ##
The 32-bit E-Class microprocessor is designed for low-power compute environments, automotive, and IoT applications. Operating on an in-order 3-stage pipeline below 200MHz, it supports all RISC-V ISA extensions. Noteworthy for running Real-Time Operating Systems (RTOS) like Zephyr OS and FreeRTOS, the E-Class competes with ARM's M-class cores. Key applications include smart cards, motor controls, and home automation.

**PINAKA (E32-A35)**:
PINAKA, built around the E-Class, is a 32-bit microcontroller with 4KB ROM and 128KB BRAM. Featuring 32 GPIO pins (8 for onboard LEDs and switches), it includes a Platform Level Interrupt Controller (PLIC), Timer (CLINT), 2 SPI, 3 UART, 2 I2C, 6 PWM, Xilinx ADC, Soft Float library, Physical Memory Protection (PMP), an onboard FTDI-based debugger, and Pin Mux support with Arduino-compatible assignments.

**PARASHU (E32-A100)**:
Sharing the E-Class configuration, PARASHU is a 32-bit microcontroller with 4KB ROM and 256MB DDR. Suited for various applications, it provides additional memory capacity.

## C-Class ##
The 64-bit C-Class, a 6-stage microcontroller, supports the entire RISC-V ISA. Customizable for 200-800MHz mid-range compute systems, it competes with ARM's Cortex A35/A55. Compatible with Linux, SEL4, and FreeRTOS, the C-Class caters to both low-power and high-performance variants.

**VAJRA (C64-A100)**:
VAJRA, built around the C-Class, is a single-chip 64-bit microcontroller with 4KB ROM and 256MB DDR3 RAM. Targeted at mid-range applications like industrial controllers and the desktop market, VAJRA shares a configuration with PARASHU.

## Peripherals ##

| Peripherals   | info                            |
|---------------|---------------------------------|
| GPIO Pins     | 32                              |
| Upper         | 8 pins Onboard LEDs and switches|
| PLIC          | 1                               |
| Counter       | 1                               |
| SPI           | 2                               |
| UART          | 3                               |
| I2C           | 2                               |
| PWM           | 6                               |
| ADC           | Xilinx                          |
| CLINT         | 1                               |
| Float         | Soft library                    |
| PMP           | Enabled                         |
| Debugger      | Onboard FTDI based              |
| Pin Mux       | Yes                             |
| Ethernet lite | PARASHU, VAJRA                  |