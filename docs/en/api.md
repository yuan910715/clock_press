<script setup>
import ApiLayoutControls from '../.vitepress/components/ApiLayoutControls.vue'
</script>

# API

<ApiLayoutControls
  group-label="API page layout"
  hide-left-label="Hide left menu"
  show-left-label="Show left menu"
  hide-right-label="Hide page outline"
  show-right-label="Show page outline"
/>

If you are a developer and want to integrate clock configuration into your app, Home Assistant, Node-RED, scripts, or another system, use this API reference.

This document covers two independent API categories:

- The Cloud Discovery API, hosted on a public server
- The Device API, provided by the clock's ESP32 on the local network

`These APIs use different hosts, transport protocols, response formats, and CORS policies. Do not mix them.`

## 1. API Categories

| Category | Service Location | Base URL | Response Format | CORS | Primary Use |
| --- | --- | --- | --- | --- | --- |
| Cloud Discovery API | Public server | `https://topyuan.top/clock/findapi` | JSON response body | Allows any origin: `*` | Obtain a clock's local IP address by matching its public egress IP |
| Device API | Clock ESP32 | `http://<clock-local-ip>` | Configuration data in HTTP response headers; response body is usually empty | No CORS headers | Read configuration, change settings, read ADC values, restart the device, or erase Wi-Fi credentials |

Recommended call sequence: first request the Cloud Discovery API to obtain candidate `localIp` values, then request `http://<localIp>/get` or another Device API endpoint from the current local network.

`If you can obtain the clock's local IP address by another method, you do not need to call the Cloud Discovery API.`

## 2. Cloud Discovery API

### 2.1 General Conventions

- Full URL: `https://topyuan.top/clock/findapi`
- Service location: public server, not the clock's ESP32
- Transport: HTTPS
- Method: `GET`
- Authentication: none
- Successful response: `200 OK` with a JSON array
- Content type: `application/json; charset=utf-8`
- CORS: `Access-Control-Allow-Origin: *`
- Cache policy: `Cache-Control: no-store`

### 2.2 Discover Devices: `GET /clock/findapi`

An app or other client can call this endpoint first to obtain the IP addresses of clocks that may be on the current local network. It can then use each returned `localIp` to call the device's `/get`, `/set`, and other endpoints.

### 2.3 How It Works

Each clock periodically reports its public IP address, local IP address, and device information to the server. The discovery endpoint uses the caller's `REMOTE_ADDR` as its public IP address and queries for devices that meet all of the following conditions:

1. The public IP reported by the device matches the caller's public IP
2. The device has reported within the last 12 hours
3. The device reported a valid local IPv4 address

Results are ordered by the most recent report time, newest first.

### 2.4 Request

```http
GET /clock/findapi.php HTTP/1.1
Host: topyuan.top
```

The request has no query parameters or body. The server derives the public IP address from the current connection; a client cannot specify a public IP address to query.

```bash
curl -i https://topyuan.top/clock/findapi
```

### 2.5 Successful Response

- Status: `200 OK`
- `Content-Type: application/json; charset=utf-8`
- `Access-Control-Allow-Origin: *`
- Body: JSON array
- Cache policy: `Cache-Control: no-store`

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

| JSON Field | Type | Description |
| --- | --- | --- |
| `chipId` | String | Clock chip ID |
| `localIp` | String | Local IPv4 address reported by the clock, for example `192.168.1.50` |
| `deviceType` | String | `ClockWise Plus`, `SuperY`, or `SuperY Lite` |
| `lastSeen` | String | Time of the last report recorded by the server, in `YYYY-MM-DD HH:mm:ss` format |

The Device API documented on this page applies to devices whose `deviceType` is `ClockWise Plus`. If you also use my other clock models, the Cloud Discovery API may return those device types as well.

If no devices are found, the endpoint returns an empty array:

```json
[]
```

### 2.6 Error Responses

| Status | Example Response | Meaning |
| --- | --- | --- |
| `400` | `{"error":"invalid_client_ip"}` | The server could not obtain a valid client IP address |
| `405` | `{"error":"method_not_allowed"}` | A method other than `GET` or `OPTIONS` was used |
| `500` | `{"error":"discovery_unavailable"}` | Server error |

