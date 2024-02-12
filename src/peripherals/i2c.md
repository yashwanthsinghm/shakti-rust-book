# I2C

**Shakti processors**, which are based on the **RISC-V architecture**, the I2C (Inter-Integrated Circuit) interface is commonly used to connect low-speed devices within embedded systems. The I2C module in Shakti processors typically includes several registers that control and manage the communication on the I2C bus. Here's a general overview of the I2C module for Shakti:

## I2C Registers:

- **Prescale Register** (Prescale - Offset 0x00):
    Size: 8 bits
    Accessible: Read/Write
    Description: Configures the I2C prescaler, which divides the system clock to determine the I2C clock frequency.
    
- **Control Register** (Control - Offset 0x08):
    Size: 8 bits
    Accessible: Read/Write
    Description: Configures and controls the I2C data transfer. Key bits include:
    **I2C_PIN**: Pending Interrupt Not, Used as a software reset.
    **I2C_ENI**: Enables the external interrupt output.
    **I2C_STA**: Transmits Start condition + Slave address.
    **I2C_STO**: Transmits the stop condition.
    **I2C_ACK**: Acknowledgement bit.

- **Data Shift Register** (Data - Offset 0x10):
    Size: 8 bits
    Accessible: Read/Write
    Description: Holds the data to be transmitted or received during I2C communication.

- **Status Register** (Status - Offset 0x18):
    Size: 8 bits
    Accessible: Read
    Description: Provides the status of I2C data transfer. Key bits include:
    **I2C_INI**: High when I2C communication is in progress.
    **I2C_STS**: Status flag in slave receiver mode for externally generated STOP condition.
    **I2C_BER**: Bus error flag for misplaced START or STOP condition.
    **I2C_AD0/I2C_LRB**: Last Received Bit and Address 0 in slave mode.
    **I2C_AAS**: Addressed As Slave bit.
    **I2C_LAB**: Lost Arbitration Bit.
    **I2C_BB**: Bus Busy bit.

- **Clock Register** (SCL - Offset 0x38):
    Size: 8 bits
    Accessible: Read/Write
    Description: Divides the I2C Prescaler clock to determine the I2C SCL (Serial Clock Line) frequency.
    
- **General Overview**:
    - I2C is used for connecting low-speed devices in embedded systems.
    - It facilitates communication between a single master and multiple slave devices.
    - Two-wire interface: SDA (Data) and SCL (Clock).
    - Each device has a unique address.
    - Synchronous communication with master controlling the clock.
    - Registers control prescaling, data transfer, status, and clock frequency.

### Rust I2C Register implmentation ###

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

pub const I2C_OFFSET: usize = 0x0004_0000;

pub const I2C_INI: u8 = 1 << 7;
pub const I2C_STS: u8 = 1 << 5;
pub const I2C_BER: u8 = 1 << 4;
pub const I2C_AD0_LRB: u8 = 1 << 3;
pub const I2C_AAS: u8 = 1 << 2;
pub const I2C_LAB: u8 = 1 << 1;
pub const I2C_BB: u8 = 1 << 0;

pub const I2C_PIN: u8 = 1 << 7;
pub const I2C_ES0: u8 = 1 << 6;
pub const I2C_ENI_LRB: u8 = 1 << 3;
pub const I2C_STA: u8 = 1 << 2;
pub const I2C_STO: u8 = 1 << 1;
pub const I2C_ACK: u8 = 1 << 0;

register_structs! {
    #[allow(non_snake_case)]
    pub RegistersBlock{
        (0x00 => PRESCALE: ReadWrite<u16>),
        (0x02 => _reserved0),
        (0x08 => CONTROL: ReadWrite<u8>),
        (0x09 => _reserved1),
        (0x10 => DATA: ReadWrite<u8>),
        (0x11 => _reserved2),
        (0x18 => STATUS : ReadWrite<u8>),
        (0x19 => _reserved3),
        (0x38 => SCL : ReadWrite<u8>),
        (0x39 => _reserved4)
,       (0x3C => @END),
    }
}

