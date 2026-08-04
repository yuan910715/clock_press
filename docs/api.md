<script setup>
import ApiLayoutControls from './.vitepress/components/ApiLayoutControls.vue'
</script>

# API

<ApiLayoutControls
  group-label="API 页面布局"
  hide-left-label="隐藏左侧菜单"
  show-left-label="显示左侧菜单"
  hide-right-label="隐藏右侧目录"
  show-right-label="显示右侧目录"
/>

如果你有开发能力，想集成时钟的配置在你的App、Home Assistant、Node-RED、脚本或其他系统中，请参考此API文档

本文档包含两类相互独立的接口
- 部署在公网服务器上的云端发现 API
- 由时钟 ESP32 在局域网内提供的设备 API

`二者的主机地址、传输协议、响应格式和 CORS 策略均不同，请勿混用`


## 1. API 分类

| 类别 | 服务位置 | 基础地址 | 响应格式 | CORS | 主要用途 |
| --- | --- | --- | --- | --- | --- |
| 云端发现 API | 公网服务器 | `https://topyuan.top/clock/findapi` | JSON 响应体 | 允许任意来源 `*` | 根据公网出口 IP 获取时钟的局域网 IP |
| 设备本地 API | 时钟 ESP32 | `http://<时钟的局域网 IP>` | 配置数据位于 HTTP Header，响应体通常为空 | 不返回 CORS Header | 读取配置、修改设置、读取 ADC、重启或清除 Wi-Fi |

推荐调用顺序：先请求云端发现 API 获得候选 `localIp`，然后在当前局域网内请求 `http://<localIp>/get` 等API

`如果你有其他方式可获取到局域网内的时钟ip，也可以不请求云端发现api`

## 2. 云端发现 API

### 2.1 通用约定

- 完整地址：`https://topyuan.top/clock/findapi`
- 服务位置：公网服务器，不在时钟 ESP32 上
- 传输协议：HTTPS
- 请求方式：`GET`
- 身份认证：无
- 成功响应：`200 OK` 和 JSON 数组
- 内容类型：`application/json; charset=utf-8`
- CORS：`Access-Control-Allow-Origin: *`
- 缓存策略：`Cache-Control: no-store`

### 2.2 发现设备：`GET /clock/findapi`

App 或其他客户端可以先调用此接口，取得可能位于当前局域网内的时钟 IP，再使用返回的 `localIp` 调用设备的 `/get`、`/set` 等接口

### 2.3 工作原理

时钟会定期把自身的公网 IP、局域网 IP 和设备信息上报到服务器。发现接口使用调用者的 `REMOTE_ADDR` 作为公网 IP，查询数据库中满足以下条件的设备：

1. 设备上报时的公网 IP 与接口调用者的公网 IP 相同
2. 设备在最近 12 小时内有过上报
3. 设备上报了有效的 IPv4 局域网地址

接口按最后上报时间从新到旧排列结果

### 2.4 请求

```http
GET /clock/findapi.php HTTP/1.1
Host: topyuan.top
```

无 Query 参数和请求体。公网 IP 由服务器根据当前连接自动取得，客户端不能通过参数指定要查询的公网 IP

```bash
curl -i https://topyuan.top/clock/findapi
```

### 2.5 成功响应

- 状态码：`200 OK`
- `Content-Type: application/json; charset=utf-8`
- `Access-Control-Allow-Origin: *`
- 响应体：JSON 数组
- 缓存策略：`Cache-Control: no-store`

```json
[
  {
    "chipId": "9730432",
    "localIp": "192.168.31.247",
    "deviceType": "ClockWise Plus",
    "lastSeen": "2026-08-04 15:26:30"
  },
  {
    "chipId": "10557104",
    "localIp": "192.168.31.180",
    "deviceType": "SuperY",
    "lastSeen": "2026-08-04 15:20:12"
  }
]
```

| JSON 字段 | 类型 | 说明 |
| --- | --- | --- |
| `chipId` | 字符串 | 时钟芯片 ID |
| `localIp` | 字符串 | 时钟上报的局域网 IPv4 地址，例如 `192.168.1.50` |
| `deviceType` | 字符串 | `ClockWise Plus`、`SuperY` 或 `SuperY Lite` |
| `lastSeen` | 字符串 | 服务器记录的最后上报时间，格式为 `YYYY-MM-DD HH:mm:ss` |