### 2.7 Discovery Limitations

A matching public IP address only indicates that devices may be on the same local network; it is not definitive proof. Carrier-grade NAT (CGNAT), enterprise networks, campus networks, VPNs, or proxies may cause unrelated clients to share a public IP address. Conversely, different IPv4 and IPv6 egress paths may prevent devices on the same local network from matching.

This endpoint enables wildcard CORS, so browser-based web apps can call it cross-origin. It accepts `GET` and `OPTIONS`; browser preflight requests receive `204 No Content`.

## 3. Device API Conventions

This section and all subsequent device endpoints are served by the clock's ESP32. They are independent of the public Cloud Discovery API described in Section 2.

### 3.1 Connection and Data Format

- Base URL: `http://<clock-local-ip>`, for example `http://192.168.1.50`
- Service location: clock ESP32
- Port: `80`
- Transport: HTTP; HTTPS is not supported
- Authentication: none
- Write format: `application/x-www-form-urlencoded`; JSON request bodies are not supported
- Successful response: read and control endpoints usually return `204 No Content` with an empty body
- Character encoding: string parameters use UTF-8 and must be URL-encoded
- CORS: the firmware does not return CORS headers

> Security notice: Any client that can reach the clock's local IP address can change settings, restart the device, or erase its Wi-Fi credentials. Expose these endpoints only on a trusted local network. Never forward them directly to the public internet.

### 3.2 Why Data Is Returned in HTTP Headers

Because the ESP32 has constrained runtime memory and response-buffer capacity, and JSON encoding adds processing overhead, the firmware does not use a conventional JSON response body. Instead, it places configuration values directly in HTTP response headers and returns an empty `204 No Content` response. This is an implementation trade-off for a resource-constrained embedded device, not a conventional REST API design. Third-party clients must read the response headers and must not rely on a response body or JSON parsing.

### 3.3 Device Endpoint Overview

All paths in the following table are relative to `http://<clock-local-ip>`:

| Method | Path | Purpose | Successful Response |
| --- | --- | --- | --- |
| `GET` | `/get` | Read all current configuration and device information | `204`; data is in the response headers |
| `POST` | `/set` | Change one or more configuration values | `204`; no response body |
| `GET` | `/read?pin=<GPIO>` | Read the ADC value of a specified GPIO | `204`; result is in the `pin` response header |
| `POST` | `/restart` | Restart the device immediately | `204`; the connection then closes |
| `POST` | `/erase` | Erase the Wi-Fi SSID and password, then restart | `204`; the connection then closes |

### 3.4 Important Compatibility Notes

1. HTTP header names are case-insensitive. Some clients automatically convert `displayBright` to `displaybright`; clients must perform case-insensitive header lookups.
2. Data from `/get` and `/read` is returned in response headers. The response body is always empty; do not attempt to parse it as JSON.
3. The device firmware does not return CORS headers. A web page loaded from a different origin—another domain, port, or protocol—cannot read these responses directly in a browser. Native apps, backend services, command-line clients, and the clock's own web UI are not subject to this restriction.
4. `/set` does not return per-field validation results. Unknown fields are ignored; some invalid numeric values are corrected, while others may be converted to `0` or truncated. Call `/get` after writing to verify the effective values.
5. A `+` in form data is interpreted as a space. For example, the `+8:00` time-zone offset must be encoded as `%2B8%3A00`. Use `curl --data-urlencode` to encode values automatically.
6. `/set` supports partial updates: omitted fields remain unchanged. Fields whose values are empty strings are also treated as omitted, so the current API cannot clear a string-valued setting.

## 4. Read All Configuration: `GET /get`

### Request

```http
GET /get HTTP/1.1
Host: 192.168.1.50
```

The request has no parameters or body.

### Response

The status is `204 No Content`, and all data is returned in custom response headers. This avoids the additional processing and memory overhead of generating and buffering a JSON response on the ESP32. For example:

```http
HTTP/1.1 204 No Content
displayBright: 205
autoBrightMin: 30
autoBrightMax: 2000
wifiSsid: MyWiFi
clockFace: 1
version: 4.2
```

