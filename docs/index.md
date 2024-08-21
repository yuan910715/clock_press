# 简介

## ClockWise项目

![ClockWise Logo](/img/clockwise_logo.png)
ClockWise是巴西人**Jonathas Barbosa**开发的开源项目，项目代码在github托管，仓库地址: https://github.com/jnthas/clockwise 我本人也在项目贡献者列表中，提交过部分优化代码

项目使用64x64像素的LED点阵进行显示，使用ESP32进行驱动，实现了时钟显示引擎以及多个表盘皮肤

项目更适合于软/硬件专业人员，需完成准备硬件、接线、web页面选择表盘、刷写固件、串行通讯配网，完成后在配置页面配置时区、NTP服务器等参数后，时钟可正常使用

## ClockWise升级版

我在使用几个月ClockWise时钟后，进行了部分优化，已提交至原作者代码仓库中

另一个想法是想将此项目修改为更适合普通用户使用  
`和我自己的开源项目SuperY Wifi Clock思路一致，https://github.com/yuan910715/Esp8266_Wifi_Matrix_Clock`

设计好电路板，硬件焊接、固件刷写由专业人员完成，普通用户只需在配置页进行配置，更换表盘也不需重刷固件。由于此思路与原作者项目有较大偏差，我将此项目独立，也就是ClockWise的升级版，遵循原项目MIT开源协议

### 升级点

升级版按照我的思路在代码层面做了大量优化，主要如下：

- **表盘皮肤集成**  
原项目的每个表盘皮肤实际是一个子代码仓库，也就是每个固件实际只编译了一个表盘皮肤，如需换肤需要重新刷写固件。我完成了实时换表盘逻辑的编写，将现有表盘皮肤全部编译，在配置页可随时自由更换表盘
- **配网方式**  
去除串口配网，配置wifi只需使用手机连接时钟发射的热点即可完成
- **本地化**  
配置页改为中文/英文可切换，默认时区改为Asia/Shanghai
- **配置页优化**  
配置项优化，新增更新日志、擦除wifi按钮、运行时长显示等，js文件放入远程服务器以防止ESP32提供大页面时崩溃
- **表盘皮肤优化**  
修改表盘重写时间逻辑，时钟达到毫秒级精度。马里奥表盘增加秒数隐藏显示
- **OTA在线更新**  
具备可持续更新能力，开发了OTA升级功能，有新固件发布时，无需重新刷写固件，时钟可自动完成更新，后续可持续增加配置项、优化功能、增加表盘皮肤……
![ota](/img/ota.png)

### 实拍图
![Real1](/img/real1.png)

![Real2](/img/real2.png)

![Real3](/img/real3.png)

![Real4](/img/real4.png)

![Real5](/img/real5.png)

![Real6](/img/real6.png)

![all](/img/all.png)

## 项目成员

<script setup>
import { VPTeamMembers } from 'vitepress/theme'

const members = [
  {
    avatar: 'https://topyuan.top/yuan.png',
    name: '冯雪原',
    title: 'Creator',
    links: [
      { icon: 'github', link: 'https://github.com/yuan910715' }
    ]
  },
]
</script>

<VPTeamMembers size="small" :members="members" />

期待你的加入