register_bitfields! {
    u32,
///I2C Prescale Register divides the System clock by (Prescale value + 1). This clock is used as
///clock input for I2C Serial Clock.
///I2C Prescaler clock = System Clock / (Prescaler Value + 1)
    PRESCALE [
        PRESCALE_VALUE OFFSET(0) NUMBITS(8) []
    ],
    ///I2C SCL Register divides the I2C Prescaler clock by (SCL value + 1). This clock is used as
///I2C SCL clock = I2C Prescaler Clock / (SCL COUNT + 1).
    SCL[
        SCL_COUNT OFFSET(0) NUMBITS(8) []
    ],
    STATUS [
///High when I2C communication in progress. Becomes low once I2C
///communication is complete.
        I2C_INI OFFSET(7) NUMBITS(1) [],
   ///     When in slave receiver mode, this flag is asserted when an
      ///  externally generated STOP condition is detected (used only in
       /// slave receiver mode).
        I2C_STS OFFSET(5) NUMBITS(1) [],
///Bus error; a misplaced START or STOP condition has been
///detected
        I2C_BER OFFSET(4) NUMBITS(1) [],
     /*    AD0(Address 0) - General Call bit used for Broadcast
        LRB - Last Received Bit through I2C Bus
        This status bit serves a dual function, and is valid only while
        PIN = 0:
        1. LRB holds the value of the last received bit over the
        I2C-bus while AAS = 0 (not addressed as slave). Normally
        this will be the value of the slave acknowledgement; thus
        checking for slave acknowledgement is done via testing of the
        LRB.
        2. AD0; when AAS = 1 (‘Addressed As Slave’ condition), the
        I2C-bus controller has been addressed as a slave. Under this
        condition, this bit becomes the ‘AD0’ bit and will be set to
        logic 1 if the slave address received was the ‘general call’
        (00H) address, or logic 0 if it was the I2C-bus controller’s own
        slave address.
        */
        I2C_AD0_LRB OFFSET(3) NUMBITS(1) [],
///Addressed As Slave’ bit. Valid only when PIN = 0. When
///acting as slave receiver, this flag is set when an incoming
///address over the I2C-bus matches the value in own address
//register
        I2C_AAS OFFSET(2) NUMBITS(1) [],
///Lost Arbitration Bit. This bit is set when, in multi-master
///operation, arbitration is lost to another master on the I2C-bus
        I2C_LAB OFFSET(1) NUMBITS(1) [],
///Bus Busy bit. This is a read-only flag indicating when the
///I2C-bus is in use. A zero indicates that the bus is busy, and
///access is not possible
        I2C_BB OFFSET(0) NUMBITS(1) []

    ],

    CONTROL [
///Pending Interrupt Not, Used as a software reset. If set to 1, all
    I2C_PIN OFFSET(7) NUMBITS(1) [],
   /// Enable Serial Output.
    ///0 - Registers can be initialized.
    ///1 - I2C Serial Transmission
    I2C_ES0 OFFSET(6) NUMBITS(1) [
        REGISTER_INITIALIZED = 0,
        I2C_SERIAL_TRANSMISSION = 1,
    ],
///Enables the external interrupt output, which is generated when
///the PIN is active (Low)
    I2C_ENI OFFSET(3) NUMBITS(1) [],
///Transmits Start condition + Slave address..
    I2C_STA OFFSET(2) NUMBITS(1) [],
///Transmits the stop condition.
    I2C_STO OFFSET(1) NUMBITS(1) [],
///Acknowledgement bit.
///1: I2C automatically sends an acknowledgement after a
///read/write transaction.
///0: I2C Master sends Negative Acknowledge to stop the I2C
///transfer
    I2C_ACK OFFSET(0) NUMBITS(1) [
        NAK = 0,
        ACK = 1,
    ]

],


}

type Registers = MMIODerefWrapper<RegistersBlock>;

pub struct I2CInner {
    registers: Registers,
}

```


Rust implementation for interacting with the I2C (Inter-Integrated Circuit) module on Shakti processors.

- **Constants**:
    - **I2C_OFFSET**: Specifies the offset address for the I2C module in memory.
    - Various constants **(I2C_INI, I2C_STS, etc.)** represent bit positions in status and control registers.

- **Register Structures**
    - **RegistersBlock** : Defines the memory-mapped I2C registers.
    - **PRESCALE**       : Configures the I2C prescaler for clock frequency.
    - **CONTROL**        : Configures and controls I2C data transfer.
    - **DATA**           : Holds data to be transmitted or received.
    - **STATUS**         : Provides status information about I2C communication.
    - **SCL**            : Configures the I2C SCL (Serial Clock Line) clock.
-**Register Bitfields**:
    - **register_bitfields!** macro defines specific bits within registers.
    - **PRESCALE**: PRESCALE_VALUE represents the prescaler value.
    - **SCL**     : SCL_COUNT represents the count for dividing the I2C Prescaler clock.
    - **STATUS**  : Defines various status bits such as **I2C_INI, I2C_STS, I2C_BER, **etc.
    - **CONTROL** : Defines control bits such as **I2C_PIN, I2C_ES0, I2C_ENI, **etc.
- **MMIODerefWrapper and Structs**:
    - **MMIODerefWrapper** : Represents a memory-mapped I/O (MMIO) wrapper for safe interaction with hardware registers.
    - **Registers**        : A wrapper for the RegistersBlock, ensuring safe access to I2C registers.
- **I2CInner Struct**:
    - **I2CInner**: A struct encapsulating the I2C module, containing an instance of the Registers wrapper.