The complete header list follows. A default value is the firmware default used on first boot or when that setting has not yet been saved.

| Response Header | Type | Meaning and Values | Default | Corresponding `/set` Field |
| --- | --- | --- | --- | --- |
| `displayBright` | Integer | Display brightness, `0`–`255`. Used as the daytime or maximum brightness in automatic and scheduled modes | `205` | `displayBright` |
| `autoBrightMin` | Integer | Nighttime LDR threshold for automatic brightness, `1`–`300` | `30` | Combined with `autoBrightMax` as `autoBright` |
| `autoBrightMax` | Integer | Bright-environment LDR threshold for automatic brightness, `800`–`4095` | `2000` | Combined with `autoBrightMin` as `autoBright` |
| `specialLed` | Enum integer | LED panel color order: `0` RGB, `1` RBG, `2` GBR | `0` | `specialLed` |
| `use24hFormat` | Boolean integer | `1` for 24-hour time; `0` for 12-hour time | `1` | `use24hFormat` |
| `ldrPin` | Integer | GPIO connected to the photoresistor; the current hardware page specifies GPIO 35 | `35` | `ldrPin` |
| `wifiSsid` | String | SSID of the Wi-Fi network to which the device is currently connected | Current connection | Read-only |
| `ntpServer` | String | NTP server hostname or IP address | `ntp2.aliyun.com` | `ntpServer` |
| `displayRotation` | Enum integer | Display rotation: `0`=0°, `1`=90°, `2`=180°, `3`=270° | `0` | `displayRotation` |
| `clockFace` | Enum integer | Current clock-face number, `1`–`27`; see the clock-face table | `1` | `clockFace` |
| `language` | Enum integer | Configuration UI language: `0` Chinese, `1` English | `0` | `language` |
| `totalYear` | Integer | Accumulated runtime, years component | `0` | Read-only |
| `totalMonth` | Integer | Accumulated runtime, months component | `0` | Read-only |
| `totalDay` | Integer | Accumulated runtime, days component | `0` | Read-only |
| `brightMethod` | Enum integer | Brightness mode: `0` ambient-light adjustment, `1` scheduled adjustment, `2` fixed brightness | `0` | `brightMethod` |
| `nightLevel` | Integer | Nighttime brightness level in scheduled mode, `1`–`5` | `1` | `nightLevel` |
| `nightStarth` | Integer | Hour when the nighttime period starts; recommended range `0`–`23` | `22` | `nightStarth` |
| `nightStartm` | Integer | Minute when the nighttime period starts; recommended range `0`–`59` | `0` | `nightStartm` |
| `nightEndh` | Integer | Hour when the nighttime period ends; recommended range `0`–`23` | `8` | `nightEndh` |
| `nightEndm` | Integer | Minute when the nighttime period ends; recommended range `0`–`59` | `0` | `nightEndm` |
| `sqtext` | String | Display value for a fixed UTC offset, such as `+8:00` or `-3:30`; used when `timemode=0` | `+8:00` | `sqtext` |
| `timemode` | Enum integer | Time-zone mode: `0` fixed UTC offset, `1` POSIX time zone with daylight-saving rules | `0` | `timemode` |
| `posix` | String | POSIX TZ string passed to the clock library | `<+8>-8` | `posix` |
| `autoChange` | Enum integer | Change the clock face daily at 00:00: `0` disabled, `1` sequential, `2` random | `1` | `autoChange` |
| `faceControl` | String | 27-character clock-face enable mask; left to right corresponds to faces 1–27, where `1` enables and `0` disables a face | 27 `1` characters | `faceControl` |
| `reversePhase` | Boolean integer | HUB75 clock phase: `1` inverted, `0` normal | `0` | `reversePhase` |
| `nightMode` | Enum integer | Nighttime behavior: `0` none, `1` turn off the LED panel, `2` show the oversized clock | `2` | `nightMode` |
| `superColor` | Integer | Oversized-clock color as a decimal RGB565 value, `0`–`65535` | `16936` | `superColor` |
| `version` | String | Current firmware version | Current version | Read-only |

