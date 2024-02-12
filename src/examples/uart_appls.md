# Uart Read and Write 

This Rust program is designed for a Shakti processor and uses the "shakti_riscv_hal" crate. It initializes a UART peripheral, sends two strings over UART, and enters an infinite loop to continuously read characters from the UART. The program is written without relying on the standard library and features inline assembly for delay operations.


```
#![no_std]
#![no_main]
#![feature(asm)]

use riscv::{asm::delay, delay};
// use cortex_m_rt::entry;
use riscv_rt::entry;
use shakti_riscv_hal::gpio::{GPIOInner, GPIO_OFFSET};
use shakti_riscv_hal::uart::{UartInner, UART_OFFSET};



#[entry]
fn main() -> ! {
    let mut uart = unsafe { UartInner::new(UART_OFFSET) };

    uart.write_uart_string("Hello from shakti \n ");

    uart.write_uart_string("END of the main function");

    loop {
        uart.read_uart_char();
    }
}

```