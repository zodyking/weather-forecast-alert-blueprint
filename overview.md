# Weather Forecast Alert Blueprint — Overview

This document explains each **automation branch** in the Home Assistant blueprint **Weather — Forecast TTS (AI Optional)** in plain language. The blueprint speaks weather forecasts and alerts through your speakers (text-to-speech) in a **news-style, professional format**, with optional AI rephrasing.

**Standard opening:** Every TTS message starts with *“The time is [e.g. 7:46 AM] and here’s your weather [forecast / update].”* so announcements sound consistent and news-like.

---

## How the blueprint is structured

The automation can be **triggered** in six different ways. When it runs, it checks which trigger fired and then runs one of **four action branches**. Each branch builds a message and speaks it on your chosen speakers (and can optionally run the message through an AI to sound more like a TV weather report).

---

## The six triggers (what can start the automation)

| Trigger | What starts it |
|--------|-----------------|
| **Time-based** | A schedule (e.g. every 3 hours at a set minute). Only runs between your configured start and end times, and only on selected days of the week. |
| **Sensor trigger** | One of your presence/motion sensors turns “on” (e.g. someone arrives home). |
| **Current change** | Your weather entity’s *current condition* changes (e.g. from “sunny” to “rainy”). |
| **Upcoming change** | A built-in timer that runs every 5 minutes to see if rain/snow is coming soon. |
| **Alarm webhook** | Something calls your webhook URL (e.g. a morning alarm app hitting the webhook when you wake up). |
| **Voice satellite** | You say a configured phrase to a voice assistant (e.g. “What’s the weather”). |

Each trigger can be turned on or off in the blueprint inputs. For time-based, you also set the time window and days of the week.

---

## The four action branches (what actually runs)

### 1. Scheduled or “arrival” forecast (Time-based + Sensor trigger)

**When it runs:** The trigger was either the **time-based schedule** or a **sensor** (e.g. motion/presence) turning on.

**What it does:**

- Fetches **daily** and **hourly** forecasts from your weather entity.
- Builds a **segmented, news-style forecast** in this order:

  1. **Opening:** *“The time is [e.g. 7:46 AM] and here’s your weather forecast.”*

  2. **Current conditions:** Right now — condition, temperature, humidity. Then **today’s high and low** (e.g. *“Today’s high 78, low 62.”*).

  3. **Hourly segments (critical changes):** Up to 3 or more time points where the day’s conditions **change most noticeably**. For each time it can mention:
     - **Precipitation:** rain/snow/mixed with time and chance (e.g. *“Rain expected 60% chance”*) when probability is above your threshold.
     - **Condition changes:** when the weather type changes (e.g. *“By 4 PM, partly cloudy”* or *“By 7 PM, clearing”*).
     - **Temperature swings (greatest only):** the blueprint looks at all hourly temps in the window, ignores minor changes, and announces only the **greatest swings** — the period’s **high** and **low** and when they occur (e.g. *“High around 78 degrees”* at the warmest hour, *“Low around 62 degrees”* at the coolest hour). It only mentions these when the day’s range (high minus low) is at least your **Temperature Change Threshold** (degrees).
     Segments are combined per time (e.g. *“By 4 PM: Rain expected 60% chance, partly cloudy, high around 78 degrees.”*). You control how many such time points with **Hourly Forecast Segments** (1–6; default 3). Minor temperature changes are ignored; only the day’s high and low (when the swing meets your threshold) are announced.

  4. **Daily forecast:** **Tomorrow** and the next few days — for each day: high, low, and condition (e.g. *“Tomorrow, high 75 low 58, partly cloudy. Wednesday high 72 low 56, chance of rain.”*). You control how many days with **Daily Forecast Days** (1–5; default 3).

- If you turned on **AI rewrite**, it can rephrase that full message (without changing facts), then speaks. Otherwise it speaks the message as-is.
- Sets volume, waits the pre-roll delay, then speaks on each selected speaker.

**In short:** “Give me a full, segment-by-segment forecast: current conditions and today’s high/low, then key hourly changes (with times and %), then the next few days.”

---

### 2. “Weather just changed” update (Current change)

**When it runs:** The trigger was a **state change** on your weather entity (the current condition actually changed).

**What it does:**

