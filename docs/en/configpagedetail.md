# Configuration Details

The configuration page can adapt to PC browsers and mobile browsers

## Top Information

### Change log
Clock version update history, you can view the update date and content of each version, if you have features you would like to add, please contact me: <a href="mailto:admin@topyuan.top">admin@topyuan.top</a>  
`The clock has a built-in automatic update function. You may open the configuration page every once in a while and find that the clock version has been updated and new functions have been added`

### Erase WiFi
Erase the WiFi information recorded by the clock

### Restart
Restart the clock immediately

## Timezone
Configure your time zone. Examples:Asia/Shanghai, America/Sao_Paulo, Europe/Lisbon  
The clock will automatically connect to the remote server to convert Asia/Shanghai, America/Sao_Paulo, Europe/Lisbon to the corresponding POSIX time zone expressions  
`For example, Asia/Shanghai will be converted to CST-8, which means it is 8 hours ahead of UTC time zone`  
`POSIX time zone expressions with daylight saving time are more complicated; see the POSIX Timezone String section for more details`

## NTP Server
Set the NTP server address, the clock will synchronize with this NTP server. If the configuration is wrong, the clock will not be able to synchronize, the time will start from 1970-1-1 8:00  
`There are many open NTP servers on the Internet, such as pool.ntp.org ntp2.aliyun.com, etc.`

## Use 24h format?
Whether to use the 24-hour format. If the turned on, the 24-hour format will be used

## Display Bright
Set the display brightness of the LED module
- 0 - Dark (display off)
- 255 - Super bright  
`When automatic brightness adjustment is turned off, this value indicates the actual brightness value of the LED module`  
`When automatic brightness adjustment is turned on, this value indicates the brightness value at maximum illumination`

## Automatic Bright
The clock is connected to a LDR, and the collected value will change according to the light intensity. Automatic brightness adjustment can be configured, and the brightness of the LED module will automatically adjust according to the ambient light intensity  
The two values ​​that need to be configured are 
- **the collected value of the photoresistor when the light is the darkest**
- **the collected value of the photoresistor when the light is the highest**

Turn on/off:
- When the maximum value is 0, the automatic brightness adjustment is turned off and the clock will not automatically adjust the brightness
- When the maximum value is greater than 0, automatic brightness adjustment is turned on  
`Example：When the configuration is minimum value 0 and maximum value 2000, according to the value collected by the LDR, when it is >=2000, it is adjusted to the maximum brightness, and when it is 0, it is the minimum brightness (the display will not be turned off)`  
`Example：When the configuration is set to a minimum value of 100 and a maximum value of 2000, the light is adjusted to the maximum brightness when LDR value >=2000, the minimum brightness when LDR value is 100, and the display is turned off when LDR value <100. This is suitable for use in bedrooms where the clock needs to be turned off in dim light`  
`When GL5539 photoresistor + 10K voltage divider resistor is used, the best maximum value is set to 1800-2700. Please adjust according to actual needs`

## LDR Pin
Here you can get the current LDR collection value in real time  
`You can test it by covering the photoresistor with your hand and collecting the value`

## Rotation
LED rotates to suit different placements

## Clock Face
Change clockface

## POSIX Timezone String
Manually configure the POSIX time zone string. This is an advanced configuration, used when the time zone is invalid or you want to customize the daylight saving time configuration. This configuration will override the time zone configuration  
POSIX strings define standard time zone, daylight saving time zone, daylight saving time start time, and daylight saving time end time. If you need to learn more about POSIX, please visit https://github.com/yuan910715/Esp8266_Wifi_Matrix_Clock/blob/master/posix.md  
`Leave this blank to automatically get the POSIX time zone string from the server based on the time zone`  

## Language
Change the configuration page language

## Bottom Information

### Version

Displays the current clock firmware version

### Running Time

Displays the time the clock has been running, increasing every 24 consecutive hours

### Connected WiFi

Display the WiFi SSID of the current clock connection