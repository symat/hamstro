
# TODO:
- pi zero failed to connect to WIFI after Bluetooth is disabled and the pi is plugged-out and plugged-in again to power.Looks like Bluetooth is necessary for WIFI to work… (?) TODO: verify



# Setting up the development environment on Rapsberry Pi Zero W

1. Download the Pi Imager: https://www.raspberrypi.com/software/
2. Use Rapsberry Pi OS Lite (32bit), debian trixie, configure:

   * hostname: hamstro
   * localization: TODO: esperanto keyboard?
   * user: hamstro, password: REDACTED
   * make sure you have WIFI selected and SSH authentication enabled
   * disable Rapsberry Pi Connect

3. SSH to the PI and finish the setup:

```
sudo apt update
sudo apt install -y git python3-pip vim gcc build-essential gcc-avr binutils-avr avr-libc gdb-avr mc


sudo pip install --break-system-packages --resume-retries  --root-user-action pymcuprog==3.19.4.61

echo "dtoverlay=disable-bt" | sudo tee -a /boot/firmware/config.txt
sudo systemctl disable hciuart
sudo raspi-config nonint do_serial_cons 0
sudo raspi-config nonint do_serial_hw 1


sudo reboot

#maybe reboot is not even enough, but need to unplug from power (?) - TODO
```



## UART settings

see: https://www.raspberrypi.com/documentation/computers/configuration.html#configure-uarts

we need to use the fast UART for UPDI programming and the miniUART as secondary / Bluetooth interface. As for the moment we don't use Bluetooth, we simply disable the Bluetooth interface and setting the first (PL011,UART0) as the primary UART.

To list the available uart interfaces, use: `python -m serial.tools.list\_ports`



## using pymcuprog, hello World + FUSE setup

First test if we see the microcontroller.

```
pymcuprog ping -d attiny3224 -t uart -u /dev/ttyAMA0 
```

Then write a hello-world blink program, blink.c (assuming the LED is connected to pin 7:

```
// running on 10MHz: using 20MHz internal oscillator with prescale 2
#define F_CPU 10000000UL

#include <avr/io.h>
#include <avr/builtins.h> // Required for protected writes
#include <util/delay.h>

int main(void) {
    // Set the data direction register for Port B pin 2 (PB2) as output
    //PORTB.DIR |= (1 << 2); 
    PORTB.DIRSET = PIN2_bm;

    // define the prescaler to 2 by setting the control register to 0xE1
    // The register is write-protected, so a protected write is necessary.
    // (The CPU_CCP register (Change Protection) must be set to 0xD8 first.)
    _PROTECTED_WRITE(CLKCTRL.MCLKCTRLB, 0xE1);
    

    while (1) {
        // Toggle the LED on (set pin high)
        PORTB.OUT |= (1 << 2);
        //PORTB.OUTSET = PIN2_bm;
        _delay_ms(1000); // Wait for 1000 milliseconds

        // Toggle the LED off (set pin low)
        //PORTB.OUT \&= ~(1 << 2);
        PORTB.OUTCLR = PIN2_bm;
        _delay_ms(1000); // Wait for another 1000 milliseconds
    }
}
 

```

# build the code and upload it to the AVR mcu

```
avr-gcc -mmcu=attiny3224 -O1 -c blink.c -o blink.o \&\&
avr-gcc -mmcu=attiny3224 blink.o -o blink.elf \&\&
avr-objcopy -O ihex blink.elf blink.hex \&\&
pymcuprog write -d attiny3224 -t uart -u /dev/ttyAMA0 -f blink.hex

#optionally: also erase memory, then write the program and even verify it
pymcuprog write -d attiny3224 -t uart -u /dev/ttyAMA0 -f blink.hex --erase --verify


```

# read fuse values

the fuse values are mapped to address 0x1280-0x1288. The reserved bits of the fuses must be set to 1.

```
pymcuprog read -d attiny3224 -t uart -u /dev/ttyAMA0 -m fuses -o 0x0 -b 9

```

the default configuration (`0x001280: 00 00 7E FF FF F6 FF 00 00`) means:

* no watchdog, no BOD
* 20MHz internal oscillator
* CRC disabled on the memory
* RESET disabled (the UPDI pin is used only for UPDI)
* 64 ms start-up time between power-on and code execution
* the entire Flash is in BOOTCODE section (no APPCODE section defined)



