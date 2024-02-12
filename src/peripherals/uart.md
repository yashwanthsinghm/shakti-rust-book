# UART 

The UART module in the Shakti SoC (System on Chip) provides asynchronous serial communication with RS-232 (RS-422/485) interface. It operates in a non-return-to-zero (NRZ) mode, supporting data transfer rates that can be configured through the UARTBAUD register. Here's a concise overview of key UART registers and their functionalities:

### UART Registers ###
1. **BAUD Register (Address: 0x00)**
Configures the Baud rate based on the set Baud value.
Baud Rate calculation: Baud_value = (Clock Frequency) / (16 * Baudrate)
2. **Status Register (Address: 0x0C)**
Provides status information about transmit and receive operations.
Break, frame, overrun, and parity error flags.
Flags indicating receiver and transmitter buffer status.
3. **Control Register (Address: 0x14)**
Configures UART operation, character size, parity, and stop bits.
Character size (data length), parity, and stop bits settings.
4. **TX_REG Register (Address: 0x04)**
Write-only register for transmitting data. Data to be transmitted is written into this register.
5. **RCV_REG Register (Address: 0x08)**
Read-only register for receiving data. The received value can be read from this register.
6. **IEN Register (Address: 0x18)**
Enables interrupts based on the set bit values in the Interrupt Enable Register.
Interrupts for various conditions like threshold, break error, frame error, overrun, parity error, and buffer status.
7. **DELAY Register (Address: 0x10)**
Controls delayed transmit by specifying the required delay count.
8. **IQCYCLES Register (Address: 0x1C)**
Holds the number of input qualification cycles for the receiver pin.
9. **RX_THRESHOLD Register (Address: 0x20)**
Holds the receiver FIFO threshold value. Generates corresponding status bits and interrupts when the FIFO level crosses the threshold.
These registers collectively define the configuration, status, and control aspects of the UART module in the Shakti SoC, facilitating efficient and configurable serial communication.


### Rust UART Register implmentation ###

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

pub const UART_OFFSET: usize = 0x0001_1300;
pub const STS_TX_FULL_FLAG: u8 = 0x02;
pub const STS_RX_NOT_EMPTY_FLAG: u8 = 0x08;

pub const BREAK_ERROR: u8 = 1 << 7;
pub const FRAME_ERROR: u8 = 1 << 6;
pub const OVERRUN: u8 = 1 << 5;
pub const PARITY_ERROR: u8 = 1 << 4;
pub const STS_RX_FULL: u8 = 1 << 3;
pub const STS_RX_NOT_EMPTY: u8 = 1 << 2;
pub const STS_TX_FULL: u8 = 1 << 1;
pub const STS_TX_EMPTY: u8 = 1 << 0;

register_structs! {
    #[allow(non_snake_case)]
    pub RegistersBlock{
        (0x00 => UBR: ReadWrite<u16>),
        (0x02 => _reserved0),
        (0x04 => TX_REG: WriteOnly<u32>),
        (0x08 => RCV_REG: ReadOnly<u32, RCV_REG::Register>),
        (0x0C => USR : ReadOnly<u8, USR::Register>),
        (0x0D => _reserved1),
        //(0x0E => _reserved2),
        (0x10 => DELAY: ReadWrite<u32, DELAY::Register>),
        // (0x12 => _reserved3),
        (0x14 => UCR: ReadWrite<u32, UCR::Register>),
        // (0x16 => _reserved4),
        (0x18 => IEN: ReadWrite<u32, IEN::Register>),
        // (0x1A => _reserved5),
        (0x1C => IQCYCLES: ReadWrite<u32, IQCYCLES::Register>),
        // (0x1D => _reserved6),
        // (0x1E => _reserved7),
        // (0x1F => _reserved11),
        (0x20 => RX_THRESHOLD: WriteOnly<u32, RX_THRESHOLD::Register>),
        // (0x21 => _reserved8),
        // (0x22 => _reserved9),
        // (0x23 => _reserved10),
        (0x24 => @END),
    }
}

