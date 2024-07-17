# Hardware

## LED

Use P2.5 full-color LED display module, often used in weddings, concerts and other large screens, compatible with P3 and other LED sizes  
Size: 160mm x 160mm  
Point spacing: 2.5mm  
LED Package: SMD 2121  
Number of LEDs: 64 x 64  
Weight: 0.2kg  
Interface： HUB75E  
Optimum viewing distance: 1-15M  
Operating temperature: -25-50℃

## ESP32

ESP32 is a high-performance, low-power Wi-Fi and Bluetooth dual-mode system-on-chip (SoC) launched by Espressif Systems, it is widely used in the fields of Internet of Things, smart home, wearable devices, etc. It is based on the extremely low-power Tensilica Xtensa LX6 microprocessor and integrates a wealth of peripherals and sensor interfaces
![ESP32](/img/esp32.png)

## PCB

The HUB75 terminal has many connections, I designed an integrated PCB adapter board according to the size of the LED module. While connecting the ESP32 and the HUB75 terminal, I also added a LDR position and added an independent power supply
![PCB1](/img/pcb1.png)

HUB75 terminal - ESP32 connection diagram
![HUB75](/img/hub75.png)

## LDR

Connect a GL5539 photoresistor and a 10K voltage divider resistor so that the clock can automatically adjust the display brightness according to the ambient light brightness

## Power Supply

Independent power supply to prevent high current from burning out the diode on the ESP32 development board