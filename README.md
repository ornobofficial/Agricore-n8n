# 🌱 AgriPilot — AI Farm Automation (n8n)

An n8n workflow that turns live weather data and your farm records into daily decisions: when to irrigate, what weather and pest risks to expect, what fertilizer to apply, how much you will harvest, and which machines need service.

---

## 📋 Features

| Lane | Trigger | What it does |
|:---:|---|---|
| **1. Daily farm report** | Every day, 6:00 AM | Fetches the forecast, computes water deficit and pest risk, and the AI returns an irrigation plan, weather risks and pest actions. Sent to Telegram and logged to Google Sheets. Urgent days are marked with 🚨. |
| **2. Soil analysis** | Webhook (POST) | Takes pH, N, P, K and crop, and returns a fertilizer plan with timing. |
| **3. Harvest log** | Webhook (POST) | Saves each harvest (crop, field, quantity, area, quality) to Google Sheets. |
| **4. Yield forecast** | Every Monday, 7:00 AM | Predicts the next yield per crop from harvest history, with a range and confidence level. |
| **5. Equipment reminders** | Every day, 7:00 AM | Sends a Telegram reminder for machines due for service within 7 days or overdue. |

---

## ⚙️ How it works

### Workflow overview

```text
Daily 6AM      -> Farm Config -> Open-Meteo API -> Compute metrics -> AI Advisor -> Telegram + Sheet log
Soil webhook   -> AI Soil Advisor -> Sheet log -> JSON response
Harvest webhook -> Google Sheet -> JSON response
Monday 7AM     -> Harvest history -> Aggregate stats -> AI Yield Predictor -> Telegram
Daily 7AM      -> Equipment sheet -> Find due maintenance -> Telegram
```

### Forecast metrics

**Metrics computed from the forecast:**

- Rain in the next 48 h
- Evapotranspiration
- Water deficit
- Min/max temperature
- Wind
- Average humidity
- Heat stress
- Frost risk
- Heavy rain
- A pest-risk score

---

## 📦 Requirements

- n8n (self-hosted or cloud) with the AI / LangChain nodes
- OpenAI API key (default model `gpt-4o-mini`; swappable for Claude, Gemini or a local model)
- Telegram bot (create with [@BotFather](https://t.me/BotFather)) and your chat ID
- Google Sheets account (OAuth2)
- Weather: [Open-Meteo](https://open-meteo.com/), free, no API key

---

## 🚀 Setup

### 1. Import the workflow

In n8n: **Workflows → Import from file** and select `AgriPilot_Farm_Automation.json`.

### 2. Add credentials

Add the following credentials:

- **OpenAI**
- **Telegram**
- **Google Sheets**

### 3. Configure your farm

Open the **Farm Config** node and set:

| Parameter | Description |
|---|---|
| `lat`, `lon` | Your farm location, required |
| `crop` | Crop |
| `growth_stage` | Growth stage |
| `soil_type` | Soil type |
| `area_m2` | Area in square meters |
| `irrigation_method` | Irrigation method |

### 4. Replace placeholders

- `YOUR_TELEGRAM_CHAT_ID` in the Telegram nodes
- `YOUR_SHEET_ID` in the Google Sheets nodes

### 5. Create Google Sheets tabs

Create these tabs in your Google Sheet:

| Tab | Columns |
|---|---|
| `daily_log` | date, crop, irrigation_needed, liters_per_m2, pest_risk, rain_48h, tmax, summary |
| `soil_log` | date, crop, input, advice |
| `harvest` | date, crop, field, quantity_kg, area_m2, quality, notes |
| `equipment` | equipment, last_service_date, interval_days |

### 6. Activate the workflow

Test each lane manually, then set the workflow to **Active**.

---

## 🔌 API Usage

### 1. Log a harvest

```bash
curl -X POST https://YOUR_N8N_URL/webhook/harvest-log \
  -H "Content-Type: application/json" \
  -d '{"crop":"tomato","field":"A1","quantity_kg":420,"area_m2":500,"quality":"good"}'
```

### 2. Analyze soil

```bash
curl -X POST https://YOUR_N8N_URL/webhook/soil-analysis \
  -H "Content-Type: application/json" \
  -d '{"crop":"tomato","pH":5.8,"N":20,"P":15,"K":120,"organic_matter":2.5,"area_m2":1000}'
```

---

## 🛠️ Customization

| Customization | What to change |
|---|---|
| **Different crop or location** | Edit **Farm Config**. |
| **Schedules** | Change the cron expressions in the trigger nodes. |
| **Model** | Swap the **OpenAI Model** node for another chat model. |
| **Prompts** | Edit the system messages in the AI nodes to match your crop and region. |

---

## 🗺️ Roadmap

- [ ] Commodity price alerts
- [ ] Drone image crop-health analysis with a vision model
- [ ] Greenhouse sensor alerts
- [ ] Telegram bot for logging harvests by chat
- [ ] Dashboard on top of the Google Sheet data

---

## 📝 Notes

> ⚠️ AI output is decision support, not a replacement for an agronomist. Verify fertilizer and pest advice before acting.

> 🔒 Never commit exported workflows containing real credentials, chat IDs or sheet IDs.

---

## 📄 License

**MIT**
