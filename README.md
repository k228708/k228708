<div align="center">

# 🌤️ Weather Info & Forecast Bot

### Dialogflow ES × FastAPI × OpenWeatherMap

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Dialogflow](https://img.shields.io/badge/Dialogflow-ES-FF9800?style=for-the-badge&logo=dialogflow&logoColor=white)](https://dialogflow.cloud.google.com)
[![OpenWeatherMap](https://img.shields.io/badge/OpenWeatherMap-API-E96E2C?style=for-the-badge&logo=openweathermap&logoColor=white)](https://openweathermap.org/api)
[![ngrok](https://img.shields.io/badge/ngrok-Tunnel-1F1E37?style=for-the-badge&logo=ngrok&logoColor=white)](https://ngrok.com)

> A production-ready conversational weather bot that delivers real-time and forecasted weather data
> for any city worldwide — built with Google Dialogflow ES, a FastAPI webhook backend, and the OpenWeatherMap API.

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Conversational Flow](#-conversational-flow)
- [Project Structure](#-project-structure)
- [Intents & NLP Design](#-intents--nlp-design)
- [API Reference](#-api-reference)
- [Setup & Installation](#-setup--installation)
- [Sample Outputs](#-sample-outputs)
- [Test Prompts](#-test-prompts)

---

## 🧭 Overview

This project implements a full end-to-end chatbot system for querying weather information through natural language. The bot handles:

| Capability | Description |
|---|---|
| 🌡️ **Current Weather** | Real-time weather for any global city |
| 📅 **Specific Date Forecast** | Weather for a user-specified future date |
| 📆 **N-Day Forecast** | Custom forecast window (e.g., next 3 days) |
| 🗓️ **8-Day Forecast** | Full forecast horizon in one request |
| ⚠️ **Past Date Guard** | Graceful error for historical date queries |

---

## 🏗️ Architecture

```
╔══════════════════════════════════════════════════════════════════════╗
║                        SYSTEM ARCHITECTURE                          ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║   ┌─────────────┐     Natural      ┌──────────────────┐             ║
║   │             │     Language     │                  │             ║
║   │    USER     │ ───────────────► │  DIALOGFLOW ES   │             ║
║   │  (Browser / │                  │                  │             ║
║   │   Mobile)   │ ◄─────────────── │  NLP / Intent    │             ║
║   │             │   Formatted      │  Classification  │             ║
║   └─────────────┘   Response       └────────┬─────────┘             ║
║                                             │                       ║
║                                    WebhookRequest (JSON)            ║
║                                             │                       ║
║                                    ┌────────▼─────────┐             ║
║                                    │                  │             ║
║                                    │   ngrok Tunnel   │             ║
║                                    │  (HTTPS ↔ HTTP)  │             ║
║                                    │                  │             ║
║                                    └────────┬─────────┘             ║
║                                             │                       ║
║                                    ┌────────▼─────────┐             ║
║                                    │                  │             ║
║                                    │  FASTAPI SERVER  │             ║
║                                    │   POST /webhook  │             ║
║                                    │                  │             ║
║                                    │  ┌────────────┐  │             ║
║                                    │  │  Intent    │  │             ║
║                                    │  │  Router    │  │             ║
║                                    │  └─────┬──────┘  │             ║
║                                    │        │         │             ║
║                                    │  ┌─────▼──────┐  │             ║
║                                    │  │  Weather   │  │             ║
║                                    │  │  Service   │  │             ║
║                                    │  └─────┬──────┘  │             ║
║                                    └────────┼─────────┘             ║
║                                             │                       ║
║                                      REST API Call                  ║
║                                      (city, date)                   ║
║                                             │                       ║
║                                    ┌────────▼─────────┐             ║
║                                    │                  │             ║
║                                    │  OPENWEATHERMAP  │             ║
║                                    │      API         │             ║
║                                    │  /weather        │             ║
║                                    │  /forecast       │             ║
║                                    │  /onecall        │             ║
║                                    │                  │             ║
║                                    └──────────────────┘             ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 💬 Conversational Flow

```
╔══════════════════════════════════════════════════════════════════════════╗
║                      CONVERSATIONAL FLOW DIAGRAM                        ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                          ║
║   USER                    DIALOGFLOW BOT               WEBHOOK + API     ║
║    │                            │                            │           ║
║    │──── "Hi" ─────────────────►│                            │           ║
║    │                            │                            │           ║
║    │◄─── "Hi! I'm the Weather   │                            │           ║
║    │      Bot. How may I        │                            │           ║
║    │      help you?" ───────────│                            │           ║
║    │                            │                            │           ║
║    │──── "What is the current   │                            │           ║
║    │      weather?" ───────────►│                            │           ║
║    │                            │                            │           ║
║    │◄─── "Please provide        │                            │           ║
║    │      your city." ──────────│                            │           ║
║    │                            │                            │           ║
║    │──── "My city is Karachi" ─►│                            │           ║
║    │                            │──── city ─────────────────►│           ║
║    │                            │                            │──► OWM    ║
║    │                            │                            │◄── data   ║
║    │                            │◄─── weather_info ──────────│           ║
║    │◄─── "The current weather   │                            │           ║
║    │      for Karachi is..."  ──│                            │           ║
║    │                            │                            │           ║
║    │──── "Weather forecast from │                            │           ║
║    │      April 27 for          │                            │           ║
║    │      Toronto" ────────────►│                            │           ║
║    │                            │──── city, date ───────────►│           ║
║    │                            │                            │──► OWM    ║
║    │                            │                            │◄── data   ║
║    │                            │◄─── weather_info ──────────│           ║
║    │◄─── "Forecasted weather    │                            │           ║
║    │      for Toronto Apr 27"───│                            │           ║
║    │                            │                            │           ║
║    │──── "Thanks!" ────────────►│                            │           ║
║    │◄─── "Is there anything     │                            │           ║
║    │      else I can help       │                            │           ║
║    │      you with?" ───────────│                            │           ║
║    │                            │                            │           ║
║    │──── "No thanks" ──────────►│                            │           ║
║    │◄─── "Thank you for         │                            │           ║
║    │      contacting. Goodbye!" │                            │           ║
║    │                            │                            │           ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## 📁 Project Structure

```
weather_bot/
│
├── 📄 main.py                  # FastAPI app entry point & route definitions
├── 📄 webhook_handler.py       # Intent routing & request parsing logic
├── 📄 weather_service.py       # OpenWeatherMap API integration layer
├── 📄 response_formatter.py    # Styled text output formatter
│
├── 📄 .env                     # Environment variables (API keys) — NOT committed
├── 📄 requirements.txt         # Python package dependencies
└── 📄 README.md                # This file
```

### Module Responsibilities

```
┌─────────────────────────────────────────────────────────────┐
│                        main.py                              │
│   FastAPI app — defines POST /webhook route                 │
│   Receives raw Dialogflow JSON body                         │
└──────────────────────────┬──────────────────────────────────┘
                           │ passes body dict
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   webhook_handler.py                        │
│   Reads intent name from queryResult                        │
│   Extracts city / date / number parameters                  │
│   Routes to the correct async handler function              │
└──────────────────────────┬──────────────────────────────────┘
                           │ calls with city, date, days
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   weather_service.py                        │
│   Geocodes city to lat/lon via Geocoding API                │
│   Fetches current weather or forecast from OWM              │
│   Returns structured Python dict                            │
└──────────────────────────┬──────────────────────────────────┘
                           │ raw weather dict
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  response_formatter.py                      │
│   Formats weather dict into styled human-readable text      │
│   Handles emoji mapping, box headers, date formatting       │
│   Returns fulfillmentText string back to webhook_handler    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🤖 Intents & NLP Design

### Intent Map

```
Dialogflow ES Agent — Weather Bot
│
├── Default Welcome Intent
│   └── Triggers on: "Hi", "Hello", "Hey", "Start"
│   └── Response: Greeting message (no webhook)
│
├── weather.current                          [WEBHOOK ✓]
│   ├── Required param: @sys.geo-city
│   └── Triggers on: "What's the weather in London?"
│
├── weather.forecast                         [WEBHOOK ✓]
│   ├── Required param: @sys.geo-city
│   ├── Optional param: @sys.date
│   └── Triggers on: "Forecast for Karachi"
│
├── weather.forecast.specific-date          [WEBHOOK ✓]
│   ├── Required param: @sys.geo-city
│   ├── Required param: @sys.date
│   └── Triggers on: "Weather in Toronto on April 27"
│
├── weather.forecast.days                   [WEBHOOK ✓]
│   ├── Required param: @sys.geo-city
│   ├── Required param: @sys.number
│   └── Triggers on: "Forecast for Karachi for next 3 days"
│
├── weather.thanks
│   └── Triggers on: "Thanks", "Thank you"
│   └── Response: "Is there anything else I can help you with?"
│
├── weather.goodbye
│   └── Triggers on: "Bye", "No thanks", "That's all"
│   └── Response: "Thank you for contacting. Goodbye!"
│
└── Default Fallback Intent
    └── Triggers on: Unrecognised input
    └── Response: Suggests weather-related queries
```

### Parameter Extraction Logic

```
Incoming Dialogflow Parameters
         │
         ├── "city" or "geo-city" ──► extract_city()
         │        │
         │        ├── String  → use directly
         │        └── Dict    → extract .city or .name key
         │
         ├── "date" or "date-time" ──► extract_date()
         │        │
         │        └── ISO 8601 string → datetime.fromisoformat() → date object
         │
         └── "number" or "days" ──► extract_days_number()
                  │
                  └── Float string → cast to int → capped at 5
```

---

## 📡 API Reference

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Health check |
| `GET` | `/health` | Health status (JSON) |
| `POST` | `/webhook` | Dialogflow fulfillment webhook |

### Webhook Request Format (Dialogflow → FastAPI)

```json
POST /webhook
Content-Type: application/json

{
  "queryResult": {
    "queryText": "Weather in Toronto on April 27",
    "intent": {
      "displayName": "weather.forecast.specific-date"
    },
    "parameters": {
      "geo-city": "Toronto",
      "date": "2026-04-27T12:00:00+00:00"
    }
  }
}
```

### Webhook Response Format (FastAPI → Dialogflow)

```json
{
  "fulfillmentText": "╔══════...\n📍 TORONTO — WEATHER FORECAST\n...",
  "fulfillmentMessages": [
    {
      "text": {
        "text": ["╔══...📍 TORONTO — WEATHER FORECAST..."]
      }
    }
  ]
}
```

### Intent Routing

```
POST /webhook
      │
      ▼
  Read intent.displayName
      │
      ├── "weather.current"                → handle_current_weather()
      ├── "weather.forecast"               → handle_forecast()
      ├── "weather.forecast.specific-date" → handle_specific_date_weather()
      ├── "weather.forecast.days"          → handle_forecast_n_days()
      └── (unknown)                        → default fallback message
```

---

## ⚙️ Setup & Installation

### Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Python | 3.10+ | Runtime |
| pip | latest | Package manager |
| ngrok | any | HTTPS tunnel for local dev |
| OpenWeatherMap account | — | Free API key |
| Google account | — | Dialogflow ES access |

---

### Step 1 — Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/weather-bot.git
cd weather-bot
```

---

### Step 2 — Create and activate virtual environment

```bash
# Create
python -m venv venv

# Activate on Windows
venv\Scripts\activate

# Activate on macOS / Linux
source venv/bin/activate
```

---

### Step 3 — Install dependencies

```bash
pip install -r requirements.txt
```

---

### Step 4 — Configure environment variables

Create a `.env` file in the project root:

```bash
OPENWEATHER_API_KEY=your_actual_api_key_here
```

> Get your free key at [openweathermap.org/api](https://openweathermap.org/api).
> For 5-day forecasts, subscribe to **One Call API 3.0** (free tier: 1,000 calls/day).

---

### Step 5 — Run the FastAPI server

```bash
uvicorn main:app --reload --port 8080
```

| URL | Purpose |
|-----|---------|
| `http://localhost:8080` | Server root |
| `http://localhost:8080/docs` | Swagger UI (interactive docs) |
| `http://localhost:8080/redoc` | ReDoc API docs |

---

### Step 6 — Expose with ngrok

```bash
# Port must match uvicorn port above
ngrok http 8080
```

Copy the HTTPS forwarding URL shown in the terminal:
```
Forwarding   https://xxxx-xxxx.ngrok-free.app -> http://localhost:8080
```

> ⚠️ **Common mistake**: ngrok port must match uvicorn port exactly.

---

### Step 7 — Configure Dialogflow Fulfillment

1. Open [dialogflow.cloud.google.com](https://dialogflow.cloud.google.com)
2. Select your agent → **Fulfillment** in the left sidebar
3. Toggle **Webhook** to **Enabled**
4. Paste URL: `https://xxxx-xxxx.ngrok-free.app/webhook`
5. Click **Save**

---

### Step 8 — Verify the connection

```bash
# Health check
curl https://xxxx-xxxx.ngrok-free.app/
# Expected: {"status":"ok","message":"Weather Bot Webhook is running 🌤️"}

# Test webhook directly
curl -X POST https://xxxx-xxxx.ngrok-free.app/webhook \
  -H "Content-Type: application/json" \
  -d '{"queryResult":{"intent":{"displayName":"weather.current"},"parameters":{"geo-city":"Karachi"}}}'
```

---

## 📸 Sample Outputs

### Current Weather
```
╔══════════════════════════════╗
║ 📍 KARACHI — CURRENT WEATHER ║
╚══════════════════════════════╝
🗓️  24 Apr 2026
──────────────────────────────────────
📅 Friday, 24 Apr 2026
☀️ Clear sky
🌡️  High: 36.2°C | Low: 28.1°C | Avg: 32.1°C
🌧️  Rain chance: 0%
💧 Humidity: 62%
💨 Wind: 18.3 km/h
```

### Specific Date Forecast
```
╔══════════════════════════════════╗
║ 📍 TORONTO — WEATHER FORECAST   ║
╚══════════════════════════════════╝
🗓️  27 Apr 2026
──────────────────────────────────────
📅 Monday, 27 Apr 2026
⛅ Partly cloudy
🌡️  High: 19.4°C | Low: 6.5°C | Avg: 12.6°C
🌧️  Rain chance: 15%
💧 Humidity: 48%
💨 Wind: 11.5 km/h
```

### Past Date Error
```
⚠️ Sorry! I cannot show weather for 20 Apr 2026 because it's in the past.
I can only provide:
  • Current weather 🌤️
  • Forecast for today and up to 8 days ahead 📅

Try asking: "What's the weather in Toronto today?"
```

---

## 🧪 Test Prompts

### Current Weather
```
What is the weather in Karachi?
Current weather in London
How is the weather in Dubai?
```

### Default 5-Day Forecast
```
Forecast for Karachi
Weather forecast for New York
Show forecast for Paris
```

### Specific Date (Future)
```
What will the weather be in Toronto on April 27?
Weather in London on May 1st 2026
How will the weather be in Berlin on April 30?
```

### N-Day Forecast
```
Forecast for Karachi for next 3 days
Show me 5 day forecast for Dubai
Next 2 days weather for London
```

### Past Date — Expected Error Response
```
Weather in Toronto on April 20 2026
What was the weather in Karachi on March 15?
```

### Slot Filling
```
What is the weather?
→ Bot: "Please provide your city."
→ You: Karachi
```



## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Chatbot | Google Dialogflow ES | Intent classification & entity extraction |
| Backend Framework | Python FastAPI | Webhook server & REST API |
| ASGI Server | Uvicorn | Production-grade async server |
| HTTP Client | HTTPX (async) | Non-blocking API calls to OWM |
| Weather Data | OpenWeatherMap API v2.5 / v3.0 | Real-time & forecast weather data |
| Dev Tunnel | ngrok | Expose localhost over HTTPS |
| Config | python-dotenv | Environment variable management |

---

<div align="center">

Built for **Evaluation Test 1** — Weather Info & Forecast Bot

Submitted to [submissions@interactcx.com](mailto:submissions@interactcx.com)

</div>
