# SPI

The SPI (Serial Peripheral Interface) is a synchronous serial I/O (Input/Output) port commonly used for communication between microcontrollers and peripheral devices. Here's a breakdown of the SPI registers for a Shakti RISC-V processor:

## SPI Registers ##
- **SPI_CR1 (Offset: 'h00)**   : 32 bits, Read and Write - SPI Control Register 1.
- **SPI_CR2 (Offset: 'h04)**   : 32 bits, Read and Write - SPI Control Register 2.
- **SPI_SR (Offset: 'h08)**    : 32 bits, Read Only - SPI Status Register.
- **SPI_DR1 (Offset: 'h0C)**   : 32 bits, Read and Write - SPI Data Register 1.
- **SPI_DR2 (Offset: 'h10)**   : 32 bits, Read and Write - SPI Data Register 2.
- **SPI_DR3 (Offset: 'h14)**   : 32 bits, Read and Write - SPI Data Register 3.
- **SPI_DR4 (Offset: 'h18)**   : 32 bits, Read and Write - SPI Data Register 4.
- **SPI_DR5 (Offset: 'h1C)**   : 32 bits, Read and Write - SPI Data Register 5.
- **SPI_CRCPR (Offset: 'h20)** : 32 bits, Reserved - SPI CRC Polynomial Register.
- **SPI_RXCRCR (Offset: 'h24)**: 32 bits, Reserved - SPI RX CRC Register.
- **SPI_TXCRCR (Offset: 'h28)**: 32 bits, Reserved - SPI TX CRC Register.

These registers control various aspects of the SPI module, including data transfer, CRC (Cyclic Redundancy Check), and status information. Developers can interact with these registers to configure and manage SPI communication in their applications.


### Rust SPI Register implmentation ###

```

use riscv::asm::delay;
use tock_registers::{
    fields::FieldValue,
    interfaces::{ReadWriteable, Readable, Writeable},
    register_bitfields, register_structs,
    registers::{ReadOnly, ReadWrite, WriteOnly},
};

use crate::common::MMIODerefWrapper;

use self::SPI_CR1::SPI_TOTAL_BITS_TX;

pub const SPI_OFFSET: usize = 0x0002_0000;

register_bitfields! {
    u32,

    SPI_CR1 [

        /// Expected Total Number of bits to be received.
        SPI_TOTAL_BITS_RX OFFSET(24) NUMBITS(8) [

        ],

        /// Total Number of bits to be transmitted.
        SPI_TOTAL_BITS_TX OFFSET(16) NUMBITS(8) [

        ],

        /// Bidirectional data mode enable. This bit enables
        /// half-duplex communication using a common single
        /// bidirectional data line.
        /// 0: 2-line unidirectional data mode selected
        /// 1: 1-line unidirectional data mode selected
        SPI_BIDIMODE OFFSET(15) NUMBITS(1) [
            OUTPUT_ENABLE = 1,
            PUTPUT_DISABLE = 0,
        ],

        /// Output enable in bidirectional mode This bit combined with
        /// the BIDI-MODE bit selects the direction of transfer in bi-direction mode.
        /// 0: receive-only mode (Output Disabled)
        /// 1: transmit-only mode (Output Enabled)
        SPI_BIDIODE OFFSET(14) NUMBITS(1) [
            OUTPUT_ENABLE = 1,
            PUTPUT_DISABLE = 0,
        ],


        /// Hardware CRC calculation Enable.
        /// 0: CRC calculation disable
        /// 1: CRC calculation enable
        SPI_CRCEN OFFSET(13) NUMBITS(1) [
            HWCRC_ENABLE = 1,
            HWCRC_DISABLE = 0,
        ],

        /// Transmit CRC Next.
        /// 0: Next Transmit value is from Tx buffer
        /// 1: Next Transmit value is from Rx buffer
        SPI_CCRCNEXT OFFSET(12) NUMBITS(1) [
            CRCNEXT_RX = 1,
            CRCNEXT_TX = 0,
        ],

        /// CRC Length bit is set and cleared by software to select CRC Length
        SPI_CRCL OFFSET(11) NUMBITS(1) [

        ],

        /// Receive only mode enabled. This bit enables simplex
        /// communication using a single unidirectional line to
        /// receive data exclusively. Keep BIDIMODE bit clear when
        /// receiving the only mode is active.
        SPI_RXONLY OFFSET(10) NUMBITS(1) [

        ],

        /// Software Slave Management. When the SSM bit is set,
        /// the NSS pin input is replaced with the value from the SSI
        /// bit.
        /// 0: Software slave management disabled
        /// 1: Software slave management enabled
        SPI_SSM OFFSET(9) NUMBITS(1) [
            SSM_ENABLED  = 1,
            SSM_DISABLED = 0,
        ],

        /// Internal Slave Select.This bit has an effect only when the
        /// SSM bit is set. The value of this bit is forced onto the
        /// NSS pin and the I/O value of the NSS pin is ignored
        SPI_SSI OFFSET(8) NUMBITS(1) [

        ],

        /// Frame Format
        /// 0: data is transmitted/received with the MSB first
        /// 1: data is transmitted/received with the LSB first
        /// Note: This bit should not be changed when communication is ongoing
        SPI_LSBFIRST OFFSET(7) NUMBITS(1) [
            LSB_FIRST = 1,
            MSB_FIRST = 0,
        ],

        /// SPI Enable
        /// 0: SPI is disabled
        /// 1: SPI is enabled
        SPI_SPE OFFSET(6) NUMBITS(1) [
            ENABLED  = 1,
            DISABLED = 0,
        ],

        /// Baud Rate Control
        /// 000: fCLK/2
        /// 001: fCLK/4
        /// 010: fCLK/8
        /// 011: fCLK/16
        /// 100: fCLK/32
        /// 101: fCLK/64
        /// 110: fCLK/128
        /// 111: fCLK/256
        /// Note:This bit should not be changed when communication is ongoing
        SPI_BR OFFSET(3) NUMBITS(3) [

        ],

        /// Master Selection
        /// 0: Slave Configuration
        /// 1: Master Configuration
        /// Note This bit should not be changed when communication is ongoing
        SPI_MSTR OFFSET(2) NUMBITS(1) [
            SLAVE_CONFIG   = 1,
            MASTER_CONFIG  = 0,
        ],

        ///Clock Polarity
        ///0: CLK is 0 when idle
        ///1: CLK is 1 when idle
        SPI_CPOL OFFSET(1) NUMBITS(1) [
            ONE_IDLE   = 1,
            ZERO_IDLE  = 0,
        ],

        ///Clock Phase
        ///0: The first clock transition is the first data capture edge
        ///1: The second clock transition is the first data capture edge
        SPI_CPHA OFFSET(0) NUMBITS(1) [
            SECOND_CLK = 1,
            FIRST_CLK  = 0,
        ]
    ],

    SPI_CR2 [
        ///SPI_TOTAL_BITS_RX OFFSET(24) NUMBITS(7) [],
        SPI_RX_IMM_START OFFSET(16) NUMBITS(1) [],
        SPI_RX_START OFFSET(15) NUMBITS(1) [],
        SPI_LDMA_TX_START OFFSET(14) NUMBITS(1) [],
        SPI_LDMA_RX OFFSET(13) NUMBITS(1) [],

        /// FIFO reception threshold is used to set the threshold of
        /// the RXFIFO that triggers an RXNE event.
        /// 0: RXNE event is generated if the FIFO level is greater
        /// than or equal to 1/2 (16-bit)
        /// 1: RXNE event is generated if the FIFO level is greater
        /// than or equal to 1/4 (8-bit)
        SPI_FRXTH OFFSET(12) NUMBITS(1) [],

        /// Reserved bits
        /// SPI_DS OFFSET(8) NUMBITS(4) [],

        /// Interrupt enable for TXE event.
        /// 0: TXE interrupt masked
        /// 1: TXE interrupt is not interrupt masked
        SPI_TXEIE OFFSET(7) NUMBITS(1) [],

        /// Interrupt enable for RXNE event
        /// 0: RXNE interrupt masked
        /// 1: RXNE interrupt is not interrupt masked
        SPI_RXNEIE OFFSET(6) NUMBITS(1) [
            RXNE_UNMASKED = 1,
            RXNE_MASKED = 0,
        ],

        /// when an error condition occurs.
        /// 0: Error interrupt masked
        /// 1: Error interrupt not masked.
        //Frame Error (Sets when the stopis zero)
        SPI_ERRIE OFFSET(5) NUMBITS(1) [
            MASKED_INT   = 1,
            UNMASKED_INT = 0,
        ],

        /// Reserved bits
        /// SPI_FRF OFFSET(4) NUMBITS(1) [],
        /// SPI_NSSP OFFSET(3) NUMBITS(1) [],

        /// SS output enable
        /// 0: SS output is disabled in master mode and the SPI interface
        /// can work in a multi-master configuration
        /// 1: SS output is enabled in master mode and when the SPI
        /// interface is enabled. The SPI interface cannot work in a
        /// multi-master environment.
        SPI_SSOE OFFSET(2) NUMBITS(1) [
            SSOE_ENABLED  = 1,
            SSOE_DISABLED = 0,
        ],

        // Reserved bits
        // SPI_TXDMAEN OFFSET(1) NUMBITS(1) [],
        // SPI_RXDMAEN OFFSET(0) NUMBITS(1) [],

    ],

    SPI_SR[

        SPI_FTLVL OFFSET(11) NUMBITS(2) [],
        SPI_FRLVL OFFSET(9) NUMBITS(2) [],

        SPI_FRE OFFSET(8) NUMBITS(1) [],
        SPI_BSY OFFSET(7) NUMBITS(1) [],
        SPI_OVR OFFSET(6) NUMBITS(1) [],

        SPI_MODF OFFSET(5) NUMBITS(1) [],

        SPI_CRCERR OFFSET(4) NUMBITS(1) [],

        SPI_TXE OFFSET(1) NUMBITS(1) [],

        SPI_RXNE OFFSET(0) NUMBITS(1) []

    ],

    SPI_DR1 [
        DR1 OFFSET(0) NUMBITS(32) []
    ],

    SPI_DR2 [
        DR2 OFFSET(0) NUMBITS(32) []
    ],

    SPI_DR3 [
        DR3 OFFSET(0) NUMBITS(32) []
    ],

    SPI_DR4 [
        DR4 OFFSET(0) NUMBITS(32) []
    ],

    SPI_DR5 [
        DR5 OFFSET(0) NUMBITS(32) []
    ],


}

register_structs! {
    #[allow(non_snake_case)]
    pub RegisterBlock{
        (0x00 => SPI_CR1: ReadWrite<u32, SPI_CR1::Register>),
        (0x04 => SPI_CR2: ReadWrite<u32, SPI_CR2::Register>),
        (0x08 => SPI_SR:  ReadOnly <u32, SPI_SR ::Register>),
        (0x0C => SPI_DR1: ReadWrite <u32, SPI_DR1::Register>),
        (0x10 => SPI_DR2: ReadWrite <u32, SPI_DR2::Register>),
        (0x14 => SPI_DR3: ReadWrite <u32, SPI_DR3::Register>),
        (0x18 => SPI_DR4: ReadWrite <u32, SPI_DR4::Register>),
        (0x1C => SPI_DR5: ReadWrite <u32, SPI_DR5::Register>),
        (0x20 => _reserved),
        (0x2C => @END),
    }
}

/// Abstraction for the associated MMIO registers.
type Registers = MMIODerefWrapper<RegisterBlock>;

pub struct SPIInner {
    registers: Registers,
}

```

The code includes necessary dependencies from tock_registers and defines a constant SPI_OFFSET representing the memory-mapped offset for the SPI module.

**Bitfields Definition**
The code defines bitfields for various SPI registers **(SPI_CR1, SPI_CR2, SPI_SR, SPI_DR1, SPI_DR2, SPI_DR3, SPI_DR4, SPI_DR5)**. Each register has different bitfields representing configuration options and status flags.

**Register Structures**
The code uses the register_structs! macro to define the memory-mapped register block for the SPI module. It includes registers like **SPI_CR1, SPI_CR2, SPI_SR, SPI_DR1, SPI_DR2, SPI_DR3, SPI_DR4, and SPI_DR5**. Each register is associated with its offset in memory, and their types are specified based on read or write access and the bitfields defined earlier.

**SPIInner Struct**
The SPIInner struct is defined to encapsulate the SPI module's registers. It contains a field registers of type **MMIODerefWrapper<RegisterBlock>**, providing a convenient way to access the SPI module's registers.

Overall, the code sets up a structured representation of the SPI module's registers with bitfields and provides an abstraction (SPIInner) for interacting with these registers. It's a common pattern in embedded systems programming to use such abstractions to manage hardware peripherals.