# PolygonSMSBot — Real-Time Stock Price Alert System

## Background

This system was built during my time as a Python Developer at a remote trading platform startup (2020–2021), developed for the personal use of the trading principal. It served as the human monitoring layer alongside the automated trading bot — allowing a trader to set price alerts on any US stock and receive instant notifications the moment the market hit a target level, without watching screens all day.

The bot ran in production. The included backup data files (`current_day.txt`, `previous_week_highest_volatility.txt`) are real captured market data from live sessions.

---

## What It Does

A trader sends the bot a Telegram message containing a stock ticker and a target price. The bot monitors Polygon's real-time market data stream continuously. The moment that ticker crosses the target price, the bot sends an SMS alert via Twilio and a Telegram message simultaneously.

**Example interaction:**
```
Trader → Bot:  "AAPL 150.00"
Bot:           Monitors AAPL live price
Bot → Trader:  SMS + Telegram alert when AAPL hits $150.00
```

The bot also accepts commands:
- `price of AAPL` — returns the current live price of a ticker
- `show pending commands` — lists all active price alerts in the queue

---

## Architecture

### `run/stock_bot.py` — Bot Orchestrator
Coordinates all components: Telegram listener, Polygon data stream, Messenger logic. Supports scheduled startup at a specific UTC time or immediate start. Handles daily session boundaries — a configurable UTC time marks end of trading day.

### `message_handler/messenger.py` — Command Parser & Alert Engine
Parses inbound Telegram messages and extracts ticker/price commands. Validates ticker symbols against a master list of known exchange-listed stocks (491KB company tickers CSV). Maintains a pending command queue — all active price watches in memory. Sends formatted alert messages when price targets are hit.

### `polygon_data/polygon_data_stream.py` — Real-Time Market Data
WebSocket connection to Polygon's live trade data stream. Subscribes to all tickers (`T.*`) for full market coverage. Includes backup mode — writes market state to disk every 5 minutes. On startup, can recover from last backup to avoid data loss after a crash or restart.

### `telegram_bot/telegram_bot.py` — Telegram Interface
Receives inbound messages from whitelisted Telegram user IDs only — unauthorized users are silently ignored. Sends outbound alerts and command responses. Runs in a dedicated background thread to avoid blocking the data stream.

### `twilio_sms/sms_handler.py` — SMS Delivery
Outbound SMS alerts via Twilio REST API. Supports queued sending with failure handling. Inbound SMS can also trigger the same command parsing flow as Telegram.

### `polygon_data/OTCTickerList.py` — Symbol Classification
Distinguishes OTC (over-the-counter) from exchange-listed stocks. OTC stocks are typically illiquid and subject to manipulation — filtering them out keeps alerts focused on legitimate market instruments.

---

## Data Files

| File | Description |
|------|-------------|
| `app_data/company_tickers.csv` | 491KB master list of all known ticker symbols — used for input validation |
| `app_data/non_otc_symbols.csv` | 444KB list of exchange-listed (non-OTC) symbols |
| `app_data/otc_symbols.csv` | 14KB list of OTC symbols |
| `app_data/backup_data/current_day.txt` | 987KB real market data captured during a live session |
| `app_data/week_data/previous_week_highest_volatility.txt` | 117KB weekly volatility rankings from live market data |

The backup data files are real captured production data — not generated or simulated.

---

## Security

- **Telegram whitelist:** Only explicitly allowed Telegram user IDs can interact with the bot. Messages from any other user are silently dropped.
- **Credential isolation:** All API keys (Polygon, Twilio, Telegram token) are loaded from credential files not included in this repository.

---

## Notification Channels

| Channel | Library | Use |
|---------|---------|-----|
| Telegram | pyTelegramBotAPI (telebot) | Inbound commands + outbound alerts |
| SMS | Twilio REST API | Outbound price alerts |

---

## Tech Stack

- **Language:** Python
- **Market Data:** Polygon.io WebSocket (real-time trade stream)
- **Messaging:** Twilio (SMS) · pyTelegramBotAPI (Telegram)
- **Data:** pandas · CSV symbol lists
- **Concurrency:** Python threading
- **Persistence:** Disk-based backup/recovery for session state

---

## Important Notes

This repository reflects personal work contributed during employment at a trading platform startup, shared here for portfolio reference. The system was built for private use by the trading principal.

Credentials, Telegram IDs, and Twilio account details are not included. All sensitive fields in the code are empty strings or placeholders.

This code is shared for portfolio and reference purposes only. It is not intended as financial advice.

---

## Author

**Tanjil Hasan**
github.com/tanjilh136 · linkedin.com/in/tanjil-hasan-650246169