Command-line example:

```bash
curl -i http://192.168.1.50/get
```

## 5. Change Configuration: `POST /set`

### Request Format

```http
POST /set HTTP/1.1
Host: 192.168.1.50
Content-Type: application/x-www-form-urlencoded

displayBright=180&use24hFormat=1
```

A single request may include one or more fields. On success, the endpoint returns `204 No Content` with no response body and does not echo the resulting configuration in custom headers.

### Writable Fields

| Form Field | Input Format | Valid/Recommended Values | Description |
| --- | --- | --- | --- |
| `displayBright` | Decimal integer | `0`–`255` | Set display brightness |
| `autoBright` | Fixed-format string | `MMMM,XXXX` | Set the minimum and maximum LDR thresholds together; both values must be four digits, for example `0030,2000` |
| `specialLed` | Integer | `0`, `1`, or `2` | RGB / RBG / GBR panel color order |
| `reversePhase` | Boolean string | `0` or `1` | Invert the panel phase |
| `use24hFormat` | Boolean string | `0` or `1` | Use 24-hour time |
| `ldrPin` | Integer | `35` on the current hardware | Change the photoresistor GPIO; the firmware does not verify that the pin supports ADC |
| `ntpServer` | String | Valid NTP hostname or IP address | Synchronize immediately after changing the server |
| `displayRotation` | Integer | `0`–`3` | Represent 0°, 90°, 180°, and 270°, respectively |
| `clockFace` | Integer | `1`–`27` | Switch the current clock face |
| `language` | Integer | `0` or `1` | Change the configuration UI language; takes effect after reopening the configuration page |
| `brightMethod` | Integer | `0`, `1`, or `2` | Automatic, scheduled, or fixed brightness |
| `nightLevel` | Integer | `1`–`5` | Values outside the range are changed to `1` |
| `nightStarth` | Integer | `0`–`23` | Nighttime start hour; the firmware does not range-check this field |
| `nightStartm` | Integer | `0`–`59` | Nighttime start minute; the firmware does not range-check this field |
| `nightEndh` | Integer | `0`–`23` | Nighttime end hour; the firmware does not range-check this field |
| `nightEndm` | Integer | `0`–`59` | Nighttime end minute; the firmware does not range-check this field |
| `sqtext` | URL-encoded string | For example `+8:00` | Display value for fixed-offset mode; submit it together with `posix` |
| `timemode` | Integer | `0` or `1` | Fixed offset or daylight-saving-aware mode |
| `posix` | URL-encoded string | Valid POSIX TZ string | The firmware does not validate the syntax; validate it in the client using the regular expression below |
| `autoChange` | Integer | `0`, `1`, or `2` | Disable, sequential, or random clock-face switching |
| `faceControl` | 27-character string | Only `0` and `1` | Character N controls clock face N; enabling at least two faces is recommended |
| `nightMode` | Integer | `0`, `1`, or `2` | No action, turn off the display, or show the oversized clock |
| `superColor` | Decimal integer | `0`–`65535` | RGB565 color for the oversized clock |

`wifiSsid`, `totalYear`, `totalMonth`, `totalDay`, and `version` are read-only; submitting them to `/set` has no effect. `autoBrightMin` and `autoBrightMax` also cannot be submitted independently; use the combined `autoBright` field.

Regular expression for validating `posix`:

^([a-zA-Z]{1,6}|<[a-zA-Z0-9+-]{1,6}>)([+-]?([0-9]|1[0-4])(:[0-5]\d)?)((([a-zA-Z]{1,6}|<[a-zA-Z0-9+-]{1,6}>)([+-]?([0-9]|1[0-4])(:[0-5]\d)?)?)(,M([1-9]|1[0-2]).([1-5]).([0-6])(\/([0-9]|1[0-9]|2[0-4])(:[0-5]\d)?)?,M([1-9]|1[0-2]).([1-5]).([0-6])(\/([0-9]|1[0-9]|2[0-4])(:[0-5]\d)?)?)?)?$

### 5.1 Common Write Examples

Set brightness and 24-hour time:

