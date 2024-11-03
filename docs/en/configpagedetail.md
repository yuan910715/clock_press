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
Set your timezone  
- Single timezone  
Suitable for countries/regions with a single time zone, simply drag the slider or manually enter  
`eg:+8:00`
- Smart DST timezone  
For countries/regions that are suitable for using daylight saving time, a POSIX time zone string needs to be defined. Posix defines the time zones for standard time, daylight saving time, and when daylight saving time starts and ends  
If you need to learn more about POSIX, please visit https://github.com/yuan910715/Esp8266_Wifi_Matrix_Clock/blob/master/posix.md  
`eg:CST6CDT,M3.2.0,M11.1.0`  

## NTP Server
Set the NTP server address(default:ntp2.aliyun.com), the clock will synchronize with this NTP server. If the configuration is wrong, the clock will not be able to synchronize, the time will start from 1970-1-1 8:00  
**DO NOT USE pool.ntp.org** it is an excellent large pool of NTP servers, but occasionally provides an unavailable node, lasts for dozens of minutes, since the LED will freeze during NTP synchronization and will retry every 5 seconds after NTP synchronization fails, the clock will freeze every 5 seconds during this period. The clock will freeze noticeably on some clockface (such as Pac Man)  
`There are many open NTP servers on the Internet, list of high-quality NTP servers:`
```
Google:
time.google.com
time1.google.com
time2.google.com
time3.google.com
time4.google.com

Cloudflare:
time.cloudflare.com

Facebook:
time.facebook.com
time1.facebook.com
time2.facebook.com
time3.facebook.com
time4.facebook.com
time5.facebook.com

Microsoft:
time.windows.com

Apple:
time.apple.com
time1.apple.com
time2.apple.com
time3.apple.com
time4.apple.com
time5.apple.com
time6.apple.com
time7.apple.com
```

## Use 24h format?
Whether to use the 24-hour format. If the turned on, the 24-hour format will be used

## Brightness adjustment method
- Adjust according to ambient light  
Automatically adjust brightness based on ambient light values collected by photoresistors
- Adjust according to time  
Adjust the brightness by time according to the set night time period
- Fixed brightness value  
Fixed value, no adjustment

## Max brightness/Daytime brightness/Brightness value
Set the display brightness of the LED module
- 0 - Dark (display off)
- 255 - Super bright  
`When Adjust according to ambient light, this value indicates the brightness value at maximum illumination`  
`When Adjust according to time, this value represents the brightness value during the daytime period`  
`When Fixed brightness value, this value indicates the actual brightness value of the LED module`  

## Automatic Bright
The clock is connected to a LDR, and the collected value will change according to the light intensity. Automatic brightness adjustment can be configured, and the brightness of the LED module will automatically adjust according to the ambient light intensity  
The two values ​​that need to be configured are 
- **the collected value of the photoresistor when the light is the darkest**
- **the collected value of the photoresistor when the light is the highest**

`Example：When the configuration is minimum value 0 and maximum value 2000, according to the value collected by the LDR, when it is >=2000, it is adjusted to the maximum brightness, and when it is 0, it is the minimum brightness (the display will not be turned off)`  
`Example：When the configuration is set to a minimum value of 100 and a maximum value of 2000, the light is adjusted to the maximum brightness when LDR value >=2000, the minimum brightness when LDR value is 100, and the display is turned off when LDR value <100. This is suitable for use in bedrooms where the clock needs to be turned off in dim light`  
`When GL5539 photoresistor + 10K voltage divider resistor is used, the best maximum value is set to 1800-2700. Please adjust according to actual needs`

## LDR Pin
Here you can get the current LDR collection value in real time  
`You can test it by covering the photoresistor with your hand and collecting the value`

## Night settings
- Night brightness level  
When Adjust according to time, the brightness level during the nighttime period. Set to 0 means turn off the display
- Night mode starts  
Start time of night mode
- Night mode ends  
End time of night mode

## Rotation
LED rotates to suit different placements

## Clock Face
Change clockface

## Auto Change Clock Face
Automatically change the clockface at 0:00 every day, sequence or random

## Language
Change the configuration page language

## Bottom Information

### Version

Displays the current clock firmware version

### Running Time

Displays the time the clock has been running, increasing every 24 consecutive hours

### Connected WiFi

Display the WiFi SSID of the current clock connection