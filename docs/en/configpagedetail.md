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
The value is **the collected value of the photoresistor when the light is the highest**  
- Default : 2000  
The clock will be divided into 10 brightness levels. Example: default 2000, the collected value of per 200 is a brightness level, the value affects the sensitivity of automatic brightness adjustment  
`If this value is too small, the clock will still maintain a high brightness level even when the ambient light is low`  
`If this value is too high, the clock will still maintain a low brightness level even in high ambient light`  

Read LDR value  
- Here you can get the current LDR collection value in real time  
`You can test it by covering the photoresistor with your hand and collecting the value`

Night strategy  
- Nothing  
Do nothing
- Turn off led  
Turn off led, until the light intensity increases
- Display big clock  
Display big clock, until the light intensity increases 

Night LDR value  
- When the collected value of the LDR is less than this value, the LED will execute a night strategy, Default : 30  
`If the ambient light is too high at night to enter night mode, you can increase this value appropriately`  
`If clock enter night mode at daytime, you can reduce this value appropriately`  
`If the night mode still does not exit after the light is restored, this value can be appropriately reduced`

Big clock color  
- Configure the font color of the big clock, choose from pre made common colors or fill in RGB565 color values by yourself

## Night settings
- Night brightness level  
When Adjust according to time, the brightness level during the nighttime period. Set to 0 means turn off the display
- Night mode starts  
Start time of night mode
- Night mode ends  
End time of night mode

Night strategy  
- Nothing  
Do nothing
- Turn off led  
Turn off led, until end time of night mode
- Display big clock  
Display big clock, until end time of night mode

Big clock color  
- Configure the font color of the big clock, choose from pre made common colors or fill in RGB565 color values by yourself

## Rotation
LED rotates to suit different placements

## Clock Face
Change clockface

## Auto Change Clock Face
Automatically change the clockface at 0:00 every day, sequence or random

## Special LED
For differcent Leds  
If your LED color is wrong, please adjust the color option  
If your LED has ghosting or misalignment, please try adjusting the reverse phase

## Language
Change the configuration page language

## Bottom Information

### Version

Displays the current clock firmware version

### Running Time

Displays the time the clock has been running, increasing every 24 consecutive hours

### Connected WiFi

Display the WiFi SSID of the current clock connection