```bash
curl -i -X POST http://192.168.1.50/set \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "displayBright=180&use24hFormat=1"
```

Set automatic-brightness thresholds. The exact `autoBright` format is a four-digit minimum, an ASCII comma, and a four-digit maximum:

```bash
curl -i -X POST http://192.168.1.50/set \
  --data-urlencode "autoBright=0030,2000"
```

Set a fixed UTC+8 time zone. Submit all three related fields in the same request:

```bash
curl -i -X POST http://192.168.1.50/set \
  --data-urlencode "sqtext=+8:00" \
  --data-urlencode "timemode=0" \
  --data-urlencode "posix=<+8>-8"
```

Set a POSIX time zone with daylight-saving rules:

```bash
curl -i -X POST http://192.168.1.50/set \
  --data-urlencode "timemode=1" \
  --data-urlencode "posix=EST5EDT,M3.2.0,M11.1.0"
```

Set the nighttime period to 22:30–07:00 and turn off the display at night:

```bash
curl -i -X POST http://192.168.1.50/set \
  --data "nightLevel=1&nightStarth=22&nightStartm=30&nightEndh=7&nightEndm=0&nightMode=1"
```

Enable sequential automatic clock-face switching and allow only faces 1, 2, and 3 in the rotation:

```bash
curl -i -X POST http://192.168.1.50/set \
  --data "autoChange=1&faceControl=111000000000000000000000000"
```

### 5.2 Firmware Coercion and Clamping Behavior

- `autoBright` is parsed at fixed character offsets: the first four characters are the minimum and characters 6–9 are the maximum. Submit exactly nine characters in the form `0030,2000`.
- `autoBrightMin` values below `1` are changed to `1`; values above `300` are changed to `300`.
- `autoBrightMax` values below `800` are changed to `800`; values above `4095` are changed to `4095`.
- A `nightLevel` outside `1`–`5` is changed to `1`.
- Most other integer parameters are not range-checked; clients must ensure that values are valid.
- Unknown and read-only fields do not produce an error; the response may still be `204`.

## 6. Read ADC: `GET /read`

Reads the raw `analogRead()` value for a specified GPIO. This can be used to obtain the ADC reading from the photoresistor.

### Input

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `pin` | Integer | Yes | GPIO number; the current hardware connects the LDR to GPIO 35 by default |

### Output

- Status: `204 No Content`
- Response body: empty
- Response header: `pin: <raw-ADC-value>`

Example:

```bash
curl -i "http://192.168.1.50/read?pin=35"
```

```http
HTTP/1.1 204 No Content
pin: 1842
```

The firmware converts the parameter to an unsigned 8-bit integer and passes it directly to `analogRead()` without checking whether the pin is valid. Third-party clients should not allow end users to supply an arbitrary GPIO number. Use the `ldrPin` returned by `/get`.

## 7. Restart the Device: `POST /restart`

This endpoint has no parameters or request body. The firmware attempts to send `204` and then immediately restarts.

```bash
curl -i -X POST http://192.168.1.50/restart
```

The connection closes immediately during restart. Some clients may report a network disconnection before receiving the complete response.

## 8. Erase Wi-Fi Credentials: `POST /erase`

This endpoint has no parameters or request body. It erases only the saved Wi-Fi SSID and password, then immediately restarts the device.

```bash
curl -i -X POST http://192.168.1.50/erase
```

After this call, the device is no longer reachable at its previous IP address and returns to the Wi-Fi provisioning flow. This is a destructive operation and should require confirmation before it is executed.

## 9. Clock-Face Numbers

| Number | Clock Face | Number | Clock Face |
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
/* Use a wider content area for the API page while retaining the right-hand outline. */
@media (min-width: 1280px) {
  .VPDoc.has-aside:has(.vp-doc._clock_en_api) .content-container {
    max-width: 900px;
  }
}

