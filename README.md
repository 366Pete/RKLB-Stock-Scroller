# RKLB-Stock-Scroller
Scrolling stock ticker for Rocket Lab price
# 🚀 RKLB LED Stock Ticker Based off of https://github.com/lajoshanko/esphome_max7219_game_of_life project

A real-time scrolling stock ticker for **Rocket Lab USA ($RKLB)** built with a NodeMCU ESP8266 and a 4-module MAX7219 8×32 LED matrix, powered by ESPHome and Home Assistant.

Displays the current price and percent change from the previous day's close, fetched directly from Yahoo Finance every 2 minutes.

<img width="3072" height="4080" alt="PXL_20260421_144638548" src="https://github.com/user-attachments/assets/dbb9f81d-43c4-47bb-8f4a-f31017582322" />
<img width="3072" height="4080" alt="PXL_20260421_144634804" src="https://github.com/user-attachments/assets/6b1c7a03-d2df-400c-9c00-997a06e5718a" />


---

## Features

- Scrolling LED display showing `$RKLB $90.61 +5.23(+6.12%)`
- Live price data fetched from Yahoo Finance every 2 minutes
- Fully standalone — no Home Assistant required to run the display
- OTA (over-the-air) firmware updates via ESPHome
- Built with ESPHome — no custom firmware or Arduino IDE needed

---

## Parts List

| Part | Details |
|------|---------|
| NodeMCU v2 | ESP8266-based microcontroller |
| MAX7219 LED Matrix | 8×32 (4 chained 8×8 modules) |
| USB power supply | 5V, 1A minimum |
| Jumper wires | 5× female-to-female |

**Total cost:** ~$10–25 USD sourced from AliExpress or Amazon.

---

## Wiring

| NodeMCU Pin | Label | MAX7219 Pin |
|-------------|-------|-------------|
| GPIO14 | D5 | CLK |
| GPIO13 | D7 | DIN (MOSI) |
| GPIO12 | D6 | CS |
| VIN (5V) | | VCC |
| GND | | GND |

```
NodeMCU          MAX7219 Matrix
┌─────────┐      ┌─────────────┐
│ D5/GPIO14├─────►CLK          │
│ D7/GPIO13├─────►DIN          │
│ D6/GPIO12├─────►CS           │
│ VIN (5V) ├─────►VCC          │
│ GND      ├─────►GND          │
└─────────┘      └─────────────┘
```

---

## Prerequisites

- [Home Assistant](https://www.home-assistant.io/) with the [ESPHome add-on](https://esphome.io/guides/getting_started_hassio.html) installed
- Your NodeMCU connected via USB for the first flash (OTA after that)
- A 2.4GHz WiFi network (ESP8266 does not support 5GHz)

---

## Setup

### 1. Clone this repo

```bash
git clone https://github.com/YOURUSERNAME/rklb-led-ticker.git
cd rklb-led-ticker
```

### 2. Create your `secrets.yaml`

This file is **not included in the repo** and should never be committed. Create it in the same directory as `rklb-ticker.yaml`:

```yaml
# secrets.yaml
wifi_ssid: "YourWiFiName"
wifi_password: "YourWiFiPassword"
ota_password: "choose_a_password"
```

### 3. Flash the firmware

In the ESPHome dashboard in Home Assistant:

1. Click **+ New Device**
2. Choose **Open YAML** and paste in `rklb-ticker.yaml`
3. Click **Install** → **Plug into this computer** for the first flash
4. All future updates can be done wirelessly via OTA

### 4. Power it up

Once flashed and powered, the display will:
1. Connect to WiFi
2. Sync time via SNTP
3. Fetch RKLB price data within the first 2 minutes
4. Begin scrolling the ticker

---

## Configuration

All tunable settings are at the top of `rklb-ticker.yaml`:

| Setting | Location | Default | Description |
|---------|----------|---------|-------------|
| Brightness | `display: intensity:` | `5` | 0–15, adjust for room lighting |
| Scroll speed | `scroll_speed:` | `100ms` | Higher = slower scroll |
| Update interval | `interval:` | `120s` | How often to fetch new price |
| Scroll delay | `scroll_delay:` | `1000ms` | Pause before scroll starts |
| Scroll dwell | `scroll_dwell:` | `3000ms` | Pause at end before loop |

---

## Display Format

```
  $RKLB  $90.61 +5.23(+6.12%)
```

- Price is the current market price (or last close outside market hours)
- Change and percent are relative to the **previous day's closing price**
- Market hours: Monday–Friday 9:30 AM – 4:00 PM ET (NASDAQ)

---

## How It Works

The ESP8266 makes a direct HTTPS request to Yahoo Finance's chart API every 2 minutes:

```
https://query1.finance.yahoo.com/v8/finance/chart/RKLB?interval=1d&range=2d
```

It parses `regularMarketPrice` and `chartPreviousClose` from the JSON response, computes the dollar and percent change, formats the string, and updates the scrolling display. No API key required.

---

## Troubleshooting

**Display shows `$RKLB ERR`**
- Check the ESPHome logs for HTTP status code
- Ensure the device has a stable WiFi connection
- Yahoo Finance's unofficial API occasionally returns errors — the display will recover on the next fetch

**WiFi won't connect**
- ESP8266 only supports 2.4GHz — make sure your SSID is broadcasting on 2.4GHz
- Add `fast_connect: true` under `wifi:` if your router hides the SSID

**Text is upside down or mirrored**
- Change `rotate_chip: 0` to `rotate_chip: 2` in the display block

**Scroll direction is wrong**
- Toggle `reverse_enable: false` to `reverse_enable: true`

---

## Adapting for Other Stocks

To track a different ticker, change the symbol in the URL and display string:

```yaml
# In the script:
url: "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=2d"

# In the display lambda:
std::string msg = "  $AAPL  " + id(price_str) + "   ";
```

---

## License

MIT License — free to use, modify, and share.

---

## Acknowledgments
- Idea and 3D print File based on https://github.com/lajoshanko/esphome_max7219_game_of_life
- Built with [ESPHome](https://esphome.io/)
- Price data from [Yahoo Finance](https://finance.yahoo.com/)
- Inspired by a love of Rocket Lab and hardware projects