register_bitfields! {
    u32,

    UBR [
        BAUD OFFSET(0) NUMBITS(16) []
    ],

    /// UART Status register
    USR [
        /// Transmit FIFO empty. The meaning of this bit depends on the state of the FEN bit in the
        /// Line Control Register, LCR_H.
        ///
        /// - If the FIFO is disabled, this bit is set when the transmit holding register is empty.
        /// - If the FIFO is enabled, the TXFE bit is set when the transmit FIFO is empty.
        /// - This bit does not indicate if there is data in the transmit shift register.
        BREAK_ERROR OFFSET(7) NUMBITS(1) [],
        ///Break Error (Sets when the data and stop are both zero
        FRAME_ERROR OFFSET(6) NUMBITS(1) [],
        ///Frame Error (Sets when the stopis zero)
        OVERRUN OFFSET(5) NUMBITS(1) [],
        ///Overrun Error (A data overrun error occurred in the receive
        ///shift register. This happens when additional data arrives
        ///while the FIFO is full. )
        PARITY_ERROR OFFSET(4) NUMBITS(1) [],
        ///Parity Error (Sets when The receive character does not
        ///have correct parity information and is suspect.

        STS_RX_FULL OFFSET(3) NUMBITS(1) [],
        ///Receiver Full (Sets when the Receive Buffer is Full)

        STS_RX_NOT_FULL OFFSET(2) NUMBITS(1) [],
        /// Receiver Not Empty (Sets when there is some data in the
        ///Receive Buffer).
        STS_TX_FULL OFFSET(1) NUMBITS(1) [
            EMPTY = 0,
            FULL = 1,
        ],
        ///Transmitter Full (Sets when the transmit Buffer is full)
        STS_TX_EMPTY OFFSET(0) NUMBITS(1) [
            EMPTY = 1,
            FULL = 0,
        ]
        //Transmitter Empty(Sets when the Transmit Buffer is empty).
    ],

    UCR [
        UART_TX_RX_LEN OFFSET(5) NUMBITS(6) [],
        //Character size of data. Maximum length is 32 bits.
        PARITY OFFSET(3) NUMBITS(2) [
            None = 0b00,
            Odd = 0b01,
            Even = 0b10,
            Unused = 0b11

        ],
         //Insert Parity bits
         //00 - None
         //01 - Odd
        //10- Even
        // 11 - Unused or Undefined
        STOP_BITS OFFSET(1) NUMBITS(2) [

            StopBits1 = 0b00,
            StopBits1_5 = 0b01,
            StopBits2 = 0b10


        ],
        //Stop bits
       //00 - 1 Stop bits
        //01 - 1.5 Stop bits
        //10 - 2 Stop bits
    ],

    TX_REG [

        TX_DATA OFFSET(0) NUMBITS(32) []
    ],


    RCV_REG [

        RX_DATA OFFSET(0) NUMBITS(32) []
    ],

    IEN [

        ENABLE_TX_EMPTY OFFSET(0) NUMBITS(1) [],
        ENABLE_TX_FULL OFFSET(1) NUMBITS(1) [],
        ENABLE_RX_NOT_EMPTY OFFSET(2) NUMBITS(1) [],
        ENABLE_RX_FULL OFFSET(3) NUMBITS(1) [],
        ENABLE_PARITY_ERROR OFFSET(4) NUMBITS(1) [],
        ENABLE_OVERRUN OFFSET(5) NUMBITS(1) [],
        ENABLE_FRAME_ERROR OFFSET(6) NUMBITS(1) [],
        ENABLE_BREAK_ERROR OFFSET(7) NUMBITS(1) [],
        ENABLE_RX_THRESHOLD OFFSET(8) NUMBITS(1) []
    ],
///Delayed Transmit control is done by providing the required delay in UART DELAY register.
      DELAY [
        COUNT OFFSET(0) NUMBITS(8) []
      ],
///UART IQCYCLES Register holds the number of input qualification cycles for the receiver pin. .
      IQCYCLES[
        COUNT OFFSET(0) NUMBITS(8) []
      ],
    RX_THRESHOLD [
        /// UART RX_THRESHOLD register holds the receiver FIFO threshold value, when the RX FIFO
        ///level increases beyond the threshold, corresponding status bit will be set and when interrupt is
       ///enabled, interrupt will be raised
        FIFO_RX OFFSET(0) NUMBITS(8) []
    ]

}
/// Abstraction for the associated MMIO registers.
type Registers = MMIODerefWrapper<RegistersBlock>;

pub struct UartInner {
    registers: Registers,
}


```


- **Dependencies**:

The code relies on the MMIODerefWrapper from the common module.
It uses various modules and traits from the riscv and tock_registers crates.

- **Register Definitions**:

The RegistersBlock struct defines memory-mapped I/O (MMIO) registers for the UART module.
Register addresses and their offsets are specified, along with their bitfields.

- **Constants**:

UART_OFFSET: Specifies the offset for the UART module in memory.
Flags and error constants such as STS_TX_FULL_FLAG, STS_RX_NOT_EMPTY_FLAG, BREAK_ERROR, etc., are defined.

- **Register Bitfields**:

The register_bitfields! macro is used to define bitfields for various registers.
It provides convenient access to specific bits within the registers.
- **Register Types**:

Read, Write, and ReadWrite traits are implemented for certain register types, indicating their accessibility.

- **Structs**:

RegistersBlock: Represents the MMIO registers for the UART module.
UartInner: Encapsulates the inner workings of the UART, including access to MMIO registers.

- **Struct Initialization**:

Registers: A wrapper around RegistersBlock, providing safe access to the MMIO registers.
UartInner: Contains a Registers instance.

This code essentially establishes a clear abstraction for working with the UART module in the Shakti RISC-V processor, allowing developers to interact with the UART hardware through safe and well-defined Rust abstractions.