- Looks at what the weather changed *from* and *to*, and the current temperature.
- Builds a short update with the **news-style opening**, then the change: *“The time is 7:46 AM and here’s your weather update. It’s now 72 degrees and partly cloudy.”*
- Only speaks if the condition **really changed** (not just a small update).
- Optional AI rewrite, then sets volume and speaks on each speaker.

**In short:** “The current weather condition just changed — open with the time and ‘weather update’, then say what it is now and the temperature.”

---

### 3. “Rain (or snow) is coming soon” alert (Upcoming change)

**When it runs:** The trigger was the **every-5-minutes** check (upcoming change).

**What it does:**

- Fetches the **hourly** forecast.
- Finds the **next** time period when the forecast says rain, snow, or other precipitation.
- Checks: right now it’s *not* precipitating, but that precip **is** expected within the next N minutes (N = your “minutes before announce” setting, e.g. 30).
- If that’s true, it builds a message with the **news-style opening**: *“The time is 7:46 AM and here’s your weather update. Rain expected in about 25 minutes.”*
- Optional AI rewrite, then speaks on each speaker at the upcoming-change volume.

**In short:** “Warn me a few minutes before rain or snow is supposed to start, with the standard time + weather update opening.”

---

### 4. “Good morning” / alarm forecast (Alarm webhook)

**When it runs:** The trigger was the **webhook** (e.g. your alarm app calling the blueprint’s webhook when you wake up).

**What it does:**

- Fetches **daily** and **hourly** forecasts.
- Builds a **personalized** wake-up message that starts with the **news-style opening** and then the forecast: *“The time is 7:46 AM and here’s your weather forecast. Currently 68 degrees and partly cloudy. Expect rain later around 4 PM with a 60% chance. High 75 degrees. Low 58 degrees.”*
- Prepends **“Good morning [your name], …”** (if you set a name in the blueprint), so the full line is e.g. *“Good morning John, The time is 7:46 AM and here’s your weather forecast. …”*
- Optional AI rewrite, then speaks on each speaker at the alarm volume.

**In short:** “When my alarm goes off (or something hits the webhook), give me a personalized morning forecast with the standard time + forecast opening.”

---

## Shared behavior across branches

- **News-style opening:** Every TTS message starts with *“The time is [time] and here’s your weather [forecast / update].”* so all announcements sound professional and consistent.
- **Speakers:** Every branch that speaks uses the same list of media players you configured (e.g. smart speakers).
- **TTS:** All speech goes through your chosen TTS engine and voice.
- **Pre-roll:** A short delay (e.g. 150 ms) runs before each TTS call so the speaker is ready and the start of the sentence isn’t cut off.
- **AI rewrite:** If you enabled it and set an AI task entity, the blueprint can send the built message to the AI to rephrase it in a “TV meteorologist” style before speaking; facts (numbers, times, conditions) stay the same.
- **Precipitation settings:** Where the blueprint mentions rain/snow, it uses your **precipitation probability threshold** and **hours ahead** so it only mentions precip when it’s relevant. The **Scheduled/arrival** branch also uses **Hourly Forecast Segments** (how many critical time points to announce), **Temperature Change Threshold** (degrees; for “temperatures rising/dropping” mentions), and **Daily Forecast Days** (how many days ahead to include).

---

## Summary table

| Branch | Trigger(s) | In one sentence |
|--------|------------|------------------|
| **Scheduled / arrival forecast** | Time-based, Sensor | Segmented forecast: opening + current conditions + today’s high/low + hourly critical changes (precip, condition, temp) + daily (tomorrow and next days). |
| **Current change** | Weather entity state change | Short update when the current weather condition actually changes. |
| **Upcoming change** | Every 5 min check | Alert a few minutes before rain/snow is expected to start. |
| **Alarm webhook** | Webhook call | Personalized “Good morning” forecast when something (e.g. alarm) calls the webhook. |

The **Voice satellite** trigger (e.g. “What’s the weather”) is configured in the blueprint; when that phrase is used, the automation is triggered. The blueprint defines four action branches above; voice uses the same trigger/condition system but does not have its own branch in the `choose` block, so voice-triggered runs may not speak unless your setup or a future blueprint version adds a branch for it.

---

*This overview describes the behavior of the blueprint as defined in `weather-forecast.yaml`; it does not modify the blueprint.*
