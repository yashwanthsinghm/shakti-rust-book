### GDB (GNU Debugger) ###

GDB is a powerful debugger commonly used in embedded development. It allows developers to inspect and debug programs running on embedded systems. GDB supports various architectures, including ARM, making it suitable for debugging ARM Cortex-M microcontrollers.

Installation Steps for GDB on Linux:

open terminal 

- ``` sudo apt-get install gdb         ```
- ``` gdb --version  ``` 

### OpenOCD (Open On-Chip Debugger) ###

OpenOCD acts as a bridge between GDB and the debugging hardware, such as ST-Link, on your embedded system. It facilitates communication by translating GDB's TCP/IP-based remote debug protocol to the specific protocol used by the debugging hardware.

Installation Steps for GDB and OpenOCD on Linux:

- ``` sudo apt-get install openocd         ```
- ``` openocd --version  ``` 

### Rustup and Nightly ###

**Install Rustup**:

- ``` curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh ```
- ``` source $HOME/.cargo/env ```

**Install Nightly**:

After installing Rustup, you can install the nightly toolchain:

``` rustup toolchain install nightly ```
``` rustup default nightly ```

**Add riscv target supported for Shakti** 

- ```rustup target add thumbv7em-none-eabihf```
- ``` rustup show ```
 
 **Note : rustup show will show all the installed tool chains and targets, which helps to verify.**
