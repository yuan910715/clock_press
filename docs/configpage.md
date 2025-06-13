# 进入配置页

所有对时钟的设置都在配置页完成，配置页由时钟提供，需知道时钟的IP地址后，使用同WiFi下的手机/PC浏览器访问此IP即可进入配置页

## 通过topyuan获取IP

访问https://topyuan.top/clock/find 服务器会给出你的时钟IP  
`此功能通过判断访问者公网IP和时钟上报时的公网IP对比来判断返回哪个时钟的本地IP，访问此页面的设备需和时钟处在同一WiFi下`

## 通过LED获取IP

每次给时钟通电，当成功连接WiFi后，LED会显示时钟的IP地址  

![configpage1](/img/configpage1.png)

## 通过路由器获取IP
你也可以在你自己的路由器管理页面/App查看时钟ip  
`为防止时钟IP变化，你也可以在此将其设置为固定静态ip，大部分Linux内核路由器默认为静态ip`