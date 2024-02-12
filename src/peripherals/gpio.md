# GPIO 

The GPIO functionality in the SHAKTI processor is a versatile feature, enabling the generation of custom waveforms, activation of signals, and handling interrupts. Here are additional details about the GPIO operation:
## GPIO Registers

- **GPIO DIRECTION Register**:
    The GPIO DIRECTION register configures the GPIO pin as either an input or output. This register plays a crucial role in determining the operational mode of the GPIO pins.

- **GPIO DATA Register**:
    The GPIO DATA register holds the input data from GPIO or the output data to GPIO. It is accessible in 1-byte, 2-byte, and 4-byte sizes, providing flexibility in data manipulation.

- **Interrupt Handling (GPIO Pins 0 - 7)**:
    GPIO pins 0 - 7 can accept external events as interrupts. To use a GPIO pin as an interrupt source, the respective GPIO pin(s) must be configured as inputs. This feature enhances the processor's ability to respond to external stimuli.

**GPIO Registers**:

- **GPIO_DIRECTION_CNTRL_REG (Offset 'h00)**:
    This 32-bit register controls the direction of GPIO pins, allowing configuration as either input or output. It supports both read and write operations.

- **GPIO_DATA_REG (`h08)**:
    This register, accessible in 32, 16, or 8 bits, serves as the GPIO Data Register. It supports read and write operations, facilitating data handling for GPIO pins.

- **Sequence of Execution**:

    - To effectively utilize the GPIO functionality, follow this sequence:
        - Write into the GPIO Direction register (GPIO_DIRECTION_CNTRL_REG) to configure the GPIO pin as an input or output.
        - Write appropriate values to the GPIO Data register (GPIO_DATA_REG) based on the desired input or output data.

    These steps ensure proper configuration and operation of the GPIO pins, making them valuable for a wide range of applications in signal processing and interrupt-driven scenarios.

### Rust GPIO Register implmentation ###

```

use crate::common::MMIODerefWrapper;
use riscv::{
    asm::{delay, nop},
    register,
};
use tock_registers::{
    interfaces::{Readable, Writeable},
    register_bitfields, register_structs,
    registers::{ReadOnly, ReadWrite, WriteOnly},
};

//--------------------------------------------------------------------------------------------------
// Private Definitions
//--------------------------------------------------------------------------------------------------

pub const GPIO_OFFSET: usize = 0x0004_0100;

register_structs! {
    #[allow(non_snake_case)]
    pub RegistersBlock{
        (0x00 => DIRECTION_CR_REG: ReadWrite<u32>),
        (0x04 => _reserved0),
        (0x08 => DATA_REG: ReadWrite<u32>),
        (0x0C => @END),
    }
}

register_bitfields! {
    u32,

    DIRECTION_CR_REG [
        CR OFFSET(0) NUMBITS(32) []
    ],
    SCL[
        DATA_REG OFFSET(0) NUMBITS(32) []
    ],

}

type Registers = MMIODerefWrapper<RegistersBlock>;

pub struct GPIOInner {
    registers: Registers,
}


```



- **Memory-Mapped I/O (MMIO)**:

use crate::common::MMIODerefWrapper: Imports a wrapper for MMIO, which is a technique for interfacing with hardware registers by mapping them to specific memory addresses.

- **RISC-V Assembly and Registers**:

use riscv::{asm::{delay, nop}, register}: Imports RISC-V assembly instructions (delay, nop) and register-related functionality. RISC-V is an instruction set architecture commonly used in embedded systems.

- **Tock Registers**:

use tock_registers::{interfaces::{Readable, Writeable}, register_bitfields, register_structs, registers::{ReadOnly, ReadWrite, WriteOnly}};: Imports Tock Registers, a library for working with hardware registers in embedded systems. It provides traits for reading and writing registers, as well as utilities for defining register bitfields.

- **GPIO Offset**:

pub const GPIO_OFFSET: usize = 0x0004_0100;: Defines the offset for GPIO registers in memory.

- **Register Block Definition**:

register_structs! { ... }: Defines a block of memory-mapped registers for GPIO. This includes the Direction Control Register (DIRECTION_CR_REG) and Data Register (DATA_REG), specifying their offsets and types.

- **Register Bitfields**:

register_bitfields! { ... }: Defines specific bitfields within the registers, such as the CR field in the Direction Control Register and the DATA_REG field in the Data Register.

- **Registers Wrapper**:

type Registers = MMIODerefWrapper<RegistersBlock>;: Defines a type Registers that wraps the MMIO registers, providing a safe interface for accessing and modifying them.

- **GPIOInner Structure**:

pub struct GPIOInner { registers: Registers, }: Defines a structure encapsulating the GPIO registers. This structure is likely used to abstract GPIO operations and provide a more organized interface.