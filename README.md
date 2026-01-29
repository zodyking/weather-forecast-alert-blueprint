# Weather — Forecast TTS (AI Optional)

![Banner](https://raw.githubusercontent.com/zodyking/weather-forecast-alert-blueprint/refs/heads/main/imageforecast.png)

Home Assistant blueprint that speaks a concise forecast to one or more speakers on a schedule or on demand. Uses daily + hourly data, mentions first likely rain/snow window, and can optionally pass the message through an AI Task without changing facts.

## Import
- **One-click (recommended):** `homeassistant://blueprint/import?url=https://raw.githubusercontent.com/zodyking/weather-forecast-alert-blueprint/refs/heads/main/weather-forecast.yaml`
- Raw YAML: https://raw.githubusercontent.com/zodyking/weather-forecast-alert-blueprint/refs/heads/main/weather-forecast.yaml

If you see a YAML error like *"mapping values are not allowed here"* or *"--tab-size-preference: 4;"* when pasting the blueprint, use the **one-click import link** above instead of copy-paste. Pasting into the HA editor can inject extra lines (e.g. from your editor) and break the YAML.

## Requirements
- A `weather` entity that supports `weather.get_forecasts` for **daily** and **hourly** (e.g., OpenWeatherMap).
- A configured **TTS** engine entity and at least one **media_player**.
- Optional: an **AI Task** entity for rephrasing.
- Optional: Conversation integration for voice phrases.

## Features
- Natural, compact TTS: greeting, “right now,” today’s high/low, first precip window with time and %.
- Multiple triggers: schedule, sensor, upcoming change, current change, webhook, voice phrase.
- Per-scenario volume and a TTS pre-roll delay to avoid clipped audio.
- Multi-speaker output.

## Inputs (summary)
- **Weather Data**: `weather_entity`.
- **TTS Settings**: `tts_engine`, `speakers`, `voice`, `volume_level`, `preroll_ms`.
- **AI Rephrasing**: `ai_task_entity`, `use_ai_rewrite`, `ai_rewrite_prompt`.
- **Time-Based**: `enable_time_based`, `hour_pattern`, `minute_offset`, `start_time`, `end_time`, `days_of_week`.
- **Sensor Triggered**: `enable_sensor_triggered`, `presence_sensors`.
- **Alarm/Webhook**: `enable_alarm_announce`, `webhook_id`, `personal_name`, `alarm_volume_level`.
- **Change Alerts**: `enable_current_change_announce`, `current_change_volume_level`, `enable_upcoming_change_announce`, `upcoming_change_volume_level`, `minutes_before_announce`.
- **Precipitation**: `precip_threshold`, `hours_ahead`.
- **Voice Satellite**: `enable_voice_satellite`, `conversation_command`.

## Triggers
- **Schedule**: every `hour_pattern` at `minute_offset`, limited by `start_time`–`end_time` and `days_of_week`.
- **Sensor**: any configured binary sensor turns **on**.
- **Current change**: weather entity state changes.
- **Upcoming change**: checks hourly forecast every 5 min and announces if precip is expected within `minutes_before_announce` and it is not precipitating now.
- **Webhook**: call `/api/webhook/{webhook_id}`.
- **Voice**: phrases in `conversation_command`.

## Webhook example
Use the `webhook_id` you set. No body required.

## Usage notes
- If first words are clipped, raise **TTS pre-roll** to 250–400 ms.
- If group speakers are unreliable, test with one speaker first.
- If your provider lacks precip probability, the precip line is skipped.

## Backup and Development folders
- **`backup/`**: stable snapshots for rollback. Do not import from here unless reverting.
- **`development/`**: experimental drafts and tests. Expect breaking changes. Do not use in production.
- Production blueprint path: `/weather-forecast.yaml`.

## Troubleshooting
- **"mapping values are not allowed here" / "--tab-size-preference: 4;"**: Import via the one-click URL above instead of pasting YAML; pasting can inject editor/browser content and break parsing.
- No audio: verify `tts_engine` and `media_player` work via Developer Tools.
- “Forecast not available.”: confirm your provider returns hourly + daily data.
- Conversation not responding: enable the Conversation integration and match the phrase exactly.

## License
See `LICENSE` if present. Otherwise restrict use to personal testing until a license is added.
