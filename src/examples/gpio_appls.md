# GPIO blinky

The Rust code controls LEDs using the Shakti platform's GPIO module. It defines a struct GPIO_ACCESS for GPIO operations, sets up constants for LEDs and GPIO registers, and implements methods to control LEDs. The main function initializes GPIO, enters a loop to toggle LEDs, and includes a simple delay function. The code provides a basic framework for LED control on the Shakti platform. Adjustments may be needed for specific GPIO configurations and timing accuracy.

```
...
...
fn main() -> !{

    let gpio_mmio_start_addr = 0x1000;
    
        
    let mut gpio_access = GPIO_ACCESS::new(); 
    gpio_access.set_direction(LED0_B|LED0_G|LED0_R|LED1_B|LED1_G|LED1_R|LED2|LED3);
    gpio_access.turn_off_ledx();  

        // Set the direction control register
    //gpio.set_direction_control(0x0);
    
        // Write to GPIO_DATA_REG to initialize GPIO pins
    //gpio.set_data_register(0x0); // Assuming initialization value is 0x0
    

    loop {
            delay_loop(DELAY1, DELAY2);

            gpio_access.turn_on_ledx(LED0_G);
            delay_loop(DELAY1, DELAY2);
         }
}
...
...
```