`deviceType`为`ClockWise Plus`的设备适用于本文档的时钟本地API，如果你还使用了我的其他类型时钟，云端发现API还可以查到其他类型的设备

没有找到设备时，接口返回空数组：

```json
[]
```

### 2.6 错误响应

| 状态码 | 响应示例 | 含义 |
| --- | --- | --- |
| `400` | `{"error":"invalid_client_ip"}` | 服务器无法取得有效的访问者 IP |
| `405` | `{"error":"method_not_allowed"}` | 使用了 `GET`、`OPTIONS` 以外的请求方式 |
| `500` | `{"error":"discovery_unavailable"}` | 服务器错误 |

### 2.7 发现结果的限制

“公网 IP 相同”只能作为设备可能处于同一局域网的判断依据，不能作为严格证明。运营商 CGNAT、企业网络、校园网、VPN 或代理可能让互不相邻的客户端共享公网 IP；反过来，IPv4/IPv6 出口不同也可能导致同一局域网内的设备无法匹配

该接口开放通配 CORS，浏览器 Web App 可以跨域直接调用。接口允许 `GET` 和 `OPTIONS`，浏览器预检请求返回 `204 No Content`

## 3. 设备本地 API 通用约定

本节及后续设备接口均由时钟 ESP32 提供，与第 2 节的公网云端发现服务无关

### 3.1 连接与数据格式

- 基础地址：`http://<时钟的局域网 IP>`，例如 `http://192.168.1.50`
- 服务位置：时钟 ESP32
- 端口：`80`
- 传输协议：HTTP，不支持 HTTPS
- 身份认证：无
- 写入格式：`application/x-www-form-urlencoded`，表单类型提交请求，不支持 JSON 请求体
- 成功响应：读取和控制接口通常返回 `204 No Content`，响应体为空
- 字符编码：字符串参数使用 UTF-8，并进行 URL 编码
- CORS：固件不返回 CORS Header

> 安全提示：任何能访问时钟局域网 IP 的客户端都可以修改设置、重启设备或清除 Wi-Fi 凭据。请只在可信局域网内开放这些接口，不要直接映射到公网

### 3.2 为什么通过 HTTP Header 返回数据

受 ESP32 运行内存、响应缓冲区以及 JSON 编码性能限制，固件没有采用常见的 JSON 响应体，而是将配置项直接放入 HTTP 响应 Header，并返回空的 `204 No Content`。这是一种针对嵌入式设备资源约束所作的实现取舍，并非常规 REST API 设计。第三方客户端必须读取响应 Header，不能依赖响应体或 JSON 解析

### 3.3 设备接口总览

下表所有路径都相对于 `http://<时钟的局域网 IP>`：

| 请求方式 | 路径 | 用途 | 成功响应 |
| --- | --- | --- | --- |
| `GET` | `/get` | 读取当前全部配置和设备信息 | `204`，数据位于响应 Header |
| `POST` | `/set` | 修改一个或多个配置项 | `204`，无响应体 |
| `GET` | `/read?pin=<GPIO>` | 读取指定 GPIO 的 ADC 值 | `204`，结果位于响应 Header `pin` |
| `POST` | `/restart` | 立即重启设备 | `204`，随后连接断开 |
| `POST` | `/erase` | 清除 Wi-Fi SSID/密码并重启 | `204`，随后连接断开 |

### 3.4 重要兼容说明

1. HTTP Header 名称不区分大小写。部分客户端会把 `displayBright` 自动转成 `displaybright`，调用方必须按不区分大小写的方式读取
2. `/get` 和 `/read` 的数据在响应 Header 中，响应体始终为空。不要尝试把响应解析成 JSON
3. 设备固件未返回 CORS Header。浏览器中由其他域名、端口或协议加载的网页不能直接读取这些响应；原生 App、后端服务、命令行程序以及时钟自身页面不受此限制
4. `/set` 不返回逐字段校验结果。未知字段会被忽略，部分非法数值会被修正，另一些可能被转换为 `0` 或发生整数截断。建议写入后再次调用 `/get` 确认实际值
5. 表单中的 `+` 会被解释为空格。例如时区 `+8:00` 必须编码为 `%2B8%3A00`。使用 `curl --data-urlencode` 可自动完成编码
6. `/set` 支持部分更新：未提交的字段保持不变。值为空字符串的字段也会被视为“未提交”，因此当前接口不能把字符串配置清空

## 4. 读取全部配置：`GET /get`

### 请求

