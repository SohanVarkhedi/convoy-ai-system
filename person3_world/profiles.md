# PROFILES — Traffic & Weather Patterns
Author: Person 3
Version: 1.0
Last Updated: YYYY-MM-DD

This document defines default traffic patterns and weather sensitivity for the sample graph.
Person 2 will use these profiles to simulate time-of-day traffic changes and Person 1 will use them when generating synthetic training data.

------------------------------------------------------------------------
1) How to read this file
- traffic bands are time ranges (24h) with mapped traffic_level: 0=low, 1=medium, 2=high
- weather entries describe seasonal probability and effect multipliers for segments/sectors
- "sector" references are logical groups of nodes/segments to simplify simulation
------------------------------------------------------------------------

## A. Sector definitions (for convenience)
- Sector North: nodes [A, B, C]
- Sector Central: nodes [E, F]
- Sector East: nodes [G, D]
- Sector West: nodes [H]

(Use sector mapping or segment-specific entries — backend can apply sector defaults where segment-specific not provided.)

------------------------------------------------------------------------
## B. Default daily traffic profiles (apply per segment unless overridden)
Format: Segment/ Sector : [time-range -> traffic_level]

Segments with explicit patterns (copy segment ids from graph.json):

- A-B:
  - 00:00-06:00 -> 0
  - 06:00-09:00 -> 2
  - 09:00-16:00 -> 1
  - 16:00-19:00 -> 2
  - 19:00-24:00 -> 0

- B-C:
  - 00:00-06:00 -> 0
  - 06:00-10:00 -> 2
  - 10:00-17:00 -> 1
  - 17:00-20:00 -> 2
  - 20:00-24:00 -> 1

- C-D:
  - 00:00-08:00 -> 1
  - 08:00-11:00 -> 2
  - 11:00-17:00 -> 1
  - 17:00-21:00 -> 2
  - 21:00-24:00 -> 1

- A-E:
  - 00:00-05:00 -> 0
  - 05:00-09:00 -> 1
  - 09:00-18:00 -> 0
  - 18:00-22:00 -> 1
  - 22:00-24:00 -> 0

- E-F:
  - 00:00-06:00 -> 0
  - 06:00-09:00 -> 1
  - 09:00-17:00 -> 1
  - 17:00-20:00 -> 2
  - 20:00-24:00 -> 1

- B-F:
  - 00:00-07:00 -> 0
  - 07:00-10:00 -> 1
  - 10:00-16:00 -> 1
  - 16:00-19:00 -> 2
  - 19:00-24:00 -> 1

- F-G:
  - 00:00-06:00 -> 0
  - 06:00-09:00 -> 2
  - 09:00-17:00 -> 1
  - 17:00-20:00 -> 2
  - 20:00-24:00 -> 1

- G-D:
  - 00:00-06:00 -> 0
  - 06:00-10:00 -> 1
  - 10:00-17:00 -> 1
  - 17:00-21:00 -> 2
  - 21:00-24:00 -> 1

- E-C:
  - 00:00-05:00 -> 0
  - 05:00-09:00 -> 1
  - 09:00-18:00 -> 0
  - 18:00-22:00 -> 1
  - 22:00-24:00 -> 0

- C-F:
  - 00:00-06:00 -> 0
  - 06:00-09:00 -> 1
  - 09:00-17:00 -> 1
  - 17:00-20:00 -> 2
  - 20:00-24:00 -> 1

- F-H:
  - 00:00-06:00 -> 0
  - 06:00-10:00 -> 0
  - 10:00-18:00 -> 0
  - 18:00-22:00 -> 1
  - 22:00-24:00 -> 0

- H-D:
  - 00:00-05:00 -> 0
  - 05:00-09:00 -> 1
  - 09:00-17:00 -> 1
  - 17:00-20:00 -> 2
  - 20:00-24:00 -> 1

