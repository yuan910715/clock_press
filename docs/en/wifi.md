# Setup WiFi

## When?

- First time use, a new, never used clock
- The clock has changed location, such as from home A to office B, so a new WiFi needs to be configured
- The connected WiFi has changed the SSID or password
- The configured WiFi password is incorrect and the clock cannot complete the connection

## How?

After the clock is powered on, if the network has not been configured or the configured network cannot be connected, the following prompt will appear:

![wifi1](/img/wifi1.png)

At this time, the clock will emit a WiFi hotspot named **-Clock-**, and you need to connect to this hotspot with your mobile phone or PC  

![wifi2](/img/wifi2.png)

After successful connection, the network configuration page will pop up automatically  
`In some special old mobile phones, the page may not pop up automatically. You can manually open the browser and go to http://192.168.4.1`

![wifi3](/img/wifi3.png)

Click **Scan WiFi**, the clock will take about 5 seconds to automatically scan the surrounding WiFi  

![wifi4](/img/wifi4.png)

Select your WiFi SSID and enter the password, if it is not scanned, you can click the **Refresh** button below to rescan. If the WiFi is a hidden access point, enter the SSID and password manually and click **Submit**  

![wifi5](/img/wifi5.png)

When this page appears, it means that the clock has received WiFi information and is trying to connect to WiFi. If the connection is successful, the clock will be able to be used normally; if it cannot be connected, please reconfigure the Wifi. The reasons for the unsuccessful connection are:  
- Wrong WiFi password
- The name of the hidden access point was entered incorrectly  
- Configured WiFi is a 5GHz access point  
`ESP32 only supports 2.4GHz WiFi`

## Erase Configured WiFi

You can erase WiFi information on the configuration page. For how to enter the configuration page, please refer to the subsequent chapters