```http
GET /get HTTP/1.1
Host: 192.168.1.50
```

无请求参数和请求体

### 响应

状态码为 `204 No Content`，所有数据通过自定义响应 Header 返回。这是为了避免 ESP32 生成和缓冲 JSON 响应带来的额外性能与内存开销。例如：

```http
HTTP/1.1 204 No Content
displayBright: 205
autoBrightMin: 30
autoBrightMax: 2000
wifiSsid: MyWiFi
clockFace: 1
version: 4.2
```

以下是完整 Header 列表。表中的默认值指首次启动或该配置尚未保存时的固件默认值

| 响应 Header | 类型 | 含义与取值 | 默认值 | `/set` 对应字段 |
| --- | --- | --- | --- | --- |
| `displayBright` | 整数 | 显示亮度，`0`～`255`。自动/定时模式下作为日间或最大亮度 | `205` | `displayBright` |
| `autoBrightMin` | 整数 | 自动亮度的夜间 LDR 阈值，`1`～`300` | `30` | 与 `autoBrightMax` 合并为 `autoBright` |
| `autoBrightMax` | 整数 | 自动亮度的明亮环境 LDR 阈值，`800`～`4095` | `2000` | 与 `autoBrightMin` 合并为 `autoBright` |
| `specialLed` | 枚举整数 | LED 面板颜色接线：`0` RGB、`1` RBG、`2` GBR | `0` | `specialLed` |
| `use24hFormat` | 布尔整数 | `1` 使用 24 小时制，`0` 使用 12 小时制 | `1` | `use24hFormat` |
| `ldrPin` | 整数 | 光敏电阻所接 GPIO；当前硬件页面固定使用 GPIO 35 | `35` | `ldrPin` |
| `wifiSsid` | 字符串 | 设备当前连接的 Wi-Fi SSID | 当前连接 | 只读 |
| `ntpServer` | 字符串 | NTP 服务器主机名或 IP | `ntp2.aliyun.com` | `ntpServer` |
| `displayRotation` | 枚举整数 | 屏幕旋转：`0`=0°、`1`=90°、`2`=180°、`3`=270° | `0` | `displayRotation` |
| `clockFace` | 枚举整数 | 当前表盘编号，`1`～`27`，详见表盘编号表 | `1` | `clockFace` |
| `language` | 枚举整数 | 配置页语言：`0` 中文、`1` English | `0` | `language` |
| `totalYear` | 整数 | 累计运行年数 | `0` | 只读 |
| `totalMonth` | 整数 | 累计运行月数 | `0` | 只读 |
| `totalDay` | 整数 | 累计运行天数 | `0` | 只读 |
| `brightMethod` | 枚举整数 | 亮度模式：`0` 环境光自动调节、`1` 按时间段调节、`2` 固定亮度 | `0` | `brightMethod` |
| `nightLevel` | 整数 | 定时亮度模式下的夜间亮度等级，`1`～`5` | `1` | `nightLevel` |
| `nightStarth` | 整数 | 夜间时段开始小时，建议 `0`～`23` | `22` | `nightStarth` |
| `nightStartm` | 整数 | 夜间时段开始分钟，建议 `0`～`59` | `0` | `nightStartm` |
| `nightEndh` | 整数 | 夜间时段结束小时，建议 `0`～`23` | `8` | `nightEndh` |
| `nightEndm` | 整数 | 夜间时段结束分钟，建议 `0`～`59` | `0` | `nightEndm` |
| `sqtext` | 字符串 | 固定 UTC 偏移显示值，例如 `+8:00`、`-3:30`；用于 `timemode=0` | `+8:00` | `sqtext` |
| `timemode` | 枚举整数 | 时区模式：`0` 固定 UTC 偏移、`1` 带夏令时规则的 POSIX 时区 | `0` | `timemode` |
| `posix` | 字符串 | 实际交给时钟库的 POSIX TZ 字符串 | `<+8>-8` | `posix` |
| `autoChange` | 枚举整数 | 每天 00:00 自动换表盘：`0` 关闭、`1` 顺序、`2` 随机 | `1` | `autoChange` |
| `faceControl` | 字符串 | 27 位表盘启用掩码；从左至右对应表盘 1～27，`1` 启用、`0` 禁用 | 27 个 `1` | `faceControl` |
| `reversePhase` | 布尔整数 | HUB75 时钟相位：`1` 反相、`0` 正常 | `0` | `reversePhase` |
| `nightMode` | 枚举整数 | 夜间策略：`0` 无、`1` 关闭 LED、`2` 显示超大时钟 | `2` | `nightMode` |
| `superColor` | 整数 | 超大时钟颜色，RGB565 十进制值，`0`～`65535` | `16936` | `superColor` |
| `version` | 字符串 | 当前固件版本 | 当前版本 | 只读 |