/* Desktop layout controls affect this API page only. */
@media (min-width: 960px) {
  html.api-layout-hide-left:has(.vp-doc._clock_en_api) .VPSidebar {
    display: none !important;
  }

  html.api-layout-hide-left:has(.vp-doc._clock_en_api) .VPContent.has-sidebar {
    padding-left: 0 !important;
  }

  html.api-layout-hide-right:has(.vp-doc._clock_en_api) .VPDoc > .container > .aside {
    display: none !important;
  }

  html.api-layout-hide-left:has(.vp-doc._clock_en_api) .content-container,
  html.api-layout-hide-right:has(.vp-doc._clock_en_api) .content-container {
    max-width: 1050px;
  }

  html.api-layout-hide-left.api-layout-hide-right:has(.vp-doc._clock_en_api) .content-container {
    max-width: 1200px;
  }
}

@media (min-width: 1440px) {
  html.api-layout-hide-left:has(.vp-doc._clock_en_api) .VPContent.has-sidebar {
    padding-left: calc((100vw - var(--vp-layout-max-width)) / 2) !important;
  }
}

.vp-doc._clock_en_api table {
  width: 100%;
  scrollbar-gutter: stable;
}

.vp-doc._clock_en_api table th,
.vp-doc._clock_en_api table td {
  min-width: 0;
  padding: 8px 10px;
  vertical-align: top;
  line-height: 1.5;
  white-space: normal;
}

.vp-doc._clock_en_api table th {
  min-width: 4.5rem;
}

