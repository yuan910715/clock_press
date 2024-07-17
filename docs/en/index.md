# Introduction

## ClockWise

![ClockWise Logo](/img/clockwise_logo.png)
ClockWise is an open source project developed by **Jonathas Barbosa**, the project code is hosted on GitHub, the repository address is: https://github.com/jnthas/clockwise , I am also on the project contributor list and have submitted some optimization code

The project use a 64x64 pixel LED matrix for display, driven by ESP32, implemented the clock display engine and multiple clockfaces

Projects are more suitable for software/hardware professionals, you need to prepare hardware, wire, select clockface on web page, flash firmware, and configure network via serial communication, after configure the time zone, NTP server and other parameters on the configuration page, and the clock can be used normally

## ClockWise Plus

After using ClockWise for a few months, I made some optimizations and push them to the original author's code repository

Another idea is to modify this project to be more suitable for ordinary users  
`It is consistent with the idea of ​​my own open source project SuperY Wifi Clock, https://github.com/yuan910715/Esp8266_Wifi_Matrix_Clock`

I designed the PCB board, hardware welding and firmware flashing are completed by professionals, ordinary users only need to configure on the configuration page, changing the clockface does not require reflashing the firmware. Since this idea is quite different from the original author's project, I made this project independent, ClockWise Plus, following the original project's MIT open source license

### Upgrades

I have done a lot of optimizations of the code, mainly as follows：

- **Clockface integration**  
Each clockface in the original project is actually a sub repository, which means that each firmware actually only compiles one clockface. If you need to change the clockface, you need to re-flash the firmware. I have completed the real-time clockface change logic, compiled all the existing clockface, and you can change it at any time on the configuration page
- **Wifi Config**  
Remove the serial port network configuration, and configure wifi by connecting your mobile phone to the clock hotspot
- **Localization**  
The configuration page has Chinese and English version now, and the default time zone is changed to Asia/Shanghai
- **Configuration page optimization**  
All configurations take effect immediately without restarting the clock, added change log, erase wifi button, running time, etc. put js file into a remote server to prevent ESP32 crashed
- **Clockface optimization**  
Modify the clockface and rewrite the time logic, so that the clock reaches millisecond accuracy. Super Mario clockface adds seconds at the first line
- **OTA update**  
The clock has the ability to continuously update, when new firmware is released, the clock can be automatically updated without re-flashing the firmware, I will add configuration items, optimize functions, and add clockfaces in the future...
![ota](/img/ota.png)

### Photos
![Real1](/img/real1.png)

![Real2](/img/real2.png)

![Real3](/img/real3.png)

![all](/img/all.png)

## Members

<script setup>
import { VPTeamMembers } from 'vitepress/theme'

const members = [
  {
    avatar: 'https://topyuan.top/yuan.png',
    name: 'Felix Feng',
    title: 'Creator',
    links: [
      { icon: 'github', link: 'https://github.com/yuan910715' }
    ]
  },
]
</script>

<VPTeamMembers size="small" :members="members" />

Hope you can join