命令行读取示例：

```bash
curl -i http://192.168.1.50/get
```

## 5. 修改配置：`POST /set`

### 请求格式

```http
POST /set HTTP/1.1
Host: 192.168.1.50
Content-Type: application/x-www-form-urlencoded

displayBright=180&use24hFormat=1
```

一次请求可以提交一个或多个字段。成功时返回 `204 No Content`，无响应体，也不通过自定义 Header 回显配置结果

### 可写字段

| 表单字段 | 输入格式 | 合法/推荐值 | 说明 |
| --- | --- | --- | --- |
| `displayBright` | 十进制整数 | `0`～`255` | 设置显示亮度 |
| `autoBright` | 固定格式字符串 | `MMMM,XXXX` | 同时设置 LDR 最小/最大值；两个数都必须补足 4 位，例如 `0030,2000` |
| `specialLed` | 整数 | `0`、`1`、`2` | RGB / RBG / GBR 面板接线 |
| `reversePhase` | 布尔字符串 | `0` 或 `1` | 反转相位 |
| `use24hFormat` | 布尔字符串 | `0` 或 `1` | 使用24小时制 |
| `ldrPin` | 整数 | 当前硬件为 `35` | 更改光敏电阻 GPIO；固件不验证该引脚是否支持 ADC |
| `ntpServer` | 字符串 | 有效 NTP 主机名/IP | 修改后立即同步 |
| `displayRotation` | 整数 | `0`～`3` | 分别表示 0°、90°、180°、270° |
| `clockFace` | 整数 | `1`～`27` | 切换当前表盘 |
| `language` | 整数 | `0` 或 `1` | 修改配置页语言；重新打开配置页面后生效 |
| `brightMethod` | 整数 | `0`、`1`、`2` | 自动、定时、固定亮度 |
| `nightLevel` | 整数 | `1`～`5` | 超出范围会被改为 `1` |
| `nightStarth` | 整数 | `0`～`23` | 夜间开始小时；固件不做范围检查 |
| `nightStartm` | 整数 | `0`～`59` | 夜间开始分钟；固件不做范围检查 |
| `nightEndh` | 整数 | `0`～`23` | 夜间结束小时；固件不做范围检查 |
| `nightEndm` | 整数 | `0`～`59` | 夜间结束分钟；固件不做范围检查 |
| `sqtext` | URL 编码字符串 | 如 `+8:00` | 固定偏移模式的显示值；需与 `posix` 同步提交 |
| `timemode` | 整数 | `0` 或 `1` | 固定偏移 / 智能夏令时 |
| `posix` | URL 编码字符串 | 合法 POSIX TZ | 固件不校验语法，需自行校验，具体请参考后续正则表达式 |
| `autoChange` | 整数 | `0`、`1`、`2` | 关闭 / 顺序 / 随机换表盘 |
| `faceControl` | 27 字符字符串 | 仅使用 `0`、`1` | 第 N 位控制第 N 个表盘；建议至少启用两个表盘 |
| `nightMode` | 整数 | `0`、`1`、`2` | 无策略 / 关屏 / 超大时钟 |
| `superColor` | 十进制整数 | `0`～`65535` | 超大时钟 RGB565 颜色 |

`wifiSsid`、`totalYear`、`totalMonth`、`totalDay` 和 `version` 只读，提交到 `/set` 不会产生作用。`autoBrightMin` 和 `autoBrightMax` 也不能单独提交，必须使用组合字段 `autoBright`

`posix`校验正则表达式：

^([a-zA-Z]{1,6}|<[a-zA-Z0-9+-]{1,6}>)([+-]?([0-9]|1[0-4])(:[0-5]\d)?)((([a-zA-Z]{1,6}|<[a-zA-Z0-9+-]{1,6}>)([+-]?([0-9]|1[0-4])(:[0-5]\d)?)?)(,M([1-9]|1[0-2]).([1-5]).([0-6])(\/([0-9]|1[0-9]|2[0-4])(:[0-5]\d)?)?,M([1-9]|1[0-2]).([1-5]).([0-6])(\/([0-9]|1[0-9]|2[0-4])(:[0-5]\d)?)?)?)?$