- B-G:
  - 00:00-06:00 -> 0
  - 06:00-09:00 -> 2
  - 09:00-16:00 -> 1
  - 16:00-21:00 -> 2
  - 21:00-24:00 -> 1

Default fallback (if a segment missing): Sector default
- Sector North: peak 06:00-10:00 and 16:00-19:00
- Sector Central: moderate peaks 07:00-09:00 and 17:00-19:00
- Sector East: evening peaks 17:00-21:00
- Sector West: mostly low traffic except shift-change hours

------------------------------------------------------------------------
## C. Weather profiles & seasonal probabilities

Purpose:
- Provide probability that a weather event (rain/flood) occurs in a day
- Define how weather maps to segment changes (via weather_sensitivity)

Season definitions (simplified):
- Dry season: low probability (p=0.05 per day)
- Monsoon season: high probability (p=0.25 per day)
- Winter: moderate probability (p=0.1 per day)

Sector weather probabilities (example):
- Sector North: Dry=0.05, Monsoon=0.20, Winter=0.08
- Sector Central: Dry=0.03, Monsoon=0.30, Winter=0.05
- Sector East: Dry=0.06, Monsoon=0.35, Winter=0.1
- Sector West: Dry=0.02, Monsoon=0.15, Winter=0.04

Weather effect mapping (applied to segments using weather_sensitivity):
- On heavy rain event:
  - `traffic_level += 1` (cap at 2)
  - `risk_level += 0.2 * weather_sensitivity` (cap at 1.0)
  - if weather_sensitivity > 0.6 and severity high -> set `is_blocked=true`
  - `base_time_min *= (1 + 0.3 * weather_sensitivity)` (increase proportionally)

- On light rain:
  - `traffic_level += 0` or +1 for high-sensitivity segments
  - `risk_level += 0.05 * weather_sensitivity`
  - `base_time_min *= (1 + 0.1 * weather_sensitivity)`

- On flooding:
  - For segments with weather_sensitivity >= 0.5: `is_blocked=true` (severity-dependent)
  - Else `base_time_min *= 1.5`

Notes:
- Severity levels: 1 (light), 2 (moderate), 3 (severe)
- Backend should apply caps: traffic_level ∈ {0,1,2}; risk_level ∈ [0,1]; base_time_min > 0

------------------------------------------------------------------------
## D. Time-of-day simulation guide (for Person 2)
- Simulation tick = 1 minute (or choose 5 minutes for faster)
- At each tick:
  - Update `traffic_level` for each segment based on current time-range mapping
  - If a weather event active:
    - apply weather effects (use severity & weather_sensitivity)
  - Apply incident overrides from incidents_catalog (incidents may set is_blocked)
- When generating synthetic data, sample convoy start times from realistic windows:
  - Operational convoys: early morning 02:00-06:00
  - Routine convoys: 08:00-18:00
  - Avoid testing all convoys at same time; spread them across windows

------------------------------------------------------------------------
## E. Quick examples (for Person 1 to generate rows)

Example 1:
- Segment: B-C at 08:15
- Baseline: traffic_level 2, risk_level 0.45
- Monsoon heavy rain (sensitivity=0.5, severity=2):
  - new traffic_level = min(2, 2+1)=2
  - new risk_level = min(1.0, 0.45 + 0.2*0.5 = 0.55)
  - new base_time = 22 * (1 + 0.3*0.5) = 25.3 min

Example 2:
- Segment: F-G at 07:30 (peak commuter)
- Baseline: traffic_level 2, risk_level 0.35
- No weather:
  - traffic stays 2, base_time stays 10

------------------------------------------------------------------------
## F. Implementation notes (for Person 2)
- Provide functions to:
  - get_traffic_level(segment_id, datetime)
  - apply_weather(segment_id, weather_event)
  - tick_simulation(current_time)
- Ensure deterministic behavior for demo scenarios: when running a scenario use fixed random seeds for weather/incident sampling.

End of profiles.md
