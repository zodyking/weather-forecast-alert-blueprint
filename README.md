# Weather — Forecast TTS (AI Optional)

![Banner](https://raw.githubusercontent.com/zodyking/weather-forecast-alert-blueprint/refs/heads/main/imageforecast.png)

A Home Assistant blueprint that speaks a concise, TV-meteorologist-style forecast to one or more speakers on a schedule or on demand. It pulls daily + hourly forecasts, calls out likely rain/snow with daypart and probability, and can optionally pass the message through an AI task for a cleaner read while preserving facts.

---

## Quick Start

**Import the blueprint into Home Assistant:**

- **One-click import:**  
  `homeassistant://blueprint/import?url=https://raw.githubusercontent.com/zodyking/weather-forecast-alert-blueprint/refs/heads/main/weather-forecast.yaml`

- **Raw YAML (for manual import):**  
  https://raw.githubusercontent.com/zodyking/weather-forecast-alert-blueprint/refs/heads/main/weather-forecast.yaml

After import, create an automation from the blueprint and fill in the inputs described below.

---

## Features

- **Natural, compact TTS**: Greets, states “right now,” today’s high/low, and mentions the first precipitation window in the next N hours with time and %.
- **Multiple trigger modes**:
  - Time-based schedule (e.g., every 3 hours at minute X)
  - Sensor-triggered (motion/occupancy)
  - “Upcoming change” alerts based on hourly forecast proximity
  - “Current change” alerts when the weather entity’s state changes
  - Webhook for alarm/wake-up use cases
  - Voice satellite trigger via the Conversation integration
- **AI rephrasing (optional)**: Sends the crafted message to an AI Task and reads back the response. The prompt enforces factual integrity.
- **Multi-speaker output**: Choose one or many media_player entities. Per-scenario volume levels.
- **TTS pre-roll**: Millisecond delay before speaking to avoid the first syllable being cut off.

---

## Requirements

- A `weather` entity that supports `weather.get_forecasts` for **daily** and **hourly** data (e.g., OpenWeatherMap).
- A configured **TTS** engine entity (e.g., Piper TTS, Google Translate TTS) and at least one **media_player**.
- Optional: An **AI Task** entity if you enable AI rephrasing.
- Optional: Conversation integration if you use voice satellite triggers.

---

## Inputs Overview

The blueprint groups inputs for clarity. Highlights:

- **Weather Data**
  - `weather_entity`: Source of current conditions + daily/hourly forecasts.

- **TTS Settings**
  - `tts_engine`: TTS entity to synthesize speech.
  - `speakers`: One or more media_player entities for output.
  - `voice`: Voice identifier compatible with your TTS engine.
  - `volume_level`: Default volume for general announcements.
  - `preroll_ms`: Delay before TTS to wake speakers.

- **AI Rephrasing (Optional)**
  - `ai_task_entity`: AI task entity to rephrase the message.
  - `use_ai_rewrite`: Toggle to enable AI pass-through.
  - `ai_rewrite_prompt`: Strict prompt that preserves facts and formatting.

- **Time-Based Announcements**
  - `enable_time_based`: Master toggle.
  - `hour_pattern`: */1, /2, /3…* hour cadence.
  - `minute_offset`: Minute within the hour to run.
  - `start_time`, `end_time`, `days_of_week`: Windowing controls.

- **Sensor Trigger Announcements**
  - `enable_sensor_triggered`: Master toggle.
  - `presence_sensors`: Binary sensors that fire announcements when turning **on**.

- **Alarm-Based Announcement**
  - `enable_alarm_announce`: Master toggle.
  - `webhook_id`: ID for `/api/webhook/{id}`.
  - `personal_name`, `alarm_volume_level`: Personal greeting and volume.

- **Change Announcements**
  - `enable_current_change_announce` + `current_change_volume_level`
  - `enable_upcoming_change_announce` + `upcoming_change_volume_level`
  - `minutes_before_announce`: Lead time window for the next precip event.

- **Precipitation Settings**
  - `precip_threshold` (%): Minimum probability to mention.
  - `hours_ahead`: How far ahead to scan hourly data.

- **Voice Satellite**
  - `enable_voice_satellite`: Respond via Conversation triggers.
  - `conversation_command`: One phrase per line.

---

## How It Works

1. **Data fetch**: Calls `weather.get_forecasts` twice: once for `daily`, once for `hourly`.
2. **Message craft**: Builds an informative but short text:
   - Greeting + day
   - Right-now conditions (state, temperature, humidity if available)
   - Today’s high/low and expected condition if it differs from “now”
   - First precip window within `hours_ahead` if `precipitation_probability ≥ precip_threshold`, with daypart/time and %
3. **Optional AI pass**: If enabled, sends the text to the AI Task with strict instructions not to alter facts.
4. **Speak**: Sets volume, waits `preroll_ms`, then runs `tts.speak` per selected speaker.

---

## Triggers and Logic

- **Time-based**: Every `hour_pattern` hours at `minute_offset`, limited to `start_time`–`end_time` and selected days.
- **Sensor-triggered**: Any configured binary sensor turns **on**.
- **Current change**: Weather entity state changes (e.g., clear → rainy).
- **Upcoming change**: Polls every 5 minutes and announces impending precipitation within `minutes_before_announce`, only if it is not already precipitating.
- **Webhook**: `POST|PUT|GET|HEAD` to `/api/webhook/{webhook_id}` generates a personal morning-style report.
- **Voice satellite**: Conversation command matches any line in `conversation_command`.

---

## Setup Tips

- Pick a weather provider with hourly probability fields. If your provider lacks precip probability, the precip line will be suppressed.
- If you hear cut-offs, increase **TTS pre-roll** to 250–400 ms.
- Use per-scenario volume inputs to keep alerts quieter or louder than regular announcements.
- If group speakers are flaky, test with a single speaker first.

---

## Webhook Example