### 5.1 常用写入示例

设置亮度和 24 小时制：

```bash
curl -i -X POST http://192.168.1.50/set \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "displayBright=180&use24hFormat=1"
```

设置自动亮度阈值。`autoBright` 的精确格式是 4 位最小值、英文逗号、4 位最大值：

```bash
curl -i -X POST http://192.168.1.50/set \
  --data-urlencode "autoBright=0030,2000"
```

设置固定 UTC+8 时区。建议将三个相关字段放在同一个请求中：

```bash
curl -i -X POST http://192.168.1.50/set \
  --data-urlencode "sqtext=+8:00" \
  --data-urlencode "timemode=0" \
  --data-urlencode "posix=<+8>-8"
```

设置带夏令时的 POSIX 时区：

```bash
curl -i -X POST http://192.168.1.50/set \
  --data-urlencode "timemode=1" \
  --data-urlencode "posix=EST5EDT,M3.2.0,M11.1.0"
```

设置夜间时段为 22:30～07:00，夜间关闭屏幕：

```bash
curl -i -X POST http://192.168.1.50/set \
  --data "nightLevel=1&nightStarth=22&nightStartm=30&nightEndh=7&nightEndm=0&nightMode=1"
```

启用顺序自动换表盘，并只允许表盘 1、2、3 参与轮换：

```bash
curl -i -X POST http://192.168.1.50/set \
  --data "autoChange=1&faceControl=111000000000000000000000000"
```

### 5.2 固件实际校正行为

- `autoBright` 使用固定位置解析：前 4 个字符为最小值，第 6～9 个字符为最大值，因此必须按 `0030,2000` 这样的 9 字符格式提交
- `autoBrightMin` 小于 `1` 会改为 `1`，大于 `300` 会改为 `300`
- `autoBrightMax` 小于 `800` 会改为 `800`，大于 `4095` 会改为 `4095`
- `nightLevel` 不在 `1`～`5` 时会改为 `1`
- 其他多数整数参数没有范围校验；调用方应自行保证取值有效
- 未知字段和只读字段不会报错，响应仍可能是 `204`

## 6. 读取 ADC：`GET /read`

读取指定 GPIO 的 `analogRead()` 原始值，可获取光敏电阻采集的ADC值

### 输入

| 名称 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `pin` | 整数 | 是 | GPIO 编号；当前硬件的 LDR 默认接 GPIO 35 |

### 输出

- 状态码：`204 No Content`
- 响应体：空
- 响应 Header：`pin: <ADC 原始值>`

示例：

```bash
curl -i "http://192.168.1.50/read?pin=35"
```

```http
HTTP/1.1 204 No Content
pin: 1842
```

固件会把参数转换为 8 位无符号整数后直接调用 `analogRead()`，不检查引脚是否合法。第三方客户端不应允许最终用户任意传入 GPIO，建议固定使用 `/get` 返回的 `ldrPin`

## 7. 重启设备：`POST /restart`

无参数、无请求体。固件先尝试发送 `204`，随后立即执行重启

```bash
curl -i -X POST http://192.168.1.50/restart
```

重启会使连接立即断开；个别客户端可能在完整收到响应前报告网络断开

## 8. 清除 Wi-Fi：`POST /erase`

无参数、无请求体。接口只清除当前保存的 Wi-Fi SSID 和密码，然后立即重启

```bash
curl -i -X POST http://192.168.1.50/erase
```

调用后设备无法再通过原 IP 访问，并会重新进入 Wi-Fi 配网流程。这是破坏性操作，调用前应增加二次确认

## 9. 表盘编号

| 编号 | 表盘 | 编号 | 表盘 |
| --- | --- | --- | --- |
| 1 | Super Mario | 15 | Shar Pei Dog |
| 2 | Pac Man | 16 | Girl |
| 3 | World Map | 17 | Kirby |
| 4 | Time In Words | 18 | Labubu-Zimomo |
| 5 | Clock Tower | 19 | Hello Kitty |
| 6 | Pokedex | 20 | Twinkle Twinkle |
| 7 | Retro Computer | 21 | Zootopia |
| 8 | Snoopy | 22 | Minecraft-Village |
| 9 | Nyan Cat | 23 | Codex |
| 10 | Transformer | 24 | Rainy Window |
| 11 | Minecraft-Torch | 25 | GTA VI |
| 12 | Coffee | 26 | Zelda-Sunrise |
| 13 | Pepsi | 27 | Particle-Time |
| 14 | Pikachu |  |  |