/* Keep compact identifiers on one line and allow descriptive columns to wrap. */
.vp-doc._clock_en_api table:nth-of-type(1) :is(th, td):nth-child(1) { min-width: 9rem; }
.vp-doc._clock_en_api table:nth-of-type(1) :is(th, td):nth-child(2) { min-width: 7rem; }
.vp-doc._clock_en_api table:nth-of-type(1) :is(th, td):nth-child(3) { min-width: 14rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(1) :is(th, td):nth-child(4) { min-width: 10rem; }
.vp-doc._clock_en_api table:nth-of-type(1) :is(th, td):nth-child(5) { min-width: 7rem; }
.vp-doc._clock_en_api table:nth-of-type(1) :is(th, td):nth-child(6) { min-width: 13rem; }

.vp-doc._clock_en_api table:nth-of-type(2) :is(th, td):nth-child(1) { min-width: 8rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(2) :is(th, td):nth-child(2) { min-width: 6rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(2) :is(th, td):nth-child(3) { min-width: 18rem; }

.vp-doc._clock_en_api table:nth-of-type(3) :is(th, td):nth-child(1) { min-width: 5rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(3) :is(th, td):nth-child(2) { min-width: 15rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(3) :is(th, td):nth-child(3) { min-width: 14rem; }

.vp-doc._clock_en_api table:nth-of-type(4) :is(th, td):nth-child(1) { min-width: 5rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(4) :is(th, td):nth-child(2) { min-width: 9rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(4) :is(th, td):nth-child(3) { min-width: 13rem; }
.vp-doc._clock_en_api table:nth-of-type(4) :is(th, td):nth-child(4) { min-width: 14rem; }

.vp-doc._clock_en_api table:nth-of-type(5) :is(th, td):nth-child(1) { min-width: 9rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(5) :is(th, td):nth-child(2) { min-width: 7rem; }
.vp-doc._clock_en_api table:nth-of-type(5) :is(th, td):nth-child(3) { min-width: 18rem; }
.vp-doc._clock_en_api table:nth-of-type(5) :is(th, td):nth-child(4) { min-width: 8rem; }
.vp-doc._clock_en_api table:nth-of-type(5) :is(th, td):nth-child(5) { min-width: 13rem; }

.vp-doc._clock_en_api table:nth-of-type(6) :is(th, td):nth-child(1) { min-width: 9rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(6) :is(th, td):nth-child(2) { min-width: 9rem; }
.vp-doc._clock_en_api table:nth-of-type(6) :is(th, td):nth-child(3) { min-width: 12rem; }
.vp-doc._clock_en_api table:nth-of-type(6) :is(th, td):nth-child(4) { min-width: 18rem; }

.vp-doc._clock_en_api table:nth-of-type(7) :is(th, td):nth-child(1) { min-width: 5rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(7) :is(th, td):nth-child(2) { min-width: 5rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(7) :is(th, td):nth-child(3) { min-width: 6rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(7) :is(th, td):nth-child(4) { min-width: 17rem; }

.vp-doc._clock_en_api table:nth-of-type(8) :is(th, td):nth-child(1),
.vp-doc._clock_en_api table:nth-of-type(8) :is(th, td):nth-child(3) { min-width: 5rem; white-space: nowrap; }
.vp-doc._clock_en_api table:nth-of-type(8) :is(th, td):nth-child(2),
.vp-doc._clock_en_api table:nth-of-type(8) :is(th, td):nth-child(4) { min-width: 9rem; white-space: nowrap; }

/* Keep the field name visible while a table is scrolled horizontally on desktop. */
@media (min-width: 768px) {
  .vp-doc._clock_en_api table th:first-child,
  .vp-doc._clock_en_api table td:first-child {
    position: sticky;
    left: 0;
    z-index: 1;
  }

  .vp-doc._clock_en_api table th:first-child {
    z-index: 2;
    background: var(--vp-c-bg-soft);
  }

  .vp-doc._clock_en_api table tbody tr:nth-child(odd) td:first-child {
    background: var(--vp-c-bg);
  }

  .vp-doc._clock_en_api table tbody tr:nth-child(even) td:first-child {
    background: var(--vp-c-bg-soft);
  }
}

/* Present the two longest field-reference tables as label-value cards on mobile. */
@media (max-width: 767px) {
  .vp-doc._clock_en_api table:nth-of-type(5),
  .vp-doc._clock_en_api table:nth-of-type(6) {
    display: block;
    width: 100%;
    overflow: visible;
    border-collapse: separate;
  }

  .vp-doc._clock_en_api table:nth-of-type(5) thead,
  .vp-doc._clock_en_api table:nth-of-type(6) thead {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
    white-space: nowrap;
    clip-path: inset(50%);
  }

  .vp-doc._clock_en_api table:nth-of-type(5) tbody,
  .vp-doc._clock_en_api table:nth-of-type(6) tbody {
    display: block;
  }

  .vp-doc._clock_en_api table:nth-of-type(5) tr,
  .vp-doc._clock_en_api table:nth-of-type(6) tr {
    display: block;
    margin: 16px 0;
    overflow: hidden;
    border: 1px solid var(--vp-c-divider);
    border-radius: 10px;
    background: var(--vp-c-bg);
  }

  .vp-doc._clock_en_api table:nth-of-type(5) td,
  .vp-doc._clock_en_api table:nth-of-type(6) td {
    display: grid;
    grid-template-columns: minmax(7rem, 34%) minmax(0, 1fr);
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

  .vp-doc._clock_en_api table:nth-of-type(5) td:last-child,
  .vp-doc._clock_en_api table:nth-of-type(6) td:last-child {
    border-bottom: 0;
  }

  .vp-doc._clock_en_api table:nth-of-type(5) td::before,
  .vp-doc._clock_en_api table:nth-of-type(6) td::before {
    color: var(--vp-c-text-2);
    font-weight: 600;
  }

  .vp-doc._clock_en_api table:nth-of-type(5) td:nth-child(1)::before { content: "Response Header"; }
  .vp-doc._clock_en_api table:nth-of-type(5) td:nth-child(2)::before { content: "Type"; }
  .vp-doc._clock_en_api table:nth-of-type(5) td:nth-child(3)::before { content: "Meaning and Values"; }
  .vp-doc._clock_en_api table:nth-of-type(5) td:nth-child(4)::before { content: "Default"; }
  .vp-doc._clock_en_api table:nth-of-type(5) td:nth-child(5)::before { content: "/set Field"; }

  .vp-doc._clock_en_api table:nth-of-type(6) td:nth-child(1)::before { content: "Form Field"; }
  .vp-doc._clock_en_api table:nth-of-type(6) td:nth-child(2)::before { content: "Input Format"; }
  .vp-doc._clock_en_api table:nth-of-type(6) td:nth-child(3)::before { content: "Valid/Recommended"; }
  .vp-doc._clock_en_api table:nth-of-type(6) td:nth-child(4)::before { content: "Description"; }
}
</style>