<style>
/* API 页面使用更宽的正文区域，同时保留右侧目录。 */
@media (min-width: 1280px) {
  .VPDoc.has-aside:has(.vp-doc._clock_api) .content-container {
    max-width: 900px;
  }
}

/* 桌面端布局按钮只影响当前 API 页面。 */
@media (min-width: 960px) {
  html.api-layout-hide-left:has(.vp-doc._clock_api) .VPSidebar {
    display: none !important;
  }

  html.api-layout-hide-left:has(.vp-doc._clock_api) .VPContent.has-sidebar {
    padding-left: 0 !important;
  }

  html.api-layout-hide-right:has(.vp-doc._clock_api) .VPDoc > .container > .aside {
    display: none !important;
  }

  html.api-layout-hide-left:has(.vp-doc._clock_api) .content-container,
  html.api-layout-hide-right:has(.vp-doc._clock_api) .content-container {
    max-width: 1050px;
  }

  html.api-layout-hide-left.api-layout-hide-right:has(.vp-doc._clock_api) .content-container {
    max-width: 1200px;
  }
}

@media (min-width: 1440px) {
  html.api-layout-hide-left:has(.vp-doc._clock_api) .VPContent.has-sidebar {
    padding-left: calc((100vw - var(--vp-layout-max-width)) / 2) !important;
  }
}

.vp-doc._clock_api table {
  width: 100%;
  scrollbar-gutter: stable;
}

.vp-doc._clock_api table th,
.vp-doc._clock_api table td {
  min-width: 0;
  padding: 8px 10px;
  vertical-align: top;
  line-height: 1.5;
  white-space: normal;
}

.vp-doc._clock_api table th {
  min-width: 4.5rem;
}

/* 短字段保持单行，说明类字段允许自然换行。 */
.vp-doc._clock_api table:nth-of-type(1) :is(th, td):nth-child(1) { min-width: 7rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(1) :is(th, td):nth-child(2) { min-width: 6.5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(1) :is(th, td):nth-child(3) { min-width: 15rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(1) :is(th, td):nth-child(4) { min-width: 8rem; }
.vp-doc._clock_api table:nth-of-type(1) :is(th, td):nth-child(5) { min-width: 8rem; }
.vp-doc._clock_api table:nth-of-type(1) :is(th, td):nth-child(6) { min-width: 12rem; }

.vp-doc._clock_api table:nth-of-type(2) :is(th, td):nth-child(1) { min-width: 8rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(2) :is(th, td):nth-child(2) { min-width: 5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(2) :is(th, td):nth-child(3) { min-width: 16rem; }

.vp-doc._clock_api table:nth-of-type(3) :is(th, td):nth-child(1) { min-width: 4.5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(3) :is(th, td):nth-child(2) { min-width: 15rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(3) :is(th, td):nth-child(3) { min-width: 12rem; }

.vp-doc._clock_api table:nth-of-type(4) :is(th, td):nth-child(1) { min-width: 6rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(4) :is(th, td):nth-child(2) { min-width: 10rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(4) :is(th, td):nth-child(3) { min-width: 9rem; }
.vp-doc._clock_api table:nth-of-type(4) :is(th, td):nth-child(4) { min-width: 11rem; }

.vp-doc._clock_api table:nth-of-type(5) :is(th, td):nth-child(1) { min-width: 8rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(5) :is(th, td):nth-child(2) { min-width: 5.5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(5) :is(th, td):nth-child(3) { min-width: 16rem; }
.vp-doc._clock_api table:nth-of-type(5) :is(th, td):nth-child(4) { min-width: 6rem; }
.vp-doc._clock_api table:nth-of-type(5) :is(th, td):nth-child(5) { min-width: 10rem; }

.vp-doc._clock_api table:nth-of-type(6) :is(th, td):nth-child(1) { min-width: 9rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(6) :is(th, td):nth-child(2) { min-width: 7rem; }
.vp-doc._clock_api table:nth-of-type(6) :is(th, td):nth-child(3) { min-width: 9rem; }
.vp-doc._clock_api table:nth-of-type(6) :is(th, td):nth-child(4) { min-width: 16rem; }

.vp-doc._clock_api table:nth-of-type(7) :is(th, td):nth-child(1) { min-width: 4.5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(7) :is(th, td):nth-child(2) { min-width: 4.5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(7) :is(th, td):nth-child(3) { min-width: 4.5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(7) :is(th, td):nth-child(4) { min-width: 14rem; }

.vp-doc._clock_api table:nth-of-type(8) :is(th, td):nth-child(1),
.vp-doc._clock_api table:nth-of-type(8) :is(th, td):nth-child(3) { min-width: 4.5rem; white-space: nowrap; }
.vp-doc._clock_api table:nth-of-type(8) :is(th, td):nth-child(2),
.vp-doc._clock_api table:nth-of-type(8) :is(th, td):nth-child(4) { min-width: 9rem; white-space: nowrap; }

/* 桌面端横向滚动时固定首列，始终保留当前字段上下文。 */
@media (min-width: 768px) {
  .vp-doc._clock_api table th:first-child,
  .vp-doc._clock_api table td:first-child {
    position: sticky;
    left: 0;
    z-index: 1;
  }

  .vp-doc._clock_api table th:first-child {
    z-index: 2;
    background: var(--vp-c-bg-soft);
  }

  .vp-doc._clock_api table tbody tr:nth-child(odd) td:first-child {
    background: var(--vp-c-bg);
  }

  .vp-doc._clock_api table tbody tr:nth-child(even) td:first-child {
    background: var(--vp-c-bg-soft);
  }
}

/* 手机端将两张最长的字段表转换成标签-值卡片。 */
@media (max-width: 767px) {
  .vp-doc._clock_api table:nth-of-type(5),
  .vp-doc._clock_api table:nth-of-type(6) {
    display: block;
    width: 100%;
    overflow: visible;
    border-collapse: separate;
  }

  .vp-doc._clock_api table:nth-of-type(5) thead,
  .vp-doc._clock_api table:nth-of-type(6) thead {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
    white-space: nowrap;
    clip-path: inset(50%);
  }

  .vp-doc._clock_api table:nth-of-type(5) tbody,
  .vp-doc._clock_api table:nth-of-type(6) tbody {
    display: block;
  }

  .vp-doc._clock_api table:nth-of-type(5) tr,
  .vp-doc._clock_api table:nth-of-type(6) tr {
    display: block;
    margin: 16px 0;
    overflow: hidden;
    border: 1px solid var(--vp-c-divider);
    border-radius: 10px;
    background: var(--vp-c-bg);
  }

  .vp-doc._clock_api table:nth-of-type(5) td,
  .vp-doc._clock_api table:nth-of-type(6) td {
    display: grid;
    grid-template-columns: minmax(6.5rem, 34%) minmax(0, 1fr);
    gap: 12px;
    width: 100%;
    min-width: 0;
    padding: 9px 12px;
    border: 0;
    border-bottom: 1px solid var(--vp-c-divider);
    background: transparent;
    white-space: normal;
    overflow-wrap: anywhere;
  }

  .vp-doc._clock_api table:nth-of-type(5) td:last-child,
  .vp-doc._clock_api table:nth-of-type(6) td:last-child {
    border-bottom: 0;
  }

  .vp-doc._clock_api table:nth-of-type(5) td::before,
  .vp-doc._clock_api table:nth-of-type(6) td::before {
    color: var(--vp-c-text-2);
    font-weight: 600;
  }

  .vp-doc._clock_api table:nth-of-type(5) td:nth-child(1)::before { content: "响应 Header"; }
  .vp-doc._clock_api table:nth-of-type(5) td:nth-child(2)::before { content: "类型"; }
  .vp-doc._clock_api table:nth-of-type(5) td:nth-child(3)::before { content: "含义与取值"; }
  .vp-doc._clock_api table:nth-of-type(5) td:nth-child(4)::before { content: "默认值"; }
  .vp-doc._clock_api table:nth-of-type(5) td:nth-child(5)::before { content: "/set 对应字段"; }

  .vp-doc._clock_api table:nth-of-type(6) td:nth-child(1)::before { content: "表单字段"; }
  .vp-doc._clock_api table:nth-of-type(6) td:nth-child(2)::before { content: "输入格式"; }
  .vp-doc._clock_api table:nth-of-type(6) td:nth-child(3)::before { content: "合法/推荐值"; }
  .vp-doc._clock_api table:nth-of-type(6) td:nth-child(4)::before { content: "说明"; }
}
</style>
